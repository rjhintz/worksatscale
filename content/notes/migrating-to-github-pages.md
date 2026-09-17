+++
title = "Migrating worksatscale.com from Namecheap to GitHub Pages"
date = 2026-09-16
draft = false
+++

Works@Scale's website was originally hosted on Namecheap shared hosting, using their cPanel tool, deployed by zipping/compressing local files and uploading manually. This note documents the migration to a git-based, automatically-deployed setup in August 2026.

**Source control.** Created a public GitHub repository (`rjhintz/worksatscale`) in a largely dormant GitHub account and moved the Hugo site source there, replacing the desktop-only workflow. The Hugo files were created by an earlier AI-assisted migration from WordPress.

**Generating an access token.** To let Claude push commits and configure the repo directly, a GitHub personal access token was created at github.com/settings/tokens?type=beta: fine-grained, scoped to "Only select repositories" → worksatscale only, with Repository permissions set to Contents (Read and write), Pages (Read and write), and Workflows (Read and write), and a 7-day expiration. This narrow scope meant the token could push to this one repository and nothing else, with no access to other repos, account settings, or billing. It was pasted into the chat for that session's work, then revoked once no longer needed. This became the standard pattern for every subsequent work session: generate a fresh short-lived token, do the work, revoke, or let it expire.

**Automated deployment.** Added a GitHub Actions workflow (`.github/workflows/hugo.yml`) that rebuilds the site with Hugo and deploys it to GitHub Pages on every push to main. The Hugo version required one correction: initially pinned to 0.139.0, which failed because the Ananke theme requires Hugo 0.146.0 or newer; repinned to 0.165.0, which resolved it.

**GitHub Pages and custom domain.** Enabled Pages with GitHub Actions as the build source, and configured worksatscale.com as the custom domain. Both steps required the repo's Administration permission, which the scoped access token deliberately excluded, so they were done manually via the GitHub UI.

**DNS cutover.** At Namecheap's Advanced DNS settings, replaced the A record pointing at the old cPanel server with four A records pointing to GitHub Pages' IP addresses (185.199.108.153, .109.153, .110.153, .111.153), and added a CNAME for www pointing to rjhintz.github.io. A leftover A record for www (pointing at the old host) was found and removed after it caused a certificate mismatch error. Once removed, DNS resolved correctly for both the root domain and www.

**SSL.** GitHub Pages auto-issues and renews its own certificate once DNS is verified. There was a delay between DNS resolving correctly and the certificate actually issuing, during which browsers showed cache-related "Not Secure" warnings; confirmed resolved via an incognito window once the certificate was live. The Namecheap-issued PositiveSSL/Comodo certificate tied to the old hosting plan, although valid to Jan 11, 2027, is no longer in use.

**A bug found and fixed.** The site's custom stylesheet (custom.css) had never actually been loading in any browser, on any prior version of the site. It lived at `assets/css/custom.css`, a path the Ananke theme's asset bundler doesn't check, so the link in the page `<head>` pointed at a file that didn't exist in the built output. Moved it to `static/css/custom.css`, which Hugo copies through verbatim with no theme-specific resolution logic involved.

**Typography and color fixes**, once the stylesheet was actually loading:

- Switched the site to a system font stack (native OS font on each device, falling back to Arial).
- Fixed link colors (black in body text, white in nav/header). This required including `:link`/`:visited` in the selectors, since browsers' own default link-color rule uses a pseudo-class that otherwise outranks a plain element selector.
- Found and overrode two theme utility classes (`.serif` and `.athelas`) that the page template applies directly to body copy and page titles on every content page, forcing a serif font regardless of the site-wide font rule. A short-lived monospace treatment for the About page was tried and then reverted in favor of one consistent typeface site-wide.
- Reduced the site-wide type scale roughly 20%, bringing body copy from 20px down to 16px, with headings scaled proportionally to preserve the existing hierarchy.

**Homepage hero.** Brightened the background photo, added a subtle bottom-weighted gradient overlay for text legibility (replacing the previous flat/no overlay), and reduced an oversized headline (5rem default) to a more balanced size (2.25–3rem).

**Contact page image.** Re-cropped to show more of the visitor crowd and less wall/ceiling, per direct feedback on the original framing.

**Email.** Investigation revealed info@worksatscale.com is very likely provisioned as part of the cPanel hosting plan itself, not a separate Namecheap Private Email subscription, meaning it would have stopped working once that hosting plan lapsed. To address this, free Namecheap Email Forwarding was set up before the cutover date, redirecting info@worksatscale.com to an existing personal inbox. This was tested and confirmed working, with a message sent from an unrelated third-party address arriving correctly. The volume of email isn't significant, but good practice is that people sending email to a worksatscale.com address should expect it to be received.

**Version tagging.** Once the migration and initial cleanup were stable, an annotated git tag (`migration-complete`) was added at that commit as a named reference point, not a formal versioning scheme for every change, just a bookmark for "the site was fully off Namecheap and stable" as of that point.

**Net effect.** Hosting costs eliminated entirely, previously ~$55.88/yr for the Namecheap Value shared hosting plan, now $0 on GitHub Pages. Ongoing cost is limited to domain registration (~$14–16/yr at Namecheap, cheaper alternatives like Porkbun exist for the 2027 renewal) and, if kept, email (~$15–17/yr, or $0 via free forwarding to an existing inbox).

Namecheap's own Help documentation, checked directly as of this writing, states that hosting plans expire without further action after a grace period when they aren't renewed, though what actually happens in practice may not perfectly match documented policy. Before that expiration, the old cPanel account's files were reviewed for anything worth archiving separately; nothing was found worth keeping.
