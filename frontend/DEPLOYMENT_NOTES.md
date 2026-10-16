# The Complete A-to-Z Frontend Deployment Guide & Notes (Marvelam)

> **Zero-Code-Touch Guarantee:** This is an independent documentation note. Your project code in `Marvelam-Frontend` remains 100% untouched.

---

## 📋 The Complete A-to-Z Deployment Roadmap

Deploying a modern React + Vite application involves 7 phases:
1. **Phase 1:** Pre-Deployment Verification (Testing build locally)
2. **Phase 2:** Git & GitHub Setup (Local to Cloud)
3. **Phase 3:** Choosing and Preparing the Host (Vercel / Render)
4. **Phase 4:** Configuring Environment Variables & Build Settings
5. **Phase 5:** Setting up SPA Routing Rewrites (Crucial for React Router)
6. **Phase 6:** Live Deployment & Sanity Verification
7. **Phase 7:** Ongoing Updates & CI/CD (Future pushes)

---

## Phase 1: Pre-Deployment Local Verification (Test First!)

Before pushing to the internet, always confirm that your code builds without errors locally.

1. Open your terminal in your project folder (`Marvelam-Frontend`).
2. Run the build command:
   ```bash
   npm run build
   ```
   * What this does: Vite bundles your JSX, Tailwind CSS, and assets into static HTML/CSS/JS files inside the `dist/` folder.
   * If it succeeds with a green output, your code is ready.
   * If it fails (e.g. missing import or syntax error), fix the issue locally before pushing.
3. *(Optional)* Preview the production build locally:
   ```bash
   npm run preview
   ```
   * This simulates how the deployed site will look and feel. Press `Ctrl + C` to stop previewing.

---

## Phase 2: Git & GitHub Setup (Pushing Code to the Cloud)

Hosting platforms like Vercel and Render deploy directly by pulling from GitHub.

### Step 1: Create an Empty Repository on GitHub
1. Go to [github.com/new](https://github.com/new).
2. Set **Repository name**: `marvelam-frontend`.
3. Set visibility: **Public** or **Private**.
4. **Do NOT check** "Add a README", ".gitignore", or "License" (keep it totally empty).
5. Click **Create repository** and copy the URL (e.g., `https://github.com/acthlon/marvelam-frontend.git`).

### Step 2: Push Your Local Code
Run these commands in your `Marvelam-Frontend` terminal:
```bash
# 1. Initialize local Git repo
git init

# 2. Stage all files
git add .

# 3. Create your first commit
git commit -m "feat: initial commit of Marvelam frontend"

# 4. Rename default branch to main
git branch -M main

# 5. Link local folder to your GitHub repo
git remote add origin https://github.com/acthlon/marvelam-frontend.git

# 6. Push to GitHub with upstream tracking (-u)
git push -u origin main
```

---

## Phase 3: Connect to Hosting Provider

You can deploy on **Vercel** (recommended for speed) or **Render** (where your backend lives).

---

### Option A: Vercel (Fastest & Simplest)

1. Go to [vercel.com](https://vercel.com/) and click **Sign Up** or **Log In** with **GitHub**.
2. From the Vercel dashboard, click **"Add New..."** → **"Project"**.
3. Under "Import Git Repository", locate `marvelam-frontend` and click **Import**.
4. **Configure Project:**
   * **Framework Preset:** Vite (automatically detected).
   * **Root Directory:** `./`
   * **Build Command:** `npm run build`
   * **Output Directory:** `dist`
5. **Environment Variables:**
   * Expand the **Environment Variables** accordion.
   * **Key:** `VITE_API_BASE_URL`
   * **Value:** `https://marvelam.onrender.com/api/v1/`
   * Click **Add**.
6. Click **Deploy**.
7. Vercel will build the project and output a live URL (e.g., `https://marvelam-frontend.vercel.app`).

---

### Option B: Render (Same Host as Your Backend)

1. Log in to [render.com](https://render.com/).
2. Click **"New +"** (blue button top right) → Select **"Static Site"**.
3. Connect your GitHub account and select `marvelam-frontend`.
4. Fill in the configuration:
   * **Name:** `marvelam-frontend`
   * **Branch:** `main`
   * **Build Command:** `npm run build`
   * **Publish Directory:** `dist`
5. Under **Environment Variables**, click **Add Environment Variable**:
   * **Key:** `VITE_API_BASE_URL`
   * **Value:** `https://marvelam.onrender.com/api/v1/`
6. **Configure SPA Rewrite (Mandatory on Render):**
   * Scroll down to **"Redirects/Rewrites"**.
   * Click **Add Rule**:
     * **Type:** `Rewrite`
     * **Source:** `/*`
     * **Destination:** `/index.html`
7. Click **Create Static Site**.

---

## Phase 4: Understanding the SPA Routing Rewrite (The "404 on Refresh" Fix)

### The Problem:
In React Router, navigation between pages (like `/products`, `/cart`, `/checkout`, `/account`) is virtual (client-side). When a real user directly refreshes `https://yoursite.com/cart`, the web server looks on disk for a physical file called `cart/index.html`. It doesn't find it, so it gives a `404 Not Found`.

### The Solution:
Tell the server: *"Whenever any path is requested (`/*`), send back `/index.html` with status code 200, and let React Router handle the page display."*

* **On Render:** Configured via the **Redirects/Rewrites** table (`/*` -> `/index.html`, Rewrite).
* **On Vercel:** Usually works automatically, or configured via a small `vercel.json` if needed:
  ```json
  {
    "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
  }
  ```
* **On Netlify:** Configured in `public/_redirects`:
  ```text
  /*    /index.html   200
  ```

---

## Phase 5: Post-Deployment Verification (Sanity Check)

Once the deployment finishes and you get your live HTTPS link:
1. **Homepage Check:** Visit the root URL `https://your-domain.vercel.app/` — verify that banners, products, and images render.
2. **API Connection Check:** Open browser DevTools (`F12` or Right-Click → Inspect → **Network** tab). Verify that API calls to `https://marvelam.onrender.com/api/v1/...` return `200 OK` (not CORS errors or 500s).
3. **Route Refresh Check:** Click into `/cart` or `/products/some-id`. Press **Refresh (F5)** in your browser. Verify you do not get a 404 error.
4. **Responsive Check:** Test on mobile browser or toggle Device Mode in DevTools.

---

## Phase 6: How Future Updates Work (CI/CD)

You never have to manually re-upload files or re-run setup screens.

Every time you make improvements in the future:
```bash
# 1. Stage your changes
git add .

# 2. Commit with a note
git commit -m "added new feature or bugfix"

# 3. Push to GitHub
git push
```

* GitHub instantly alerts Vercel/Render.
* The host downloads your new commit, runs `npm run build`, and updates your live website in seconds with **zero downtime**.

---

## Phase 7: (Optional) Adding a Custom Domain

When you are ready to use a custom domain (e.g., `marvelam.com` or `www.marvelam.com`):
1. In Vercel or Render, go to **Settings** → **Domains**.
2. Type your domain (e.g., `marvelam.com`).
3. The platform will give you DNS records to add:
   * **A Record:** pointing to the host's IP address.
   * **CNAME Record:** pointing `www` to `cname.vercel-dns.com` (or render equivalent).
4. Go to your domain registrar (Namecheap, GoDaddy, Hostinger, etc.), add those DNS records, and within 30 minutes your custom domain and free automatic SSL/HTTPS certificate will be active.

---

## 8. Strategy: Deploying First vs. Cleaning Up First

### Should you deploy now even if your code isn't fully cleaned up?

**YES, deploy first!** In professional software engineering, this is known as **"Deploy Early and Iterate"**.

#### The One Rule:
* **The code must compile (`npm run build` passes).**
* If `npm run build` succeeds locally, you can deploy immediately. You do **not** need to wait until your code is 100% clean, refactored, or perfect.
* Minor visual bugs, uncleaned files, or pending features **will not crash the build**.

#### Why Deploying First is the Smarter Approach:
1. **Tests the Full Infrastructure Early:** You confirm immediately that Git, GitHub, Vercel/Render, and your Backend API (`https://marvelam.onrender.com/api/v1/`) communicate properly in production (CORS, SSL, cookies, headers).
2. **Speed & Momentum:** You get a live link right away to see how the site feels on actual phones and devices.
3. **Iterative Cleaning:** Once your deployment pipeline is hooked up, any time you clean a component, refactor a file, or fix a bug, you simply run:
   ```bash
   git add .
   git commit -m "refactor: cleaned up product page"
   git push
   ```
   and your live site updates automatically in under a minute.

#### When NOT to Deploy First:
* Only when `npm run build` fails with an error (e.g. fatal syntax error, broken import). A failed build will cause the hosting platform to abort deployment. Fix the blocking build error, then deploy right away.

---

## 9. Customizing Your URL on Vercel

### A. Changing to a shorter free Vercel subdomain (e.g. `marvelam.vercel.app`):
1. Go to your project on **Vercel**.
2. Click **Settings** (tab at the top) → **Domains** (menu on the left).
3. Look at your existing domain or use the **"Add"** box.
4. Type: `marvelam.vercel.app` and click **Add**.
5. **Availability check:**
   * If available: Vercel instantly assigns `https://marvelam.vercel.app` to your live app!
   * If already taken by someone else worldwide: Try close alternatives like:
     * `marvelam-store.vercel.app`
     * `marvelamatelier.vercel.app`
     * `marvelam-app.vercel.app`
6. You can delete the old auto-generated domain or keep both.

### B. Changing to your own custom domain (e.g. `marvelam.com` or `marvelam.ng`):
* In the same **Settings** → **Domains** tab, enter your registered domain (e.g. `marvelam.com`).
* Add the DNS records Vercel provides to your domain registrar (Namecheap, GoDaddy, Whogohost, etc.).

---

## 10. Troubleshooting: Why Data Isn't Loading (CORS Error)

### The Issue:
When running on `localhost:5173`, data loads fine. But once deployed to Vercel (e.g., `https://mac-frontend.vercel.app`), the browser blocks API requests and data is empty.

### Why this happens:
Your Django backend uses `python-decouple` with `cast=Csv()`:
* In `mac/settings/base.py`:
  `CORS_ALLOWED_ORIGINS = config("CORS_ALLOWED_ORIGINS", cast=Csv())`
  `CSRF_TRUSTED_ORIGINS = config("CSRF_TRUSTED_ORIGINS", cast=Csv())`
* It reads the allowed origins **directly from your Render Environment Variables**.
* Because `https://mac-frontend.vercel.app` is not in that list, browser security blocks the API requests.

### How to Fix in Render Dashboard (No Code Edit Needed!):
1. Open your **[Render.com Dashboard](https://dashboard.render.com/)**.
2. Click on your backend web service (e.g. **`marvelam`**).
3. In the left menu, click **"Environment"**.
4. Find **`CORS_ALLOWED_ORIGINS`** and append your Vercel domains with commas (no spaces, no brackets):
   ```text
   http://localhost:5173,https://mac-frontend.vercel.app,https://marvelam.vercel.app
   ```
5. Find **`CSRF_TRUSTED_ORIGINS`** and also append your Vercel domains:
   ```text
   http://localhost:5173,https://mac-frontend.vercel.app,https://marvelam.vercel.app
   ```
6. Click **"Save Changes"**.
7. Render will automatically redeploy/restart the backend in ~60 seconds. Once done, refresh your Vercel site and data will load immediately!




