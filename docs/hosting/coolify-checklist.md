# Coolify Deployment Checklist

Use this checklist to ensure a smooth deployment of Sure to Coolify.

## Pre-Deployment

- [ ] Coolify instance is running and accessible
- [ ] You have admin access to Coolify dashboard
- [ ] (Optional) Domain name is ready and DNS can be configured
- [ ] You have generated `SECRET_KEY_BASE` (run: `openssl rand -hex 64`)
- [ ] You have chosen a strong `POSTGRES_PASSWORD` (minimum 32 characters recommended)

## Deployment Steps

### 1. Service Creation
- [ ] Logged into Coolify dashboard
- [ ] Created new Docker Compose service
- [ ] Named the service (e.g., "Sure Finance")

### 2. Configuration
- [ ] Copied `compose.coolify.yml` content to Coolify
- [ ] Saved the compose configuration

### 3. Environment Variables
Required variables set:
- [ ] `SECRET_KEY_BASE` (generated with openssl)
- [ ] `POSTGRES_USER` (default: sure_user)
- [ ] `POSTGRES_PASSWORD` (strong password)
- [ ] `POSTGRES_DB` (default: sure_production)

Optional variables (if needed):
- [ ] `OPENAI_ACCESS_TOKEN` (for AI features)
- [ ] `TWELVE_DATA_API_KEY` (for market data)
- [ ] `ONBOARDING_STATE` (open/closed/invite_only)

### 4. Domain Configuration (Optional)
- [ ] Added domain in Coolify domains tab
- [ ] Updated DNS A record to point to server IP
- [ ] Waited for DNS propagation (can take up to 24 hours)
- [ ] Verified SSL certificate is active

### 5. Deployment
- [ ] Clicked "Deploy" button in Coolify
- [ ] Monitored deployment logs for errors
- [ ] Waited for all services to become healthy (2-3 minutes)
- [ ] Verified all containers are running (web, worker, db, redis)

## Post-Deployment

### 6. Initial Access
- [ ] Accessed application at domain or `http://server-ip:3000`
- [ ] Login page loads successfully
- [ ] Clicked "Create your account"
- [ ] Successfully created first user account
- [ ] Logged in successfully

### 7. Basic Functionality Test
- [ ] Dashboard loads
- [ ] Can create a new account (financial account)
- [ ] Can add a transaction manually
- [ ] Can navigate through different pages
- [ ] No console errors in browser

### 8. Backup Configuration
- [ ] Configured database backup schedule in Coolify
- [ ] Tested backup creation manually
- [ ] Verified backup location and accessibility
- [ ] Documented backup restoration procedure

## Ongoing Maintenance

### Weekly
- [ ] Check application health status
- [ ] Review logs for errors
- [ ] Monitor resource usage (CPU, RAM, disk)

### Monthly
- [ ] Review and test backups
- [ ] Check for application updates
- [ ] Review security advisories
- [ ] Update SSL certificates if needed (Coolify handles this automatically)

### As Needed
- [ ] Update Sure when new versions are released
- [ ] Scale resources if performance degrades
- [ ] Adjust worker count if background jobs are slow

## Troubleshooting Checklist

If something goes wrong:

- [ ] Checked Coolify logs for all services
- [ ] Verified all environment variables are set correctly
- [ ] Confirmed all services show as "healthy"
- [ ] Checked server firewall allows ports 80/443
- [ ] Verified DNS is pointing to correct IP
- [ ] Tried redeploying the service
- [ ] Checked server has enough resources (CPU, RAM, disk)
- [ ] Consulted [troubleshooting guide](docs/hosting/coolify.md#troubleshooting)

## Security Checklist

- [ ] `SECRET_KEY_BASE` is unique and secure (64+ hex characters)
- [ ] `POSTGRES_PASSWORD` is strong (32+ characters)
- [ ] Database port (5432) is NOT exposed externally
- [ ] Redis port (6379) is NOT exposed externally
- [ ] HTTPS/SSL is enabled via Coolify
- [ ] Regular backups are configured
- [ ] Coolify itself is kept updated
- [ ] Server OS is updated regularly

## Optional: AI Features

If enabling AI features:

- [ ] `OPENAI_ACCESS_TOKEN` is set
- [ ] OpenAI account has spending limits configured
- [ ] Tested chat feature
- [ ] Tested rule suggestions
- [ ] (Optional) Configured custom OpenAI endpoint

## Optional: Advanced Configuration

- [ ] Configured custom email SMTP settings
- [ ] Set up external PostgreSQL database
- [ ] Configured resource limits for containers
- [ ] Set up monitoring/alerting
- [ ] Configured multiple worker containers for scaling

## Documentation

- [ ] Documented your specific configuration
- [ ] Saved environment variables securely
- [ ] Noted domain and SSL configuration
- [ ] Created runbook for common tasks
- [ ] Shared access credentials with team (if applicable)

## Getting Help

If you're stuck:

1. **Check the logs**: Coolify → Your Service → Logs
2. **Review the guide**: [docs/hosting/coolify.md](docs/hosting/coolify.md)
3. **Search existing issues**: [GitHub Issues](https://github.com/we-promise/sure/issues)
4. **Ask for help**: [GitHub Discussions](https://github.com/we-promise/sure/discussions)

---

## Completed? 🎉

Congratulations! You've successfully deployed Sure to Coolify.

**Next steps:**
- Connect your bank accounts (if supported in your region)
- Import transactions via CSV
- Set up budgets and categories
- Explore AI features (if configured)

**Share your success:**
- Consider contributing to the project
- Share feedback in GitHub Discussions
- Help others in the community

Happy budgeting! 💰
