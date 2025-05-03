# 🚀 Node.js Deployment on Amazon EC2 with GitHub Actions

This guide outlines the step-by-step process for deploying a Node.js application to an Amazon EC2 instance using **Nginx**, **PM2**, and **GitHub Actions** for continuous deployment.

---

## 📦 Requirements

- Node.js project hosted on GitHub
- Amazon EC2 instance (Ubuntu)
- SSH access configured
- GitHub Secrets for deployment

---

## 🖥️ EC2 Server Setup Instructions

### Step 1: Update Package Repositories

```bash
sudo apt update
```

### Step 2: Install Node.js

```bash
sudo apt-get install -y nodejs
```

### Step 3: Install Nginx (Reverse Proxy)

```bash
sudo apt-get install -y nginx
```

### Step 4: Install PM2 to Manage Node.js Processes

```bash
sudo npm install -g pm2
```

---

## 🌐 Configure Nginx as a Reverse Proxy

1. Open the default Nginx configuration:

```bash
cd /etc/nginx/sites-available
sudo nano default
```

2. Add the following inside the `server` block:

```nginx
location /api {
    rewrite ^/api/(.*)$ /api/$1 break;
    proxy_pass http://localhost:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

3. Restart Nginx:

```bash
sudo systemctl restart nginx
```

---

## 🚀 Start Your Node.js App with PM2

Navigate to your application directory and run:

```bash
cd /path/to/your/app
pm2 start server.js --name=Backend
```

Optional: Restart the app anytime with:

```bash
pm2 restart Backend
```

---

## 🔁 Setup GitHub Actions for Continuous Deployment

Create a workflow file at `.github/workflows/deploy.yml`:

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Copy files to EC2
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          source: "."
          target: "/home/${{ secrets.EC2_USER }}/app"

      - name: SSH and deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            cd /home/${{ secrets.EC2_USER }}/app
            npm install
            pm2 restart Backend || pm2 start server.js --name=Backend
```

---

## 🔐 Required GitHub Secrets

| Key         | Description                         |
|-------------|-------------------------------------|
| `EC2_HOST`  | Your EC2 Public IP or DNS           |
| `EC2_USER`  | Default is usually `ubuntu`         |
| `EC2_KEY`   | Your SSH private key (as plain text)|

---

## 📁 Example Project Structure

```
.
├── server.js
├── package.json
├── .github/
│   └── workflows/
│       └── deploy.yml
└── README.md
```

---

## ✅ Final Tips

- Ensure your app listens on `localhost:8000`
- Use `pm2 save` and `pm2 startup` for reboot persistence
- Use HTTPS via Certbot for production security

---

## 🧑‍💻 Author

Deployed with ❤️ using EC2, PM2, Nginx, and GitHub Actions.

---

Happy Coding! 🚀
