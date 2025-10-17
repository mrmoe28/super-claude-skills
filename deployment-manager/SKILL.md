---
name: deployment-manager
description: Manage Vercel deployments, GitHub workflows, and NeonDB integrations with automated setup, monitoring, and troubleshooting for production environments
license: Apache-2.0
allowed-tools:
  - Bash(vercel*:*)
  - Bash(gh*:*)
  - Bash(git*:*)
  - Bash(npm*:*)
  - Read
  - Write
  - Edit
metadata:
  version: "1.0.0"
  platforms: "Vercel, GitHub, NeonDB"
---

# Deployment Manager

Expert management of Vercel deployments, GitHub workflows, and NeonDB integrations.

## Core Principles

### CRITICAL DEPLOYMENT RULES
- **NEVER** create new Vercel projects without explicit user permission
- **NEVER** deploy or redeploy applications without being asked
- **ALWAYS** ask before making any deployment-related changes
- **ALWAYS** describe what will happen before deployment
- **WAIT** for approval before proceeding

### Deployment Philosophy
- Deployments should be predictable
- Changes should be reversible
- Errors should be caught early
- Logs should be monitored
- Database changes should be careful

## Vercel Management

### Project Setup

**Link existing project:**
```bash
# Navigate to project
cd /path/to/project

# Link to Vercel (will prompt for team/project)
vercel link

# Or link with specific configuration
vercel link --yes
```

**Environment Variables:**
```bash
# List environment variables
vercel env ls

# Add environment variable
vercel env add [name]

# Remove environment variable
vercel env rm [name]

# Pull environment variables to local
vercel env pull .env.local

# Pull from specific environment
vercel env pull .env.local --environment=production
```

### Deployment Commands

**Preview Deployment:**
```bash
# Deploy to preview (branch deploy)
vercel

# Deploy specific directory
vercel ./dist

# Deploy with environment variables
vercel --env KEY=value
```

**Production Deployment:**
```bash
# Deploy to production
vercel --prod

# Deploy specific commit to production
git checkout <commit-hash>
vercel --prod
```

**Deployment Management:**
```bash
# List deployments
vercel ls

# List deployments for specific project
vercel ls [project-name]

# Get deployment URL
vercel ls --json | jq '.[0].url'

# Inspect deployment
vercel inspect [deployment-url]

# View deployment logs
vercel logs [deployment-url]

# Follow logs in real-time
vercel logs [deployment-url] --follow

# Cancel deployment
vercel cancel [deployment-url]

# Delete deployment
vercel rm [deployment-url]
```

### Integration Management

**NeonDB Integration:**
```bash
# Add Neon integration (creates DB + env vars)
vercel integration add neon

# Remove integration
vercel integration remove neon

# List integrations
vercel integration ls
```

**Other Integrations:**
```bash
# Popular integrations
vercel integration add upstash      # Redis
vercel integration add planetscale  # MySQL
vercel integration add mongodb      # MongoDB
vercel integration add checkly      # Monitoring
```

### Domain Management

**Add Domain:**
```bash
# Add domain to project
vercel domains add [domain]

# Add domain to specific project
vercel domains add [domain] [project-name]

# List domains
vercel domains ls

# Remove domain
vercel domains rm [domain]
```

**DNS Configuration:**
```bash
# Get DNS configuration
vercel dns ls [domain]

# Add DNS record
vercel dns add [domain] [name] [type] [value]

# Example: Add A record
vercel dns add example.com @ A 76.76.21.21
```

## GitHub Integration

### Automatic Deployments

**Setup (Vercel Auto-Deploys from GitHub):**
1. Vercel automatically detects GitHub repos
2. Every push to `main` → Production deployment
3. Every PR → Preview deployment
4. Deployment status updates in PR

**Disable Auto-Deploy:**
```bash
# In vercel.json
{
  "github": {
    "enabled": false
  }
}
```

**Custom Branch Deploys:**
```bash
# In vercel.json
{
  "github": {
    "enabled": true,
    "autoAlias": true,
    "silent": false,
    "autoJobCancelation": true
  },
  "git": {
    "deploymentEnabled": {
      "main": true,
      "staging": true
    }
  }
}
```

### Pull Request Integration

**PR Preview URLs:**
- Each PR gets unique preview URL
- URL format: `[project]-[git-branch]-[scope].vercel.app`
- Status checks in PR
- Comments with deployment URL

**PR Deployment Comments:**
```bash
# Vercel bot automatically comments with:
# ✅ Deployment successful
# 🔗 Preview URL: https://...
# 📊 Lighthouse scores
```

## NeonDB Management

### Database Setup

**Via Vercel Integration:**
```bash
# Add Neon integration
vercel integration add neon

# This automatically:
# - Creates Neon database
# - Adds DATABASE_URL to Vercel
# - Configures connection pooling
```

**Manual Setup:**
```bash
# Get connection string from Neon dashboard
# Add to Vercel environment variables
vercel env add DATABASE_URL

# Pull to local
vercel env pull .env.local
```

### Connection Strings

**Pooled Connection (Recommended for serverless):**
```
postgresql://user:pass@host-pooler.region.aws.neon.tech/dbname
```

**Direct Connection:**
```
postgresql://user:pass@host.region.aws.neon.tech/dbname
```

### Database Migrations

**Using Drizzle (Recommended):**
```bash
# Generate migration
npx drizzle-kit generate:pg

# Push to database
npx drizzle-kit push:pg

# Run in production (after deploy)
vercel env pull .env.local
npm run db:migrate
```

**Using Prisma:**
```bash
# Generate migration
npx prisma migrate dev --name [migration-name]

# Deploy to production
npx prisma migrate deploy

# Generate client
npx prisma generate
```

## Monitoring & Troubleshooting

### Deployment Logs

**View Logs:**
```bash
# Recent deployment logs
vercel logs

# Specific deployment
vercel logs [deployment-url]

# Live logs (follow)
vercel logs [deployment-url] --follow

# Filter by severity
vercel logs [deployment-url] --severity error

# Logs from specific function
vercel logs [deployment-url] --function api/users
```

### Build Logs

**Check Build Process:**
```bash
# Inspect deployment build
vercel inspect [deployment-url]

# View build logs in output
# Look for:
# - npm install output
# - Build commands
# - Environment variables (sanitized)
# - Error messages
```

### Common Issues & Solutions

**Build Failures:**
```bash
# 1. Check local build
npm run build

# 2. Check node version
node --version
# Ensure matches package.json engines

# 3. Check environment variables
vercel env ls

# 4. Check build logs
vercel logs [deployment-url]
```

**Environment Variable Issues:**
```bash
# Verify variables exist
vercel env ls

# Check variable is in correct environment
vercel env ls --environment=production

# Pull and test locally
vercel env pull .env.local
npm run dev
```

**Database Connection Issues:**
```bash
# Test connection string locally
vercel env pull .env.local
npm run db:test

# Check Neon database status
# Visit Neon dashboard

# Verify connection string format
# Pooled for serverless: -pooler.neon.tech
```

## CI/CD Workflows

### GitHub Actions Integration

**Example Workflow:**
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
      - run: npm run build
```

**Vercel + GitHub Actions:**
```yaml
# Deploy to Vercel from Actions
- name: Deploy to Vercel
  run: vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
  env:
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
```

## Security Best Practices

### Environment Variables
- ✅ Never commit `.env` files
- ✅ Use `.env.example` for templates
- ✅ Rotate secrets regularly
- ✅ Use different values for dev/staging/prod
- ✅ Prefix client-side vars with `NEXT_PUBLIC_`

### Database Security
- ✅ Use connection pooling for serverless
- ✅ Rotate database passwords
- ✅ Use read-only users when possible
- ✅ Enable SSL for connections
- ✅ Implement proper access controls

### Deployment Security
- ✅ Enable deployment protection
- ✅ Require approvals for production
- ✅ Use preview deployments for testing
- ✅ Monitor deployment logs
- ✅ Set up alerts for failures

## Configuration Files

### vercel.json
```json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "regions": ["iad1"],
  "env": {
    "NODE_ENV": "production"
  },
  "build": {
    "env": {
      "NEXT_PUBLIC_API_URL": "https://api.example.com"
    }
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api.example.com/:path*"
    }
  ]
}
```

### .vercelignore
```
# Dependencies
node_modules/

# Testing
coverage/
*.test.ts
*.spec.ts

# Environment
.env
.env.local
.env*.local

# Development
.vscode/
.idea/

# Build
.next/
out/
dist/

# Misc
.DS_Store
*.log
```

## Rollback Procedures

### Quick Rollback
```bash
# List recent deployments
vercel ls

# Promote previous deployment to production
vercel promote [previous-deployment-url]

# Or deploy specific commit
git checkout [previous-commit]
vercel --prod
```

### Emergency Rollback
```bash
# 1. Immediately rollback to last known good
vercel promote [last-good-deployment-url]

# 2. Investigate the issue
vercel logs [failed-deployment-url]

# 3. Fix locally
git revert [bad-commit]
git push

# 4. Verify auto-deploy or manual deploy
vercel --prod
```

## Monitoring Setup

### Vercel Analytics
```bash
# Enable in Vercel Dashboard
# Or add to next.config.ts
module.exports = {
  experimental: {
    webVitalsAttribution: ['CLS', 'LCP']
  }
}
```

### Custom Monitoring
```typescript
// Send deployment events
fetch('https://monitoring.example.com/deploy', {
  method: 'POST',
  body: JSON.stringify({
    environment: process.env.VERCEL_ENV,
    commit: process.env.VERCEL_GIT_COMMIT_SHA,
    branch: process.env.VERCEL_GIT_COMMIT_REF
  })
})
```

## When to Use This Skill

- Setting up deployments
- Managing environment variables
- Troubleshooting deployment issues
- Configuring database connections
- Setting up CI/CD
- Monitoring production
- Rolling back deployments
- Managing domains
- Configuring integrations

## Success Criteria

Deployment is successful when:
- ✅ Build completes without errors
- ✅ All environment variables set
- ✅ Database connection works
- ✅ Application accessible at URL
- ✅ No errors in logs
- ✅ All health checks pass
- ✅ Preview deployments work
- ✅ Rollback procedure tested
