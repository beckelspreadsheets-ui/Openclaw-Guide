# OpenClaw Setup — Windows Guide

Complete guide to set up an OpenClaw AI assistant on a Hetzner VPS from Windows. ~30 minutes.

**What you'll need:**
- Windows 10/11 with Windows Terminal or PowerShell
- Credit/debit card for Hetzner (~$8/month)
- Anthropic API key (~$20/month) + OpenAI API key (~$20/month)

**Total monthly cost:** ~$48/month

---

## Step 1: Install Windows Terminal (2 min)

If you don't have Windows Terminal:
1. Open **Microsoft Store**
2. Search for **Windows Terminal**
3. Install it

Open Windows Terminal (or PowerShell if you prefer).

---

## Step 2: Generate Your SSH Key (2 min)

In Windows Terminal:

```powershell
# Check if you already have a key
dir ~\.ssh\id_ed25519.pub

# If not found, create one:
ssh-keygen -t ed25519 -C "your-email@example.com"
# Press Enter for all defaults (no passphrase needed)

# Display the public key (copy this)
type ~\.ssh\id_ed25519.pub
```

Select and copy the entire output (starts with `ssh-ed25519`).

---

## Step 3: Create Hetzner Account (3 min)

1. Go to https://accounts.hetzner.com/login
2. Click **Sign Up** → enter email + password → verify email
3. Go to **Billing** → **Add Payment Method** → add card

---

## Step 4: Create the VPS (5 min)

1. Go to https://console.hetzner.cloud
2. Click **Create Project** → name it `OpenClaw`
3. Click **Add Server** inside the project

**Configure:**
- **Image:** Ubuntu 24.04 LTS
- **Type:** Shared CPU → **CX32 (4 vCPU, 8GB RAM, 80GB disk) → $8/month
- **Location:** Closest to you (US: Ashburn or Hillsboro)
- **SSH Key:** Click "Add SSH Key" → paste your key from Step 2 → name it `My PC`
- **Server name:** `openclaw-server`

Click **Create & Buy Now**. Wait 30 seconds.

Copy the **Public IPv4** address shown (e.g., `123.45.67.89`).

---

## Step 5: Connect to Your Server (2 min)

In Windows Terminal:

```powershell
ssh root@YOUR_IP_ADDRESS
# Type "yes" to accept fingerprint
```

You should see: `root@openclaw-server:~#`

**If SSH doesn't work:** Windows 10+ has OpenSSH built-in. If yours doesn't:
1. Settings → Apps → Optional Features → Add a feature
2. Search for "OpenSSH Client" → Install

---

## Step 6: Create OpenClaw User (3 min)

Never run OpenClaw as root. Create a dedicated user:

```bash
# Create user (set a strong password when prompted)
adduser openclaw

# Give sudo access
usermod -aG sudo openclaw

# Copy SSH key to new user
mkdir -p /home/openclaw/.ssh
cp ~/.ssh/authorized_keys /home/openclaw/.ssh/
chown -R openclaw:openclaw /home/openclaw/.ssh
chmod 700 /home/openclaw/.ssh
chmod 600 /home/openclaw/.ssh/authorized_keys

# Switch to the new user
su - openclaw
```

---

## Step 7: Install Node.js & OpenClaw (5 min)

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc

# Install Node.js 22
nvm install 22
nvm alias default 22

# Verify
node --version  # Should show v22.x.x

# Install OpenClaw
npm install -g openclaw
```

---

## Step 8: Run Setup Wizard (5 min)

```bash
openclaw setup
```

The wizard walks you through:
1. **Model provider** → Choose **Anthropic**
2. **API key** → Get from https://console.anthropic.com/settings/keys → paste it
3. **Channel** → Choose **Loopback** for now (we'll add Telegram next)

After setup, test it works:
```bash
openclaw chat
# Type a message, get a response, Ctrl+C to exit
```

---

## Step 9: Security Hardening (5 min)

```bash
# Disable password SSH login (key-only from now on)
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Set up firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw --force enable

# Install brute-force protection
sudo apt install -y fail2ban
sudo systemctl enable fail2ban

# Enable auto security updates
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## Step 10: Set Up Tailscale (5 min)

Tailscale creates a private encrypted network between your devices. Without it, you'd need to keep SSH open to the public internet (risky). With Tailscale, your server is invisible to everyone except your devices.

### 10a: Create a Tailscale Account

1. Go to https://login.tailscale.com/start
2. Sign up with **Google** or **GitHub** (whichever you prefer)
3. You'll land on the Tailscale admin console — leave this tab open

### 10b: Install Tailscale on Your VPS

SSH into your server (still using the public IP for now):

```powershell
ssh openclaw@YOUR_PUBLIC_IP
```

Then install and connect:

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Start Tailscale — this prints a login URL
sudo tailscale up
```

You'll see something like:
```
To authenticate, visit:
https://login.tailscale.com/a/abc123def456
```

**Copy that URL** → open it in your browser → click **Connect**. Your server is now on your private network.

Get your server's Tailscale IP:
```bash
tailscale ip -4
# Shows something like: 100.64.0.2
```

**Write this IP down** — you'll use it instead of the public IP from now on.

### 10c: Install Tailscale on Your Windows PC

1. Download from https://tailscale.com/download/windows
2. Run the installer → follow prompts
3. Tailscale appears in your system tray (bottom-right, near the clock)
4. Click the icon → **Log in** → sign in with the **same account** from step 10a
5. It should say **Connected** with a green dot

### 10d: Test the Private Connection

Open a **new Windows Terminal** and SSH using the Tailscale IP:

```powershell
ssh openclaw@100.64.0.2
# (use YOUR Tailscale IP from step 10b)
```

If it connects, you're golden. **From now on, always use this IP to connect.**

### 10e: (Optional) Disable Public SSH

Now that Tailscale works, you can block SSH from the public internet entirely:

```bash
# On your VPS — remove the public SSH rule and only allow Tailscale
sudo ufw delete allow ssh
sudo ufw allow in on tailscale0 to any port 22
sudo ufw reload
```

After this, the ONLY way to SSH in is through Tailscale. Much safer.

### 10f: Install Tailscale on Your Phone (Optional)

1. Install Tailscale from App Store (iOS) or Play Store (Android)
2. Log in with the same account
3. Now you can SSH from your phone too (using apps like Termius)

---
## Step 11: Run OpenClaw as a Service (3 min)

Set up systemd so OpenClaw runs 24/7 and auto-restarts:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/openclaw-gateway.service << 'EOF'
[Unit]
Description=OpenClaw Gateway
After=network-online.target

[Service]
ExecStart=%h/.nvm/versions/node/v22.22.0/bin/node \
  %h/.openclaw/lib/node_modules/openclaw/dist/entry.js \
  gateway --port 18789
Restart=always
RestartSec=5
Environment=HOME=%h

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway

# Make it survive logouts
loginctl enable-linger openclaw

# Verify it's running
systemctl --user status openclaw-gateway
```

---

## Step 12: Connect Telegram (5 min)

1. Open Telegram → search for **@BotFather**
2. Send `/newbot`
3. Pick a name (e.g., "My AI Assistant")
4. Pick a username (must end in `bot`, e.g., `alexai_bot`)
5. Copy the **token** BotFather gives you

Get your Telegram user ID:
1. Search for **@userinfobot** on Telegram
2. Send any message → it replies with your numeric ID (e.g., `123456789`)

Configure OpenClaw:
```bash
# SSH into your VPS first
ssh openclaw@100.x.y.z

# Set the bot token
openclaw config set channels.telegram.accounts.default.botToken "YOUR_BOT_TOKEN"

# Set who can talk to the bot (your numeric ID)
openclaw config set channels.telegram.dmPolicy "allowlist"
openclaw config set channels.telegram.allowFrom '["YOUR_NUMERIC_ID"]'

# Restart to apply
systemctl --user restart openclaw-gateway
```

Now send a message to your bot on Telegram. It should respond! 🎉

---

## Step 13: Install Codex CLI (3 min)

Codex gives you a second AI model (GPT-5.4) that's included with ChatGPT Pro. It also serves as your fallback when Claude hits rate limits.

```bash
# Install Codex CLI
npm install -g @openai/codex

# Login (opens a browser link — click it to authenticate)
codex login

# Configure for best results
cat > ~/.codex/config.toml << 'EOF'
model = "gpt-5.4"
reasoning_effort = "high"
reasoning_summary = "detailed"
EOF

# Test it works
codex exec "echo hello world"
```

### Set Up as Fallback Model

So your agent automatically switches to Codex when Claude is rate-limited:

```bash
# Edit openclaw.json to add fallback
openclaw config set agents.defaults.model '{"primary":"anthropic/claude-opus-4-6","fallbacks":["openai-codex/gpt-5.4"]}'
systemctl --user restart openclaw-gateway
```

Now if Claude hits its limit, your agent seamlessly continues on GPT-5.4.

## Step 14: Verify Everything (2 min)

```bash
# Gateway running?
systemctl --user status openclaw-gateway

# Tailscale connected?
tailscale status

# Firewall active?
sudo ufw status

# Node version?
node --version

# OpenClaw version?
openclaw --version
```

If all green, you're done. Your AI assistant is running 24/7.

---

## What's Next

1. **Talk to your agent** — send messages on Telegram, let it learn who you are
2. **Read the [Post-Setup Guide](post-setup-guide.md)** — lessons from 6 weeks of production use
3. **Turn on reasoning** — `openclaw config set agents.defaults.thinkingDefault high` then restart
4. **Explore skills** — check https://clawhub.com for things your agent can learn

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `ssh` not recognized | Install OpenSSH: Settings → Apps → Optional Features → OpenSSH Client |
| `openclaw: command not found` | Run `source ~/.bashrc` then `nvm use 22` |
| Bot doesn't respond | Check token + allowFrom: `openclaw config get channels.telegram` |
| Can't SSH after hardening | Use Tailscale IP (`100.x.y.z`), not public IP |
| Gateway keeps crashing | Check logs: `journalctl --user -u openclaw-gateway --since "5 min ago"` |
| Permission denied | Make sure you're user `openclaw`, not `root` |
| PowerShell shows weird characters | Use Windows Terminal instead of plain PowerShell |
