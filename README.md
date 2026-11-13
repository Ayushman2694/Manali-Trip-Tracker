# Manali Trip Split — GitHub Pages + Cloudflare Worker + MongoDB Atlas

This project keeps the website static on GitHub Pages. A small Cloudflare Worker provides the secure API connection to MongoDB Atlas. There is no always-on server to maintain and no database credential in the browser.

## Project layout

- `index.html` — the mobile-friendly Manali expense tracker (GitHub Pages)
- `worker/` — Cloudflare Worker API source (deploy separately)

## 1. Create MongoDB Atlas database

1. Sign up at https://www.mongodb.com/atlas and create a free/shared cluster.
2. In **Database Access**, create a database user with a strong, unique password. Give it **Read and write to any database** only if you need a quick setup; preferably create a custom role limited to `readWrite` on the `manali_trip` database.
3. In **Network Access**, add `0.0.0.0/0` so Cloudflare Workers can reach the cluster. Workers do not have a single fixed outbound IP on the standard setup. This permits network access from any IP, so compensate with a strong database password, a least-privilege database user, and the secret-only connection string. Never put the connection string in `index.html` or GitHub.
4. Open your cluster → **Connect** → **Drivers** → choose Node.js and copy the `mongodb+srv://...` connection string. Replace the username/password placeholders. URL-encode special characters in the database password. The Worker uses database `manali_trip` and collection `shared_trip`; MongoDB creates them on the first save.

Atlas docs: https://www.mongodb.com/docs/atlas/connect-to-database-deployment/

## 2. Deploy the Cloudflare Worker

1. Create/sign in at https://dash.cloudflare.com/.
2. Install Node.js (LTS) on your computer if it is not already installed.
3. Open a terminal in this project's `worker` folder:

   ```bash
   cd worker
   npm install
   npx wrangler login
   ```

4. Edit `worker/wrangler.toml`. Replace `https://ayushman2694.github.io` with your exact GitHub Pages **origin** (no repository path and no trailing slash). Example: `https://aviral123.github.io`.
5. Add the following encrypted secrets. Run each command and paste the requested value when prompted:

   ```bash
   npx wrangler secret put MONGODB_URI
   npx wrangler secret put TRIP_PASSWORD
   npx wrangler secret put JWT_SECRET
   ```

   - `MONGODB_URI`: the Atlas connection string.
   - `TRIP_PASSWORD`: the shared password you will give to the trip group.
   - `JWT_SECRET`: a long, random secret (at least 32 random characters). Do not reuse the group password.

6. Deploy:

   ```bash
   npx wrangler deploy
   ```

7. Cloudflare prints a URL similar to `https://manali-trip-api.YOUR-SUBDOMAIN.workers.dev`.

Cloudflare's current official MongoDB Worker guide: https://developers.cloudflare.com/workers/tutorials/connect-to-mongodb-atlas/
Cloudflare secret documentation: https://developers.cloudflare.com/workers/configuration/secrets/

## 3. Connect the GitHub Pages website to the Worker

1. Open the root `index.html` file.
2. Find this line near the start of the JavaScript:

   ```js
   const API_BASE='https://YOUR-WORKER-NAME.YOUR-SUBDOMAIN.workers.dev/api';
   ```

3. Replace it with your actual Worker URL plus `/api`, for example:

   ```js
   const API_BASE='https://manali-trip-api.YOUR-SUBDOMAIN.workers.dev/api';
   ```

4. Save the file.

## 4. Publish the website on GitHub Pages

1. Create a **public** GitHub repository, e.g. `manali-trip-split`.
2. Upload `index.html` to the repository root. You may also upload this `worker` folder and README as source documentation; do not upload any real credentials or `.env` files.
3. In repository **Settings → Pages**, choose **Deploy from a branch**, branch `main`, folder `/ (root)`, then Save.
4. Wait for GitHub Pages to publish. The site URL will be `https://ayushman2694.github.io/manali-trip-split/`.

The GitHub Pages origin (the part before `/manali-trip-split/`) must exactly match `ALLOWED_ORIGIN` in `worker/wrangler.toml`. If you change that value later, redeploy the Worker with `npx wrangler deploy`.

## 5. Use it with the group

1. Open the GitHub Pages link on each phone.
2. Enter the shared trip password when prompted.
3. Add the travellers and record expenses. The app checks for group changes every 7 seconds.
4. Each member can use **Add to Home Screen** in their mobile browser.

## Security and limitations

- The MongoDB connection string and JWT signing secret are Cloudflare Worker secrets, not browser code.
- The shared trip password is checked by the Worker. A signed session token is stored in session storage and expires after 12 hours.
- Anyone who has both the website URL and group password can view and modify the trip. Share the password only with the group.
- The app stores one shared trip document. Avoid editing from multiple phones at the exact same moment; the most recent full-document save can overwrite a simultaneous save. Export a JSON backup before a major reset/import.
- The page uses external scenic photos, so those images need an internet connection.
- This is a lightweight group expense tracker, not a high-security financial ledger. Do not store card details, bank credentials, or other sensitive information in it.
