#  Portfolio CI/CD Deployment using Jenkins & AWS EC2

This project demonstrates a **complete production-style CI/CD pipeline** for deploying a my personal Portfolio application using:

- GitHub
- Jenkins
- AWS EC2
- Nginx
- SSH-based Deployment

Whenever code is pushed to GitHub, Jenkins automatically:

1. Clones the repository  
2. Installs dependencies  
3. Builds the project  
4. Deploys the production build (`dist/`)  
5. Reloads Nginx  
6. Makes the portfolio live  

---

#  Architecture Overview

![Architecture Diagram](./img/Architecture%20Dig.png)

---

#  Infrastructure Setup

##   1. Jenkins Server (AWS EC2)

- Ubuntu Server
- Jenkins Installed
- NodeJS Installed
- Connected to GitHub via Webhook
- SSH access to Target Server

##  2. Target Server (AWS EC2)

- Ubuntu Server
- Nginx Installed
- Deployment Directory: `/var/www/portfolio`
- Serves static files from build output

![](./img/Screenshot%202026-02-23%20153012.png)

---

#  Step 1: Install Jenkins on Master Server

Connect to Master:
```
ssh -i key.pem ubuntu@<MASTER_SERVER_IP>
```
Update system:
```
sudo apt update
```
Install Java:
```
sudo apt install openjdk-17-jdk -y
```
Install Jenkins:

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl start jenkins
sudo systemctl enable jenkins
```
![](./img/Screenshot%202026-02-23%20153638.png)


Access Jenkins:

```
http://<JENKINS_SERVER_IP>:8080
```
![](./img/Screenshot%202026-02-23%20153751.png)
---

#  Step 2: Install NodeJS on Jenkins Server

```bash
sudo apt install nodejs npm -y
node -v
npm -v
```

---

#  Step 3: Install Nginx on Target Server

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
![](./img/Screenshot%202026-02-23%20154058.png)

![](./img/Screenshot%202026-02-23%20154047.png)

---

#  Step 4: Configure Nginx

Edit config:

```bash
sudo nano /etc/nginx/sites-available/default
```

Update root:

```
root /var/www/portfolio;
index index.html;
```
![](./img/Screenshot%202026-02-23%20154323.png)

Restart:

```bash
sudo systemctl restart nginx
```

---

#  Step 5: Create Jenkins Pipeline Job

1. Open Jenkins Dashboard
2. Click **New Item**
3. Name: `Portfolio-CICD`
4. Select **Pipeline**
5. Click OK

![](./img/Screenshot%202026-02-23%20160250.png)

---

#  Step 6: Configure GitHub in Jenkins

Enable:

```
GitHub hook trigger for GITScm polling
```

Select:

```
Pipeline script from SCM
```

SCM: Git

Repository URL:

```
https://github.com/aftab0045/Portfolio-CICD-Deplyoment.git
```

Branch:

```
main
```

Script Path:

```
Jenkinsfile
```
![](./img/Screenshot%202026-02-23%20160529.png)


---

#  Step 7: Add SSH Credentials

Go to:

```
Manage Jenkins → Credentials → Global → Add Credentials
```

Kind:

```
SSH Username with private key
```

ID:

```
portfolio-key
```
![](./img/Screenshot%202026-02-23%20161239.png)

---

#  Step 8: Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        SERVER_IP = '<TARGET_SERVER_IP>'
        SSH_CREDENTIAL = 'portfolio-key'
        REMOTE_USER = 'ubuntu'
        REMOTE_PATH = '/var/www/portfolio'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/aftab0045/Portfolio-CICD-Deplyoment.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Project') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy dist Folder') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'rm -rf ${REMOTE_PATH}/*'
                        scp -o StrictHostKeyChecking=no -r dist/* ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Reload Nginx') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            sudo systemctl reload nginx
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Portfolio deployed successfully!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
```

---

#  Step 9: Configure GitHub Webhook

Go to:

```
GitHub → Settings → Webhooks → Add Webhook
```

Payload URL:

```
http://<JENKINS_SERVER_IP>:8080/github-webhook/
```

Content Type:

```
application/json
```

Event:

```
Just the push event
```

![](./img/Screenshot%202026-02-23%20162353.png)
---

#  Step 10: Deploy Application

Push code:

```bash
git add .
git commit -m "Portfolio update"
git push origin main
```

Jenkins will automatically trigger deployment.

![](./img/Screenshot%202026-02-23%20160920.png)

---

#  Step 11: Verify Successful Build

Go to Jenkins:

- Open Pipeline Job
- Open Build History
- Click Latest Build
- Open Console Output

You should see:

```
Portfolio deployed successfully!
Finished: SUCCESS
```

![](./img/Screenshot%202026-02-23%20163902.png)
![](./img/Screenshot%202026-02-23%20163918.png)
---

#  Step 12: Verify GitHub Webhook

Go to:

```
GitHub → Settings → Webhooks → Recent Deliveries
```

You should see:

```
Status Code: 200
Last delivery was successful
```

![](./img/Screenshot%202026-02-23%20162620.png)


---

#  Step 13: Access Live Portfolio

Open browser:

```
http://<TARGET_SERVER_IP>
```
![](./img/Screenshot%202026-02-23%20163842.png)

---

#  Step 14: Automatic Redeployment

Modify any file:

```bash
git add .
git commit -m "Updated UI"
git push origin main
```

Jenkins will:

- Pull latest code
- Build project
- Deploy `dist/`
- Reload Nginx
- Update live website

---

#  Project Structure

```
Portfolio-CICD-Deplyoment/
│
├── src/
├── public/
├── index.html
├── package.json
├── Jenkinsfile
├── vite.config.ts
├── tailwind.config.ts
├── README.md
└── screenshots/
```


#  DevOps Concepts Demonstrated

- CI/CD Automation
- Jenkins Declarative Pipeline
- GitHub Webhook Integration
- SSH-based Remote Deployment
- Static Site Deployment using Nginx
- Production Build Handling (Vite dist folder)
- AWS EC2 Infrastructure
- Real-world Deployment Workflow



---

#  Conclusion

This project demonstrates a complete real-world CI/CD pipeline used in production environments.

By integrating GitHub, Jenkins, AWS EC2, SSH deployment, and Nginx, the entire deployment process is fully automated.

Every push to GitHub triggers an automatic build and deployment, eliminating manual steps, reducing errors, and improving release speed.

This setup follows industry-standard DevOps best practices and provides a strong foundation for scaling toward containerized and cloud-native deployments.

---



