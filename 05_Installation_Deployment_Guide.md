# Installation & Deployment Guide
# Accurate Enterprise 5 - Clone System

## Document Information
- **Project Name**: Accurate Enterprise 5 Clone
- **Document Version**: 1.0
- **Date**: February 2026

---

## 1. SYSTEM REQUIREMENTS

### 1.1 Hardware Requirements

#### Development Environment
- **CPU**: Intel Core i5 or equivalent (4 cores minimum)
- **RAM**: 8 GB minimum, 16 GB recommended
- **Storage**: 50 GB SSD
- **Network**: Broadband internet connection

#### Production Environment

**Application Server** (per instance)
- **CPU**: 8 cores minimum
- **RAM**: 16 GB minimum, 32 GB recommended
- **Storage**: 100 GB SSD
- **Network**: 1 Gbps

**Database Server** (Primary)
- **CPU**: 16 cores minimum
- **RAM**: 64 GB minimum
- **Storage**: 1 TB SSD (RAID 10)
- **Network**: 10 Gbps

**Load Balancer**
- **CPU**: 4 cores
- **RAM**: 8 GB
- **Storage**: 50 GB SSD
- **Network**: 10 Gbps

### 1.2 Software Requirements

#### Operating System
- **Recommended**: Ubuntu 22.04 LTS or Ubuntu 24.04 LTS
- **Alternative**: CentOS 8+, Debian 11+, RHEL 8+

#### Runtime & Dependencies
- **Node.js**: 20.x LTS
- **PostgreSQL**: 14.x or higher
- **Redis**: 7.x
- **Nginx**: 1.24.x
- **Docker**: 24.x (for containerized deployment)
- **Docker Compose**: 2.x

---

## 2. DEVELOPMENT SETUP

### 2.1 Prerequisites Installation

#### Install Node.js 20 LTS

```bash
# Using NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x
```

#### Install PostgreSQL 14+

```bash
# Add PostgreSQL repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Install PostgreSQL
sudo apt-get update
sudo apt-get install postgresql-14 postgresql-contrib-14

# Start and enable service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verify installation
sudo -u postgres psql --version
```

#### Install Redis

```bash
sudo apt-get install redis-server

# Start and enable service
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Verify installation
redis-cli --version
```

#### Install Git

```bash
sudo apt-get install git
git --version
```

### 2.2 Project Setup

#### Clone Repository

```bash
git clone https://github.com/yourcompany/accurate-erp.git
cd accurate-erp
```

#### Install Dependencies

```bash
# Backend dependencies
cd backend
npm install

# Frontend dependencies
cd ../frontend
npm install
```

#### Database Setup

```bash
# Create databases
sudo -u postgres psql << EOF
-- Create master database
CREATE DATABASE master_db;

-- Create default company database
CREATE DATABASE company_001_db;

-- Create database user
CREATE USER erp_user WITH PASSWORD 'secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE master_db TO erp_user;
GRANT ALL PRIVILEGES ON DATABASE company_001_db TO erp_user;
EOF
```

#### Run Migrations

```bash
cd backend

# Run master database migrations
npm run migrate:master

# Run company database migrations
npm run migrate:company -- --company=001
```

#### Seed Initial Data

```bash
# Seed master database with default data
npm run seed:master

# Seed company database with chart of accounts
npm run seed:company -- --company=001
```

### 2.3 Environment Configuration

#### Create `.env` file for Backend

```bash
cd backend
cp .env.example .env
```

**.env** file content:

```env
# Application
NODE_ENV=development
PORT=3000
APP_NAME="Accurate ERP"
APP_URL=http://localhost:3000

# Master Database
MASTER_DB_HOST=localhost
MASTER_DB_PORT=5432
MASTER_DB_NAME=master_db
MASTER_DB_USER=erp_user
MASTER_DB_PASSWORD=secure_password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT
JWT_SECRET=your_jwt_secret_key_change_this_in_production
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# File Storage
STORAGE_TYPE=local
STORAGE_PATH=./uploads

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password
SMTP_FROM="ERP System <noreply@yourdomain.com>"

# Logging
LOG_LEVEL=debug
```

#### Create `.env` file for Frontend

```bash
cd frontend
cp .env.example .env
```

**.env** file content:

```env
REACT_APP_API_URL=http://localhost:3000/api/v1
REACT_APP_NAME="Accurate ERP"
```

### 2.4 Running Development Server

#### Start Backend Server

```bash
cd backend
npm run dev
```

Server will start on `http://localhost:3000`

#### Start Frontend Development Server

```bash
cd frontend
npm start
```

Frontend will start on `http://localhost:3001` and proxy API requests to backend

### 2.5 Running Tests

```bash
# Backend unit tests
cd backend
npm test

# Backend integration tests
npm run test:integration

# Backend end-to-end tests
npm run test:e2e

# Frontend tests
cd frontend
npm test
```

---

## 3. PRODUCTION DEPLOYMENT

### 3.1 Server Preparation

#### Update System

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

#### Install Required Software

```bash
# Install Node.js, PostgreSQL, Redis, Nginx (as shown in section 2.1)

# Install additional tools
sudo apt-get install -y \
    build-essential \
    git \
    curl \
    wget \
    ufw \
    fail2ban \
    certbot \
    python3-certbot-nginx
```

#### Configure Firewall

```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow ssh

# Allow HTTP and HTTPS
sudo ufw allow 'Nginx Full'

# Check status
sudo ufw status
```

### 3.2 Database Setup for Production

#### Secure PostgreSQL

```bash
# Edit pg_hba.conf
sudo nano /etc/postgresql/14/main/pg_hba.conf

# Change peer to md5 for local connections
# local   all             all                                     md5

# Restart PostgreSQL
sudo systemctl restart postgresql
```

#### Create Production Databases

```bash
sudo -u postgres psql << EOF
-- Create production databases
CREATE DATABASE prod_master_db;
CREATE DATABASE prod_company_001_db;

-- Create production user
CREATE USER prod_erp_user WITH PASSWORD 'very_secure_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE prod_master_db TO prod_erp_user;
GRANT ALL PRIVILEGES ON DATABASE prod_company_001_db TO prod_erp_user;

-- Configure connection limits
ALTER USER prod_erp_user CONNECTION LIMIT 50;
EOF
```

#### Optimize PostgreSQL for Production

Edit `/etc/postgresql/14/main/postgresql.conf`:

```ini
# Memory
shared_buffers = 8GB
effective_cache_size = 24GB
maintenance_work_mem = 2GB
work_mem = 64MB

# Connections
max_connections = 200

# WAL
wal_buffers = 16MB
checkpoint_completion_target = 0.9
wal_level = replica

# Query Planner
random_page_cost = 1.1
effective_io_concurrency = 200

# Logging
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_line_prefix = '%m [%p] %q%u@%d '
log_min_duration_statement = 1000
```

Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

### 3.3 Application Deployment

#### Clone Repository

```bash
cd /opt
sudo git clone https://github.com/yourcompany/accurate-erp.git
sudo chown -R $USER:$USER accurate-erp
cd accurate-erp
```

#### Install Dependencies (Production)

```bash
# Backend
cd backend
npm ci --production

# Frontend
cd ../frontend
npm ci
npm run build
```

#### Configure Environment

```bash
cd /opt/accurate-erp/backend
cp .env.production .env
nano .env
```

**Production .env**:

```env
NODE_ENV=production
PORT=3000

# Master Database
MASTER_DB_HOST=localhost
MASTER_DB_PORT=5432
MASTER_DB_NAME=prod_master_db
MASTER_DB_USER=prod_erp_user
MASTER_DB_PASSWORD=very_secure_password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=redis_secure_password

# JWT
JWT_SECRET=production_jwt_secret_change_this_strong_random_value
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# File Storage (MinIO or S3)
STORAGE_TYPE=minio
MINIO_ENDPOINT=minio.yourdomain.com
MINIO_PORT=9000
MINIO_ACCESS_KEY=your_access_key
MINIO_SECRET_KEY=your_secret_key
MINIO_BUCKET=erp-files
MINIO_USE_SSL=true

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@yourdomain.com
SMTP_PASSWORD=your_app_password
SMTP_FROM="ERP System <noreply@yourdomain.com>"

# Logging
LOG_LEVEL=info
LOG_FILE_PATH=/var/log/accurate-erp/app.log
```

#### Run Migrations

```bash
npm run migrate:master
npm run migrate:company -- --company=001
```

### 3.4 Process Manager (PM2)

#### Install PM2

```bash
sudo npm install -g pm2
```

#### Create PM2 Ecosystem File

```bash
nano /opt/accurate-erp/ecosystem.config.js
```

**ecosystem.config.js**:

```javascript
module.exports = {
  apps: [
    {
      name: 'erp-api',
      cwd: '/opt/accurate-erp/backend',
      script: './src/server.js',
      instances: 4,
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production'
      },
      error_file: '/var/log/accurate-erp/err.log',
      out_file: '/var/log/accurate-erp/out.log',
      log_date_format: 'YYYY-MM-DD HH:mm Z',
      merge_logs: true,
      autorestart: true,
      max_memory_restart: '1G'
    }
  ]
};
```

#### Start Application with PM2

```bash
# Create log directory
sudo mkdir -p /var/log/accurate-erp
sudo chown -R $USER:$USER /var/log/accurate-erp

# Start application
cd /opt/accurate-erp
pm2 start ecosystem.config.js

# Save PM2 process list
pm2 save

# Setup PM2 to start on boot
pm2 startup systemd
# Run the command that PM2 outputs
```

#### PM2 Management Commands

```bash
# List all processes
pm2 list

# Monitor processes
pm2 monit

# View logs
pm2 logs erp-api

# Restart application
pm2 restart erp-api

# Reload (zero-downtime)
pm2 reload erp-api

# Stop application
pm2 stop erp-api

# Delete from PM2
pm2 delete erp-api
```

### 3.5 Nginx Configuration

#### Create Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/accurate-erp
```

**accurate-erp** configuration:

```nginx
upstream backend {
    least_conn;
    server localhost:3000;
    # Add more backend servers for load balancing
    # server backend2.yourdomain.com:3000;
    # server backend3.yourdomain.com:3000;
}

# Rate limiting
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;

server {
    listen 80;
    server_name erp.yourdomain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name erp.yourdomain.com;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/erp.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/erp.yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Root directory for frontend
    root /opt/accurate-erp/frontend/build;
    index index.html;

    # Logging
    access_log /var/log/nginx/erp-access.log;
    error_log /var/log/nginx/erp-error.log;

    # Max upload size
    client_max_body_size 50M;

    # API routes
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;

        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Login endpoint rate limiting
    location /api/v1/auth/login {
        limit_req zone=login_limit burst=3 nodelay;

        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Frontend routes
    location / {
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Health check endpoint
    location /health {
        access_log off;
        proxy_pass http://backend;
    }
}
```

#### Enable Site and Restart Nginx

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/accurate-erp /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

#### Setup SSL Certificate

```bash
# Obtain SSL certificate from Let's Encrypt
sudo certbot --nginx -d erp.yourdomain.com

# Auto-renewal is configured by default
# Test renewal
sudo certbot renew --dry-run
```

### 3.6 Redis Configuration for Production

```bash
sudo nano /etc/redis/redis.conf
```

**Important settings**:

```ini
# Bind to localhost only
bind 127.0.0.1

# Set password
requirepass your_redis_secure_password

# Max memory
maxmemory 2gb
maxmemory-policy allkeys-lru

# Persistence
save 900 1
save 300 10
save 60 10000

# AOF
appendonly yes
appendfsync everysec
```

Restart Redis:

```bash
sudo systemctl restart redis-server
```

---

## 4. DOCKER DEPLOYMENT

### 4.1 Dockerfile (Backend)

**backend/Dockerfile**:

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --production

# Copy source code
COPY . .

# Final stage
FROM node:20-alpine

WORKDIR /app

# Copy from builder
COPY --from=builder /app /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "src/server.js"]
```

### 4.2 Dockerfile (Frontend)

**frontend/Dockerfile**:

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage with Nginx
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 4.3 Docker Compose

**docker-compose.yml**:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14-alpine
    container_name: erp-postgres
    environment:
      POSTGRES_USER: erp_user
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: master_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    ports:
      - "5432:5432"
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U erp_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: erp-redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: erp-backend
    environment:
      NODE_ENV: production
      MASTER_DB_HOST: postgres
      MASTER_DB_PORT: 5432
      MASTER_DB_NAME: master_db
      MASTER_DB_USER: erp_user
      MASTER_DB_PASSWORD: ${POSTGRES_PASSWORD}
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_PASSWORD: ${REDIS_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    volumes:
      - ./uploads:/app/uploads
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: erp-frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    restart: unless-stopped

  minio:
    image: minio/minio:latest
    container_name: erp-minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - minio_data:/data
    ports:
      - "9000:9000"
      - "9001:9001"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

volumes:
  postgres_data:
  redis_data:
  minio_data:

networks:
  default:
    name: erp-network
```

### 4.4 Docker Environment File

**.env**:

```env
# PostgreSQL
POSTGRES_PASSWORD=secure_postgres_password

# Redis
REDIS_PASSWORD=secure_redis_password

# JWT
JWT_SECRET=secure_jwt_secret_key

# MinIO
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=secure_minio_password
```

### 4.5 Deploy with Docker Compose

```bash
# Build and start containers
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down

# Rebuild containers
docker-compose up -d --build

# Scale backend
docker-compose up -d --scale backend=3
```

---

## 5. MONITORING & MAINTENANCE

### 5.1 Application Monitoring

#### Install Monitoring Tools

```bash
# Prometheus
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Grafana
docker run -d \
  --name grafana \
  -p 3001:3000 \
  grafana/grafana
```

### 5.2 Database Backup Script

**backup-database.sh**:

```bash
#!/bin/bash

BACKUP_DIR="/backup/database"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup master database
pg_dump -U prod_erp_user -h localhost prod_master_db | gzip > $BACKUP_DIR/master_db_$DATE.sql.gz

# Backup company databases
pg_dump -U prod_erp_user -h localhost prod_company_001_db | gzip > $BACKUP_DIR/company_001_db_$DATE.sql.gz

# Delete old backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $DATE"
```

#### Schedule Backup with Cron

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /opt/scripts/backup-database.sh >> /var/log/backup.log 2>&1
```

### 5.3 Log Rotation

**/etc/logrotate.d/accurate-erp**:

```
/var/log/accurate-erp/*.log {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 nodejs nodejs
    sharedscripts
    postrotate
        pm2 reloadLogs
    endscript
}
```

---

## 6. TROUBLESHOOTING

### 6.1 Common Issues

#### Database Connection Error

```bash
# Check PostgreSQL is running
sudo systemctl status postgresql

# Check connection
psql -U prod_erp_user -h localhost -d prod_master_db

# Check pg_hba.conf settings
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

#### Application Not Starting

```bash
# Check PM2 logs
pm2 logs erp-api

# Check environment variables
pm2 env 0

# Restart application
pm2 restart erp-api
```

#### Nginx 502 Bad Gateway

```bash
# Check backend is running
pm2 list

# Check Nginx error log
sudo tail -f /var/log/nginx/erp-error.log

# Test Nginx configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

### 6.2 Performance Tuning

#### Database Optimization

```sql
-- Analyze all tables
ANALYZE;

-- Vacuum to reclaim space
VACUUM FULL;

-- Reindex
REINDEX DATABASE prod_master_db;

-- Check slow queries
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

#### Redis Memory Management

```bash
# Check memory usage
redis-cli info memory

# Clear cache if needed
redis-cli FLUSHALL
```

---

## 7. SECURITY CHECKLIST

- [ ] Change all default passwords
- [ ] Enable firewall (UFW)
- [ ] Configure fail2ban
- [ ] Install SSL certificate
- [ ] Disable root SSH login
- [ ] Setup SSH key authentication
- [ ] Configure database backups
- [ ] Enable audit logging
- [ ] Setup intrusion detection
- [ ] Regular security updates
- [ ] Monitor system logs
- [ ] Configure rate limiting
- [ ] Enable HTTPS only
- [ ] Implement CSP headers
- [ ] Regular vulnerability scans

---

## APPENDICES

### Appendix A: Server Sizing Guide
### Appendix B: Backup & Recovery Procedures
### Appendix C: Scaling Strategies
### Appendix D: Migration Guide

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Support**: support@yourdomain.com
