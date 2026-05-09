# Complete AWS Deployment Guide for TravelMemory MERN Stack Application

# Introduction

This document contains the complete end-to-end deployment process followed for deploying the TravelMemory MERN stack application on AWS Cloud infrastructure.

The purpose of this project was to understand how production-level applications are deployed using cloud services and scalable infrastructure.

The deployment architecture includes:

- React Frontend
- Node.js + Express Backend
- MongoDB Atlas Database
- AWS EC2
- Nginx Reverse Proxy
- PM2 Process Manager
- Application Load Balancer
- Auto Scaling Group
- Cloudflare DNS
- SSL using Certbot

This document also includes:

- Complete deployment steps
- Configuration details
- Commands executed
- Errors faced
- Troubleshooting process
- Security practices followed
- Learning outcomes

---

# Project Overview

## Application Name

TravelMemory

## Technology Stack Used

| Technology | Purpose |
|---|---|
| React.js | Frontend UI |
| Node.js | Runtime Environment |
| Express.js | Backend Framework |
| MongoDB Atlas | Database |
| AWS EC2 | Application Hosting |
| Nginx | Reverse Proxy |
| PM2 | Backend Process Management |
| AWS ALB | Traffic Distribution |
| Auto Scaling Group | Scalability |
| Cloudflare | DNS + SSL |

---

# Objective of the Project

The objective of this assignment was to:

- Deploy a MERN stack application on AWS EC2
- Configure frontend and backend communication
- Configure reverse proxy using Nginx
- Keep backend running using PM2
- Create scalable architecture using Load Balancer and Auto Scaling Group
- Connect custom domain using Cloudflare
- Enable HTTPS security using SSL certificates
- Understand production deployment workflow

---

# High-Level Architecture

## Request Flow
<img width="3192" height="1513" alt="image" src="https://github.com/user-attachments/assets/75cc0848-73c6-417e-ba4c-73ae6eafd57a" />


Users → Cloudflare → Application Load Balancer → EC2 Instances → Backend API → MongoDB Atlas

---

# Why Each Service Was Used

## Why EC2 Was Used

Amazon EC2 was used to host the MERN application.

Benefits:

- Full server access
- Linux server management experience
- Real-world deployment practice
- Scalable cloud infrastructure

---

## Why MongoDB Atlas Was Used

MongoDB Atlas is a managed cloud database service.

Benefits:

- No manual database installation
- Automatic backups
- Cloud accessibility
- Easy connection with backend

---

## Why Nginx Was Used

Backend application runs internally on:

localhost:3000

Instead of exposing backend port directly, Nginx was used as reverse proxy.

Benefits:

- Improved security
- Better request handling
- Easier frontend hosting
- Standard production practice

---

## Why PM2 Was Used

When backend is started using:

```bash
node index.js
```

application stops after terminal closes.

PM2 helps by:

- Running app in background
- Restarting automatically on crash
- Starting automatically after reboot
- Managing logs

---

## Why Load Balancer Was Used

Application Load Balancer distributes traffic across multiple EC2 instances.

Benefits:

- High availability
- Fault tolerance
- Better scalability
- Reduced downtime

---

## Why Auto Scaling Group Was Used

Auto Scaling automatically creates or removes EC2 instances based on load.

Benefits:

- Better performance
- High availability
- Automatic scaling
- Cost optimization

---

# Prerequisites

Before starting deployment the following accounts and tools were prepared:

- AWS Account
- GitHub Account
- MongoDB Atlas Account
- Cloudflare Account
- Domain Name
- SSH Key Pair
- Internet Connection
- Git Installed Locally

---

# Step 1 — Create MongoDB Atlas Database

## Purpose

MongoDB Atlas is used as cloud-hosted database service.

---

## Steps Followed

### 1. Open MongoDB Atlas

Visited:

https://www.mongodb.com/atlas

---

### 2. Create Project

Created project named:

TravelMemory

---

### 3. Create Cluster

Selected:

- AWS Provider
- Free Shared Cluster
- Closest Region

---

### 4. Create Database User

Created:

- Username
- Password

Used these credentials later in connection string.

---

### 5. Configure Network Access

Added:

0.0.0.0/0

This temporarily allows connections from any IP address.

---

### 6. Obtain Connection String

Example:

```text
mongodb+srv://username:password@cluster.mongodb.net/travelmemory
```

This connection string is used inside backend .env file.

---

# Step 2 — Launch AWS EC2 Instance

## Purpose

EC2 instance acts as Linux server hosting the MERN application.

---

## Configuration Used

| Setting | Value |
|---|---|
| AMI | Amazon Linux 2023 |
| Instance Type | t2.micro |
| Storage | 16 GB |
| Architecture | x86_64 |

---

## Steps Performed

### 1. Open EC2 Dashboard

Logged into AWS Console and opened EC2 service.

---

### 2. Click Launch Instance

Configured instance settings.

---

### 3. Create Key Pair

Created:

travelmemory-key.pem

Downloaded key safely.

---

### 4. Configure Security Group

Opened required ports:

| Port | Purpose |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3000 | Backend Testing |

---

### 5. Launch Instance

Successfully launched EC2 instance.

---

# Step 3 — Connect to EC2 Instance

Connected using SSH.

```bash
ssh -i travelmemory-key.pem ec2-user@PUBLIC-IP
```

Successfully connected to Linux terminal.

---

# Step 4 — Update Linux Packages

Updated packages.

```bash
sudo dnf update -y
```

Purpose:

- Install latest updates
- Improve security
- Improve compatibility

---

# Step 5 — Install Required Software

## Install Git

```bash
sudo dnf install git -y
```

Verified:

```bash
git --version
```

---

## Install Node.js

```bash
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo dnf install nodejs -y
```

Verified:

```bash
node -v
npm -v
```

---

## Install Nginx

```bash
sudo dnf install nginx -y
```

Started nginx.

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

Verified nginx status.

```bash
sudo systemctl status nginx
```

---

## Install PM2

```bash
sudo npm install -g pm2
```

Purpose:

- Run backend in background
- Restart backend automatically
- Keep backend alive after logout

---

# Step 6 — Clone GitHub Repository

Cloned repository.

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
```

Moved into project directory.

```bash
cd TravelMemory
```

---

# Step 7 — Backend Setup

Moved into backend directory.

```bash
cd backend
```

Installed backend dependencies.

```bash
npm install
```

---

# Step 8 — Configure Environment Variables

Created .env file.

```bash
nano .env
```

Added:

```env
PORT=3000
MONGO_URI=YOUR_MONGODB_CONNECTION_STRING
JWT_SECRET=travelmemorysecret
```

---

# Step 9 — Test Backend Application

Started backend manually.

```bash
node index.js
```

Verified backend using:

http://EC2-PUBLIC-IP:3000

Backend responded successfully.

Stopped process using:

CTRL + C

---

# Step 10 — Configure PM2

Started backend using PM2.

```bash
pm2 start index.js --name backend
```

Checked status.

```bash
pm2 status
```

Saved PM2 configuration.

```bash
pm2 save
```

Enabled PM2 startup service.

```bash
pm2 startup
```

Executed generated command.

---

# Step 11 — Configure Nginx Reverse Proxy

## Purpose

Nginx handles requests on port 80 and forwards them internally to backend running on port 3000.

---

## Create Nginx Configuration

```bash
sudo nano /etc/nginx/conf.d/travelmemory.conf
```

Added:

```nginx
server {
    listen 80;

    server_name _;

    location / {
        proxy_pass http://localhost:3000;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

---

## Validate Nginx Configuration

```bash
sudo nginx -t
```

Output:

syntax is ok

test is successful

---

## Restart Nginx

```bash
sudo systemctl restart nginx
```

Enabled nginx on boot.

```bash
sudo systemctl enable nginx
```

---

# Step 12 — Frontend Setup

Moved into frontend directory.

```bash
cd ~/TravelMemory/frontend
```

Installed frontend dependencies.

```bash
npm install
```

---

# Step 13 — Configure Frontend Backend URL

Opened:

src/urls.js

Changed:

```javascript
http://localhost:3000
```

To:

```javascript
http://YOUR-DOMAIN
```

Reason:

Frontend cannot communicate with localhost after deployment.

---

# Step 14 — Create React Production Build

Generated optimized frontend build.

```bash
npm run build
```

Build folder created successfully.

---

# Step 15 — Host Frontend Using Nginx

Copied build files.

```bash
sudo cp -r build/* /usr/share/nginx/html/
```

Restarted nginx.

```bash
sudo systemctl restart nginx
```

Frontend became publicly accessible.

---

# Step 16 — Create AMI

## Purpose

AMI is used to create reusable EC2 images.

Benefits:

- Launch identical instances
- Useful for Auto Scaling
- Saves deployment time

---

## Steps

1. Open EC2 Dashboard
2. Select Instance
3. Actions → Image → Create Image

Created image:

travelmemory-ami

---

# Step 17 — Create Target Group

## Purpose

Load Balancer forwards traffic to instances registered inside Target Group.

---

## Configuration

| Setting | Value |
|---|---|
| Type | Instances |
| Protocol | HTTP |
| Port | 80 |

Health Check Path:

/

---

# Step 18 — Create Application Load Balancer

Created internet-facing ALB.

Configured:

| Setting | Value |
|---|---|
| Listener | HTTP : 80 |
| Availability Zones | Multiple |
| Target Group | Existing Target Group |

ALB DNS name generated successfully.

---

# Step 19 — Create Launch Template

Created Launch Template using:

- Existing AMI
- Security Group
- Key Pair
- Instance Type

Purpose:

Reusable configuration for Auto Scaling Group.

---

# Step 20 — Create Auto Scaling Group

Configured:

| Setting | Value |
|---|---|
| Minimum Instances | 2 |
| Desired Instances | 2 |
| Maximum Instances | 4 |

Attached existing Load Balancer.

---

# Step 21 — Configure Cloudflare Domain

Added custom domain inside Cloudflare dashboard.

Updated nameservers in domain registrar.

---

# DNS Records Configured

## CNAME Record

www → LOAD-BALANCER-DNS

---

## A Record

@ → EC2-PUBLIC-IP

---

# Step 22 — Configure SSL Certificate

Installed Certbot.

```bash
sudo dnf install certbot python3-certbot-nginx -y
```

Generated SSL certificate.

```bash
sudo certbot --nginx -d yourdomain.com
```

HTTPS enabled successfully.

---

# Major Problems Faced During Deployment

# Problem 1 — Frontend Could Not Connect Backend

## Cause

Frontend was still calling:

localhost:3000

which only works locally.

---

## Solution

Updated backend API URL inside:

src/urls.js

with deployed domain.

---

# Problem 2 — Backend Stopped After Logout

## Cause

Backend was started using:

```bash
node index.js
```

which terminates after SSH session closes.

---

## Solution

Used PM2 process manager.

---

# Problem 3 — Nginx Restart Failed

## Cause

Syntax errors inside nginx configuration.

---

## Solution

Validated nginx configuration.

```bash
sudo nginx -t
```

---

# Problem 4 — Load Balancer Health Check Failed

## Causes

- Backend not running
- Port mismatch
- Incorrect health check path
- Nginx issues
- Security Group restrictions

---

## Solution

Verified:

- PM2 backend status
- Nginx service
- Port 80 accessibility
- Security Group rules

---

# Problem 5 — MongoDB Atlas Connection Failed

## Cause

MongoDB Atlas blocked external connection.

---

## Solution

Added:

0.0.0.0/0

inside Network Access.

---

# Problem 6 — SSL Certificate Failed

## Cause

DNS propagation delay.

---

## Solution

Waited for DNS propagation and reran certbot.

---

# Security Best Practices Followed

- Configured Security Groups properly
- Enabled HTTPS
- Used Cloudflare protection
- Used Nginx reverse proxy
- Avoided exposing backend publicly
- Used PM2 process management

---

# Additional Concepts Learned

Through this deployment the following concepts were learned:

- Reverse Proxy Architecture
- Linux Server Administration
- Cloud Networking
- SSL/TLS Workflow
- DNS Resolution
- Load Balancing
- Auto Scaling
- Production Deployment Workflow
- Cloud Infrastructure Management

---

# Learning Outcomes

This project provided hands-on experience with:

- AWS EC2 deployment
- Linux server management
- MongoDB Atlas integration
- Nginx configuration
- PM2 process management
- Application Load Balancer
- Auto Scaling Group
- Cloudflare DNS setup
- SSL certificate management
- Troubleshooting deployment issues

---

# Final Outcome

Successfully deployed the TravelMemory MERN stack application on AWS Cloud infrastructure with:

- Multiple EC2 instances
- Load Balancer
- Auto Scaling Group
- MongoDB Atlas
- Cloudflare DNS
- HTTPS Security

The application became publicly accessible through custom domain with scalable cloud architecture.

---

# Conclusion

This project successfully demonstrated deployment of a MERN stack application using AWS Cloud infrastructure and production-level deployment practices.

The project helped in understanding:

- Cloud deployment workflow
- Linux server management
- Reverse proxy configuration
- Load balancing
- Scalable architecture
- SSL configuration
- Production debugging

This deployment provided practical exposure to real-world DevOps and cloud infrastructure concepts.

---

# GitHub Repository

https://github.com/sairamraavi/TravelMemory

