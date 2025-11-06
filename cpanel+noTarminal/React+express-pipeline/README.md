🔥 Perfect Sandipan! 🔥
তুমি এখন প্রোডাকশন লেভেলে যাচ্ছো — তাই নিচে তোমার জন্য সম্পূর্ণ **GitHub-ready `README.md`** লিখে দিলাম।

এই ফাইলটা তুমি সরাসরি তোমার প্রজেক্টে কপি করে দিতে পারো
(পাথ: `/var/www/html/myproject/README.md` বা GitHub repo root)।

এটা React + Express (MERN) প্রজেক্টের জন্য তৈরি,
GitHub private repo + DigitalOcean auto deploy setup সহ 🚀

---

```markdown
# 🚀 React + Express Auto Deployment (GitHub → DigitalOcean)

> **Goal:**  
> Automatically deploy your **React + Express (MERN)** app from a **private GitHub repository**  
> to a **DigitalOcean droplet**, with full SSH security and PM2-based backend management.

---

## 📁 Project Structure

```

/var/www/html/myproject/
│
├── client/   ← React Frontend
└── server/   ← Express Backend

````

---

## 🧩 Requirements

✅ GitHub private repository  
✅ DigitalOcean droplet with SSH access  
✅ Node.js + npm installed on the server  
✅ Git installed (`git --version`)  
✅ PM2 (recommended for backend management)

---

## ⚙️ Step 1: SSH into Your Server

```bash
ssh root@your_server_ip
````

or (if using a non-root user)

```bash
ssh deploy@your_server_ip
```

---

## 🪜 Step 2: Prepare the Server

```bash
sudo apt update
sudo apt install -y git nodejs npm
```

Create your project directory:

```bash
sudo mkdir -p /var/www/html/myproject
sudo chown -R $USER:$USER /var/www/html/myproject
cd /var/www/html/myproject
```

---

## 🔑 Step 3: Setup SSH Key (Server → GitHub)

Generate SSH key on your server:

```bash
ssh-keygen -t ed25519 -C "deploy@myproject"
```

(Press Enter for all questions)

View the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire line and add it to your GitHub repo:

➡️ **GitHub → Repo → Settings → Deploy Keys → Add Deploy Key**

* Title: `DigitalOcean Deploy Key`
* Key: *(paste your public key here)*
* ✅ Check **Allow write access**

---

## 🧠 Step 4: Test the SSH Connection

```bash
ssh -T git@github.com
```

Expected output:

```
Hi sndpbag/myproject! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## 🪜 Step 5: Clone Your Project

```bash
cd /var/www/html/myproject
git clone git@github.com:sndpbag/myproject.git .
```

---

## ⚙️ Step 6: Initial Setup

### Build React

```bash
cd client
npm install
npm run build
```

### Setup Express

```bash
cd ../server
npm install
node index.js
```

✅ Test in browser → `http://your_server_ip:3000`

---

## 🧩 Step 7: Configure GitHub Actions (Auto Deploy)

In your project repo, create the file:
`.github/workflows/deploy.yml`

```yaml
name: 🚀 Auto Deploy to DigitalOcean

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            cd /var/www/html/myproject
            git fetch origin main
            git reset --hard origin/main

            # Build React client
            cd client
            npm ci
            npm run build

            # Restart Express backend
            cd ../server
            npm ci
            pkill node || true
            pm2 restart all || pm2 start index.js --name myproject

            echo "✅ Deployment completed successfully"
```

---

## 🔐 Step 8: Add GitHub Secrets

Go to → **GitHub → Repo → Settings → Secrets → Actions**

| Name       | Example                                 |
| ---------- | --------------------------------------- |
| `SSH_HOST` | `your_server_ip`                        |
| `SSH_PORT` | `22`                                    |
| `SSH_USER` | `root` or `deploy`                      |
| `SSH_KEY`  | *(paste your private key content here)* |

> ⚠️ Paste the full private key (from `/home/youruser/.ssh/id_ed25519`)

Example key format:

```
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----
```

---

## ⚙️ Step 9: Express Server Configuration

In your `server/index.js`:

```js
const express = require("express");
const path = require("path");
const app = express();

app.use(express.static(path.join(__dirname, "../client/build")));

app.get("*", (req, res) => {
  res.sendFile(path.join(__dirname, "../client/build", "index.html"));
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## ⚙️ Step 10: Run with PM2 (Production Mode)

Install PM2 globally:

```bash
sudo npm install -g pm2
```

Start your backend:

```bash
cd /var/www/html/myproject/server
pm2 start index.js --name myproject
pm2 save
pm2 startup
```

✅ Now Express restarts automatically after reboots.

---

## 🧩 Step 11: Trigger Deployment

Anytime you push changes to GitHub:

```bash
git add .
git commit -m "update navbar"
git push origin main
```

GitHub → Actions will:

1️⃣ SSH into your DigitalOcean server
2️⃣ Pull latest code
3️⃣ Build React app
4️⃣ Restart Express app via PM2
5️⃣ ✅ Deploy automatically

---

## 🧠 Step 12: Debugging Common Issues

| Issue                                    | Reason                           | Fix                                   |
| ---------------------------------------- | -------------------------------- | ------------------------------------- |
| `Permission denied (publickey)`          | Key mismatch                     | Regenerate SSH key, re-add Deploy Key |
| `ssh.ParsePrivateKey: ssh: no key found` | Incomplete key in GitHub Secrets | Re-copy full private key              |
| `npm: command not found`                 | Node missing                     | `sudo apt install nodejs npm`         |
| React build missing                      | Build failed                     | `npm run build` manually              |
| App not updating                         | Cache or wrong branch            | Clear PM2 + Browser cache             |

---

## 🧰 Manual Commands Cheat Sheet

```bash
# SSH into server
ssh root@your_server_ip

# Go to project
cd /var/www/html/myproject

# Pull latest code manually
git fetch origin main
git reset --hard origin/main

# React build
cd client && npm run build

# Restart PM2 backend
cd ../server && pm2 restart myproject

# Check logs
pm2 logs myproject
```

---

## ✅ Final Checklist

| Task                                    | Status |
| --------------------------------------- | ------ |
| SSH key created on server               | ✅      |
| Public key added to GitHub Deploy Keys  | ✅      |
| Private key added to GitHub Secrets     | ✅      |
| GitHub Action configured (`deploy.yml`) | ✅      |
| PM2 running backend                     | ✅      |
| React auto builds on deploy             | ✅      |
| Deployment success log in Actions       | ✅      |

---

## 🎉 Done!

Every time you push to GitHub `main` branch,
GitHub Actions will automatically deploy your React + Express app to DigitalOcean,
build the frontend, restart the backend, and make your website live instantly. 🚀

---

### 💬 Author

**Sandipan Kr Bag**
Full Stack Web Developer | Trainer | Creator of "sndp bag 4 you"
GitHub: [@sndpbag](https://github.com/sndpbag)

---

```

---

## ✍️ Author

**👨‍💻 Sandipan Kr Bag** — *Full Stack Web Developer*  

📺 **YouTube:** [sndp bag 4 you](https://www.youtube.com/@sndpbag4you)  
💼 **GitHub:** [@sndpbag](https://github.com/sndpbag)  
🌐 **Portfolio:** [creazioneinteriors.in](https://creazioneinteriors.in)
