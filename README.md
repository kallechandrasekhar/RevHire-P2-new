# Rev-Hire Deployment Guide

## Prerequisites

### EC2 Instance Setup (Amazon Linux)
1. Launch an Amazon Linux EC2 instance
2. Install Docker and Docker Compose:
   ```bash
   sudo yum update -y
   sudo amazon-linux-extras install docker
   sudo service docker start
   sudo usermod -a -G docker ec2-user
   # Install docker-compose
   sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

### GitHub Secrets Setup
Go to Repository → Settings → Secrets and variables → Actions and add:

| Secret Name | Value |
|-------------|-------|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_PASSWORD` | Your Docker Hub password/token |
| `EC2_SSH_KEY` | Open your .pem file with a text editor and paste the entire content (including -----BEGIN... and -----END...) |
| `EC2_USER` | ec2-user (default for Amazon Linux AMI) |
| `EC2_HOST` | Your EC2 public IP or DNS (e.g., 54.123.45.67) |

### Deployment Keys Setup
1. Generate SSH key on EC2:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "deployment@ec2"
   ```
2. Add the public key (`cat ~/.ssh/id_rsa.pub`) as a deploy key in GitHub repo settings
3. Clone the repo on EC2:
   ```bash
   git clone git@github.com:yourusername/Rev_Hire.git
   ```

## Deployment

1. Push code to `main` branch - triggers CI build
2. Go to Actions tab → "CI/CD Pipeline" → "Run workflow"
3. Select "deploy: true" and run

## Access
- Frontend: `http://your-ec2-ip`
- Backend: `http://your-ec2-ip:8080`