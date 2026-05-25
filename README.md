# GitHub + jsDelivr Image Hosting — Setup Guide

This guide walks you through setting up **GitHub + jsDelivr** as the image host for the MCQ JSON Tool's Figure Updater. This is the recommended option because jsDelivr has **no rate limits** and serves images from a fast global CDN — unlike Google Drive, which can throttle image requests on busy quizzes.

**Total setup time: about 5 minutes.** You only do this once.

---

## How it works

When you crop a figure and click *Apply Figures to This Question*, the tool:

1. Commits the cropped image file into a GitHub repository you own.
2. Builds a jsDelivr CDN link to that file, in the form:
   `https://cdn.jsdelivr.net/gh/OWNER/REPO@BRANCH/PATH/filename.webp`
3. Writes that CDN link into your quiz JSON.

jsDelivr automatically mirrors any public GitHub repository, so once the image is committed, its CDN URL works immediately and forever — with no per-request limits.

---

## Step 1 — Create a GitHub account

Skip this step if you already have one.

1. Go to `https://github.com/signup`.
2. Enter your email, a password, and a username.
3. Verify your email address when GitHub sends the confirmation.

A free account is all you need.

---

## Step 2 — Create a repository for your images

This repository will store every cropped figure. Keep it dedicated to images so it stays tidy.

1. Go to `https://github.com/new`.
2. Fill in the form:
   - **Repository name:** something clear, such as `mcq-images`.
   - **Visibility:** choose **Public**. This is important — jsDelivr only serves files from public repositories, and a public token scope is simpler to manage.
   - **Initialize with a README:** tick this box. A repository must contain at least one file before it can accept uploads, and the README satisfies that.
3. Click **Create repository**.

Note the **owner/repo** combination — for example `myname/mcq-images`. You will need it in Step 4. The "owner" is your GitHub username (or an organization name).

> **Tip:** You can reuse one image repository across many quizzes. There is no need to create a new repository each time.

---

## Step 3 — Create a Personal Access Token

The tool needs permission to commit files to your repository. This is granted with a Personal Access Token (PAT) — a long secret string that acts like a scoped password.

### Option A — Classic token (simplest)

1. Open `https://github.com/settings/tokens/new`.
   (The Figure Updater also has a **create one** link next to the token field that opens this page pre-filled.)
2. Set the fields:
   - **Note:** something memorable, such as `MCQ Figure Updater`.
   - **Expiration:** choose a duration. A longer expiry means fewer re-setups; pick what your security comfort allows.
   - **Scopes:** tick **`public_repo`**. This grants write access to your public repositories and nothing else.
     - If your image repository is **private**, tick the broader **`repo`** scope instead.
3. Scroll down and click **Generate token**.
4. **Copy the token immediately.** It starts with `ghp_...`. GitHub shows it only once — if you lose it, you must generate a new one.

### Option B — Fine-grained token

Fine-grained tokens are more restrictive and also work:

1. Open `https://github.com/settings/personal-access-tokens/new`.
2. Under **Repository access**, choose **Only select repositories** and pick your image repository.
3. Under **Permissions → Repository permissions**, set **Contents** to **Read and write**.
4. Generate and copy the token.

> **Security note:** Treat the token like a password. The Figure Updater stores it only in your own browser (`localStorage`) and sends it directly to GitHub — it is never shared with anyone else. Still, do not paste it into public chats or commit it into a repository. If it ever leaks, revoke it at `https://github.com/settings/tokens`.

---

## Step 4 — Configure the Figure Updater

1. Open the MCQ JSON Tool and go to the **Figure Updater** tab.
2. Find the **Image Hosting** panel.
3. Make sure the **GitHub + jsDelivr** tab is selected (it is the default).
4. Fill in the four fields:

| Field | What to enter | Example |
|-------|---------------|---------|
| **Repository** | Your `owner/repo` from Step 2 | `myname/mcq-images` |
| **Branch** | The repository's default branch | `main` |
| **Folder path** | *(Optional)* a subfolder to keep images organised | `figures` |
| **Personal Access Token** | The `ghp_...` token from Step 3 | `ghp_xxxxxxxxxxxx` |

5. Click **Save & Verify**.

The tool checks that the repository exists and that the token can write to it. On success you will see a green confirmation message. Your configuration is saved in the browser, so you will not need to re-enter it next time.

> **About the branch:** Most new repositories use `main`. If yours is different (older repositories sometimes use `master`), enter that instead. You can see the branch name at the top-left of your repository page on GitHub.

> **About the folder path:** Leave it blank to commit images to the repository root, or enter something like `figures` or `exam-2026/figures` to group them. The folder is created automatically on the first upload.

---

## Step 5 — Use it

You are done. From now on:

1. Crop a figure from the PDF — it is held locally while you resize and preview.
2. Click **Apply Figures to This Question**.
3. The tool commits each new figure to GitHub and writes its jsDelivr URL into the JSON.

The progress label shows the upload status, for example *Uploading 2/3 to GitHub (Option B)…*.

---

## Verifying it worked

After applying a figure:

- Open your repository on GitHub — you should see the new `.webp` files (inside your folder, if you set one).
- Paste a generated CDN URL into a browser tab — the image should load directly.

A jsDelivr URL takes a few seconds to become available the very first time after a commit, then it is cached globally.

---

## Troubleshooting

**"GitHub rejected the token (401)"**
The token is wrong, expired, or was not copied fully. Generate a fresh token (Step 3) and paste it again. Make sure there are no leading/trailing spaces.

**"GitHub repo or branch not found (404)"**
Check three things: the repository name is spelled exactly as `owner/repo`; the branch name matches your repository's actual default branch; and — if the repository is private — the token has the full `repo` scope, not just `public_repo`.

**"GitHub upload failed (422)"**
A file with that exact name already exists. The tool uses timestamped filenames, so this is rare; simply re-crop to get a new name.

**"Token may lack write access"**
The token was accepted but does not have permission to commit. Regenerate it with the `public_repo` scope (or `repo` for a private repository).

**The image does not display in the quiz**
Confirm the repository is **Public**. jsDelivr cannot serve files from private repositories — if your repository must be private, use the Google Drive host instead.

**Images load slowly the first time**
This is normal. jsDelivr caches a file globally on first request; subsequent loads are fast everywhere.

---

## Why GitHub + jsDelivr over Google Drive

| | GitHub + jsDelivr | Google Drive |
|---|---|---|
| Rate limits on image views | None | Can throttle on heavy traffic |
| Delivery speed | Global CDN, very fast | Slower, varies by region |
| Setup effort | One-time, ~5 minutes | Requires OAuth connection |
| Cost | Free | Free |
| Best for | Published quizzes with real traffic | Quick tests, private files |

Google Drive remains available as an option in the same panel — switch to the **Google Drive** tab any time. But for any quiz that real students will use, GitHub + jsDelivr is the dependable choice.

---

## Quick reference

- **Sign up:** `https://github.com/signup`
- **New repository:** `https://github.com/new` (Public, with README)
- **New classic token:** `https://github.com/settings/tokens/new` (scope: `public_repo`)
- **Manage / revoke tokens:** `https://github.com/settings/tokens`
- **CDN URL format:** `https://cdn.jsdelivr.net/gh/OWNER/REPO@BRANCH/PATH/file.webp`
