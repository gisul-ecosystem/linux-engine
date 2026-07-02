# GitHub Actions CI/CD Setup Guide

## 📋 Overview

This repository uses GitHub Actions for automated building, testing, and deployment of the Linux Engine Docker image to Docker Hub.

## 🔐 Required GitHub Secrets

You need to configure the following secret in your GitHub repository:

### DOCKER_TOKEN

This is your Docker Hub access token for authentication.

### How to Create Docker Hub Access Token:

1. **Log in to Docker Hub**
   - Go to https://hub.docker.com/
   - Sign in with username: `gisul-docker`

2. **Create Access Token**
   - Click on your username (top-right) → **Account Settings**
   - Go to **Security** tab
   - Click **New Access Token**
   - Name: `github-actions-linux-engine`
   - Access permissions: **Read, Write, Delete**
   - Click **Generate**
   - **IMPORTANT**: Copy the token immediately (you won't see it again!)

3. **Add Secret to GitHub Repository**
   - Go to your GitHub repository: https://github.com/gisul-ecosystem/linux-engine
   - Click **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `DOCKER_TOKEN`
   - Value: Paste the Docker Hub token you copied
   - Click **Add secret**

## 🚀 Workflow Overview

The deployment pipeline consists of 4 jobs:

### 1. **Test Job**
Runs on every push and pull request:
- ✅ Python syntax validation
- ✅ Configuration verification (`verify_config.py`)
- ✅ Dependency installation
- ✅ Optional test execution

### 2. **Build and Push Job**
Runs after tests pass (excludes PRs):
- 🏗️ Builds multi-platform Docker image (amd64 & arm64)
- 🏷️ Tags with multiple strategies (branch, version, sha)
- 📤 Pushes to Docker Hub: `gisulecosystem/linux-engine`
- 💾 Uses layer caching for faster builds

### 3. **Security Scan Job**
Runs after successful build:
- 🔍 Scans image with Trivy for vulnerabilities
- 📊 Uploads results to GitHub Security tab
- ⚠️ Identifies security issues in dependencies

### 4. **Deployment Notification**
Final status notification:
- ✅ Success/failure summary
- 📦 Image availability confirmation
- 🔗 Direct link to Docker Hub

## 🏷️ Docker Image Tagging Strategy

The workflow automatically creates multiple tags:

| Trigger | Example Tags |
|---------|--------------|
| Push to `main` | `latest`, `main`, `main-abc1234` |
| Push to `develop` | `develop`, `develop-abc1234` |
| Tag `v1.0.0` | `v1.0.0`, `1.0`, `1`, `latest` |
| Commit SHA | `main-abc1234` (short SHA) |

## 📦 Docker Image Details

- **Registry**: Docker Hub (docker.io)
- **Repository**: `gisulecosystem/linux-engine`
- **Username**: `gisul-docker`
- **Platforms**: 
  - `linux/amd64` (x86_64)
  - `linux/arm64` (ARM 64-bit)

## 🔄 Triggering the Workflow

### Automatic Triggers:

1. **Push to main branch**
   ```bash
   git push origin main
   ```
   Creates: `latest`, `main`, `main-<sha>` tags

2. **Push to develop branch**
   ```bash
   git push origin develop
   ```
   Creates: `develop`, `develop-<sha>` tags

3. **Create version tag**
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```
   Creates: `v1.0.0`, `1.0`, `1`, `latest` tags

4. **Pull Request**
   - Only runs tests (no build/push)

### Manual Trigger:

You can manually trigger the workflow from GitHub:
1. Go to **Actions** tab
2. Select **Build and Deploy Docker Image**
3. Click **Run workflow**
4. Choose branch
5. Click **Run workflow**

## 📊 Monitoring Deployments

### View Workflow Status:

1. Go to GitHub repository
2. Click **Actions** tab
3. Select a workflow run
4. View job logs and deployment summary

### Deployment Summary:

Each successful deployment creates a summary showing:
- Docker image tags
- Pull command
- Run command
- Platform support

### Docker Hub:

View your images at:
- https://hub.docker.com/r/gisulecosystem/linux-engine

## 🐛 Troubleshooting

### Issue: "Authentication Failed" Error

**Cause**: Invalid or expired Docker Hub token

**Solution**:
1. Generate new token on Docker Hub
2. Update `DOCKER_TOKEN` secret on GitHub
3. Re-run workflow

### Issue: Build Fails on Tests

**Cause**: Configuration or syntax errors

**Solution**:
1. Run locally: `python verify_config.py`
2. Fix any errors
3. Commit and push changes

### Issue: Multi-platform Build Fails

**Cause**: Platform-specific dependency issues

**Solution**:
1. Check Dockerfile for platform-specific commands
2. Use `--platform` flag for local testing:
   ```bash
   docker buildx build --platform linux/amd64,linux/arm64 .
   ```

### Issue: Security Scan Finds Vulnerabilities

**Cause**: Outdated dependencies or base image

**Solution**:
1. Check Security tab for details
2. Update `requirements.txt`:
   ```bash
   pip install --upgrade -r requirements.txt
   pip freeze > requirements.txt
   ```
3. Update base image in Dockerfile:
   ```dockerfile
   FROM python:3.11-slim  # Use latest patch version
   ```

## 🔒 Security Best Practices

1. **Never commit tokens** to repository
2. **Rotate access tokens** regularly (every 90 days)
3. **Use least privilege** - only required permissions
4. **Monitor security alerts** in GitHub Security tab
5. **Review Trivy scan results** after each deployment
6. **Keep dependencies updated** regularly

## 📝 Workflow Configuration Files

### Main Workflow:
- `.github/workflows/deploy.yml` - Complete CI/CD pipeline

### Key Configuration:
```yaml
env:
  DOCKER_USERNAME: gisul-docker
  IMAGE_NAME: gisulecosystem/linux-engine
  REGISTRY: docker.io
```

## 🚢 Deployment Commands

After successful build, use these commands:

### Pull Latest Image:
```bash
docker pull gisulecosystem/linux-engine:latest
```

### Pull Specific Version:
```bash
docker pull gisulecosystem/linux-engine:v1.0.0
```

### Run Container:
```bash
docker run -d \
  --name linux-engine \
  -p 4041:4041 \
  -e LOG_LEVEL=WARNING \
  -e WEB_CONCURRENCY=4 \
  --restart always \
  gisulecosystem/linux-engine:latest
```

### Using Docker Compose:
```bash
# Pull latest image
docker compose pull

# Start service
docker compose up -d

# View logs
docker compose logs -f terminal-engine
```

## 📈 Workflow Performance

### Build Times (Approximate):
- **Test Job**: 2-3 minutes
- **Build Job**: 5-8 minutes (first build)
- **Build Job**: 2-4 minutes (with cache)
- **Security Scan**: 1-2 minutes
- **Total Pipeline**: 8-15 minutes

### Optimization Features:
- ✅ Docker layer caching
- ✅ pip dependency caching
- ✅ Multi-stage builds
- ✅ Buildkit enabled

## 🔄 Versioning Strategy

We recommend following Semantic Versioning (SemVer):

- **MAJOR** version (v2.0.0): Breaking changes
- **MINOR** version (v1.1.0): New features, backward compatible
- **PATCH** version (v1.0.1): Bug fixes

### Creating a Release:

```bash
# Update version
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tag
git push origin v1.0.0

# GitHub Actions will automatically:
# - Build and test
# - Create Docker images with tags: v1.0.0, 1.0, 1, latest
# - Run security scan
# - Publish to Docker Hub
```

## 🔧 Customization

### Modify Build Platforms:

Edit `.github/workflows/deploy.yml`:
```yaml
platforms: linux/amd64,linux/arm64,linux/arm/v7
```

### Add Environment Variables:

Edit `.github/workflows/deploy.yml`:
```yaml
build-args: |
  LOG_LEVEL=WARNING
  CUSTOM_VAR=value
```

### Change Image Registry:

To use GitHub Container Registry (ghcr.io) instead:
```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ghcr.io/${{ github.repository }}
```

Then update login action:
```yaml
- name: Log in to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

## 📞 Support

### Common Issues:

1. **Build failures**: Check Actions logs in GitHub
2. **Token issues**: Verify secret is set correctly
3. **Image not updating**: Clear Docker Hub cache or use specific tag
4. **Multi-platform issues**: Test locally with buildx

### Resources:

- Docker Hub: https://hub.docker.com/r/gisulecosystem/linux-engine
- GitHub Actions Docs: https://docs.github.com/en/actions
- Docker Buildx Docs: https://docs.docker.com/buildx/

## ✅ Setup Checklist

- [x] Create `.github/workflows/deploy.yml`
- [ ] Generate Docker Hub access token
- [ ] Add `DOCKER_TOKEN` secret to GitHub
- [ ] Push to main branch to trigger first build
- [ ] Verify image on Docker Hub
- [ ] Test pulling and running image
- [ ] Set up branch protection rules (optional)
- [ ] Configure deployment notifications (optional)

---

**Status**: 🚀 Ready for Deployment  
**Last Updated**: July 2, 2026  
**Maintained By**: Gisul Ecosystem Team
