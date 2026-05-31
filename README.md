<div align="center">

# 🏛️ GovDoc Vault

### Secure Government Document Storage & Family Sharing Portal

[![Firebase](https://img.shields.io/badge/Firebase-10.12.2-orange?logo=firebase)](https://firebase.google.com)
[![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-yellow?logo=javascript)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Live-brightgreen)](https://govdoc-prod.web.app)

**A production-grade, government-style digital document management system**  
Built with Vanilla JS + Firebase — No frameworks, no dependencies.

[🌐 Live Demo](https://govdoc-prod.web.app/) · [📖 Deployment Guide](DEPLOYMENT.md) · [🐛 Report Bug](https://github.com/your-username/gov-doc-vault/issues)

---

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Firebase Setup](#-firebase-setup)
- [Environment Configuration](#-environment-configuration)
- [Running Locally](#-running-locally)
- [Firestore Schema](#-firestore-schema)


---

## 🔍 Overview

GovDoc Vault is a national-level digital document management portal inspired by DigiLocker. Citizens can securely store government-issued documents, manage them, and share controlled access with family members — all protected by OTP-based authentication and server-side security rules.

Built as a full-stack project using only Vanilla HTML, CSS, and JavaScript on the frontend with Firebase as the complete backend infrastructure.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔐 **OTP Authentication** | Phone-based login via Firebase Auth — no passwords |
| 📄 **Document Upload** | Upload PDF, JPG, PNG — max 5 MB per file |
| ✏️ **Document Management** | Edit metadata, delete with Storage + Firestore cleanup |
| 👨‍👩‍👧 **Family Sharing** | Share documents by mobile number with view access |
| 🚫 **Revoke Access** | Remove family member access instantly |
| 👤 **User Profile** | Name, DOB, verified phone, document statistics |
| 📊 **Family Access Page** | View all shared documents and family member access |
| 📝 **Audit Logging** | Every action logged with timestamp, module, masked PII |
| 🛡️ **Security Rules** | Firestore + Storage rules enforce owner-only access |
| 📱 **Responsive Design** | Works on mobile, tablet, and desktop |

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | Vanilla HTML5, CSS3, JavaScript ES Modules |
| **Authentication** | Firebase Authentication (Phone OTP) |
| **Database** | Cloud Firestore |
| **File Storage** | Firebase Storage |
| **Hosting** | Firebase Hosting |
| **Design** | Custom CSS Design System — no frameworks |
| **Fonts** | Noto Sans (Google Fonts — multilingual support) |

> Zero npm dependencies. Zero build tools. Zero frameworks.

---

## 🚀 Getting Started

### Prerequisites

```bash
node --version   # v18 or higher required
npm install -g firebase-tools
firebase --version   # v13 or higher
```

### 1. Clone the repository

```bash
git clone https://github.com/your-username/gov-doc-vault.git
cd gov-doc-vault
```

### 2. Set up Firebase

See [Firebase Setup](#-firebase-setup) below.

### 3. Configure environment

```bash
cp __env.example.js __env.js
# Edit __env.js with your Firebase credentials
```

### 4. Run locally

```bash
firebase emulators:start --only hosting
# Open http://localhost:5000
```

---

## 🔥 Firebase Setup

### Step 1 — Create Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → Name it `govdoc-vault`
3. Disable Google Analytics (not required)

### Step 2 — Enable services

```
Authentication  → Sign-in method → Phone → Enable
Firestore       → Create database → Production mode → Region: asia-south1
Storage         → Get started → Production mode → Same region
```

### Step 3 — Get config values

```
Firebase Console → Project Settings → Your Apps → Web App (</> icon)
→ Register app → Copy the firebaseConfig object
```

### Step 4 — Link project

```bash
firebase login
firebase use --add   # select your project, alias: prod
```

### Step 5 — Deploy security rules

```bash
firebase deploy --only firestore,storage --project prod
```

---

## ⚙️ Environment Configuration

Create `__env.js` in the project root (this file is gitignored):

```js
window.__GOV_ENV = {
  FIREBASE_API_KEY:             'your-api-key',
  FIREBASE_AUTH_DOMAIN:         'your-project.firebaseapp.com',
  FIREBASE_PROJECT_ID:          'your-project-id',
  FIREBASE_STORAGE_BUCKET:      'your-project.appspot.com',
  FIREBASE_MESSAGING_SENDER_ID: 'your-sender-id',
  FIREBASE_APP_ID:              'your-app-id',
};
```

> ⚠️ **Never commit `__env.js`** — it contains your Firebase credentials.  
> For production deployment, generate this file from CI/CD secrets.

---

## 💻 Running Locally

```bash
# Start local server (uses your real Firebase project)
firebase emulators:start --only hosting

# App available at:
http://localhost:5000
```

**Test OTP without burning SMS quota:**
```
Firebase Console → Authentication → Sign-in method
→ Phone → Phone numbers for testing → Add number
  Phone: +91 9999999999
  Code:  123456
```

---

## 🚢 Deployment

### Deploy everything

```bash
firebase deploy --project prod
```

### Deploy specific parts

```bash
# Only website files (HTML/CSS/JS)
firebase deploy --only hosting --project prod

# Only Firestore rules
firebase deploy --only firestore --project prod

# Only Storage rules
firebase deploy --only storage --project prod
```

### GitHub Actions CI/CD

Add these secrets to your GitHub repository:

| Secret | Value |
|---|---|
| `FIREBASE_TOKEN` | Output of `firebase login:ci` |
| `PROD_API_KEY` | Firebase apiKey |
| `PROD_AUTH_DOMAIN` | Firebase authDomain |
| `PROD_PROJECT_ID` | Firebase projectId |
| `PROD_STORAGE_BUCKET` | Firebase storageBucket |
| `PROD_SENDER_ID` | Firebase messagingSenderId |
| `PROD_APP_ID` | Firebase appId |

See [DEPLOYMENT.md](DEPLOYMENT.md) for the complete CI/CD pipeline.

---

## 🔒 Security Architecture

### Authentication
- Phone OTP only — zero password storage risk
- `browserSessionPersistence` — token expires when tab closes
- Every protected page guarded by `onAuthStateChanged`

### Firestore Rules
- **Default deny-all** — only explicitly allowed operations pass
- `ownerId` is immutable after document creation
- Shared members get **read-only access** — cannot write or delete
- Field-level validation on every write (title length, file size cap)
- `familyLinks` are append-only — delete permanently blocked for audit

### Storage Rules
- Path-scoped to `documents/{uid}/` — cross-user access impossible
- Server-side MIME validation: `application/pdf`, `image/jpeg`, `image/png`, `image/jpg`
- File size capped at **5 MB** at the rules layer
- `update: false` — silent file overwrite permanently blocked

### Logging
- Every action logged via `logger.service.js`
- Phone numbers **masked** to last 4 digits in all logs (`+91XXXXXX3210`)
- Session log cleared on sign-out (PII hygiene)
- Debug logs suppressed in production (`APP_ENV=production`)
- 
---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 👨‍💻 Author

Ronak Vaghela 

<div align="center">

Made with ❤️ Passion and for the design purpose

⭐ Star this repo if you found it useful!

</div>
