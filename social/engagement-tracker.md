---
name: engagement-tracker
description: Log and analyze social media engagement metrics across platforms
argument-hint: "<log|summary|trend> [--platform twitter|linkedin|substack|all] [--period week|month|quarter]"
metadata:
  domain: social
  permission-tier: read-write
  openclaw: {"emoji": "📊", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-write.
Read-write skills may create and modify data files in .lifestack/data/social/.
No data is deleted without explicit user confirmation.
Social media credentials are NEVER stored in lifestack data files.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

| Token | Meaning | Default |
|-------|---------|---------|
| `log` | Record engagement metrics for a post | — |
| `summary` | Show engagement summary for a period | — |
| `trend` | Analyze engagement trends over time | — |
| `--platform <p>` | Target platform (twitter, linkedin, substack, all) | all |
| `--period <p>` | Time period (week, month, quarter) | month |
| `--date <d>` | Date for the log entry | today |
| `--post-id <id>` | Identifier for the specific post | — |
| `--likes <n>` | Number of likes | 0 |
| `--comments <n>` | Number of comments | 0 |
| `--shares <n>` | Number of shares/retweets/reposts | 0 |
| `--impressions <n>` | Number of impressions/views | 0 |
| `--followers <n>` | Current follower count (snapshot) | — |

If no command is provided, default to `summary --period month`.

Validate:
- Command must be one of: log, summary, trend
- Platform must be one of: twitter, linkedin, substack, all
- Numeric fields must be non-negative integers
- Date must be valid YYYY-MM-DD format

## Phase 1: Load Metrics Data

Read the metrics data file at `.lifestack/data/social/metrics.md`.

If the file does not exist, create it with this header:

```markdown
# Social Media Engagement Metrics

| date | platform | post_id | likes | comments | shares | impressions |
|------|----------|---------|-------|----------|--------|-------------|
```

Also check for a follower snapshot section at the end of the file:

```markdown
## Follower Snapshots

| date | platform | followers |
|------|----------|-----------|
```

Parse both tables into structured data.

## Phase 2: Execute Command

### log

Record engagement metrics for a post:

1. Require at least: platform, and at least one metric (likes, comments, shares, or impressions)
2. Use `--date` or default to today
3. Use `--post-id` if provided, otherwise generate from date + platform
4. Append a row to the metrics table
5. If `--followers` is provided, also append to the follower snapshots table

### summary

Generate an engagement summary:

1. Filter metrics by `--platform` and `--period`
2. Calculate aggregates:
   - Total likes, comments, shares, impressions
   - Average engagement per post (likes + comments + shares) / post count
   - Top-performing post (highest total engagement)
   - Engagement rate: (likes + comments + shares) / impressions * 100
3. If follower data exists, show follower change over the period
4. Present as a formatted report

### trend

Analyze engagement trends:

1. Filter metrics by `--platform` and `--period`
2. Group data by week
3. Calculate week-over-week changes for each metric
4. Identify:
   - Best-performing week
   - Best-performing content type (if post_id contains type info)
   - Growth or decline trend direction
   - Average engagement trajectory (improving, stable, declining)
5. Generate actionable recommendations based on trends

## Phase 3: Persist Changes

For `log` command:

1. Write updated metrics data back to `.lifestack/data/social/metrics.md`
2. Maintain the markdown table format
3. Keep entries sorted by date descending (newest first)

## Output

### log
```
Logged engagement metrics:
  Date: {date}
  Platform: {platform}
  Post: {post_id}
  Likes: {likes} | Comments: {comments} | Shares: {shares} | Impressions: {impressions}
```

### summary
```
Engagement Summary — {platform} ({period})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Posts tracked: {count}
Total impressions: {impressions}
Total engagement: {likes + comments + shares}
Avg engagement/post: {avg}
Engagement rate: {rate}%

Top post: {post_id} ({total_engagement} engagements)

Followers: {current} ({change:+/-} over period)
```

### trend
```
Engagement Trends — {platform} ({period})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| week | likes | comments | shares | impressions | eng_rate |
|------|-------|----------|--------|-------------|----------|
| ...  | ...   | ...      | ...    | ...         | ...      |

Trend: {improving|stable|declining}
Best week: {week} ({reason})

Recommendations:
- {recommendation_1}
- {recommendation_2}
- {recommendation_3}
```

Log the action to `.lifestack/audit/YYYY-MM-DD.md`:
```
- HH:MM — social/engagement-tracker {command} — {summary of action}
```
