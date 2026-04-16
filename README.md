# SensAI — AI Career Coach

SensAI is a full-stack AI-powered career coaching platform that helps job seekers prepare for interviews, craft ATS-friendly resumes and cover letters, and stay informed about their industry's salary ranges, demand, and trends. It's built with Next.js 15 (App Router), Prisma + PostgreSQL, Clerk for auth, Google's Gemini for AI, Inngest for scheduled background jobs, and a Tailwind + shadcn/ui front-end.



---

## Features

- **AI-Powered Career Guidance** — personalized advice tailored to the user's industry, sub-industry, skills, and experience captured during onboarding.
- **Interview Preparation** — generate role-specific technical/behavioral quizzes, take mock interviews, review per-question answers, and receive AI-generated improvement tips. Performance is scored and tracked over time.
- **Smart Resume Builder** — markdown-based resume editor with AI assistance, ATS scoring, and PDF export (via `html2pdf.js`).
- **AI Cover Letters** — generate tailored cover letters per company/role from a job description; drafts are saved and editable.
- **Industry Insights Dashboard** — live view of salary ranges, growth rate, demand level, top skills, market outlook, and key trends for the user's industry. Insights are auto-regenerated weekly by a cron-driven Inngest function.
- **Authentication & Onboarding** — Clerk-based sign-in / sign-up with a protected onboarding flow that routes users into the main app.
- **Light / Dark Mode** — theme toggle via `next-themes`.

---

## Tech Stack

| Layer            | Tech                                                                 |
| ---------------- | -------------------------------------------------------------------- |
| Framework        | [Next.js 15](https://nextjs.org/) (App Router, Turbopack dev)        |
| Language         | JavaScript / JSX, React 19                                           |
| Styling          | Tailwind CSS, `tailwindcss-animate`, shadcn/ui + Radix primitives    |
| Auth             | [Clerk](https://clerk.com/)                                          |
| Database         | PostgreSQL (Neon recommended) via [Prisma](https://www.prisma.io/)   |
| AI               | [Google Generative AI](https://ai.google.dev/) (Gemini 1.5 Flash)    |
| Background Jobs  | [Inngest](https://www.inngest.com/) (weekly industry-insights cron)  |
| Forms/Validation | `react-hook-form` + `zod`                                            |
| Charts           | `recharts`                                                           |
| Markdown/PDF     | `@uiw/react-md-editor`, `react-markdown`, `html2pdf.js`              |

---

## Project Structure

```
SensAI/
├── actions/                # Server actions (resume, cover letter, interview, dashboard, user)
├── app/
│   ├── (auth)/             # Clerk sign-in / sign-up route group
│   ├── (main)/             # Protected app (dashboard, interview, resume, cover letter, onboarding)
│   ├── api/inngest/        # Inngest webhook endpoint
│   ├── layout.js           # Root layout (Clerk + ThemeProvider)
│   └── page.js             # Marketing/landing page
├── components/             # Shared UI (header, hero, shadcn/ui primitives)
├── data/                   # Static content (features, FAQs, industries, testimonials)
├── hooks/                  # Reusable hooks (e.g., use-fetch)
├── lib/
│   ├── checkUser.js        # Syncs Clerk user → DB
│   ├── prisma.js           # Prisma client singleton
│   └── inngest/            # Inngest client + scheduled functions
├── prisma/
│   ├── schema.prisma       # User, Assessment, Resume, CoverLetter, IndustryInsight
│   └── migrations/
├── middleware.js           # Clerk auth + protected route matcher
└── next.config.mjs
```

### Data Model (high level)

- **User** — Clerk-linked profile (industry, bio, experience, skills).
- **Assessment** — quiz results per user (score, questions, category, AI improvement tip).
- **Resume** — one markdown resume per user with optional ATS score + feedback.
- **CoverLetter** — many per user; linked to a company/role and job description.
- **IndustryInsight** — one per industry; cached salary ranges, growth rate, demand level, top skills, outlook, and trends, refreshed weekly.

---

## Getting Started

### Prerequisites

- Node.js 18.18+ (Node 20 LTS recommended)
- A PostgreSQL database (e.g., a free [Neon](https://neon.tech/) project)
- A [Clerk](https://clerk.com/) application (publishable + secret keys)
- A [Google AI Studio](https://aistudio.google.com/) API key for Gemini

### 1. Clone and install

```bash
git clone https://github.com/Tran0408/SensAI.git
cd SensAI
npm install
```

`postinstall` automatically runs `prisma generate`.

### 2. Configure environment variables

Create a `.env` file in the repo root:

```env
# Database
DATABASE_URL=postgresql://user:password@host/db

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/onboarding
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding

# Gemini
GEMINI_API_KEY=
```

### 3. Run database migrations

```bash
npx prisma migrate deploy
# or, during development:
npx prisma migrate dev
```

### 4. Start the dev server

```bash
npm run dev
```

The app runs on [http://localhost:3000](http://localhost:3000) with Turbopack.

### 5. (Optional) Run the Inngest dev server

Industry insights are generated by a scheduled Inngest function (`Generate Industry Insights`, `0 0 * * 0`). To exercise it locally:

```bash
npx inngest-cli@latest dev
```

Point it at `http://localhost:3000/api/inngest`.

---

## Available Scripts

| Script          | Description                                    |
| --------------- | ---------------------------------------------- |
| `npm run dev`   | Start the Next.js dev server with Turbopack.   |
| `npm run build` | Production build.                              |
| `npm run start` | Start the production server (after `build`).   |
| `npm run lint`  | Run ESLint with the Next.js config.            |

---

## Protected Routes

The following route groups require a signed-in Clerk user (see [middleware.js](middleware.js)):

- `/dashboard`
- `/resume`
- `/interview`
- `/ai-cover-letter`
- `/onboarding`

Unauthenticated requests are redirected to Clerk's sign-in.

---

## How the AI Pieces Fit Together

- **On-demand** generations (quiz questions, improvement tips, resume feedback, cover letters) are made via server actions in [actions/](actions/), calling Gemini through `@google/generative-ai`.
- **Scheduled** industry insights are produced by [lib/inngest/function.js](lib/inngest/function.js), which runs every Sunday at 00:00 UTC, iterates over every industry in the DB, prompts Gemini for a strict-JSON response, and writes the result back to `IndustryInsight`.

---

## Deployment

The app is optimized for Vercel:

1. Import the repo into Vercel.
2. Set all environment variables listed above.
3. Ensure `DATABASE_URL` points to a production PostgreSQL instance.
4. Configure Inngest for production (add your Inngest event/signing keys as env vars and register the `/api/inngest` endpoint in the Inngest dashboard).

---

## License

No license file is provided in this repository. All rights reserved by the original authors unless a license is added.

---

## Credits

Originally built following the "Full Stack AI Career Coach" tutorial — see the [walkthrough video](https://youtu.be/UbXpRv5ApKA).
