# Fixer: Social Cross-Post

Common failure patterns and remediation steps.

## "Content too long for Twitter"

**Cause**: The Twitter-adapted content exceeds the 280 character limit.
**Fix**: Ensure the Twitter adaptation step in `cross-post.md` enforces the 280 character limit strictly. The adaptation should truncate or rephrase the content, not simply copy the full body. Verify character count includes the URL (URLs count as 23 characters on Twitter via t.co wrapping).

## "Missing platform format"

**Cause**: The cross-post prepare command did not generate output for all target platforms.
**Fix**: Check that `social/cross-post.md` has adaptation logic for all three platforms (Twitter, LinkedIn, Substack Notes). Each platform must have its own formatting rules defined. If a platform section is missing, the skill file needs to be updated with that platform's format spec.

## "Approval not shown"

**Cause**: The cross-post skill has an `approval-required` tier, meaning adapted content must be shown to the user for review before any posting action. The preview step was skipped.
**Fix**: Verify that the skill's permission tier enforces a preview/approval gate. The workflow must display all adapted content and wait for explicit approval before proceeding to any post action. If running in test mode, the preview output must still be generated even if posting is skipped.

## "URL missing from adapted content"

**Cause**: The adapted content for one or more platforms dropped the URL during formatting.
**Fix**: Ensure the cross-post adaptation preserves the URL in every platform version. For Twitter, the URL should be appended at the end. For LinkedIn and Substack, it can be embedded inline or as a footer link.

## "Encoding or special character issues"

**Cause**: Titles or body text containing quotes, ampersands, or emoji may break template substitution.
**Fix**: Ensure fixture input values are properly escaped before substitution into the run command template. Use raw strings or escape special characters in the body and title fields.
