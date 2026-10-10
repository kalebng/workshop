# Reference

**Version:** 1.0 · **Updated:** 10 Oct 2026 · **Owner:** Kaleb

---

## Never put in this file
- Passwords, keys, tokens or any secret
- Open bugs, known weaknesses, admin paths, non-public URLs
- Names of projects not yet public
- Non-public personal or business information

Bugs and weaknesses go somewhere actually private: a private repo's issues, or GitHub's private security advisory for a public repo. Issues in a public repo are public.

---

## How this works

1. **Modules.** Rules are grouped into modules named after bike parts. Each project picks what it needs. **FRAME is mandatory.**
2. **Project files.** Every project repo has three files:
   - `LEBBIE.md`: ID card (modules, adopted version, approved exceptions, last audit)
   - `GEOMETRY.md`: the project's purpose, audience, direction and open questions (the "why")
   - `DECISIONS.md`: the project's own rules (the "what")
3. **Which version is enforced.** A project follows the version on its ID card. The latest version is read only to detect changes; new rules apply to a project after Kaleb approves adopting them.
4. **Snapshots.** Every published version is tagged in git (`v1.0`, `v1.1`…), so any version can be read exactly:
   - Latest: `https://raw.githubusercontent.com/kalebng/workshop/main/ref.md`
   - Specific: `https://raw.githubusercontent.com/kalebng/workshop/v1.0/ref.md`
   GitHub may serve a cached copy for a few minutes after an edit.
5. **Priority.** Adopted rules override a project unless its ID card lists an exception approved by Kaleb (date and reason).
6. **Project rules** for one project only live in that project's `DECISIONS.md`. Reference rules here by ID (e.g. `F-5`); never copy them.
7. **Who changes this file.** Only Kaleb. Never merge pull requests from others. Claude reads `main` or a version tag only.
8. **Changing a rule.**
   - Wording fix or clarification (same behavior): edit in place, note it in the changelog, minor version bump.
   - Behavior change: mark the old rule `Replaced by X-Nb`, add the new rule, minor bump.
   - Rule removed or meaning reversed: major bump (`1.x → 2.0`); every project re-audits.
9. **Statuses:** `Active` · `Draft` (being defined, not enforced) · `Replaced`

---

## Modules

| Module | Who takes it | Status |
|---|---|---|
| **FRAME** | Every project (mandatory) | Active |
| **LIGHTS** | Projects with public pages meant to be found by search and AI | Active |
| **HEADUNIT** | Content sites with places, routes, photos and local knowledge | Active |
| **DRIVETRAIN** | Logged-in tools and apps | Draft |

**Reserved for future modules:**
- **BRAKES**: security & privacy
- **PAINT**: look & feel
- **SADDLEBAG**: data, storage, backups
- **BELL**: contact forms & visitor input. Before any form goes live, BELL must define at minimum: server-side validation (including server-side Turnstile verification if used), rate limiting, input size limits, and safe handling of submitted data.

---

## FRAME — every project

**F-1 · Ask before executing** · Active
Claude proposes; Kaleb approves before anything is built or changed. Claude Code works on branches; Kaleb merges.

**F-2 · Secrets** · Active
- Credentials live only in encrypted secret storage (Cloudflare Worker/Pages secrets, GitHub Actions secrets). Never in plaintext variables, `PUBLIC_`/client-exposed build variables, logs, or committed `.env` files.
- GitHub secret scanning and push protection are on for every repo where available.
- If a secret is ever exposed: revoke it, rotate it, review where it was used, then remove it from history.

**F-3 · Accounts protected** · Active
Two-factor authentication on GitHub and Cloudflare.

**F-4 · One address per page** · Active
- HTTPS only; `http://` permanently redirects to `https://`.
- `www.lebbie.cc` permanently redirects to `lebbie.cc`. Subdomains do not get `www.` variants.
- One trailing-slash style per project, enforced with permanent (301/308) redirects. Verified end-to-end with the hosting platform's own routing (no loops, no contradiction with the canonical).

**F-5 · AI training: not permitted** · Active
**Owner policy:** Lebbie content may not be used for AI model training without payment or explicit permission.
**Technical enforcement** (honored by compliant crawlers; cannot stop every third party or remove data collected earlier):
- **Training crawlers**, disallowed in `robots.txt` and blocked at Cloudflare: `GPTBot`, `ClaudeBot`, `CCBot`
- **Training control tokens**, `robots.txt` only (no user agent of their own): `Google-Extended`, `Applebot-Extended`

`Google-Extended` also controls grounding in Gemini Apps and Vertex AI. It does not affect Google Search, AI Overviews or AI Mode, which are governed by `Googlebot`.

**F-6 · AI search and user fetch: allowed** · Active
On public pages, allow at minimum: `Googlebot`, `Bingbot`, `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`, `PerplexityBot`, `Perplexity-User`.
Some user-triggered fetchers may not follow `robots.txt`; allowing them simply matches their behavior.

**F-7 · Stated policy = enforced policy** · Active
- Every public host serves its own `robots.txt` stating F-5 and F-6, including a `Sitemap:` line where LIGHTS applies.
- Cloudflare blocks the F-5 training crawlers **by name**, not through a broad category that may also catch search crawlers. Cloudflare may treat crawlers that do both search and training (e.g. Googlebot) by their strictest rule; that must never block F-6 crawlers.
- `/robots.txt` stays reachable to all crawlers, including blocked ones, so they can read the opt-out.
- Never allow or bypass bots by user-agent string alone; rely on Cloudflare's verified-bot identification.
- If the stated or enforced policy changes, both change in the same update.

Cloudflare settings record (domain-wide). Use the dashboard's actual labels. F-7 is compliant only once every row is confirmed with a date.

| Setting | Target | Confirmed (date) |
|---|---|---|
| F-5 training crawlers blocked by name | Block | — |
| AI search / user-fetch crawlers | Allow | — |
| `/robots.txt` reachable by all | Yes | — |
| Managed robots.txt | Off (each project serves its own) | — |
| Always Use HTTPS | On | — |
| `www.lebbie.cc` → `lebbie.cc` | Permanent redirect | — |

**F-8 · Spot-check search access after access-affecting changes** · Active
After a change that can affect crawler access (crawler rules, redirects, hosting), run a live URL test in Google Search Console and Bing Webmaster Tools on representative pages. This verifies Google and Bing fetching only; it is not proof for other crawlers, every URL, or indexing.

**F-9 · Project files and Claude block** · Active
Every project repo has `LEBBIE.md`, `GEOMETRY.md` and `DECISIONS.md` (from `templates/`), and the Lebbie block in its `CLAUDE.md` (from `templates/CLAUDE-block.md`). Work that contradicts the project's GEOMETRY is proposed to Kaleb first, like any rule change.

**F-10 · No competing copies** · Active
Platform hostnames (`*.workers.dev`, `*.pages.dev`), preview deployments and any other alternate host either redirect to the real address, are disabled, or are not indexable.

**F-11 · Real "not found"** · Active
Unknown addresses return a genuine 404 page and status, not the home page or an app shell.

---

## LIGHTS — public pages meant to be found

**L-1 · Facts in the page, interactive layer on top** · Active
Every essential fact is in the page's HTML. Maps, animations and widgets are an added layer and never the only place a fact exists. Googlebot renders JavaScript; a Vercel/MERJ crawl-log study (Dec 2024) found the OpenAI and Anthropic crawlers did not execute it.
Check: fetch the raw page with `curl`; key facts must appear.

**L-2 · Unique page basics** · Active
Every page has its own title, meta description and self-referencing canonical. These are signals; search engines may still choose otherwise.

**L-3 · Sitemap with honest dates** · Active
- Lists published canonical pages only; referenced in `robots.txt`; submitted in Google Search Console and Bing Webmaster Tools.
- `lastmod` comes from the content's own `updatedAt`, changed only for meaningful content changes; never build or deploy time.
- No `priority` or `changefreq`.
- If a published page isn't indexed after a few weeks, inspect it in Search Console before changing anything. Don't repeatedly resubmit.

**L-4 · Structured data describes only what's visible and true** · Active
No ratings, reviews, prices or facts that aren't shown and true.

**L-5 · Not now: llms.txt, IndexNow, image sitemaps** · Active
Revisit only when a measurable need appears.

**L-6 · Languages** · Active
English and Bahasa Indonesia mixed naturally using real local names. The page declares its main language (`lang`), and passages in the other language are marked with their own `lang`. No translated copies or `hreflang` until full translations exist.

**L-7 · Share previews** · Active
Every public page has its own Open Graph title, description, image and URL (matching its canonical), so links shared on WhatsApp, Instagram and Threads show a proper preview card. Preview images are Lebbie's own or used with permission, wide format (e.g. 1200×630).

---

## HEADUNIT — places, routes, photos, local knowledge

**H-1 · Flat, permanent addresses** · Active
`/places/{slug}` · `/routes/{slug}` · `/notes/{slug}` · `/segments/{slug}` (see H-5). Destination hubs at the top level (e.g. `/bintan`). Area is shown on the page, not in the address. Slugs are lowercase, real local names.

**H-2 · The slug is the ID** · Active
No separate hidden IDs. Renaming a slug requires a permanent redirect. The build fails on broken references or duplicate slugs. Revisit if renames or cross-project links become painful.

**H-3 · Observations: dated, sourced, aging visibly** · Active
- Every local-knowledge note records: **claim** · **observedOn** · **source** (`self` / `friend` / `official` / `unverified`) · **status** (`current` / `stale`). Date and source are shown to visitors.
- Time-sensitive notes (toilet access, business operation, road works, swimming conditions, wildlife) older than 12 months automatically show "may be outdated" until re-checked.
- Never present an observation as permanent, guaranteed, or a safety certification. No "safe to swim" claims from personal experience alone.
- Safety notes (water, wildlife, hazards) state which kind they are: personal observation, credible report, confirmed sighting, or official warning. Official guidance is shown first when it exists. A local report is never generalized to a whole area.

**H-4 · Geography as two labels** · Active
Each area records `island` and `admin` (e.g. `kota-tanjungpinang`, `kab-bintan`). No drawn boundaries.

**H-5 · Segments live inside routes first** · Active
A climb or stretch is a section of its route page until it belongs to 2+ published routes; then it gets its own page.

**H-6 · Third-party places: "written about", never "owned by"** · Active
Structured data: a `WebPage` (or `Article`) by Lebbie whose `about` is a `Place` (or fitting subtype) with name and coordinates. No ratings, reviews, opening hours or prices. Never implies Lebbie owns or operates it. This is Lebbie's own accuracy policy, stricter than schema.org requires.

**H-7 · Photo and location privacy** · Active
Before publishing:
- Strip GPS and camera data (EXIF) from photos unless the location is intentionally public.
- Never reveal private homes, exact private locations, or people's identities without their permission.
- Credit sources; get permission before publishing others' photos or tips.

**H-8 · Category and facilities are separate** · Active
A place has one **category** (what it is, e.g. minimart, café, beach) and any number of **facilities** (what it has, e.g. water, snacks, toilet, shade). One place record serves both browsing and practical searches; facilities never create duplicate places.

---

## DRIVETRAIN — logged-in tools and apps

**Status: Draft.** To be defined from the first audit of a project that uses it. Before any project with logins goes live, DRIVETRAIN must define at minimum: authentication, authorization, session handling, input validation, and what data is stored and exposed. Until then, FRAME applies.

---

## Guardrails

**Every build (fails the deploy), per adopted module:**
- FRAME: `robots.txt` parsed with a real robots parser; for representative URLs, F-5 crawlers resolve to disallowed and F-6 crawlers to allowed.
- LIGHTS: unique title, description, canonical and share-preview tags on every page; sitemap present with content-based `lastmod`.
- HEADUNIT: no duplicate slugs or broken references; no ratings, reviews, hours or prices in place-page structured data.

**After each release, and monthly (automated smoke test, alerts Kaleb when results change from the last run):**
- Record the full response chain for `http://`, `www`, trailing-slash and an unknown address.
- Record responses when requesting as F-5 and F-6 crawler names.

These requests imitate crawler names from an ordinary server. They show changes in redirects and Cloudflare rules, not how genuine verified crawlers are treated. Real Google/Bing access is spot-checked under F-8.

**Yearly (January):** Kaleb reviews every Active rule.

---

## Changelog

| Version | Date | Change |
|---|---|---|
| 1.0 | 10 Oct 2026 | Created from the lebbie.cc discoverability review (9 Oct 2026); revised after an independent audit before first publication. FRAME, LIGHTS, HEADUNIT Active; DRIVETRAIN Draft. |
