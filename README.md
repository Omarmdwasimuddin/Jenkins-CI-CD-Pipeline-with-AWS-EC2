# Jenkins CI/CD Pipeline with AWS EC2

> **Next.js প্রজেক্টকে AWS EC2-এ Jenkins দিয়ে Automatically Deploy করার সম্পূর্ণ গাইড**

📌 **EC2-তে Jenkins Install করার প্রক্রিয়া দেখুন:**
[Jenkins Install on Ubuntu →](https://github.com/Omarmdwasimuddin/Jenkins-Install-on-Ubuntu)

---

## ধাপ ১: Next.js প্রজেক্ট তৈরি ও GitHub-এ Push করো

```bash
# Next.js প্রজেক্ট তৈরি করো
npx create-next-app@latest my-nextjs-app
cd my-nextjs-app

# GitHub-এ নতুন repository তৈরি করে push করো
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

---

## ধাপ ২: Jenkins Dashboard খোলো

Browser-এ নিচের URL দিয়ে Jenkins খোলো:

```
http://18.143.149.173:8080/
```

---

## ধাপ ৩: Node.js ও npm ইনস্টল করো (EC2 Server-এ)

EC2 Server-এর Terminal-এ কমান্ড দাও:

```bash
sudo apt install nodejs npm -y
```

> ✅ এটি না থাকলে পরে `npm install` ও `npm run build` কাজ করবে না

---

## ধাপ ৪: Jenkins-কে Sudo Permission দাও

Terminal-এ কমান্ড দাও:

```bash
sudo visudo
```

ফাইলের শেষে নিচের লাইনটি যোগ করো:

```
jenkins ALL=(ALL) NOPASSWD: ALL
```

**Save ও Exit করো:**
- `Ctrl + S` → Save
- `Ctrl + X` → Exit

> ✅ এটি না করলে Jenkins `sudo` কমান্ড রান করতে পারবে না

---

## ধাপ ৫: প্রজেক্টে Jenkinsfile তৈরি করো

প্রজেক্টের **root directory**-তে `Jenkinsfile` নামের ফাইল তৈরি করো এবং নিচের কোড পেস্ট করো:

```groovy
node {
    def appDir = 'var/www/nextjs-app'

    stage('Clean Workspace') {
        echo 'Cleaning workspace....'
        deleteDir()
    }

    stage('Clone Repo') {
        echo 'Cloning repository...'
        git(
            branch: 'main',
            url: 'https://github.com/Omarmdwasimuddin/Jenkins-CI-CD-Pipeline-with-AWS-EC2'
        )
    }

    stage('Deploy to EC2') {
        echo 'Deploying to EC2...'
        sh """
            sudo mkdir -p ${appDir}
            sudo chown -R jenkins:jenkins ${appDir}
            rsync -av --delete --exclude='.git' --exclude='node_modules' ./ ${appDir}
            cd ${appDir}
            sudo npm install
            sudo npm run build
            sudo fuser -k 3000/tcp || true
            sudo npm start
        """
    }
}
```

**Jenkinsfile টি GitHub-এ push করো:**

```bash
git add Jenkinsfile
git commit -m "Add Jenkinsfile"
git push origin main
```

---

## ধাপ ৬: GitHub Webhook সেটআপ করো

GitHub-এ প্রজেক্ট Repository-তে যাও এবং নিচের ধাপ অনুসরণ করো:

**Settings → Webhooks → Add webhook**

| ফিল্ড | মান |
|-------|-----|
| Payload URL | `http://18.143.149.173:8080/github-webhook/` |
| Content type | `application/json` |
| Which events? | `Just the push event` *(default)* |

**Add webhook** বাটনে ক্লিক করো।

> ⚠️ Payload URL-এর শেষে `/github-webhook/` — স্ল্যাশ মিস করো না

---

## ধাপ ৭: Jenkins-এ Pipeline Job তৈরি করো

### ৭.১ — New Item তৈরি

1. Jenkins Dashboard → **New Item** ক্লিক করো
2. নিচের তথ্য দাও:

   | ফিল্ড | মান |
   |-------|-----|
   | Item name | `nextjs-aws` |
   | Type | `Pipeline` |

3. **OK** ক্লিক করো

### ৭.২ — Trigger সেট করো

**Build Triggers** সেকশনে গিয়ে চেক দাও:

```
✅ GitHub hook trigger for GITScm polling
```

### ৭.৩ — Pipeline Script সেট করো

**Pipeline** সেকশনে নিচের মতো সেট করো:

| ফিল্ড | মান |
|-------|-----|
| Definition | `Pipeline script from SCM` |
| SCM | `Git` |
| Repository URL | তোমার GitHub Repo URL |
| Branch Specifier | `*/main` |

4. **Save** বাটনে ক্লিক করো

---

## ধাপ ৮: Pipeline Test করো

প্রজেক্টে যেকোনো পরিবর্তন করে GitHub-এ push করো:

```bash
# যেকোনো ফাইলে পরিবর্তন করো, তারপর:
git add .
git commit -m "Test CI/CD trigger"
git push origin main
```

> ✅ Push করার সাথে সাথে Jenkins স্বয়ংক্রিয়ভাবে **Build শুরু** করবে

---

## ✅ CI/CD Flow একনজরে

```
Code Push (GitHub)
       ↓
GitHub Webhook Trigger
       ↓
Jenkins Build শুরু
       ↓
Stage 1: Workspace Clean
       ↓
Stage 2: Repo Clone
       ↓
Stage 3: EC2-তে Deploy
  ├── mkdir + chown
  ├── rsync (ফাইল কপি)
  ├── npm install
  ├── npm run build
  └── npm start (Port 3000)
       ↓
🚀 App Live: http://<EC2-IP>:3000
```

---

## 📋 Quick Reference

| বিষয় | তথ্য |
|-------|------|
| Jenkins URL | `http://18.143.149.173:8080` |
| App Deploy Path | `/var/www/nextjs-app` |
| App Port | `3000` |
| Webhook URL | `http://18.143.149.173:8080/github-webhook/` |
| Pipeline Branch | `main` |
