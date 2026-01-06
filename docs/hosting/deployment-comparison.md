# Deployment Comparison: Docker Compose vs Coolify

This document helps you choose the right deployment method for your Sure self-hosting needs.

## Quick Comparison

| Feature | Manual Docker Compose | Coolify |
|---------|----------------------|---------|
| **Complexity** | Medium | Low |
| **Setup Time** | 15-30 minutes | 5-10 minutes |
| **UI Management** | Command line only | Web-based dashboard |
| **SSL/HTTPS** | Manual setup | Automatic (Let's Encrypt) |
| **Updates** | Manual commands | One-click or automatic |
| **Monitoring** | Manual (logs via CLI) | Built-in dashboard |
| **Backups** | Manual setup | Built-in support |
| **Best For** | Advanced users, automation | Beginners, quick setup |

## Manual Docker Compose

### ✅ Pros
- **Full Control**: Complete control over every aspect
- **No Dependencies**: Just Docker and Docker Compose needed
- **Lightweight**: Minimal overhead
- **Portable**: Works anywhere Docker runs
- **Automation-Friendly**: Easy to integrate with CI/CD

### ❌ Cons
- **Manual SSL**: Need to setup reverse proxy (Nginx/Traefik) + Let's Encrypt
- **Manual Updates**: Run commands to pull and restart
- **No UI**: Command-line only management
- **Manual Monitoring**: Need to setup your own logging/monitoring
- **More Maintenance**: You handle everything

### Best For
- DevOps professionals
- Users comfortable with command line
- Automated deployments
- Integration with existing infrastructure
- Minimal dependencies

### Setup Guide
📖 [Manual Docker Compose Guide](./hosting/docker.md)

---

## Coolify Deployment

### ✅ Pros
- **User-Friendly**: Web-based dashboard
- **Automatic SSL**: Free HTTPS via Let's Encrypt
- **One-Click Updates**: Update with a button click
- **Built-in Monitoring**: Logs, health checks, resource usage
- **Backup Support**: Easy backup configuration
- **Multiple Apps**: Manage many apps from one interface
- **Git Integration**: Auto-deploy from repository

### ❌ Cons
- **Additional Layer**: Requires Coolify installation first
- **More Resources**: Coolify itself needs resources
- **Less Control**: Abstraction hides some details
- **Coolify Updates**: Need to keep Coolify updated too

### Best For
- Beginners to self-hosting
- Users who prefer GUI over CLI
- Managing multiple applications
- Quick proof-of-concept deployments
- Teams with mixed technical skills

### Setup Guide
📖 [Coolify Deployment Guide](./hosting/coolify.md)  
🚀 [Quick Start](./COOLIFY_QUICKSTART.md)

---

## Recommended Setup by Use Case

### 🏠 Personal Use (1-5 users)
**Recommendation**: Coolify

- Easy to manage
- Automatic updates
- Simple backup configuration
- SSL out of the box

### 🏢 Small Team (5-20 users)
**Recommendation**: Either works well

- **Coolify**: If team prefers simplicity
- **Docker Compose**: If team is technical

### 🏭 Organization (20+ users)
**Recommendation**: Docker Compose

- Better for automation
- Easier integration with existing infrastructure
- More control over resources
- Can integrate with enterprise monitoring

### 🧪 Testing/Development
**Recommendation**: Docker Compose

- Faster to spin up/down
- Lighter weight
- Easier to script
- No extra dependencies

### 🎓 Learning/First Time
**Recommendation**: Coolify

- Gentler learning curve
- Visual feedback
- Less chance of misconfiguration
- Built-in best practices

---

## Resource Requirements

### Manual Docker Compose (Minimum)
- **CPU**: 1 core
- **RAM**: 1 GB
- **Storage**: 10 GB
- **Network**: Port 80, 443, 3000

### Coolify + Sure (Minimum)
- **CPU**: 2 cores
- **RAM**: 2 GB (Coolify needs ~512MB)
- **Storage**: 20 GB
- **Network**: Port 80, 443

### Recommended for Both
- **CPU**: 2+ cores
- **RAM**: 4 GB
- **Storage**: 50 GB SSD
- **Network**: 100 Mbps+

---

## Migration Between Methods

### From Docker Compose to Coolify

1. **Backup your data**:
   ```bash
   docker exec sure_db pg_dump -U sure_user sure_production > backup.sql
   docker run --rm -v sure_storage:/data -v $(pwd):/backup alpine tar czf /backup/storage.tar.gz -C /data .
   ```

2. **Install Coolify** on your server

3. **Deploy Sure via Coolify** using the compose.coolify.yml

4. **Restore data**:
   ```bash
   # Restore database
   docker exec -i sure_db psql -U sure_user sure_production < backup.sql
   
   # Restore storage
   docker run --rm -v sure_storage:/data -v $(pwd):/backup alpine tar xzf /backup/storage.tar.gz -C /data
   ```

### From Coolify to Docker Compose

1. **Backup via Coolify** (or manually as above)

2. **Copy compose.example.yml** to your server

3. **Copy your .env variables** from Coolify to `.env` file

4. **Start services**:
   ```bash
   docker compose up -d
   ```

5. **Restore data** (same commands as above)

---

## Decision Flowchart

```
Start: Want to self-host Sure?
    │
    ├─ Are you comfortable with command line? 
    │   │
    │   ├─ Yes → Do you need automation/CI-CD?
    │   │   │
    │   │   ├─ Yes → Use Manual Docker Compose
    │   │   └─ No → Either works, Coolify is easier
    │   │
    │   └─ No → Use Coolify
    │
    └─ Do you manage multiple self-hosted apps?
        │
        ├─ Yes → Use Coolify (manage all in one place)
        └─ No → Either works, Docker Compose is lighter
```

---

## Getting Help

### Docker Compose Support
- 📖 [Docker Documentation](https://docs.docker.com/)
- 💬 [Sure Discussions](https://github.com/we-promise/sure/discussions)

### Coolify Support
- 📖 [Coolify Docs](https://coolify.io/docs)
- 💬 [Coolify Discord](https://coolify.io/discord)
- 💬 [Sure Discussions](https://github.com/we-promise/sure/discussions)

---

## Still Not Sure?

Try this: Install Coolify and deploy Sure there first. If you later decide you want more control, you can always migrate to manual Docker Compose. Coolify uses standard Docker Compose files, so migration is straightforward!

**Start with Coolify → Migrate to Docker Compose if needed**

This approach lets you get up and running quickly, then optimize later if desired.
