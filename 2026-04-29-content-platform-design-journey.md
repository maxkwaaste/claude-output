# Content Platform Design Journey
## From Idea to Spec in One Brainstorming Session

**Date:** April 29, 2026
**Project:** Proxuma Content Platform for Dxfferent Consultants
**Tools used:** Claude Code (Opus 4.6), Excalidraw MCP, Open Brain

---

## Full Conversation History

### 1. The Brief

**Max:**
> *[shares msp_journey_overzicht.pdf]*
>
> Brainstorm with me. I want this list to become the basis for our consultants to create content. They need a super easy interface with which they can just pick a topic and dump their text. This should then be tagged as done, and the content will be transformed by Claude (subscription), but that part comes later, into various kinds of content. Initially this will be hosted on dxfferent.com, later to our internal app. The purpose is to farm our experts for relevant content, in a way that's convenient for them. AI cannot be the source of content, but it can transform their content into all kinds of pieces. Everything on here must be editable inline, it is our content writing platform.

**Claude:** *[Reads PDF, searches Open Brain, invokes brainstorming skill]*

The PDF contains 9 pages covering ~30 sub-topics across 6 domains (Identity, Organisation, GRC, Positioning, Portfolio, Technology, Blueprint, Data Driven). Each sub-topic has structured Problemen/Frustraties/Vragen columns. That's the topic menu for consultants. dxfferent.com is WordPress with REST API, Polylang, Yoast SEO, and ACF Pro.

---

### 2. Understanding the Content Structure

**Claude:** Some of what we're working on might be easier to explain if I can show it to you in a web browser. Want to try it?

**Max:**
> What about claude.ai/design? Can we collaborate through that? Also step 2: *[shares proxuma_contentbibliotheek.html]* has all the kinds of categories, content types etc. Please read thoroughly so you understand what our intention is here, I fail at explaining.

**Claude:** *[Reads contentbibliotheek HTML]*

Now I get it much better. The contentbibliotheek is the missing puzzle piece. You've already mapped 63 concrete content ideas with 6 content categories (Bewijs & resultaat, Educatie & uitleg, Markt & visie, Contrair & polariserend, Eigen verhaal, Tools & resources), each linked back to MSP Journey source topics and with hook templates baked in.

The full picture: the MSP Journey provides the **problem space** (30 sub-topics), and the contentbibliotheek provides the **content assignments** (63 ideas with category, hook format, and source). The platform is the interface where consultants claim a topic, dump their expertise, and it gets queued for AI transformation.

---

### 3. Clarifying Questions

**Claude:** **Who are the consultants writing content?** Are these Dxfferent employees, external MSP consultants/partners, or both?

**Max:**
> Let's do HTML, or even Excalidraw MCP that I've got setup. The consultants are Dxfferent consultants with all the MSP/Autotask knowledge. There's about 5 of them.

**Claude:** Good. 5 internal Dxfferent consultants, so no auth needed for v1. **What does "dump their text" look like in practice?**
- A) Free-form text field
- B) Lightly guided (hook template + context visible)
- C) Structured input (specific fields)

**Max:**
> If they want to do free form they should be able to. Claude can then categorize. If they want to pick a topic this should also be available as an option. Also no auth, make sure it's also noindex for Google.

**Claude:** Both entry paths: pick a topic from the list, or just dump raw text and let Claude sort it later. No auth, noindex. **Where does the content go after a consultant submits it?**
- A) Single-page app -- everything on one page
- B) Separate flows -- browse page + writing page

**Max:**
> All raw text needs to be stored in that form and indexed, well organized. All content that flows from there also. We also want to keep track of whether we've posted it, in a database of sorts, and we need a nice color visual display of what has and hasn't been posted. Lastly, a content calendar where we can queue the messages, with future connection to our LinkedIn profile ideally, and of course dxfferent.com. Also the ability to track all the page views and LinkedIn tracking data in there would be ideal for us to steer which topics are the best to post about, which types of content etc.

---

### 4. Scope Check -- Breaking It Into Phases

**Claude:** I need to flag something before we go further. What started as a "consultants pick a topic and dump text" tool has grown into a full content operations platform with at least 5 distinct subsystems:

1. Content input -- topic picker + free-form dump
2. Content storage & indexing
3. Publishing tracker -- visual status board
4. Content calendar -- scheduling queue with LinkedIn + dxfferent.com
5. Analytics -- page views, LinkedIn metrics, performance steering

That's a real product. **Can we build this in phases?**
- **Phase 1** -- Content harvesting tool. Ship within days.
- **Phase 2** -- AI transformation + content calendar + publishing status
- **Phase 3** -- LinkedIn/dxfferent.com integrations + analytics

**Max:**
> Yes makes total sense. The app will later be built into github.com/Proxuma-Nova/claude-harness so please take that into account, that the architecture is well designed, well described and can be ported over later. What I need right now is Phase 1 done by tomorrow.

**Claude:** *[Explores claude-harness repo]* It's a manifest-driven distribution system. **Storage for Phase 1?**
- A) JSON file -- dead simple but no querying
- B) SQLite -- single file, proper querying, easy to migrate
- C) Supabase -- proper backend but adds setup time

I recommend **B (SQLite)**.

**Max:**
> B it is. You have SiteGround and WP access to dxfferent.com. Can you work it with just that?

**Claude:** Yes. SiteGround runs PHP with SQLite support (PDO). A standalone app at dxfferent.com/content/, completely separate from WordPress.

**Max:**
> Use a URL like content.dxfferent.com instead.

**Claude:** `content.dxfferent.com` it is. Separate subdomain, separate document root, WordPress never sees the requests.

---

### 5. UI Design -- Filter System

**Claude:** **How should the 63 topics be organized on screen?** Three options:
- A) Filter by content category, source shown per card
- B) Filter by MSP Journey domain, category as badge
- C) Both filters available

**Max:**
> Could we do an Excalidraw/HTML collab on this decision, that you show me a mockup of all three?

**Claude:** *[Creates 102 elements on Excalidraw canvas]*

All three options on canvas:
- Option A (red) -- category filters only
- Option B (blue) -- domain filters only
- Option C (green, recommended) -- two filter rows: DOMEIN + TYPE

**Max:**
> Option C. Be sure to save what you drew for later.

**Claude:** *[Exports Excalidraw scene]* Saved. Option C -- dual filter rows.

---

### 6. Editor View & WordPress Considerations

**Claude:** **When a consultant clicks a topic card, accordion or full-width editor?**

**Max:**
> Full width.

**Claude:** Here's the approach: Single-page app with PHP REST API. Frontend: one HTML file, vanilla JS, no build step. Backend: one api.php, SQLite via PDO.

**Max:**
> For now you will be dealing with the dxfferent.com WordPress theme and quirks. Be sure your setup can handle that.

**Claude:** The subdomain gets its own document root on SiteGround, completely separate from WordPress. No theme interference, no .htaccess conflicts. I'll add Cache-Control headers to bypass SiteGround caching.

---

### 7. Detailed Design -- Four Sections

**Section 1: Database Schema** -- Three tables (topics, submissions, authors). 63 topics pre-seeded from JSON. Submissions can be linked to topic or standalone.

**Max:** Good.

**Section 2: API Endpoints** -- GET/PATCH topics, GET/POST/PATCH/DELETE submissions, GET/POST authors, GET stats. All JSON, Cache-Control: no-store.

**Max:** Yes, will be perfected later though.

**Section 3: Frontend Views** -- View 1 (Topic Board): header, stats, dual filters, card list, free-form button. View 2 (Full-width Editor): writing area with auto-save, context panel with hook + MSP Journey context. Mobile-responsive.

**Max:** Good.

**Section 4: File Structure & Deployment** -- index.html, api.php, seed.json, data/content.db. Create subdomain, SSH in, drop files.

**Max:** All good.

---

### 8. Spec Written & Handoff

**Claude:** *[Writes spec, self-reviews]* Spec written to `docs/superpowers/specs/2026-04-29-content-platform-design.md`.

**Max:**
> Write superpower write plan prompt for next Claude to do that.

**Claude:** *[Creates handoff prompt, copies to clipboard, sets up notes tab, saves to Open Brain]* Done. Handoff prompt invokes `/superpowers:writing-plans`, points to spec, source files, SSH creds.

**Max:**
> Standby to assist if next Claude doesn't understand it.

**Claude:** Standing by.

**Max:** *[Next session picks it up cleanly]*

**Claude:** Looks like it picked everything up. No issues. Let it run.

---

## The 6 Content Categories

| Category | Count | Purpose | Example Hook |
|----------|-------|---------|-------------|
| Bewijs & resultaat | 12 | Case studies, data analysis, proof | "Ik analyseerde [X] en vond [Y]" |
| Educatie & uitleg | 16 | How-tos, explainers, frameworks | "Complex ding makkelijk gemaakt" |
| Markt & visie | 10 | Market predictions, vision | "Toekomst van [markt / tool]" |
| Contrair & polariserend | 14 | Myth-busting, contrarian | "Waarom [mythe] niet klopt" |
| Eigen verhaal | 7 | Personal stories, lessons | "Wat ik nooit meer zou doen" |
| Tools & resources | 4 | Comparisons, lists, templates | "[X] vs [Y] voor MSP's" |

## Architecture

```
content.dxfferent.com/
  index.html          -- SPA (topic board + editor)
  api.php             -- REST API (CRUD on SQLite)
  seed.json           -- 63 topics + MSP Journey context
  data/
    content.db        -- SQLite (auto-created)
    .htaccess         -- Deny from all
  .htaccess           -- URL rewriting, noindex, cache headers
```

### Database
```
topics (id, title, hook, category, domain, subdomain, status, claimed_by, created_at)
submissions (id, topic_id, author, raw_content, category, domain, created_at, updated_at)
authors (id, name, created_at)
```

### API
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | /api/topics | List all (filters: category, domain, status) |
| GET | /api/topics/:id | Single topic with submissions |
| PATCH | /api/topics/:id | Update status, claimed_by, title, hook |
| POST | /api/submissions | Create submission |
| PATCH | /api/submissions/:id | Edit content |
| GET | /api/stats | Counts by status, category, domain |

## Three Phases

1. **Phase 1 (now):** Content harvesting tool. Topic board, editor, free-form input, SQLite, status tracking. Ship April 30.
2. **Phase 2 (next):** AI transformation, content calendar, publishing status board.
3. **Phase 3 (later):** LinkedIn/dxfferent.com publishing, analytics dashboard.

## Portability to Proxuma

- `seed.json` feeds a Laravel seeder directly
- API contract maps 1:1 to Laravel resource routes
- SQLite schema translates to a Laravel migration
- Frontend decoupled via `/api/*`
- Packaged as a `claude-harness` module

## Files & References

| What | Where |
|------|-------|
| Full spec | `~/docs/superpowers/specs/2026-04-29-content-platform-design.md` |
| Excalidraw mockups | `~/Downloads/Autoriteit/content-platform-mockups.excalidraw` |
| Mockup PNG | `~/Downloads/Autoriteit/content-platform-mockups.png` |
| Source: contentbibliotheek | `~/Downloads/Autoriteit/proxuma_contentbibliotheek.html` |
| Source: MSP Journey | `~/Downloads/Autoriteit/msp_journey_overzicht.pdf` |
| HTML presentation | `~/ClaudeCode/claude-output/2026-04-29-content-platform-design-journey.html` |
| This markdown | `~/ClaudeCode/claude-output/2026-04-29-content-platform-design-journey.md` |
