# 🦞 OpenClaw Setup Guide

Your complete guide to setting up an OpenClaw AI assistant on your own VPS.

## What Is This?

OpenClaw is an open-source personal AI assistant that runs on your own server. It connects to your messaging apps (Telegram, Discord, WhatsApp, etc.), has access to tools (files, web search, shell commands), and remembers things between conversations.

Think of it like having a personal ChatGPT that:
- Runs 24/7 on a $5/month server
- Connects to YOUR apps and services
- Has persistent memory across conversations
- Can run tasks in the background while you sleep
- You fully control — your data stays on your machine

## 📚 Guides

### Setup (pick your OS)
- **[Mac Setup Guide](guides/setup-mac.md)** — VPS + OpenClaw from a Mac (~30 min)
- **[Windows Setup Guide](guides/setup-windows.md)** — VPS + OpenClaw from Windows (~30 min)

### After Setup
- **[Post-Setup Guide](guides/post-setup-guide.md)** — What we learned in 6 weeks: reasoning, memory management, automation, security, multi-agent, cost control, and every gotcha we hit

## 🧠 Agent Workspace Files

These files are what make your AI assistant "you." They're loaded every session:

| File | Purpose |
|------|---------|
| `SOUL.md` | Your agent's personality and values |
| `USER.md` | Info about you (the human) — fill this in together |
| `IDENTITY.md` | Agent's name, creature, role — discovered through conversation |
| `AGENTS.md` | Operating rules and workspace conventions |
| `MEMORY.md` | Long-term memory — curated learnings |
| `TOOLS.md` | Your environment-specific notes (SSH hosts, API keys, etc.) |
| `memory/` | Daily session logs |

### First Session Flow

When your agent wakes up for the first time, it won't have a name or identity. That's by design. Here's what happens:

1. **It reads SOUL.md and USER.md** — gets its personality foundation and learns about you
2. **It introduces itself** — acknowledges it's new and asks what you need
3. **You work together** — through real conversation, not a questionnaire
4. **Identity emerges** — it picks a name, emoji, role that fits
5. **It updates its files** — IDENTITY.md, USER.md get filled in organically

Don't rush this. The best AI assistants develop personality through use, not configuration.

## 🏗️ Building Your Agent Team

Start with one agent. Get it working. Learn how it thinks. Then expand.

### Solo Agent (Start Here)
One agent connected to Telegram. Does everything — research, writing, organizing, answering questions. This is your foundation.

### Adding Specialists (When Ready)
Once your main agent is solid, you can add specialists:
- Each new agent gets its own workspace, personality, and Telegram bot
- Agents can communicate with each other (`sessions_send`, `sessions_spawn`)
- Your main agent can orchestrate the team

**Don't build a team of 10 agents on day one.** Start with one. Add when you feel a real need.

## 💰 Costs

| Item | Cost |
|------|------|
| Hetzner VPS (CX32, 8GB RAM) | ~$8/month |
| Anthropic API (Claude) | ~$20/month |
| OpenAI API (Codex/GPT-5.4) | ~$20/month |
| **Total** | **~$48/month** |

## 📖 Resources

- **OpenClaw Docs:** https://docs.openclaw.ai
- **OpenClaw GitHub:** https://github.com/openclaw/openclaw
- **Community Discord:** https://discord.com/invite/clawd
- **Find Skills:** https://clawhub.com
