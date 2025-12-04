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

![Image](https://i.sstatic.net/HcgJH.png?utm_source=chatgpt.com)

![Image](https://i.sstatic.net/2Afmi.png?utm_source=chatgpt.com)

---

## ✅ Git Clone — Local Workspace

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20220326004125/screencapturegithubarpit456jainfirstrepogfg20220326003141-300x189.png?utm_source=chatgpt.com)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20200707150457/gitcommit.png?utm_source=chatgpt.com)

---

## ✅ NGINX Status — Running on EC2

![Image](https://static0.xdaimages.com/wordpress/wp-content/uploads/wm/2023/12/nginx-on-ubuntu.jpg?utm_source=chatgpt.com)

![Image](https://www.devopshint.com/wp-content/uploads/2023/05/nginx-status.png?utm_source=chatgpt.com)

---

## ✅ AWS EC2 Infrastructure — Terraform Provisioned

![Image](https://i.sstatic.net/S5KNX.png?utm_source=chatgpt.com)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AJu9PmH76d9W0w_J8.png?utm_source=chatgpt.com)

---

## ✅ Final Deployed Website

![Image](https://equitydatascience.com/wp-content/uploads/Nexus-Poduction.gif?utm_source=chatgpt.com)

![Image](https://www.sonatype.com/hs-fs/hubfs/1-2025_Website-Assets/product_ui/Repo-6-UI-Product.png?height=2535\&name=Repo-6-UI-Product.png\&width=3685\&utm_source=chatgpt.com)

---

## ✅ GitHub Webhook Integration

![Image](https://www.robinwieruch.de/static/4c1dbc3df7f277d85773090cc01d558a/72e01/github-webhook.jpg?utm_source=chatgpt.com)

![Image](https://i.ytimg.com/vi/b_DVXgiByec/maxresdefault.jpg?utm_source=chatgpt.com)

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
<PUT YOUR MAIN.TF HERE>
```

---

# 🧩 variables.tf

```hcl
<PUT YOUR VARIABLES.TF HERE>
```

---

# 🧩 user_data.sh

```bash
<PUT YOUR USER_DATA.SH HERE>
```

---

# 🔄 Jenkinsfile (CI/CD Pipeline)

```groovy
<PUT YOUR JENKINSFILE HERE>
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


