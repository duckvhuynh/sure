# Deploying Sure to Coolify

This guide provides step-by-step instructions for deploying Sure to Coolify, a self-hosted Heroku/Netlify alternative.

## Prerequisites

- A Coolify instance running (see [Coolify installation](https://coolify.io/docs/installation))
- Access to the Coolify dashboard
- A domain name (optional, but recommended)

## Quick Start

### Step 1: Create a New Service in Coolify

1. Log in to your Coolify dashboard
2. Navigate to your project or create a new one
3. Click **+ Add New Resource**
4. Select **Docker Compose - Empty Service**
5. Give it a name (e.g., "Sure Finance")

### Step 2: Configure Docker Compose

1. In the service configuration, navigate to the **Compose** tab
2. Copy the contents of [`compose.coolify.yml`](../compose.coolify.yml) from this repository
3. Paste it into the Coolify compose editor
4. Click **Save**

### Step 3: Set Environment Variables

Navigate to the **Environment Variables** section and add the following:

#### Required Variables

```bash
# Generate this with: openssl rand -hex 64
SECRET_KEY_BASE=your_generated_secret_key_here

# Database credentials
POSTGRES_USER=sure_user
POSTGRES_PASSWORD=your_secure_database_password_here
POSTGRES_DB=sure_production
```

#### Optional Variables

```bash
# OpenAI for AI features (incurs costs - set spend limits!)
OPENAI_ACCESS_TOKEN=sk-your-openai-api-key

# Or use a custom OpenAI-compatible endpoint (e.g., LM Studio, Ollama)
OPENAI_URI_BASE=https://your-llm-endpoint.com/v1
OPENAI_MODEL=gpt-4

# Market data API
TWELVE_DATA_API_KEY=your_twelve_data_api_key

# SimpleFIN bank sync
SIMPLEFIN_DEBUG_RAW=false
SIMPLEFIN_INCLUDE_PENDING=false

# Onboarding control (open, closed, invite_only)
ONBOARDING_STATE=open

# Port (Coolify usually handles this)
PORT=3000
```

### Step 4: Configure Domain (Optional)

1. Navigate to the **Domains** tab
2. Add your domain name (e.g., `sure.yourdomain.com`)
3. Coolify will automatically handle SSL certificates via Let's Encrypt
4. Update your DNS to point to your Coolify server:
   - **A Record**: `sure.yourdomain.com` → Your server IP
   - Or **CNAME**: `sure.yourdomain.com` → Your Coolify domain

### Step 5: Deploy

1. Click **Deploy** in the Coolify UI
2. Monitor the deployment logs
3. Wait for all services to become healthy (this may take 2-3 minutes on first deploy)

### Step 6: Create Your Account

1. Once deployed, navigate to your domain or `http://your-server-ip:3000`
2. Click **"Create your account"** on the login page
3. Enter your email and password
4. Start using Sure!

## Architecture Overview

The Coolify deployment includes:

- **Web Service**: Rails application server (port 3000)
- **Worker Service**: Sidekiq background job processor
- **Database**: PostgreSQL 16 (Alpine)
- **Cache**: Redis 7 (Alpine)

All services run on an internal Docker network for security.

## Persistence & Backups

### Volume Management

Coolify automatically manages these persistent volumes:

- `sure_storage`: Application file storage
- `sure_postgres_data`: PostgreSQL database
- `sure_redis_data`: Redis cache

### Setting Up Backups

**Database Backups:**

1. In Coolify, navigate to your service
2. Go to **Scheduled Tasks**
3. Create a new task:
   ```bash
   docker exec sure_db pg_dump -U sure_user sure_production > /backups/sure-$(date +%Y%m%d-%H%M%S).sql
   ```
4. Schedule it to run daily (e.g., at 2:00 AM)

**Volume Backups:**

You can also backup the Docker volumes directly:

```bash
# Backup storage volume
docker run --rm -v sure_storage:/data -v /backups:/backup alpine tar czf /backup/sure-storage-$(date +%Y%m%d).tar.gz -C /data .

# Backup database volume
docker run --rm -v sure_postgres_data:/data -v /backups:/backup alpine tar czf /backup/sure-db-$(date +%Y%m%d).tar.gz -C /data .
```

## Updating Sure

### Automatic Updates

To enable automatic updates:

1. In Coolify, navigate to your service
2. Go to **Settings**
3. Enable **Auto Deploy** with a schedule (e.g., check daily at 3:00 AM)
4. Coolify will automatically pull the latest image and restart services

### Manual Updates

To manually update:

1. Go to your service in Coolify
2. Click **Deploy**
3. Coolify will pull the latest image and restart

### Version Pinning

To pin to a specific version, edit the compose file in Coolify:

```yaml
services:
  web:
    image: ghcr.io/we-promise/sure:v1.2.3  # Pin to specific version
```

Available tags:
- `latest` - Latest alpha/development build
- `stable` - Latest stable release
- `v1.2.3` - Specific version

See all versions: https://github.com/we-promise/sure/pkgs/container/sure

## Monitoring & Logs

### Viewing Logs

In Coolify:
1. Navigate to your service
2. Click **Logs**
3. Select the container you want to view (web, worker, db, redis)

### Health Checks

All services include health checks:
- **Web**: HTTP check on `/up` endpoint
- **Worker**: Process check for Sidekiq
- **Database**: PostgreSQL ready check
- **Redis**: Redis ping check

Coolify will automatically restart unhealthy containers.

## Troubleshooting

### Service Won't Start

**Check logs:**
```bash
# Via Coolify UI: Logs tab

# Via SSH (if you have server access):
docker logs sure_web
docker logs sure_worker
docker logs sure_db
docker logs sure_redis
```

**Common issues:**

1. **Database connection errors**: 
   - Ensure `SECRET_KEY_BASE` is set
   - Check `POSTGRES_PASSWORD` matches in all services
   - Wait for database to be healthy (check logs)

2. **Port conflicts**:
   - Change the `PORT` environment variable if 3000 is in use
   - Update the port mapping in compose file

3. **Permission errors**:
   - The app runs as user `rails` (UID 1000)
   - Ensure volume permissions are correct

### Reset Database (⚠️ WARNING: Deletes all data)

If you need to start fresh:

```bash
# Via Coolify server SSH
cd /path/to/your/coolify/data
docker compose down
docker volume rm sure_postgres_data
docker compose up -d
```

### Can't Access Application

1. **Check domain DNS**: Ensure your domain points to the Coolify server
2. **Check firewall**: Ensure port 80/443 (or your custom port) is open
3. **Check logs**: Look for errors in web service logs
4. **Verify deployment**: Ensure all services show as "healthy" in Coolify

## Advanced Configuration

### Custom OpenAI-Compatible Endpoints

To use LM Studio, Ollama, or other local LLMs:

```bash
# LM Studio example
OPENAI_URI_BASE=http://host.docker.internal:1234/v1
OPENAI_MODEL=your-model-name

# Ollama example
OPENAI_URI_BASE=http://host.docker.internal:11434/v1
OPENAI_MODEL=llama2
```

### Email Configuration

Add these environment variables for email functionality:

```bash
SMTP_ADDRESS=smtp.gmail.com
SMTP_PORT=587
SMTP_DOMAIN=yourdomain.com
SMTP_USER_NAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_AUTHENTICATION=plain
SMTP_ENABLE_STARTTLS_AUTO=true
```

### External Database

To use an external PostgreSQL database:

1. Remove the `db` service from compose file
2. Update environment variables:
   ```bash
   DB_HOST=your-external-db-host.com
   DB_PORT=5432
   POSTGRES_USER=your_db_user
   POSTGRES_PASSWORD=your_db_password
   POSTGRES_DB=your_db_name
   ```

### Resource Limits

Add resource limits to prevent container resource exhaustion:

```yaml
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
```

## Security Recommendations

1. **Always set a strong `SECRET_KEY_BASE`**: Generate with `openssl rand -hex 64`
2. **Use strong database passwords**: At least 32 characters
3. **Keep the database internal**: Don't expose port 5432 externally
4. **Enable Coolify's automatic SSL**: Let Coolify handle HTTPS certificates
5. **Regular backups**: Set up automated daily backups
6. **Monitor logs**: Check for suspicious activity regularly
7. **Update regularly**: Keep Sure and Coolify updated

## Performance Tuning

### For Small Deployments (1-5 users)

The default configuration works well. Consider:

```yaml
redis:
  command: redis-server --appendonly yes --maxmemory 128mb --maxmemory-policy allkeys-lru
```

### For Medium Deployments (5-50 users)

Increase Redis memory and add resource limits:

```yaml
redis:
  command: redis-server --appendonly yes --maxmemory 512mb --maxmemory-policy allkeys-lru
  deploy:
    resources:
      limits:
        memory: 768M
```

### For Large Deployments (50+ users)

Consider:
- Running multiple worker containers
- Using an external managed PostgreSQL database
- Setting up Redis persistence
- Adding a CDN for static assets

## Getting Help

- **Documentation**: https://github.com/we-promise/sure/wiki
- **Discussions**: https://github.com/we-promise/sure/discussions
- **Issues**: https://github.com/we-promise/sure/issues
- **Coolify Docs**: https://coolify.io/docs

## Contributing

Found an issue or want to improve this guide? Please open a PR or issue at:
https://github.com/we-promise/sure

---

**Happy self-hosting! 🚀**
