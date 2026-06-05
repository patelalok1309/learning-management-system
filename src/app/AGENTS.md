# AGENTS.md

## Scope
These instructions apply to `src/app/**`, including route groups, pages, layouts, API routes, and route-local components.

## Routing Conventions
- This project uses Next.js App Router route groups:
  - `(auth)` for Clerk sign-in/sign-up pages.
  - `(dashboard)` for the authenticated dashboard and teacher tools.
  - `(course)` for student course consumption.
  - `api` for route handlers.
- Keep page files as `page.tsx`, layouts as `layout.tsx`, and API handlers as `route.ts`.
- Put route-owned components in an adjacent `_components` folder. Do not promote a component to `src/components` unless it is reused outside the route area.
- Dynamic route params use objects such as `{ params }: { params: { courseId: string } }`.

## Server Components
- Pages and layouts should stay server components unless they need client hooks or browser-only APIs.
- Fetch page data directly with Prisma via `db` where the data is page-specific.
- Use `redirect("/")` for unauthorized or missing resource fallbacks, following existing dashboard pages.
- For teacher pages, include `userId` in course lookups so users cannot access another teacher's resources.

## Client Components
- Add `"use client"` at the top for forms, sidebar toggles, drag-and-drop lists, rich text editor wrappers, UploadThing widgets, toasts, and router refresh interactions.
- Client mutations generally call local API routes with `axios`, show `react-hot-toast` messages, and then call `router.refresh()`.
- Validate forms with `zod` and `react-hook-form`; reuse `Form`, `FormField`, `FormItem`, `FormControl`, and `FormMessage` from `@/components/ui/form`.

## API Routes
- Import `db` from `@/lib/db`, auth helpers from `@clerk/nextjs/server`, and `NextResponse` from `next/server`.
- Each mutating route must verify auth before reading request data that could be used for writes.
- Scope course mutations by both `id` and `userId` when the course is teacher-owned.
- Return JSON with explicit statuses. Existing routes use `401` for unauthorized, `404` for missing resources, `400` for invalid state, and `500` for unexpected failures.
- Keep external side effects in sync with database changes, especially Mux asset deletion and Stripe checkout metadata.

## UI And Layout
- Use existing layout density: dashboard pages use `p-6`, responsive grids, small section headings, and route-local form panels.
- Prefer shared primitives from `@/components/ui` and shared components such as `Banner`, `IconBadge`, `CoursesList`, and `NavbarRoutes`.
- Do not add new global providers in route components; root providers belong in `src/app/layout.tsx`.
