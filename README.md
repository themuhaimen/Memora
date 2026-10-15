# MEMORA

MEMORA is a responsive reminder workspace for keeping track of documents, medicines, orders, bills, and personal tasks. The React app keeps a local copy for quick access, while the Node API stores reminder schedules and Web Push subscriptions for background delivery.

## Features

- Responsive desktop workspace and compact mobile navigation
- Reminder capture, photo attachments, calendar, search, and status filters
- Local browser storage for fast startup and offline-friendly access
- Service-worker notifications and server-scheduled Web Push
- Animated brand mark and reduced-motion support
- No login or profile fields; reminders are private to the current browser installation

Photo capture currently helps organize and annotate an image; it does not call a hosted vision/AI model.

## Run Locally

```sh
npm install
npm run push:keys
npm run dev:full
```

Open the Vite URL printed by the dev server, normally `http://localhost:5173`. On app open, MEMORA offers a permission dialog. Notifications remain off unless the user explicitly enables them, browser permission is granted, and the per-device push subscription and schedule are saved successfully. `npm run push:keys` creates local VAPID credentials in `.env`; this file is ignored by Git. Keep the server running for scheduled delivery. The browser must support Web Push, and permission must be granted.

Useful commands:

```sh
npm run dev       # Frontend only
npm run server    # API server (port 8787)
npm run build     # Production frontend build
npm run lint
```

## Android APK (Windows)

The Android Studio project is generated with Capacitor. Install Android Studio, then open **More Actions → SDK Manager** and install **Android SDK Platform 34**, **Android SDK Build-Tools 34.0.0**, **Android SDK Platform-Tools**, and the latest **Android SDK Command-line Tools**. Java 17 is already required by this project.

In a new PowerShell window, set the SDK path for that terminal and build:

```powershell
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"
$env:ANDROID_SDK_ROOT = $env:ANDROID_HOME
$env:Path += ";$env:ANDROID_HOME\platform-tools;$env:ANDROID_HOME\cmdline-tools\latest\bin"
npm run android:apk
```

The debug APK is created at `android/app/build/outputs/apk/debug/app-debug.apk`. To open the native project in Android Studio instead, run `npm run android:open`.

Build first before using the server to serve production files. The API exposes `/api/health`; Vite proxies `/api` to the local Node server during development. After `npm run build`, `npm run server` serves both the built app and API on one origin.

## Architecture And Deployment

- `src/` is the React/Vite client.
- `server/` is an Express API with persistent single-instance JSON storage.
- `public/sw.js` receives push events and opens the app when a notification is clicked.
- Reminder details and attached photos stay in that browser's local storage. After explicit notification opt-in, the server receives only the schedule fields needed for delivery; each browser gets a random private device key and its own push subscription. There is no API to read another device's reminder list, and a push is sent only to its owning browser subscription.
- Web Push uses VAPID keys generated locally; set `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, and `VAPID_SUBJECT` in the server environment in deployment.
- Production push requires HTTPS, a reachable always-on server, and a browser/device that supports Web Push. A closed-app notification is delivered by the browser's push service, not by a JavaScript timer.

This is a local-first, per-browser foundation, not an account-backed multi-device service. Before public deployment, replace JSON storage with a managed database, add account-based recovery only if cross-device sync is required, configure backups/monitoring, and set an appropriate VAPID contact. Clearing site data removes that browser's reminders and private device key; reminders do not automatically move between devices.
