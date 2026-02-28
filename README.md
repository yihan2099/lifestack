# lifestack

Composable markdown skills for AI agents to manage your life — finance, social media, health.

## What is lifestack?

lifestack is a collection of markdown-based skills that any AI agent can pick up and use to help manage everyday life tasks. Each skill is a self-contained `.md` file with structured frontmatter, safety guardrails, and a clear execution pattern.

Skills are organized by domain — finance, social media, health — and designed to compose together. An agent can run a single skill in isolation or chain multiple skills into workflows.

lifestack works with [OpenClaw](https://github.com/anthropics/openclaw)-compatible agents using the SKILL.md format, and with generic AI agents like Claude Code. No framework lock-in.

## Philosophy

**Safety-first.** Every skill declares a permission tier. Destructive operations require double confirmation. All actions are logged to an append-only audit trail. Safety preambles are protected from context compaction. This approach was inspired by gaps identified in OpenClaw's architecture — lifestack treats safety as a first-class design constraint, not an afterthought.

**Composable.** Skills are small, focused, and independent. They read and write to well-defined data paths. You can combine them into workflows without hidden coupling.

**Measurable.** Skills that track data (budgets, habits, mood) produce structured output that other skills can consume. Progress is visible and queryable.

**Dual-compatible.** Skills follow the OpenClaw SKILL.md frontmatter format so they work in OpenClaw-powered agents out of the box. They also work with any agent that can read markdown instructions — no special runtime required.

## Quickstart

```bash
# Clone the repo
git clone https://github.com/yihan2099/lifestack.git

# Run the onboard skill to set up your local data directory
# (Your agent reads the skill and follows the instructions)
lifestack/core/onboard.md
```

The onboard skill creates your `.lifestack/` directory with the required folder structure and walks you through initial configuration.

## Domains

### Finance

Budget tracking, expense categorization, subscription auditing, savings goals. All data stays local in `.lifestack/data/finance/`.

Skills: `budget-track`, `expense-report`, `subscription-audit`, `savings-goal`

### Social

Social media post drafting, scheduling, cross-platform publishing, engagement tracking. Posting requires `approval-required` permission — the agent always shows you a preview before publishing.

Skills: `draft-post`, `schedule-post`, `engagement-report`

### Health

Habit tracking, mood logging, sleep patterns, exercise logging. Health data never leaves your machine.

Skills: `habit-track`, `mood-log`, `sleep-track`, `exercise-log`, `weekly-review`

## Core Skills

Core skills handle lifestack's own infrastructure:

| Skill | Description |
|-------|-------------|
| `onboard` | Initialize `.lifestack/` directory and user config |
| `checkpoint` | Create/restore/list state snapshots |
| `audit-view` | Query the action audit trail |
| `skill-check` | Validate a skill file against the format spec |

## skill-optimizer Integration

lifestack skills are compatible with [skill-optimizer](https://github.com/yihan2099/skill-optimizer). After using a skill, skill-optimizer can:

- Track execution success/failure rates
- Identify skills that frequently need user corrections
- Suggest refinements to skill instructions based on usage patterns
- Auto-tune argument defaults based on your preferences

To enable integration, ensure skill-optimizer is available in your agent's skill path. lifestack skills include the `openclaw` metadata field for compatibility.

## Directory Structure

```
lifestack/
├── README.md
├── LICENSE
├── SAFETY.md
├── skill-format.md
├── core/
│   ├── onboard.md
│   ├── checkpoint.md
│   ├── audit-view.md
│   └── skill-check.md
├── finance/
│   ├── budget-track.md
│   ├── expense-report.md
│   ├── subscription-audit.md
│   └── savings-goal.md
├── social/
│   ├── draft-post.md
│   ├── schedule-post.md
│   └── engagement-report.md
└── health/
    ├── habit-track.md
    ├── mood-log.md
    ├── sleep-track.md
    ├── exercise-log.md
    └── weekly-review.md
```

User data lives outside the repo in a local `.lifestack/` directory:

```
.lifestack/
├── config.yaml
├── data/
│   ├── finance/
│   ├── social/
│   └── health/
├── audit/
│   └── YYYY-MM-DD.md
└── checkpoints/
```

## Safety Model

lifestack uses a 4-tier permission model:

| Tier | Can do | Example |
|------|--------|---------|
| `read-only` | Read data, generate reports | `expense-report`, `audit-view` |
| `read-write` | Create/update data files | `budget-track`, `mood-log` |
| `approval-required` | Modify external systems (preview first) | `draft-post`, `schedule-post` |
| `destructive` | Irreversible operations (double confirm) | Bulk delete, account unlinking |

Every action is logged to an append-only audit trail. Write operations create automatic checkpoints for rollback. See [SAFETY.md](SAFETY.md) for the full safety model.

## License

MIT License. See [LICENSE](LICENSE).
