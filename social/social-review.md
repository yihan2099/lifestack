---
name: social-review
description: Generate comprehensive weekly social media performance review
argument-hint: "weekly [--period YYYY-MM-DD:YYYY-MM-DD]"
metadata:
  domain: social
  permission-tier: read-only
  openclaw: {"emoji": "📋", "requires": {"bins": []}}
---

<!-- SAFETY PREAMBLE — DO NOT SUMMARIZE, COMPACT, OR REMOVE -->
This skill operates at permission tier: read-only.
Read-only skills do not create, modify, or delete any data files.
This skill only reads from .lifestack/data/social/ to generate reports.
Social media credentials are NEVER stored in lifestack data files.
All actions are logged to .lifestack/audit/YYYY-MM-DD.md.
<!-- END SAFETY PREAMBLE -->

## Input Parsing

Parse the user's command and arguments:

| Token | Meaning | Default |
|-------|---------|---------|
| `weekly` | Generate weekly social media review | — |
| `--period <start:end>` | Custom date range (YYYY-MM-DD:YYYY-MM-DD) | last 7 days |
| `--format <f>` | Output format (summary, detailed) | detailed |

If no command is provided, default to `weekly` for the last 7 days.

Validate:
- Command must be: weekly
- Period dates must be valid YYYY-MM-DD format
- Start date must be before end date
- Range should not exceed 14 days (for weekly review scope)

## Phase 1: Load All Social Data

Read all social data files (all read-only):

1. `.lifestack/data/social/calendar.md` — content calendar entries
2. `.lifestack/data/social/metrics.md` — engagement metrics and follower snapshots
3. `.lifestack/data/social/posts.md` — cross-posted content

If any files are missing, note what data is unavailable and generate the review with available data only. Suggest which skills to use to start collecting the missing data.

## Phase 2: Compile Weekly Review

Gather data for the review period (default: last 7 days) and compile each section:

### Section 1: Posts Published

1. Count posts published per platform during the review period
2. List each post with:
   - Date published
   - Platform
   - Topic/content summary
   - Current engagement metrics (if logged)
3. Compare to previous week's posting frequency

### Section 2: Engagement Summary

1. Aggregate engagement metrics across all platforms for the period:
   - Total likes, comments, shares, impressions
   - Average engagement rate per platform
   - Total engagement change vs previous period
2. Highlight any significant changes (>20% increase or decrease)

### Section 3: Follower Changes

1. Compare follower snapshots from start and end of period
2. Calculate net change per platform
3. Calculate growth rate percentage
4. Note any unusual spikes or drops

### Section 4: Top-Performing Content

1. Rank all posts from the period by total engagement (likes + comments + shares)
2. Show the top 3 posts with:
   - Platform and date
   - Content summary
   - Engagement breakdown
   - What likely made it perform well (timing, topic, format)
3. If fewer than 3 posts, show all available

### Section 5: Upcoming Calendar

1. Show content calendar entries for the next 7 days
2. Include: date, platform, topic, status
3. Flag any days without scheduled content
4. Flag any platforms with no upcoming content

### Section 6: Recommendations

Generate 3-5 actionable recommendations for the coming week based on:
- What content performed best (do more of this)
- What content underperformed (adjust or avoid)
- Gaps in posting schedule (fill these)
- Platform-specific opportunities
- Engagement trend direction (capitalize or course-correct)

## Output

```
Weekly Social Media Review
{start_date} — {end_date}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Posts Published

| date | platform | topic | likes | comments | shares | impressions |
|------|----------|-------|-------|----------|--------|-------------|
| ...  | ...      | ...   | ...   | ...      | ...    | ...         |

Total: {count} posts ({comparison} vs previous week)
Breakdown: Twitter {n} | LinkedIn {n} | Substack {n}

## Engagement Summary

| platform | likes | comments | shares | impressions | eng rate |
|----------|-------|----------|--------|-------------|----------|
| twitter  | {n}   | {n}      | {n}    | {n}         | {n}%     |
| linkedin | {n}   | {n}      | {n}    | {n}         | {n}%     |
| substack | {n}   | {n}      | {n}    | {n}         | {n}%     |
| **total** | **{n}** | **{n}** | **{n}** | **{n}**   | **{n}%** |

Week-over-week: {change:+/-}% total engagement

## Follower Changes

| platform | start | end | change | growth |
|----------|-------|-----|--------|--------|
| twitter  | {n}   | {n} | {+/-n} | {n}%  |
| linkedin | {n}   | {n} | {+/-n} | {n}%  |
| substack | {n}   | {n} | {+/-n} | {n}%  |

## Top-Performing Content

1. **{post_title}** ({platform}, {date})
   {likes} likes | {comments} comments | {shares} shares | {impressions} impressions
   Why it worked: {analysis}

2. **{post_title}** ({platform}, {date})
   {likes} likes | {comments} comments | {shares} shares | {impressions} impressions
   Why it worked: {analysis}

3. **{post_title}** ({platform}, {date})
   {likes} likes | {comments} comments | {shares} shares | {impressions} impressions
   Why it worked: {analysis}

## Upcoming Calendar (Next 7 Days)

| date | platform | topic | status |
|------|----------|-------|--------|
| ...  | ...      | ...   | ...    |

{warnings about gaps if any}

## Recommendations for Next Week

1. {recommendation_1}
2. {recommendation_2}
3. {recommendation_3}
4. {recommendation_4}
5. {recommendation_5}
```

Log the action to `.lifestack/audit/YYYY-MM-DD.md`:
```
- HH:MM — social/social-review weekly — {start_date} to {end_date}
```
