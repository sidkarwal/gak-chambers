# GAK Chambers — Static Site

Static clone of gakchambers.com (Sid's law firm, Chandigarh — WordPress/Elementor, hosted on GoDaddy), built to migrate the site off that platform onto a free static host.

## State (2026-09-25)

- Homepage-only clone. Confirmed the live site is a true one-pager — every internal nav link resolves back to `index.html`, no separate subpages exist.
- Deployed repo: `sidkarwal/gak-chambers` on GitHub, served from a `pages` branch (not `main` — standing convention for GitHub Pages deploys).
- Live at https://sidkarwal.github.io/gak-chambers/.
- GitHub Pages custom domain is set to `gakchambers.com` (`CNAME` file in repo root). **DNS was cut over 2026-09-25** via a Cloudflare API token (Zone:DNS:Edit scope) Sid provided: apex A records now point to the 4 GitHub Pages IPs (185.199.108/109/110/111.153, unproxied/DNS-only), `www` CNAME points to `sidkarwal.github.io` (unproxied). MX, SPF, DMARC, autodiscover, and the Intune enterprise-enrollment/registration CNAMEs were left untouched. GitHub is already serving the new clone over HTTP (verified via direct-IP curl) — HTTPS cert provisioning on GitHub's side was still in progress as of cutover; check `gh api repos/sidkarwal/gak-chambers/pages` and enable `https_enforced` once the cert issues.
- Note: this Mac's local DNS cache (`dscacheutil`) can lag behind real propagation by a while — always verify with `dig ... @1.1.1.1` or `curl --resolve` before assuming DNS hasn't taken effect.

## Content edits made per Sid

- Removed Sahil Chhabra's team card entirely (photo, name, title, email, LinkedIn icon).
- Changed Rana Gurtej Singh's title from "Principal Associate" to "Counsel".
- Replaced all "GAK Partners" brand text with "GAK Chambers" site-wide.
- 2026-09-25: emails updated too — all 4 addresses (`nikhil@`, `rana@`, `contact@`, `careers@`) changed from `@gakpartners.com` to `@gakchambers.com` (reversing the earlier instruction to leave them as-is). Not verified whether these mailboxes are actually provisioned in the M365 tenant yet — MX for gakchambers.com points to Outlook, but that's domain-level, not per-mailbox.
- Centered the (now 2-person) team row on desktop: scoped CSS added before `</head>` —
  `.elementor-element-2627f03 > .elementor-container { justify-content: center }` inside `@media (min-width: 1025px)`.

## Cleanup done to make the mirror fully self-contained

wget's mirror pulled the rendered homepage, but a few things still called out to the internet or carried WordPress cruft — all fixed:
- Google Fonts (Roboto, Roboto Slab) were loading from `gakpartners.com` — downloaded all 25 `.woff2` files locally into `wp-content/uploads/elementor/google-fonts/fonts/` and rewrote the CSS to relative paths.
- Removed a GoDaddy analytics/tracking script block that phoned home to `img1.wsimg.com` on every page load.
- Removed dead WordPress-only endpoints (REST API link, RSD/xmlrpc link, oEmbed links) and deleted the orphaned RSS/REST stub files wget had saved for them.
- Fixed the favicon `msapplication-TileImage` meta tag to point locally instead of the live URL.

No other external calls remain except intentional links: LinkedIn profiles/company page, and the credit link to sidkarwal.com.

## Next step: enable HTTPS enforcement

DNS cutover is done (see above). Remaining: once GitHub finishes issuing the Let's Encrypt cert for `gakchambers.com` (check `gh api repos/sidkarwal/gak-chambers/pages`), enable "Enforce HTTPS". The Cloudflare MCP server available in this environment has no DNS record tools (Workers/KV/D1/R2 only) and the `wrangler` OAuth token has no DNS scope either — DNS edits went through the Cloudflare REST API directly with a scoped API token Sid provided ad hoc, not through any standing tool.

## Repo mechanics

- Local working copy = deploy checkout. Git remote `origin` = `https://github.com/sidkarwal/gak-chambers.git`, on branch `pages`.
- To ship a change: edit locally, `git add -A && git commit -m "..." && git push origin pages`. No build step — GitHub Pages serves the branch root as-is.
