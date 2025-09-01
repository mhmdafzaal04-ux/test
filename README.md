# Huly Self-Hosted - GitHub Deployment

🚀 **Ready-to-deploy Huly self-hosted platform optimized for Coolify**

This repository contains everything needed to deploy Huly on your infrastructure using Coolify as the deployment platform.

## 🎯 Quick Start

### Prerequisites
- OVHcloud VPS (4GB RAM minimum, 2+ vCPUs)
- Coolify installed on your server
- GitHub repository (this one!)
- Domain name (optional, can start with IP)

### 🚀 Deploy with Coolify

1. **Create New Resource in Coolify**
   - Go to your Coolify dashboard
   - Create new project: "Huly Production"
   - Add new resource → "Public Repository"
   - Repository URL: `https://github.com/your-username/your-repo`

2. **Configure Main Service**
   - Set **Main Service**: `front`
   - Set **Port**: `8080`
   - Enable **Health Checks**

3. **Set Environment Variables**
   Copy the variables from `.env.example` and set these **critical secrets** in Coolify:

   ```bash
   # 🚨 Generate these secure values (NEVER commit to Git!)
   CR_USER_PASSWORD=your_secure_database_password
   REDPANDA_ADMIN_PWD=your_secure_redpanda_password  
   SECRET=your_secure_32_character_secret
   CR_DB_URL=postgres://huly_prod:your_secure_database_password@cockroach:26257/defaultdb
   
   # 🌐 Domain configuration
   HOST_ADDRESS=your-vps-ip-or-domain
   SECURE=     # Leave empty initially, set to 's' when using HTTPS
   
   # 📱 App settings
   HULY_VERSION=s0.7.204
   TITLE=Huly Self-Hosted
   DEFAULT_LANGUAGE=en
   ```

4. **Deploy**
   - Click "Deploy" in Coolify
   - Wait 3-5 minutes for all services to start
   - Monitor deployment logs

5. **Access Your Instance**
   - Open: `http://your-vps-ip` 
   - Create your first admin account
   - Set up your workspace

## 📋 Environment Variables

### 🚨 Required (Set in Coolify UI)
```bash
CR_USER_PASSWORD=    # Database password - Generate with: openssl rand -hex 32
REDPANDA_ADMIN_PWD=  # Message queue password - Generate with: openssl rand -hex 32
SECRET=              # App secret - Generate with: openssl rand -hex 32
HOST_ADDRESS=        # Your domain or VPS IP
```

### 📧 Optional: Email Configuration
```bash
# SMTP (Gmail, Outlook, etc.)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com  
SMTP_PASSWORD=your-app-password
MAIL_FROM=noreply@yourdomain.com

# OR Amazon SES
SES_ACCESS_KEY=your-ses-key
SES_SECRET_KEY=your-ses-secret
SES_REGION=us-east-1
```

### 🔔 Optional: Push Notifications  
```bash
PUSH_PUBLIC_KEY=your-vapid-public-key
PUSH_PRIVATE_KEY=your-vapid-private-key
```

## 🌐 Domain Setup (When Ready)

1. **Point DNS to your VPS**
   - Create A record: `huly.yourdomain.com` → `your-vps-ip`

2. **Update Coolify Configuration**
   - Change `HOST_ADDRESS` to your domain
   - Set `SECURE=s` for HTTPS  
   - Enable SSL in Coolify (Let's Encrypt)
   - Redeploy

## 📊 Architecture

This deployment includes:

- **CockroachDB** - Primary database
- **Redpanda** - Message queue  
- **Elasticsearch** - Search engine
- **MinIO** - File storage
- **Front** - Web interface (main service)
- **Account** - Authentication service
- **Transactor** - Core business logic
- **Collaborator** - Real-time collaboration
- **Fulltext** - Search indexing
- **Stats** - Analytics
- **Datalake** - File management

## 🔧 Resource Requirements

### Minimum (Testing)
- **RAM**: 4GB
- **CPU**: 2 vCPUs
- **Storage**: 50GB SSD

### Recommended (Production)
- **RAM**: 8GB+
- **CPU**: 4 vCPUs+  
- **Storage**: 100GB+ SSD

## 🛡️ Security

- All secrets managed via Coolify environment variables
- No sensitive data in Git repository
- HTTPS enforced in production (via Coolify)
- Regular security updates via `HULY_VERSION`

## 📝 Optional Features

### 🤖 AI Assistant (OpenAI)
```bash
OPENAI_API_KEY=your-openai-key
AI_URL=http://aibot:4010
```

### 📹 Video Calls (LiveKit)
```bash  
LIVEKIT_HOST=wss://your-livekit-server
LIVEKIT_API_KEY=your-livekit-key
LIVEKIT_API_SECRET=your-livekit-secret
```

### 🔐 OAuth (GitHub/OIDC)
```bash
GITHUB_CLIENT_ID=your-github-id
GITHUB_CLIENT_SECRET=your-github-secret
```

## 🆘 Troubleshooting

### Services Won't Start
1. Check Coolify deployment logs
2. Verify all required environment variables are set
3. Ensure your server has enough resources (4GB+ RAM)
4. Check that ports are available

### Database Connection Issues
1. Verify `CR_USER_PASSWORD` matches in both `CR_USER_PASSWORD` and `CR_DB_URL`
2. Check if CockroachDB service is healthy in Coolify
3. Restart the entire stack if needed

### Web Interface Not Loading
1. Check if `front` service is running on port 8080
2. Verify `HOST_ADDRESS` is correct
3. Check Coolify reverse proxy configuration

## 📚 Documentation

- **Complete Setup Guide**: See `docs/DEPLOYMENT.md`
- **Security Checklist**: See `docs/SECURITY.md`  
- **Backup Strategy**: See `docs/BACKUP.md`

## 🔄 Updates

To update Huly:
1. Change `HULY_VERSION` in Coolify environment variables
2. Redeploy the stack
3. Monitor logs for successful startup

## 💬 Support

- **Issues**: Create GitHub issues for problems
- **Documentation**: [Huly Self-Hosted](https://github.com/hcengineering/huly-selfhost)
- **Community**: Huly Discord server

## 📜 License

This deployment configuration is provided as-is. Huly itself follows its own licensing terms.

---

**🎉 Happy collaborating with Huly!**

Made for easy deployment on OVHcloud + Coolify ☁️