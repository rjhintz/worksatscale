+++
title = "Publishing a note to the Notes section"
date = 2026-08-25
draft = true
+++

A step-by-step reference for adding a new technical note to the site, written for whenever you're returning to this after a long gap and need the full picture again — not just the commands.

### Background you'll want back in your head

This site is built with **Hugo**, a tool that takes plain text files and turns them into the actual HTML pages you see in a browser. You don't write HTML directly — you write Markdown (the same kind of light formatting used in most note-taking apps), and Hugo does the conversion. The site's source files live in a **GitHub repository** called `rjhintz/worksatscale`, and every time a change is pushed there, an automated process (**GitHub Actions**) rebuilds the site and publishes it live within a minute or two. There is no manual "upload" step anymore — that was the old Namecheap/cPanel way of doing things, which this setup replaced.

### Step 1: Generate a GitHub access token

Before Claude (or any tool) can make changes to the repository on your behalf, it needs a **personal access token** — a temporary password-like credential scoped narrowly to just this task. You generate this yourself; Claude cannot create it for you.

1. Go to https://github.com/settings/tokens?type=beta (this is your GitHub account's token management page).
2. Click "Generate new token."
3. Give it a name you'll recognize later (e.g. "worksatscale notes update").
4. Set expiration to something short — 7 days is plenty for a single editing session.
5. Under **Repository access**, choose "Only select repositories" and pick `worksatscale`.
6. Under **Permissions**, find "Repository permissions" and set:
   - **Contents**: Read and write
   - **Pages**: Read and write
   - **Workflows**: Read and write (needed because publishing can touch the automated build process)
7. Click Generate, then copy the token (it starts with `github_pat_...`) and paste it into the chat when asked.
8. **Before pasting it anywhere, or once done using it**, double-check the permissions actually saved as "Read and write" — GitHub's UI has occasionally shown a token as having repo access in summary while the specific permission (e.g. Contents) was still set to read-only underneath, which silently blocks any push. If a push fails with a permission error despite the token looking correctly scoped, this is the first thing to check.
9. **When you're done with the session**, revoke it from that same token page — it's a temporary credential, not something to leave active indefinitely.

### Step 2: Understand the file format (front matter)

Every note is a single file living in the repository at `content/notes/<some-name>.md`. The `.md` means it's a Markdown file. At the very top of that file is a block that looks like this:

```toml
+++
title = "Your Note Title"
date = 2026-08-24
draft = false
+++
```

This block is called **front matter**, and the format inside it is called **TOML** (pronounced "TOM-uhl," like "camel" with a T) — a simple way of writing `key = value` settings. Hugo reads this block to know things about the page that aren't part of the visible content itself: what to call it, when it was written, and whether it should be shown to visitors yet. The `+++` lines mark the start and end of this block; everything below the second `+++` is the actual note content, written in plain Markdown.

Here's what each field does:

- **`title`** — The headline shown both on the note's own page and in the list of notes on the Notes section homepage. Keep it short enough to read comfortably in a list.
- **`date`** — Controls the order notes appear in (newest first, automatically — you don't have to do anything else to make that happen). Use the date you're actually publishing it, not the date you started writing.
- **`draft`** — This is the one that controls visibility. `draft = true` means the file exists in the repository (so it's safely saved and backed up) but does **not** appear anywhere on the live website. `draft = false` means it's live and visible to anyone who visits the Notes section. This is your safety net: you can write and rewrite a note over several sessions with `draft = true`, and only flip it to `false` once you're actually ready to publish.

### Step 3: Pick a filename

The filename becomes part of the web address for that note. For example, a file named `content/notes/github-pages-migration.md` will be reachable at `worksatscale.com/notes/github-pages-migration/`. Use lowercase letters and hyphens instead of spaces — no capital letters, no underscores, no punctuation.

### Step 4: Write the content

Below the front matter block, write in plain Markdown: `#` for headings, `**bold**` for bold text, blank lines between paragraphs, and so on. If you want to show a snippet of code or a config file, wrap it in three backticks. The site automatically styles anything inside those backtick blocks (monospace font, light gray background) — you don't need to add any formatting yourself.

### Step 5: Publish

Once the file exists with `draft = false` and the content is how you want it, it needs to be **committed and pushed** to the repository — the two-step process git uses to save a change and send it to GitHub. If you're doing this through Claude: just say you're ready, and Claude will handle the commit and push using the access token from Step 1. Within a minute or two, GitHub's automated build process picks up the change and the note appears live on the site — no separate action needed to "publish" beyond that push.

### One thing to remember about the Notes section specifically

The Notes section displays the same full-photo header as every other page on the site, using the image set in `content/notes/_index.md`'s front matter (`featured_image`) — this is the current, intended look.
