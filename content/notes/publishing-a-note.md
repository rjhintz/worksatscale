+++
title = "Publishing a tech note to the Notes section"
date = 2026-09-16
draft = false
+++

A step-by-step reference for adding a new technical note to the site.

## Background

This site is built with [Hugo](https://gohugo.io/), a free to use, open source tool that converts plain text files into HTML.

Instead of writing HTML directly, you write in [Markdown](https://www.markdownguide.org/getting-started/), the same formatting language used in note-taking apps like Obsidian, Bear, and Typora. Hugo converts from Markdown to HTML during the automated build that runs after a "push," not as you write. The "push" sends saved, finished work, called commits, to a repository, a "repo," on GitHub.

The site's source files live in a public GitHub repository called `rjhintz/worksatscale`. The repo is public because GitHub's free tier requires it for Pages hosting. That is, anyone can view the source, though only you or an AI helper with an authorizing token can push changes.

You write the actual content of a change, whether typing directly into GitHub's web interface, editing on your own computer, or directing an AI helper to make the edit on your behalf in a chat session.

Once a change is ready, it needs to be pushed to that repository. If you're working through an AI helper, the helper does this step for you once you say the change is ready, using a temporary access token you generate and provide for that session. The same token can cover multiple changes, but eventually expires based on the time period you set when you create it and it then needs replacing. A seven day expiry is ok.

If you're working from your own computer, you run the git commands yourself:

* `git add` to stage the changed file
* `git commit` to save a snapshot with a short commit message describing the change
* `git push` to send it to GitHub

This requires the software package git to be installed locally, plus some form of credential set up on that machine so that the GitHub site recognizes you.

If you're returning to this after a break and aren't sure whether you have a valid credential set up already, the simplest test is to try a push. If it completes without asking you for anything, a credential is already stored on your computer from before. If it prompts you for a token or password, either none was ever saved or GitHub stopped accepting the old one and it's time to set up a new one.

One credential option is a personal access token, generated the same way as described earlier, then remembered locally after its first use through your computer's built-in credential storage, so you're not asked to re-enter it on every push.

### Mac credential storage and management

For more context, here's an explanation of Mac credential storage and management.

**Keychain Access setup.** Nothing needs to be actively set up in the way you'd configure a new app because Keychain Access is a built-in part of macOS itself, not something you install or turn on. For git specifically, the piece that connects the two is a small program called a "credential helper" that comes bundled with git (usually already active by default on a Mac, though it can be checked/configured with `git config --get credential.helper`, which should show something like `osxkeychain` if it's active).

**Populating Keychain Access.** You don't manually add anything for git's purposes. The first time you push and enter your personal access token at the prompt, git's credential helper automatically hands that token to Keychain and stores it there right then, tagged to that specific GitHub address. Nothing else needs to happen because it's a side effect of a successful push, not a separate step.

**Viewing and managing what's stored.** Open Keychain Access (search for it in Spotlight since it's no longer in the regular Applications folder as of recent macOS versions) and search for "github.com" there. It would show the stored token as an entry, which you can inspect or manually delete if needed.

Both Keychain Access and the newer Passwords app read from the same underlying storage, but Passwords doesn't manage the kind of entry git creates, so only Keychain Access is relevant here.

Practically, for a git credential you'd almost never need to open Keychain Access. It's meant to work invisibly in the background. The main reason you'd go looking is to manually delete a stale or revoked credential if git ever starts failing after a token expires and doesn't prompt you correctly on its own.

### SSH key

The other credential option is an SSH key. This is a cryptographic pair generated once with a single command (`ssh-keygen`), producing a private key that stays on your computer permanently and a public key you paste into your GitHub account settings. Navigate to Settings (under profile at right) → SSH and GPG keys → New SSH key.

When you push, GitHub and your computer perform a handshake that only the matching private key can pass, so your identity is confirmed without the key itself ever being sent anywhere. This is a different mechanism from a token, and a different kind of storage too. The private key isn't held in a credential manager but sits as a protected file on your computer, optionally guarded by its own passphrase. Either is a one-time setup, unlike the short-lived tokens described above for working through an AI helper.

An SSH key doesn't expire on its own the way the short-lived tokens described above do. Instead it stays valid until you remove it. To revoke one, go to Settings → SSH and GPG keys on GitHub and delete it there, which takes effect immediately. Removing it stops that key from working against your account going forward, but the private key file still exists on your own computer exactly as before, so if a key is ever actually compromised rather than just being retired, the safer response is to generate a brand new key pair rather than reuse the old one.

Long-lived SSH keys carry a real tradeoff. Unlike short-lived tokens, a compromised key stays valid indefinitely unless manually revoked, so protecting the key itself (a passphrase, full-disk encryption) matters more than it would for something that expires on its own.

The moment a push completes, [GitHub Actions](https://github.com/features/actions), an automated process that runs on GitHub's own servers without a manual trigger, takes over. It rebuilds the site and publishes it live, usually within a minute or two. There's no automatic notification by default; the reliable way to confirm it finished is to check the repository's Actions tab for a green checkmark, or just visit the live site directly after waiting a bit. There is no manual file upload step, as with the earlier Namecheap/cPanel process.

## Step 1: Generating a GitHub access token

Before an AI helper (or any tool) can make changes to the repository on your behalf, it needs a GitHub "fine grained" personal access token. This is a temporary password-like credential scoped narrowly to just this task that you generate yourself. An AI helper cannot create it for you because the token acts as a highly secure, password-like credential linked directly to your specific GitHub account and its permissions.

1. Go to https://github.com/settings/tokens?type=beta (this is your GitHub account's token management page).
2. Click "Generate new token."
3. Give it a name you'll recognize later (e.g. "[date stamp] worksatscale notes update").
4. Set expiration to something short such as 7 days, which is plenty for a single editing session, even with revisions.
5. Under Repository access, choose "Only select repositories" and pick worksatscale.
6. Under Permissions, find "Repository permissions" and set:
   - Contents: Read and write
   - Pages: Read and write
   - Workflows: Read and write (needed because publishing can touch the automated build process)
7. Double-check the permissions actually saved as "Read and write" — GitHub's UI has occasionally shown a token as having repo access in summary while the specific permission (e.g. Contents) was still set to read-only underneath, which silently blocks any push.
8. If a push fails with a permission error despite the token looking correctly scoped, this is the first thing to check.
9. Click Generate, then copy the token (it starts with `github_pat_...`) and paste it into the chat when asked.
10. When you're done with the session, revoke it from that same token page — it's a temporary credential, not something to leave active indefinitely, though a 7 day expiry can be ok.

## Step 2: Understanding the file format (front matter)

Every note is a single file in the repository at `content/notes/<some-name>.md`. The `.md` means it's a Markdown file. At the top of that file is a block that looks like this:

```toml
+++
title = "Your Note Title"
date = 2026-08-24
draft = false
+++
```

This block is called front matter, and the format inside it is called TOML (pronounced "TOM-uhl," like "camel" with a T) — a simple way of writing key = value settings. Hugo reads this block to know things about the page that aren't part of the visible content itself such as what to call it, when it was written, and whether it should be shown to visitors yet. The `+++` lines mark the start and end of this block; everything below the second `+++` is the actual note content, written in Markdown.

Here's what each field does:

- title — The headline shown both on the note's own page and in the list of notes on the Notes section homepage. Keep it short enough to read easily in a list.
- date — YYYY-MM-DD format. Controls the order notes appear in (newest first, automatically — you don't have to do anything else to make that happen). Use the date you're actually publishing it, not the date you started writing. Zero-padded, always two digits for month and day, regardless of whether the number is single-digit. TOML dates follow the standard ISO format, so September 5 is written 2026-09-05.
- draft — Controls visibility. draft = true means the file exists in the repository (so it's safely saved and backed up) but does not appear anywhere on the live website. draft = false means it's live and visible to anyone who visits the Notes section. This is your safety net. You can write and rewrite a note over several sessions with draft = true, and only flip it to false once you're actually ready to publish.

## Step 3: Pick a filename

The filename becomes part of the web address for that note. For example, a file named `content/notes/github-pages-migration.md` will be reachable at `worksatscale.com/notes/github-pages-migration/`. Use lowercase letters and hyphens instead of spaces with no capital letters, no underscores, and no other punctuation.

## Step 4: Write the content

Below the front matter block, write in Markdown: `#` for headings, `**bold**` for bold text, blank lines between paragraphs, and so on.

If you want to show a snippet of code or a config file, wrap it in three backticks. The site automatically styles anything inside those backtick blocks as monospace font, light gray background so you don't need to add any formatting yourself.

## Step 5: Publish

Once the file is how you want it and marked `draft = false`, it needs to be committed and pushed to the repository. This is the two-step process git uses to save a change and send it to GitHub. If you're doing this through an AI helper, just say you're ready, and the AI helper will handle the commit and push using the access token from Step 1 you shared, so make sure it's still valid. Within a minute or two, GitHub's automated build process picks up the change and the note appears live on the site. There's no separate action needed to "publish" beyond that push.
