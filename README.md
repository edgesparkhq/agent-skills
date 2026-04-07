# EdgeSpark Agent Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

Official agent skills by EdgeSpark.

Agent Skills to help developers using AI agents with EdgeSpark. Agent Skills are
folders of instructions, scripts, and resources that agents can discover and use
to do things more accurately and efficiently. Compatible with 37+ AI agents
including Claude Code, GitHub Copilot, Cursor, Codex, and many others.

The skills in this repo follow the [Agent Skills](https://agentskills.io/)
format.

## Installation

### Install all skills

```bash
npx skills add edgesparkhq/agent-skills
```

### Install a specific skill

```bash
npx skills add edgesparkhq/agent-skills --skill building-edgespark-apps
```

## Available Skills

<details>
<summary><strong>building-edgespark-apps</strong></summary>

Comprehensive EdgeSpark development skill covering full-stack app development
with database, auth, storage, and deployment workflows.

**Use when:**

- Working with a project that has `edgespark.toml`
- Using the `edgespark` CLI, server SDK types, or `@edgespark/web`
- Configuring auth providers, storage buckets, or database schemas
- Deploying or debugging EdgeSpark applications
- Working with Hono routes, Drizzle ORM schemas, or presigned upload flows

</details>

## Usage

Skills are automatically available once installed. The agent will use them when
relevant tasks are detected.

**Examples:**

```
Add a new database-backed API endpoint
```

```
Set up GitHub OAuth for my EdgeSpark app
```

```
Help me implement file uploads with presigned URLs
```

```
Deploy my EdgeSpark app to production
```

## Skill Structure

Each skill follows the [Agent Skills Open Standard](https://agentskills.io/):

- `SKILL.md` - Required skill manifest with frontmatter (name, description, metadata)
- `references/` - (Optional) Reference files for detailed documentation

## License

MIT. See [LICENSE](./LICENSE).
