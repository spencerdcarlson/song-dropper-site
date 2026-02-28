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

4. Save. DNS can take a few minutes up to 48 hours. Back in GitHub **Settings → Pages → Custom domain**, the domain should eventually show as verified.

---

## 4. Verify

Open in a browser:

**https://songdropper.com/.well-known/assetlinks.json**

You should see your JSON with the correct `package_name` and `sha256_cert_fingerprints`. It must be **HTTPS**.

---

## 5. How this repo is used (submodule)

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
