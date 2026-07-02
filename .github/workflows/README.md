# GitHub Actions Workflows

## Available Workflows

### 1. Build and Deploy Docker Image (`deploy.yml`)

**Purpose**: Automated CI/CD pipeline for building, testing, and deploying Docker images to Docker Hub.

**Triggers**:
- Push to `main` or `develop` branches
- Version tags (e.g., `v1.0.0`)
- Pull requests (tests only)
- Manual dispatch

**Jobs**:
1. **Test**: Validates configuration and runs tests
2. **Build and Push**: Creates multi-platform Docker images
3. **Security Scan**: Scans for vulnerabilities with Trivy
4. **Notification**: Deployment status summary

**Output**: 
- Docker images at `gisulecosystem/linux-engine:latest`
- Multi-platform support (amd64, arm64)
- Security scan results in GitHub Security tab

## Configuration

### Required Secrets:
- `DOCKER_TOKEN`: Docker Hub access token for `gisul-docker` account

### Environment Variables:
```yaml
DOCKER_USERNAME: gisul-docker
IMAGE_NAME: gisulecosystem/linux-engine
REGISTRY: docker.io
```

## Usage

### Automatic Deployment:
```bash
# Deploy to production (latest tag)
git push origin main

# Deploy to development
git push origin develop

# Create versioned release
git tag v1.0.0
git push origin v1.0.0
```

### Manual Deployment:
1. Go to Actions tab in GitHub
2. Select "Build and Deploy Docker Image"
3. Click "Run workflow"
4. Choose branch and click "Run workflow"

## Monitoring

- **Workflow runs**: GitHub Actions tab
- **Docker images**: https://hub.docker.com/r/gisulecosystem/linux-engine
- **Security alerts**: GitHub Security tab

## Documentation

See `GITHUB_ACTIONS_SETUP.md` for complete setup instructions and troubleshooting guide.
