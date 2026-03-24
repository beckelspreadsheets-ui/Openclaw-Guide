# After Setup — What We Learned in 6 Weeks

Everything the main guide doesn't cover. These are lessons from running OpenClaw in production across multiple businesses and 13 agents. Read this after your agent is running.

---

## 🔥 Day 1 Essentials (Do These Immediately)

### 1. Turn On Reasoning

This is the single most impactful setting. Without it, your agent gives surface-level answers that sound confident but miss nuance.

```bash
openclaw config set agents.defaults.thinkingDefault high
openclaw gateway restart
```

We ran for **weeks** without this and couldn't figure out why everything felt shallow. Don't make the same mistake.

### 2. Set Up Model Fallback

When your primary model hits rate limits, you need a backup or your agent goes silent.

```json5
// In openclaw.json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai-codex/gpt-5.4"]
      }
    }
  }
}
```

**Important:** If using Codex as fallback, the OAuth token expires every ~10 days. Set up a weekly cron to refresh it, or you'll discover it's dead at the worst possible moment.

### 3. Set Up the Gateway as a Systemd Service

The setup guide uses PM2, but systemd is more reliable — it auto-restarts on crashes and starts on boot without extra config:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/openclaw-gateway.service << 'EOF'
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/openclaw/.nvm/versions/node/v22.22.0/bin/node \
  /home/openclaw/.openclaw/lib/node_modules/openclaw/dist/entry.js \
  gateway --port 18789
Restart=always
RestartSec=5
Environment=HOME=/home/openclaw

[Install]
WantedBy=default.target
EOF

# Enable and start
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway

# Make it survive logouts
loginctl enable-linger openclaw
```

Check status anytime: `systemctl --user status openclaw-gateway`

### 4. Lock Down File Permissions

Your secrets directory should be readable only by you:

```bash
chmod 600 ~/.openclaw/secrets/*
chmod 700 ~/.openclaw/secrets/
```

We found API keys sitting at 664 (world-readable). Fix this immediately.

---

## 📁 Workspace Files — What They Actually Do

| File | Loaded When | Purpose |
|------|------------|---------|
| `CLAUDE.md` | Every session | Operating instructions for the agent |
| `SOUL.md` | Every session | Personality, values, boundaries |
| `USER.md` | Every session | Info about you (the human) |
| `IDENTITY.md` | Every session | Agent's name, creature, role |
| `AGENTS.md` | Every session | Workspace rules, conventions |
| `MEMORY.md` | Every session | Long-term curated memory |
| `TOOLS.md` | Every session | Environment-specific notes |
| `HEARTBEAT.md` | On heartbeat poll | What to check periodically |
| `BOOTSTRAP.md` | First run only | First-run instructions (delete after) |

**Keep these files lean.** Every token in these files is loaded every single message. A 2,000-line AGENTS.md wastes context on every turn.

**Target sizes:**
- CLAUDE.md: < 200 lines
- SOUL.md: < 50 lines
- USER.md: < 50 lines
- MEMORY.md: < 200 lines (archive old stuff to `memory/archive/`)
- AGENTS.md: < 100 lines

---

## 🧠 Memory Management

### The Problem
Your agent wakes up fresh every session. Without files, it remembers nothing.

### The Solution
- **Daily notes** (`memory/YYYY-MM-DD.md`) — raw session logs
- **Weekly summaries** (`memory/YYYY-WXX.md`) — consolidated weekly
- **Long-term** (`MEMORY.md`) — curated essentials
- **Archive** (`memory/archive/`) — old dailies after summarizing

### Weekly Maintenance
Every Sunday, review daily files from the past week:
1. Consolidate into `memory/YYYY-WXX.md`
2. Update `MEMORY.md` with anything worth keeping long-term
3. Move old dailies to `memory/archive/`

You can automate this with an OpenClaw cron job (see Automation section).

---

## ⏰ Automation — Cron vs Heartbeat

### Heartbeat
Your agent wakes up every N minutes and checks `HEARTBEAT.md` for tasks.

Good for: batching multiple checks together, things that can drift ±15 min.

```json5
// In openclaw.json
{
  agents: {
    defaults: {
      heartbeat: { intervalMinutes: 30 }
    }
  }
}
```

### Cron (Built-in Scheduler)
Jobs persist to disk and survive gateway restarts.

Good for: exact timing, isolated tasks, one-shot reminders.

```bash
# One-shot reminder
openclaw cron add --name "Check email" --at "20m" \
  --session isolated --message "Check inbox for anything urgent" \
  --announce --channel telegram --delete-after-run

# Recurring task
openclaw cron add --name "Weekly cleanup" --cron "0 6 * * 0" \
  --tz "America/Phoenix" --session isolated \
  --message "Consolidate memory files from this week" \
  --announce --channel telegram
```

### ⚠️ The #1 Cron Mistake

**Never set up work in your active session and then restart the gateway.** The restart kills your session and the work never runs.

**Always:** Schedule cron jobs FIRST → verify they appear in `openclaw cron list` → THEN restart.

---

## 🔗 Connecting Your Phone (Tailscale + OpenClaw Companion)

Tailscale creates a private network between your devices. After installing on your VPS:

### On Your Phone
1. Install the Tailscale app (iOS/Android)
2. Log in with the same account you used on the VPS
3. Your phone and VPS are now on the same private network

### Connecting OpenClaw Companion App (if available)
1. Install OpenClaw companion app
2. It will scan for your gateway on the Tailscale network
3. Or manually enter: `http://100.x.y.z:18789` (your VPS Tailscale IP)

### SSH from Phone
Install Termius, Blink, or any SSH app → connect to your Tailscale IP.

---

## 🛡️ Security Checklist

After your first week, verify:

- [ ] SSH password auth disabled (`PasswordAuthentication no` in `/etc/ssh/sshd_config`)
- [ ] UFW firewall active (`sudo ufw status`)
- [ ] Fail2ban running (`sudo systemctl status fail2ban`)
- [ ] Auto-updates enabled (`sudo systemctl status unattended-upgrades`)
- [ ] Tailscale connected (`tailscale status`)
- [ ] Secrets permissions locked (`ls -la ~/.openclaw/secrets/`)
- [ ] Not running as root (`whoami` → should say `openclaw`)
- [ ] Gateway auth token set (not running with open access)

---

## 💬 Multi-Channel Setup

### Telegram (Recommended Start)
Easiest to set up. One bot per agent via @BotFather.

### Discord (For Communities)
One bot per agent. Each needs Message Content Intent enabled.

### WhatsApp (For Business)
Links via QR code. More complex but great for client-facing work.

Each channel can route to different agents:
```json5
{
  bindings: [
    { agentId: "main", match: { channel: "telegram" } },
    { agentId: "work", match: { channel: "discord" } }
  ]
}
```

---

## 🤖 When to Add More Agents

**Don't add agents until you feel the pain.** Signs you need a specialist:

- Your main agent's context is regularly > 50% full
- You're switching between very different tasks (personal vs business)
- You want different personalities for different channels
- A specific task (SEO, coding, monitoring) needs deep dedicated context

### Adding Your Second Agent
```bash
openclaw agents add work
```

This creates:
- New workspace at `~/.openclaw/workspace-work/`
- New session store at `~/.openclaw/agents/work/`
- Blank SOUL.md, USER.md, IDENTITY.md for the new agent

Then create a Telegram bot for it and add bindings.

### Communication Between Agents
```json5
{
  tools: {
    agentToAgent: { enabled: true, allow: ["main", "work"] },
    sessions: { visibility: "all" }
  }
}
```

Set `session.agentToAgent.maxPingPongTurns: 1` — otherwise agents will chat back and forth burning tokens on "thanks!" "you're welcome!" loops.

---

## 💰 Cost Management

### What Burns Tokens
1. **Large workspace files** — loaded every message. Keep them lean.
2. **Heartbeats** — every 30 min = 48 agent turns/day. Use sparingly.
3. **Multi-agent chat loops** — set maxPingPongTurns to 1.
4. **Long conversations** — use `/reset` when switching topics.
5. **Subagent spawns** — each spawn is a full agent turn.

### Cost Estimates (Anthropic)
- Opus: ~$15/million input tokens, $75/million output tokens
- Sonnet: ~$3/million input, $15/million output
- Typical day (moderate use): $0.50-2.00
- Heavy build session: $5-15

### Free/Cheap Alternatives
- OpenRouter free models (Step 3.5 Flash, Gemma, etc.) for low-priority agents
- Codex OAuth (free with ChatGPT Pro subscription) for fallback

---

## 📋 Common Issues We Hit

| Problem | Cause | Fix |
|---------|-------|-----|
| Agent sounds shallow/generic | Reasoning is off | `thinkingDefault: "high"` in config |
| Agent dies during gateway restart | Session killed by restart | Schedule cron jobs BEFORE restarting |
| Model fallback doesn't work | OAuth token expired | Refresh Codex token, check `auth-profiles.json` |
| Agent can't message other agents | Session visibility too restrictive | Set `tools.sessions.visibility: "all"` |
| Subagent can't access files | Sandboxed by default | Set `sandbox.mode: "off"` on the agent |
| Telegram bot doesn't respond | Wrong allowFrom format | Use numeric user ID, not username |
| "Command not found" after SSH | nvm not loaded | `source ~/.bashrc` or `nvm use 22` |
| Agent context fills up fast | Workspace files too large | Trim MEMORY.md, archive old daily notes |
| Cron job runs but nothing happens | Wrong session target | Use `--session isolated` for standalone tasks |

---

## 📂 Recommended Folder Structure (After 1 Month)

```
~/.openclaw/workspace/
├── CLAUDE.md
├── SOUL.md / USER.md / IDENTITY.md / AGENTS.md
├── MEMORY.md
├── TOOLS.md
├── HEARTBEAT.md           ← add when you have recurring checks
├── memory/
│   ├── 2026-03-24.md      ← daily logs
│   ├── 2026-W12.md        ← weekly summaries
│   └── archive/           ← old dailies
├── research/              ← research outputs
├── plans/                 ← project plans
├── scripts/               ← automation scripts
└── work-logs/             ← task completion summaries
```

---

## 🚀 30-Day Milestones

**Week 1:** Agent running, Telegram connected, first real conversations, identity established.

**Week 2:** Memory system working (daily notes accumulating), HEARTBEAT.md added for periodic checks, first cron job set up.

**Week 3:** Skills installed (web search, GitHub, email), agent helping with real work, considering second agent.

**Week 4:** Agent team forming (if needed), automation running, workflow patterns established. You should be spending less time managing the agent and more time working alongside it.
