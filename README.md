# Land Ledger

Track a land purchase: payments to the seller, additional costs (legal checks, registration, broker, etc.), and loan EMIs.
No backend. Data lives in a JSON file, optionally synced to your Google Drive. Exports to Excel.

## 1. Host it free on GitHub Pages

1. Create a **public** GitHub repository (for example `land-ledger`). Free GitHub plans only serve Pages from public repos.
2. Upload everything in this folder, **including the hidden `.github/workflows/deploy.yml`**, to the repository root.
3. Go to **Settings → Pages**. Under *Build and deployment → Source*, choose **GitHub Actions**.
4. Every push to `main` now deploys the app to `https://<your-username>.github.io/land-ledger/`.
   You can also run it by hand from the **Actions** tab → *Deploy to GitHub Pages* → **Run workflow**.

Only the app files (`index.html` and the icons) are published. The README and anything else in the repo are not.

## 2. Turn on Google Drive sync (optional, free)

1. Open https://console.cloud.google.com and create a project (any name).
2. **APIs & Services → Library**: search for **Google Drive API** and click **Enable**.
3. **APIs & Services → OAuth consent screen** (Google Auth Platform):
   - User type: **External**. App name: Land Ledger. Add your email as support and developer contact.
   - Scopes: add `.../auth/drive.file`.
   - **Test users / Audience**: add your own Gmail address (and anyone else who will use it).
   - Leave the app in **Testing** mode. That is fine for personal use.
4. **APIs & Services → Credentials → Create credentials → OAuth client ID**:
   - Application type: **Web application**.
   - Authorized JavaScript origins: `https://<your-username>.github.io` (no trailing slash, no repo path).
   - Create, then copy the **Client ID** (ends in `.apps.googleusercontent.com`).
5. In the GitHub repo, go to **Settings → Secrets and variables → Actions → Variables** tab → **New repository variable**.
   Name: `GOOGLE_CLIENT_ID`. Value: the client ID. Save, then re-run the deploy workflow (Actions tab → Run workflow).
   The ID never appears in your repo's code; the workflow inserts it into the published page at deploy time.
6. Open the app, tap **Connect Google Drive**, and sign in. Google will warn that the app is unverified; choose *Continue* (it's your own app).

### How sync behaves
- The app creates `land-ledger.json` in your Drive and saves every change to it about 1.5 seconds later.
- It uses the `drive.file` permission, so it can only see files it created, not the rest of your Drive.
- On another device, open the same URL and tap **Sync with Drive** to load the latest data.
- Google sign-in lasts about an hour. After that, tap **Sync with Drive** again.
- If the Drive file was changed on another device while this one also had unsaved edits, the app asks which version to keep instead of overwriting.
- Already have an exported JSON file? Use **Import JSON** first, then **Connect Google Drive**. The imported data is uploaded.

## Testing locally
Double-clicking `index.html` works for everything except Drive sync (Google requires an `http(s)` origin).
For Drive testing locally, run `npx serve .` and add `http://localhost:3000` as another authorized origin.

## Security: what someone with your link can and can't do

**There are no secrets in `index.html`.** The Google client ID is a public identifier, not a password. Google only accepts it
from the origin you listed (`https://<your-username>.github.io`), so copying it to another site doesn't work.

**Your data is never in the page or the repo.** Someone who opens your link gets an empty app. Your data lives in:
- your Google Drive (`land-ledger.json`), protected by your Google account, and
- the browser storage of the devices you used, encrypted if you turn on App lock.

**They can't reach your Drive.** Signing in only ever gives access to the Drive of whoever signs in, and while the Google
app is in Testing mode, only accounts on your *Test users* list can sign in at all. Don't publish the Google app, and
keep that list to yourself (and family if needed).

**App lock (Property details → App lock).** Set a passphrase to lock the app on each device. The local copy is then
encrypted with AES-GCM (256-bit key derived with PBKDF2-SHA256, 310,000 rounds) and the app locks after 10 minutes of no use.
The Google sign-in is dropped when it locks. There is no passphrase reset, so keep Drive sync or JSON exports as your backup.

**Other things worth doing**
- Turn on 2-Step Verification for your Google account. It is the real key to your data.
- Exported JSON and Excel files are not encrypted. Keep them in Drive or somewhere private, not in the GitHub repo.
- `index.html` sets a Content-Security-Policy that only lets the page load code from the CDNs it needs and only send
  data to Google. If you add another library or service, add its domain to that line.
- To revoke access at any time: https://myaccount.google.com/permissions → Land Ledger → Remove access.
