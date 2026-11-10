# 220kV Warud Substation Management Suite — GitHub + Firebase

This repository packages the existing single-file management suite for web hosting and later Android packaging.

## What is included

- `index.html` — existing four-module suite with Firebase login gate for OT and TA Bill Managers.
- Attendance and Monthly Shift Duty Chart — open directly without login.
- OT Bill Manager and TA Bill Manager — require an authenticated employee account.
- Admin account management — create employees, list accounts, reset password to `Welcome1`, enable/disable accounts.
- All employee accounts are allowed into both OT and TA modules.
- PWA manifest and service worker for installable web-app behaviour.
- Firebase Cloud Functions for secure account administration.

## Important limitations / data handling

The four original modules are preserved and continue to use their existing browser `localStorage` storage. This means each phone/computer keeps its own bill and attendance data; this starter does **not** synchronize the OT/TA records between devices. Authentication is centralized through Firebase. If shared records across all employees/devices are required, a separate data-sync phase is needed.

GitHub Pages is static hosting only. Firebase Authentication and Cloud Functions provide the login and protected account-management operations. Do not put service-account keys or admin passwords in this repository.

## Part A — Create Firebase project

1. Open https://console.firebase.google.com/ and create a project, for example `warud-substation-suite`.
2. In **Build → Authentication → Sign-in method**, enable **Email/Password**.
3. In **Build → Firestore Database**, create a database (production mode is fine).
4. In **Project settings → General → Your apps**, add a Web app. Copy its Firebase configuration values.
5. In Firebase Authentication → Users, add the first user:
   - Email: `pdpatil@warud-management.firebaseapp.com`
   - Password: `Welcome1`
   - Display name: `Prashant D. Patil`
6. Copy the Firebase Web config into `index.html`, replacing these placeholders in `FIREBASE_CONFIG`:
   - `PASTE_FIREBASE_API_KEY`
   - `PASTE_FIREBASE_PROJECT_ID`
   - `PASTE_FIREBASE_APP_ID`
   Also replace `PASTE_FIREBASE_PROJECT_ID` in `authDomain` with the real project ID.

The login ID is translated internally to the email alias `loginid@warud-management.firebaseapp.com`; employees only type their short attendance login ID. No email inbox is needed for these accounts.

## Part B — Give `pdpatil` administrator privileges

This one-time step uses a private service-account key. Never upload it to GitHub or share it.

1. Install Node.js 20 LTS on your computer.
2. In Firebase Console → Project settings → Service accounts, generate a private key and save it in the project folder as `serviceAccountKey.json`.
3. Open a terminal in the project folder and run:

   ```bash
   npm install firebase-admin
   node scripts/set-admin.js pdpatil
   ```

4. Delete `serviceAccountKey.json` from the project folder after the role is assigned, or keep it securely outside the repository. `.gitignore` excludes it, but check GitHub upload selections carefully.
5. Sign out and sign back in as `pdpatil` to refresh the admin claim.

## Part C — Deploy secure Firebase backend

1. Install Firebase CLI:

   ```bash
   npm install -g firebase-tools
   firebase login
   ```

2. Rename `.firebaserc.example` to `.firebaserc` and replace `YOUR_FIREBASE_PROJECT_ID` with your project ID.
3. Deploy Firestore rules and Cloud Functions:

   ```bash
   cd functions
   npm install
   cd ..
   firebase deploy --only firestore:rules,functions
   ```

4. Cloud Functions deployment may require the Firebase project to use the **Blaze (pay-as-you-go)** plan and a billing account. Review current Firebase pricing before enabling billing. The app's expected light usage may fit free quotas, but billing is not guaranteed to be zero.

## Part D — Publish with GitHub Pages

1. Sign in at https://github.com/ and create a repository named `warud-substation-management`. For GitHub Free with GitHub Pages, the repository generally needs to be **Public**. The application source code will therefore be visible to anyone, but employee passwords and records must never be stored in GitHub. If you need a Private repository on a free plan, keep GitHub as private source control and publish the site using Firebase Hosting instead.
2. Upload the contents of this folder to the repository root. Upload `index.html`, `manifest.webmanifest`, `service-worker.js`, `firestore.rules`, `firebase.json`, `functions/`, `scripts/`, `icons/`, and `.gitignore`. Do not upload `serviceAccountKey.json`.
3. Open repository **Settings → Pages**.
4. Under **Build and deployment**, choose **Deploy from a branch** → `main` → `/(root)` → Save.
5. Wait for GitHub Pages to publish. Your URL will look like `https://YOUR_GITHUB_NAME.github.io/warud-substation-management/`.
6. In Firebase Console → Authentication → Settings → Authorized domains, add `YOUR_GITHUB_NAME.github.io` if it is not already listed.
7. Open the published URL and test administrator login.

Note: The repository contains application source, not employee data. Firebase security is enforced by Authentication and Cloud Functions; never commit service-account keys or passwords.

## Part E — Create employee accounts

1. Sign in to the published app using `pdpatil` and your administrator password.
2. Select **User Management**.
3. Enter the employee attendance login ID (for example `dmvirkhare`), employee name, and initial password `Welcome1`.
4. Select **Create employee account**.
5. Repeat for the 20 employees. Every employee can use both OT and TA Bill Managers.
6. Employees should change their initial password after first login. The current admin panel can reset a forgotten password to `Welcome1`.

The current uploaded attendance module has a 17-person employee master. Add or correct the remaining employee names in the attendance module separately; employee login accounts are managed independently in the admin panel.

## Part F — Android app

After the web app is live:

1. Open https://www.pwabuilder.com/ in Chrome.
2. Enter the GitHub Pages URL and run the PWA checks.
3. Confirm the manifest and icon are detected; generate the Android package (APK/AAB) using the Android packaging option offered by PWABuilder.
4. Test the APK on your Android phone before distributing it to staff.

The PWA can also be installed directly from Chrome using **⋮ → Add to Home screen**. A separately signed APK is a later packaging step, not included in this source ZIP.

## Initial credentials

- Administrator login ID: `pdpatil`
- Initial administrator password: `Welcome1` (change after first login)
- Employee initial password: `Welcome1` (set per employee during account creation)

Do not use the same initial password permanently. Share each employee's credentials privately.
