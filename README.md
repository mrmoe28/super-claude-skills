# Super Claude Skills

A curated collection of Claude AI skills from various sources, organized and ready to use.

## What are Skills?

Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Skills teach Claude how to complete specific tasks in a repeatable way.

## Installation

### Claude Code

Install this marketplace by running:

```bash
/plugin marketplace add mrmoe28/super-claude-skills
```

Or manually:

```bash
cd ~/.claude/plugins/marketplaces
git clone https://github.com/mrmoe28/super-claude-skills.git
```

After installation, you can use skills by mentioning them. For example:
- "use the pdf skill to extract text from this file"
- "create an Excel spreadsheet with the xlsx skill"
- "design a canvas with the canvas-design skill"

### Claude.ai

Upload individual skill folders to Claude.ai following the [official instructions](https://support.claude.com/en/articles/12512180-using-skills-in-claude).

## Available Skills

### 📄 Document Skills
Professional document creation and editing tools:

- **xlsx** - Create, edit, and analyze Excel spreadsheets with formulas, formatting, and data analysis
- **docx** - Create, edit, and analyze Word documents with tracked changes, comments, and formatting
- **pptx** - Create, edit, and analyze PowerPoint presentations with layouts, templates, and charts
- **pdf** - Comprehensive PDF toolkit for extraction, creation, merging, splitting, and form handling

### 🎨 Creative Skills
Design and artistic tools:

- **canvas-design** - Design beautiful visual art in PNG and PDF formats using design philosophies
- **algorithmic-art** - Create generative art using p5.js with seeded randomness and particle systems
- **slack-gif-creator** - Create animated GIFs optimized for Slack's size constraints
- **theme-factory** - Style artifacts with pre-set professional themes or generate custom themes
- **brand-guidelines** - Apply consistent brand colors and typography to artifacts

### 💻 Development Skills
Technical and development tools:

- **artifacts-builder** - Build complex claude.ai HTML artifacts using React, Tailwind CSS, and shadcn/ui
- **mcp-builder** - Guide for creating high-quality MCP servers to integrate external APIs
- **webapp-testing** - Test local web applications using Playwright for UI verification and debugging
- **skill-creator** - Guide for creating effective skills that extend Claude's capabilities

### 🏢 Enterprise Skills
Business and communication tools:

- **internal-comms** - Write internal communications like status reports, newsletters, and FAQs

## Usage Examples

### Document Creation
```
"Create an Excel spreadsheet with Q4 sales data using the xlsx skill"
"Generate a Word document with our product specifications using the docx skill"
"Extract all form fields from contract.pdf using the pdf skill"
```

### Creative Work
```
"Design a hero image for our website using the canvas-design skill"
"Create a generative art piece with flowing particles using the algorithmic-art skill"
"Make a celebration GIF for Slack using the slack-gif-creator skill"
```

### Development
```
"Build an interactive dashboard artifact using the artifacts-builder skill"
"Create an MCP server for the GitHub API using the mcp-builder skill"
"Test the login flow on localhost:3000 using the webapp-testing skill"
```

## Creating Your Own Skills

Each skill is a folder containing a `SKILL.md` file with:

```markdown
---
name: my-skill-name
description: A clear description of what this skill does
---

# My Skill Name

[Instructions that Claude will follow]

## Examples
- Example usage patterns

## Guidelines
- Best practices and tips
```

For more details, see the [skill-creator](./skill-creator/) skill or check out [Anthropic's documentation](https://support.claude.com/en/articles/12512198-creating-custom-skills).

## Credits

Skills in this repository are sourced from:
- [Anthropic Skills](https://github.com/anthropics/skills) - Official example skills from Anthropic

## License

Skills are licensed under their respective licenses:
- Most example skills: Apache 2.0
- Document skills: Source-available (see individual skill folders for details)

## Contributing

To add new skills to this marketplace:

1. Fork this repository
2. Add your skill folder with a `SKILL.md` file
3. Update `.claude-plugin/marketplace.json` to include your skill
4. Submit a pull request

## Support

- [What are skills?](https://support.claude.com/en/articles/12512176-what-are-skills)
- [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [Creating custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)

## Version

1.0.0 - Initial release with skills from Anthropic's official repository
