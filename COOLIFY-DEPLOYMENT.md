# Huly Self-Hosted Deployment on OVHcloud with Coolify

This guide walks you through deploying Huly on OVHcloud using Coolify for easy container orchestration and management.

## Prerequisites

### OVHcloud VPS Requirements
- **Minimum**: 2 vCPUs, 4GB RAM, 50GB SSD
- **Recommended**: 4 vCPUs, 8GB RAM, 100GB SSD
- **Operating System**: Ubuntu 22.04 LTS or Ubuntu 24.04 LTS

### Domain Configuration
- A domain name (e.g., `huly.yourdomain.com`)
- DNS A record pointing to your OVHcloud VPS IP address

### Coolify Installation
Coolify must be installed on your OVHcloud VPS. Follow the [official installation guide](https://coolify.io/docs/installation).

```bash
# Quick Coolify installation
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

## Step-by-Step Deployment

### 1. Prepare Your Local Configuration

First, ensure you've completed the local setup to understand the application structure:

```bash
# Clone and set up locally (if not done already)
git clone https://github.com/hcengineering/huly-selfhost.git
cd huly-selfhost

# Run local setup to understand the configuration
./setup.sh

# Generate VAPID keys for push notifications
npm install -g web-push
web-push generate-vapid-keys
```

### 2. Server Preparation

SSH into your OVHcloud VPS and prepare the system:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y curl wget git

# Ensure Docker is installed (required for Coolify)
sudo systemctl status docker

# Create data directories with proper permissions
sudo mkdir -p /var/huly/{elasticsearch,files,cockroach/{data,certs},redpanda}
sudo chown -R 1000:1000 /var/huly
```

### 3. Coolify Project Setup

1. **Access Coolify Dashboard**
   - Navigate to `https://your-vps-ip:8000`
   - Complete the initial setup if first time

2. **Create New Project**
   - Click "New Project"
   - Name: `huly-production`
   - Description: `Huly Self-Hosted Platform`

3. **Add New Resource**
   - Select "Docker Compose"
   - Choose your project
   - Name: `huly-stack`

### 4. Docker Compose Configuration

Upload the production compose file to Coolify:

**File: `compose-production.yml`** (use the generated file from local setup)

Key modifications for Coolify:
- Remove the nginx service (Coolify handles reverse proxy)
- Expose the front service on port 8080
- Use environment variables for all configuration

### 5. Environment Variables Configuration

In Coolify, configure the following environment variables:

#### Core Configuration
```bash
HULY_VERSION=s0.7.204
DOCKER_NAME=huly_production
HOST_ADDRESS=huly.yourdomain.com
SECURE=true
TITLE=Huly Self-Hosted
DEFAULT_LANGUAGE=en
LAST_NAME_FIRST=true
```

#### Database Configuration
```bash
CR_DATABASE=defaultdb
CR_USERNAME=huly_prod
CR_USER_PASSWORD=your_secure_db_password_here
CR_DB_URL=postgres://huly_prod:your_secure_db_password_here@cockroach:26257/defaultdb
```

#### Message Queue Configuration
```bash
REDPANDA_ADMIN_USER=admin
REDPANDA_ADMIN_PWD=your_secure_redpanda_password_here
```

#### Storage Paths
```bash
VOLUME_ELASTIC_PATH=/var/huly/elasticsearch
VOLUME_FILES_PATH=/var/huly/files  
VOLUME_CR_DATA_PATH=/var/huly/cockroach/data
VOLUME_CR_CERTS_PATH=/var/huly/cockroach/certs
VOLUME_REDPANDA_PATH=/var/huly/redpanda
```

#### Security
```bash
SECRET=your_secure_32_character_secret_here
```

#### Optional: Email Configuration (SMTP)
```bash
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
MAIL_FROM=noreply@yourdomain.com
```

#### Optional: Push Notifications
```bash
PUSH_PUBLIC_KEY=your_vapid_public_key_here
PUSH_PRIVATE_KEY=your_vapid_private_key_here
```

### 6. Coolify-Specific Compose File

Create a simplified compose file for Coolify (without nginx):

```yaml
name: huly_production
services:
  cockroach:
    image: cockroachdb/cockroach:latest-v24.2
    command: start-single-node --accept-sql-without-tls
    environment:
      - COCKROACH_DATABASE=${CR_DATABASE}
      - COCKROACH_USER=${CR_USERNAME}
      - COCKROACH_PASSWORD=${CR_USER_PASSWORD}
    volumes:
      - cockroach_data:/cockroach/cockroach-data
      - cockroach_certs:/cockroach/certs
    restart: unless-stopped

  redpanda:
    image: docker.redpanda.com/redpandadata/redpanda:v24.3.6
    command:
      - redpanda
      - start
      - --kafka-addr internal://0.0.0.0:9092,external://0.0.0.0:19092
      - --advertise-kafka-addr internal://redpanda:9092,external://localhost:19092
      - --pandaproxy-addr internal://0.0.0.0:8082,external://0.0.0.0:18082
      - --advertise-pandaproxy-addr internal://redpanda:8082,external://localhost:18082
      - --schema-registry-addr internal://0.0.0.0:8081,external://0.0.0.0:18081
      - --rpc-addr redpanda:33145
      - --advertise-rpc-addr redpanda:33145
      - --mode dev-container
      - --smp 1
      - --default-log-level=info
    volumes:
      - redpanda_data:/var/lib/redpanda/data
    environment:
      - REDPANDA_SUPERUSER_USERNAME=${REDPANDA_ADMIN_USER}
      - REDPANDA_SUPERUSER_PASSWORD=${REDPANDA_ADMIN_PWD}
    healthcheck:
      test: ['CMD', 'rpk', 'cluster', 'info', '-X', 'user=${REDPANDA_ADMIN_USER}', '-X', 'pass=${REDPANDA_ADMIN_PWD}']
      interval: 10s
      timeout: 5s
      retries: 10

  minio:
    image: minio/minio
    command: server /data --address ":9000" --console-address ":9001"
    volumes:
      - minio_data:/data
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    healthcheck:
      test: ['CMD', 'mc', 'ready', 'local']
      interval: 5s
      retries: 10
    restart: unless-stopped

  elastic:
    image: elasticsearch:7.14.2
    command: |
      /bin/sh -c "./bin/elasticsearch-plugin list | grep -q ingest-attachment || yes | ./bin/elasticsearch-plugin install --silent ingest-attachment;
      /usr/local/bin/docker-entrypoint.sh eswrapper"
    volumes:
      - elastic_data:/usr/share/elasticsearch/data
    environment:
      - ELASTICSEARCH_PORT_NUMBER=9200
      - BITNAMI_DEBUG=false
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms2048m -Xmx2048m
      - http.cors.enabled=true
      - http.cors.allow-origin=http://localhost:8082
    healthcheck:
      interval: 20s
      retries: 10
      test: curl -s http://localhost:9200/_cluster/health | grep -vq '"status":"red"'
    restart: unless-stopped

  rekoni:
    image: hardcoreeng/rekoni-service:${HULY_VERSION}
    environment:
      - SECRET=${SECRET}
    deploy:
      resources:
        limits:
          memory: 1G
    restart: unless-stopped

  stats:
    image: hardcoreeng/stats:${HULY_VERSION}
    environment:
      - PORT=4900
      - SERVER_SECRET=${SECRET}
    restart: unless-stopped

  account:
    image: hardcoreeng/account:${HULY_VERSION}
    environment:
      - SERVER_PORT=3000
      - SERVER_SECRET=${SECRET}
      - DB_URL=${CR_DB_URL}
      - TRANSACTOR_URL=ws://transactor:3333;wss://${HOST_ADDRESS}/_transactor
      - STORAGE_CONFIG=datalake|http://datalake:4030
      - FRONT_URL=https://${HOST_ADDRESS}
      - STATS_URL=https://${HOST_ADDRESS}/_stats
      - MODEL_ENABLED=*
      - ACCOUNTS_URL=https://${HOST_ADDRESS}/_accounts
      - ACCOUNT_PORT=3000
      - QUEUE_CONFIG=redpanda:9092
    restart: unless-stopped

  transactor:
    image: hardcoreeng/transactor:${HULY_VERSION}
    environment:
      - SERVER_PORT=3333
      - SERVER_SECRET=${SECRET}
      - DB_URL=${CR_DB_URL}
      - STORAGE_CONFIG=datalake|http://datalake:4030
      - FRONT_URL=https://${HOST_ADDRESS}
      - ACCOUNTS_URL=http://account:3000
      - FULLTEXT_URL=http://fulltext:4700
      - STATS_URL=http://stats:4900
      - LAST_NAME_FIRST=${LAST_NAME_FIRST:-true}
      - QUEUE_CONFIG=redpanda:9092
    restart: unless-stopped

  collaborator:
    image: hardcoreeng/collaborator:${HULY_VERSION}
    environment:
      - COLLABORATOR_PORT=3078
      - SECRET=${SECRET}
      - ACCOUNTS_URL=http://account:3000
      - STATS_URL=http://stats:4900
      - STORAGE_CONFIG=datalake|http://datalake:4030
    restart: unless-stopped

  workspace:
    image: hardcoreeng/workspace:${HULY_VERSION}
    environment:
      - SERVER_SECRET=${SECRET}
      - DB_URL=${CR_DB_URL}
      - TRANSACTOR_URL=ws://transactor:3333;wss://${HOST_ADDRESS}/_transactor
      - STORAGE_CONFIG=datalake|http://datalake:4030
      - MODEL_ENABLED=*
      - ACCOUNTS_URL=http://account:3000
      - STATS_URL=http://stats:4900
      - QUEUE_CONFIG=redpanda:9092
    restart: unless-stopped

  fulltext:
    image: hardcoreeng/fulltext:${HULY_VERSION}
    environment:
      - SERVER_SECRET=${SECRET}
      - DB_URL=${CR_DB_URL}
      - FULLTEXT_DB_URL=http://elastic:9200
      - ELASTIC_INDEX_NAME=huly_storage_index
      - STORAGE_CONFIG=datalake|http://datalake:4030
      - REKONI_URL=http://rekoni:4004
      - ACCOUNTS_URL=http://account:3000
      - STATS_URL=http://stats:4900
      - QUEUE_CONFIG=redpanda:9092
    restart: unless-stopped

  datalake:
    image: hardcoreeng/datalake:${HULY_VERSION}
    depends_on:
      - minio
      - cockroach
      - stats
      - account
    environment:
      - PORT=4030
      - SECRET=${SECRET}
      - ACCOUNTS_URL=http://account:3000
      - STATS_URL=http://stats:4900
      - DB_URL=${CR_DB_URL}
      - BUCKETS=huly,eu|http://minio:9000?accessKey=minioadmin&secretKey=minioadmin
      - QUEUE_CONFIG=redpanda:9092
    restart: unless-stopped

  front:
    image: hardcoreeng/front:${HULY_VERSION}
    ports:
      - "8080:8080"  # Coolify will handle the reverse proxy
    environment:
      - SERVER_PORT=8080
      - SERVER_SECRET=${SECRET}
      - ACCOUNTS_URL=https://${HOST_ADDRESS}/_accounts
      - ACCOUNTS_URL_INTERNAL=http://account:3000
      - REKONI_URL=https://${HOST_ADDRESS}/_rekoni
      - STATS_URL=https://${HOST_ADDRESS}/_stats
      - FILES_URL=https://${HOST_ADDRESS}/_datalake/blob/:workspace/:blobId/:filename
      - UPLOAD_URL=https://${HOST_ADDRESS}/_datalake/upload/form-data/:workspace
      - PREVIEW_CONFIG=image|https://${HOST_ADDRESS}/_datalake/image/fit=cover,width=:width,height=:height,dpr=:dpr/:workspace/:blobId;video|https://${HOST_ADDRESS}/_datalake/meta/:workspace/:blobId
      - ELASTIC_URL=http://elastic:9200
      - COLLABORATOR_URL=wss://${HOST_ADDRESS}/_collaborator
      - STORAGE_CONFIG=datalake|http://datalake:4030
      - TITLE=${TITLE:-Huly Self-Hosted}
      - DEFAULT_LANGUAGE=${DEFAULT_LANGUAGE:-en}
      - LAST_NAME_FIRST=${LAST_NAME_FIRST:-true}
      - DESKTOP_UPDATES_CHANNEL=${HULY_VERSION}
    restart: unless-stopped

volumes:
  elastic_data:
  minio_data:
  cockroach_data:
  cockroach_certs:
  redpanda_data:
```

### 7. Coolify Service Configuration

1. **Set Main Service**
   - In the resource settings, set `front` as the main service
   - Port: `8080`

2. **Configure Domain**
   - Add your domain: `huly.yourdomain.com`
   - Enable SSL (Let's Encrypt)
   - Coolify will automatically handle HTTPS

3. **Configure Reverse Proxy Routes**
   Add these custom routes in Coolify's proxy settings:

   ```
   /_accounts/* -> account:3000/*
   /_transactor -> transactor:3333 (WebSocket)
   /_collaborator -> collaborator:3078 (WebSocket)
   /_datalake/* -> datalake:4030/*
   /_stats/* -> stats:4900/*
   /_rekoni/* -> rekoni:4004/*
   ```

### 8. Deployment

1. **Deploy the Stack**
   - Click "Deploy" in Coolify
   - Monitor the deployment logs
   - Wait for all services to be healthy (2-5 minutes)

2. **Verify Deployment**
   ```bash
   # Check service status
   curl -I https://huly.yourdomain.com
   
   # Should return HTTP 200 OK
   ```

### 9. Post-Deployment Configuration

#### Create First Admin User
1. Navigate to `https://huly.yourdomain.com`
2. Click "Sign Up" and create your admin account
3. Complete the workspace setup

#### Optional: Disable Public Signups
If you want invite-only access:
1. Add environment variable: `DISABLE_SIGNUP=true`
2. Redeploy the stack

#### Set Up Monitoring
- Use Coolify's built-in monitoring
- Set up alerts for service failures
- Monitor resource usage (CPU, Memory, Disk)

### 10. Backup Strategy

Set up automated backups for critical data:

```bash
# Create backup script
#!/bin/bash
BACKUP_DIR="/backup/huly/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# Backup database
docker exec huly_production-cockroach-1 \
  ./cockroach dump defaultdb --insecure > $BACKUP_DIR/database.sql

# Backup file storage
tar -czf $BACKUP_DIR/files.tar.gz -C /var/huly files/

# Backup configuration
cp /path/to/compose-production.yml $BACKUP_DIR/
cp /path/to/.env $BACKUP_DIR/

# Upload to OVHcloud Object Storage (optional)
# Use rclone or similar tools
```

## Troubleshooting

### Common Issues

1. **Services Won't Start**
   - Check resource limits (RAM/CPU)
   - Verify environment variables
   - Check Coolify deployment logs

2. **Database Connection Errors**
   - Ensure CockroachDB is healthy
   - Verify database credentials
   - Check network connectivity between services

3. **SSL Certificate Issues**
   - Verify domain DNS points to server
   - Check Coolify SSL settings
   - Ensure port 80/443 are accessible

4. **Performance Issues**
   - Increase server resources
   - Optimize Elasticsearch heap size
   - Monitor disk space usage

### Getting Help

- **Huly Documentation**: [GitHub Repository](https://github.com/hcengineering/huly-selfhost)
- **Coolify Documentation**: [coolify.io/docs](https://coolify.io/docs)
- **Community Support**: Huly Discord/GitHub Issues

## Security Recommendations

1. **Regular Updates**
   - Update Huly version regularly
   - Keep server OS updated
   - Update Coolify regularly

2. **Access Control**
   - Use strong passwords
   - Enable 2FA where possible
   - Limit SSH access

3. **Monitoring**
   - Set up log monitoring
   - Monitor for unusual activity
   - Regular security audits

4. **Backups**
   - Regular automated backups
   - Test backup restoration
   - Store backups securely

---

This deployment guide provides a production-ready Huly installation on OVHcloud using Coolify. For additional customization or enterprise features, refer to the official Huly documentation.