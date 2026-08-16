# LURE'AY — Luxury Unisex Salon & Makeover Studio

A single-file, self-contained website built for **LURE'AY**, a real luxury unisex salon in Banjara Hills, Hyderabad, India.

**Live demo:** https://majestic-medovik-22cf8a.netlify.app/
**Built and designed by:** Anwarullah Khan Nuzair (Prime Web Design Service)
**Final production hosting:** Hostinger (this Netlify link is a client-review demo, not the permanent home)

---

## What this is

One `index.html` file — no build step, no framework, no backend. All CSS lives in a `<style>` block, all JavaScript in a `<script>` block, and every image is embedded directly as a base64 `data:` URI so the page is fully portable. The three video files ride alongside as separate sibling `.mp4` files (too large to base64 without bloating the page).

```
index.html                 the entire site
video-lashes-offer.mp4     Influencers section video
video-testimonials.mp4     Influencers section video
video-guest-visit.mp4      Influencers section video
og-image.jpg               real social-share preview image (1200x630)
robots.txt / sitemap.xml   basic SEO crawl files
```

## Design approach

Every piece of content is **real** — real address, real phone number, real Google rating (4.7★ / 500+ reviews), real interior photos sourced from the salon's own Google Maps listing, real testimonial quotes, real tagline ("Your Lure Is Magical — Let Them Take The Plunge," taken directly from the salon's own wall decal). Nothing on this site is fabricated or placeholder content.

Palette (gold / espresso / cream / blush) was sampled directly from the salon's real interior photos rather than a generic template. Fonts: Playfair Display for headings, Great Vibes for the script accent, Poppins for body text.

## Tech / features

- Scroll-triggered fade-up reveal animations (see "Bugs & Fixes" below for how this evolved)
- Netlify Forms-wired consultation form (with a spam honeypot field)
- Live embedded Google Maps
- Click-to-play video cards with real extracted poster frames
- Full on-page SEO: canonical URL, Open Graph, Twitter Card, JSON-LD `BeautySalon` structured data, sitemap.xml, robots.txt

## Deployment

Deployed via Netlify CLI:
```
netlify deploy --auth=<personal-access-token> --site=majestic-medovik-22cf8a --dir=. --prod
```
(Video files exceed the 10MB browser-upload limit, so CLI deploy from disk is required rather than Netlify's drag-and-drop UI.)

---

## Bugs found and fixed during development

### 1. Wrong brand name in multiple places
The nav, hero, page title, and footer had drifted to say "Luxury Unisex Salon **& Spa**" instead of the real name, "Luxury Unisex Salon **& Makeover Studio**" (confirmed against the actual Google Maps listing and the salon's real logo image). Fixed all four occurrences — legitimate uses of the word "Spa" elsewhere (room/service names like "Body Spa & Massage") were left untouched since those are correct.

### 2. Scroll-reveal animation didn't work on mobile touch scrolling
The "fade up as you scroll" effect on service/treatment cards worked perfectly with a mouse wheel or trackpad, but silently failed on real phones during a finger-swipe scroll.

- **First attempt:** tuned the `IntersectionObserver`'s `threshold` and `rootMargin` — helped, but didn't fully fix it.
- **Root cause:** mobile Safari/Chrome can throttle or stall `IntersectionObserver` callbacks specifically during momentum/touch-scroll animations. This is a real, documented mobile browser quirk, not a code logic bug.
- **Real fix:** added a second, independent detection method that doesn't depend on `IntersectionObserver` timing at all — a `revealVisibleCards()` function that directly checks each card's `getBoundingClientRect()` position, wired to fire on both `scroll` and `touchmove` events (passive listeners, `requestAnimationFrame`-throttled). Now the reveal fires reliably regardless of *how* the user scrolls.

### 3. Videos didn't play on the live site
A prior deploy had only uploaded the single `index.html` file — not the three sibling `.mp4` files it references via relative `src=`. The videos genuinely weren't on the server at all. Fixed by deploying the whole folder together (not just the HTML) via Netlify CLI. Verified afterward with direct `fetch()` + HTTP Range requests confirming all three videos return correct byte sizes and support `206 Partial Content` streaming.

### 4. The entire site was accidentally set to Private
Netlify silently defaults newly-created projects to "Private" visibility. Every check during development looked fine because it was done from an already-logged-in browser session — but a real, logged-out visitor was getting a `401 Unauthorized` error instead of the website. Caught by testing with a clean `curl` request (no cookies) and seeing the 401. Fixed via Netlify's Project configuration → Visitor access → set to Public (both Production and Deploy Previews). Re-verified with the same clean request afterward, now returning `200 OK`.

**Lesson:** never trust "it loads fine" from an authenticated browser tab alone — always verify public reachability with a cookie-less request.

### 5. Page felt "unstable" on mobile — sliding sideways, flashing white space
Real root cause: the contact form's spam-protection honeypot field was hidden using the old `position:absolute; left:-9999px;` trick. This silently made the entire page's scrollable width thousands of pixels wider than the actual screen. On mobile, the natural rubber-band overscroll bounce had all that extra empty space to slide into — revealing the browser's default white background beyond the site's actual dark theme.

**Fix (two parts):**
1. Replaced the honeypot's hiding technique with the standard accessible "visually-hidden" CSS pattern (`width:1px; height:1px; overflow:hidden; clip:rect(0,0,0,0);`) instead of flinging it off-canvas.
2. Added `overflow-x:hidden` and an explicit background color to **both** `html` and `body` (not just `body`) as a safety net against any future overflow.

Verified by temporarily disabling the overflow clipping and confirming zero horizontal overflow remained.

---

*Built with [Claude Code](https://claude.com/claude-code).*
