# iMakeCarKeys — Deployment Guide
## GitHub → Cloudflare Pages → Claude handles all commits

This document covers everything in one place:
1. One-time GitHub setup
2. One-time Cloudflare Pages setup
3. How to push changes live using Claude
4. Common update scenarios with exact wording to use

---

## How the pipeline works

```
You tell Claude what to change
        ↓
Claude edits the file(s) in this conversation
        ↓
Claude pushes a commit to GitHub via the API
        ↓
Cloudflare Pages detects the push and rebuilds
        ↓
Live site updated — usually within 60 seconds
```

No FTP. No dashboard. No terminal. You describe the change to Claude and say "push it."

---

## Part 1 — One-time GitHub setup (you do this once)

### Step 1 — Create a GitHub account (if you don't have one)
Go to **github.com** → Sign up. Free tier is fine.

### Step 2 — Create a new repository
1. Click the **+** in the top-right → **New repository**
2. Name it: `imakecarkeys` (or anything — just remember it)
3. Set to **Public** (required for free Cloudflare Pages) or **Private** (requires Cloudflare's free plan still works)
4. Leave "Add README" **unchecked** — we're uploading existing files
5. Click **Create repository**

### Step 3 — Upload the site files
On the empty repo page, click **uploading an existing file**.

Drag and drop **all files from this folder** — every `.html`, `.jpg`, `_headers`, `_redirects`, `robots.txt`, `sitemap.xml`, and this `DEPLOY.md`. Do **not** create subfolders — everything goes in the root.

Scroll down → **Commit changes** → green button.

### Step 4 — Create a Personal Access Token (for Claude)
This token lets Claude push changes to GitHub on your behalf.

1. GitHub → your avatar (top-right) → **Settings**
2. Left sidebar, scroll to bottom → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)**
4. **Generate new token (classic)**
5. Note: `Claude iMakeCarKeys`
6. Expiration: **No expiration** (or 1 year — your choice)
7. Scopes: check **`repo`** (top-level — this includes all sub-options)
8. Click **Generate token**
9. **Copy the token immediately** — you only see it once

> ⚠️ Treat this like a password. Don't paste it in email or share it publicly.
> Store it somewhere safe — a password manager like 1Password or Bitwarden works well.

**What to give Claude:** When you want Claude to push changes, paste this into the conversation:
```
My GitHub: YOUR-USERNAME/imakecarkeys
Token: ghp_xxxxxxxxxxxxxxxxxxxx
```
Claude will use it to commit the change and then you can revoke/rotate the token anytime from GitHub settings.

---

## Part 2 — One-time Cloudflare Pages setup (you do this once)

### Step 1 — Create a Cloudflare account
Go to **cloudflare.com** → Sign up. The free plan covers everything this site needs.

### Step 2 — Connect Pages to GitHub
1. Cloudflare dashboard → left sidebar → **Workers & Pages**
2. Click **Pages** tab → **Create application**
3. Click **Connect to Git** → **Connect GitHub**
4. Authorize Cloudflare to access your GitHub account
5. Select the `imakecarkeys` repository → **Begin setup**

### Step 3 — Configure the build
On the "Set up builds and deployments" screen:

| Setting | Value |
|---|---|
| Production branch | `main` |
| Framework preset | `None` |
| Build command | *(leave completely blank)* |
| Build output directory | `/` |

Click **Save and Deploy**.

Cloudflare copies the files and gives you a live URL like `https://imakecarkeys-abc123.pages.dev`. That's your site. Every push to `main` from now on redeploys it automatically.

### Step 4 — Connect your real domain (optional but recommended)
1. In your Pages project → **Custom domains** tab → **Set up a custom domain**
2. Enter `imakecarkeys.com`
3. If your domain's DNS is already managed by Cloudflare → it wires up automatically
4. If DNS is at GoDaddy, Namecheap, etc. → Cloudflare shows you a `CNAME` record to add there
5. HTTPS/SSL is automatic and free — no certificate to buy

---

## Part 3 — How to have Claude push changes

Once the token is set up, every update follows this pattern:

### The exact words to say to Claude

**For content changes:**
> "Update the phone number on index.html from 561-203-9232 to 561-999-0000. My GitHub is andyhanks/imakecarkeys and my token is ghp_xxx. Push it."

**For new reviews:**
> "Add a new 5-star review from Maria G. in West Palm Beach: 'Made a key for my BMW right in my driveway. Done in 20 minutes.' Push it to GitHub — andyhanks/imakecarkeys, token ghp_xxx."

**For FAQ additions:**
> "Add a new FAQ: Q: Do you work on motorcycles? A: Not at this time — I specialize in cars, trucks, and SUVs. Push it. Repo is andyhanks/imakecarkeys, token ghp_xxx."

**For image swaps:**
> "Replace mobile-car-locksmith-west-palm-beach.jpg with this new photo [attach image]. Keep the same filename. Push to GitHub."

**For anything:**
> "Here's what I want changed: [describe it]. Repo: andyhanks/imakecarkeys, token: ghp_xxx."

Claude will make the edit, confirm what changed, push the commit, and tell you the live URL to verify.

---

## Part 4 — What Claude does under the hood

When you give Claude the repo name and token, here is exactly what happens — no magic, all auditable:

1. Claude reads the current file from GitHub (via the GitHub API)
2. Makes the requested change
3. Encodes the new file content
4. Calls `PUT /repos/{owner}/{repo}/contents/{path}` with the token
5. GitHub records the commit with your account as the author
6. Cloudflare's webhook fires → Pages rebuilds → live in ~60 seconds

You can see every commit in your GitHub repo's **commits** history, including what changed and when. Nothing is hidden.

---

## Part 5 — Preview before going live

Any change pushed to a branch *other than* `main` gets a private preview URL — it doesn't touch the live site.

Tell Claude:
> "Make that change but push it to a branch called `draft-new-faq` so I can preview it first. Repo: andyhanks/imakecarkeys, token: ghp_xxx."

Cloudflare will comment a preview URL (looks like `https://draft-new-faq.imakecarkeys.pages.dev`). Once you're happy:
> "Looks good. Merge it to main and push it live."

---

## Part 6 — File inventory (what each file does)

| File | Purpose | Safe to edit? |
|---|---|---|
| `index.html` | The entire main page — HTML, CSS, JS all inline | Yes — primary file |
| `privacy.html` | Privacy policy — standalone page with widget | Yes |
| `accessibility.html` | Accessibility statement — standalone page with widget | Yes |
| `404.html` | Not-found page — Cloudflare serves this automatically | Yes |
| `_headers` | Cloudflare security + cache rules | Only if you know what you're doing |
| `_redirects` | URL redirect rules for Cloudflare | Yes — add old URL redirects here |
| `robots.txt` | Search engine crawl instructions | Yes |
| `sitemap.xml` | Page list for search engines | Update when adding new pages |
| `*.jpg` | Site photos — referenced by filename in index.html | Replace by filename |
| `imakecarkeys-social-preview.jpg` | The 1200×630 image that appears when you share the link on social | Replace to update share card |

---

## Part 7 — Hooking up analytics or Meta Pixel

Open `index.html` and find this comment in the `<script>` block:

```javascript
function applyConsent(state){
  if (state.analytics){ document.documentElement.dataset.analytics = "on"; /* load analytics here */ }
  if (state.marketing){ document.documentElement.dataset.marketing = "on"; /* load Meta Pixel here */ }
}
```

Paste your analytics loader (Google Analytics `gtag`, Plausible snippet, Meta Pixel base code) inside the matching `if` block. They will **only fire after the visitor consents** — which is required for GDPR/CCPA compliance and keeps the site clean for visitors who decline.

Example for Google Analytics:
```javascript
if (state.analytics){
  document.documentElement.dataset.analytics = "on";
  var s = document.createElement("script");
  s.src = "https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX";
  s.async = true;
  document.head.appendChild(s);
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag("js", new Date());
  gtag("config", "G-XXXXXXXXXX");
}
```

Tell Claude the analytics ID and say "add it to the consent-gated analytics block and push it."

---

## Part 8 — Keeping the token safe

The Personal Access Token is a credential. Rules:

- **Never paste it in a public place** — not in a GitHub issue, not in a public Slack, not in email
- **Rotate it periodically** — GitHub → Settings → Developer settings → delete old token, create new one, give new one to Claude
- **Revoke it immediately** if you think it was exposed — the site stays live; only future pushes are blocked until you create a new one
- You can create multiple tokens with different names if multiple people manage the site — that way you can revoke one without affecting the others

---

## Quick reference card

```
LIVE URL:      https://imakecarkeys.com
PAGES URL:     https://imakecarkeys.pages.dev  (fallback)
GITHUB REPO:   github.com/YOUR-USERNAME/imakecarkeys
BRANCH:        main (deploys live)
BUILD:         none — files deploy as-is

TO PUSH A CHANGE:
  Tell Claude what to change.
  Provide: repo name + GitHub token.
  Claude edits → commits → Cloudflare deploys → live in ~60 sec.

TO PREVIEW FIRST:
  Ask Claude to push to a named branch instead of main.
  Approve the Cloudflare preview URL.
  Tell Claude to merge to main when ready.
```

---

*Built by ThatsKrispy · andy@thatskrispy.com*
