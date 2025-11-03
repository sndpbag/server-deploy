গাইড: GitHub থেকে cPanel-এ অটোমেটিক Laravel ডিপ্লয় (SSH বা টার্মিনাল ছাড়া)যেহেতু আপনার cPanel-এ SSH বা টার্মিনাল অ্যাক্সেস নেই, তাই আমরা ফাইল আপলোড করার জন্য FTP এবং প্রসেসটি অটোমেট করার জন্য GitHub Actions ব্যবহার করবো।পূর্ব-প্রস্তুতিআপনাকে দুটি জিনিস প্রস্তুত রাখতে হবে:১.  cPanel-এ একটি FTP অ্যাকাউন্ট: আপনার cPanel-এ লগইন করে "FTP Accounts" সেকশন থেকে একটি নতুন FTP অ্যাকাউন্ট তৈরি করুন।* গুরুত্বপূর্ণ: FTP অ্যাকাউন্ট তৈরি করার সময় "Directory" পাথটি সঠিকভাবে দিন। আপনি যদি চান আপনার Laravel প্রজেক্টটি public_html ফোল্ডারের ভেতরে থাকুক, তবে পাথটি public_html দিন।* অ্যাকাউন্ট তৈরি করার পর নিচের তিনটি জিনিস নোট করুন:1.  FTP Host/Server: (সাধারণত ftp.yourdomain.com বা আপনার সার্ভারের IP)2.  FTP Username: (e.g., laravel_user@yourdomain.com)3.  FTP Password: (আপনি যে পাসওয়ার্ডটি সেট করেছেন)২.  প্রোডাকশন .env ফাইল: আপনার লোকাল কম্পিউটারে আপনার প্রোডাকশন সার্ভারের জন্য (cPanel) একটি .env ফাইল রেডি করুন। এখানে ডাটাবেসের সঠিক নাম, ইউজারনেম, পাসওয়ার্ড এবং APP_URL ঠিকমতো সেট করুন। APP_DEBUG অবশ্যই false করে দিন।ধাপে ধাপে নির্দেশিকাধাপ ১: cPanel-এর Document Root সেট করা (প্রস্তাবিত)Laravel-এর সিকিউরিটির জন্য public ফোল্ডারটিই শুধু বাইরে থেকে অ্যাক্সেস করা উচিত।১.  cPanel-এ "Domains" বা "Subdomains" সেকশনে যান।২.  আপনার যে ডোমেইনে অ্যাপটি চালাবেন, সেটির "Document Root" পরিবর্তন করুন।৩.  যদি আপনার প্রজেক্টটি public_html ফোল্ডারে রাখেন, তবে Document Root সেট করুন: public_html/public। এটিই Laravel-এর স্ট্যান্ডার্ড পদ্ধতি।ধাপ ২: GitHub Repository-তে "Secrets" যোগ করাআপনার FTP এবং .env ফাইলের সংবেদনশীল তথ্য আমরা সরাসরি কোডে লিখবো না। এগুলো GitHub Secrets-এ রাখবো।১.  আপনার Laravel প্রজেক্টের GitHub রিপোজিটরিতে যান।২.  Settings > Secrets and variables > Actions ট্যাবে যান।৩.  "New repository secret" বাটনে ক্লিক করে নিচের ৪টি সিক্রেট তৈরি করুন:* `FTP_SERVER`:
    * **Value:** আপনার FTP Host/Server (e.g., `ftp.yourdomain.com`)
* `FTP_USERNAME`:
    * **Value:** আপনার FTP ইউজারনেম।
* `FTP_PASSWORD`:
    * **Value:** আপনার FTP পাসওয়ার্ড।
* `ENV_FILE`:
    * **Value:** আপনার প্রোডাকশন `.env` ফাইলের সম্পূর্ণ কন্টেন্ট (যা আগে প্রস্তুত করেছেন) কপি করে এখানে পেস্ট করুন।
ধাপ ৩: GitHub Action Workflow ফাইল তৈরি করাএখন আমরা GitHub-কে বলবো যে কোড পুশ হলে কী কী কাজ করতে হবে।১.  আপনার লোকাল Laravel প্রজেক্টের রুট ফোল্ডারে (যেখানে app, config ফোল্ডারগুলো আছে) যান।২.  .github নামে একটি নতুন ফোল্ডার তৈরি করুন।৩.  .github ফোল্ডারের ভেতরে workflows নামে আরেকটি ফোল্ডার তৈরি করুন।৪.  workflows ফোল্ডারের ভেতরে deploy.yml নামে একটি ফাইল তৈরি করুন।অর্থাৎ, আপনার প্রজেক্টের পাথটি হবে: .github/workflows/deploy.ymlধাপ ৪: deploy.yml ফাইলে কোড যোগ করাআপনার তৈরি করা deploy.yml ফাইলে নিচের কোডটি সম্পূর্ণ কপি করে পেস্ট করুন। এই ফাইলের একটি কপি আমি (deploy.yml) আলাদাভাবেও দিয়েছি।name: Deploy Laravel to cPanel via FTP

on:
  push:
    branches:
      - main  # অথবা আপনি যে ব্রাঞ্চে পুশ করলে ডিপ্লয় করতে চান (e.g., master)

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1' # আপনার Laravel প্রজেক্টের রিকোয়ারমেন্ট অনুযায়ী পিএইচপি ভার্সন দিন

    - name: Create .env file
      run: echo "${{ secrets.ENV_FILE }}" > .env
      
    - name: Install Composer Dependencies
      run: composer install --no-dev --optimize-autoloader

    - name: Generate Laravel Key
      run: php artisan key:generate
      
    - name: Cache configuration
      run: |
        php artisan config:cache
        php artisan route:cache
        php artisan view:cache

    - name: Deploy files via FTP
      uses: SamKirkland/FTP-Deploy-Action@4.3.3
      with:
        server: ${{ secrets.FTP_SERVER }}
        username: ${{ secrets.FTP_USERNAME }}
        password: ${{ secrets.FTP_PASSWORD }}
        server-dir: /public_html/   # আপনার cPanel FTP অ্যাকাউন্টের রুট ডিরেক্টরি পাথ দিন
        exclude: |
          **/.git*
          **/.git*/**
          **/node_modules/**
          .github/**
          storage/logs/**
          storage/framework/sessions/**
          storage/framework/views/**
          storage/framework/cache/data/**



          Your-Laravel-Project/
│
├── .github/
│   └── workflows/
│       └── deploy.yml   <--- (ফাইলটি এখানে থাকবে)
│
├── app/
├── config/
├── public/
├── routes/
├── composer.json
└── ... (আপনার প্রজেক্টের অন্যান্য সব ফাইল)




নোট: deploy.yml ফাইলের server-dir: অংশটি সাবধানে পরিবর্তন করুন। এটি আপনার FTP ইউজার যে ফোল্ডারে অ্যাক্সেস পায়, তার সাপেক্ষে পাথ। সাধারণত এটি / বা /public_html/ হয়।ধাপ ৫: কোড Push করা এবং ডিপ্লয়মেন্ট দেখা১.  আপনার প্রজেক্টে নতুন তৈরি করা .github/workflows/deploy.yml ফাইলটি সহ সমস্ত পরিবর্তন git add ., git commit -m "Add deploy workflow" এবং git push করুন।২.  GitHub-এ আপনার রিপোজিটরিতে যান এবং Actions ট্যাবে ক্লিক করুন।৩.  আপনি দেখতে পাবেন "Deploy Laravel to cPanel via FTP" নামে একটি নতুন ওয়ার্কফ্লো চালু হয়েছে। এটি হলুদ দেখাবে (অর্থাৎ চলছে)।৪.  কিছুক্ষণ পর এটি সবুজ (সফল) বা লাল (ব্যর্থ) হবে। সবুজ হলে বুঝবেন আপনার কোড cPanel-এ ডিপ্লয় হয়ে গেছে।ধাপ ৬: cPanel-এ ফোল্ডার পারমিশন (শুধু প্রথমবার)প্রথমবার সফলভাবে ডিপ্লয় হওয়ার পর, আপনাকে cPanel-এর "File Manager"-এ গিয়ে দুটি ফোল্ডারের পারমিশন ঠিক করতে হবে যাতে Laravel ফাইল লিখতে পারে।১.  storage ফোল্ডারের উপর রাইট-ক্লিক করে Change Permissions-এ যান এবং 775 সেট করুন।২.  bootstrap/cache ফোল্ডারের উপর রাইট-ক্লিক করে Change Permissions-এ যান এবং 775 সেট করুন।ব্যাস! আপনার কাজ শেষ। এরপর থেকে যখনি আপনি main (বা নির্দিষ্ট) ব্রাঞ্চে কোড পুশ করবেন, এই প্রসেসটি অটোমেটিক চলবে এবং আপনার সাইট আপ-টু-ডেট হয়ে যাবে।