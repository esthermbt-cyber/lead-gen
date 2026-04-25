Perform a full GitHub publish sequence for the current project. Follow every step below in order, stopping and reporting an error if any step fails.

---

## Step 1 — Generate or update README.md

Read the project files to understand what the app does. Then write (or overwrite) `README.md` in the repo root with the following sections:

- **Project name + one-line description** (infer from code/existing README)
- **Features** — bullet list of key features (infer from code)
- **Tech stack** — languages, APIs, libraries used
- **Setup & usage** — how to open/run the app, how to get any required API keys
- **Screenshots** — a placeholder section: `> 📸 _Add screenshots here_`
- **License** — `MIT` if no license file exists, otherwise match the existing one

Write the README in clean GitHub-flavored markdown. Do not include the word "Claude" or mention AI authorship.

---

## Step 2 — Create or update .github/workflows/deploy.yml

Write the following file exactly (create parent directories if needed):

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - uses: actions/deploy-pages@v4
        id: deployment
```

---

## Step 3 — Stage and commit all changes

Run:
```
git add .
git commit -m "docs: add README and GitHub Pages deployment"
```

If there is nothing to commit (working tree clean), skip the commit and continue.

---

## Step 4 — Push to GitHub

Run:
```
git push
```

If the remote is not set, report the error and stop.

---

## Step 5 — Enable GitHub Pages via gh CLI

Run the following to set the Pages source to GitHub Actions:
```
gh api --method POST repos/{owner}/{repo}/pages --field build_type=workflow
```

Resolve `{owner}` and `{repo}` from the git remote URL. If the Pages source is already set (API returns 409 or "already enabled"), treat it as success and continue.

Then immediately trigger the workflow so the first deployment starts:
```
gh workflow run deploy.yml
```

---

## Step 6 — Update repo description and topics via gh CLI

Infer a short one-sentence description from the project and a set of relevant topic tags (3–6 lowercase hyphenated tags, e.g. `lead-generation`, `apify`, `javascript`).

Run:
```
gh repo edit --description "<inferred description>"
gh repo edit --add-topic <topic1> --add-topic <topic2> ...
```

---

## Step 7 — Print the live URL

Derive the GitHub Pages URL from the owner and repo name:
```
https://{owner}.github.io/{repo}/
```

Print a final summary:
```
✅ Done! Your app will be live at:
   https://{owner}.github.io/{repo}/

The GitHub Actions workflow is running — it takes ~60 seconds.
Check progress: https://github.com/{owner}/{repo}/actions
```
