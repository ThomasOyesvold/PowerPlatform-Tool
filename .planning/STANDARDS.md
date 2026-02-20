# Development Standards: Power Platform Roadmap Planner

**Created:** 2026-02-20
**Purpose:** Enforce audit, traceability, and documentation standards across all code

---

## Code Documentation Standards

### JSDoc Comments (AUDIT-01)

**Required for:**
- All exported functions
- All React components
- All Server Actions
- All database schema definitions
- All utility functions

**Format:**
```typescript
/**
 * Creates a new project in the database with RLS enforcement.
 *
 * @param formData - Form data containing project name, description, and client info
 * @returns Object with errors (if validation failed) or redirects to project detail page
 *
 * @reasoning
 * Uses Server Actions pattern instead of API routes to reduce boilerplate
 * and leverage Next.js 15 form integration. RLS policies enforce user-level
 * access control at database layer, preventing unauthorized access even if
 * application-layer checks are bypassed.
 *
 * @traceability PROJ-01, AUDIT-01
 */
export async function createProject(formData: FormData) {
  // Implementation
}
```

**Key sections:**
- `@param` - Parameter description
- `@returns` - Return value description
- `@reasoning` - WHY this approach was chosen (critical for future maintenance)
- `@traceability` - Links to requirements this satisfies

### Inline Comments (AUDIT-06)

**Required for:**
- Non-obvious logic
- Performance optimizations
- Security decisions
- Workarounds or hacks
- Complex algorithms

**Format:**
```typescript
// REASONING: Use adjacency list pattern (parent_id) instead of flat structure
// to support future nested components (Phase 2) without schema migration.
// Costs nothing now, prevents breaking change later.
export const projects = pgTable('projects', {
  id: uuid('id').defaultRandom().primaryKey(),
  parent_id: uuid('parent_id').references(() => projects.id, {
    onDelete: 'cascade'
  }),
  // ... other fields
})

// SECURITY: RLS policies are applied at database level, not application level.
// This prevents data exposure via direct database queries or forgotten endpoints.
// See: https://supabase.com/docs/guides/database/postgres/row-level-security
```

**Prefix guidelines:**
- `REASONING:` - Design decisions
- `SECURITY:` - Security-related choices
- `PERFORMANCE:` - Optimization rationale
- `WORKAROUND:` - Temporary fixes (include ticket/issue number)
- `TODO:` - Future improvements (include target phase)

---

## Database Audit Logging (AUDIT-02)

### Audit Log Table

All database mutations (INSERT, UPDATE, DELETE) are logged:

```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  user_id UUID NOT NULL REFERENCES auth.users(id),
  action TEXT NOT NULL, -- 'create', 'update', 'delete'
  table_name TEXT NOT NULL,
  record_id UUID NOT NULL,
  old_data JSONB, -- Previous state (NULL for create)
  new_data JSONB, -- New state (NULL for delete)
  ip_address INET,
  user_agent TEXT
);

CREATE INDEX idx_audit_log_user ON audit_log(user_id);
CREATE INDEX idx_audit_log_timestamp ON audit_log(timestamp DESC);
CREATE INDEX idx_audit_log_table_record ON audit_log(table_name, record_id);
```

### Server Action Pattern

All Server Actions that mutate data must log:

```typescript
'use server'

import { logAuditEvent } from '@/lib/audit'

export async function createProject(formData: FormData) {
  const session = await verifySession()

  // Validate
  const validated = projectSchema.safeParse({...})
  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors }
  }

  // Insert
  const [project] = await db
    .insert(projects)
    .values({...validated.data, user_id: session.userId})
    .returning()

  // AUDIT: Log creation event
  await logAuditEvent({
    userId: session.userId,
    action: 'create',
    tableName: 'projects',
    recordId: project.id,
    newData: project,
  })

  revalidatePath('/projects')
  redirect(`/projects/${project.id}`)
}
```

---

## Changelog Maintenance (AUDIT-03)

### CHANGELOG.md Format

Track all features, fixes, and changes:

```markdown
# Changelog

All notable changes to Power Platform Roadmap Planner will be documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

## [Unreleased]

### Added
- PROJ-01: Create new projects with name, description, client info
- PROJ-02: Edit existing project details
- PROJ-03: Delete projects
- AUDIT-01: JSDoc comments on all exported functions
- AUDIT-02: Database audit logging for all mutations

### Changed
(None yet)

### Deprecated
(None yet)

### Removed
(None yet)

### Fixed
(None yet)

### Security
- RLS policies enforce user-level data isolation
- Server Actions validate all inputs with Zod schemas

## [0.1.0] - 2026-02-20

### Added
- Initial Next.js 15 scaffolding with App Router
- Supabase integration for auth and database
- Drizzle ORM with type-safe schema

[Unreleased]: https://github.com/ThomasOyesvold/PowerPlatform-Tool/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/ThomasOyesvold/PowerPlatform-Tool/releases/tag/v0.1.0
```

**Update rules:**
- Add entries immediately when features are implemented (not at release time)
- Use requirement IDs (PROJ-01, AUDIT-02) for traceability
- Group by category (Added, Changed, Fixed, Security)
- Update after each plan execution completes

---

## Error Handling & Logging (AUDIT-04)

### Error Logging Service

Capture full context for debugging:

```typescript
// lib/logger.ts
export interface ErrorContext {
  userId?: string
  action: string
  component: string
  metadata?: Record<string, any>
}

export async function logError(
  error: Error,
  context: ErrorContext
): Promise<void> {
  const errorLog = {
    timestamp: new Date().toISOString(),
    message: error.message,
    stack: error.stack,
    name: error.name,
    userId: context.userId,
    action: context.action,
    component: context.component,
    metadata: context.metadata,
    environment: process.env.NODE_ENV,
  }

  // Log to console in development
  if (process.env.NODE_ENV === 'development') {
    console.error('ERROR:', errorLog)
  }

  // Store in database for production
  await db.insert(errorLogs).values(errorLog)

  // TODO Phase 6: Send to error tracking service (Sentry, LogRocket)
}
```

### Usage in Server Actions

```typescript
export async function createProject(formData: FormData) {
  try {
    const session = await verifySession()
    // ... implementation
  } catch (error) {
    await logError(error as Error, {
      userId: session?.userId,
      action: 'createProject',
      component: 'actions/projects.ts',
      metadata: {
        formData: Object.fromEntries(formData),
      },
    })

    return {
      errors: { _form: ['Failed to create project. Please try again.'] },
    }
  }
}
```

---

## Git Commit Standards (AUDIT-05)

### Conventional Commits Format

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature (maps to MINOR version)
- `fix` - Bug fix (maps to PATCH version)
- `docs` - Documentation only
- `style` - Code style/formatting (no logic change)
- `refactor` - Code restructuring (no behavior change)
- `perf` - Performance improvement
- `test` - Adding tests
- `chore` - Build process, dependencies, tooling

**Scope:**
- Phase or feature area: `auth`, `projects`, `ui`, `db`, `api`

**Subject:**
- Imperative mood: "add feature" not "added feature"
- No period at end
- Max 72 characters

**Body:**
- Explain WHAT and WHY (not HOW - code shows that)
- Reference requirements: `Implements PROJ-01, AUDIT-05`
- Explain reasoning for non-obvious decisions

**Footer:**
- Breaking changes: `BREAKING CHANGE: <description>`
- Issue references: `Closes #123`

**Examples:**

```
feat(auth): add email/password authentication

Implements signup, login, and logout Server Actions using Supabase Auth.
Uses Server Components for initial auth checks and middleware for route
protection.

Reasoning: Supabase Auth chosen over custom implementation to avoid
reinventing session management, token refresh, and OAuth integrations.
RLS policies provide database-level security even if middleware is bypassed.

Implements: PROJ-01, AUDIT-01, AUDIT-05
```

```
fix(projects): prevent duplicate project creation on double-click

Added debouncing to create project button to prevent multiple submissions
when users double-click the submit button.

Bug discovered during UAT testing - users would create 2-3 identical projects
by accident. TanStack Query mutation with `onMutate` prevents multiple
in-flight requests.

Fixes: #45
```

---

## Enforcement During Execution

**Plan execution agents must:**

1. **Before writing code:**
   - Check if function/component needs JSDoc (if exported → yes)
   - Check if logic is non-obvious (if yes → inline comment)

2. **After writing code:**
   - Verify JSDoc includes `@reasoning` and `@traceability`
   - Verify inline comments explain WHY not WHAT

3. **When mutating database:**
   - Add audit logging call after successful mutation
   - Include old_data for updates, new_data for creates

4. **When creating commits:**
   - Use conventional commit format
   - Include detailed body explaining reasoning
   - Reference requirement IDs

5. **After plan completes:**
   - Update CHANGELOG.md with features/changes
   - Verify all audit requirements satisfied

**Verification checklist (added to plan verification steps):**

```bash
# Check for JSDoc on exports
grep -r "^export" src/ | wc -l
grep -r "@reasoning" src/ | wc -l  # Should match or exceed

# Check for audit logging
grep -r "logAuditEvent" src/actions/ | wc -l  # Should exist for all mutations

# Check commit format
git log -1 --pretty=%B | head -1 | grep -E "^(feat|fix|docs|style|refactor|perf|test|chore)\("

# Check CHANGELOG updated
grep "## \[Unreleased\]" CHANGELOG.md -A 20 | grep -E "(PROJ|AUDIT|PLAN|COMP|GOV|TMPL|BRAND|PRES|COLLAB|EXPORT)"
```

---

## Benefits

**For future debugging:**
- Full audit trail of all data changes (who, when, what)
- Error logs with complete context (stack trace, user, action, metadata)
- Git history explains WHY decisions were made

**For maintenance:**
- JSDoc shows reasoning behind patterns ("why Server Actions not API routes?")
- Inline comments explain non-obvious logic
- Changelog provides human-readable history

**For onboarding:**
- New developers can understand decisions through code comments
- Traceability links code to requirements
- Conventional commits make git history scannable

---

*Standards enforced from Phase 1 onward. All code must comply before merging.*
