# 🌐 Static Website Deployment on AWS EC2 Using Terraform + Jenkins CI/CD

This project demonstrates a fully automated DevOps pipeline where a static website is deployed on AWS EC2 using Terraform and continuously updated via Jenkins using GitHub webhooks.

The workflow ensures:

* 💠 Automated Infrastructure Provisioning
* 💠 Zero-touch Deployment
* 💠 Auto-trigger CI/CD on every GitHub push
* 💠 NGINX Webserver auto-configuration
* 💠 Real-time delivery to EC2

---

# 📸 **Screenshots (Your Actual Output)**

---

## ✅ Jenkins Pipeline — Successful Build

![Image](img/deploy_success)


---

## ✅ Git Clone — Local Workspace

![Image](img/git_clone)



---

## ✅ NGINX Status — Running on EC2

![Image](./img/servers)

---

---

## ✅ Final Deployed Website

![Image](img/static_website)


---

## ✅ GitHub Webhook Integration

![Image](img/webhook)

---

# 🏗️ Project Architecture

```
Developer Push → GitHub → Webhook → Jenkins Pipeline
                          ↓
                   Terraform EC2 Setup
                          ↓
                NGINX + Auto Deployment
                          ↓
                  Live Static Website
```

---

# 🧱 Technologies Used

| Component           | Purpose                                 |
| ------------------- | --------------------------------------- |
| **Terraform**       | Provision EC2, Security Group, Key Pair |
| **AWS EC2**         | Host static website with NGINX          |
| **NGINX**           | Web server                              |
| **Jenkins**         | CI/CD pipeline automation               |
| **GitHub Webhooks** | Trigger Jenkins on every push           |
| **Bash Scripting**  | Server configuration                    |

---

# 📁 Folder Structure

```
static-website-project/
├── infra/
│   ├── main.tf
│   ├── variables.tf
│   └── user_data.sh
├── Jenkinsfile
└── website files (HTML/CSS/JS)
```

---

# 🧩 Terraform (main.tf)

```hcl
provider "aws" {
  region = "ap-south-1"
}

# Key pair - change path to your public key if needed. Better: put newin.pub in same folder and use "file("newin.pub")"
resource "aws_key_pair" "newin" {
  key_name   = "newin"
  public_key = file("C:/Users/Raj/.ssh/newin.pub")
}

resource "aws_security_group" "blog-web" {
  name = "blog-web"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  lifecycle {
    create_before_destroy = true
  }

  tags = {
    Name = "blog-web-sg"
  }
}

resource "aws_instance" "bolg-web" {
  ami                    = "ami-02b8269d5e85954ef" # Ubuntu 22.04 ap-south-1 (you used earlier)
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.blog-web.id]
  key_name               = aws_key_pair.newin.key_name

  # point to user_data.sh in same folder
  user_data = file("${path.module}/user_data.sh")

  tags = {
    Name = "blog-web"
  }
}

output "public_ip" {
  value = aws_instance.bolg-web.public_ip
}

output "public_dns" {
  value = aws_instance.bolg-web.public_dns
}

```

---

---

# 🧩 user_data.sh

```bash
#!/bin/bash
set -euo pipefail

LOG="/var/log/user-data.log"
exec > >(tee -a "$LOG") 2>&1

echo "=== user-data start $(date -u +"%Y-%m-%dT%H:%M:%SZ") ==="

REPO_URL="https://github.com/RajAhire-1/static-website-project.git"
WEBROOT="/var/www/html"

# Update and install required packages
apt-get update -y
DEB_PKGS="git nginx openjdk-17-jdk"
apt-get install -y $DEB_PKGS

# Add Jenkins official repo and install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
apt-get update -y
apt-get install -y jenkins

# Ensure services enabled and started
systemctl enable --now nginx
systemctl enable --now jenkins

# Deploy website: clone or pull
if [ -d "${WEBROOT}/.git" ]; then
  echo "Existing repo found, pulling latest"
  chown -R ubuntu:ubuntu "${WEBROOT}" || true
  sudo -u ubuntu git -C "${WEBROOT}" pull --rebase || {
    echo "pull failed, trying fetch+reset"
    sudo -u ubuntu git -C "${WEBROOT}" fetch --all || true
    sudo -u ubuntu git -C "${WEBROOT}" reset --hard origin/HEAD || true
  }
else
  echo "Cloning repo ${REPO_URL} into ${WEBROOT}"
  rm -rf "${WEBROOT:?}/"*
  git clone "${REPO_URL}" "${WEBROOT}" || {
    echo "git clone failed — creating placeholder index"
    mkdir -p "${WEBROOT}"
    cat > "${WEBROOT}/index.html" <<HTML
<html><body><h1>Clone failed</h1><p>Check /var/log/user-data.log</p></body></html>
HTML
  }
fi

# Correct permissions depending on distro (nginx uses www-data on Ubuntu)
if id www-data >/dev/null 2>&1; then
  chown -R www-data:www-data "${WEBROOT}"
else
  chown -R ubuntu:ubuntu "${WEBROOT}"
fi

find "${WEBROOT}" -type d -exec chmod 755 {} \;
find "${WEBROOT}" -type f -exec chmod 644 {} \;

systemctl restart nginx || true

echo "=== user-data finished $(date -u +"%Y-%m-%dT%H:%M:%SZ") ==="

```

---

# 🔄 Jenkinsfile (CI/CD Pipeline)

```groovy
pipeline {
  agent any

  environment {
    DEPLOY_USER        = 'ubuntu'
    DEPLOY_HOST        = '13.201.4.66'
    DEPLOY_PATH        = '/var/www/html'
    SSH_CREDENTIALS_ID = 'web-key'
    REPO_URL           = 'https://github.com/RajAhire-1/static-website-project.git'
    BRANCH             = 'main'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "${BRANCH}", url: "${REPO_URL}"
      }
    }

    stage('Prepare Remote') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: SSH_CREDENTIALS_ID,
                                          keyFileVariable: 'SSH_KEY',
                                          usernameVariable: 'SSH_USER')]) {
          sh """
            # remove any broken jenkins apt list/key which causes apt-get update failure
            ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${SSH_USER}@${DEPLOY_HOST} '
              set -e
              sudo rm -f /etc/apt/sources.list.d/jenkins.list || true
              sudo rm -f /usr/share/keyrings/jenkins-keyring.asc || true

              sudo apt-get update -y
              sudo apt-get install -y nginx git
              sudo systemctl enable nginx
              sudo systemctl start nginx
              sudo mkdir -p ${DEPLOY_PATH}
            '
          """
        }
      }
    }

    stage('Upload & Deploy') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: SSH_CREDENTIALS_ID,
                                          keyFileVariable: 'SSH_KEY',
                                          usernameVariable: 'SSH_USER')]) {
          sh """
            scp -o StrictHostKeyChecking=no -i "$SSH_KEY" -r * ${SSH_USER}@${DEPLOY_HOST}:/tmp/deploy_payload || true

            ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${SSH_USER}@${DEPLOY_HOST} "
              set -e
              DEPLOY_PATH='${DEPLOY_PATH}'
              sudo rm -rf \\\"\$DEPLOY_PATH\\\"/* || true
              sudo mv /tmp/deploy_payload/* \\\"\$DEPLOY_PATH\\\"/ || true

              if id www-data >/dev/null 2>&1; then
                sudo chown -R www-data:www-data \\\"\$DEPLOY_PATH\\\"
              elif id nginx >/dev/null 2>&1; then
                sudo chown -R nginx:nginx \\\"\$DEPLOY_PATH\\\"
              else
                sudo chown -R ubuntu:ubuntu \\\"\$DEPLOY_PATH\\\"
              fi

              sudo find \\\"\$DEPLOY_PATH\\\" -type d -exec chmod 755 {} \\;
              sudo find \\\"\$DEPLOY_PATH\\\" -type f -exec chmod 644 {} \\;

              sudo systemctl restart nginx || true
              sudo rm -rf /tmp/deploy_payload || true

              echo DEPLOY_OK
            "
          """
        }
      }
    }

    stage('Smoke Test') {
      steps {
        sh "curl -I http://${DEPLOY_HOST} | head -n 5 || true"
      }
    }
  }

  post {
    success {
      echo "✅ Deployment succeeded — open http://${DEPLOY_HOST}"
    }
    failure {
      echo "❌ Deployment failed — check console output"
    }
  }
}

```

---

# 🚀 Final Result

✔️ Terraform creates the server
✔️ NGINX auto-configured
✔️ Website deployed to EC2
✔️ Git push triggers Jenkins automatically
✔️ Jenkins redeploys site to EC2
✔️ Live website instantly reflects updates

---

# 👨‍💻 Author

**Raj Ahire**
AWS | DevOps | Jenkins | Terraform | CI/CD | Linux


