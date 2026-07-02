# Self-Hosted GitHub Runner Setup Guide

## 📋 Overview

This guide will help you set up a self-hosted GitHub Actions runner on your VM for automated deployments.

## 🖥️ Prerequisites

### On Your VM:
- Ubuntu/Debian or similar Linux distribution
- Docker installed and running
- Git installed
- `curl` and `jq` installed
- Port 4041 available
- Sudo/root access

### On GitHub:
- Repository admin access
- `DOCKER_TOKEN` secret configured

## 🚀 Step 1: Install Docker (if not already installed)

```bash
# Update package list
sudo apt update

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group (to run docker without sudo)
sudo usermod -aG docker $USER

# Verify installation
docker --version

# Start Docker service
sudo systemctl enable docker
sudo systemctl start docker
```

**⚠️ Important**: Log out and log back in for the docker group change to take effect.

## 🤖 Step 2: Add Self-Hosted Runner to GitHub

### Navigate to Runner Settings:
1. Go to your GitHub repository: https://github.com/gisul-ecosystem/linux-engine
2. Click **Settings** tab
3. Click **Actions** → **Runners**
4. Click **New self-hosted runner**

### Choose Your Platform:
- **Operating System**: Linux
- **Architecture**: x64 (or ARM if applicable)

### Follow GitHub's Instructions:

GitHub will show you commands like these (use the exact commands from your page):

```bash
# Download the runner
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure the runner
./config.sh --url https://github.com/gisul-ecosystem/linux-engine --token YOUR_TOKEN_HERE

# During configuration:
# - Enter runner name: linux-engine-vm-runner
# - Enter runner group: [press Enter for default]
# - Enter labels: [press Enter for default]
# - Enter work folder: [press Enter for default]
```

## 🔧 Step 3: Install Runner as a Service

Run the runner as a systemd service so it starts automatically:

```bash
# Install the service
cd ~/actions-runner
sudo ./svc.sh install

# Start the service
sudo ./svc.sh start

# Check status
sudo ./svc.sh status

# View logs
journalctl -u actions.runner.* -f
```

## ✅ Step 4: Verify Runner is Connected

1. Go back to GitHub: **Settings** → **Actions** → **Runners**
2. You should see your runner listed with a green "Idle" status
3. If it shows "Offline", check the service logs

## 🔒 Step 5: Configure Firewall (if applicable)

Allow port 4041 for the application:

```bash
# UFW firewall
sudo ufw allow 4041/tcp
sudo ufw reload

# Or iptables
sudo iptables -A INPUT -p tcp --dport 4041 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

## 📦 Step 6: Install Required Tools on VM

The runner needs these tools for the deployment workflow:

```bash
# Install curl and jq (if not already installed)
sudo apt update
sudo apt install -y curl jq

# Verify installations
curl --version
jq --version
docker --version
```

## 🧪 Step 7: Test the Deployment

### Trigger a Deployment:

```bash
# On your development machine
cd /path/to/linux-engine
git add .
git commit -m "Test self-hosted runner deployment"
git push origin main
```

### Monitor the Deployment:

1. **On GitHub**:
   - Go to **Actions** tab
   - Watch the workflow run
   - Check each job's logs

2. **On Your VM**:
   ```bash
   # Watch runner logs
   journalctl -u actions.runner.* -f
   
   # Check Docker containers
   docker ps
   
   # View application logs
   docker logs -f linux-engine
   
   # Test the application
   curl http://localhost:4041/health
   ```

## 📊 Workflow Overview

The updated workflow now has 5 jobs:

```
1. test (ubuntu-latest)
   └─> Validates code and configuration

2. build-and-push (ubuntu-latest)
   └─> Builds and pushes Docker image to Docker Hub

3. deploy-to-vm (self-hosted) ⭐ NEW
   └─> Pulls image and deploys to your VM
   
4. security-scan (ubuntu-latest)
   └─> Scans for vulnerabilities

5. deployment-notification (ubuntu-latest)
   └─> Reports overall status
```

## 🎯 What the deploy-to-vm Job Does:

1. ✅ Logs in to Docker Hub
2. ✅ Pulls latest image
3. ✅ Stops old container (if exists)
4. ✅ Starts new container with correct settings
5. ✅ Waits for service to be healthy
6. ✅ Verifies deployment
7. ✅ Cleans up old images

## 🔍 Troubleshooting

### Runner Shows Offline:

```bash
# Check service status
sudo systemctl status actions.runner.*

# Restart service
sudo ./svc.sh stop
sudo ./svc.sh start

# Check logs
journalctl -u actions.runner.* -n 50
```

### Deployment Job Fails:

```bash
# Check if Docker is running
sudo systemctl status docker

# Check Docker permissions
docker ps  # Should work without sudo

# If permission denied, add user to docker group
sudo usermod -aG docker $(whoami)
# Then log out and back in
```

### Container Won't Start:

```bash
# Check for port conflicts
sudo lsof -i :4041

# View detailed logs
docker logs linux-engine

# Check Docker images
docker images | grep linux-engine
```

### Health Check Fails:

```bash
# Test manually
curl -v http://localhost:4041/health

# Check if port is accessible
netstat -tlnp | grep 4041

# Check container status
docker ps -a | grep linux-engine
```

## 🔄 Updating the Runner

To update the runner software:

```bash
cd ~/actions-runner

# Stop the service
sudo ./svc.sh stop

# Download new version (check GitHub for latest)
curl -o actions-runner-linux-x64-NEW_VERSION.tar.gz -L https://github.com/actions/runner/releases/download/vNEW_VERSION/actions-runner-linux-x64-NEW_VERSION.tar.gz

# Extract
tar xzf ./actions-runner-linux-x64-NEW_VERSION.tar.gz

# Start the service
sudo ./svc.sh start
```

## 🗑️ Removing the Runner

If you need to remove the runner:

```bash
cd ~/actions-runner

# Stop the service
sudo ./svc.sh stop

# Uninstall the service
sudo ./svc.sh uninstall

# Remove the runner from GitHub
./config.sh remove --token YOUR_REMOVAL_TOKEN

# Delete the folder
cd ~
rm -rf actions-runner
```

## 📝 Runner Configuration File

The runner configuration is stored in:
```
~/actions-runner/.runner
~/actions-runner/.credentials
```

**Do not share or commit these files!**

## 🔐 Security Best Practices

1. ✅ **Use dedicated user**: Create a separate user for running actions
2. ✅ **Limit permissions**: Don't run runner as root
3. ✅ **Firewall rules**: Only open necessary ports
4. ✅ **Regular updates**: Keep runner software updated
5. ✅ **Monitor logs**: Regularly check runner and application logs
6. ✅ **Secrets management**: Never expose secrets in logs

## 📈 Monitoring Your Runner

### Check Runner Status:
```bash
# Service status
sudo systemctl status actions.runner.*

# Real-time logs
journalctl -u actions.runner.* -f

# Last 100 lines
journalctl -u actions.runner.* -n 100
```

### Check Application Status:
```bash
# Container status
docker ps | grep linux-engine

# Application logs
docker logs --tail 100 -f linux-engine

# Health check
curl http://localhost:4041/health
```

### Resource Usage:
```bash
# System resources
htop

# Docker stats
docker stats linux-engine

# Disk usage
df -h
docker system df
```

## 🎉 Success Criteria

Your setup is complete when:

✅ Runner shows "Idle" status in GitHub  
✅ Workflow runs successfully on push to main  
✅ Docker image is pulled and container starts  
✅ Health check endpoint returns `{"status":"ok"}`  
✅ Application is accessible on port 4041  
✅ Logs show no errors  

## 🆘 Getting Help

If you encounter issues:

1. Check runner logs: `journalctl -u actions.runner.* -f`
2. Check Docker logs: `docker logs linux-engine`
3. Check workflow logs in GitHub Actions tab
4. Verify all prerequisites are installed
5. Ensure firewall allows required ports

## 📞 Quick Commands Reference

```bash
# Runner Management
sudo systemctl status actions.runner.*  # Check status
sudo ./svc.sh start                     # Start runner
sudo ./svc.sh stop                      # Stop runner
sudo ./svc.sh status                    # Status
journalctl -u actions.runner.* -f       # View logs

# Docker Management
docker ps                                # List containers
docker logs -f linux-engine             # View logs
docker restart linux-engine             # Restart
docker stop linux-engine                # Stop
docker rm linux-engine                  # Remove

# Application Testing
curl http://localhost:4041/health       # Health check
docker exec -it linux-engine bash       # Shell access

# Cleanup
docker system prune -a                  # Clean all
docker image prune -a                   # Clean images
```

---

**Status**: Ready for Setup  
**Estimated Setup Time**: 15-30 minutes  
**Last Updated**: July 2, 2026
