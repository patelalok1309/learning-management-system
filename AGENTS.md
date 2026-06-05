# AGENTS.md

## Project Overview
- Next.js 14 App Router LMS-style app using TypeScript, React 18, Tailwind CSS, shadcn/Radix UI primitives, Clerk auth, Prisma/PostgreSQL, Stripe checkout, Mux video, UploadThing uploads, and Zustand for small client state.
- Source lives under `src/`; App Router routes are in `src/app`; shared UI is in `src/components`; server-side query helpers are in `src/actions`; integration clients and utilities are in `src/lib`.
- Use the `@/*` alias for imports from `src/*`.

## Commands
- Install dependencies: `npm install`
- Run locally: `npm run dev`
- Build: `npm run build`
- Lint: `npm run lint`
- Generate Prisma client after schema changes: `npx prisma generate`
- Apply local schema changes during development: `npx prisma db push`
- Existing `postinstall` runs `prisma migrate deploy`; be aware this can touch the configured database.

## Coding Style
- Keep TypeScript and React patterns consistent with the existing code: function components, explicit props interfaces, and direct async server components for data-loading pages.
- Use semicolons and double quotes in new/edited TypeScript where practical. Existing files have uneven formatting; avoid broad formatting churn.
- Prefer named exports for reusable components and helpers, matching most local component files.
- Keep edits scoped. Do not rename files or normalize casing unless the change requires it.
- Use `@/components/ui/*` primitives and `cn()` from `@/lib/utils` rather than adding new UI libraries or custom class merge helpers.
- Use lucide-react icons for new UI icons unless an existing component already uses another icon package.

## App Architecture
- Server components are the default in `src/app`. Add `"use client"` only for components that use hooks, browser APIs, form state, drag-and-drop, toasts, router refresh/push, or client-only libraries.
- Route-local UI belongs in `_components` folders near the route that owns it. Cross-route UI belongs in `src/components`.
- Shared data reads that are not direct page-specific queries belong in `src/actions`, using the existing `get-*` naming style.
- API route handlers live in `src/app/api/**/route.ts` and should use `NextResponse.json(...)`.
- Database access goes through `db` from `@/lib/db`; do not instantiate `PrismaClient` elsewhere.

## Auth And Authorization
- Clerk middleware protects all non-public routes. Current public routes are sign-in, sign-up, and UploadThing.
- Server pages usually use `auth()` and redirect unauthenticated users with `redirect("/")`.
- API routes must independently verify auth with `auth()` or `currentUser()` and scope writes by `userId`.
- For teacher-owned resources, always include `userId` in Prisma `where` clauses when reading/updating/deleting courses.

## Data And Integrations
- Prisma models include Course, Chapter, Category, Attachment, MuxData, UserProgress, Purchase, and StripeCustomer.
- Preserve cascade relationships and unique constraints when changing schema or queries.
- Stripe checkout stores `courseId` and `userId` in session metadata. Keep currency behavior aligned with the current INR pricing helpers unless intentionally changing product requirements.
- Mux assets are deleted when course/chapter video data is removed. Preserve cleanup paths when editing delete flows.
- UploadThing Tailwind integration is enabled through `withUt(...)` in `tailwind.config.ts`.

## UI Patterns
- Styling is Tailwind-first with shadcn CSS variables from `src/app/globals.css`.
- Dark mode is class-based via `ThemeProvider`; new UI should use existing semantic classes where possible.
- Forms use `react-hook-form`, `zod`, `zodResolver`, local shadcn form primitives, `axios` for API calls, `react-hot-toast` for feedback, and `router.refresh()` after mutations.
- Keep form components small and route-local. Follow the edit/view toggle pattern used by course and chapter forms.
- For tables, use `@tanstack/react-table` with route-local column definitions as in teacher courses.

## Error Handling
- Existing API routes return `{ success: false, message: "..." }` with proper HTTP statuses for errors.
- Log server errors with bracketed route/action labels such as `[COURSES UPDATE]`.
- User-facing client errors generally use `toast.error("Something went wrong")`.

## Verification
- For behavior changes, run `npm run lint` at minimum.
- For route, data, auth, payment, upload, or video changes, also run `npm run build` when feasible.
- After Prisma schema changes, run `npx prisma generate`; use migrations or `db push` according to the user's workflow.
- There are no project tests configured, so validate changed flows manually when possible.

## Safety Notes
- Do not read, print, or commit `.env`, `credentials.txt`, database URLs, Clerk, Stripe, Mux, or UploadThing secrets.
- Do not edit `node_modules`, `.next`, `.vercel`, or generated Prisma client output.
- Be careful with destructive Prisma, Stripe, Mux, and file deletion operations. Prefer reversible local changes unless explicitly asked.
