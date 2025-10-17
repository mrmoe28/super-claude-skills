# 🚀 Super Claude Skills

<div align="center">

**The Ultimate Collection of Claude AI Skills for Developers**

![Skills](https://img.shields.io/badge/skills-24-blue?style=for-the-badge)
![License](https://img.shields.io/badge/license-Apache%202.0-green?style=for-the-badge)
![Version](https://img.shields.io/badge/version-2.0.0-orange?style=for-the-badge)
![GitHub Stars](https://img.shields.io/github/stars/mrmoe28/super-claude-skills?style=for-the-badge)

*Transform Claude into a powerhouse for software development, system design, and automation*

[Installation](#-installation) • [Skills](#-available-skills) • [Usage](#-usage-examples) • [Contributing](#-contributing)

</div>

---

## 📋 Table of Contents

- [What are Skills?](#-what-are-skills)
- [Installation](#-installation)
- [Available Skills](#-available-skills)
  - [Document Skills](#-document-skills)
  - [Creative Skills](#-creative-skills)
  - [Development Skills](#-development-skills)
  - [Enterprise Skills](#-enterprise-skills)
  - [Workflow Skills](#-workflow-skills)
- [Quick Start](#-quick-start)
- [Usage Examples](#-usage-examples)
- [Skill Comparison](#-skill-comparison)
- [Creating Custom Skills](#-creating-your-own-skills)
- [Contributing](#-contributing)
- [License](#-license)

---

## 💡 What are Skills?

Skills are **specialized instruction sets** that Claude loads dynamically to excel at specific tasks. Think of them as expert personas that Claude can adopt on-demand.

```
Regular Claude → "I can help with that"
Claude + Skills → "Let me use my database-optimizer skill to analyze this query..."
```

**Benefits:**
- ✅ **Expert-level performance** on complex tasks
- ✅ **Consistent patterns** across projects
- ✅ **Faster problem-solving** with proven methodologies
- ✅ **Reusable knowledge** that improves over time

---

## 📦 Installation

### Claude Code (Recommended)

```bash
# One command installation
/plugin marketplace add mrmoe28/super-claude-skills
```

### Manual Installation

```bash
# Clone to marketplace directory
cd ~/.claude/plugins/marketplaces
git clone https://github.com/mrmoe28/super-claude-skills.git

# Or update existing installation
cd ~/.claude/plugins/marketplaces/super-claude-skills
git pull
```

### Claude.ai

Upload individual skill folders from this repo to Claude.ai following the [official guide](https://support.claude.com/en/articles/12512180-using-skills-in-claude).

---

## 🎯 Available Skills

### 📄 Document Skills
*Professional document creation and manipulation*

| Skill | Description | Use Case | Difficulty |
|-------|-------------|----------|------------|
| **xlsx** | Excel spreadsheet creation & analysis | Financial reports, data analysis | ⭐⭐ |
| **docx** | Word document editing with tracking | Contracts, reports, documentation | ⭐⭐ |
| **pptx** | PowerPoint presentation builder | Pitch decks, training materials | ⭐⭐ |
| **pdf** | PDF toolkit (extract, merge, split) | Form processing, document management | ⭐⭐⭐ |

**Example Use:**
```
"Use xlsx skill to create a budget spreadsheet with formulas"
"Extract form fields from invoice.pdf using the pdf skill"
```

---

### 🎨 Creative Skills
*Design and artistic content generation*

| Skill | Description | Use Case | Difficulty |
|-------|-------------|----------|------------|
| **canvas-design** | Beautiful PNG/PDF visuals | Social media graphics, posters | ⭐⭐ |
| **algorithmic-art** | Generative art with p5.js | Unique artwork, backgrounds | ⭐⭐⭐ |
| **slack-gif-creator** | Animated GIFs for Slack | Team celebrations, reactions | ⭐⭐ |
| **theme-factory** | Pre-built design themes | Consistent branding, UI styling | ⭐ |
| **brand-guidelines** | Apply brand standards | Corporate design consistency | ⭐ |

**Example Use:**
```
"Design a hero image for my landing page using canvas-design"
"Create a generative art NFT with algorithmic-art"
```

---

### 💻 Development Skills
*Technical tools for building and testing*

| Skill | Description | Use Case | Difficulty |
|-------|-------------|----------|------------|
| **artifacts-builder** | React artifacts with shadcn/ui | Interactive demos, prototypes | ⭐⭐ |
| **mcp-builder** | MCP server creation guide | API integrations, tool building | ⭐⭐⭐ |
| **webapp-testing** | Playwright test automation | UI testing, debugging | ⭐⭐ |
| **skill-creator** | Create custom skills | Extending Claude's capabilities | ⭐⭐ |

**Example Use:**
```
"Build an interactive calculator artifact using artifacts-builder"
"Create an MCP server for the Notion API with mcp-builder"
```

---

### 🏢 Enterprise Skills
*Business and communication tools*

| Skill | Description | Use Case | Difficulty |
|-------|-------------|----------|------------|
| **internal-comms** | Company communications | Newsletters, announcements, FAQs | ⭐ |

**Example Use:**
```
"Write a quarterly newsletter using internal-comms skill"
```

---

### ⚡ Workflow Skills
*Professional development automation and problem-solving*

#### 🛠️ Core Development

| Skill | Description | Best For | Difficulty |
|-------|-------------|----------|------------|
| **nextjs-pro** | Next.js 15 + React 19 expert | Modern web apps with App Router | ⭐⭐ |
| **project-init** | Automated project setup | Quick starts (GitHub + Vercel + Neon) | ⭐⭐ |
| **code-enforcer** | Quality standards enforcement | Zero-tolerance code review | ⭐⭐ |
| **web-builder** | SuperDesign + ShadCN UI | Beautiful, accessible interfaces | ⭐⭐ |

#### 🔧 Problem Solving

| Skill | Description | Best For | Difficulty |
|-------|-------------|----------|------------|
| **debug-protocol** | Systematic debugging | Root cause analysis, error fixing | ⭐⭐ |
| **database-optimizer** | Query & performance optimization | Slow queries, N+1 problems | ⭐⭐⭐ |
| **api-designer** | Scalable REST API design | Versioning, auth, rate limiting | ⭐⭐⭐ |
| **legacy-refactor** | Safe code modernization | Technical debt reduction | ⭐⭐⭐ |

#### 🏗️ Architecture & Infrastructure

| Skill | Description | Best For | Difficulty |
|-------|-------------|----------|------------|
| **system-architect** | System architecture design | Scalable applications | ⭐⭐⭐⭐ |
| **deployment-manager** | Vercel/GitHub/NeonDB ops | Production deployments | ⭐⭐⭐ |

#### 💳 Payments & Auth

| Skill | Description | Best For | Difficulty |
|-------|-------------|----------|------------|
| **subscription-builder** | Recurring billing (Stripe/Square) | SaaS subscription models | ⭐⭐⭐ |
| **square-integration** | Square payment processing | E-commerce, POS integration | ⭐⭐⭐ |
| **auth-setup** | Authentication (NextAuth/Clerk) | User authentication systems | ⭐⭐ |

---

## 🚀 Quick Start

### 1. Install the Marketplace
```bash
/plugin marketplace add mrmoe28/super-claude-skills
```

### 2. Use a Skill
Simply mention the skill name in your request:
```
"Use nextjs-pro to create a dashboard page"
"Use database-optimizer to fix these slow queries"
```

### 3. Combine Multiple Skills
```
"Use system-architect to design the architecture, then use
auth-setup to implement authentication"
```

---

## 📚 Usage Examples

### 🎯 Real-World Scenarios

#### Scenario 1: Building a SaaS Application

```markdown
Step 1: "Use system-architect to design a scalable SaaS architecture"
→ Get complete architecture with database, API, caching strategy

Step 2: "Use project-init to set up the project"
→ Creates Next.js app with GitHub, Vercel, NeonDB

Step 3: "Use auth-setup to add authentication with Google OAuth"
→ Complete NextAuth setup with middleware

Step 4: "Use subscription-builder to add Stripe subscriptions"
→ Subscription billing with webhooks

Step 5: "Use deployment-manager to deploy to production"
→ Automated deployment with monitoring
```

#### Scenario 2: Optimizing Slow Application

```markdown
Step 1: "Use debug-protocol to diagnose performance issues"
→ Systematic root cause analysis

Step 2: "Use database-optimizer to fix slow queries"
→ Index optimization, query improvements

Step 3: "Use api-designer to implement caching"
→ Multi-layer caching strategy

Result: 10x faster application
```

#### Scenario 3: Modernizing Legacy Code

```markdown
Step 1: "Use legacy-refactor to analyze this old codebase"
→ Safety assessment and refactoring plan

Step 2: "Use code-enforcer to add tests and TypeScript"
→ Type safety and test coverage

Step 3: "Use api-designer to create modern REST API"
→ Replace old endpoints with scalable API

Result: Maintainable, tested, modern code
```

---

## 🔄 Skill Comparison

### When to Use Which Skill?

| Problem | Use This Skill | Why |
|---------|---------------|-----|
| Slow database queries | `database-optimizer` | Query profiling, indexing strategies |
| Starting new project | `project-init` | Automated setup (GitHub + Vercel) |
| Need authentication | `auth-setup` | Quick NextAuth/Clerk setup |
| Old/messy code | `legacy-refactor` | Safe incremental improvements |
| Designing system | `system-architect` | Scalable architecture patterns |
| Need subscriptions | `subscription-builder` | Stripe/Square billing integration |
| API development | `api-designer` | REST best practices |
| Debugging errors | `debug-protocol` | Systematic root cause analysis |
| Building UI | `web-builder` | SuperDesign + ShadCN components |
| Payment processing | `square-integration` | Complete Square implementation |
| Deployment issues | `deployment-manager` | Vercel troubleshooting |
| Code quality | `code-enforcer` | Zero-tolerance standards |
| Next.js development | `nextjs-pro` | App Router best practices |

---

## 🎨 Skill Categories by Use Case

### 🆕 Starting a New Project
```
1. system-architect    → Design the architecture
2. project-init        → Set up infrastructure
3. auth-setup          → Add authentication
4. subscription-builder → Add billing (if SaaS)
5. deployment-manager  → Deploy to production
```

### 🐛 Fixing Performance Issues
```
1. debug-protocol      → Diagnose the problem
2. database-optimizer  → Optimize queries
3. api-designer        → Improve API design
4. code-enforcer       → Ensure quality
```

### 🔧 Maintaining Legacy Code
```
1. legacy-refactor     → Plan safe refactoring
2. code-enforcer       → Add tests and types
3. api-designer        → Modernize APIs
4. database-optimizer  → Update queries
```

### 💰 Adding Payments
```
Option A (Stripe):     subscription-builder
Option B (Square):     square-integration
Both support:          One-time + recurring payments
```

---

## 📊 Skill Difficulty Levels

**⭐ Beginner** - Ready to use, minimal configuration
- theme-factory, brand-guidelines, internal-comms

**⭐⭐ Intermediate** - Some setup required
- nextjs-pro, auth-setup, web-builder, code-enforcer

**⭐⭐⭐ Advanced** - Complex implementation
- database-optimizer, api-designer, subscription-builder, square-integration

**⭐⭐⭐⭐ Expert** - Architecture-level decisions
- system-architect, legacy-refactor (large codebases)

---

## 🛠️ Creating Your Own Skills

Each skill is just a folder with a `SKILL.md` file:

```markdown
---
name: my-custom-skill
description: What it does and when to use it
license: Apache-2.0
---

# My Custom Skill

[Your instructions here - Claude will follow these]

## When to Use
- Use case 1
- Use case 2

## Examples
```example
User request → Expected behavior
```
```

**Steps to Create:**
1. Create folder: `mkdir my-skill`
2. Add `SKILL.md` with YAML frontmatter
3. Write clear instructions
4. Test with Claude
5. Share with the community!

See the [skill-creator](./skill-creator/) skill for detailed guidance.

---

## 🤝 Contributing

We welcome contributions! Here's how:

1. **Fork this repository**
2. **Create a new skill** or improve existing ones
3. **Test thoroughly** with Claude
4. **Submit a Pull Request**

### Contribution Guidelines

- ✅ Include clear `SKILL.md` with examples
- ✅ Add to appropriate category in `marketplace.json`
- ✅ Update README with skill description
- ✅ Follow existing naming conventions
- ✅ Test skill before submitting

---

## 📈 Roadmap

- [ ] Video tutorials for complex skills
- [ ] Interactive skill playground
- [ ] Community-contributed skills section
- [ ] Skill templates generator
- [ ] Integration with popular IDEs
- [ ] Automated skill testing

---

## 💬 Support & Community

- **Issues**: [GitHub Issues](https://github.com/mrmoe28/super-claude-skills/issues)
- **Discussions**: [GitHub Discussions](https://github.com/mrmoe28/super-claude-skills/discussions)
- **Documentation**: [Claude Skills Guide](https://support.claude.com/en/articles/12512198-creating-custom-skills)

---

## 📜 License

This project is licensed under **Apache 2.0**.

- ✅ Commercial use allowed
- ✅ Modification allowed
- ✅ Distribution allowed
- ℹ️ Must include license notice

**Document Skills** (xlsx, docx, pptx, pdf) are source-available with different licensing terms.

---

## 🌟 Credits

### Skills Sources
- **Anthropic Skills**: Official example skills from [anthropics/skills](https://github.com/anthropics/skills)
- **Custom Workflow Skills**: Original implementations by the community

### Special Thanks
- Anthropic team for the Skills system
- Claude Code team for marketplace support
- All contributors and users

---

## 📊 Statistics

<div align="center">

| Metric | Count |
|--------|-------|
| **Total Skills** | 24 |
| **Categories** | 5 |
| **Workflow Skills** | 13 |
| **Lines of Documentation** | 4,000+ |
| **Supported Platforms** | Stripe, Square, Vercel, Neon |

</div>

---

<div align="center">

**Made with ❤️ by the community**

⭐ **Star this repo** if you find it useful!

[Report Bug](https://github.com/mrmoe28/super-claude-skills/issues) • [Request Feature](https://github.com/mrmoe28/super-claude-skills/issues) • [Documentation](https://support.claude.com/en/articles/12512198-creating-custom-skills)

</div>
