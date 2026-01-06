# Quick Start: Deploy Sure to Coolify

Deploy Sure in under 5 minutes with Coolify!

## What You Need

- A running Coolify instance
- 5 minutes of your time

## Steps

### 1. Create Service in Coolify

1. Open your Coolify dashboard
2. Click **+ Add New Resource** → **Docker Compose - Empty Service**
3. Name it "Sure Finance" (or whatever you prefer)

### 2. Add the Docker Compose Configuration

Copy the contents of [`compose.coolify.yml`](../../compose.coolify.yml) and paste it into the Coolify compose editor.

### 3. Set Environment Variables

In the **Environment Variables** section, add:

```bash
# REQUIRED: Generate with: openssl rand -hex 64
SECRET_KEY_BASE=your_generated_secret_key_here

# REQUIRED: Database password
POSTGRES_PASSWORD=your_secure_database_password
```

### 4. Deploy!

Click **Deploy** and wait 2-3 minutes for all services to start.

### 5. Create Your Account

Visit your domain (or server IP:3000) and create your account!

## Need More Details?

📚 **Full guide**: [docs/hosting/coolify.md](../hosting/coolify.md)

## Optional: Add AI Features

To enable AI-powered chat and insights:

```bash
OPENAI_ACCESS_TOKEN=sk-your-openai-key-here
```

⚠️ **Note**: This will incur OpenAI costs. Set spend limits!

## Optional: Custom Domain

1. In Coolify, go to **Domains** tab
2. Add your domain: `sure.yourdomain.com`
3. Update DNS: Add A record pointing to your server
4. Coolify handles SSL automatically via Let's Encrypt

## Need Help?

- 📖 Full documentation: [docs/hosting/coolify.md](../hosting/coolify.md)
- 💬 Discussions: https://github.com/we-promise/sure/discussions
- 🐛 Issues: https://github.com/we-promise/sure/issues

---

**Happy self-hosting! 🚀**
