(Perfect Explanation)

GitHub Actions → তোমার DigitalOcean সার্ভারে auto deploy করতে চাইলে,
সেটার জন্য দুইটা key লাগে:

Private Key → GitHub-এর কাছে থাকে

Public Key → Server এর কাছে থাকে

এ দুটো key একসাথে মিললে password ছাড়া secure connection তৈরি হয়।

✅ STEP 1: Local Machine-এ Deploy Key Generate করো
CMD দিয়ে generate

ssh-keygen -t ed25519 -C "github-actions-deploy" -f "%USERPROFILE%\\.ssh\github_actions_key" -N ""

🎉 Output এরকম হবে:
Your identification has been saved in github_actions_key
Your public key has been saved in github_actions_key.pub


Then files will be created:

C:\Users\user\\.ssh\github_actions_key
C:\Users\user\\.ssh\github_actions_key.pub

✅ STEP 2: Public Key → Server-এর authorized_keys এ যোগ করো

🔹 Public key open korar command (PowerShell/CMD)

type C:\Users\your-username\\.ssh\github_actions_key.pub

Example:
type C:\Users\user\\.ssh\github_actions_key.pub

নিচের মত কোড দেখা যাবে ওটা কপি কর 
C:\Users\user>type C:\Users\user\.ssh\github_actions_key.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINEQ1K+vXCySA3WGk5IiB8kV/gGtQRthw5lBKL8ByV0J github-actions-deploy

✅ 1 তারপর server এ SSH করে ঢুকো:

ssh root@YOUR_SERVER_IP

ssh root@209.38.120.113

then
password - Enfash2025!@#IN

(তুমি root ব্যবহার করছো, এটা ঠিক আছে)

✅ 2. .ssh folder ache ki na check korbo

ls -la /root

Output e jodi dekhো:
.ssh

➡️ Tahole .ssh folder already ache
❌ Jodi .ssh na thake → tumi niche command diye create korte paro:
mkdir -p /root/.ssh

✅ 3. authorized_keys ache ki na check korbo
Command dao:
ls -la /root/.ssh

Jodi output e dekhao:
authorized_keys
➡️ Tahole public key already set ache
ekhon "nano authorized_keys" lekho tar por public key ta niche paste koro, er por
ctrl+O
Enter
ctrl + X

❌ Jodi na thake → tumi create korte paro:
nano /root/.ssh/authorized_keys

Then GitHub Actions key er public key paste koro
Save → CTRL + O, Enter
Exit → CTRL + X

🔐 Very Important: Permission set kore dao
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys


🧪 STEP 4: Local থেকে Test
ssh -i ~/.ssh/github_actions_key root@YOUR_SERVER_IP




ssh -i ~/.ssh/github_actions_key root@209.38.120.113

যদি login হয়:
Welcome to Ubuntu 25.04 (GNU/Linux 6.14.0-29-generic x86_64)


এখন তোমার কাজ হলো local এ তৈরি করা Private Key → GitHub private repo-র Actions Secret এ যোগ করা।
✅ STEP 1 — Local Private Key ওপেন করো
CMD (Windows):
type %USERPROFILE%\\.ssh\github_actions_key

এটার output শুরু হবে:

-----BEGIN OPENSSH PRIVATE KEY-----


এবং শেষ হবে:

-----END OPENSSH PRIVATE KEY-----

👉 এটা পুরোটা একটুও বাদ না দিয়ে কপি করবে।

✅ STEP 2 — GitHub Repo তে Secret হিসাবে যোগ করো

তোমার private repo তে যাও:
GitHub → Repo → Settings → Secrets and variables → Actions

তারপর click:

➕ New repository secret
Secret Name লিখো:

📌 GitHub Secrets Required (Table Format)
Secret Name	Value / কী দিতে হবে	Example
SSH_PRIVATE_KEY	তোমার লোকালে generate করা github_actions_key → পুরো private key (BEGIN → END 

👉 এটা পুরোটা একটুও বাদ না দিয়ে কপি করবে।

---

## ✅ STEP 2 — GitHub Repo তে Secret হিসাবে যোগ করো

GitHub → Repo → Settings → Secrets and variables → Actions  
➕ New repository secret

---

# 📌 GitHub Secrets Required (Table Format)

| Secret Name       | Value / কী দিতে হবে                                            | Example                   |
|------------------|---------------------------------------------------------------|---------------------------|
| SSH_PRIVATE_KEY  | তোমার লোকালে generate করা github_actions_key → পুরো private key | your private key          |
| SSH_HOST         | তোমার DigitalOcean server-এর IP address                        | 167.71.xx.xx              |
| SSH_USER         | server user (তুমি root ব্যবহার করছো)                            | root                      |
| SSH_PORT         | SSH port number                                                | 22                        |
| REMOTE_PATH      | কোন folder এ deploy হবে                                        | /var/www/myapp/client     |
| PM2_APP_NAME     | Backend এর pm2 name                                            | myapp-api                 |

---

## 🚀 PM2_APP_NAME কোথায় পাবো?

Server এ SSH করে ঢুকে রান করো:



/var/www/myapp/client
/var/www/myapp/admin
/var/www/myapp/server


PM2_APP_NAME     myapp-api



🚀 PM2_APP_NAME কোথায় পাবো?

Server এ SSH করে ঢুকে রান করো:

pm2 list

এখানে output এ "Name" নামে একটা কলাম থাকবে:
| id | name         | status |
| -- | ------------ | ------ |
| 0  | myapp-server | online |
| 1  | api-server   | online |




✅ STEP 3 — এই একই steps তোমার ৩টা private repo-তেই করবে

কারণ:

client

admin

server

👉 তিনটিই আলাদা GitHub Actions workflow চালাবে
👉 তাই প্রতিটা repo-তে secret থাকা বাধ্যতামূলক

যদিও Private key → ৩টা repo-তেই একই key থাকবে।
এটা একদম perfectly ঠিক।


🚀 Express server Auto Deploy Workflow (deploy-client.yml)

👉 এই ফাইলটি Client repo-তে রাখবে:

.github/workflows/deploy.yml


name: Deploy Express Server

on:
  push:
    branches: [ "main" ]   # main branch এ push হলেই deploy হবে

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1) Repo Checkout
      - name: Checkout Code
        uses: actions/checkout@v4

      # 2) Setup Node Version
      - name: Use Node 18
        uses: actions/setup-node@v4
        with:
          node-version: "18"

      # 3) Install Dependencies (Server Side Build যদি লাগে)
      - name: Install Dependencies
        run: npm install

      # 4) SSH Private Key Load
      - name: Add SSH key
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      # 5) Server Connection Test (Optional)
      - name: Add server to known_hosts
        run: ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      # 6) Upload code to server
      - name: Upload Server Files using rsync
        run: |
          rsync -avz --delete \
            --exclude=".git" \
            --exclude="node_modules" \
            ./ ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:${{ secrets.REMOTE_PATH }}/

      # 7) Install production dependencies + PM2 Restart
      - name: Install & Restart on Server
        run: |
          ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'
            cd ${{ secrets.REMOTE_PATH }}
            npm install --production
            
            # PM2 Restart
            pm2 reload ${{ secrets.PM2_APP_NAME }} || pm2 start index.js --name "${{ secrets.PM2_APP_NAME }}"
            
            pm2 save
          EOF




🔥 Required GitHub Secrets (Client Repo)

Client repo তে এই চারটি secret অবশ্যই দিবে👇

| Secret Name         | Value                             |
| ------------------- | --------------------------------- |
| **SSH_PRIVATE_KEY** | তোমার local থেকে তৈরি private key |
| **SSH_HOST**        | DigitalOcean Server IP            |
| **SSH_USER**        | root                              |
| **SSH_PORT**        | 22                                |
| **REMOTE_PATH**     | `/var/www/myapp/client`           |

📌 Note: Client repo-তে PM2_APP_NAME লাগবে না (React build backend চালায় না)।


🚀 React Client Auto Deploy Workflow (deploy-client.yml)

👉 এই ফাইলটি Client repo-তে রাখবে:

.github/workflows/deploy.yml

এবং Push → Auto Deploy → Nginx Live Update।
✅ React Client Deploy Workflow (Copy–Paste Ready)


name: Deploy React Client

on:
  push:
    branches: [ "main" ]   # main branch এ push হলেই deploy

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1) Checkout
      - name: Checkout Code
        uses: actions/checkout@v4

      # 2) Setup Node
      - name: Use Node 18
        uses: actions/setup-node@v4
        with:
          node-version: "18"

      # 3) Install Dependencies
      - name: Install Dependencies
        run: npm install

      # 4) Build React App
      - name: Build React Client
        run: npm run build

      # 5) Load SSH Private Key
      - name: Add SSH key
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      # 6) Add server to known_hosts
      - name: Add server to known_hosts
        run: ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      # 7) Deploy Build folder to Server
      - name: Upload build folder via rsync
        run: |
          rsync -avz --delete \
            ./dist/ \
            ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:${{ secrets.REMOTE_PATH }}/

      # 8) Restart nginx (optional)
      - name: Reload Nginx
        run: |
          ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} "sudo systemctl restart nginx"



🚀 Admin Panel Auto Deploy Workflow (Vite Build)

File path (Admin repo ভিতরে):
.github/workflows/deploy.yml

Copy → Paste → Done!



name: Deploy Admin Panel (Vite)

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1) Checkout Code
      - name: Checkout Code
        uses: actions/checkout@v4

      # 2) Setup Node
      - name: Use Node 18
        uses: actions/setup-node@v4
        with:
          node-version: "18"

      # 3) Install Dependencies
      - name: Install Dependencies
        run: npm install

      # 4) Build Vite Admin App
      - name: Build Admin App
        run: npm run build

      # 5) Load SSH Private Key
      - name: Add SSH key
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      # 6) Add server to known_hosts
      - name: Add server to known_hosts
        run: ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      # 7) Upload dist → server
      - name: Upload dist folder via rsync
        run: |
          rsync -avz --delete \
            ./dist/ \
            ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:${{ secrets.REMOTE_PATH }}/

      # 8) Restart Nginx (optional)
      - name: Restart Nginx
        run: ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} "sudo systemctl restart nginx"
		
		
		
		🔥 Required GitHub Secrets (Admin Repo)
		
		Admin repo-র Settings → Secrets → Actions এ এইগুলো add করো:
		| Secret Name         | Value                  |
| ------------------- | ---------------------- |
| **SSH_PRIVATE_KEY** | তোমার deploy key       |
| **SSH_HOST**        | DigitalOcean server IP |
| **SSH_USER**        | root                   |
| **SSH_PORT**        | 22                     |
| **REMOTE_PATH**     | `/var/www/myapp/admin` |


📌 PM2_APP_NAME লাগবে না
(Admin React project PM2/Node server না – শুধু build serve হয় Nginx দিয়ে)















