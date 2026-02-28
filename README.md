# Song Dropper site (songdropper.com)

This repo is the **public** site for **songdropper.com**. It exists only to serve one file:

**`.well-known/assetlinks.json`** — required for [Android App Links](https://developer.android.com/training/app-links). When someone taps a link like `https://songdropper.com/s/...` (e.g. in an SMS), Android checks this file to verify the Song Dropper app is allowed to open that domain, then opens the app directly (no browser).

No server logic, no redirects. Just static hosting so that **https://songdropper.com/.well-known/assetlinks.json** returns the JSON.

---

## 1. SHA256 fingerprints in assetlinks.json

`.well-known/assetlinks.json` must list the **SHA256** of the certificate(s) used to sign the Song Dropper Android app.

**Debug (for testing):**
```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```
Copy the **SHA256** line (format `AA:BB:CC:...`).

**Release (Play Store):**  
Use the same `keytool` command with your release keystore (or the certificate Google uses for Play App Signing if you use that). Add each SHA256 as a string in the `sha256_cert_fingerprints` array in `assetlinks.json`.

---

## 2. GitHub Pages hosting

1. In this repo on GitHub: **Settings** → **Pages** (left sidebar).
2. Under **Build and deployment**:
   - **Source:** Deploy from a branch.
   - **Branch:** `main` (or your default branch), folder **/ (root)**.
   - Click **Save**.
3. Under **Custom domain**:
   - Enter **`songdropper.com`** and click **Save**.
   - GitHub will show a DNS reminder; leave the page open — you’ll add the records in GoDaddy next.

GitHub will serve the repo root at that domain, so **https://songdropper.com/.well-known/assetlinks.json** will serve your file.

---

## 3. DNS at GoDaddy (point songdropper.com to GitHub)

1. Go to [GoDaddy](https://www.godaddy.com) → **My Products** → **songdropper.com** → **DNS** (or **Manage DNS**).
2. Add **A** records for the apex domain so `songdropper.com` (no www) points to GitHub Pages:

   | Type | Name | Value         | TTL  |
   |------|------|----------------|------|
   | A    | @    | 185.199.108.153 | 600  |
   | A    | @    | 185.199.109.153 | 600  |
   | A    | @    | 185.199.110.153 | 600  |
   | A    | @    | 185.199.111.153 | 600  |

   If there is already an **A** or **CNAME** for **@**, remove or replace it so only these four A records apply to the apex.

3. (Optional) For **www.songdropper.com**, add a **CNAME**:
   - **Name:** `www`
   - **Value:** `spencerdcarlson.github.io`  
   (Use your GitHub username if different.)

4. Save. Check with:
   ```bash
   dig songdropper.com +short A
   ```
   You should see the four GitHub IPs. DNS can take a few minutes up to 48 hours.

---

## 4. Domain verification (TXT record) — recommended

GitHub recommends **verifying** the domain so only your account can use it with Pages (avoids takeover if the site is ever disabled). This uses a **DNS TXT record**, not a file in the repo.

1. In GitHub, go to the **repo** **song-dropper-site** → **Settings** → **Pages**. Under **Custom domain**, enter `songdropper.com` and click **Save** if you haven’t already.
2. If GitHub shows a message like **“Verify”** or **“Add a DNS TXT record”**, it will give you:
   - **Name/host:** e.g. `_github-pages-challenge-spencerdcarlson.songdropper.com` (your username + domain)
   - **Value:** a short string GitHub generates (e.g. `abc123...`)
3. In **GoDaddy** → **DNS** for songdropper.com, add a **TXT** record:
   - **Type:** TXT  
   - **Name:** `_github-pages-challenge-spencerdcarlson` (only the subdomain part; GoDaddy may add `.songdropper.com` automatically)  
   - **Value:** the exact value GitHub showed  
   - **TTL:** 600 (or default)
4. Save. After DNS propagates (minutes to a few hours), check:
   ```bash
   dig _github-pages-challenge-spencerdcarlson.songdropper.com +nostats +nocomments +nocmd TXT
   ```
   You should see the value you set.
5. Back in GitHub **Settings → Pages**, next to the domain click **Verify** (or **Continue verifying**). Once it succeeds, the domain will show as verified.

You do **not** add any `.txt` file to this repo — verification is done only via DNS.

---

## 5. Verify the site and assetlinks

Open in a browser:

**https://songdropper.com/.well-known/assetlinks.json**

You should see your JSON with the correct `package_name` and `sha256_cert_fingerprints`. It must be **HTTPS**. If the A records are correct but this URL doesn’t load yet, wait for DNS and ensure **Enforce HTTPS** is enabled under Pages settings (can take a while after the domain is set).

---

## 6. How this repo is used (submodule)

The **Song Dropper Android** app repo (private) includes this repo as a **git submodule** at `docs/app-links`. To change `assetlinks.json` or this README:

```bash
cd docs/app-links   # from the Android repo root
# edit .well-known/assetlinks.json or README.md
git add .
git commit -m "Update assetlinks"
git push origin main
cd ../..
git add docs/app-links
git commit -m "Update app-links submodule"
git push   # push the Android repo so the new submodule ref is saved
```
