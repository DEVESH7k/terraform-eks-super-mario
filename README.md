## 🚀 Deploy Super Mario Game on AWS EKS using Terraform

👋 Introduction

Welcome to the world of clusters, containers, and cloud automation! In this guide, we’ll take a nostalgic detour into childhood by deploying the legendary **Super Mario game** on an **Amazon EKS (Elastic Kubernetes Service)** cluster.

But this is more than fun and games — it's a practical deep dive into modern DevOps practices. We’ll begin by testing the game locally with Docker, then move to cloud provisioning using Terraform, and finally deploy Super Mario on EKS.

Whether you're a DevOps enthusiast or a beginner, you’ll gain hands-on exposure to:

* Docker containerization
* Kubernetes services and deployments
* Infrastructure as Code (IaC) with Terraform

**Let’s-a go!** 🍄🐢

---

🔧 Pre-Requisites

Ensure your system is ready to roll:

✅ **Requirements:**

* Docker installed and running
* Basic understanding of Docker and Kubernetes

---

🧪 Testing the Game Locally with Docker

**Step 1:** Start the Docker container

```bash
docker run -d -p 8080:80 deveshkhatik007/mario:latest
```

**Step 2:** Access the Game
Go to: [http://localhost:8080](http://localhost:8080)

---

🛠️ Installing Required Tools

📦 Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client --output=yaml
```

☁️ Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

**Configure Credentials:**

1. Go to IAM → Create User → Name: `mario`
2. Attach `AdministratorAccess` policy
3. Create access key

```bash
aws configure
```

🧰 Install Terraform

```bash
sudo apt-get update && sudo apt-get install -y wget gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

sudo apt update && sudo apt install terraform -y
which terraform
```

---

📁 Setup Project Structure

```bash
mkdir mario-game && cd mario-game
```

✍️ deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mario-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mario
  template:
    metadata:
      labels:
        app: mario
    spec:
      containers:
      - name: mario-container
        image: deveshkhatik007/mario:latest
        ports:
        - containerPort: 80
```

✍️ service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mario-service
spec:
  type: LoadBalancer
  selector:
    app: mario
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

📂 Terraform Configuration

```bash
mkdir terr-config && cd terr-config
```

🧾 provider.tf

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

🪣 backend.tf

```hcl
terraform {
  backend "s3" {
    bucket = "devesh-mario-bucket"
    key    = "EKS/terraform.tfstate"
    region = "ap-south-1"
  }
}
```

Create the bucket:

```bash
aws s3 mb s3://devesh-mario-bucket
```

🧩 main.tf

*Full EKS, IAM roles, VPC, subnets, and node group code provided in your input. (Omitted here for brevity)*

---

🚀 Deployment Steps

1️⃣ Terraform Init, Plan, Apply

```bash
cd terr-config
terraform init
terraform plan
terraform apply --auto-approve
```

2️⃣ Update kubeconfig

```bash
aws eks update-kubeconfig --name EKS_CLOUD --region ap-south-1
```

3️⃣ Deploy App to EKS

```bash
cd ../
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

4️⃣ Get LoadBalancer URL

```bash
kubectl describe service mario-service
```

🎉 **Congratulations!** Your Super Mario game is now live on your AWS EKS cluster!

---

🧹 Clean-Up Resources

Avoid extra AWS charges:

```bash
cd terr-config
terraform destroy --auto-approve
```

---

🎯 Conclusion

You've successfully:

* Set up AWS IAM
* Tested Dockerized Mario locally
* Installed CLI tools
* Provisioned EKS via Terraform
* Deployed and accessed the game using Kubernetes
* Cleaned up AWS infrastructure

Whether you're starting in DevOps or enhancing your cloud-native skills, this project gives you real-world practice — with a nostalgic twist. 🍄🎮

📲 **Stay Connected:**
[Devesh Khatik | LinkedIn](https://www.linkedin.com/in/deveshkhatik)

**#docker #kubernetes #terraform #aws #eks #devops**
