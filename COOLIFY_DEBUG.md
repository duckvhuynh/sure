# Coolify Deployment Debugging Guide

Your environment variables are **correct** ✅:
- SECRET_KEY_BASE: 128 characters ✅
- POSTGRES_PASSWORD: Strong password ✅  
- POSTGRES_USER: admin ✅
- POSTGRES_DB: sure_production ✅
- SELF_HOSTED: true (already in compose file) ✅

Since the environment is correct but the container is still failing health checks, the issue is likely with the Rails boot process or database initialization.

## Step 1: Get Container Logs (MOST IMPORTANT)

### In Coolify UI:
1. Click on your service name
2. Find the **web** container in the list
3. Click on **Logs** button
4. Look for these specific error patterns:

#### Common Error Patterns to Look For:

**A. Database Connection Error**
```
PG::ConnectionBad
could not connect to server
FATAL: database "sure_production" does not exist
```

**B. Asset Compilation Error**
```
Sprockets::Rails::Helper::AssetNotFound
couldn't find file
```

**C. Migration Error**
```
ActiveRecord::PendingMigrationError
Migrations are pending
```

**D. Redis Connection Error**
```
Error connecting to Redis
Connection refused
```

**E. Secret Key Error**
```
secret_key_base
Missing `secret_key_base`
```

---

## Step 2: Manual Database Initialization (If Logs Show Database Issues)

If logs show "database does not exist" or migration errors:

### Option A: Via Coolify Terminal
1. In Coolify, click on **web** container
2. Click **Terminal** button
3. Run these commands:
```bash
# Create database
rails db:create

# Run migrations
rails db:migrate

# Load seed data (optional)
rails db:seed

# Check database connection
rails runner "puts ActiveRecord::Base.connection.active?"
```

### Option B: Via Server SSH
```bash
# Find your web container ID
docker ps | grep web-ycgss8oooo84kkgw0sksckgs

# Execute commands in container
docker exec -it <container-id> bash

# Inside container, run:
rails db:create
rails db:migrate
rails db:seed
exit

# Restart container
docker restart <container-id>
```

---

## Step 3: Check Container Health Manually

```bash
# Check if Rails process is running
docker exec <web-container-id> pgrep -f 'puma|rails'

# Check if port 3000 is listening
docker exec <web-container-id> netstat -tlnp | grep 3000

# Test database connection
docker exec <web-container-id> psql -h db -U admin -d sure_production -c "SELECT version();"

# Test Redis connection
docker exec <web-container-id> redis-cli -h redis ping
```

---

## Step 4: Updated Health Check

I've updated the `compose.coolify.yml` file with:
- **Longer start period**: 120s (was 60s) - gives Rails more time to boot
- **More retries**: 10 (was 5) - more forgiving for slow servers
- **Better process check**: `pgrep -f 'puma|rails'` - catches both Puma and Rails processes
- **Debug logging**: `RAILS_LOG_LEVEL: debug` - more verbose logs

**Redeploy** with the updated compose file.

---

## Step 5: Common Solutions

### Solution 1: Database Not Initialized
**Symptom**: Logs show "database does not exist"

**Fix**:
```bash
# In container terminal
rails db:create
rails db:migrate
```

### Solution 2: Rails Process Not Starting
**Symptom**: Health check fails but no error in logs

**Fix**: Check server resources:
```bash
# Check available memory
free -h

# Check container resource usage
docker stats

# Ensure at least 2GB RAM available
```

### Solution 3: Asset Precompilation Issues
**Symptom**: Logs show Sprockets errors

**Fix**: The Docker image should have precompiled assets, but if not:
```bash
# In container terminal
rails assets:precompile
```

### Solution 4: Port Already in Use
**Symptom**: "Address already in use" in logs

**This should NOT happen** with the updated compose file (no port mappings).
If it does, there's a conflict with Coolify's proxy configuration.

---

## Step 6: If Still Failing - Nuclear Option

If all else fails, completely reset:

### In Coolify:
1. **Stop** the service
2. Go to **Storages** tab
3. **Delete** the `postgres-data` volume ⚠️ (destroys all data)
4. Go to **Environment Variables**
5. Verify all variables are correct (especially SECRET_KEY_BASE is 128 chars)
6. **Redeploy**

### Via Server SSH:
```bash
# Stop and remove everything
docker compose -p ycgss8oooo84kkgw0sksckgs down -v

# Remove orphaned volumes
docker volume prune

# Redeploy in Coolify UI
```

---

## What to Share if You Need More Help

If the issue persists, please share:

1. **Full web container logs** (first 100 lines from boot)
2. **Output of these commands**:
   ```bash
   docker ps -a
   docker logs <web-container-id> 2>&1 | head -100
   docker inspect <web-container-id> | grep -A 20 Health
   ```
3. **Server specs**: RAM, CPU, disk space
4. **Coolify version**

---

## Expected Successful Boot Sequence

When working correctly, logs should show:

```
=> Booting Puma
=> Rails 7.x.x application starting in production
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Puma version: 6.x.x
* Ruby version: 3.x.x
* Min threads: 5
* Max threads: 5
* Environment: production
* Listening on http://0.0.0.0:3000
Use Ctrl-C to stop
```

Then the health check should pass and Coolify will route traffic to your domain.

---

## Next Steps

1. **Check the web container logs first** - this will tell us exactly what's wrong
2. Share the logs if you need help interpreting them
3. Try the manual database initialization if logs show database errors
4. Redeploy with the updated compose file (longer health check timeout)

The configuration is correct, so the issue is with the startup process. The logs will reveal the exact problem.
