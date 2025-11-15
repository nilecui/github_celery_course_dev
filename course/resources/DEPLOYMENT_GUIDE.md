# Course Deployment Guide

**Last Updated:** 2025-11-15
**Purpose:** Guide for deploying the Celery Course Development Platform

---

## Overview

This guide covers the deployment of the Celery Course Development Platform in various environments, from local development setups to production deployments.

## Deployment Options

### 1. Local Development Setup

#### Prerequisites
- Docker and Docker Compose installed
- Git for version control
- 8GB+ RAM available
- 20GB+ free disk space

#### Setup Instructions
```bash
# Clone the repository
git clone <repository-url>
cd github_celery_course_dev

# Start development environment
docker-compose up -d

# Verify services are running
docker-compose ps

# Access course platform
# Documentation: http://localhost:7001
# RabbitMQ Management: http://localhost:15672 (guest/guest)
# Redis: localhost:6379
```

#### Environment Configuration
```bash
# Copy environment template
cp .env.example .env

# Edit configuration
nano .env
```

### 2. Production Deployment

#### System Requirements
- **CPU:** 4+ cores
- **RAM:** 16GB+ recommended
- **Storage:** 100GB+ SSD
- **Network:** Reliable internet connection

#### Docker Compose Production Setup
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - docs
      - api

  docs:
    build:
      context: ./celery/docker
      dockerfile: Dockerfile
    command: make livehtml
    environment:
      - PYTHONUNBUFFERED=1
    volumes:
      - ./celery/docs:/celery/docs
      - docs_build:/celery/docs/_build

  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/coursedb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14
    environment:
      - POSTGRES_DB=coursedb
      - POSTGRES_USER=courses
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
  docs_build:
```

#### SSL/TLS Configuration
```nginx
# nginx.conf
server {
    listen 443 ssl http2;
    server_name course.example.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;

    location / {
        proxy_pass http://docs:7001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 3. Kubernetes Deployment

#### Prerequisites
- Kubernetes cluster (v1.20+)
- kubectl configured
- Helm 3.0+

#### Namespace and Configuration
```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: celery-course
```

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: course-config
  namespace: celery-course
data:
  BROKER_URL: "redis://redis-service:6379"
  RESULT_BACKEND: "redis://redis-service:6379"
  COURSE_ENV: "production"
```

#### Documentation Service Deployment
```yaml
# k8s/docs-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docs-service
  namespace: celery-course
spec:
  replicas: 3
  selector:
    matchLabels:
      app: docs-service
  template:
    metadata:
      labels:
        app: docs-service
    spec:
      containers:
      - name: docs
        image: celery-course/docs:latest
        ports:
        - containerPort: 7001
        envFrom:
        - configMapRef:
            name: course-config
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /
            port: 7001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 7001
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Service and Ingress Configuration
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: docs-service
  namespace: celery-course
spec:
  selector:
    app: docs-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 7001
  type: ClusterIP

---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: course-ingress
  namespace: celery-course
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - course.example.com
    secretName: course-tls
  rules:
  - host: course.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: docs-service
            port:
              number: 80
```

#### Horizontal Pod Autoscaler
```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: docs-service-hpa
  namespace: celery-course
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: docs-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 4. Cloud Platform Deployment

#### AWS ECS Deployment

**Task Definition:**
```json
{
  "family": "celery-course",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "course-docs",
      "image": "your-registry/celery-course/docs:latest",
      "portMappings": [
        {
          "containerPort": 7001,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "BROKER_URL",
          "value": "redis://elasticache-cluster:6379"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/celery-course",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

**Service Definition:**
```json
{
  "cluster": "course-cluster",
  "serviceName": "course-docs-service",
  "taskDefinition": "celery-course",
  "desiredCount": 2,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-12345", "subnet-67890"],
      "securityGroups": ["sg-12345"],
      "assignPublicIp": "ENABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:region:account:targetgroup/course-targets/12345",
      "containerName": "course-docs",
      "containerPort": 7001
    }
  ]
}
```

#### Google Cloud Run Deployment

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 7001

CMD ["gunicorn", "--bind", "0.0.0.0:7001", "--workers", "4", "app:app"]
```

**Deployment Command:**
```bash
gcloud builds submit --tag gcr.io/PROJECT-ID/celery-course
gcloud run deploy celery-course \
  --image gcr.io/PROJECT-ID/celery-course \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --memory 512Mi \
  --cpu 1 \
  --max-instances 10
```

#### Azure Container Instances

**Deployment Template:**
```yaml
# azure-deployment.yaml
apiVersion: 2021-03-01
location: eastus
name: celery-course-group
properties:
  containers:
  - name: course-docs
    properties:
      image: yourregistry.azurecr.io/celery-course:latest
      ports:
      - port: 7001
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 2.0
      environmentVariables:
      - name: BROKER_URL
        value: redis://redis-cache:6379
  ipAddress:
    type: Public
    ports:
    - port: 80
      protocol: TCP
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

## Monitoring and Logging

### Prometheus Monitoring

**Configuration:**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'celery-course'
    static_configs:
      - targets: ['docs-service:7001']
    metrics_path: '/metrics'
    scrape_interval: 30s

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Grafana Dashboard

**Dashboard Configuration:**
```json
{
  "dashboard": {
    "title": "Celery Course Platform",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      }
    ]
  }
}
```

### ELK Stack for Logging

**Logstash Configuration:**
```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][service] == "celery-course" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }

    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "celery-course-%{+YYYY.MM.dd}"
  }
}
```

## Backup and Recovery

### Database Backup

**Automated Backup Script:**
```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# PostgreSQL backup
pg_dump -h postgres -U courses coursedb > $BACKUP_DIR/postgres_$DATE.sql

# Redis backup
redis-cli --rdb $BACKUP_DIR/redis_$DATE.rdb

# Compress backups
tar -czf $BACKUP_DIR/course_backup_$DATE.tar.gz \
  $BACKUP_DIR/postgres_$DATE.sql \
  $BACKUP_DIR/redis_$DATE.rdb

# Clean up old backups (keep 7 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

**Kubernetes CronJob for Backup:**
```yaml
# k8s/backup-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: course-backup
  namespace: celery-course
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:14
            command:
            - /bin/bash
            - -c
            - |
              pg_dump -h postgres -U courses coursedb | gzip > /backup/postgres_$(date +%Y%m%d).sql.gz
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

### Disaster Recovery Plan

**Recovery Steps:**
1. **Infrastructure Recovery**
   ```bash
   # Restore Kubernetes manifests
   kubectl apply -f k8s/

   # Wait for services to be ready
   kubectl wait --for=condition=available --timeout=300s deployment/docs-service
   ```

2. **Data Recovery**
   ```bash
   # Restore PostgreSQL
   psql -h postgres -U courses coursedb < backup.sql

   # Restore Redis
   redis-cli --rdb backup.rdb
   ```

3. **Verification**
   ```bash
   # Health checks
   curl http://course.example.com/health

   # Functionality tests
   python -m pytest tests/integration/
   ```

## Security Considerations

### Network Security

**Firewall Configuration:**
```bash
# Allow only necessary ports
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw allow 22/tcp    # SSH (restricted IPs)
ufw enable
```

**Docker Security:**
```yaml
# docker-compose.security.yml
services:
  docs:
    security_opt:
      - no-new-privileges:true
    user: "1000:1000"  # Non-root user
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
```

### Application Security

**Environment Variables:**
```bash
# Use secrets for sensitive data
echo "DATABASE_PASSWORD=supersecretpassword" >> .env
echo "SECRET_KEY=$(openssl rand -hex 32)" >> .env

# Set file permissions
chmod 600 .env
```

**Container Scanning:**
```bash
# Scan for vulnerabilities
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image celery-course/docs:latest
```

## Performance Optimization

### Caching Strategy

**Redis Cache Configuration:**
```python
# cache.py
import redis
from functools import wraps

redis_client = redis.Redis(host='redis', port=6379, db=0)

def cache_result(expiration=300):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"

            # Try to get from cache
            cached = redis_client.get(cache_key)
            if cached:
                return pickle.loads(cached)

            # Execute function and cache result
            result = func(*args, **kwargs)
            redis_client.setex(cache_key, expiration, pickle.dumps(result))

            return result
        return wrapper
    return decorator
```

### CDN Integration

**CloudFront Configuration:**
```yaml
# cloudfront.yaml
Resources:
  CourseCDN:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
          - Id: S3-course-bucket
            DomainName: !GetAtt CourseBucket.DomainName
            S3OriginConfig:
              OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${OriginAccessIdentity}'
        DefaultCacheBehavior:
          TargetOriginId: S3-course-bucket
          ViewerProtocolPolicy: redirect-to-https
          CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad  # Managed-CachingOptimized
```

## Maintenance and Updates

### Rolling Updates

**Kubernetes Rolling Update:**
```yaml
# k8s/rolling-update.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docs-service
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

**Docker Compose Rolling Update:**
```bash
# Zero-downtime update
docker-compose pull docs
docker-compose up -d --no-deps docs
docker-compose exec docs celery -A app inspect ping
```

### Health Checks

**Application Health Endpoint:**
```python
# health.py
from flask import Flask, jsonify
import redis
import psycopg2

app = Flask(__name__)

@app.route('/health')
def health_check():
    status = {
        'status': 'healthy',
        'checks': {}
    }

    # Check database
    try:
        conn = psycopg2.connect(DATABASE_URL)
        conn.close()
        status['checks']['database'] = 'healthy'
    except Exception as e:
        status['checks']['database'] = f'unhealthy: {str(e)}'
        status['status'] = 'unhealthy'

    # Check Redis
    try:
        redis_client.ping()
        status['checks']['redis'] = 'healthy'
    except Exception as e:
        status['checks']['redis'] = f'unhealthy: {str(e)}'
        status['status'] = 'unhealthy'

    return jsonify(status), 200 if status['status'] == 'healthy' else 503
```

## Troubleshooting

### Common Issues

1. **Container Won't Start**
   ```bash
   # Check logs
   docker-compose logs docs

   # Check resource usage
   docker stats

   # Verify configuration
   docker-compose config
   ```

2. **Performance Issues**
   ```bash
   # Monitor resource usage
   top
   iotop

   # Check network latency
   ping redis-service
   ```

3. **Database Connection Issues**
   ```bash
   # Test database connectivity
   psql -h postgres -U courses -d coursedb

   # Check connection pool
   SELECT * FROM pg_stat_activity;
   ```

### Debug Mode

**Development Debugging:**
```yaml
# docker-compose.debug.yml
services:
  docs:
    environment:
      - FLASK_DEBUG=1
      - CELERY_DEBUG=1
    ports:
      - "5678:5678"  # Debugpy port
    command: python -m debugpy --listen 0.0.0.0:5678 --wait-for-client app.py
```

This deployment guide provides comprehensive instructions for deploying the Celery Course Development Platform in various environments, with detailed configurations for security, monitoring, and maintenance.