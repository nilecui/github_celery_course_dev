# 课程部署指南（中文版）

**最后更新：** 2025-11-15
**目的：** 指导在各种环境中部署 Celery 课程开发平台

---

## 概述

本指南涵盖了 Celery 课程开发平台在不同环境中的部署，从本地开发设置到生产部署。

## 部署选项

### 1. 本地开发设置

#### 先决条件
- 安装了 Docker 和 Docker Compose
- 用于版本控制的 Git
- 8GB+ RAM 可用
- 20GB+ 免费磁盘空间

#### 设置说明
```bash
# 克隆仓库
git clone <repository-url>
cd github_celery_course_dev

# 启动开发环境
docker-compose up -d

# 验证服务正在运行
docker-compose ps

# 访问课程平台
# 文档：http://localhost:7001
# RabbitMQ 管理：http://localhost:15672（guest/guest）
# Redis：localhost:6379
```

#### 环境配置
```bash
# 复制环境模板
cp .env.example .env

# 编辑配置
nano .env
```

### 2. 生产部署

#### 系统要求
- **CPU：** 4+ 核心
- **RAM：** 16GB+ 推荐
- **存储：** 100GB+ SSD
- **网络：** 可靠的互联网连接

#### Docker Compose 生产设置
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

#### SSL/TLS 配置
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

### 3. Kubernetes 部署

#### 先决条件
- Kubernetes 集群（v1.20+）
- 配置好的 kubectl
- Helm 3.0+

#### 命名空间和配置
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

#### 文档服务部署
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

### 4. 云平台部署

#### AWS ECS 部署

**任务定义：**
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

#### Google Cloud Run 部署

**Dockerfile：**
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 7001

CMD ["gunicorn", "--bind", "0.0.0.0:7001", "--workers", "4", "app:app"]
```

**部署命令：**
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

## 监控和日志

### Prometheus 监控

**配置：**
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

### Grafana 仪表板

**仪表板配置：**
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
      }
    ]
  }
}
```

## 备份和恢复

### 数据库备份

**自动化备份脚本：**
```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# PostgreSQL 备份
pg_dump -h postgres -U courses coursedb > $BACKUP_DIR/postgres_$DATE.sql

# Redis 备份
redis-cli --rdb $BACKUP_DIR/redis_$DATE.rdb

# 压缩备份
tar -czf $BACKUP_DIR/course_backup_$DATE.tar.gz \
  $BACKUP_DIR/postgres_$DATE.sql \
  $BACKUP_DIR/redis_$DATE.rdb

# 清理旧备份（保留7天）
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

### Kubernetes CronJob 用于备份
```yaml
# k8s/backup-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: course-backup
  namespace: celery-course
spec:
  schedule: "0 2 * * *"  # 每天凌晨2点
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

### 灾难恢复计划

**恢复步骤：**
1. **基础设施恢复**
   ```bash
   # 恢复 Kubernetes 清单
   kubectl apply -f k8s/

   # 等待服务就绪
   kubectl wait --for=condition=available --timeout=300s deployment/docs-service
   ```

2. **数据恢复**
   ```bash
   # 恢复 PostgreSQL
   psql -h postgres -U courses coursedb < backup.sql

   # 恢复 Redis
   redis-cli --rdb backup.rdb
   ```

## 安全考虑

### 网络安全

**防火墙配置：**
```bash
# 只允许必要的端口
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw allow 22/tcp    # SSH（限制IP）
ufw enable
```

**Docker 安全：**
```yaml
# docker-compose.security.yml
services:
  docs:
    security_opt:
      - no-new-privileges:true
    user: "1000:1000"  # 非 root 用户
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
```

### 应用安全

**环境变量：**
```bash
# 使用敏感数据的密钥
echo "DATABASE_PASSWORD=supersecretpassword" >> .env
echo "SECRET_KEY=$(openssl rand -hex 32)" >> .env

# 设置文件权限
chmod 600 .env
```

## 性能优化

### 缓存策略

**Redis 缓存配置：**
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

            # 尝试从缓存获取
            cached = redis_client.get(cache_key)
            if cached:
                return pickle.loads(cached)

            # 执行函数并缓存结果
            result = func(*args, **kwargs)
            redis_client.setex(cache_key, expiration, pickle.dumps(result))

            return result
        return wrapper
    return decorator
```

### CDN 集成

**CloudFront 配置：**
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

## 维护和更新

### 滚动更新

**Kubernetes 滚动更新：**
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

**Docker Compose 滚动更新：**
```bash
# 零停机更新
docker-compose pull docs
docker-compose up -d --no-deps docs
docker-compose exec docs celery -A app inspect ping
```

### 健康检查

**应用健康端点：**
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

    # 检查数据库
    try:
        conn = psycopg2.connect(DATABASE_URL)
        conn.close()
        status['checks']['database'] = 'healthy'
    except Exception as e:
        status['checks']['database'] = f'unhealthy: {str(e)}'
        status['status'] = 'unhealthy'

    # 检查 Redis
    try:
        redis_client.ping()
        status['checks']['redis'] = 'healthy'
    except Exception as e:
        status['checks']['redis'] = f'unhealthy: {str(e)}'
        status['status'] = 'unhealthy'

    return jsonify(status), 200 if status['status'] == 'healthy' else 503
```

## 故障排除

### 常见问题

1. **容器无法启动**
   ```bash
   # 检查日志
   docker-compose logs docs

   # 检查资源使用
   top
   iotop

   # 验证配置
   docker-compose config
   ```

2. **性能问题**
   ```bash
   # 监控资源使用
   top
   iotop

   # 检查网络延迟
   ping redis-service
   ```

3. **数据库连接问题**
   ```bash
   # 测试数据库连接
   psql -h postgres -U courses -d coursedb

   # 检查连接池
   SELECT * FROM pg_stat_activity;
   ```

### 调试模式

**开发调试：**
```yaml
# docker-compose.debug.yml
services:
  docs:
    environment:
      - FLASK_DEBUG=1
      - CELERY_DEBUG=1
    ports:
      - "5678:5678"  # Debugpy 端口
    command: python -m debugpy --listen 0.0.0.0:5678 --wait-for-client app.py
```

这个部署指南为在各种环境中部署 Celery 课程开发平台提供了全面的说明，并包含了详细的安全、监控、维护和故障排除指导。