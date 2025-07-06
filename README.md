## 🚀 Deploy Super Mario Game on AWS EKS using Terraform
![image](https://github.com/user-attachments/assets/b00ea063-7afc-402a-97df-81a6584ef1a9)


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
![image](https://github.com/user-attachments/assets/f7b80e09-ef14-40db-a3bc-d20a3206f1d2)


**Step 2:** Access the Game
Go to: [http://localhost:8080](http://localhost:8080)
![image](https://github.com/user-attachments/assets/ad36ed9f-8ad5-4b8c-b686-fb50f151752c)

---

🛠️ Installing Required Tools

📦 Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client --output=yaml
```
![image](https://github.com/user-attachments/assets/5673c2de-342e-458b-afa2-186caf6ee777)

☁️ Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

**Configure Credentials:**

1. Go to IAM → Create User → Name: `mario`
2. ![image](https://github.com/user-attachments/assets/b5f6a742-76f0-4ba4-961e-97eda768ae66)

3. Attach `AdministratorAccess` policy
4. ![image](https://github.com/user-attachments/assets/0c4aa37a-4b62-4c13-b666-3c6ea28f013c)

5. Create access key
6. ![image](https://github.com/user-attachments/assets/c282db93-b684-4dc6-ba14-21c136765507)


```bash
aws configure
```
You’ll be prompted to enter:

AWS Access Key ID
AWS Secret Access Key
Default Region (e.g., ap-south-1)
Output Format (you can leave it as json)
✅ You’re now authenticated and ready to interact with AWS from your

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
![image](https://github.com/user-attachments/assets/55c21430-30e1-4e54-8f44-af62fd07e614)


```bash
cd terr-config
terraform init
terraform plan
terraform apply --auto-approve
```
![image](https://github.com/user-attachments/assets/672bba56-d6f2-4b5e-8378-83ba0b4075be)

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
![image](https://github.com/user-attachments/assets/c235b4f4-327e-4da9-8877-3f2fb75bc5e1)


4️⃣ Get LoadBalancer URL

```bash
kubectl describe service mario-service
```
![image](https://github.com/user-attachments/assets/297f51ea-9548-47c4-b2c5-2b37a6582bcb)

🎉 **Congratulations!** Your Super Mario game is now live on your AWS EKS cluster!
![image](https://github.com/user-attachments/assets/9317e848-50b3-4827-8d70-ac5efbde9a1b)

---

🧹 Clean-Up Resources

Avoid extra AWS charges:

```bash
cd terr-config
terraform destroy --auto-approve
```
![image](https://github.com/user-attachments/assets/87b8494b-665b-479b-9940-060bef788145)
![image](https://github.com/user-attachments/assets/f935bc0c-2d50-4099-bdae-5f70ddeb12da)

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
