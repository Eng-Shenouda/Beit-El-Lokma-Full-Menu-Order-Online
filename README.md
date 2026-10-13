<div align="center">

# 🍽️ Beit El Lokma — بيت اللقمة

**Full-menu online ordering for a restaurant — Firebase Auth on the front, n8n automation on the back.**

### 🔗 [Live Demo → order-site-ff280.web.app](https://order-site-ff280.web.app/)

![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/Vanilla_JS-F7DF1E?logo=javascript&logoColor=black)
![Firebase](https://img.shields.io/badge/Firebase-Auth_%2B_Hosting-FFCA28?logo=firebase&logoColor=black)
![n8n](https://img.shields.io/badge/n8n-Workflow-EA4B71?logo=n8n&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

</div>

---

## 🇪🇬 نبذة سريعة

موقع طلب أونلاين لمطعم "بيت اللقمة": منيو كامل (+45 صنف) بواجهة عربية RTL. العميل يسجّل دخول (إيميل/باسورد أو Google)، يملا السلة، ويبعت الطلب. الطلب بيتراجع على السيرفر في **n8n**، يتسجل في **Google Sheets**، ويتبعت إيميل تأكيد للعميل وإيميل تنبيه لصاحب المطعم.

---

## 🌐 Live Demo

**https://order-site-ff280.web.app/**

Sign up with an email (or Google), add items to the cart, and place an order to see the full flow: validation → Google Sheets → confirmation emails.

---

## ✨ Features

- **Arabic-first, RTL UI** — responsive, accessible, respects `prefers-reduced-motion`.
- **Full menu, 7 categories, 45+ items** — meals, burgers & sandwiches, grills, crispy chicken, sides, desserts, drinks.
- **Live cart** — quantity steppers (max 20 per item), running total, floating cart bar on mobile.
- **Authentication required before ordering** — Email/Password and Google Sign-In via Firebase Auth.
- **Server-side validation** — the n8n workflow verifies the Firebase ID token, re-prices every item from its own menu, and validates name, phone and quantities. The client's total is never trusted.
- **Order logging** — every valid order is appended to a Google Sheet.
- **Automatic emails** — confirmation to the customer (with ETA) and a new-order alert to the owner.
- **Friendly error handling** — Arabic error messages for auth and order failures.

---

## 🏗️ Architecture

```
┌──────────────┐   ID token + order    ┌─────────────────────────────────────────────┐
│  Static site │ ────────────────────▶ │                n8n Workflow                 │
│ (Firebase    │                       │                                             │
│  Hosting)    │                       │  Webhook                                    │
│              │                       │     ↓                                       │
│ Firebase Auth│                       │  Verify Login  (Firebase accounts:lookup)   │
└──────────────┘                       │     ↓                                       │
        ▲                              │  Build Order   (validate + price on server) │
        │                              │     ↓                                       │
        │   { order_id, eta_minutes }  │  Is Order Valid?                            │
        └───────────────────────────── │   ├─ yes → Google Sheets (Orders)           │
                                       │   │          → Email customer → Email owner │
                                       │   │          → Respond 200                  │
                                       │   └─ no  → Respond 400 { errors }           │
                                       └─────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
.
├── index.html                    # Whole front-end: login gate, menu, cart, order form
├── config.js                     # Firebase config + n8n webhook URL (edit once)
├── 404.html                      # Firebase Hosting default 404 page
├── firebase.json                 # Hosting config (serves the project root)
├── .firebaserc                   # Firebase project alias
├── add_website_branch.json       # n8n workflow: complete website-order branch (import this)
├── add_to_existing_workflow.json # n8n nodes to bolt onto an existing workflow
└── LICENSE                       # MIT
```

> `add_website_branch.json` and `add_to_existing_workflow.json` are excluded from the deployed site via `firebase.json`.

---

## 🚀 Getting Started

### Prerequisites

- A [Firebase](https://console.firebase.google.com/) project
- An [n8n](https://n8n.io/) instance (cloud or self-hosted) with **Gmail** and **Google Sheets** credentials
- [Firebase CLI](https://firebase.google.com/docs/cli) — `npm i -g firebase-tools`

### 1. Firebase

1. Create a project and register a **Web app**.
2. **Authentication → Sign-in method**: enable **Email/Password** and **Google**.
3. **Authentication → Settings → Authorized domains**: add your hosting domain (and `localhost` for local testing).

### 2. Configure the front-end

Open `config.js` and fill in your values:

```js
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_WEB_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  appId: "YOUR_APP_ID"
};
const WEBHOOK_URL = "https://YOUR-N8N-HOST/webhook/YOUR-PATH";
```

### 3. Set up n8n

1. Import **`add_website_branch.json`** into n8n.
2. Replace the placeholders:

   | Where | Placeholder | Replace with |
   |---|---|---|
   | `Verify Web Login` node (URL) | `FIREBASE_WEB_API_KEY` | Your Firebase Web API key |
   | `Build Order` node (code) | `OWNER_EMAIL`, `ETA_MINUTES` | Owner's email and delivery estimate |
   | `Append Website Order` node | `REPLACE_WITH_SPREADSHEET_ID` | Your Google Sheet ID |
   | Gmail / Sheets nodes | `REPLACE_ME` | Your n8n credentials |

3. Create a sheet tab named **`Orders`** with these headers:
   `Received At` · `Order ID` · `Customer Name` · `Phone` · `Item` · `Quantity` · `Total`
4. Make sure the webhook **path** matches the one in `WEBHOOK_URL` (the template uses `web-order`), then **activate** the workflow and use the **production** webhook URL.

> Already have a workflow? Use `add_to_existing_workflow.json` instead — it adds the verify → validate → email → respond nodes to a webhook you already own.

### 4. Run locally

```bash
# any static server works
npx serve .
# or
python3 -m http.server 5500
```

Open `http://localhost:5500`. Opening `index.html` directly from the filesystem (`file://`) will not work with Firebase Auth.

### 5. Deploy

```bash
firebase login
firebase deploy --only hosting
```

---

## 🔌 Webhook API

**`POST`** `WEBHOOK_URL`

```json
{
  "idToken": "<Firebase ID token>",
  "customer_name": "Mohamed Ali",
  "phone": "01012345678",
  "items": [
    { "name": "وجبة العائلة", "qty": 1 },
    { "name": "كولا", "qty": 2 }
  ]
}
```

**Success — `200`**

```json
{ "status": "success", "order_id": "ORD-1234567", "eta_minutes": 45 }
```

**Validation error — `400`**

```json
{ "status": "error", "errors": ["رقم الموبايل غير صحيح"] }
```

### Validation rules

| Field | Rule |
|---|---|
| `idToken` | Must be a valid, unexpired Firebase ID token |
| `customer_name` | At least 2 characters (falls back to the account's display name) |
| `phone` | Egyptian mobile: `^01[0125]\d{8}$` |
| `items` | Non-empty, ≤ 45 lines; each name must exist in the server menu |
| `qty` | Integer from 1 to 20 |
| `total` | Computed by the workflow — never taken from the client |

---

## 🍔 Updating the Menu

The menu has to be edited in **two places** and kept identical:

1. **`index.html`** — the `RAW` constant (format: `category|emoji|name|description|price`).
2. **n8n `Build Order` node** — the `MENU` object (name → price).

If an item name or price differs between the two, the order will be rejected or priced differently.

> `config.js` also contains a `MENU` / `EMOJI` stub (4 items) that `index.html` does **not** use; it can be safely removed.

---

## 🔐 Security Notes

- The Firebase **Web API key is not a secret** (it identifies the project), but restrict it by HTTP referrer in Google Cloud Console and keep authorized domains tight.
- Orders are trusted only after the ID token is verified server-side and prices are recomputed in n8n.
- The template webhook uses `allowedOrigins: "*"`. For production, set it to your site's origin.
- Never commit n8n credentials, OAuth tokens, or `.env` files.

---

## 🛠️ Tech Stack

| Layer | Tech |
|---|---|
| Front-end | HTML, CSS, vanilla JavaScript (no build step) |
| Auth | Firebase Authentication (JS SDK v10, compat) |
| Hosting | Firebase Hosting |
| Automation | n8n (Webhook, HTTP Request, Code, IF, Google Sheets, Gmail) |
| Fonts | Cairo, Lalezar (Google Fonts) |

---

## 🗺️ Roadmap Ideas

- Delivery address and order notes
- Order status tracking for customers
- Admin dashboard for the owner
- Online payment
- Multi-language menu (AR/EN toggle)

---

## 📄 License

Released under the [MIT License](LICENSE) © 2026 Shenouda Romany.
