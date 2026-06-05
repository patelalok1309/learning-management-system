# AGENTS.md

## Scope
These instructions apply to Prisma schema and migration files in `prisma/**`.

## Current Database
- Datasource is PostgreSQL through `DATABASE_URL`.
- Prisma client generator enables `fullTextSearchPostgres`.
- Main models: `Course`, `Attachment`, `Category`, `Chapter`, `MuxData`, `UserProgress`, `Purchase`, and `StripeCustomer`.

## Schema Guidelines
- Preserve ownership fields such as `Course.userId` and purchasing/progress uniqueness constraints.
- Keep indexes for foreign keys and query paths used by the app, especially `categoryId`, `courseId`, `chapterId`, and compound unique constraints.
- Use cascade deletes intentionally. Current course deletion cascades to attachments, chapters, purchases, progress, and Mux data through relations.
- When adding fields used for publish readiness, update both schema and completion checks in the relevant page/action/API code.

## Migration Workflow
- After editing `schema.prisma`, run `npx prisma generate`.
- Use `npx prisma db push` for local prototyping only if that matches the user's workflow.
- For committed database changes, create migrations instead of editing old migration SQL.
- Do not modify applied migrations unless the user explicitly asks for migration repair.

## Safety
- Never print or commit `DATABASE_URL`.
- Be careful with commands that deploy migrations or reset databases; ask before destructive database operations.
