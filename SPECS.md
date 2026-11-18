\# SPECS.md — TeacherToolchest Architecture \& Setup Guide



\## 1. Project Overview \& Goals

\*\*TeacherToolchest\*\* (`https://teachertoolchest.web.app`) is a modular, multi-tool AI platform built for K–12 educators. It solves two core operational constraints:

1\. \*\*Firebase Project Consolidation:\*\* Houses an unlimited number of small, single-purpose AI classroom utilities inside \*\*one\*\* Firebase project to stay under district project quotas (e.g., 10-project limits) and remain 100% on the Firebase \*\*Spark (Free) Tier\*\*.

2\. \*\*Cloneable Starter Template:\*\* The `v1.0.0-starter` release (`template-base` branch) contains the core \*\*TeacherToolchest\*\* shell plus one reference tool (`exit-ticket-gen`). Any teacher can fork or clone the repository, update four configuration points, and deploy their own independent instance via Google AI Studio, Git, and Firebase Hosting.



\---



\## 2. Tech Stack

\* \*\*Frontend:\*\* React 19, TypeScript, Vite

\* \*\*Styling \& UI:\*\* Tailwind CSS, Lucide React (`lucide-react`)

\* \*\*AI SDK:\*\* Google Gen AI SDK (`@google/genai`) — default model: `gemini-2.5-flash`

\* \*\*Backend \& Hosting (Spark Free Tier):\*\*

&#x20; \* \*\*Firebase Authentication:\*\* Google Sign-In (with optional `@district.org` domain restriction)

&#x20; \* \*\*Cloud Firestore:\*\* Namespaced per-user, per-tool persistence

&#x20; \* \*\*Firebase Hosting:\*\* Static SPA deployment



\---



\## 3. Directory Structure

```text

├── GEMINI.md                  # AI coding agent instructions (for AI Studio / Gemini CLI)

├── SPECS.md                   # Project architecture \& teacher clone setup guide

├── .env.example               # Template for local environment variables

├── .firebaserc                # Firebase CLI project alias target

├── firebase.json              # Firebase Hosting \& Firestore config

├── firestore.rules            # Strict user-scoped Firestore security rules

└── src/

&#x20;   ├── config/

&#x20;   │   ├── appConfig.ts       # App branding ("TeacherToolchest"), domain filter, default model

&#x20;   │   └── firebaseConfig.ts  # Firebase project credentials (teachers edit this on clone)

&#x20;   ├── lib/

&#x20;   │   ├── firebase.ts        # Firebase Auth \& Firestore initialization

&#x20;   │   ├── gemini.ts          # Unified @google/genai client (supports .env + BYOK)

&#x20;   │   └── storage.ts         # Namespaced Firestore CRUD helpers for tools

&#x20;   ├── components/

&#x20;   │   ├── layout/            # AppShell, Sidebar, Navbar, ApiKeyModal, PrivacyBanner

&#x20;   │   └── ui/                # Reusable buttons, cards, copy-to-clipboard, loaders

&#x20;   ├── types/

&#x20;   │   └── tool.ts            # Shared TypeScript interfaces for micro-tools

&#x20;   └── tools/

&#x20;       ├── registry.ts        # Central manifest registering all active tools

&#x20;       └── exit-ticket-gen/   # Reference Starter Tool #1

&#x20;           ├── index.ts       # Exports ToolDefinition manifest

&#x20;           └── ExitTicketTool.tsx

```



\---



\## 4. Micro-Tool Plugin Contract

Every tool inside `src/tools/<tool-slug>/` must implement the `ToolDefinition` interface defined in `src/types/tool.ts`:



```typescript

import { ComponentType } from 'react';



export type ToolCategory = 'Planning' | 'Assessment' | 'Communication' | 'Differentiation' | 'Productivity';



export interface ToolDefinition {

&#x20; id: string;                  // Must match folder slug (e.g., 'exit-ticket-gen')

&#x20; name: string;                // Display title (e.g., 'Exit Ticket Generator')

&#x20; description: string;         // 1-sentence summary shown on dashboard card

&#x20; category: ToolCategory;

&#x20; icon: string;                // Lucide icon name (e.g., 'ClipboardCheck')

&#x20; component: ComponentType;    // Lazy-loaded or direct React component

&#x20; version: string;

}

```



\---



\## 5. Firestore Schema \& Security Rules

To prevent data collisions across dozens of teacher-built tools inside one database, all tool data is strictly scoped under the authenticated user's UID and the tool's `id`:



\* \*\*Path:\*\* `users/{userId}/tools/{toolId}/saved\_items/{itemId}`

\* \*\*Document Fields:\*\*

&#x20; \* `title` (string)

&#x20; \* `inputPayload` (map)

&#x20; \* `outputContent` (string or map)

&#x20; \* `createdAt` (timestamp)

&#x20; \* `updatedAt` (timestamp)



\*\*`firestore.rules`:\*\*

```javascript

rules\_version = '2';

service cloud.firestore {

&#x20; match /databases/{database}/documents {

&#x20;   match /users/{userId}/tools/{toolId}/saved\_items/{itemId} {

&#x20;     allow read, write: if request.auth != null \&\& request.auth.uid == userId;

&#x20;   }

&#x20; }

}

```



\---



\## 6. API Key \& Free-Tier Security Model

Because Firebase Hosting on the Spark Free Tier serves static assets without server-side Cloud Functions, Gemini API calls run from the browser via `@google/genai`:

\* \*\*Single-Teacher / Personal Deployment:\*\* Set `VITE\_GEMINI\_API\_KEY` in `.env.local` before running `npm run build \&\& firebase deploy`. Restrict the API key in Google Cloud Console to your Firebase Hosting domain (e.g., `https://teachertoolchest.web.app/\*` or your cloned `https://<your-project-id>.web.app/\*`).

\* \*\*Shared Multi-Teacher Deployment:\*\* Leave `VITE\_GEMINI\_API\_KEY` blank. Each logged-in teacher enters their own free Google AI Studio API key once in the \*\*TeacherToolchest\*\* settings modal, stored locally in their browser (`localStorage`).



\---



\## 7. Teacher Clone \& Setup Checklist (`v1.0.0-starter`)

If you cloned \*\*TeacherToolchest\*\* (`template-base` branch) to run your own independent Google AI + Firebase pipeline, complete these steps:



| Step | File / Location | What to Change |

| :--- | :--- | :--- |

| \*\*1. Create Firebase Project\*\* | \[Firebase Console](https://console.firebase.google.com/) | Create 1 project on the \*\*Spark (Free)\*\* plan (e.g., `smith-toolchest`). Enable \*\*Authentication (Google Sign-In)\*\*, \*\*Cloud Firestore\*\*, and \*\*Hosting\*\*. |

| \*\*2. Link Firebase CLI\*\* | `.firebaserc` | Replace `"default": "teachertoolchest"` with your new Firebase Project ID (e.g., `"default": "smith-toolchest"`). |

| \*\*3. Add Web Config\*\* | `src/config/firebaseConfig.ts` | Replace the upstream `teachertoolchest` credentials with your new Firebase Web App `firebaseConfig` object (`apiKey`, `authDomain`, `projectId`, etc.). |

| \*\*4. Customize Branding\*\* | `src/config/appConfig.ts` | Update `appName` (e.g., `"Mr. Smith's Toolchest"`) and optionally set `allowedEmailDomain` (e.g., `"myschool.org"`) to restrict sign-ins to colleagues. |

| \*\*5. Add Gemini Key\*\* | `.env.local` (or in-app UI) | Copy `.env.example` to `.env.local` and add `VITE\_GEMINI\_API\_KEY=...`, OR leave blank to enter keys in the browser UI. |

| \*\*6. Deploy Rules \& App\*\* | Local Terminal | Run `npm install`, `firebase deploy --only firestore:rules`, then `npm run build \&\& firebase deploy`. |

