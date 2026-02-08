# Domain & Environment Setup Guide for Nexio

**Primary Domain:** https://nexio.work

---

## Cloudflare DNS Configuration

### Add Domain to Cloudflare

1. **Login to Cloudflare Dashboard:**
   ```
   curl "https://dash.cloudflare.com/login"
   ```

2. **Add Custom Domain:**
   - Navigate to: **Workers & Pages**
   - Click: **Custom Domains**
   - Add domain: `nexio.work`
   - Configure DNS records (Cloudflare will provide these):
     ```
     Type: CNAME
     Name: @ (or your subdomain like www)
     Target: nexio.pages.dev (for staging)
     Target: nexio.work (for production)
     Proxy: No
     TTL: Auto
     ```

3. **Verify DNS:**
   - Wait for DNS propagation (usually 5-15 minutes)
   - Cloudflare dashboard will show "Active" when ready

---

## Environment Configuration

### Development (VPS Sandbox)
- **Environment:** `development`
- **URL:** `http://localhost:3000` (Next.js)
- **API:** `http://localhost:8787` (Hono Worker - local mock)
- **Database:** Local SQLite file (for development)
- **Command:** `pnpm dev`

### Staging
- **Environment:** `staging`
- **URL:** `https://nexio.work/staging` (if configured) or `https://nexio.pages.dev` (default)
- **API:** `https://api-staging.nexio.work`
- **Domain:** Configure `staging.nexio.work` as CNAME pointing to staging Pages
- **Command:** `wrangler pages deploy ./apps/web/dist --env staging`

### Production
- **Environment:** `production`
- **URL:** `https://nexio.work`
- **API:** `https://api.nexio.work`
- **Domain:** Configure `nexio.work` as CNAME pointing to production Pages
- **Command:** `wrangler pages deploy ./apps/web/dist --env production`

---

## Wrangler Multi-Environment Commands

### Environment-Specific Deployment

```bash
# Deploy to Staging
wrangler pages deploy ./apps/web/dist --env staging

# Deploy to Production
wrangler pages deploy ./apps/web/dist --env production

# Deploy API to Staging
wrangler deploy apps/api --env staging

# Deploy API to Production
wrangler deploy apps/api --env production
```

### Environment Variables

Set environment variables per environment:

```bash
# Staging
wrangler secret put API_SECRET_STAGING --env staging

# Production
wrangler secret put API_SECRET_PRODUCTION --env production

# Development (local - use .dev file)
# Add to .dev file (or .env.local):
API_SECRET=local_dev_secret
```

---

## GitHub Secrets Configuration

### Required Secrets

Add these secrets to GitHub repository (Settings → Secrets and variables → Actions):

```
CLOUDFLARE_API_TOKEN        # Cloudflare API token (already provided)
CLOUDFLARE_ACCOUNT_ID       # Cloudflare Account ID (auto-detected by token)
```

### How to Add Secrets

1. Go to: https://github.com/Nexio-work/nexio/settings/secrets/actions
2. Click: **New repository secret**
3. Add each secret:
   - Name: `CLOUDFLARE_API_TOKEN`
   - Value: `vVt86umf1bl96nKSJE8vV1kGBUG-gqoM5eBtMbmi`
   - Click: **Add secret**
4. Repeat for `CLOUDFLARE_ACCOUNT_ID` (optional, will be auto-detected)

---

## Initial Setup Commands

### First-Time Setup

```bash
# 1. Install Wrangler CLI
npm install -g wrangler

# 2. Login with token
wrangler login --api-token vVt86umf1bl96nKSJE8vV1kGBUG-gqoM5eBtMbmi

# 3. Create D1 database
wrangler d1 create nexio-db

# 4. Create R2 bucket
wrangler r2 bucket create nexio-assets

# 5. Create KV namespace
wrangler kv namespace create

# 6. Verify domains
# Go to Cloudflare Dashboard and configure nexio.work

# 7. Test deployment to staging
wrangler pages deploy ./apps/web/dist --env staging

# 8. Test deployment to production
wrangler pages deploy ./apps/web/dist --env production
```

### Daily Workflow (after initial setup)

```bash
# Update dependencies
pnpm install

# Run tests
pnpm test

# Deploy to staging
wrangler pages deploy ./apps/web/dist --env staging

# Test staging environment
# Open https://nexio.work/staging and verify

# Deploy to production
wrangler pages deploy ./apps/web/dist --env production

# Test production environment
# Open https://nexio.work and verify
```

---

## Monitoring & Troubleshooting

### Check Deployment Status

```bash
# Check Workers status
wrangler deployments list

# Check Pages deployments
wrangler pages deployment list

# Check DNS propagation
dig nexio.work
```

### Common Issues

**Issue:** DNS not propagating
- **Solution:** Wait 5-15 minutes, check with `dig nexio.work`

**Issue:** Build fails
- **Solution:** Check logs with `wrangler tail`

**Issue:** Worker errors
- **Solution:** Check Cloudflare dashboard: Workers → Real-time logs

---

## Next Steps

1. **Configure nexio.work domain** in Cloudflare dashboard
2. **Add CLOUDFLARE_ACCOUNT_ID secret** to GitHub (if needed)
3. **Test deployment to staging** after Planning agent completes
4. **Deploy to production** after Sprint 1 completion
5. **Set up monitoring** (Sentry, analytics)

---

**Created:** 2026-02-08
**For:** Nexio - Agentic OS Project
