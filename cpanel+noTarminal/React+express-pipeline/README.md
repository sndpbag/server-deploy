# 🚀 React Auto Deployment Guide (GitHub → DigitalOcean)

> **Objective:**  
> Automatically deploy your **React application** from a **private GitHub repository** to a **DigitalOcean droplet** with full SSH security and seamless CI/CD pipeline.

---

## 📋 Table of Contents

- [Project Structure](#-project-structure)
- [Requirements](#-requirements)
- [Server Setup](#%EF%B8%8F-server-setup)
- [SSH Key Configuration](#-ssh-key-configuration)
- [GitHub Actions Setup](#-github-actions-setup)
- [Nginx Configuration](#-nginx-configuration)
- [Troubleshooting](#-troubleshooting)
- [Manual Deployment Commands](#-manual-deployment-commands)

---

## 📁 Project Structure

```
/var/www/myapp/frontend/IACST-School/
├── dist/              ← Built React files (production)
├── src/               ← Source code
├── public/            ← Static assets
├── package.json
└── vite.config.js
```

---

## ✅ Requirements

- ✅ GitHub private repository
- ✅ DigitalOcean droplet with SSH access
- ✅ Node.js 18+ and npm installed on server
- ✅ Git installed (`git --version`)
- ✅ Nginx web server
- ✅ Basic knowledge of Linux commands

---

## ⚙️ Server Setup

### Step 1: SSH into Your Server

```bash
ssh root@your_server_ip
```

### Step 2: Install Required Packages

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y git nodejs npm nginx

# Verify installations
node --version
npm --version
git --version
nginx -v
```

### Step 3: Create Project Directory

```bash
# Create directory structure
sudo mkdir -p /var/www/myapp/frontend/IACST-School

# Set proper ownership
sudo chown -R $USER:$USER /var/www/myapp

# Navigate to project directory
cd /var/www/myapp/frontend/IACST-School
```

---

## 🔑 SSH Key Configuration

### Step 1: Generate SSH Key on Server (Server → GitHub)

```bash
# Generate ED25519 SSH key
ssh-keygen -t ed25519 -C "deploy@iacst-school" -f ~/.ssh/deploy_key -N ""

# View the public key
cat ~/.ssh/deploy_key.pub
```

**Copy the entire output** (starts with `ssh-ed25519`)

### Step 2: Add Deploy Key to GitHub

1. Go to your GitHub repository
2. Navigate to **Settings** → **Deploy Keys** → **Add Deploy Key**
3. Fill in:
   - **Title:** `DigitalOcean Deploy Key`
   - **Key:** Paste your public key
   - ✅ Check **Allow write access** (if needed)
4. Click **Add key**

### Step 3: Test SSH Connection

```bash
ssh -T git@github.com
```

✅ Expected output:
```
Hi username/IACST-School! You've successfully authenticated, but GitHub does not provide shell access.
```

### Step 4: Clone Your Repository

```bash
cd /var/www/myapp/frontend/IACST-School

# Clone using SSH
git clone git@github.com:username/IACST-School.git .
```

---

## 🔑 SSH Key for GitHub Actions (GitHub → Server)

### Step 1: Generate SSH Key on Your Local Machine

```bash
# Generate key for GitHub Actions
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/github_actions_key -N ""

# View public key (add this to server)
cat ~/.ssh/github_actions_key.pub

# View private key (add this to GitHub Secrets)
cat ~/.ssh/github_actions_key
```

### Step 2: Add Public Key to Server

SSH into your server and add the public key:

```bash
# Add public key to authorized_keys
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... github-actions-deploy" >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Verify the key was added
cat ~/.ssh/authorized_keys
```

### Step 3: Add Private Key to GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add the following secrets:

| Secret Name | Value | Example |
|------------|-------|---------|
| `SSH_HOST` | Your server IP address | `164.52.xxx.xxx` |
| `SSH_USER` | Server username | `root` or `ubuntu` |
| `SSH_KEY` | Complete private key | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `SSH_PORT` | SSH port | `22` |

⚠️ **Important:** When adding `SSH_KEY`, paste the **complete private key** including:
- `-----BEGIN OPENSSH PRIVATE KEY-----`
- All the encrypted content
- `-----END OPENSSH PRIVATE KEY-----`

### Step 4: Test SSH Connection from Local

```bash
# Test if the key works
ssh -i ~/.ssh/github_actions_key root@your_server_ip
```

---

## 🚀 GitHub Actions Setup

### Step 1: Create Workflow File

In your repository, create `.github/workflows/deploy.yml`:

```yaml
name: 🚀 Build & Deploy React Client

on:
  push:
    branches:
      - main

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Install Dependencies & Build
        run: |
          npm ci
          npm run build

      - name: 1. Clean Old Files on Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            rm -rf /var/www/myapp/frontend/IACST-School/*
            echo "🧹 Old build cleaned."

      - name: 2. Upload New Build to Server
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          source: "dist/*"
          target: "/var/www/myapp/frontend/IACST-School"
          strip_components: 1

      - name: 3. Reload Nginx
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            sudo systemctl reload nginx
            echo "🔁 Nginx Reloaded at $(date)" >> /var/www/myapp/frontend/IACST-School/deploy.log
            echo "✅ Deployment completed successfully!"
```

### Step 2: Commit and Push

```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deployment workflow"
git push origin main
```

---

## 🌐 Nginx Configuration

### Step 1: Create Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/iacst-school
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    root /var/www/myapp/frontend/IACST-School;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### Step 2: Enable the Site

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/iacst-school /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### Step 3: Setup SSL (Optional but Recommended)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal is enabled by default
```

---

## 🔧 Troubleshooting

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| `Permission denied (publickey)` | SSH key not configured | Re-add public key to server's `authorized_keys` |
| `Could not resolve "./router/Router"` | Case-sensitive path issue | Use `git mv` to rename folder correctly |
| `ssh: handshake failed` | Private key not in GitHub Secrets | Add complete private key to `SSH_KEY` secret |
| `npm: command not found` | Node.js not installed | Run `sudo apt install nodejs npm` |
| Build files not updating | Cache issue | Clear browser cache and hard reload |
| Nginx 404 error | Wrong root path | Check `root` directive in Nginx config |

### Debugging Commands

```bash
# Check GitHub Actions logs
# Go to: Repository → Actions → Click on workflow run

# SSH into server and check
ssh root@your_server_ip

# View Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check if files deployed
ls -la /var/www/myapp/frontend/IACST-School/

# Test Nginx configuration
sudo nginx -t

# Check Nginx status
sudo systemctl status nginx

# View deployment log
cat /var/www/myapp/frontend/IACST-School/deploy.log
```

---

## 📝 Manual Deployment Commands

If automatic deployment fails, use these commands:

```bash
# 1. SSH into server
ssh root@your_server_ip

# 2. Navigate to project
cd /var/www/myapp/frontend/IACST-School

# 3. Pull latest changes
git fetch origin main
git reset --hard origin/main

# 4. Install dependencies
npm ci

# 5. Build project
npm run build

# 6. Copy build files (if needed)
# Already in correct location

# 7. Reload Nginx
sudo systemctl reload nginx

# 8. Check status
curl http://localhost
```

---

## ✅ Final Checklist

Before going live, ensure:

- [ ] Server SSH key added to GitHub Deploy Keys
- [ ] GitHub Actions SSH key added to server's `authorized_keys`
- [ ] All GitHub Secrets configured correctly (`SSH_HOST`, `SSH_USER`, `SSH_KEY`, `SSH_PORT`)
- [ ] GitHub Actions workflow file created (`.github/workflows/deploy.yml`)
- [ ] Nginx configured and running
- [ ] Domain pointing to server IP (if using custom domain)
- [ ] SSL certificate installed (recommended)
- [ ] First deployment successful
- [ ] Website accessible via browser

---

## 🎉 Deployment Flow

Every time you push to the `main` branch:

1. 🔄 GitHub Actions triggers automatically
2. 📦 Dependencies installed and project built
3. 🧹 Old files cleaned from server
4. 📤 New build files uploaded to server
5. 🔁 Nginx reloaded to serve new content
6. ✅ Website live with latest changes

**Total deployment time:** ~2-3 minutes ⚡

---

## 📚 Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [DigitalOcean Community Tutorials](https://www.digitalocean.com/community/tutorials)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Vite Build Guide](https://vitejs.dev/guide/build.html)

---

## 💬 Author

**Sandipan Kr Bag**  
Full Stack Web Developer | Trainer | Creator

📺 **YouTube:** [sndp bag 4 you](https://www.youtube.com/@sndpbag4you)  
💼 **GitHub:** [@sndpbag](https://github.com/sndpbag)  
🌐 **Portfolio:** [creazioneinteriors.in](https://creazioneinteriors.in)

---

## 📄 License

This guide is free to use and modify for your projects.

---

**⭐ If this guide helped you, please star the repository!**