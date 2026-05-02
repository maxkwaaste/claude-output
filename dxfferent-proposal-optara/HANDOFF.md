# Optara IT Glue Reimplementation -- Dxfferent Quote

## Status
Quote HTML is built and ready for review. Not yet deployed to dxfferent.com. Email draft ready with copy button.

## Files

| File | What |
|------|------|
| `index.html` | Full proposal -- dark theme, two-page flow (quote -> confirmation with acceptance form) |
| `email-joel.html` | Styled HTML email for Joel, white background, copy-to-clipboard button |
| `logos/` | Reference customer logos (see logo section below) |

## Prospect

- **Company**: Optara
- **Contact**: Joel Garcia, jgarcia@optara.com
- **Currency**: EUR
- **REF**: 2026-043
- **Source**: Deep dive call with Melissa (Dxfferent) who reviewed their IT Glue environment

## Pricing

- **EUR 30,000 one-time** -- IT Glue reimplementation project (50% at kick-off, 50% at go-live)
- **EUR 2,500/mo GaaS** -- Guide as a Service after go-live (3-month initial term, then monthly)
- Single option (no GaaS/BaaS split)

## Key context from Joel (transcript at ~/Downloads/Optara Dxfferent Deep Dive IT Glue.vtt)

- Joel is sales/business, not technical. Thinks in revenue and cost.
- "I'm losing money" -- Lifecycle integration pulls incorrect data from IT Glue, missing upsell opportunities
- "Are we using assets the right way? Topologies? Site summaries? SOPs/checklists?"
- "I'd rather pay someone rather than we have to spend 40, 80 hours"
- "It doesn't matter what it's going to take to fix it" -- price insensitive
- Committed to IT Glue: "the amount of time to fix it is better than switching ships"
- Has to sell this internally to operations manager and NetOps engineer
- Documentation spread across IT Glue, SharePoint, Teams

## Key context from Melissa

- Optara's IT Glue is "totaal niet ingericht, super basic, veel verouderde data"
- Timeline: 3-6 months
- Melissa has done 25+ IT Glue implementations
- Reference customers (via GaaS, not necessarily standalone IT Glue projects): Cubics, IT Synergy, Refa ICT, ICT&CO, possibly Aspect ICT

## Proposal structure

1. **Hero** -- Prepared for Optara
2. **Cover letter** -- References Joel's specific pain points from the call
3. **Stats** -- 150+ MSP implementations, 2-week sprints, 3-phase programme
4. **What We Found** -- 6 discovery cards (integrations pulling incorrectly, assets & configs, site summaries & topologies, docs spread across systems, SOPs & checklists, AI & automation readiness)
5. **What This Can Do For Your Business** -- 4 green-bordered impact cards (Lifecycle, tool ROI, technician efficiency, automation readiness). Hedged language ("can", "typically", "is designed to"). Callout references Joel's own words about time. References line: "25+ IT Glue implementations, references available on request"
6. **Our Approach** -- 5 steps (Review [Done], Kick-off, Implementation, Go-live, Optimize & Automate)
7. **Programme** -- 3 phases (Foundation, Build & Clean, Adoption & Automation)
8. **Investment** -- EUR 30k one-time + EUR 2,500/mo GaaS card
9. **Q&A** -- 5 questions
10. **Terms** -- Standard Dxfferent terms, payment within 14 days, Dutch law
11. **Page 2: Confirmation** -- Selection summary, "This is Dxfferent", what's included, programme recap, investment summary, T&C, acceptance form (FormSubmit to max@proxuma.com), signature block

## Logos -- TODO for next session

Downloaded to `logos/` directory. Need to be added as a reference customer logo row in the proposal.

### Best file per company

| Company | File | Dimensions | Background | Notes |
|---------|------|-----------|------------|-------|
| Cubics | `cubics.png` | 225x225 | White with transparency | Black "C" square icon. Needs inversion for dark bg, or place in light pill |
| IT Synergy | `itsynergy.png` | 4288x1026 | Transparent | Blue wordmark + icon. Works on dark bg directly. Huge file, resize to ~400px wide |
| Refa ICT | `refa.png` | 168x75 | Transparent | White logo. Works directly on dark bg. Small but usable |
| ICT&CO | `ictenco-og.png` | 54x74 | Transparent (black on transparent) | Stacked "ICT&CO." in rounded rectangle. Very small, needs upscaling or use as-is with padding. Needs inversion for dark bg |
| Aspect ICT | `aspect-ict.png` | 400x150 | White/colored | Full color logo with swirl + wordmark. Has its own background, works either way |

### Logo sources (for re-downloading at higher res if needed)

- Cubics: `https://www.cubics.nl/hubfs/Cubics%20logo.png` (favicon/icon)
- IT Synergy: `https://itsynergy.nl/wp-content/uploads/2024/01/IT-Synergy-logo.png` (full logo)
- Refa ICT: `https://www.refa.nl/img/refa_logo.png` (white on transparent)
- ICT&CO: `https://www.ictenco.nl/cache/e85c6a75a6d0075613dc1d5255730036/favicon.png` (tiny but real logo)
- Aspect ICT: `https://www.aspect-ict.nl/hs-fs/hubfs/Opzet%201.png` (full logo from HubSpot)

### Recommended approach for logo row

Add a band after the "references available on request" line. Gray or subtle treatment -- not too prominent since these were GaaS clients, not necessarily IT Glue-specific projects. Something like a row of grayscale/muted logos with "Selected clients our consultants have worked with" as a label. For the dark proposal theme, use CSS `filter: brightness(0) invert(1)` to make all logos white, or place them in light-colored rounded containers.

## Deploy instructions (when ready)

```bash
# Create directory
ssh -i ~/.ssh/id_ed25519_siteground_dxfferent -p 18765 u110-srp2wn2ngthp@ssh.dxfferent.com "mkdir -p /home/customer/www/dxfferent.com/public_html/proposal/optara-itglue/"

# Upload proposal
scp -i ~/.ssh/id_ed25519_siteground_dxfferent -P 18765 ~/ClaudeCode/claude-output/dxfferent-proposal-optara/index.html u110-srp2wn2ngthp@ssh.dxfferent.com:/home/customer/www/dxfferent.com/public_html/proposal/optara-itglue/index.html

# Upload .htaccess (noindex)
echo 'Header set X-Robots-Tag "noindex, nofollow"' | ssh -i ~/.ssh/id_ed25519_siteground_dxfferent -p 18765 u110-srp2wn2ngthp@ssh.dxfferent.com "cat > /home/customer/www/dxfferent.com/public_html/proposal/optara-itglue/.htaccess"

# Purge cache
ssh -i ~/.ssh/id_ed25519_siteground_dxfferent -p 18765 u110-srp2wn2ngthp@ssh.dxfferent.com "cd /home/customer/www/dxfferent.com/public_html && wp sg purge"
```

Live URL after deploy: `https://dxfferent.com/proposal/optara-itglue/`

## Still TODO

1. Add reference customer logo row to proposal (logos downloaded, need styling)
2. Update email CTA link after deploy (currently points to `dxfferent.com/proposal/optara-itglue/`)
3. Max review of final proposal
4. Deploy to dxfferent.com
5. Generate Joshua review message (Dutch, informal)
6. Log to Open Brain with prospect details
