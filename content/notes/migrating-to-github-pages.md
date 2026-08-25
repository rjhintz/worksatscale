+++
title = "Migrating worksatscale.com from Namecheap to GitHub Pages"
date = 2026-08-25
draft = true
+++

Works@Scale's website was originally hosted on Namecheap shared hosting (cPanel), deployed by zipping local files and uploading manually. This note documents the migration to a git-based, automatically-deployed setup.

**Source control.** Created a public GitHub repository (`rjhintz/worksatscale`) and moved the Hugo site source there, replacing the desktop-only workflow. A short-lived, narrowly-scoped access token (Contents, Pages, and Workflows permissions only, on this one repository) was used to push the initial commit and configure the repo, then revoked once no longer needed.

**Automated deployment.** Added a GitHub Actions workflow (`.github/workflows/hugo.yml`) that rebuilds the site with Hugo and deploys it to GitHub Pages on every push to `main`. The Hugo version required one correction: initially pinned to 0.139.0, which failed because the Ananke theme requires Hugo 0.146.0 or newer; repinned to 0.165.0, which resolved it.

**GitHub Pages and custom domain.** Enabled Pages with GitHub Actions as the build source, and configured `worksatscale.com` as the custom domain. Both steps required the repo's Administration permission, which the scoped access token deliberately excluded, so they were done manually via the GitHub UI.

**DNS cutover.** At Namecheap's Advanced DNS settings, replaced the A record pointing at the old cPanel server with four A records pointing to GitHub Pages' IP addresses (185.199.108.153, .109.153, .110.153, .111.153), and added a CNAME for `www` pointing to `rjhintz.github.io`. A leftover A record for `www` (pointing at the old host) was found and removed after it caused a certificate mismatch error; once removed, DNS resolved correctly for both the root domain and `www`.

**SSL.** GitHub Pages auto-issues and renews its own certificate once DNS is verified. There was a delay between DNS resolving correctly and the certificate actually issuing, during which browsers showed cache-related "Not Secure" warnings; confirmed resolved via an incognito window once the certificate was live. The Namecheap-issued PositiveSSL/Comodo certificate tied to the old hosting plan (valid to Jan 11, 2027) is no longer in use.

**A real bug found and fixed.** The site's custom stylesheet (`custom.css`) had never actually been loading in any browser, on any prior version of the site — it lived at `assets/css/custom.css`, a path the Ananke theme's asset bundler doesn't check, so the link in the page `<head>` pointed at a file that didn't exist in the built output. Moved it to `static/css/custom.css`, which Hugo copies through verbatim with no theme-specific resolution logic involved.

**Typography and color fixes**, once the stylesheet was actually loading:

- Switched the site to a system font stack (native OS font on each device, falling back to Arial).
- Fixed link colors (black in body text, white in nav/header) — this required including `:link`/`:visited` in the selectors, since browsers' own default link-color rule uses a pseudo-class that otherwise outranks a plain element selector.
- Found and overrode two theme utility classes (`.serif` and `.athelas`) that the page template applies directly to body copy and page titles on every content page, forcing a serif font regardless of the site-wide font rule. A short-lived monospace treatment for the About page was tried and then reverted in favor of one consistent typeface site-wide.
- Reduced the site-wide type scale roughly 20%, bringing body copy from 20px down to 16px, with headings scaled proportionally to preserve the existing hierarchy.

**Homepage hero** (the large image-and-headline block at the top of the page). Brightened the background photo, added a subtle bottom-weighted gradient overlay for text legibility (replacing the previous flat/no overlay), and reduced an oversized headline (5rem default) to a more balanced size (2.25–3rem).

**Contact page image.** Re-cropped to show more of the visitor crowd and less wall/ceiling, per direct feedback on the original framing.

**Email.** Investigation revealed `info@worksatscale.com` is very likely provisioned as part of the cPanel hosting plan itself, not a separate Namecheap Private Email subscription — meaning it will stop working once that hosting plan lapses unless separately migrated to Namecheap's free Email Forwarding before then.

**Net effect.** Hosting costs eliminated entirely (previously ~$55.88/yr for the Namecheap Value shared hosting plan, now $0 on GitHub Pages). Ongoing cost is limited to domain registration (~$14–16/yr at Namecheap, cheaper alternatives like Porkbun exist for the 2027 renewal) and, if kept, email (~$15–17/yr, or $0 via free forwarding to an existing inbox).
