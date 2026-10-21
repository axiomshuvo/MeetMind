# MeetMind

> **This project is a full-stack SaaS platform designed for AI-powered video conferencing.**

MeetMind brings intelligent, real-time collaboration to every meeting. Built on a modern
Next.js + Express + MongoDB stack, it delivers crisp WebRTC video conferencing enriched with
AI-powered real-time transcription, smart meeting summaries, sentiment analysis, action-item
extraction, and personalized meeting insights — all in a secure, multi-tenant SaaS platform.

---

## Table of Contents

- [MeetMind](#meetmind)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Development](#development)
- [Testing](#testing)
- [Deployment](#deployment)
- [AI Features](#ai-features)
- [Security](#security)
- [License](#license)

---

## Features

### Core Video Conferencing
- **WebRTC video & audio calls** — peer-to-peer and SFU-based media streaming
- **Smart waiting rooms** with host moderation and participant approval
- **Screen sharing** with multiple display selection
- **Real-time chat** (text chat, reactions, and emoji) alongside video
- **Recording & playback** with automatic cloud storage

### AI-Powered Intelligence
- **Real-time transcription** with speaker diarization
- **Live captions** rendered on-screen during meetings
- **AI meeting summaries** — concise, structured recaps generated post-meeting
- **Sentiment analysis** — gauge meeting mood in real time
- **Action-item extraction** — automatically identify and assign tasks
- **Key topic detection** — surface the most discussed subjects

### SaaS Platform
- **Multi-tenant architecture** — isolated workspaces per organization
- **Subscription billing** with Stripe integration (tiers & add-ons)
- **User authentication** via email/password, OAuth (Google, GitHub), and SSO (SAML/OIDC)
- **Role-based access control** (owner, admin, member)
- **Meeting scheduling** with calendar sync (Google, Outlook, iCal)
- **Analytics dashboard** — usage metrics, engagement, and AI insights

---

## Tech Stack

| Layer         | Technology                                              |
|---------------|---------------------------------------------------------|
| **Frontend**  | [Next.js 16](https://nextjs.org) (App Router), React 19 |
| **Styling**   | Tailwind CSS v4, shadcn/ui, tw-animate-css              |
| **Rich UI**   | React day-picker, embla-carousel, react-resizable-panels |
| **Syntax**    | Shiki (GitHub Light & Dark themes)                      |
| **Backend**   | Next.js API routes + Node/Express microservice          |
| **Database**  | MongoDB (Mongoose ODM)                                  |
| **Real-time** | WebSockets (engine.io / socket.io)                      |
| **Media**     | WebRTC (peerjs / mediasoup SFU)                         |
| **AI**        | OpenAI / Anthropic SDK (transcription, summarization)   |
| **Auth**      | NextAuth.js (OAuth, credentials, SAML, OIDC)            |
| **Billing**   | Stripe Connect                                        |
| **Storage**   | AWS S3 / Vercel Blob (recordings, assets)               |
| **Deploy**    | Vercel (frontend) + Railway / Fly.io (backend)          |

---

## Architecture

```
                    ┌─────────────────────────────────────┐
                    │              Browser               │
                    │  Next.js App (React Server + Client) │
                    └──────┬──────────────────────┬───────┘
                           │                      │
         HTTPS (REST)      │   WebSocket (engine) │
                           ▼                      ▼
              ┌──────────────────┐    ┌──────────────────┐
              │  Next.js API     │    │  Express Worker  │
              │  Routes          │    │  (Signaling &    │
              │  (Auth, Billing, │◄──►│   Media Signaling │
              │   AI, DB proxy)  │    │   / SFU)         │
              └────────┬─────────┘    └────────┬─────────┘
                       │                       │
                       ▼                       ▼
              ┌──────────────────┐    ┌──────────────────┐
              │    MongoDB       │    │  Redis (Pub/Sub) │
              │  (multi-tenant)  │    │  (realtime bus)  │
              └──────────────────┘    └──────────────────┘
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        ┌──────────┐    ┌──────────────┐
        │  S3/Blob │    │  Stripe      │
        │ (storage)│    │ (billing)    │
        └──────────┘    └──────────────┘
                       │
                       ▼
              ┌──────────────────┐
              │  OpenAI /        │
              │  Anthropic       │
              │  (AI services)   │
              └──────────────────┘
```

**Data flow:**

1. A user joins a meeting via the Next.js frontend over HTTPS.
2. Next.js handles auth, billing, and AI orchestration through API routes.
3. The Express worker runs the WebRTC signaling server and an SFU for media routing.
4. Audio/video streams flow peer-to-SF­U-to-peer; AI services transcribe and analyze in real time.
5. MongoDB stores users, workspaces, meetings, recordings, and AI results.
6. Redis powers pub/sub for real-time events (chat, reactions, presence).

---

## Project Structure

```
meetmind/
├── src/
│   ├── app/                  # Next.js App Router
│   │   ├── api/              # Route handlers (auth, billing, AI, db)
│   │   ├── auth/             # Authentication pages (sign-in, callback)
│   │   ├── dashboard/        # Workspace / meeting dashboard
│   │   ├── meetings/         # Meeting lobby & room components
│   │   ├── billing/          # Stripe checkout & subscription pages
│   │   ├── layout.tsx        # Root layout with providers
│   │   └── page.tsx          # Landing / home page
│   ├── components/           # Shared React components
│   │   ├── ui/               # shadcn/ui primitives
│   │   ├── meeting/          # Video tile, chat sidebar, controls
│   │   ├── transcription/    # Live captions & speaker labels
│   │   └── dashboard/        # Charts (Recharts), analytics widgets
│   ├── hooks/                # Custom React hooks
│   │   ├── use-webrtc.ts     # WebRTC peer/SFU connection logic
│   │   ├── use-meeting.ts    # Meeting state management
│   │   └── use-mobile.ts     # Mobile breakpoint detection
│   ├── lib/                  # Utilities & service clients
│   │   ├── db.ts             # MongoDB connection helper
│   │   ├── ai.ts             # AI service wrapper (OpenAI/Anthropic)
│   │   ├── stripe.ts         # Stripe client helpers
│   │   ├── auth.ts           # NextAuth configuration
│   │   ├── highlighter.tsx   # Shiki code highlighting
│   │   └── utils.ts          # Shared utilities (cn, formatters)
│   └── styles/               # Global CSS & Tailwind base
├── server/                   # Express backend (signaling, SFU)
│   ├── src/
│   │   ├── signaling.ts      # WebSocket signaling server
│   │   ├── sfu.ts            # Media SFU (mediasoup)
│   │   └── routes/           # Express REST routes
│   └── tsconfig.json
├── public/                   # Static assets & favicons
├── next.config.ts            # Next.js + Turbopack config
├── tsconfig.json             # TypeScript config
├── components.json           # shadcn/ui configuration
├── tailwind.config.js        # Tailwind configuration (v4)
├── package.json
├── .env.example              # Required environment variables
└── README.md
```

---

## Prerequisites

- **Node.js** 20.18+ or 22.x
- **npm** 10+ (or pnpm/yarn)
- **MongoDB** 7.0+ (local or Atlas)
- **Redis** 7+ (local or managed)
- **Stripe CLI** (for local billing testing)
- **OpenAI / Anthropic API key** (for AI features)

---

## Getting Started

```bash
# 1. Clone the repository
git clone https://github.com/your-org/meetmind.git
cd meetmind

# 2. Install dependencies
npm install

# 3. Copy environment variables
cp .env.example .env.local

# 4. Configure your .env.local (see Configuration below)

# 5. Start the Express signaling/SFU server (if running locally)
npm run server:dev

# 6. Run the Next.js frontend in development
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the landing page.

---

## Configuration

Copy `.env.example` and fill in the values. All variables are documented in the template:

```bash
cp .env.example .env.local
```

| Variable                     | Description                                  | Required |
|-----------------------------|----------------------------------------------|----------|
| `NEXTAUTH_URL`              | Fully-qualified site URL                     | Yes      |
| `NEXTAUTH_SECRET`           | NextAuth secret for JWT signing              | Yes      |
| `DATABASE_URL`              | MongoDB connection string                    | Yes      |
| `REDIS_URL`                 | Redis connection string                      | Yes      |
| `OPENAI_API_KEY`            | OpenAI API key (transcription/summarization) | No       |
| `ANTHROPIC_API_KEY`         | Anthropic API key                            | No       |
| `STRIPE_SECRET_KEY`         | Stripe secret key                            | Yes      |
| `STRIPE_WEBHOOK_SECRET`     | Stripe webhook signing secret                | Yes      |
| `GOOGLE_CLIENT_ID`          | Google OAuth client id                       | No       |
| `GOOGLE_CLIENT_SECRET`      | Google OAuth client secret                   | No       |
| `AWS_ACCESS_KEY_ID`         | S3 access key (recordings storage)           | No       |
| `AWS_SECRET_ACCESS_KEY`     | S3 secret key                                | No       |
| `AWS_S3_BUCKET`             | S3 bucket name                               | No       |

---

## Development

```bash
# Type-check
npm run build         # or: npx tsc --noEmit

# Lint
npm run lint          # runs ESLint on the whole project

# Frontend dev server (hot reload)
npm run dev

# Backend dev server
npm run server:dev

# Run both (requires concurrently / turbo)
npm run dev:all
```

### Scripts

| Script            | Description                                      |
|-------------------|--------------------------------------------------|
| `dev`             | Start Next.js dev server                         |
| `dev:all`         | Start frontend + backend dev servers concurrently|
| `build`           | Production build (Next.js + backend)             |
| `start`           | Start the production server                      |
| `lint`            | Run ESLint                                       |
| `server:dev`      | Start the Express signaling/SFU server in watch  |
| `test`            | Run the test suite                              |

---

## Testing

The project uses [Vitest](https://vitest.dev) for unit tests and [Playwright](https://playwright.dev)
for end-to-end tests.

```bash
npm run test          # unit + integration tests
npm run test:e2e      # Playwright e2e tests
npm run test:watch    # watch mode
npm run test:coverage # coverage report
```

---

## Deployment

### Production build

```bash
npm run build
npm start
```

### Recommended platform setup

| Component      | Recommended host         |
|---------------|--------------------------|
| Frontend      | Vercel          |
| Backend API   | Railway / Fly.io         |
| MongoDB       | MongoDB Atlas            |
| Redis         | Redis Cloud / Upstash    |
| Storage       | AWS S3 / Vercel Blob     |
| SFU (signaling)| Railway / Fly.io       |

### Docker

A multi-stage `Dockerfile` and `docker-compose.yml` are provided for local
and self-hosted deployments.

```bash
docker compose up --build
```

---

## AI Features

MeetMind's AI layer is modular and can route between OpenAI and Anthropic
providers based on cost and capability:

| Feature                | Description                                           |
|------------------------|-------------------------------------------------------|
| **Transcription**      | Whisper / GPT-4o-transcribe real-time audio-to-text  |
| **Diarization**        | Speaker identification and labeling                   |
| **Summarization**      | GPT-4o-mini structured summaries with key points      |
| **Sentiment**          | Per-speaker sentiment analysis                        |
| **Action items**       | Extract tasks with owner assignment suggestions       |
| **Topic clustering**   | NLP topic modeling across meeting transcripts         |

AI results are stored alongside the meeting record and are accessible via the
dashboard and exported with meeting artifacts.

---

## Security

- All API routes and backend services enforce **authentication & authorization**
- **Rate limiting** on auth endpoints and AI-heavy routes
- **Input validation** with Zod schemas on every boundary
- **Secrets** loaded from environment variables — never committed
- **CORS** & **CSP** headers configured via Next.js headers
- Recordings and transcripts are **encrypted at rest**

See [SECURITY.md](SECURITY.md) for the full policy.

---

## License

This project is licensed under the [MIT License](LICENSE).
