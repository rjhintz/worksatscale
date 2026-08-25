+++
title = "Migrating worksatscale.com from Namecheap to GitHub Pages"
date = 2026-08-25
draft = true
+++

Works@Scale's website was originally hosted on Namecheap shared hosting via cPanel (Namecheap's web-hosting control panel), deployed by zipping local files and uploading manually. This note documents the migration to a git-based, automatically-deployed setup, built with Hugo (the tool that converts the site's plain-text content files into the actual web pages visitors see).

**Source control.** Created a public GitHub repository (`rjhintz/worksatscale`) and moved the Hugo site source there, replacing the desktop-only workflow. A short-lived, narrowly-scoped access token (Contents, Pages, and Workflows permissions only, on this one repository) was used to push the initial commit and configure the repo, then revoked once no longer needed.

**Generating an access token.** To let Claude push commits and configure the repo directly, a GitHub personal access token was created at github.com/settings/tokens?type=beta: fine-grained, scoped to "Only select repositories" → `worksatscale` only, with Repository permissions set to Contents (Read and write), Pages (Read and write), and Workflows (Read and write), and a 7-day expiration. This narrow scope meant the token could push to this one repository and nothing else — no access to other repos, account settings, or billing. It was pasted into the chat for that session's work, then revoked once no longer needed. This became the standard pattern for every subsequent work session: generate a fresh short-lived token, do the work, revoke or let it expire.

**Automated deployment.** Added a GitHub Actions workflow (`.github/workflows/hugo.yml`) — GitHub's own automated task runner — that rebuilds the site with Hugo and deploys it to GitHub Pages on every push to `main`. The Hugo version required one correction: initially pinned to 0.139.0, which failed because the Ananke theme requires Hugo 0.146.0 or newer; repinned to 0.165.0, which resolved it.

**GitHub Pages and custom domain.** Enabled Pages with GitHub Actions as the build source (i.e., told GitHub Pages to publish whatever the Actions workflow produces, rather than a raw branch of files). Configured `worksatscale.com` as the custom domain. Both steps required the repo's Administration permission, which the scoped access token deliberately excluded, so they were done manually via the GitHub UI.

**DNS cutover.** At Namecheap's Advanced DNS settings, replaced the A record (the DNS setting that points a domain at a specific server's numeric address) pointing at the old cPanel server with four A records pointing to GitHub Pages' IP addresses (185.199.108.153, .109.153, .110.153, .111.153), and added a CNAME record (a DNS setting that points one address, like `www`, at another name rather than a raw number) for `www` pointing to `rjhintz.github.io`. A leftover A record for `www` (pointing at the old host) was found and removed after it caused a certificate mismatch error; once removed, DNS resolved correctly for both the root domain and `www`.

**SSL.** GitHub Pages auto-issues and renews its own certificate (the credential that lets a browser show the padlock/HTTPS and encrypt traffic to the site) once DNS is verified. There was a delay between DNS resolving correctly and the certificate actually issuing, during which browsers showed cache-related "Not Secure" warnings; confirmed resolved via an incognito window once the certificate was live. The Namecheap-issued PositiveSSL/Comodo certificate tied to the old hosting plan (valid to Jan 11, 2027) is no longer in use.

**A real bug found and fixed.** The site's custom stylesheet (`custom.css`, the file controlling fonts, colors, and layout details beyond the theme's defaults) had never actually been loading in any browser, on any prior version of the site — it lived at `assets/css/custom.css`, a path the Ananke theme's asset bundler (the internal process that gathers and prepares style files for the finished site) doesn't check, so the link in the page's `<head>` (the part of a web page's code that isn't visible content, but tells the browser what to load) pointed at a file that didn't exist in the built output. Moved it to `static/css/custom.css`, which Hugo copies through as-is with no theme-specific processing involved.

**Typography and color fixes**, once the stylesheet was actually loading:

- Switched the site to a system font stack (a prioritized list of fonts — the browser uses the first one available on the visitor's device, such as San Francisco on a Mac or Segoe UI on Windows, falling back to Arial if none of the preferred ones are present).
- Fixed link colors (black in body text, white in nav/header) — this required including `:link`/`:visited` in the selectors. These are pseudo-classes (special CSS instructions that target a specific state of an element — here, whether a link has been visited before) and were needed because browsers' own default link-color rule uses a pseudo-class internally, which otherwise outranks a plain, simpler style rule.
- Found and overrode two theme utility classes (`.serif` and `.athelas` — pre-built style shortcuts bundled with the Ananke theme, each tied to a specific look) that the page template applies directly to body copy and page titles on every content page, forcing a serif font regardless of the site-wide font rule. A short-lived monospace treatment for the About page was tried and then reverted in favor of one consistent typeface site-wide.
- Reduced the site's overall type scale (the set of font sizes used for headings, body text, and other elements, sized consistently relative to each other) roughly 20%, bringing body copy from 20px down to 16px, with headings scaled proportionally to preserve the existing hierarchy.

**Homepage hero** (the large image-and-headline block at the top of the page). Brightened the background photo, added a subtle bottom-weighted gradient overlay for text legibility (replacing the previous flat/no overlay), and reduced an oversized headline (5rem default) to a more balanced size (2.25–3rem).

**Contact page image.** Re-cropped to show more of the visitor crowd and less wall/ceiling, per direct feedback on the original framing.

**Email.** Investigation revealed `info@worksatscale.com` is very likely provisioned as part of the cPanel hosting plan itself, not a separate Namecheap Private Email subscription — meaning it will stop working once that hosting plan lapses unless separately migrated to Namecheap's free Email Forwarding before then.

**Version tagging.** Once the migration and initial cleanup were stable, an annotated git tag (`migration-complete`) — a named bookmark pointing at a specific point in the project's history — was added at that commit as a reference point, not a formal versioning scheme for every change.

**Net effect.** Hosting costs eliminated entirely (previously ~$55.88/yr for the Namecheap Value shared hosting plan, now $0 on GitHub Pages). Ongoing cost is limited to domain registration (~$14–16/yr at Namecheap, cheaper alternatives like Porkbun exist for the 2027 renewal) and, if kept, email (~$15–17/yr, or $0 via free forwarding to an existing inbox).
