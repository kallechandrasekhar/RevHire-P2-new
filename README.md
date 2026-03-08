# Rev-Hire Deployment Guide

## Prerequisites

### EC2 Instance Setup (Amazon Linux)
1. Launch an Amazon Linux EC2 instance
2. Install required packages:
   ```bash
   sudo yum update -y
   sudo yum install git -y
   sudo amazon-linux-extras install docker
   sudo service docker start
   sudo usermod -a -G docker ec2-user
   # Install Docker Compose (latest)
   sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   # Install Docker Buildx (for ec2-user)
   mkdir -p ~/.docker/cli-plugins
   curl -L https://github.com/docker/buildx/releases/latest/download/buildx-$(uname -s)-$(uname -m) -o ~/.docker/cli-plugins/docker-buildx
   chmod +x ~/.docker/cli-plugins/docker-buildx
   # Verify
   docker buildx version
   # Logout and login again for docker group
   ```
3. Clone your repository:
   ```bash
   cd /home/ec2-user
   git clone git@github.com:yourusername/Rev_Hire.git
   # Or if already cloned, ensure the path is /home/ec2-user/Rev_Hire
   ```

### GitHub Secrets Setup
Go to Repository → Settings → Secrets and variables → Actions and add:

| Secret Name | Value |
|-------------|-------|
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

### Troubleshooting SSH Key Issues

If you get "Key is invalid. You must supply a key in OpenSSH public key format":

**Recommended Solution: Generate a new OpenSSH key pair**

1. **Generate new SSH key on your local machine**:
   ```bash
   ssh-keygen -t rsa -b 2048 -f deploy_key -N ""
   # This creates deploy_key and deploy_key.pub
   ```

2. **Add the public key to EC2**:
   ```bash
   # SSH to EC2 using your existing method
   ssh -i your-existing-key.pem ec2-user@your-ec2-ip
   
   # On EC2, add the public key
   echo "paste_content_of_deploy_key.pub_here" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

3. **Update GitHub secret**:
   - Copy the entire content of `deploy_key` (private key file)
   - Paste it as the `EC2_SSH_KEY` secret in GitHub

4. **Test the new key**:
   ```bash
   ssh -i deploy_key ec2-user@your-ec2-ip
   ```

**Alternative: Convert existing PEM key**
```bash
# Convert PEM to OpenSSH format
openssl rsa -in your-key.pem -out openssh_key
# Use content of openssh_key for the secret
```

### Deployment Path Issues
If you get "No such file or directory" for the repo path:
- Ensure your repository is cloned to `/home/ec2-user/Rev_Hire` on EC2
- If different, update the path in `.github/workflows/deploy.yml`
- Run `pwd` on EC2 to confirm the current directory after SSH