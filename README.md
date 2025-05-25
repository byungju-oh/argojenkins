# Flask Shop Application with CI/CD Pipeline

A containerized Flask e-commerce application with automated CI/CD pipeline using Jenkins and ArgoCD, deployed on Kubernetes.

## 🏗️ Architecture

This project implements a microservices architecture with the following components:

- **Frontend**: Nginx reverse proxy
- **Backend**: Flask application with Celery for background tasks
- **Cache/Message Broker**: Redis
- **Database**: MySQL
- **Storage**: Google Cloud Storage (GCS)
- **CI/CD**: Jenkins + ArgoCD (GitOps)

## 📋 Prerequisites

- Kubernetes cluster
- Docker Hub account
- GitHub repositories (source code + manifest repository)
- Jenkins with required plugins
- ArgoCD installed on Kubernetes
- Google Cloud credentials for GCS

## 🚀 Quick Start

### 1. Clone Repositories

```bash
# Application source code
git clone https://github.com/byungju-oh/shop.git

# Kubernetes manifests (this repository)
git clone https://github.com/byungju-oh/argojenkins.git
```

### 2. Configure Secrets

Create the required Kubernetes secrets:

```bash
# MySQL and application secrets
kubectl apply -f was/sec.yaml

# Google Cloud credentials
kubectl create secret generic google-credentials \
  --from-file=google-credentials.json=/path/to/your/gcs-credentials.json
```

### 3. Deploy with ArgoCD

```bash
# Apply ArgoCD application
kubectl apply -f application.yaml
```

## 📁 Project Structure

```
├── nginx/
│   ├── dep.yaml          # Nginx deployment and ConfigMap
│   └── ser.yaml          # Nginx service
├── was/
│   ├── dep.yaml          # Flask application deployment and service
│   ├── redis.yaml        # Redis deployment and service
│   ├── celery.yaml       # Celery worker deployment
│   ├── sec.yaml          # Secrets configuration
│   └── version.txt       # Current application version
├── Jenkinsfile           # CI/CD pipeline configuration
├── Jenkinsfile.back      # Backup pipeline configuration
└── application.yaml      # ArgoCD application definition
```

## 🔧 Configuration

### Environment Variables

The Flask application uses the following environment variables:

- `DATABASE_URI`: MySQL connection string
- `SECRET_KEY`: Flask secret key
- `GCS_BUCKET_NAME`: Google Cloud Storage bucket name
- `GOOGLE_APPLICATION_CREDENTIALS`: Path to GCS credentials
- `CELERY_BROKER_URL`: Redis URL for Celery broker
- `CELERY_RESULT_BACKEND`: Redis URL for Celery results

### Resource Limits

#### Flask Application
- **Requests**: 250Mi memory, 210m CPU
- **Limits**: 280Mi memory, 220m CPU

#### Nginx
- **Requests**: 64Mi memory, 250m CPU
- **Limits**: 128Mi memory, 500m CPU

#### Celery Worker
- **Requests**: 250Mi memory, 210m CPU
- **Limits**: 280Mi memory, 220m CPU

## 🔄 CI/CD Pipeline

The Jenkins pipeline automates the following process:

1. **Clone Repositories**: Fetches source code and manifests
2. **Version Management**: Automatically increments version numbers
3. **Build**: Creates Docker image with new version tag
4. **Push**: Uploads image to Docker Hub
5. **Update Manifests**: Updates Kubernetes manifests with new image version
6. **GitOps**: Commits changes to trigger ArgoCD sync
7. **Notifications**: Sends Slack notifications on success/failure

### Pipeline Triggers

The pipeline can be triggered by:
- Git webhook on source code changes
- Manual execution
- Scheduled builds

## 🎯 Features

### Application Features
- E-commerce functionality
- Background task processing with Celery
- File storage integration with Google Cloud Storage
- Health checks and monitoring
- Horizontal scaling ready

### DevOps Features
- **GitOps**: ArgoCD manages deployments
- **Automated Versioning**: Semantic version management
- **Health Monitoring**: Liveness and readiness probes
- **Load Balancing**: Internal load balancer configuration
- **Secrets Management**: Kubernetes secrets for sensitive data

## 🔍 Monitoring & Health Checks

### Health Endpoints
- **Liveness Probe**: `GET /` (port 5000)
- **Readiness Probe**: `GET /` (port 5000)

### Probe Configuration
- **Initial Delay**: 30 seconds (Flask), 5 seconds (Nginx)
- **Period**: 10 seconds
- **Timeout**: Default (1 second)

## 🌐 Network Configuration

### Services
- **Nginx**: LoadBalancer service on port 80
- **Flask**: Internal LoadBalancer (10.178.0.100:80)
- **Redis**: ClusterIP service on port 6379

### DNS Configuration
- Custom DNS policy with ndots=2 for cluster-first resolution

## 🚨 Troubleshooting

### Common Issues

1. **Image Pull Errors**
   ```bash
   kubectl describe pod <pod-name>
   # Check if image exists in Docker Hub
   ```

2. **Secret Mount Issues**
   ```bash
   kubectl get secrets
   kubectl describe secret mysql-secret
   ```

3. **Service Connection Issues**
   ```bash
   kubectl get services
   kubectl get endpoints
   ```

### Logs
```bash
# Flask application logs
kubectl logs -f deployment/flask-app

# Celery worker logs
kubectl logs -f deployment/celery

# Nginx logs
kubectl logs -f deployment/nginx-proxy
```

## 📈 Scaling

### Horizontal Scaling
```bash
# Scale Flask application
kubectl scale deployment flask-app --replicas=3

# Scale Celery workers
kubectl scale deployment celery --replicas=2
```

### Vertical Scaling
Modify resource requests/limits in the respective deployment YAML files.

## 🔐 Security

- Secrets are base64 encoded and stored in Kubernetes secrets
- Google Cloud credentials are mounted as files
- Internal load balancer for Flask application
- No root privileges for containers

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📞 Support

For issues and questions:
- Create an issue in this repository
- Check the troubleshooting section
- Review application logs

---

**Current Version**: 1.21
