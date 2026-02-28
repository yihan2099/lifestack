---
name: onboard
description: Interactive interview that creates user profile and configures lifestack domains
argument-hint: "[--reset] [--domain finance|social|health]"
metadata:
  domain: core
  permission-tier: read-write
  openclaw: {"emoji": "👋", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Read-write skills may create and modify files in the .lifestack/ directory.
All write operations create checkpoints before modifying existing files.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the command arguments:

- `--reset` flag: If present, back up existing profile before overwriting
- `--domain <name>`: If present, only onboard for the specified domain (finance, social, or health)
- No arguments: Run full interactive onboarding for all domains

Validate:
- If `--domain` is provided, confirm it is one of: `finance`, `social`, `health`
- If `--reset` is provided, check that `.lifestack/profile.md` exists (warn if nothing to reset)

## Phase 1: Welcome

Display a welcome message explaining what lifestack is and what onboarding will do:

- Explain that lifestack is a personal life management system powered by AI skills
- List the available domains: finance, social, health
- Explain that the interview will ask about goals and preferences for each domain
- If `--reset` flag is set, inform the user that existing profile will be backed up

## Phase 2: Backup (if --reset)

If `--reset` flag is present and `.lifestack/profile.md` exists:

1. Call the `checkpoint` skill: `checkpoint auto .lifestack/profile.md`
2. Call the `checkpoint` skill: `checkpoint auto .lifestack/config.md`
3. Confirm backup was created before proceeding

## Phase 3: Interview

Ask the user about their goals and preferences. For each active domain, ask targeted questions:

**General questions (always ask):**
- What should lifestack call you? (name/alias)
- What is your primary goal for using lifestack?
- How often do you want check-ins? (daily, weekly, monthly)

**Finance domain:**
- What are your top 3 financial goals? (e.g., save for house, pay off debt, invest)
- What is your monthly budget target?
- Do you want spending alerts? (yes/no, threshold amount)

**Social domain:**
- Who are the key people you want to stay in touch with? (names, relationship)
- How often do you want relationship reminders? (weekly, biweekly, monthly)
- Any important dates to track? (birthdays, anniversaries)

**Health domain:**
- What are your health goals? (e.g., exercise 3x/week, sleep 8hrs, meditate)
- Do you want daily health check-ins? (yes/no)
- Any metrics to track? (weight, steps, water intake)

If `--domain` flag limits scope, only ask questions for that domain plus the general questions.

## Phase 4: Create Profile

Write `.lifestack/profile.md` with the interview responses:

```markdown
# Lifestack Profile

- **Name**: {name}
- **Primary Goal**: {goal}
- **Check-in Frequency**: {frequency}
- **Created**: {YYYY-MM-DD HH:MM}
- **Last Updated**: {YYYY-MM-DD HH:MM}

## Finance Goals
{finance responses, or "Domain not activated" if skipped}

## Social Goals
{social responses, or "Domain not activated" if skipped}

## Health Goals
{health responses, or "Domain not activated" if skipped}
```

## Phase 5: Create Config

Write `.lifestack/config.md` with active domains and settings:

```markdown
# Lifestack Configuration

## Active Domains
- finance: {true/false}
- social: {true/false}
- health: {true/false}

## Settings
- check-in-frequency: {daily|weekly|monthly}
- notification-style: {minimal|standard|verbose}
- auto-checkpoint: true
- audit-logging: true

## Domain Settings

### Finance
- budget-alerts: {true/false}
- budget-threshold: {amount or "none"}

### Social
- reminder-frequency: {weekly|biweekly|monthly}
- track-dates: {true/false}

### Health
- daily-checkin: {true/false}
- tracked-metrics: {list or "none"}
```

## Phase 6: Activate Domains

For each active domain:
1. Create the domain directory if it doesn't exist: `.lifestack/{domain}/`
2. Create an initial state file: `.lifestack/{domain}/state.md` with default structure
3. Log the activation via audit-trail: `audit-trail log "Activated {domain} domain"`

## Phase 7: Summary

Log the onboarding via audit-trail: `audit-trail log "Onboarding completed for domains: {list}"`

## Output

Return a structured summary:

```
onboard complete

profile: .lifestack/profile.md
config:  .lifestack/config.md
domains: [list of activated domains]
backup:  {checkpoint ID if --reset was used, otherwise "n/a"}
```
