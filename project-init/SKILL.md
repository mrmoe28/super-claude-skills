---
name: project-init
description: Automate complete Next.js project setup with GitHub repository, Vercel deployment, and NeonDB database configuration following the qinit workflow
license: Apache-2.0
allowed-tools:
  - Bash(gh*:*)
  - Bash(vercel*:*)
  - Bash(git*:*)
  - Bash(npm*:*)
  - Bash(npx*:*)
  - Bash(cd*:*)
  - Bash(mkdir*:*)
  - Read
  - Write
  - Edit
metadata:
  version: "1.0.0"
  workflow: "qinit"
---

# Project Init

Automates the complete Next.js project setup workflow (qinit) with GitHub, Vercel, and NeonDB integration.

## What This Skill Does

Executes the full project initialization workflow:
1. Verifies required CLIs are installed and authenticated
2. Creates Next.js 15 project with TypeScript and Tailwind
3. Initializes Git and creates GitHub repository
4. Links to Vercel and integrates NeonDB
5. Sets up environment variables
6. Creates configuration files
7. Performs initial deployment

## Pre-Setup Questions

**ALWAYS ask the user these questions before starting:**

1. **Project name** (kebab-case format)
2. **Description** (1 sentence)
3. **Tech stack** (default: Next.js 15 + Neon + Vercel)
4. **GitHub visibility** (public or private - default: public)

## Setup Workflow

### Phase 1: CLI Verification
```bash
# Check if GitHub CLI is authenticated
gh auth status

# Check if Vercel CLI is authenticated
vercel whoami

# If not authenticated, guide user to:
# - gh auth login
# - vercel login
```

### Phase 2: Project Creation
```bash
# Navigate to Desktop
cd ~/Desktop

# Create project directory
mkdir project-name
cd project-name

# Create Next.js 15 app
npx create-next-app@latest . \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --no-turbopack
```

### Phase 3: Git & GitHub
```bash
# Initialize Git
git init

# Stage all files
git add .

# Initial commit
git commit -m "chore: initial setup"

# Create GitHub repository
gh repo create project-name --public --source=. --remote=origin

# Push to GitHub
git push -u origin main
```

### Phase 4: Vercel & NeonDB
```bash
# Link to Vercel
vercel link --yes

# Add Neon integration (creates DB and DATABASE_URL)
vercel integration add neon

# Pull environment variables
vercel env pull .env.local

# Deploy to production
vercel --prod
```

### Phase 5: Configuration Files

**Create .env.example:**
```bash
# Database
DATABASE_URL=

# Add other required env vars as template
```

**Create vercel.json:**
```json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"]
}
```

**Create/Update next.config.ts:**
```typescript
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typescript: {
    ignoreBuildErrors: false,
  },
  eslint: {
    ignoreDuringBuilds: false,
  },
  typedRoutes: true,
}

export default nextConfig
```

**Create docs/SETUP.md:**
```markdown
# Setup Instructions

## Prerequisites
- Node.js 18+
- npm or yarn
- GitHub account
- Vercel account

## Installation

1. Clone the repository:
   \`\`\`bash
   git clone <repo-url>
   cd <project-name>
   \`\`\`

2. Install dependencies:
   \`\`\`bash
   npm install
   \`\`\`

3. Set up environment variables:
   \`\`\`bash
   cp .env.example .env.local
   \`\`\`

4. Run development server:
   \`\`\`bash
   npm run dev
   \`\`\`

## Deployment

This project is automatically deployed to Vercel on every push to main.

## Database

Database hosted on Neon. Connection string in environment variables.
```

**Update README.md:**
```markdown
# [Project Name]

[Project Description]

## Tech Stack

- Next.js 15
- React 19
- TypeScript
- Tailwind CSS 4
- Neon PostgreSQL
- Vercel Hosting

## Getting Started

See [Setup Instructions](./docs/SETUP.md)

## Development

\`\`\`bash
npm run dev      # Start dev server
npm run build    # Build for production
npm run lint     # Run ESLint
\`\`\`

## Deployment

Deployed automatically to Vercel: [Production URL]
```

### Phase 6: .gitignore Configuration
```
# dependencies
node_modules/
.pnp
.pnp.js

# testing
coverage/

# next.js
.next/
out/
build/
dist/

# environment variables
.env
.env*.local

# vercel
.vercel

# typescript
*.tsbuildinfo
next-env.d.ts

# misc
.DS_Store
*.pem

# debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# local env files
.env.local
.env.development.local
.env.test.local
.env.production.local
```

## Critical Rules

### MANDATORY CHECKS
- ✅ CLI authentication verified BEFORE starting
- ✅ No Turbopack flag in create-next-app
- ✅ .env.local added to .gitignore
- ✅ Environment variables never committed
- ✅ Initial deployment successful
- ✅ GitHub repository created
- ✅ Vercel project linked
- ✅ NeonDB integrated

### NEVER DO
- ❌ Create Vercel project before GitHub repo
- ❌ Skip CLI authentication checks
- ❌ Manually create NeonDB (use Vercel integration)
- ❌ Use --turbopack flag
- ❌ Track .env.local in Git
- ❌ Skip initial deployment
- ❌ Proceed without user approval

## Success Verification

After setup, verify:
1. GitHub repository is created and pushed
2. Vercel project is linked and deployed
3. NeonDB integration is active
4. Environment variables are pulled locally
5. Production deployment is live
6. .env.local exists locally (gitignored)
7. Documentation is complete

## Output Format

Provide the user with:
- GitHub repository URL
- Vercel deployment URL
- NeonDB connection status
- Local development command
- Next steps

## When to Use This Skill

- Starting a new Next.js project
- Setting up production infrastructure
- Integrating GitHub + Vercel + NeonDB
- Automating the qinit workflow
- Creating properly configured projects

## Error Handling

If any step fails:
1. Stop immediately
2. Report exact error
3. Don't proceed to next steps
4. Guide user to fix the issue
5. Resume only after confirmation
