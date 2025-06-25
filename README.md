# 🗳️ Docker Voting App – Manual Deployment (EC2 + Docker)

This project demonstrates a containerized voting application built using Python, Node.js, .NET, Redis, and PostgreSQL. The setup avoids Docker Compose, using individual Docker containers on an AWS EC2 instance.

## 📌 Repository

GitHub: [https://github.com/mohammedashiqu/docker-voting-app-new](https://github.com/mohammedashiqu/docker-voting-app-new)

## 📁 Folder Structure.
├── vote/ # Flask Voting app (Python)
├── result/ # Result app (Node.js)
├── worker/ # Worker service (.NET Core)
└── README.md

## 🔧 Project Components

This project includes the following services:

1. **Voting App** – Flask-based web frontend to cast votes (Python).
2. **Redis** – Message queue to temporarily store votes.
3. **Worker** – Background service that consumes votes and writes them to the database (.NET Core).
4. **DB** – PostgreSQL database for storing results.
5. **Result App** – Web app to display the voting results (Node.js).

## ⚙️ Step-by-Step Deployment Guide

### 🟢 Step 1: Launch EC2 Instance

- Create an **Amazon Linux 2** or **Ubuntu EC2 instance**
- Configure **Security Group** with inbound rules:
  - TCP: `20`, `1000`, `1001`, `2345`, `443` (custom ports for app access)

### 🟢 Step 2: Connect to EC2 Instance
Use **MobaXterm** or any SSH terminal to connect:
ssh -i "your-key.pem" ec2-user@your-ec2-public-ip

🟢 Step 3: Install Git and Clone the Repository
sudo yum install git -y       
git clone https://github.com/mohammedashiqu/docker-voting-app-new.git
cd docker-voting-app-new/

🟢 Step 4: Install Docker and Start Docker Service
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user

🟢 Step 5: Build Docker Images
Each directory has its own Dockerfile. Run the following commands inside the root repo directory:
docker build -t voting-app ./vote
docker build -t result-app ./result
docker build -t worker ./worker
docker pull redis
docker pull postgres

🟢 Step 6: Create Docker Containers & Network
Run the containers manually as below:
# Redis container
docker run -itd --name=redis -h=redis redis

# PostgreSQL container
docker run -itd --name=db -h=db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 2345:5432 \
  postgres

# Result app container (Node.js)
docker run -itd --name=result \
  --link db:db \
  -p 1001:80 \
  result-app

# Voting app container (Flask)
docker run -itd --name=voting-app \
  --link redis:redis \
  -p 1000:80 \
  voting-app

# Worker container (.NET)
docker run -itd --name=worker \
  --link redis:redis \
  --link db:db \
  worker
⚠️ Note: If using Docker networks instead of --link, you can create a custom bridge network and attach all containers to it.
