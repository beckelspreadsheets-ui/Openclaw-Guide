# OpenClaw on Hetzner VPS — Complete Beginner's Guide
## For Alexander (Your Friend)

---

## 📋 Overview

We're setting up a $4.50/month VPS on Hetzner to run OpenClaw 24/7. Entire process ~30 minutes.

**What you'll need:**
- Credit/debit card (Hetzner charges ~$5/month)
- SSH key for security (I'll help you generate one)
- Terminal access (Mac/Linux) or PuTTY (Windows)

**Total monthly cost after setup:**
- VPS: $4.50
- AI APIs (Anthropic/OpenAI): $10-15
- **Total: ~$15/month**

---

## 🔐 PHASE 0: PREPARATION (5 min)

### Step 1: Create SSH Key (if you don't have one)

**On Mac/Linux:**
```bash
# Check if you already have an SSH key
ls -la ~/.ssh/id_*.pub

# If you see "No such file", create one:
ssh-keygen -t ed25519 -C "alex@example.com"
# Press Enter for all defaults (no password needed)
```

**On Windows (PuTTY):**
- Download PuTTY from putty.org
- Open PuTTYgen → "Generate" → move mouse
- Save public key as `alex-key.pub`
- Save private key as `alex-key.ppk` (keep this safe!)

---

## 🌐 PHASE 1: HETZNER ACCOUNT (5 min)

### Step 2: Go to Hetzner Cloud
- URL: https://accounts.hetzner.com/login
- Click **"Sign Up"**
- Use your real email (alex@example.com)
- Create a strong password
- Verify email (check inbox, click link)

### Step 3: Add Payment Method
- Log into your new account
- Go to **Billing** → **Add Payment Method**
- Add credit card (Visa/Mastercard only)
- No prepayment needed — monthly billing

---

## 🖥️ PHASE 2: CREATE THE VPS (10 min)

### Step 4: Login to Hetzner Cloud Console
- URL: https://console.hetzner.cloud
- Login with your new credentials

### Step 5: Create a New Project (Optional but clean)
- Click **"Create Project"** or use default
- Name it `OpenClaw`
- Click **"Create Project"**

### Step 6: Add a Server

In the project dashboard:

1. **Select Image:**
   - Scroll to "Images" section
   - Choose **Ubuntu 24.04 LTS** (most compatible with OpenClaw)
   - NOT Ubuntu 22.04, NOT Debian

2. **Choose Type:**
   - **Location:** Pick the closest to you (Falkenstein Germany or Nuremberg)
   - **Type:** **Shared CPU** (cheapest)
   - **Plan:** **CX21** (2 vCPU, 4GB RAM, 40GB disk) — $4.50/month
   - **Price:** ~$4.50/mo ( billed hourly)
   - Volume type: **Local** (fastest)

3. **Add SSH Key:**
   - Click **"Add SSH Key"**
   - **Title:** `Alexander's OpenClaw`
   - **Public Key:** Paste contents of your `~/.ssh/id_ed25519.pub` file
     (Mac/Linux: `cat ~/.ssh/id_ed25519.pub | pbcopy` or `xclip`)
   - **OR** if using PuTTY, click "Use browser" to upload the `.pub` file
   - Click **"Add SSH Key"**

4. **Name your server:**
   - `openclaw-server` or anything you like

5. **Click "Create & Buy Now"**
   - Review: Ubuntu 24.04, CX21, $4.50/mo
   - Accept terms
   - Click **"Create Server"**

**Wait 30 seconds** — Hetzner will provision your server.

---

## 💻 PHASE 3: CONNECT TO YOUR SERVER (5 min)

### Step 7: Get Your Server IP

- After provisioning, you'll see a "Public IPv4" address: `123.456.789.123`
- Copy that IP address

### Step 8: Connect via SSH

**Mac/Linux:**
```bash
ssh root@123.456.789.123
# Type "yes" the first time to accept the fingerprint
```

**Windows (PuTTY):**
- Open PuTTY
- Hostname: `123.456.789.123`
- Port: 22
- Connection → SSH → Auth → Browse → select your `alex-key.ppk`
- Click **"Open"**

You should now see a black screen with:
```
Welcome to Ubuntu 24.04 LTS...
root@your-server:~#
```

**You're connected!**

---

## 🛠️ PHASE 4: INITIAL SERVER SETUP (10 min)

### Step 9: Create a Non-Root User

OpenClaw should never run as root. We'll create a user called `openclaw`:

```bash
# Create user
adduser openclaw
# Follow prompts:
# - Enter password (make something strong, write it down)
# - Confirm password
# - Name: "OpenClaw"
# - Room: press Enter
# - Phone: press Enter
# - Other: press Enter
# - Is this correct? Y

# Add to sudo group
usermod -aG sudo openclaw

# Copy your SSH key to the new user
mkdir -p /home/openclaw/.ssh
cp ~/.ssh/authorized_keys /home/openclaw/.ssh/
chown -R openclaw:openclaw /home/openclaw/.ssh
chmod 700 /home/openclaw/.ssh
chmod 600 /home/openclaw/.ssh/authorized_keys

# Switch to the new user
su - openclaw
```

You should now see: `openclaw@your-server:~$`

---

## 📦 PHASE 5: INSTALL OPENCLAW (10 min)

### Step 10: Install Node.js v22

OpenClaw requires Node.js version 22 or higher.

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# Reload your shell
source ~/.bashrc

# Install Node.js 22
nvm install 22
nvm use 22
nvm alias default 22

# Verify
node --version
# Should output: v22.x.x
```

If you see `v18` or `v20`, the nvm install failed — let me know.

---

### Step 11: Install OpenClaw Globally

```bash
npm install -g openclaw

# Verify installation
openclaw --version
```

Should show something like `OpenCLaw CLI v2026.3.x`.

---

## ⚙️ PHASE 6: RUN SETUP WIZARD

### Step 12: Start Interactive Setup

```bash
openclaw setup
```

The wizard will ask you questions:

**1. AI Provider:**
- Anthropic (Claude) — recommended, best quality
- OpenAI (GPT) — cheaper, good quality
- Google (Gemini) — cheapest

**Recommendation for Alexander:** **Anthropic Claude Sonnet 4** (best balance)

**2. API Key:**
- You'll need an API key from your chosen provider
- Go to: console.anthropic.com (or platform.openai.com, or aistudio.google.com)
- Create an account if needed
- Create API key (usually a 32-character string)
- Copy it, paste when prompted

**3. Gateway Mode:**
- Choose **"Loopback"** (localhost only) — **SAFEST**
- This keeps the gateway private to your server only

**4. Model:**
- Claude Sonnet 4 (if using Anthropic)
- GPT-4.1 (if using OpenAI)
- Gemini Pro (if using Google)

**5. Channels:**
- Choose **Telegram** (easiest) or Discord
- Telegram guide: https://docs.openclaw.com/channels/telegram
- Discord guide: https://docs.openclaw.com/channels/discord

*If you need help with Telegram bot setup, I'll walk you through that separately.*

---

## 🔒 PHASE 7: SECURITY HARDENING (CRITICAL)

**Do NOT skip this.** 21,000+ OpenClaw instances were exposed in 2026 because people skipped security.

### Step 13: Disable SSH Password Authentication

```bash
sudo nano /etc/ssh/sshd_config
```

Find and change these lines (or add if missing):
```
PasswordAuthentication no
PermitRootLogin no
```

Save (Ctrl+X, then Y, then Enter), then restart SSH:

```bash
sudo systemctl restart sshd
```

### Step 14: Set Up Firewall (UFW)

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
```

Type `y` when prompted.

---

### Step 15: Install Fail2Ban (Brute Force Protection)

```bash
sudo apt update
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check it's running:
```bash
sudo systemctl status fail2ban
```

---

### Step 16: Enable Automatic Security Updates

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -f noninteractive unattended-upgrades
```

---

## 🌍 PHASE 8: TAILSCALE (PRIVATE NETWORK)

Tailscale lets you securely access your VPS from your laptop/phone without opening ports to the internet.

### Step 17: Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

You'll see an output like:
```
To authenticate, visit: https://login.tailscale.com/start/xxxxx
```

**Open that URL on your computer/phone** → log in with a Google/GitHub account → authorize.

After that, your server gets a private Tailscale IP like `100.x.y.z`. From now on, you SSH to that IP instead of the public one:

```bash
ssh openclaw@100.x.y.z
```

This keeps your server hidden from the public internet. ✅

---

## 🚀 PHASE 9: START OPENCLAW AS A SERVICE

### Step 18: Install PM2 (Process Manager)

```bash
sudo npm install -g pm2
```

### Step 19: Start OpenClaw with PM2

```bash
cd ~
pm2 start "openclaw gateway start" --name openclaw
pm2 save
pm2 startup
# Follow the instructions PM2 prints (copy-paste the command it gives)
```

Your OpenClaw is now running as a background service. It will auto-start on reboot.

### Step 20: Check Status

```bash
pm2 status
```

Should show `online` next to `openclaw`.

---

## 📱 PHASE 10: CONNECT A MESSAGING CHANNEL

### Step 21: Set Up Telegram Bot (easiest)

1. Open Telegram, search for **@BotFather**
2. Send `/newbot`
3. Choose name: `YourNameOpenClaw`
4. Choose username: must end in `bot` (e.g., `myopenclawbot`)
5. Copy the **Bot Token** (looks like `1234567890:ABCdefGhIJKlmNoPQRsTUVwxyz`)
6. Run on server:
   ```bash
   openclaw config set channels.telegram.token "PASTE_TOKEN_HERE"
   ```
7. Get your Telegram User ID:
   - Message `@getidsbot` in Telegram
   - Copy the numeric ID
   ```
   openclaw config set channels.telegram.allowFrom "YOUR_USER_ID"
   ```
8. Restart:
   ```bash
   pm2 restart openclaw
   ```

Now message your bot in Telegram — it should respond! 🎉

---

## ✅ PHASE 11: TEST EVERYTHING

### Step 22: Check Configuration

```bash
openclaw doctor
```

This checks everything:
- Node.js version (should be 22)
- Permissions
- Gateway reachable
- Any issues

Fix anything it flags.

### Step 23: Send a Test Message

Go to your Telegram bot (or Discord channel), send:

```
Hello! What can you do?
```

Your AI should reply. If it does — **success**. You now have your own AI assistant running 24/7 on a $4.50 VPS.

---

## 🆘 TROUBLESHOOTING

**"openclaw: command not found"**
- Run `source ~/.bashrc` or open a new terminal
- Make sure nvm is loaded: `nvm use 22`

**"Permission denied" when using sudo**
- OpenClaw should NOT be run as root. Make sure you're user `openclaw`:
  `whoami` should return `openclaw`
- If not: `su - openclaw`

**Bot doesn't respond in Telegram**
- Check token: `openclaw config get channels.telegram.token`
- Check allowFrom: `openclaw config get channels.telegram.allowFrom` (must be YOUR numeric user ID, not username)
- Check status: `pm2 logs openclaw` (look for errors)
- Restart: `pm2 restart openclaw`

**Server won't SSH**
- Are you using the Tailscale IP? (`tailscale ip`)
- Is the server running? Check in Hetzner console (should say "Running")

---

## 💰 ONGOING COSTS

Monthly bill from Hetzner:
- CX21 server: **$4.50** (billed hourly, max $4.50/mo)
- Backups ($1 optional): skip
- Snapshots ($0.10/GB): skip

API costs separate:
- Anthropic: ~$8-20/month (depending on usage)
- Total: ~$15/month for a fully functional AI assistant

---

## 📞 NEED HELP?

Contact Seth if anything's unclear. Or ask in the OpenClaw Discord:
- Server: https://discord.gg/clawd
- Channel: #users-helping-users

---

## ⏱️ QUICK RECAP (15 min video script)

1. Sign up for Hetzner Cloud → add payment
2. Create Ubuntu 24.04 CX21 server → add SSH key
3. SSH into server → create `openclaw` user → copy SSH key
4. Install nvm → Node.js 22 → OpenClaw CLI
5. Run `openclaw setup` → pick Anthropic → paste API key → Loopback mode
6. Hardening: disable root login, disable password auth, ufw firewall, fail2ban
7. Install Tailscale → `tailscale up` → authenticate via browser
8. PM2: `pm2 start openclaw` → `pm2 save` → `pm2 startup`
9. Set up Telegram bot: @BotFather → get token → `openclaw config set`
10. Test PM2, test Telegram message

Done. Server costs $4.50/mo, lives forever 24/7. AI costs extra.

---

## 🎯 WHAT'S NEXT

Once OpenClaw is running:
- Add custom skills from ClawHub (be careful — vet them)
- Edit `~/.openclaw/SOUL.md` to customize your AI's personality
- Set up more channels (Discord, WhatsApp)
- Install monitoring: `openclaw doctor --watch` to monitor health

Enjoy your own AI assistant. 🦝
