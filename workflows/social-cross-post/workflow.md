# Workflow: Social Cross-Post

## Run Command

```
You are testing the social/cross-post skill. Perform the following operations:

1. Take the following content and prepare it for cross-posting:
   - Title: "{{title}}"
   - URL: {{url}}
   - Body: "{{body}}"
   - CTA: "{{cta}}"

2. Adapt the content for each platform:
   - Twitter/X (280 character limit)
   - LinkedIn (professional tone, longer format)
   - Substack Notes (newsletter-friendly)

3. Preview each platform's adapted version and show the output

Use the social/cross-post.md skill for all operations. Report each platform's adapted content.
```

## Success Criteria

- Output contains Twitter-adapted content that is 280 characters or fewer
- Output contains LinkedIn-adapted content with professional formatting
- Output contains Substack Notes-adapted content
- Each platform version includes the URL {{url}}
- Each platform version preserves the core message from {{body}}
- Preview is shown for each platform before any posting action

## Fixtures

See `fixtures.md` for test data. Each fixture substitutes:
- `title` — content title
- `url` — link to include
- `body` — main content text
- `cta` — call-to-action text

## Constraints

- Timeout: 180000ms
- Max turns: 25
