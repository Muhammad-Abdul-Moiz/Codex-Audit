# Codex Audit

Know what an AI coding agent changed before you merge it: Codex Audit gives engineering leads a fast risk review of a diff or agent session.

## The Problem

AI coding agents can now change hundreds of lines in minutes, but the human review process has not become faster or clearer. A reviewer can miss a hardcoded credential, a deleted test, or a risky dependency while trying to keep up. Codex Audit addresses that gap at the moment teams are adopting agent-generated pull requests in production.

## The Solution

Codex Audit accepts a git diff or AI coding-agent transcript and turns it into a review dashboard. The dashboard combines a 0-100 risk score, an executive summary, severity-ranked flags, a plain-English decision log, and an interactive reviewer checklist so a technical lead knows where to focus before merging.

## How It Works

1. Paste a git diff or session transcript into the input page, or select **Load Demo Data** for an instant review.
2. The Next.js API route validates the request and sends recognizable code changes to GPT-5.6 through the OpenAI Responses API.
3. GPT-5.6 is prompted to act as a senior code reviewer: detect secrets, dependency and security changes, deleted tests, large deletions, and infrastructure risk, then explain the likely intent of each major change in plain English.
4. The response is constrained to a strict JSON schema and rendered as the risk dashboard, with Markdown export available client-side.

## Tech Stack

- Next.js 15 App Router and TypeScript
- React 19
- Tailwind CSS 4 with a small custom stylesheet for the dark SaaS UI
- OpenAI GPT-5.6 via the OpenAI Responses API
- Zod for request and structured-report validation
- Lucide React for interface icons
- In-memory processing only; no database or authentication layer

## Setup Instructions

Requirements: Node.js 20.9 or newer and npm 10 or newer. The checked-in package metadata targets npm 11.16.0.

1. Clone the repository. Replace `<repo-url>` with the URL for this submission:

   ```bash
   git clone <repo-url>
   ```

2. Enter the project folder:

   ```bash
   cd codex-audit
   ```

3. Install dependencies:

   ```bash
   npm install
   ```

4. Copy the environment template and add your API key:

   macOS/Linux:

   ```bash
   cp .env.example .env.local
   ```

   Windows PowerShell:

   ```powershell
   Copy-Item .env.example .env.local
   ```

   Then set:

   ```dotenv
   OPENAI_API_KEY=your_openai_api_key_here
   ```

5. Start the development server:

   ```bash
   npm run dev
   ```

6. Open [http://localhost:3000](http://localhost:3000). The app binds to local IPv4 port 3000; if `localhost` resolves incorrectly on your machine, use [http://127.0.0.1:3000](http://127.0.0.1:3000).

## 7. Sample Data / How to Test It

On the input page, click **Load Demo Data**, then click **Analyze changes**. The demo diff intentionally contains three review signals:

- A fake hardcoded live-style API key in `src/lib/billing.ts`
- A deleted `src/lib/billing.test.ts` test file
- A new `fast-jwt` dependency in `package.json`

With a working OpenAI API key, expect a **High** risk result (normally 70+ out of 100) with Security or Secrets, Testing, and Dependencies flags. The exact score and wording can vary because GPT-5.6 is evaluating the evidence, but all three signals should be represented in the flags or checklist.

## 8. Testing Without Rebuilding (Developer Tools requirement)

No account, database, migration, or seed step is required. After `npm install`, the app runs immediately with only `OPENAI_API_KEY` required for live analysis; the input page, demo data, validation, and non-code fallback can be exercised without any other service. The app processes requests in memory and exposes no persistence UI.

For a production build check, stop the dev server before running `npm run build`; do not run both commands against the same `.next` directory at the same time.

## 9. How Codex Accelerated This Build

Fill in the bracketed placeholders with specifics from your actual Codex session before submitting.

- **Next.js and API scaffolding** - Codex created the App Router structure, Tailwind setup, typed API route, and shared report schema in one pass - this removed repetitive project wiring and gave the judge-readable architecture a clean starting point.
- **product UI and interaction design** - Codex shaped the input flow, loading states, risk dashboard, timeline, checklist, and Markdown export as one coherent dark-mode experience - this shortened the gap between functional code and a demo-ready product surface.
- **GPT-5.6 integration** - Codex wrote the senior-reviewer prompt, strict structured-output contract, Zod validation, and server-side error handling - this made the model's role explicit and kept malformed output out of the UI.
- **stress-test fix** - Codex tested empty, 2,000-line, non-code, and over-limit inputs, then hardened malformed-response parsing and friendly network errors - this prevented raw parser messages and blank screens during judging.
- **install/startup recovery** - Codex traced npm's `Link.matches` crash to stale pnpm-generated `node_modules`, regenerated an npm lockfile, and resolved a stale port-3000 process - this turned a setup failure into a repeatable `npm install` -> `npm run dev` path.

## 10. Architecture Notes

The MVP deliberately has no database: a review is a single in-memory request, and the browser keeps checklist state locally for that session. `/api/analyze` rejects empty or oversized input, recognizes non-code text deterministically, and sends only recognizable changes to GPT-5.6. GPT-5.6 returns strict structured JSON through the Responses API, and the same Zod contract validates the server response before React renders it. Network failures, malformed responses, missing keys, and model failures become concise user-facing messages rather than stack traces.

## 11. License

This repository is released under the MIT License. See [`LICENSE`](LICENSE) for the full text.
