# OpenClaw Setup — Windows Guide

Complete guide to set up an OpenClaw AI assistant on a Hetzner VPS from Windows. ~30 minutes.

**What you'll need:**
- Windows 10/11 with Windows Terminal or PowerShell
- Credit/debit card for Hetzner (~$5/month)
- Anthropic API key ($10-20/month for Claude)

**Total monthly cost:** ~$15-25/month

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
- **Type:** Shared CPU → **CX22** (2 vCPU, 4GB RAM) → $4.50/month
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

## Step 10: Install Tailscale (3 min)

Tailscale gives you a private, encrypted connection to your server from anywhere.

**On your VPS:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Open the URL it shows → log in with Google/GitHub → authorize.

Your server now has a private IP like `100.x.y.z`.

**On your Windows PC:**
1. Download Tailscale from https://tailscale.com/download/windows
2. Install and log in with the **same account**
3. Your PC and VPS are now on a private network

From now on, connect using the Tailscale IP:
```powershell
ssh openclaw@100.x.y.z
```

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

## Step 13: Verify Everything (2 min)

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
