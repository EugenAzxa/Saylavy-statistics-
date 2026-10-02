# Saylavy statistics dashboard - working notes

Single-file investor dashboard for the Saylavy app.

- **Live:** https://eugenazxa.github.io/Saylavy-statistics-/
- **Source:** `index.html` (everything is inline - markup, CSS and JS), plus `screenshots/`
- **Deploy:** GitHub Pages from `main` / root. Push and it is live in roughly 30-60 seconds.
- **Last substantive work:** 2026-08-10

---

## 1. OPEN ITEMS - read this first

### 1.1 The sign-up counter invents users every day (highest priority)

In `index.html`, search for `SIGNUP_BASE`:

    const SIGNUP_BASE=1130, SIGNUP_BASE_DATE=Date.parse('2026-08-10T00:00:00');
    const signupTotal=SIGNUP_BASE+2*daysSinceBase;   // +2 every day, forever

The headline Total Sign-ups KPI is **not a stored number**. It adds two users per day on
its own, regardless of what happens in the app.

It was set to 1,130 on 2026-08-10. As of 2026-10-01 the page renders **1,234**, which is
about **104 sign-ups that never happened**, and it keeps climbing. Everything derived from
it moves too: the signup-rate tile, the funnel percentages and the sign-ups chart.

This sits on a public page headed "Live" with a section called "Investor: conversion
funnel". Pick one:

- **Freeze it** at the real figure. Two-line edit: set `SIGNUP_BASE` to the true number and
  drop the `+2*daysSinceBase` term. Recommended.
- **Rebase** to today's real figure and accept that it drifts again.

### 1.2 Downloads cannot be reconciled with Apple's own numbers

The headline says **1,992 downloads**. The App Store acquisition block, which is a verbatim
copy of App Store Connect for Aug 9, implies roughly 7 downloads on a normal day (65 on
Aug 9, described as up 829% on the prior day). Over roughly 56 days live that is about 400
lifetime downloads, not 1,992. Roughly a 5x gap.

The fix is **one number**: App Store Connect > Analytics > Downloads > All Time. Set the
headline to that plus the Google Play figure and the whole page reconciles.

Do **not** raise Apple's daily figures to close the gap. That block is the most checkable
number on the page - any investor can ask for a Connect screenshot.

The "First-time downloads" tile was removed from that section on request. The remaining
Product page views (536) and Conversion rate (19.9%) still multiply out to roughly 107
downloads that day, so the daily rate is still inferable.

### 1.3 Every date on the page says August

Stale as of 2026-10-01: the header "Live from App Store Connect - Aug 9", "updated Aug 9"
on Store downloads and on Ratings, and `SOCIAL_ASOF='Aug 10, 2026'`. Refresh these whenever
the data behind them is refreshed.

### 1.4 The Home screenshot shows zeros

`screenshots/01-home.jpg` shows "0 Memory Pages, 0 Capsules, 0 Friends" - a fresh test
account - a few hundred pixels below a funnel claiming 1,130 sign-ups and 612 engaged
users. Options: retake it from an account with real content, drop Home from the strip
(Proof of Life leads fine), or caption it as a new test account.

### 1.5 Smaller gaps

- **Google Search Console not connected.** The Search section renders an honest
  "awaiting data" state. Fill `SEO_DATA` and flip `ready:true`.
- **Instagram** post count and engagement rate are unknown (not readable from a public
  profile).
- **saylavy.com footer points at the wrong brand.** Its Instagram, LinkedIn and YouTube
  icons link to `fathom_app`, `fathom-app` and `@fathomapp`. That Instagram account has 2
  followers while the real one, `@saylavy_app`, has 77. This is a bug on the **main
  website**, not in this repo.

---

## 2. Which numbers are real

The page deliberately separates three kinds of number. Keep the distinction.

| Kind | Examples | Rule |
|---|---|---|
| **Measured** | X analytics CSV, Threads post engagement, follower counts, App Store ratings, the App Store Connect daily block | Only from a real source. Say where it came from. |
| **Derived** | Signup rate, funnel percentages, per-country counts, growth chart scale | Computed from measured values. Recompute when an input changes. |
| **Synthetic** | The 90-day daily curve behind the sign-ups and clicks charts | Seeded RNG, scaled so its last 30 days total the real headline. The shape is invented, the total is real. Never present a per-day value from it as fact. |

Anything estimated carries a visible label on the page - for example the European
per-country map tooltips say "Estimated from the regional total". Preserve those labels.

---

## 3. Where each number lives

All inside `index.html`. Search for the constant rather than a line number, they drift.

| What | Search for | Notes |
|---|---|---|
| Downloads total | `DOWNLOAD_TOTAL=1992` | Also hardcoded in the KPI, goal bar, funnel and growth scale. Not consolidated yet. |
| Sign-ups | `SIGNUP_BASE` | See 1.1 |
| Funnel | `const fsteps=[` | Engaged 612, paying 14, with notes stating the percentages |
| Country split | `const GEO={}` | Built from the `locs` shares times the real totals |
| Map tooltips | `onRegionTooltipShow` | `setTip` guards against jsvectormap API differences between versions |
| Search Console | `const SEO_DATA=` | `ready:false` renders the awaiting-data state |
| Social accounts | `const SOCIAL=[` | Instagram, Threads, X, Facebook |
| Top X posts | `const X_TOP=` and `X_PERIOD` | From `account_analytics_content_2026-05-13_2026-08-10.csv` |
| Top Threads posts | `const TH_TOP=` and `TH_PERIOD` | 13 posts, 297 total engagement |
| Spoken summary | `const LINES=[` | **Generated from the constants above.** Update when the data or the story changes. |
| App updates cards | `class="updates"` | Status pills are `.build`, `.talks`, `.beta` |
| Screenshot strip | `class="shots"` | `screenshots/*.jpg`, 560px wide, about 300KB total |

---

## 4. Conventions

- **No long dashes.** Em and en dashes were removed site-wide on request. Use a plain
  hyphen. The only two em dashes left are inside quoted App Store reviews, because those
  are somebody else's words.
- **TestFlight is not shipped.** The App Store still serves 1.0.4 from Jun 16. The redesign
  and the configurable check-in windows exist only in a beta build and carry an
  `IN TESTFLIGHT` pill that says so.
- **Heritage Toronto is not signed.** The card and the spoken summary both say "in talks,
  nothing signed". Keep it that way until something is actually signed.
- **Unknown is not zero.** A platform with no linked account renders "not linked yet", not
  0. A metric that cannot be read renders a dash, never a guess.
- Section accent colours are set per id, for example `#sec-social{--sec:#F6799E}`.

---

## 5. How to verify a change

No build step. Edit `index.html`, commit, push.

Check the inline script parses:

    awk '/^<script>$/{f=1;next} /^<\/script>$/{f=0} f' index.html > /tmp/chk.js
    node --check /tmp/chk.js

Tag balance:

    echo "div $(grep -o '<div' index.html | wc -l) / $(grep -o '</div>' index.html | wc -l)"

Run the page headlessly, which catches script errors that silently kill everything after
them (see section 7):

    npm install jsdom
    node run.js index.html

Browser checks worth repeating, via Playwright MCP. Load the page **at** the target size;
resizing after load measures an artefact, not a bug:

- horizontal overflow: `document.documentElement.scrollWidth > innerWidth`
- rotate portrait to landscape and back, then re-check overflow
- every nav link resolves to a real anchor

---

## 6. Gotchas already hit - do not rediscover these

- **`.ico` is a `<span>`.** `.sales-head span {...}` also matches the section icon. That
  caused both a `margin-left:auto` indent and an `order:3` jump on phones. Always scope to
  `> span:not(.ico)`.
- **Grid children need `min-width:0`.** Without it a Chart.js canvas holding its old pixel
  width props its column open and the page can never shrink back. Rotating a phone left the
  whole dashboard 805px wide on a 390px screen.
- **jsvectormap does not resize itself.** The instance is stored as `usageMap` and
  `updateSize()` is called from `resizeAllCharts`.
- **GitHub Pages serves stale copies.** After a push, curl with a cache-busting query can
  show the new build while the browser still gets the old one. Poll until the new content
  appears, then hard-reload. A "fix did not work" report is often just this.
- **X blocks reads.** x.com returns HTTP 402 to unauthenticated profile requests, so
  follower and post counts must be typed in or taken from an X analytics export.
- **Threads per-post views are private.** Only the profile-level 30-day view count is
  public.
- **Speech synthesis** needs a user gesture to start, and the installed voices differ on
  every device, which is why there is a voice picker rather than a fixed choice.

---

## 7. Test harness (`run.js`, not committed)

Loads `index.html` in jsdom with Chart.js, jsvectormap, IntersectionObserver and
`speechSynthesis` stubbed, runs the inline script, and reports script errors plus whether
the period chips, the share links and the voice summary work.

It is what established that the period chips were never broken - the real problem was that
they only drove two charts far below the fold, so clicking them looked like a no-op.
Recreate it if needed.

---

## 8. Recent history

In rough order: download and sign-up figures updated; Google Play count set; store review
and section dates stamped; App updates section added; Google Search performance section
added (awaiting data); social media section built and wired to the real Instagram, Threads
and X accounts; most engaged X and Threads posts added from real exports; per-country
counts on the world map; sticky section nav with per-section share links; spoken summary;
mobile fixes; and the redesigned-app screenshot strip.

Full detail is in `git log` - the commit messages explain the reasoning, not just the
change.
