# Summarist

**Live Demo(https://summarist-ahmed-ali.vercel.app/)**

A web application that gives readers the core ideas of best‑selling books in a few minutes. Users can browse curated lists, read short summaries, listen to audio versions, search the catalog, and unlock the full library through a Premium subscription.

Built as a clone / homage of the popular book‑summary apps, this project demonstrates a modern Next.js 16 stack wired to Firebase services and Stripe billing.

---

## What the product does

Summarist is a "books in minutes" reader and listener. The app is split into a public marketing site and a logged‑in app area.

**Public landing page (`/`)**
- Hero section with the value proposition ("Gain more knowledge in less time")
- Feature highlights (read or listen, recommendations, briefcasts)
- Statistics, member reviews, and download/rating numbers
- Footer with marketing links

**Authentication**
- Modal‑based sign in / sign up
- Email + password (Firebase Auth)
- Google sign‑in (Firebase popup)
- Anonymous "Login as a Guest" mode
- Auth state is shared globally through a custom `AuthProvider` React context

**For You (`/for-you`)**
- A "Selected just for you" featured book
- "Recommended for you" carousel
- "Suggested books" carousel
- Loading skeletons while data is fetched in parallel

**Book detail (`/book/[id]`)**
- Title, author, sub‑title, rating, duration, type, key‑idea count
- "Read" and "Listen" actions
- Book and author descriptions
- Premium gating — non‑premium users hitting a premium book are nudged to subscribe

**Audio player (`/player/[id]`)**
- Custom HTML5 audio player with play/pause, ±10 second skip, scrubber, and time display
- Loads track metadata for duration

**Search**
- Debounced search bar in the navbar (live results as you type)
- Calls a Firebase Cloud Function and renders a dropdown of matches with cover, title, author, and duration

**Subscriptions (`/choose-plan`)**
- Two plans: Premium Plus Yearly ($100/yr with a 7‑day free trial) and Premium Monthly ($10/mo)
- FAQ accordion
- Checkout is handled through Firestore + the official "Run Payments with Stripe" Firebase Extension — a checkout session document is written to Firestore and the extension responds with a Stripe Checkout URL the user is redirected to
- Premium status is read back from the user's `subscriptions` collection

**Settings (`/settings`)**
- Shows the current plan (Basic / Premium) and account email
- Upgrade button for Basic users
- Login prompt for unauthenticated visitors

**App shell**
- Responsive sidebar (For You, My Library, Highlights, Search, Settings, Help, Login/Logout)
- Mobile sidebar toggle with overlay
- Sidebar is only mounted on authenticated app routes; the marketing pages keep the standalone navbar

---

## Tech stack

**Framework & language**
- Next.js 16 (App Router, React Server Components, server/client split)
- React 19
- TypeScript 5

**Styling**
- Tailwind CSS 4 (via `@tailwindcss/postcss`)
- CSS Modules per‑component (`*.module.css`)
- `next/font` with the Geist and Geist Mono families

**State management**
- Redux Toolkit (`@reduxjs/toolkit`)
- React‑Redux (`react-redux`)
- Used for the global authentication modal slice (open / close, login vs. register mode)

**Authentication & data**
- Firebase Web SDK
  - Firebase Auth (email/password, Google, anonymous)
  - Cloud Firestore (user profiles, premium flag, Stripe customer/subscription docs)
  - Cloud Functions (book catalog API)

**Payments**
- Stripe, integrated through the Firebase "Run Payments with Stripe" extension
  - Client writes a `checkout_sessions` doc to Firestore
  - Extension creates a Stripe Checkout session and writes the URL back
  - `subscriptions` sub‑collection is the source of truth for the user's plan

**Backend / API**
- Firebase Cloud Functions hosted at `us-central1-summaristt.cloudfunctions.net`
  - `getBooks?status=recommended | suggested | selected`
  - `getBook?id=…`
  - `getBooksByAuthorOrTitle?search=…`

**UI helpers**
- `react-icons` (Ai, Bs, Bi, Ri icon sets)
- HTML5 `<audio>` for playback and metadata‑based duration

**Tooling**
- ESLint 9 with `eslint-config-next`
- PostCSS
- `dotenv` for local env loading

---

## Project structure

```
summarist-ahmed-ali/
├── app/
│   ├── page.tsx                  # Landing page
│   ├── layout.tsx                # Root layout, providers, nav, app shell
│   ├── globals.css
│   ├── for-you/page.tsx          # Authenticated home
│   ├── book/[id]/                # Book detail (server component) + subscription gate
│   ├── player/[id]/              # Audio player
│   ├── choose-plan/page.tsx      # Pricing & Stripe checkout
│   ├── settings/page.tsx         # Account & plan
│   ├── components/               # NavBar, Sidebar, AuthModal, BookCarousel, BookCard,
│   │                             #   BookDuration, SearchBar, AppShell, Footer, LoginButton
│   ├── components/styles/        # CSS modules
│   ├── Provideers/               # ReduxProvider + AuthenticationProvider (typo preserved)
│   └── Redux/                    # Redux store + AuthenticationModalSlice
├── firebase/
│   └── init.ts                   # Firebase app/auth/firestore initialisation
├── util/
│   ├── API.tsx                   # Book + search Cloud Function clients, types
│   └── Stripe.tsx                # Stripe checkout + premium status helpers
├── public/                       # Logo, landing image, login art, pricing art, Google icon
├── eslint.config.mjs
├── next.config.ts
├── postcss.config.mjs
├── tsconfig.json
└── package.json
```

---

## Getting started

Install dependencies and run the dev server:

```bash
npm install
npm run dev
```

Then open `http://localhost:3000`.

### Available scripts

| Script          | What it does                       |
| --------------- | ---------------------------------- |
| `npm run dev`   | Starts the Next.js dev server      |
| `npm run build` | Production build                   |
| `npm run start` | Runs the production build          |
| `npm run lint`  | Lints the project with ESLint      |

### Environment variables

Create a `.env.local` file in the project root with your Firebase and Stripe price IDs:

```env
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=

NEXT_PUBLIC_MONTHLY_PRICE_ID=
NEXT_PUBLIC_YEARLY_PRICE_ID=
```

Firebase needs Authentication (Email/Password, Google, Anonymous), Firestore, and the "Run Payments with Stripe" extension installed for the subscription flow to work end‑to‑end.

---

## Deployment

The project is a standard Next.js app and deploys cleanly to [Vercel](https://vercel.com/). Make sure the environment variables above are set in the deployment dashboard. Firebase services and the Stripe extension live in your Firebase project independently of the Next.js host.
