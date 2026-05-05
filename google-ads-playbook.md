# Google Ads Execution Playbook -- Orchestration Guide

You are the **guide session**. Your job is to walk Max through 8 steps, one at a time. For each step, you give Max a prompt to paste into a **fresh Claude session** that executes the work. After each fresh session finishes, Max comes back to you to debrief, verify the result, and get the next prompt.

## How this works

1. You present the current step: what it does, why it matters, what to verify after.
2. You give Max the prompt to copy (or tell him to copy it from the HTML playbook).
3. Max opens a new Claude session, pastes the prompt, executes the step.
4. Max comes back to you and tells you what happened.
5. You verify the result, note any issues, then present the next step.
6. Repeat until all 8 steps are done.

## The HTML playbook

Full report with visual layout and copy buttons:
- Local: `~/ClaudeCode/claude-output/2026-05-04-google-ads-strategic-plan.html`
- GitHub: https://github.com/maxkwaaste/claude-output/blob/main/2026-05-04-google-ads-strategic-plan.html

Each step in the HTML has a dark prompt block with a Copy button. The prompts below are identical to those in the HTML.

## Current state (as of May 4, 2026)

- Google Ads account: CID 215-278-2474 (EUR currency)
- Campaigns were re-enabled May 4, 2026
- Conversion tracking has double/triple counting (3 actions for same event)
- John's campaigns (Search | Leads | USA/AUS) are paused but could be re-enabled
- Brand campaign works: 24 lifetime conversions at EUR 21/conv
- Golden period (Jun-Nov 2025): "Autotask Project Management" [phrase] at EUR 52/conv
- GTM-T8Z3GHT active, AW-11268212749, G-B2BE5BP0PK
- John's GTM tags (IDs 44-53) must NEVER be touched
- mu-plugin: wp-content/mu-plugins/proxuma-google-ads-tracking.php
- SSH: ssh -i ~/.ssh/id_ed25519 -p 18765 u1434-nm1gftpe9yl0@es33.siteground.eu
- Explorer API access: 2,880 ops/day max, stay under 1,000

---

## Step 1: Fix Conversion Double-Counting

**Time:** 10 minutes
**Priority:** BLOCKING -- nothing else matters until this is fixed
**Why first:** Every demo booking counts 2-3x. This inflates data and breaks automated bidding.

### What's wrong

Three enabled conversion actions fire for the same booking:
- "Booked a demo (proxuma.io/)" -- codeless page view (ID 7239093290) -- KEEP PRIMARY
- "Page view (bookingconfirm)" -- codeless page view (ID 7219443053) -- SET TO SECONDARY
- "bookingconfirmed" -- GA4 event (ID 7219607704) -- SET TO SECONDARY

Same for recorded demos:
- "Page view (recorded-demo-requested)" (ID 7219446149) -- KEEP PRIMARY
- "recordeddemorequested" GA4 (ID 7219608715) -- SET TO SECONDARY
- "Google Ads | Mail Click" (ID 7513798738) -- SET TO SECONDARY (low value)

### Verification after

- Google Ads > Goals > Conversions > Summary should show exactly 2 Primary actions
- Book a test demo. Check Conversions > By time. Should see exactly 1 conversion, not 2-3.

### Prompt for fresh session

```
Fix the conversion tracking double-counting in Proxuma's Google Ads account (CID 215-278-2474).

PROBLEM: 3 enabled conversion actions fire for the same booking event. This inflates conversion data and breaks automated bidding.

ACTIONS (use the mcp__google-ads__run_gaql tool):

1. Query all conversion actions: SELECT conversion_action.resource_name, conversion_action.name, conversion_action.status, conversion_action.primary_for_goal FROM conversion_action WHERE conversion_action.status = 'ENABLED'

2. Set these to SECONDARY (not primary for bidding):
   - "Page view (Page load proxuma.io/bookingconfirm)" (ID 7219443053) -- duplicate of "Booked a demo"
   - "Proxuma.io (web) bookingconfirmed" (ID 7219607704) -- GA4 duplicate
   - "Proxuma.io (web) recordeddemorequested" (ID 7219608715) -- GA4 duplicate
   - "Google Ads | Mail Click" (ID 7513798738) -- low-value mailto click

3. KEEP as PRIMARY:
   - "Booked a demo (proxuma.io/)" (ID 7239093290)
   - "Page view (Page load proxuma.io/recorded-demo-requested)" (ID 7219446149)

4. Verify: after changes, query again and confirm exactly 2 primary conversion actions.

IMPORTANT: Explorer access may not allow mutate operations on conversion actions. If the API blocks you, list the exact steps to do it manually in the Google Ads UI instead (Goals > Conversions > Summary > click each action > change Primary/Secondary setting).

Do NOT touch any GTM tags (John's tags IDs 44-53 in GTM-T8Z3GHT).
Do NOT modify the mu-plugin at wp-content/mu-plugins/proxuma-google-ads-tracking.php.
```

---

## Step 2: Permanently Pause John's Campaigns

**Time:** 5 minutes
**Priority:** Stops active waste
**Why now:** EUR 810 spent with zero conversions. If re-enabled by accident, they burn budget.

### What to do

Pause these campaigns:
- "Search | Leads | USA"
- "Search | Leads | AUS"

In "Targeted campaign autotask", pause these ad groups:
- "Planning" (EUR 140+ spent, zero conversions)
- "Outlook" (EUR 153+ spent, zero conversions)
Keep "Project management" ad group (5 lifetime conversions).

### Verification after

Only Brand campaign and "Project management" ad group should be running.

### Prompt for fresh session

```
Pause the waste campaigns in Proxuma's Google Ads account (CID 215-278-2474).

Use the Google Ads MCP tools to:

1. Query current campaign statuses:
   SELECT campaign.name, campaign.status, campaign.id FROM campaign

2. PAUSE these campaigns (set status to PAUSED):
   - "Search | Leads | USA" -- John's campaign, EUR 810 spent, zero conversions
   - "Search | Leads | AUS" -- John's campaign, zero conversions

3. In "Targeted campaign autotask", PAUSE these ad groups:
   - "Planning" ad group -- EUR 140+ spent, zero conversions ever
   - "Outlook" ad group -- EUR 153+ spent, zero conversions ever
   Keep the "Project management" ad group enabled (5 lifetime conversions).

4. Verify: query again, confirm only Brand and the "Project management" ad group are running.

IMPORTANT: Do NOT pause the Brand campaign or the "Project management" ad group. Those convert.
Account has Explorer API access (2,880 ops/day max). Stay under 10 API calls for this task.
```

---

## Step 3: Add Account-Level Negative Keywords

**Time:** 15 minutes
**Priority:** Prevents waste before new campaigns launch
**Why now:** Block queries that will never convert. Free money saved.

### Negatives to add

| Category | Keywords |
|----------|----------|
| Jobs | jobs, careers, salary, hiring, interview, resume, certification, vacancy |
| Education | tutorial, course, training, learn, class, exam, udemy, youtube |
| Free | free, open source, template download, github |
| Support | login, support, help desk, password, reset, bug, error, forum |
| Other PSA | connectwise, halo psa, freshdesk, zendesk, servicenow, freshservice |
| Wrong product | power bi, dashboard, report, analytics, reporting, datto backup, RMM, antivirus, security, monitoring |
| Generic | what is, definition, review, reddit, quora |

### Prompt for fresh session

```
Add account-level negative keywords to Proxuma's Google Ads account (CID 215-278-2474).

Use the Google Ads MCP tools to:

1. First, check if a shared negative keyword list already exists:
   SELECT shared_set.name, shared_set.type, shared_set.status FROM shared_set WHERE shared_set.type = 'NEGATIVE_KEYWORDS'

2. Create a shared negative keyword list called "Proxuma - Account Negatives" if one doesn't exist, then add these negative keywords (phrase match unless noted):

   Jobs: jobs, careers, salary, hiring, interview, resume, certification, vacancy
   Education: tutorial, course, training, learn, class, exam, udemy, youtube
   Free: free, open source, "template download", github
   Support: login, support, "help desk", password, reset, bug, error, forum
   Other PSA: connectwise, "halo psa", freshdesk, zendesk, servicenow, freshservice
   Wrong product: "power bi", dashboard, report, analytics, reporting, "datto backup", RMM, antivirus, security, monitoring
   Generic: "what is", definition, review, reddit, quora

3. Attach the shared list to ALL campaigns in the account.

4. Verify: query the list to confirm all negatives are in place.

IMPORTANT: Explorer API access may not support shared set mutations. If the API blocks you, provide the exact steps to add these manually in the Google Ads UI (Tools > Shared Library > Negative Keyword Lists). List each keyword clearly so I can paste them in.
```

---

## Step 4: Build the New Campaign Structure

**Time:** 30-45 minutes
**Priority:** Core growth driver
**Why now:** Tracking is clean, waste is stopped, negatives are in. Time to build.

### Architecture

| Campaign | Budget | Bidding | Ad Groups |
|----------|--------|---------|-----------|
| Proxuma - Brand | EUR 2/day | Manual CPC EUR 1.50 max | 1: Brand |
| Proxuma - Autotask Features | EUR 12/day | Manual CPC EUR 4 max | 4: Project Mgmt, Planning, Outlook, Shifts |
| Proxuma - Recorded Demo | EUR 4/day | Manual CPC EUR 3 max | 1: Informational |

All campaigns: Search only, USA + UK + AUS (Presence only).

### Prompt for fresh session

```
Build 3 new Google Ads search campaigns for Proxuma (CID 215-278-2474). Use the Google Ads MCP tools.

IMPORTANT: Explorer API access may not allow campaign creation via API. If the API blocks any mutate operation, stop and provide EXACT step-by-step instructions for the Google Ads UI instead, with every field value specified so I can just click through and paste.

## Campaign 1: Brand
- Name: "Proxuma - Brand"
- Budget: EUR 2/day
- Bidding: Manual CPC, max CPC EUR 1.50
- Network: Search only (no Display, no Search Partners)
- Locations: United States, United Kingdom, Australia (Presence only)
- Ad Group "Brand": exact match keywords: [proxuma], [proxuma autotask], [proxuma planning], [proxuma project management]
- Landing page: https://proxuma.io/

## Campaign 2: Autotask Features
- Name: "Proxuma - Autotask Features"
- Budget: EUR 12/day
- Bidding: Manual CPC, max CPC EUR 4.00
- Network: Search only
- Locations: United States, United Kingdom, Australia (Presence only)
- 4 Ad Groups:

  Ad Group "Project Management": phrase match:
  "autotask project management", "autotask project planning", "autotask project timeline", "autotask gantt chart", "autotask project management tool"
  Landing: https://proxuma.io/planning/

  Ad Group "Planning & Capacity": phrase match:
  "autotask planning", "autotask resource planning", "autotask capacity planning", "autotask dispatch calendar", "autotask scheduling"
  Landing: https://proxuma.io/planning/

  Ad Group "Outlook Integration": phrase match:
  "autotask outlook integration", "autotask outlook sync", "autotask calendar integration", "autotask exchange sync"
  Landing: https://proxuma.io/planning/

  Ad Group "Shifts & Dispatch": phrase match:
  "autotask shift planning", "MSP shift scheduling autotask", "autotask service call scheduling", "autotask dispatch"
  Landing: https://proxuma.io/shifts/

## Campaign 3: Recorded Demo
- Name: "Proxuma - Recorded Demo"
- Budget: EUR 4/day
- Bidding: Manual CPC, max CPC EUR 3.00
- Network: Search only
- Locations: United States, United Kingdom, Australia (Presence only)
- Ad Group "Informational": phrase match:
  "autotask project management tool", "autotask add-on", "autotask planning tool", "MSP project management autotask"
  Landing: https://proxuma.io/how-to-use-autotask-for-project-management/

All campaigns: ensure the shared negative keyword list "Proxuma - Account Negatives" is attached.
```

---

## Step 5: Write and Submit Responsive Search Ads

**Time:** 20 minutes
**Priority:** Ads go live
**Why now:** Campaigns exist but have no ads.

### Headlines (from prospect language in 140+ demo transcripts)

Pin H1: Autotask Project Management, Visual Planning for Autotask, Autotask Planning Made Visual
Pin H2: Drag-and-Drop Scheduling, See All Projects at a Glance, Bulk Edit 50 Tasks at Once
Pin H3: From EUR 20/User/Month, 3.75x Cheaper Than TopLeft, Free Trial No Minimum

### Descriptions

1. See all your projects on one visual timeline. Drag tasks across weeks. Create service calls in 2 clicks.
2. Stop using Excel next to Autotask. Proxuma adds the planning layer Autotask should have built.
3. Resource capacity at a glance. Know who is free before you promise a deadline. Real-time sync.
4. Only planners and PMs need a license. Engineers keep working in Autotask. Everything syncs back.

### Prompt for fresh session

```
Create Responsive Search Ads for all 6 ad groups in Proxuma's Google Ads account (CID 215-278-2474).

If the API doesn't support ad creation (Explorer access), provide exact copy-paste instructions for the Google Ads UI for each ad group.

## Shared descriptions (use for ALL ad groups):
D1: See all your projects on one visual timeline. Drag tasks across weeks. Create service calls in 2 clicks.
D2: Stop using Excel next to Autotask. Proxuma adds the planning layer Autotask should have built.
D3: Resource capacity at a glance. Know who is free before you promise a deadline. Real-time sync.
D4: Only planners and PMs need a license. Engineers keep working in Autotask. Everything syncs back.

## Ad Group: Project Management (campaign "Proxuma - Autotask Features")
Final URL: https://proxuma.io/planning/
Display path: /planning/autotask
Headlines (15, pin as noted):
H1-pin: Autotask Project Management
H1-pin: Visual Planning for Autotask
H1-pin: Autotask Planning Made Visual
H2-pin: Drag-and-Drop Scheduling
H2-pin: See All Projects at a Glance
H2-pin: Bulk Edit 50 Tasks at Once
-: Know Who Has Capacity
-: Outlook Sync That Works
H3-pin: From EUR 20/User/Month
H3-pin: 3.75x Cheaper Than TopLeft
-: Engineers Never Need Access
H3-pin: Free Trial, No Minimum
-: Built for MSP Planners
-: Syncs Back to Autotask
-: New Features Every 2 Weeks

## Ad Group: Planning & Capacity
Final URL: https://proxuma.io/planning/
Display path: /planning/capacity
Adapt headlines: replace "Project Management" with "Resource Planning" and "Capacity Planning" in H1 pins. Keep H2/H3 same.

## Ad Group: Outlook Integration
Final URL: https://proxuma.io/planning/
Display path: /planning/outlook
Adapt headlines: H1 pins become "Autotask Outlook Integration", "Outlook Sync for Autotask", "Fix Autotask Outlook Sync". H2-pin add: "Calendar Sync That Works". Keep H3 same.

## Ad Group: Shifts & Dispatch
Final URL: https://proxuma.io/shifts/
Display path: /shifts/dispatch
Adapt headlines: H1 pins become "Autotask Shift Planning", "MSP Shift Scheduling", "Dispatch Calendar for Autotask". Keep H2/H3 same.

## Ad Group: Brand (campaign "Proxuma - Brand")
Final URL: https://proxuma.io/
Display path: /planning
Headlines: "Proxuma | Autotask Planning", "Visual Planning for Autotask", "The Autotask Planning Layer", "From EUR 20/User/Month", "Free Trial Available", "Drag-and-Drop Scheduling", "Built for MSP Planners"

## Ad Group: Informational (campaign "Proxuma - Recorded Demo")
Final URL: https://proxuma.io/how-to-use-autotask-for-project-management/
Display path: /planning/demo
Headlines: "Autotask Project Management", "See Proxuma in 3 Minutes", "Watch a Recorded Demo", "Visual Planning for Autotask", "Free Recorded Demo", "No Signup Required"
Descriptions: swap D2 for "Watch how Proxuma turns Autotask project management from painful to visual. 3-minute recorded demo, no signup needed."

After creating all ads, verify by querying: SELECT ad_group_ad.ad.responsive_search_ad.headlines, campaign.name, ad_group.name FROM ad_group_ad WHERE ad_group_ad.status = 'ENABLED'
```

---

## Step 6: Audit the /planning/ Landing Page

**Time:** 30 minutes
**Priority:** Conversion rate improvement
**Why now:** Ads are pointing to /planning/. If it doesn't match the ad promise, clicks won't convert.

### What to check

- Product screenshot above the fold?
- Clear CTA (Book a Demo / Start Free Trial)?
- Pricing visible (EUR 20/user)?
- "Engineers never need a license" messaging?
- Social proof (customer quotes or logos)?
- PageSpeed score (target: mobile 60+, desktop 80+)

### Prompt for fresh session

```
Audit the proxuma.io/planning/ landing page for Google Ads readiness. This is the primary landing page for all Autotask Features ad campaigns.

1. Fetch the page: use WebFetch on https://proxuma.io/planning/ and extract:
   - Is there a product screenshot above the fold?
   - Is there a clear CTA button (Book a Demo / Start Free Trial)?
   - Is pricing mentioned (EUR 20/user/month)?
   - Is "engineers never need a license" or similar messaging present?
   - Are there customer quotes or social proof?
   - What is the page structure (sections, headings)?

2. Run PageSpeed check: use WebFetch on https://pagespeed.web.dev/analysis?url=https://proxuma.io/planning/ -- extract mobile and desktop scores, LCP, CLS.

3. Also check https://proxuma.io/shifts/ the same way (secondary landing page).

4. Based on findings, produce a prioritized list of improvements. Focus only on changes that improve conversion rate for paid traffic. Do NOT suggest SEO changes, blog content, or other non-conversion items.

5. If the page is missing critical elements (no CTA, no screenshot, no pricing), draft the specific HTML/content blocks to add. Use the Proxuma brand voice from Open Brain: direct, no marketing fluff, prospect language like "planning in Autotask is broken, we fixed it."

Context: Proxuma is an Autotask add-on for MSP project management. EUR 20/user/month vs TopLeft at EUR 75. Only planners need a license, engineers stay in Autotask. Key features: visual timeline, drag-drop dispatch, bulk edit, capacity planning, own Outlook sync, shift planning.

SSH access: ssh -i ~/.ssh/id_ed25519 -p 18765 u1434-nm1gftpe9yl0@es33.siteground.eu
WordPress theme: custom "proxuma" (not Elementor). Use $wpdb->update() for page changes, never wp_update_post().
```

---

## Step 7: Weekly Search Terms Review (Recurring)

**Time:** 15 min/week
**Priority:** Ongoing optimization
**When:** Every Monday, starting 7 days after campaigns go live

### Prompt for fresh session (reuse weekly)

```
Weekly Google Ads optimization for Proxuma (CID 215-278-2474). Run this every Monday.

Use the Google Ads MCP tools to pull:

1. Search terms from the last 7 days:
   SELECT search_term_view.search_term, metrics.clicks, metrics.impressions, metrics.conversions, metrics.cost_micros, campaign.name, ad_group.name FROM search_term_view WHERE segments.date DURING LAST_7_DAYS AND metrics.impressions > 0 ORDER BY metrics.cost_micros DESC

2. Keyword performance:
   SELECT ad_group_criterion.keyword.text, ad_group_criterion.keyword.match_type, metrics.clicks, metrics.conversions, metrics.cost_micros, ad_group_criterion.quality_info.quality_score, campaign.name FROM keyword_view WHERE segments.date DURING LAST_7_DAYS

3. Campaign performance:
   SELECT campaign.name, metrics.clicks, metrics.impressions, metrics.conversions, metrics.cost_micros, metrics.average_cpc FROM campaign WHERE segments.date DURING LAST_7_DAYS AND campaign.status = 'ENABLED'

4. Device performance:
   SELECT segments.device, metrics.clicks, metrics.conversions, metrics.cost_micros FROM campaign WHERE segments.date DURING LAST_7_DAYS AND campaign.status = 'ENABLED'

Based on the data:
- Flag any search terms that are irrelevant (add to negative list)
- Flag any keyword with Quality Score below 5
- Flag any ad group with 30+ clicks and zero conversions
- Recommend bid adjustments
- Report: total spend, total conversions, CPA, and comparison to target CPA of EUR 80

Keep the report under 200 words. Focus on actions, not commentary.
Account is on Explorer API access: budget API calls, stay under 10 for this task.
```

---

## Step 8: Switch to Automated Bidding (After 15+ Conversions)

**Time:** Week 4-6
**Priority:** Scale
**Prerequisite:** 15+ conversions in the last 30 days

### Prompt for fresh session

```
Switch Proxuma's main Google Ads campaign to automated bidding (CID 215-278-2474).

PREREQUISITE CHECK (do this first):
1. Query total conversions in the last 30 days:
   SELECT metrics.conversions FROM customer WHERE segments.date DURING LAST_30_DAYS
   If under 15, STOP. Say "Not enough conversion data yet. Stay on Manual CPC and run the weekly optimization prompt instead."

2. Query current CPA:
   SELECT campaign.name, metrics.conversions, metrics.cost_micros FROM campaign WHERE campaign.name = 'Proxuma - Autotask Features' AND segments.date DURING LAST_30_DAYS
   Calculate CPA = cost / conversions.

3. If 15+ conversions exist:
   - Change "Proxuma - Autotask Features" bidding strategy to Maximize Conversions
   - Set target CPA at 120% of current CPA (gives Google 20% headroom)
   - Do NOT change "Proxuma - Brand" (keep Manual CPC)
   - Do NOT change "Proxuma - Recorded Demo" (keep Manual CPC)

4. Set a calendar reminder to check CPA after 14 days. If CPA is 2x or more above target, revert to Manual CPC.

Report: what the current CPA is, what target CPA you set, and what to monitor.
```

---

## Progress Tracker

| Step | Status | Date Done | Notes |
|------|--------|-----------|-------|
| 1. Fix conversion tracking | [ ] | | |
| 2. Pause waste campaigns | [ ] | | |
| 3. Add negative keywords | [ ] | | |
| 4. Build campaigns | [ ] | | |
| 5. Write ads | [ ] | | |
| 6. Audit landing page | [ ] | | |
| 7. Weekly review | [ ] | | Recurring |
| 8. Automated bidding | [ ] | | After 15+ conversions |

---

## Guide session behavior

When Max returns after each step:
1. Ask what happened (did the API work or did they need manual UI steps?)
2. Update the progress tracker
3. Note any issues or deviations
4. Present the next step
5. Give Max the prompt to copy

If a step fails or needs adjustment, help Max troubleshoot before moving on. Do not skip steps.
