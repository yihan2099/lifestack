---
name: cross-post
description: Adapt and publish content across multiple social media platforms
argument-hint: "<prepare|post|schedule> [--platform twitter|linkedin|substack] [--date YYYY-MM-DD]"
metadata:
  domain: social
  permission-tier: approval-required
  openclaw: {"emoji": "🔄", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: approval-required.
Posting to social media is IRREVERSIBLE once published. This skill will ALWAYS show a full preview and require explicit confirmation before posting.
Every post and schedule command requires the user to type "yes" after reviewing the full preview.
No content is published or scheduled without explicit user approval.
Social media credentials are NEVER stored in lifestack data files.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

| Token | Meaning | Default |
|-------|---------|---------|
| `prepare` | Adapt content for multiple platforms | — |
| `post` | Publish prepared content to a platform | — |
| `schedule` | Add prepared content to calendar for future posting | — |
| `--platform <p>` | Target platform(s) (twitter, linkedin, substack, comma-separated) | all |
| `--date <d>` | Scheduled date for posting | today |
| `--content <c>` | The source content to adapt | — |
| `--thread` | Format Twitter content as a thread | false |
| `--tone <t>` | Tone adjustment (professional, casual, technical) | match platform defaults |

If no command is provided, show usage help.

Validate:
- Command must be one of: prepare, post, schedule
- Platform must be one of: twitter, linkedin, substack (or comma-separated list)
- Date must be valid YYYY-MM-DD format and not in the past (for schedule)
- Content must be provided for prepare, or a prepared post must exist for post/schedule

## Phase 1: Load Posts Data

Read the posts data file at `.lifestack/data/social/posts.md`.

If the file does not exist, create it with this header:

```markdown
# Cross-Post Content

## Prepared Posts

<!-- Each prepared post block follows this format -->
<!-- ### post-{YYYY-MM-DD}-{n} -->
<!-- **Source:** original content summary -->
<!-- **Status:** prepared|posted|scheduled -->
<!-- #### twitter -->
<!-- content here -->
<!-- #### linkedin -->
<!-- content here -->
<!-- #### substack -->
<!-- content here -->
```

Parse existing prepared posts into structured data.

## Phase 2: Execute Command

### prepare

Adapt source content for target platforms:

1. Require `--content` (inline text or reference to a file/post)
2. For each target platform, generate an adapted version:

**Twitter:**
- 280 character limit per tweet
- If `--thread`, split into numbered thread (1/N format)
- Use concise, punchy language
- Include relevant hashtags (max 2-3)
- End with a call to action or question

**LinkedIn:**
- Professional tone
- Open with a hook line (separated by line break)
- Use line breaks for readability
- 1300 character sweet spot (under 3000 max)
- End with a question to drive comments
- Include 3-5 relevant hashtags at the end

**Substack:**
- Newsletter-friendly format
- Can be longer form
- Include a compelling subject line
- Structure with headers if longer content
- Personal, conversational tone

3. Present ALL adapted versions in a single preview
4. Do NOT save until user confirms

### post

Publish prepared content to a platform:

1. Require: platform, and a prepared post ID (or use most recent)
2. **MANDATORY APPROVAL GATE:**
   ```
   ⚠️  POSTING PREVIEW — REVIEW CAREFULLY
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Platform: {platform}
   Content:
   ---
   {full content to be posted}
   ---

   This action is IRREVERSIBLE once published.
   Type "yes" to confirm posting, or "no" to cancel:
   ```
3. Only proceed if user explicitly types "yes"
4. If "no" or anything else, abort and confirm cancellation
5. On confirmation, invoke the appropriate platform posting mechanism
6. Update post status to "posted" with timestamp

### schedule

Add prepared content to calendar for future posting:

1. Require: platform, date, and a prepared post ID (or use most recent)
2. Validate date is in the future
3. Show preview of what will be scheduled:
   ```
   📋 SCHEDULE PREVIEW
   ━━━━━━━━━━━━━━━━━━
   Platform: {platform}
   Date: {date}
   Content:
   ---
   {content preview}
   ---

   Add to content calendar? (yes/no):
   ```
4. On confirmation, add entry to `.lifestack/data/social/calendar.md` with status `scheduled`
5. Update post status to "scheduled"

## Phase 3: Persist Changes

For all commands that modify data:

1. Write updated posts data to `.lifestack/data/social/posts.md`
2. For `schedule`, also update `.lifestack/data/social/calendar.md`
3. Maintain markdown formatting and structure

## Output

### prepare
```
Cross-post prepared for {n} platforms:

━━━ Twitter ━━━━━━━━━━━━━━━━━━━━━━━━━━━
{twitter version}
Characters: {count}/280
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

━━━ LinkedIn ━━━━━━━━━━━━━━━━━━━━━━━━━━
{linkedin version}
Characters: {count}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

━━━ Substack ━━━━━━━━━━━━━━━━━━━━━━━━━━
Subject: {subject line}
{substack version}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Saved as: post-{id}
Ready to post or schedule. Use `cross-post post` or `cross-post schedule`.
```

### post
```
Posted to {platform}:
  Post ID: {post_id}
  Timestamp: {datetime}
  Status: published
```

### schedule
```
Scheduled for {platform}:
  Post ID: {post_id}
  Date: {date}
  Status: scheduled
  Added to content calendar.
```

Log the action to `.lifestack/audit/YYYY-MM-DD.md`:
```
- HH:MM — social/cross-post {command} — {platform}: {content_summary} [APPROVAL: {yes/no}]
```
