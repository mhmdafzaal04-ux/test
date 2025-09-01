# 🚀 Step-by-Step Coolify Deployment Guide

This guide walks you through deploying Huly on your OVHcloud VPS using Coolify with your GitHub repository.

## 📋 Before You Start

### ✅ What You Need
1. **OVHcloud VPS** - Running with Coolify installed
2. **GitHub Repository** - With the files from this setup
3. **VPS IP Address** - For initial access (domain can come later)
4. **Coolify Access** - Dashboard accessible at `https://your-vps-ip:8000`

### 🔐 Your Security Values (Keep These Safe!)
```bash
# Generated secure passwords (use these in Coolify):
CR_USER_PASSWORD=b4d641db28c6a5ba341da82b24731a6e229d5a018a9b845bb52c984e01c7f404
REDPANDA_ADMIN_PWD=dcc36a8f7122c1274920632ad7364d6c41dc09261084e2f1a83d4e8a1bbab0c0
SECRET=891b52380bee25ea3deae97239ca4e0e57936ab56e4521f8867e24ab0d779502

# Database URL (auto-constructed):
CR_DB_URL=postgres://huly_prod:b4d641db28c6a5ba341da82b24731a6e229d5a018a9b845bb52c984e01c7f404@cockroach:26257/defaultdb

# VAPID Keys (for push notifications):
PUSH_PUBLIC_KEY=BLH4UvnfmZr1huRSBzeR4jC0UMVSwYPeJ-yBjzIaMWJCntMccbVdIeDGEnj4ChxUnkgQD4dnphvEO4c6w-QMHUM
PUSH_PRIVATE_KEY=l1v9zSEO9Beps1gjbE4ShhJA8n0QfOO1M7sEp2OZvWU
```

## 🎯 Step 1: Push to Your GitHub Repository

First, let's get your code ready in GitHub:

```bash
# Navigate to your project
cd /Users/Muhammad/Huly/huly-selfhost

# Initialize new git repository (if needed)
git init

# Add your GitHub remote
git remote remove origin  # Remove old remote if exists
git remote add origin https://github.com/your-username/your-test-repo.git

# Add all files (SECURITY-VALUES.txt will be ignored by .gitignore)
git add .

# Commit
git commit -m "Initial Huly self-hosted setup for Coolify deployment

- Docker Compose optimized for Coolify
- Environment variables template  
- Security-focused .gitignore
- Complete deployment documentation
- Ready for production deployment"

# Push to your repository
git push -u origin main
```

## 🌐 Step 2: Configure Coolify

### 2.1 Create New Project
1. Open Coolify dashboard: `https://your-vps-ip:8000`
2. Click **"New Project"**
3. **Project Name**: `Huly Production`
4. **Description**: `Huly Self-Hosted Platform`
5. Click **"Continue"**

### 2.2 Add GitHub Resource
1. Click **"New Resource"**
2. Select **"Public Repository"**
3. **Repository URL**: `https://github.com/your-username/your-test-repo`
4. **Branch**: `main`
5. **Resource Name**: `huly-stack`
6. Click **"Continue"**

### 2.3 Configure Service Settings
1. **Main Service**: Select `front`
2. **Port**: `8080`
3. **Health Check**: Enable with path `/` 
4. **Domains**: Add your VPS IP (e.g., `123.456.789.123`)

## ⚙️ Step 3: Set Environment Variables

In Coolify, go to your resource → **Environment Variables** tab and add:

### 🚨 Critical Secrets (Required)
```bash
# Database Configuration
CR_USER_PASSWORD=b4d641db28c6a5ba341da82b24731a6e229d5a018a9b845bb52c984e01c7f404
CR_DB_URL=postgres://huly_prod:b4d641db28c6a5ba341da82b24731a6e229d5a018a9b845bb52c984e01c7f404@cockroach:26257/defaultdb

# Message Queue
REDPANDA_ADMIN_PWD=dcc36a8f7122c1274920632ad7364d6c41dc09261084e2f1a83d4e8a1bbab0c0

# Application Security
SECRET=891b52380bee25ea3deae97239ca4e0e57936ab56e4521f8867e24ab0d779502
```

### 🌐 Domain Configuration
```bash
# Use your VPS IP initially (replace with actual IP)
HOST_ADDRESS=123.456.789.123

# Leave empty for HTTP initially (set to 's' when HTTPS is ready)
SECURE=
```

### 📱 Application Settings
```bash
HULY_VERSION=s0.7.204
TITLE=Huly Self-Hosted
DEFAULT_LANGUAGE=en
LAST_NAME_FIRST=true
CR_DATABASE=defaultdb
CR_USERNAME=huly_prod
REDPANDA_ADMIN_USER=admin
```

### 🔔 Optional: Push Notifications
```bash
PUSH_PUBLIC_KEY=BLH4UvnfmZr1huRSBzeR4jC0UMVSwYPeJ-yBjzIaMWJCntMccbVdIeDGEnj4ChxUnkgQD4dnphvEO4c6w-QMHUM
PUSH_PRIVATE_KEY=l1v9zSEO9Beps1gjbE4ShhJA8n0QfOO1M7sEp2OZvWU
WEB_PUSH_URL=http://notification:3335
```

## 🚀 Step 4: Deploy

1. **Click "Deploy"** in Coolify
2. **Monitor Logs**: Watch the deployment progress
3. **Wait**: Allow 3-5 minutes for all services to start
4. **Health Check**: Ensure all services show as healthy

## ✅ Step 5: Verify Deployment

### 5.1 Check Service Status
In Coolify, verify all services are running:
- ✅ `front` - Web interface
- ✅ `cockroach` - Database  
- ✅ `redpanda` - Message queue
- ✅ `elastic` - Search engine
- ✅ `minio` - File storage
- ✅ `account` - Authentication
- ✅ `transactor` - Core logic

### 5.2 Access Your Huly Instance
1. **Open**: `http://your-vps-ip`
2. **Create Admin Account**: Sign up for first account (becomes admin)
3. **Create Workspace**: Set up your first workspace
4. **Test Features**: Create a document, task, or test chat

## 🌍 Step 6: Add Domain (When Ready)

When you have domain access:

### 6.1 DNS Configuration
```bash
# Create A record in your DNS provider:
huly.yourdomain.com → your-vps-ip
```

### 6.2 Update Coolify
1. **Environment Variables**: Change `HOST_ADDRESS=huly.yourdomain.com`
2. **Environment Variables**: Set `SECURE=s`
3. **Domains**: Add your domain in Coolify
4. **SSL**: Enable Let's Encrypt SSL in Coolify
5. **Deploy**: Redeploy the stack

## 🆘 Troubleshooting

### Services Won't Start
```bash
# Check these in Coolify logs:
1. Resource allocation (need 4GB+ RAM)
2. All environment variables are set
3. No port conflicts
4. Docker network is healthy
```

### Database Connection Errors
```bash
# Verify:
1. CR_USER_PASSWORD matches in both variables
2. CockroachDB service is running
3. No special characters causing URL parsing issues
```

### Web Interface Not Loading
```bash
# Check:
1. Front service is running on port 8080
2. Coolify reverse proxy is configured
3. HOST_ADDRESS is correct
4. No firewall blocking access
```

## 🔧 Advanced Configuration

### Email Integration
```bash
# Add to environment variables:
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
MAIL_FROM=noreply@yourdomain.com
MAIL_URL=http://mail:8097
```

### Security Enhancements
```bash
# Invite-only mode:
DISABLE_SIGNUP=true

# OAuth integration:
GITHUB_CLIENT_ID=your-client-id
GITHUB_CLIENT_SECRET=your-client-secret
```

## 📊 Monitoring

### Resource Usage
Monitor in Coolify:
- **CPU**: Should be under 80%
- **RAM**: 4GB minimum, 6-8GB recommended
- **Disk**: Monitor growth, plan for backups

### Health Checks
Services should respond:
- **Front**: `http://your-ip/` → 200 OK
- **Account**: Healthy in logs
- **Database**: Connection successful

## 🔄 Updates

To update Huly:
1. Change `HULY_VERSION` in environment variables
2. Click "Deploy" in Coolify  
3. Monitor deployment logs
4. Test functionality after update

---

## 🎉 Success!

Your Huly instance should now be running! 

**Next Steps:**
1. Create your admin account
2. Set up your first workspace  
3. Invite team members
4. Configure optional features (email, push notifications, etc.)
5. Set up regular backups
6. Monitor resource usage

**Support:**
- Check logs in Coolify for issues
- Refer to the main README.md for additional configuration
- Create GitHub issues for deployment problems

**Enjoy your self-hosted Huly platform! 🚀**