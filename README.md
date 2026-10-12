# Bonsai Journal V1 — setup guide

A static PWA hosted by GitHub Pages, with Firebase Google Authentication, Firestore live synchronization, and Firebase Storage photos. Family members have separate Google logins and share a single collection. Admins can create email-bound invitation codes.

## Included files
- `index.html`, `style.css`, `app.js`: application
- `firebase-config.js`: paste your own Firebase **web** configuration
- `firestore.rules`, `storage.rules`: security rules to paste into Firebase
- `manifest.webmanifest`, `sw.js`, `icon.svg`: installable PWA assets

## Step 1 — Firebase project
1. Open https://console.firebase.google.com/ and create a project, e.g. `bonsai-family-journal`.
2. Under **Project settings > General > Your apps**, register a **Web** app. Copy the `firebaseConfig` fields into `firebase-config.js` (keep the `window.BONSAI_FIREBASE_CONFIG =` wrapper). Do **not** use service account JSON or admin keys.
3. Go to **Build > Authentication > Sign-in method**, enable **Google**, and set the support email.
4. Under **Authentication > Settings > Authorized domains**, add `YOURUSERNAME.github.io` (without https:// or a path). For local testing, optionally authorize `localhost`.
5. Go to **Build > Firestore Database**, create a database in your preferred region. Choose production mode. Under **Rules**, replace with contents of `firestore.rules`, then **Publish**.
6. To upload photos, upgrade Firebase to **Blaze** and set billing alerts. Under **Build > Storage**, create a bucket. Under **Rules**, paste `storage.rules` and **Publish**. Verify `storageBucket` in your web config matches the bucket shown in the console. Firebase Storage now requires Blaze even for small projects. Billing alerts do not enforce a spending cap.

## Step 2 — GitHub Pages
1. Create a free GitHub account if needed: https://github.com/.
2. Create a **public** repository named `bonsai-journal` (do not add any private family photos or data to the repository).
3. Upload the contents of this project folder into the repository root, including `index.html`. GitHub web UI: **Add file > Upload files > Commit changes**.
4. Open **Settings > Pages > Build and deployment**. Select **Deploy from a branch**, branch `main`, folder `/ (root)`, then **Save**.
5. Wait for deployment, then open `https://YOURUSERNAME.github.io/bonsai-journal/`.
6. On iPhone, open the URL in Safari and use **Share > Add to Home Screen**. On desktops, use the same URL in a browser.

## Step 3 — Family setup
1. Open the published app and click **Sign in with Google**.
2. Select **Create family collection**, choose a family name.
3. Add trees from **Collection > Add tree**.
4. To invite someone, open **Family > Invite family member**, enter their exact Google account email, and privately send the generated invitation code.
5. That person signs in with their own Google account, pastes the code, and joins the same collection. Codes expire after seven days. Members cannot delete tree profiles.
6. Try adding a care record on one device and checking the Journal on another device. Changes should appear automatically.

## Limitations / important notes
- **Google login:** The app uses `signInWithPopup`. Some mobile browser contexts block popups, particularly in installed PWAs. If sign-in fails, first open the GitHub Pages URL in a normal Safari/Chrome tab and sign in there. Depending on platform/browser, installed PWA authentication may still need a more advanced same-origin auth flow or Firebase Hosting. GitHub Pages + Firebase redirect sign-in can be affected by cross-origin storage restrictions.
- **Photos:** Storage is required for uploads; other functions can work without enabling Storage. Photos are limited to 5 MB. Firebase download URLs are bearer links: anyone who obtains one may access that photo even if they are not a family member. Do not upload sensitive photos. Strict private-image access would require a different photo retrieval architecture.
- **Deletion:** Admin tree deletion removes only the tree profile; existing logs/photos are intentionally retained in the database/storage and would need manual cleanup. V1 does not support removing members, revoking an invitation, or transferring admin ownership.
- **Care schedule:** V1 provides a watering-check overview and last watering date, not automatic reminders or recurring due dates.
- **Offline:** The service worker caches the app shell. Full offline Firestore editing and conflict resolution are not implemented; an internet connection is needed for reliable writes and synchronization.
- **Family membership:** A Google account can join only one family in V1. The invited email must match exactly. Child Google account availability depends on Google's account policies.
- **Costs:** GitHub Pages is free for public repositories. Firestore/Google sign-in may remain within free usage quotas. Firebase Storage requires Blaze billing; set budget alerts and monitor usage. Alerts are not hard spending limits.
- **Security:** Firebase web configuration is intentionally public. Security depends on the published Firestore and Storage rules. Do not commit secrets or private credentials.
- **Backups:** This V1 does not include export/import or automated backups. For valuable long-term records, enable appropriate Firebase/Google Cloud backups or add an export feature before relying on it as the sole archive.

## Troubleshooting
- `Firebase configuration needed`: edit `firebase-config.js`, then commit changes.
- `auth/unauthorized-domain`: add the exact GitHub Pages hostname in Firebase Authentication authorized domains.
- `Missing or insufficient permissions`: publish `firestore.rules`; check the signed-in Google email matches the invite.
- `storage/unauthorized` or 402/403: publish `storage.rules`, check bucket name, and enable Blaze billing.
- Stale app after an update: reload the page; if necessary clear website cache (but not Firebase data) or increment the cache name in `sw.js`.
- If the popup is blocked on iPhone Home Screen, try the app in Safari first. For guaranteed seamless mobile auth, consider switching hosting/auth architecture.
