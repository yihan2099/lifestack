---
name: audience-insights
description: Generate audience and performance insights from logged social media metrics
argument-hint: "<report|compare> [--platform twitter|linkedin|substack|all] [--period week|month|quarter]"
metadata:
  domain: social
  permission-tier: read-only
  openclaw: {"emoji": "🔍", "requires": {"bins": []}}
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
| `report` | Generate audience insights report for a platform | — |
| `compare` | Compare metrics across platforms or time periods | — |
| `--platform <p>` | Target platform (twitter, linkedin, substack, all) | all |
| `--period <p>` | Time period (week, month, quarter) | month |
| `--compare-period <p>` | Second period for comparison | previous equivalent period |
| `--metric <m>` | Focus metric (engagement, growth, impressions, all) | all |

If no command is provided, default to `report --platform all --period month`.

Validate:
- Command must be one of: report, compare
- Platform must be one of: twitter, linkedin, substack, all
- Period must be one of: week, month, quarter

## Phase 1: Load Data

Read the following data files (all read-only):

1. `.lifestack/data/social/metrics.md` — engagement metrics and follower snapshots
2. `.lifestack/data/social/posts.md` — post content and platform versions

If either file does not exist, report what data is missing and suggest using `engagement-tracker log` to start collecting metrics.

## Phase 2: Execute Command

### report

Generate a comprehensive audience insights report:

1. Filter data by `--platform` and `--period`
2. Calculate and present:

**Growth Metrics:**
- Follower count change over period (absolute and percentage)
- Follower growth rate (daily average)
- Projected follower count at current growth rate (30/60/90 days)

**Engagement Metrics:**
- Average engagement rate: (likes + comments + shares) / impressions * 100
- Engagement per post average
- Best engagement rate achieved
- Worst engagement rate

**Content Performance:**
- Top 3 performing posts (by total engagement)
- Bottom 3 performing posts
- Average impressions per post
- Content type breakdown (if post_id contains type indicators)

**Posting Patterns:**
- Posts per week average
- Most active posting day
- Estimated best posting times (based on highest-engagement posts)

**Recommendations:**
- Generate 3-5 actionable recommendations based on the data:
  - If engagement rate < 2%: suggest content format changes
  - If posting frequency is inconsistent: suggest a content calendar
  - If one metric significantly outperforms: suggest doubling down
  - If follower growth is stagnant: suggest engagement strategies
  - Compare against platform benchmarks (Twitter avg ~1-3%, LinkedIn ~2-5%)

### compare

Compare metrics across dimensions:

**Cross-platform comparison** (when --platform is "all" or multiple):
1. Show side-by-side metrics for each platform
2. Highlight which platform performs best on each metric
3. Calculate platform-specific engagement rates
4. Identify where audience overlap opportunities exist

**Time-period comparison** (when --compare-period is specified):
1. Show metrics for both periods side by side
2. Calculate percentage changes for each metric
3. Highlight improvements and declines
4. Identify what changed between periods (posting frequency, content type, etc.)

## Output

### report
```
Audience Insights Report — {platform} ({period})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📈 Growth
  Followers: {current} ({change:+/-} | {pct}% over {period})
  Daily growth rate: {rate}/day
  Projected (30d): {projected_30} | (90d): {projected_90}

💬 Engagement
  Posts tracked: {count}
  Total impressions: {impressions}
  Avg engagement rate: {rate}%
  Avg engagement/post: {avg}
  Best post: {post_id} ({engagement} engagements, {rate}% rate)

📝 Content
  Top performing:
    1. {post_1} — {engagement_1} engagements
    2. {post_2} — {engagement_2} engagements
    3. {post_3} — {engagement_3} engagements

📅 Patterns
  Avg posts/week: {freq}
  Best day: {day}
  Posting consistency: {consistent|inconsistent}

💡 Recommendations
  - {recommendation_1}
  - {recommendation_2}
  - {recommendation_3}
```

### compare
```
Platform Comparison — {period}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| metric | twitter | linkedin | substack | best |
|--------|---------|----------|----------|------|
| followers | {n} | {n} | {n} | {platform} |
| eng rate | {n}% | {n}% | {n}% | {platform} |
| impressions | {n} | {n} | {n} | {platform} |
| posts | {n} | {n} | {n} | {platform} |

Key Insights:
- {insight_1}
- {insight_2}
- {insight_3}
```

Log the action to `.lifestack/audit/YYYY-MM-DD.md`:
```
- HH:MM — social/audience-insights {command} — {platform}, {period}
```
