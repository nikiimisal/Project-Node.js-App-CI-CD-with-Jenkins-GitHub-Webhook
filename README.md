# 🚀 Node.js App — CI/CD with Jenkins & GitHub Webhook

>This project demonstrates a **complete CI/CD pipeline** for a Node.js application using **Jenkins and GitHub Webhooks**, fully automated from **code push → build → test → deploy**.

---
<p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/oooooooooooooooooooooooooooooooooooo.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>

## 🧩 Overview

---

* Automates the deployment of a Node.js application using Jenkins CI/CD.
* Integrates GitHub Webhooks to automatically trigger builds when new code is pushed.
* Pulls the latest source code from the GitHub repository.
* Installs all required Node.js dependencies automatically.
* Builds and tests the application to ensure code quality.
* Deploys the updated application to the target server without manual steps.
* Ensures continuous integration and continuous delivery (CI/CD) for faster, reliable updates.
* Provides a fully automated workflow — from code commit to live deployment.

---

## 🧰 Tools & Technologies

* **Jenkins** – CI/CD automation
* **Node.js & npm** – App runtime & package management
* **GitHub Webhooks** – Build trigger on code push
* **PM2** – Process manager for Node.js
* **EC2 (Ubuntu)** – Target deployment environment

---
## 🔌Plugins (at minimum)

* Git Plugin → For repo clone/pull mechanism.

* GitHub Plugin → pull mechanism(git clone) // Auto-trigger builds via Webhook (on push).

* Pipeline Plugin → Defines CI/CD pipeline stages.

* SSH Agent Plugin → Enables remote server access (deploy).

* Pipeline Stage View Plugin → Shows visual stage-wise pipeline status.
  
---
## 📋 Prerequisites

* GitHub repository for your app.

* Jenkins server(Ubuntu) reachable by your Git host and target servers

* Target EC2 server(node) with SSH access

* Node.js application source code hosted on a Git repository (e.g., GitHub).

## High-level CI/CD Flow

1.  Developer pushes code to main (or feature branch).

2.  Github webhook triggers Jenkins.

3.  Jenkins pulls code, runs npm ci, runs tests.

4.  If tests pass, Jenkins builds the app (if there's a build step).

5.  Jenkins deploys the new version to the target:

6.  SSH into server, install dependencies, restart process (pm2).

7. Post-deploy smoke tests and notifications.

---

## 🪜 Step-by-Step Implementation

### **Step 1: Launch EC2s**

* Create two EC2 instances in same VPC (Default).
* Security Group:

  * Jenkins Server → Port `8080`
  * Target Server → Ports `22`, `3000`
 
  * In terminal run this commands
 
        sudo hostnamectl hostname node
        sudo apt update
        sudo apt install nodejs npm -y
        sudo npm install -g pm2


  <p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20142408.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>


---

### **Step 2: Create GitHub Repository**

* Repo Name: `node-js-app-CICD`
  https://github.com/nikiimisal/node-js-app-CICD
* Branch: `main`

#### **Add Webhook** (Optional)

* Payload URL → `http://<JENKINS_PUBLIC_IP>:8080/github-webhook/`

 <p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20142529.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>

 <p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20142750.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>
---

### **Step 3: Add SSH Credentials in Jenkins**

  >Create New Credentails
1.  Navigate → Manage Jenkins → Credentials → Global
2.  Add → **SSH Username with private key**

   * ID: `node-app-key`
   * Username: `ubuntu`
   * Paste your private key
 <p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20143758.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>
---

### **Step 4: Create Jenkins Pipeline Job**

* Item Name: `node-app-deploy-cicd`
* Item-Type: **Pipeline**
* Enable Trigger: *GitHub hook trigger for GITScm polling*
* Definition: *Pipeline script from SCM*
* Repository URL: `https://github.com/nikiimisal/node-js-app-CICD.git`
* Branch: `main`
* Script Path: `jenkinsfile`

 <p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20145323.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>
---

### **Step 5: Jenkinsfile Configuration**

```groovy
pipeline {
    agent any

    environment {
        SERVER_IP      = '16.171.161.138'
        SSH_CREDENTIAL = 'node-app-key'
        REPO_URL       = 'https://github.com/nikiimisal/node-js-app-CICD.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/home/ubuntu/node-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Upload Files to EC2') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Install Dependencies & Start App') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            cd ${REMOTE_PATH} &&
                            npm install &&
                            pm2 start app.js --name node-app || pm2 restart node-app
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Application deployed successfully!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}
```
>NOTE: Replace `SERVER_IP`, `SSH_CREDENTIAL`, `REPO_URL`, and `REMOTE_PATH` with your actual values.
---

### **Step 6: Push Code to GitHub**

```bash
git init
git add .
git commit -m ""
git push -u origin main
```
>Now we pushed code to GitHub, a webhook instantly notifies the Jenkins server. Jenkins then automatically pulls the latest code, installs dependencies, runs tests, builds the application, and deploys it to the target server
---

### **Step 7: Verify Deployment(Build)**

Open jenkin:

-- click build know
>If the pipeline runs successfully, that’s great 👍 —<br>
but if it fails, check the Console Output in Jenkins to see the errors and fix them accordingly.


 <p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20150047.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>

---
### **Step 8: Browse Application on browser**

Open in browser:

open browser and enter `http://<Node-Server-Public-Ip>:3000`

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub-Webhook/blob/main/img/Screenshot%202025-11-06%20150140.png?raw=true" width="700" alt="Initialize Repository Screenshot">
</p>

---

## 🧠 Troubleshooting

| Issue                   | Solution                                        |
| ----------------------- | ----------------------------------------------- |
| Webhook not triggering  | Ensure Jenkins Public IP accessible from GitHub |
| Permission denied (SSH) | Recheck private key and username (`ubuntu`)     |
| Jenkins build fails     | Check console log in Jenkins dashboard          |
| PM2 not restarting      | Run `pm2 list` on target EC2 to verify          |

---

## 🎯 Conclusion

By integrating Jenkins with GitHub Webhooks, this setup achieves **automated, repeatable deployments**. Every code push triggers Jenkins → builds → tests → deploys → restarts Node.js app on EC2 — completely hands-free.

✅ **Key Benefits:**

* Continuous Integration & Delivery (CI/CD)
* Automated Deployment via SSH
* Improved Speed & Reliability
* Zero manual intervention

---

**Author:** nikiimisal😜<br>
**License:** MIT

---



