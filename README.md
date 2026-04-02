# GovDoc Vault

**Secure Government Document Storage & Sharing Portal**

A production-grade citizen portal for securely storing, managing, and sharing government-issued documents with family members — built with Vanilla JS and Firebase.

---

## Overview

GovDoc Vault is a national-level digital document management system inspired by DigiLocker. Citizens can register via OTP, upload government documents, manage them securely, and grant controlled access to family members — all without passwords.

---

🔗 **Live Demo:**  
https://govdoc-prod.web.app/

---

## Features

- **OTP Authentication** — Phone-based login via Firebase Auth (no passwords)
- **Document Upload** — PDF, JPG, PNG support with type and size validation (5 MB limit)
- **Document Management** — Edit metadata, delete with Storage + Firestore cleanup
- **Family Sharing** — Share documents by phone number with access control
- **Revoke Access** — Remove family member access at any time
- **User Profile** — Name, DOB, verified phone, document statistics
- **Audit Logging** — Every action logged with timestamp, module, and masked PII
- **Security Rules** — Firestore and Storage rules enforce owner-only access server-side

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (ES Modules) |
| Auth | Firebase Authentication (Phone OTP) |
| Database | Cloud Firestore |
| File Storage | Firebase Storage |
| Hosting | Firebase Hosting |
| Styling | Custom CSS Design System (no frameworks) |

---

## Project Structure

```
gov-doc-vault/
├── index.html                     Auth entry (OTP login)
├── dashboard.html                 Document management dashboard
├── profile.html                   User profile page
├── firebase.json                  Hosting config + security headers
├── firestore.indexes.json         Composite query indexes
├── .firebaserc                    Project aliases (dev / prod)
├── .gitignore
├── .env.example
├── __env.example.js               Runtime env injection template
├── DEPLOYMENT.md                  Full deployment runbook
├── css/
│   ├── base.css                   Design tokens, reset, typography
│   ├── layout.css                 Page layouts, sidebar, dashboard
│   ├── components.css             UI components
│   └── utilities.css              Helper classes
├── js/
│   ├── config/
│   │   ├── env.js                 Runtime env loader + validation
│   │   └── firebase.config.js     Firebase init, auth persistence
│   ├── services/
│   │   ├── auth.service.js        OTP send, verify, logout
│   │   ├── db.service.js          Firestore CRUD + ownership layer
│   │   ├── storage.service.js     File upload (with progress), delete
│   │   ├── share.service.js       Share/revoke orchestration
│   │   └── logger.service.js      Structured logging, audit trail
│   ├── modules/
│   │   ├── auth.module.js         Login page UI controller
│   │   ├── documents.module.js    Dashboard data + grid controller
│   │   ├── upload.module.js       Upload form + 2-phase commit
│   │   ├── doc-management.module.js  Edit + delete controller
│   │   ├── share.module.js        Share modal UI controller
│   │   └── profile.module.js      Profile fetch/update controller
│   ├── validators/
│   │   ├── file.validator.js      File type, size, MIME validation
│   │   └── form.validator.js      Input sanitisation
│   └── utils/
│       ├── dom.utils.js           Toast, loader, DOM helpers
│       └── session.utils.js       Auth guard, redirect helpers
└── rules/
    ├── firestore.rules            Production Firestore security rules
    └── storage.rules              Production Storage security rules
```

---

## Getting Started

### Prerequisites

- Node.js ≥ 18
- Firebase CLI: `npm install -g firebase-tools`

### 1. Clone the repository

```bash
git clone https://github.com/your-username/gov-doc-vault.git
cd gov-doc-vault
```

### 2. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Create a new project
3. Enable **Phone Authentication** → Auth → Sign-in method → Phone
4. Create a **Firestore database** → Production mode → Region: `asia-south1`
5. Enable **Storage** → Production mode → Same region

### 3. Configure environment

```bash
cp __env.example.js __env.js
```

Fill in `__env.js` with your Firebase project credentials from:
**Firebase Console → Project Settings → Your Apps → SDK config**

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

> `__env.js` is gitignored — never commit this file.

### 4. Link Firebase project

```bash
firebase login
firebase use --add   # select your project, alias: dev
```

### 5. Deploy security rules

```bash
firebase deploy --only firestore,storage
```

### 6. Run locally

```bash
firebase emulators:start --only hosting
```

Open `http://localhost:5000`

---

## Security Architecture

### Authentication
- Phone OTP only — no password storage risk
- Session persistence (`browserSessionPersistence`) — token expires on tab close
- Every protected page guarded by `onAuthStateChanged`

### Firestore Rules
- Default deny-all — only explicitly allowed operations pass
- `ownerId` is immutable after document creation
- Shared members get read-only access — cannot write or delete
- Field-level validation on every write (title length, size cap, MIME type)

### Storage Rules
- Path-scoped to `documents/{uid}/` — cross-user access impossible
- MIME type validated server-side: `application/pdf`, `image/jpeg`, `image/png`
- File size capped at 5 MB at the rules layer
- `update: false` — silent file overwrite permanently blocked

### Logging
All major actions are logged via `logger.service.js`:
- Timestamp, level (INFO/WARN/ERROR/DEBUG), module, action, context
- Phone numbers masked to last 4 digits in all log output
- Session log cleared on sign-out (PII hygiene)
- Debug logs suppressed in production (`APP_ENV=production`)

---

## Deployment

See [DEPLOYMENT.md](./DEPLOYMENT.md) for the full guide including:
- Firebase Hosting deployment steps
- GitHub Actions CI/CD pipeline
- Environment variable management
- Custom domain setup
- Post-deploy verification checklist

---

## Design System

Government-grade UI — clean, minimal, high readability.

| Token | Value |
|---|---|
| Primary | `#1A3A5C` (Deep Navy) |
| Accent | `#C8A214` (Saffron Gold) |
| Background | `#F4F6F9` (Off-white) |
| Font | Noto Sans (multilingual support) |
| Grid | 8px base spacing system |
| Radius | 4–8px (document portal aesthetic) |

---

## Firestore Schema

```
users/{uid}
  name, phone, dob, createdAt, updatedAt

documents/{docId}
  ownerId, title, type, fileRef, fileURL,
  mimeType, sizeBytes, sharedWith[], uploadedAt, updatedAt

familyLinks/{linkId}
  docId, requestorId, targetPhone, targetUid,
  status (pending|accepted|revoked), createdAt, updatedAt
```

---

## License

This project is built for government-style academic and portfolio demonstration purposes.

---

## Author

Built with a phased, production-grade engineering approach using Firebase + Vanilla JS.
