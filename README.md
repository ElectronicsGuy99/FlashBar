# FlashBar

A lightweight, real-time announcement banner system powered by Firebase. Drop the banner into any webpage via an iframe, and control what it shows from a password-protected dashboard — no redeploys needed. To see a demo of the FlashBar_Dashboard, click [here]([url](https://electronicsguy99.github.io/FlashBar/FlashBar_Dashboard.html)).

---

## How it works

FlashBar has two files:

- **`FlashBar_Banner.html`** — the banner itself. Embed this in your site via an `<iframe>`. It listens to Firebase in real time and updates instantly whenever you publish a change.
- **`FlashBar_Dashboard.html`** — the admin panel. Open this in a browser, sign in, and manage your announcements. Host it anywhere — even locally.

All announcement data lives in a single Firestore document. The banner reads it live; the dashboard writes to it.

---

## Setup

### 1. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and click **Add project**.
2. Give it a name, follow the prompts, and click **Create project**.

### 2. Enable Firestore

1. In the Firebase console, go to **Build → Firestore Database**.
2. Click **Create database**.
3. Choose **Start in production mode** and pick a region close to your users.
4. Click **Enable**.

### 3. Set Firestore security rules

In the Firestore console, go to the **Rules** tab and replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /banner/main {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

Click **Publish**. This allows anyone to read the banner (so the public iframe works) but restricts writes to authenticated users only.

### 4. Enable Email/Password authentication

1. Go to **Build → Authentication** and click **Get started**.
2. Under **Sign-in method**, enable **Email/Password**.
3. Go to the **Users** tab and click **Add user**.
4. Enter the email and password you want to use to log into the dashboard.

### 5. Get your Firebase config

1. In the Firebase console, click the gear icon → **Project settings**.
2. Scroll down to **Your apps** and click the **</>** (Web) icon to register a web app.
3. Give it a nickname (e.g. `flashbar`) and click **Register app**.
4. Copy the `firebaseConfig` object that appears.

### 6. Add your config to both files

Open both `FlashBar_Banner.html` and `FlashBar_Dashboard.html` and find this block near the top of the `<script>` section in each file:

```js
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT_ID.firebasestorage.app",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

Replace each placeholder with the values from your Firebase project. Do this in **both files**.

---

## Embedding the banner

Host `FlashBar_Banner.html` somewhere publicly accessible (GitHub Pages, Netlify, your own server, etc.), then embed it on your site with an `<iframe>`:

```html
<iframe
  src="https://your-domain.com/FlashBar_Banner.html"
  style="width:100%; height:120px; border:none; display:block;"
  title="Announcements"
></iframe>
```

The banner is exactly 120px tall. When there is no active announcement, the space is still occupied — if you want to hide it when empty, you can use JavaScript to listen for `postMessage` events or simply keep it always visible.

---

## Using the dashboard

Open `FlashBar_Dashboard.html` in any browser (you can open it directly from your file system or host it). Sign in with the email and password you created in step 4.

### Creating an announcement

Click **+ Add Announcement** and fill in the fields:

| Field | Description |
|---|---|
| Announcement Text | The message shown in the banner. Over 200 characters will reduce the font size automatically. |
| Icon | Any single emoji. Defaults to 📢. |
| Background Color | Hex color for the banner background. Defaults to `#4f8ef7`. |
| Text Color | Hex color for the text. Defaults to `#ffffff`. |
| Button Text | Optional. Label for a call-to-action button. |
| Button Link | Optional. URL the button links to. Required if Button Text is set. |
| Start Date/Time | Optional. Leave blank to show the banner immediately. |
| End Date/Time | Optional. Leave blank to show the banner indefinitely. |

Click **Save to Firebase** to publish. The live banner updates within seconds.

### Scheduling announcements

You can queue multiple announcements by giving each one a start and end time. FlashBar enforces a rule: **only one announcement may be active at any given moment**. If two would overlap, a conflict warning blocks saving until you fix the dates.

Use the **⛓️ Set start time after another announcement's end** and **⛓️ Set end time before another announcement's start** buttons to chain announcements together without overlap.

Expired announcements (past their end time) are automatically removed from the list every 60 seconds while the dashboard is open.

### Tools

**Announcement Timeline** — a collapsible view of all your announcements sorted chronologically, with live countdowns for active ones.

**Preview at a specific time** — pick any date and time to see which announcement would be showing, rendered as a live preview of the banner.

### Other actions

- **Duplicate** — copies an announcement including all its settings.
- **Revert Changes** — discards any unsaved edits and restores the last saved state.
- **Clear All** — deletes every announcement from Firebase immediately.

---

## Hosting on GitHub Pages

The simplest way to host both files for free:

1. Push this repository to GitHub.
2. Go to **Settings → Pages** in your repo.
3. Under **Source**, select your branch (e.g. `main`) and click **Save**.
4. GitHub will give you a URL like `https://yourusername.github.io/your-repo/`.
5. Your banner will be at `https://yourusername.github.io/your-repo/FlashBar_Banner.html` and your dashboard at `https://yourusername.github.io/your-repo/FlashBar_Dashboard.html`.

> **Note:** The dashboard is publicly accessible at that URL. Anyone who visits it will see the login screen, but cannot make changes without your Firebase credentials. If you'd prefer it to be private, run the dashboard locally by just opening the file in your browser — it doesn't need to be hosted.

---

## Firebase free tier

FlashBar is designed to stay well within Firebase's free Spark plan limits. A single Firestore document with a real-time listener uses a negligible number of reads. You are unlikely to incur any costs unless you have extremely high traffic.

---

## Customization

All visual styling is controlled by CSS variables at the top of each file's `<style>` block. The key ones:

```css
--bg         /* page background */
--accent     /* highlight/focus color */
--success    /* "no announcements" status color */
--danger     /* error message color */
```

The banner's appearance (colors, icon, text) is set per-announcement from the dashboard, so you rarely need to edit the HTML directly.
