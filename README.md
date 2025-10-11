# agarwalkrish.com

A simple static website hosted on SiteGround and auto-deployed from GitHub via GitHub Actions.

## Project structure
/css
/images
/fonts
index.html
hometown.html
activity.html

## Local preview
Open `index.html` in your browser.  
(Keep files saved as UTF-8. Add `<meta charset="utf-8">` in `<head>` and, optionally, `AddDefaultCharset UTF-8` in `.htaccess`.)

## CI/CD (GitHub → SiteGround)

We deploy by syncing the repository to `public_html` over SSH using `rsync`.

### Repository secrets (Settings → Secrets and variables → Actions)
- `SG_HOST` — your SiteGround SSH hostname (e.g., `gsydm1010.siteground.biz`)
- `SG_USER` — your SiteGround SSH username (e.g., `uXXXX-…`)
- `SG_PORT` — your SSH port (e.g., `18765`)
- `SG_SSH_KEY` — **private** SSH key that matches the public key in SiteGround
- `SG_PATH` — deployment path, e.g. `/home/<sguser>/public_html`

### Workflow
Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to SiteGround (static)

on:
  push:
    branches: [ main ]
  workflow_dispatch:

concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Prepare SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SG_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -p "${{ secrets.SG_PORT }}" "${{ secrets.SG_HOST }}" >> ~/.ssh/known_hosts

      - name: Deploy via rsync
        run: |
          rsync -avz --delete \
            --exclude ".git*" \
            --exclude ".github/" \
            -e "ssh -i ~/.ssh/id_rsa -p ${{ secrets.SG_PORT }}" \
            ./ ${{ secrets.SG_USER }}@${{ secrets.SG_HOST }}:${{ secrets.SG_PATH }}
