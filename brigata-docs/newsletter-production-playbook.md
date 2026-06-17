---
brigata_id: a56d142d-678a-4f50-a255-3740459f3152
title: "newsletter-production-playbook"
---

# Newsletter Production Playbook

## Configuration (set per newsletter)
```
TOPIC:           e.g. "AI in IT consulting" or "manufacturing automation"
AUDIENCE:        internal | client-facing
BRAND_BG:        background color hex (e.g. #ffffff for light, #1a1a2e for dark)
BRAND_PRIMARY:   header/banner color hex
BRAND_ACCENT:    link and highlight color hex
FROM_NAME:       e.g. "The Forge" or "Ferox Insights"
FROM_EMAIL:      sender address
RECIPIENTS:      list or distribution method
CADENCE:         daily | weekly | biweekly
MIN_ARTICLES:    minimum quality articles required to publish (default: 4)
```

---

## Research Workflow

### Step 1 — Find candidates (Tavily)
Run 2–3 search queries covering the topic beat. Use date-qualified queries
when freshness filtering isn't available (e.g. "AI in manufacturing May 2026").
Collect 8–12 candidate URLs. Discard obvious vendor homepages and listicles.

### Step 2 — Read full articles (Jina Reader)
For each candidate URL, fetch full content:
```
GET https://r.jina.ai/{article_url}
```
Extract from the body:
- Publication name and date
- Author name (if bylined)
- Key statistics, percentages, or dollar figures
- Named companies or people doing something notable
- One direct quote if available

Do not summarize from snippets. If Jina can't fetch it, skip the article.

### Step 3 — Score and gate
Rate each article:

| Tier | Source type | Examples |
|------|-------------|---------|
| 1 | Trade press, major news, research firms | Reuters, WSJ, Gartner, McKinsey |
| 2 | Industry blogs, association publications | NIST, SME, AME, sector trade mags |
| 3 | Vendor blogs, PR releases | Company announcements, product launches |

**Quality gate:** Proceed only if:
- ≥ MIN_ARTICLES articles are Tier 1 or 2
- ≤ 1 article is Tier 3
- All selected articles have at least one specific data point (number, name, or quote)

If gate fails: log the skip date and reason, do not send.

---

## Content Structure

### Issue template (5 components)
1. **Header banner** — newsletter name, issue number, date
2. **Lede** (2–3 sentences) — what's the signal in this issue? The one thing
   worth knowing this period.
3. **Article cards** (3–5) — each contains:
   - Headline (linked to source)
   - Source name + publish date
   - 3–4 sentence summary with at least one specific detail
4. **Footer** — branding, unsubscribe link, sending agent disclosure if applicable
5. **Optional: one-line editorial note** — something the data suggests that the
   articles don't say outright. The "so what."

### Tone rules
- Name your sources: "Per Redwood Software's survey of 300 manufacturers..."
  not "A recent study found..."
- Avoid: "revolutionary," "game-changing," "unprecedented," "cutting-edge"
- Use: specific numbers, named organizations, concrete outcomes
- One voice throughout — no register shifts between sections

---

## HTML Email Format

### Design principles
- Table-based layout only (Outlook/Gmail compatible)
- All CSS inline — no `<style>` blocks, no external sheets
- Max width: 640px, centered
- Mobile-friendly: single column, 16px+ body font
- Images optional — design must read well without them

### Color variables (swap per CONFIGURATION block above)
```
background:     BRAND_BG
header:         BRAND_PRIMARY
accent/links:   BRAND_ACCENT
body text:      #e0e0e0 (dark theme) or #333333 (light theme)
card background:#16213e (dark) or #f5f5f5 (light)
```

### Skeleton structure
```html
<!-- Outer wrapper -->
<table width="100%" bgcolor="[BRAND_BG]">
  <tr><td align="center">
    <table width="640" cellpadding="0" cellspacing="0">

      <!-- Header banner -->
      <tr><td bgcolor="[BRAND_PRIMARY]" style="padding:32px;text-align:center;">
        <h1 style="color:#ffffff;font-family:Georgia,serif;margin:0;">
          [NEWSLETTER NAME]
        </h1>
        <p style="color:#cccccc;margin:8px 0 0;">Issue #[N] · [DATE]</p>
      </td></tr>

      <!-- Lede -->
      <tr><td style="padding:24px 32px;font-family:Arial,sans-serif;
                     font-size:15px;color:[BODY_TEXT];">
        [LEDE TEXT]
      </td></tr>

      <!-- Article card (repeat 3–5x) -->
      <tr><td style="padding:0 32px 16px;">
        <table width="100%" bgcolor="[CARD_BG]" style="border-radius:6px;">
          <tr><td style="padding:20px;">
            <a href="[URL]" style="color:[BRAND_ACCENT];font-size:16px;
               font-weight:bold;font-family:Arial,sans-serif;
               text-decoration:none;">[HEADLINE]</a>
            <p style="color:#888888;font-size:12px;margin:4px 0 8px;
               font-family:Arial,sans-serif;">
               [SOURCE] · [DATE]
            </p>
            <p style="color:[BODY_TEXT];font-size:14px;line-height:1.6;
               font-family:Arial,sans-serif;margin:0;">
               [SUMMARY WITH SPECIFIC DETAIL]
            </p>
          </td></tr>
        </table>
      </td></tr>

      <!-- Footer -->
      <tr><td style="padding:24px 32px;text-align:center;
                     font-family:Arial,sans-serif;font-size:12px;
                     color:#666666;">
        [BRANDING LINE] · [UNSUBSCRIBE LINK IF APPLICABLE]
      </td></tr>

    </table>
  </td></tr>
</table>
```

---

## State Management

Maintain two files per newsletter instance:
- `[newsletter-slug]-used-urls.txt` — one URL per line, checked before including
  any article to prevent repeats across issues
- `[newsletter-slug]-issue-counter.txt` — integer, increment on each sent issue

If quality gate fails, write a skip entry to a third file:
- `[newsletter-slug]-skip-log.txt` — date + reason (e.g. "2026-05-28: only 2 tier-1
  articles found, threshold is 4")

---

## Checklist Before Sending

- [ ] All articles read via Jina (not just snippet-summarized)
- [ ] Quality gate passed (≥ MIN_ARTICLES tier 1/2 sources)
- [ ] No URLs repeated from used-urls file
- [ ] Every summary contains at least one specific detail
- [ ] HTML renders correctly (test in both light and dark email clients if possible)
- [ ] Issue counter incremented
- [ ] Used URLs appended to state file
