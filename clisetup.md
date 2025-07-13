# CLI Setup Guide for Beginners

## What is CLI?

CLI stands for "Command Line Interface" - it's a way to use tools by typing commands instead of clicking buttons. Don't worry! You just need to copy and paste a few commands to get everything set up.

## One-Time Setup Checklist

Complete this setup once, and you'll be ready for rapid app development forever!

### ✅ **Step 1: Install Homebrew (Mac/Linux Package Manager)** 
*Duration: 5 minutes*

**What it does:** Homebrew makes it easy to install developer tools on your computer.

**Copy and paste this command in Terminal:**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**How to verify it worked:**
```bash
brew --version
```
*Should show something like "Homebrew 4.x.x"*

---

### ✅ **Step 2: Install Node.js and npm**
*Duration: 3 minutes*

**What it does:** Node.js runs JavaScript code, npm manages packages for your projects.

**Copy and paste:**
```bash
brew install node
```

**How to verify it worked:**
```bash
node --version
npm --version
```
*Should show version numbers like "v18.x.x" and "9.x.x"*

---

### ✅ **Step 3: Install Git**
*Duration: 2 minutes*

**What it does:** Git tracks changes to your code and syncs with GitHub.

**Copy and paste:**
```bash
brew install git
```

**How to verify it worked:**
```bash
git --version
```
*Should show "git version 2.x.x"*

---

### ✅ **Step 4: Install GitHub CLI**
*Duration: 3 minutes*

**What it does:** Lets you create repositories and manage GitHub from your terminal.

**Copy and paste:**
```bash
brew install gh
```

**Login to GitHub:**
```bash
gh auth login
```
*Follow the prompts - choose "GitHub.com", "HTTPS", "Yes" to git credential helper, and "Login with a web browser"*

**Grant extra permissions (saves time later):**
```bash
gh auth refresh -h github.com -s delete_repo
```
*This prevents permission issues when Claude needs to manage repositories*

**How to verify it worked:**
```bash
gh auth status
```
*Should show you're logged in to github.com*

---

### ✅ **Step 5: Install Vercel CLI**
*Duration: 2 minutes*

**What it does:** Deploys your websites and APIs to the internet instantly.

**Copy and paste:**
```bash
npm install -g vercel
```

**Login to Vercel:**
```bash
vercel login
```
*Choose your preferred login method (GitHub recommended)*

**How to verify it worked:**
```bash
vercel --version
```
*Should show version number*

---

### ✅ **Step 6: Install PostgreSQL Client**
*Duration: 3 minutes*

**What it does:** Lets you connect to and test your Aiven database.

**Copy and paste:**
```bash
brew install postgresql@15
```

**How to verify it worked:**
```bash
/opt/homebrew/opt/postgresql@15/bin/psql --version
```
*Should show PostgreSQL version*

---

### ✅ **Step 7: Install BFG Repo-Cleaner (Security Tool)**
*Duration: 2 minutes*

**What it does:** Removes sensitive information from Git history if needed.

**Copy and paste:**
```bash
brew install bfg
```

**How to verify it worked:**
```bash
bfg --version
```
*Should show version number*

---

### ✅ **Step 8: Install Claude Code CLI**
*Duration: 5 minutes*

**What it does:** This is your AI coding assistant!

**Visit:** https://docs.anthropic.com/claude-code
**Follow the installation instructions for your operating system**

**How to verify it worked:**
```bash
claude-code --version
```

---

## Quick Setup Script (Advanced Users)

If you're comfortable with terminal, you can run everything at once:

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install all tools
brew install node git gh postgresql@15 bfg
npm install -g vercel

# Setup authentication
gh auth login
gh auth refresh -h github.com -s delete_repo
vercel login

echo "✅ All tools installed! Ready for rapid app development."
```

## What Each Tool Does in Simple Terms

| Tool | What It Does | When You Use It |
|------|-------------|----------------|
| **Homebrew** | App store for developer tools | Installing other tools |
| **Node.js** | Runs JavaScript code | Building APIs and websites |
| **Git** | Saves versions of your code | Every project |
| **GitHub CLI** | Creates online code repositories | Starting new projects |
| **Vercel CLI** | Puts your app on the internet | Making apps live |
| **PostgreSQL Client** | Connects to your database | Setting up data storage |
| **BFG** | Cleans up code history | Fixing security mistakes |
| **Claude Code** | AI coding assistant | Writing code faster |

## Troubleshooting Common Issues

### "Command not found" errors
**Problem:** Terminal says it can't find a command you just installed.

**Solution:** Close and reopen your Terminal, then try again.

### "Permission denied" errors
**Problem:** You're not allowed to install something.

**Solution:** Try adding `sudo` before the command:
```bash
sudo npm install -g vercel
```

### Homebrew installation fails
**Problem:** Homebrew won't install on your system.

**Solutions:**
- Make sure you're on Mac or Linux
- Check you have internet connection
- Try the alternative installation at https://brew.sh

### GitHub CLI won't authenticate
**Problem:** `gh auth login` doesn't work.

**Solution:** 
1. Go to https://github.com/settings/tokens
2. Create a personal access token
3. Use `gh auth login --with-token` and paste your token

### Vercel CLI deployment fails
**Problem:** `vercel` command gives errors.

**Solution:** Make sure you're logged in:
```bash
vercel whoami
```
If it says "Not logged in", run `vercel login` again.

## Verification Checklist

Before starting app development, verify everything works:

```bash
# Check all tools are installed
brew --version
node --version
npm --version
git --version
gh --version
vercel --version
/opt/homebrew/opt/postgresql@15/bin/psql --version
bfg --version
claude-code --version

# Check authentication
gh auth status
vercel whoami
```

**All commands should return version numbers or "logged in" status.**

## Time Investment

- **First-time setup:** 30 minutes
- **Future projects:** 0 minutes (everything already works!)

## What You Get

After this one-time setup, you can:
- ✅ Create new apps in 2-3 hours instead of days
- ✅ Deploy to the internet with one command
- ✅ Use AI to write most of your code
- ✅ Never worry about technical configuration again

## Next Steps

Once you've completed this setup:

1. **Create an Aiven account** at https://aiven.io
2. **Create a Vercel account** at https://vercel.com
3. **Follow the rapid replication guide** to build your first app
4. **Reference aivendbsetup.md** when connecting your database

## Windows Users

If you're on Windows, you have two options:

### Option 1: Windows Subsystem for Linux (WSL)
```bash
# Install WSL first
wsl --install

# Then follow the Linux instructions above
```

### Option 2: Windows Package Managers
```powershell
# Install using Chocolatey or Scoop
choco install nodejs git gh vercel-cli postgresql
```

## Security Notes

### Safe Commands
All the commands in this guide are safe and commonly used by developers worldwide. They:
- ✅ Install official tools from trusted sources
- ✅ Don't modify system files dangerously
- ✅ Are reversible (you can uninstall if needed)

### What We're NOT Doing
- ❌ Installing untrusted software
- ❌ Modifying critical system settings
- ❌ Exposing your computer to security risks

## Getting Help

### If you get stuck:
1. **Copy the exact error message** you see
2. **Tell Claude:** "I'm setting up my development environment and got this error: [paste error]"
3. **Claude will help you fix it** step by step

### Common error patterns:
- `command not found` → Tool didn't install correctly
- `permission denied` → Need administrator access
- `network error` → Internet connection issue
- `authentication failed` → Need to log in again

## Why This Setup Matters

### Without these tools:
- 😵 Manual file uploads to deploy websites
- 😵 Complex database configuration
- 😵 No version control or backup
- 😵 Writing all code from scratch

### With these tools:
- 🚀 One-command deployment to the internet
- 🚀 Automatic database and API setup
- 🚀 Complete project history and backup
- 🚀 AI writes most code for you

**Bottom line:** 30 minutes of setup saves you hundreds of hours on every future project.

## Ready to Build?

Once you see ✅ next to all 8 steps above, you're ready to:
- Build full-stack applications in hours
- Deploy professional websites instantly
- Use AI to accelerate your development
- Create any app idea you have

**Welcome to modern, AI-assisted development!** 🎉