# Troubleshooting Coolify Deployment Errors

This guide helps you diagnose and fix common deployment issues with Sure on Coolify.

## Health Check Failures

### Symptom
```
Container web-xxx is unhealthy
dependency failed to start: container web-xxx is unhealthy
```

### Diagnosis Steps

1. **Check Container Logs in Coolify**
   - Go to your service → Click on **web** container → **Logs**
   - Look for error messages during startup

2. **Check Database Connection**
   - Verify `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` are set correctly
   - Ensure they match across all environment variables
   - Database must be created before Rails can connect

3. **Check SECRET_KEY_BASE**
   - Must be exactly 128 characters (64 hex bytes)
   - Generate with: `openssl rand -hex 64`
   - Wrong length will cause Rails to fail silently

### Common Error Messages and Fixes

#### Error: "database does not exist"
**Cause**: First-time deployment, database not initialized

**Fix**: The entrypoint should handle this automatically. If it doesn't:
```bash
# In Coolify, go to your service, click on web container → Terminal
rails db:create
rails db:migrate
rails db:seed
```

#### Error: "PG::ConnectionBad"
**Cause**: Database credentials mismatch or database not ready

**Fix**:
1. Check environment variables match between services
2. Ensure database container is healthy before web starts (already configured)
3. Verify `DB_HOST=db` matches the service name

#### Error: "Invalid secret_key_base"
**Cause**: `SECRET_KEY_BASE` is too short or missing

**Fix**:
1. Generate new key: `openssl rand -hex 64`
2. Update in Coolify environment variables
3. Redeploy

#### Error: "Address already in use (Errno::EADDRINUSE)"
**Cause**: Port conflict (should not happen with latest compose file)

**Fix**: Ensure you're using the updated `compose.coolify.yml` without port mappings

#### Error: "Redis connection refused"
**Cause**: Redis not ready or wrong URL

**Fix**:
1. Check Redis container is healthy
2. Verify `REDIS_URL=redis://redis:6379/1`
3. Ensure Redis service name is `redis`

#### Error: "Bootsnap::LoadPathCache::FallbackScan"
**Cause**: This is a warning, not an error. Can be ignored during first boot.

**Fix**: No action needed, Rails will continue booting.

## Deployment Hangs

### Symptom
Deployment seems stuck, waiting for health checks

### Fix
1. **Increase start_period** in compose file (already set to 60s)
2. **Check server resources**: Ensure adequate RAM (minimum 2GB)
3. **Check container logs** for actual errors

## Viewing Detailed Logs

### In Coolify UI
1. Service → Container (web/worker/db/redis) → **Logs**
2. Enable "Follow" to see live logs
3. Adjust date range to see historical logs

### Via Server SSH (Advanced)
```bash
# List containers
docker ps -a | grep ycgss8oooo84kkgw0sksckgs

# View web logs
docker logs <web-container-id>

# Follow web logs in real-time
docker logs -f <web-container-id>

# Check container health
docker inspect <web-container-id> | grep -A 10 Health

# Enter container shell
docker exec -it <web-container-id> bash
```

## First Deployment Checklist

Before deploying, ensure:

- [ ] `SECRET_KEY_BASE` is set (128 characters from `openssl rand -hex 64`)
- [ ] `POSTGRES_PASSWORD` is set (strong password, 32+ characters)
- [ ] `POSTGRES_USER` is set (default: admin)
- [ ] `POSTGRES_DB` is set (default: sure_production)
- [ ] Latest `compose.coolify.yml` is used (no port mappings, no container names)
- [ ] Server has at least 2GB RAM available
- [ ] Domain is configured correctly (if using custom domain)

## Database Reset (⚠️ Destroys All Data)

If you need to completely reset:

### Via Coolify UI
1. Stop the service
2. Go to **Storages** tab
3. Delete volume: `postgres-data`
4. Redeploy

### Via Server SSH
```bash
# Stop containers
docker compose -p ycgss8oooo84kkgw0sksckgs down

# Remove database volume
docker volume rm ycgss8oooo84kkgw0sksckgs_postgres-data

# Redeploy in Coolify UI
```

## Manual Database Setup (If Entrypoint Fails)

If the automatic database setup isn't working:

```bash
# Enter web container
docker exec -it <web-container-id> bash

# Inside container, run:
rails db:create
rails db:migrate
rails db:seed

# Exit and restart container
exit
docker restart <web-container-id>
```

## Performance Issues

### Container keeps restarting
**Causes**:
- Out of memory
- Database connection pool exhausted
- Rails crash loop

**Fix**:
1. Check logs for actual error
2. Increase server RAM
3. Adjust database pool size in environment:
   ```bash
   DB_POOL=5
   ```

### Slow startup (> 2 minutes)
**Causes**:
- Low resources
- Asset compilation
- Large database migrations

**Fix**:
1. Increase `start_period` to 120s in compose
2. Add more RAM/CPU to server
3. Monitor with `docker stats`

## Getting More Help

### Include These Details When Asking for Help

1. **Full container logs** (especially web container)
2. **Environment variables** (redact sensitive values)
3. **Server specs** (CPU, RAM, disk)
4. **Deployment steps taken**
5. **Error messages** from Coolify UI

### Where to Get Help

- GitHub Discussions: https://github.com/we-promise/sure/discussions
- GitHub Issues: https://github.com/we-promise/sure/issues
- Coolify Discord: https://coolify.io/discord

## Quick Debug Commands

```bash
# Check all containers status
docker compose -p ycgss8oooo84kkgw0sksckgs ps

# Check web container health
docker inspect <web-container-id> --format='{{json .State.Health}}'

# Test database connection from web container
docker exec <web-container-id> psql -h db -U admin -d sure_production -c "SELECT 1"

# Test Redis connection from web container
docker exec <web-container-id> redis-cli -h redis ping

# Check Rails environment
docker exec <web-container-id> rails runner "puts Rails.env"

# Check database exists
docker exec <db-container-id> psql -U admin -l

# Check environment variables in container
docker exec <web-container-id> env | grep -E "(SECRET_KEY|POSTGRES|DB_|REDIS)"
```

## Still Having Issues?

If none of the above solutions work:

1. **Export your environment variables** from Coolify
2. **Save your volumes/data** (backup database)
3. **Delete the service** completely in Coolify
4. **Create a fresh service** with the correct configuration
5. **Restore data** if needed

This often resolves persistent configuration issues.
