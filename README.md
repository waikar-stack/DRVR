# Fleet Ledger v2

Fleet Ledger for fleet managers, owners and drivers. The app is hosted free on GitHub Pages; logins and data are on Firebase. No installs or programming are needed: everything is set up in the browser.

Follow the deployment guide (Fleet Ledger v2: deployment guide) step by step. In short:

1. Create a Firebase project, turn on Email/Password and Google sign-in.
2. Create the Firestore database, paste `firestore.rules` into its Rules tab and publish.
3. Add the index exemption: collection `access`, field `uid`, collection-group ascending.
4. Register a web app in Firebase and copy its settings into `firebase-config.js` (edit it here on GitHub).
5. Turn on GitHub Pages (Settings → Pages → main branch, root).
6. Add `YOUR-USERNAME.github.io` to Firebase Authentication → Settings → Authorized domains.

| File | What it is |
|---|---|
| `index.html` | The app |
| `firebase-config.js` | Your Firebase settings |
| `firestore.rules` | Security rules, pasted into Firebase (not used by GitHub) |
| `manifest.webmanifest`, `sw.js`, `icon-*.png` | Phone install and offline support |
| `DATA-MODEL.md` | How the data and roles are organised |
