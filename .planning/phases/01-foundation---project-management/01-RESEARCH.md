# Phase 1: Foundation - Project Management - Research

**Researched:** 2026-02-20
**Domain:** Full-stack web application development (Next.js, PostgreSQL, React)
**Confidence:** HIGH

## Summary

Phase 1 requires a foundation that supports simple CRUD operations now while accommodating future complexity (nested components, collaboration, AI integration, and visual diagramming). The research reveals a clear modern stack consensus for 2026.

**The Standard Approach:**
Next.js 15 App Router with React Server Components provides the optimal foundation. Use PostgreSQL with Supabase for managed database + built-in auth, Drizzle ORM for type-safe database access, TanStack Query for client-side data synchronization, and Zod for schema validation. This stack offers the fastest time-to-value for Phase 1 while avoiding technical debt that would require rewrites for later phases.

**Primary recommendation:** Start with Next.js 15 + Supabase + Drizzle ORM. Use Server Components for data fetching, Server Actions for mutations, and defer complex state management until later phases need it. Simple adjacency list data model (projects with foreign keys) scales naturally to nested hierarchies in Phase 2-3.

## Standard Stack

The established libraries/tools for this domain:

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 15.x | Full-stack React framework | App Router with Server Components is the 2026 default; Server Actions simplify mutations without API routes |
| React | 19.x | UI library | Required by Next.js; useActionState hook integrates with Server Actions |
| Supabase | Latest | Managed PostgreSQL + Auth + Storage | Combines database, auth, and Row Level Security in one service; eliminates auth implementation complexity |
| Drizzle ORM | Latest | TypeScript ORM | Code-first schema, tiny bundle size, edge-compatible, better for serverless than Prisma |
| TanStack Query | v5 | Server state management | Industry standard for data fetching/caching; handles background updates, optimistic updates, and cache invalidation |
| Zod | Latest | Schema validation | Type-safe validation for both client and server; integrates with React Hook Form and Server Actions |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| React Hook Form | Latest | Form state management | Complex forms with client-side validation (defer to Phase 2+ if simple forms suffice) |
| jose | Latest | JWT encryption/decryption | If implementing custom session management (not needed with Supabase Auth) |
| bcrypt | Latest | Password hashing | If implementing custom auth (not needed with Supabase Auth) |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Supabase | Neon + custom auth | More control, more implementation work, no built-in RLS |
| Drizzle ORM | Prisma | Better DX for some, larger bundle, heavier footprint for edge/serverless |
| Next.js | Vite + React SPA | Simpler for internal tools, but Server Components reduce client bundle and simplify data fetching |
| PostgreSQL | SQLite | Simpler for single-user, doesn't scale to multi-user collaboration (Phase 7) |

**Installation:**
```bash
# Core dependencies
npm install next@latest react@latest react-dom@latest
npm install @supabase/supabase-js @supabase/ssr
npm install drizzle-orm postgres
npm install @tanstack/react-query
npm install zod

# Dev dependencies
npm install -D drizzle-kit
npm install -D @types/node @types/react @types/react-dom
npm install -D typescript
```

## Architecture Patterns

### Recommended Project Structure
```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Route group for auth pages
│   │   ├── login/
│   │   └── signup/
│   ├── (dashboard)/       # Route group for authenticated pages
│   │   ├── layout.tsx     # Dashboard layout with navigation
│   │   └── projects/      # Project management routes
│   │       ├── page.tsx   # List projects
│   │       ├── new/
│   │       └── [id]/
│   │           ├── page.tsx    # View/edit project
│   │           └── delete/
│   ├── layout.tsx         # Root layout
│   └── page.tsx           # Landing page
├── lib/
│   ├── db/
│   │   ├── schema.ts      # Drizzle schema definitions
│   │   ├── index.ts       # Database client
│   │   └── migrations/    # Generated SQL migrations
│   ├── supabase/
│   │   ├── client.ts      # Client-side Supabase client
│   │   └── server.ts      # Server-side Supabase client
│   ├── validations/
│   │   └── project.ts     # Zod schemas for project CRUD
│   └── dal.ts             # Data Access Layer (auth verification)
├── actions/
│   └── projects.ts        # Server Actions for project CRUD
└── components/
    ├── ui/                # Reusable UI components
    └── projects/          # Project-specific components
```

### Pattern 1: Server Component Data Fetching
**What:** Fetch data directly in Server Components without client-side state
**When to use:** Default pattern for displaying data (project lists, project details)
**Example:**
```typescript
// Source: https://nextjs.org/docs/app/getting-started/server-and-client-components
// app/(dashboard)/projects/page.tsx
import { getProjects } from '@/lib/db/queries'

export default async function ProjectsPage() {
  const projects = await getProjects() // Direct database query

  return (
    <div>
      <h1>Projects</h1>
      <ProjectList projects={projects} />
    </div>
  )
}
```

### Pattern 2: Server Actions for Mutations
**What:** Use Server Actions for create/update/delete operations with form validation
**When to use:** All data mutations (create project, edit project, delete project)
**Example:**
```typescript
// Source: https://nextjs.org/docs/app/guides/authentication
// actions/projects.ts
'use server'
import { z } from 'zod'
import { db } from '@/lib/db'
import { projects } from '@/lib/db/schema'
import { verifySession } from '@/lib/dal'

const projectSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  description: z.string().optional(),
  client_name: z.string().min(1, 'Client name required'),
})

export async function createProject(formData: FormData) {
  const session = await verifySession()

  const validated = projectSchema.safeParse({
    name: formData.get('name'),
    description: formData.get('description'),
    client_name: formData.get('client_name'),
  })

  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors }
  }

  const result = await db.insert(projects).values({
    ...validated.data,
    user_id: session.userId,
  })

  revalidatePath('/projects')
  redirect(`/projects/${result[0].id}`)
}
```

### Pattern 3: Row Level Security (RLS) for Data Isolation
**What:** Use Supabase RLS policies to enforce user-level access control at database layer
**When to use:** All tables containing user-specific data
**Example:**
```sql
-- Source: https://supabase.com/docs/guides/database/postgres/row-level-security
-- Enable RLS on projects table
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own projects
CREATE POLICY "Users can view own projects"
  ON projects
  FOR SELECT
  USING (auth.uid() = user_id);

-- Policy: Users can insert their own projects
CREATE POLICY "Users can create projects"
  ON projects
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Policy: Users can update their own projects
CREATE POLICY "Users can update own projects"
  ON projects
  FOR UPDATE
  USING (auth.uid() = user_id);

-- Policy: Users can delete their own projects
CREATE POLICY "Users can delete own projects"
  ON projects
  FOR DELETE
  USING (auth.uid() = user_id);
```

### Pattern 4: TanStack Query for Client-Side Mutations with Optimistic Updates
**What:** Use TanStack Query in Client Components for mutations that need optimistic UI updates
**When to use:** Defer to Phase 2+ unless immediate UI feedback needed before server response
**Example:**
```typescript
// Source: https://tanstack.com/query/latest
// components/projects/DeleteButton.tsx
'use client'
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { deleteProject } from '@/actions/projects'

export function DeleteButton({ projectId }: { projectId: string }) {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: async () => deleteProject(projectId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['projects'] })
    },
  })

  return (
    <button onClick={() => mutation.mutate()}>
      {mutation.isPending ? 'Deleting...' : 'Delete'}
    </button>
  )
}
```

### Pattern 5: Data Access Layer (DAL) for Auth Verification
**What:** Centralize authentication checks in a single verifySession function
**When to use:** Every Server Action and data fetch that requires authentication
**Example:**
```typescript
// Source: https://nextjs.org/docs/app/guides/authentication
// lib/dal.ts
import 'server-only'
import { cache } from 'react'
import { createClient } from '@/lib/supabase/server'

export const verifySession = cache(async () => {
  const supabase = await createClient()
  const { data: { user }, error } = await supabase.auth.getUser()

  if (!user || error) {
    redirect('/login')
  }

  return { isAuth: true, userId: user.id }
})
```

### Anti-Patterns to Avoid
- **Avoid:** Using middleware/proxy for database-level auth checks (CVE-2025-29927 vulnerability, performance issues)
- **Do instead:** Use Supabase RLS policies and verify session in Data Access Layer
- **Avoid:** Fetching data in Client Components with useEffect
- **Do instead:** Use Server Components for initial data fetching, TanStack Query only for client-side mutations
- **Avoid:** Creating custom API routes for simple CRUD
- **Do instead:** Use Server Actions directly from forms and Client Components
- **Avoid:** Soft deletes in Phase 1 unless explicitly required
- **Do instead:** Hard delete with audit logs if deletion tracking needed (simpler, no cascade complexity)

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Authentication | Custom JWT + password hashing + session management | Supabase Auth | Handles OAuth, email/password, magic links, session management, token refresh, RLS integration out of box |
| Form validation | Manual error state management | Zod + Server Actions | Type-safe validation on both client and server; single source of truth for validation rules |
| Data fetching/caching | Custom fetch wrappers with useState | Server Components + TanStack Query | Server Components eliminate waterfalls; TanStack Query handles cache invalidation, background refetching, optimistic updates |
| Database migrations | Hand-written SQL files | Drizzle Kit | Generates migrations from TypeScript schema; tracks migration state; supports push for dev, generate for prod |
| Session management | Custom cookie encryption | Supabase Auth or jose | Secure defaults, handles edge cases (expiration, refresh, HttpOnly cookies) |
| Access control | Conditional rendering in React | Row Level Security (RLS) | Database-level enforcement; can't be bypassed by API manipulation; works across all data access methods |

**Key insight:** In 2026, the full-stack framework ecosystem (Next.js + Supabase + Drizzle + TanStack Query) has matured to the point where custom solutions for these problems are almost always premature optimization. Use the integrated stack for Phase 1; optimize specific bottlenecks in later phases if needed.

## Common Pitfalls

### Pitfall 1: Over-engineering Phase 1 with Complex State Management
**What goes wrong:** Adding Redux/Zustand, complex client-side caching, or premature abstractions for data that doesn't change frequently
**Why it happens:** Developers anticipate future complexity and build for Phase 7 collaboration features in Phase 1
**How to avoid:** Start with Server Components for data fetching. Add TanStack Query only when you need optimistic updates or background refetching. Defer global state management to Phase 5+ when AI features require client-side orchestration
**Warning signs:** More than 10% of components are Client Components; installing state management libraries before needing them; complex data flow diagrams for simple CRUD

### Pitfall 2: Ignoring Database-Level Security (RLS)
**What goes wrong:** Relying solely on application-layer checks (middleware, API auth) allows data exposure via direct database access or forgotten endpoints
**Why it happens:** Developers from REST API background assume application controls access; unfamiliar with database-level policies
**How to avoid:** Enable Row Level Security on all tables from Day 1. Write RLS policies using auth.uid() before writing application code. Test by attempting direct database queries as different users
**Warning signs:** Tables created without RLS enabled; auth checks only in middleware; no database policies in migration files

### Pitfall 3: Treating Server Actions as Insecure by Default
**What goes wrong:** Creating unnecessary API routes because Server Actions "feel" too simple or insecure
**Why it happens:** Misconception that Server Actions bypass security; unfamiliarity with server-only code guarantees
**How to avoid:** Understand that Server Actions run server-side always, never shipped to client. Use 'server-only' package to prevent accidental client imports. Verify session in every Server Action
**Warning signs:** Creating /api routes for simple mutations; duplicating Server Action logic in API routes; importing database clients in Client Components

### Pitfall 4: Cascading Delete Complexity with Soft Deletes
**What goes wrong:** Implementing soft deletes (deleted_at flag) in Phase 1 creates cascade complexity and query pollution (every query needs WHERE deleted_at IS NULL)
**Why it happens:** Assumption that users will want to recover deleted projects; premature feature planning
**How to avoid:** Use hard deletes in Phase 1. If audit trail needed, create separate audit_logs table. Add soft delete in later phase if user feedback demands it
**Warning signs:** Adding deleted_at to tables before users request recovery features; CASCADE triggers written for soft deletes; complex query filters in every SELECT

### Pitfall 5: Choosing Wrong Migration Strategy (Push vs Generate)
**What goes wrong:** Using drizzle-kit push (schema push) in production causes data loss; using generate in development slows iteration
**Why it happens:** Confusion between development and production workflows
**How to avoid:** Development: use drizzle-kit push for rapid iteration (drops/recreates tables). Production: use drizzle-kit generate to create SQL migrations reviewed before deploy
**Warning signs:** Generated migration files in local dev; schema push commands in CI/CD; lack of migration review process

### Pitfall 6: Authentication Check Location (Middleware vs DAL)
**What goes wrong:** Putting database-level auth checks in Next.js middleware causes performance issues and false security (CVE-2025-29927)
**Why it happens:** Middleware seems like natural place for "before request" checks
**How to avoid:** Use middleware only for optimistic cookie-based redirects. Put real auth checks in Data Access Layer functions that decrypt/verify sessions before data access
**Warning signs:** Database queries in middleware; auth.getUser() calls in middleware; relying on middleware as sole auth mechanism

### Pitfall 7: Data Modeling for Current Phase Only
**What goes wrong:** Creating flat projects table structure that requires costly refactoring when Phase 2 adds components, Phase 3 adds dependencies
**Why it happens:** "YAGNI" taken too far; not considering one schema version ahead
**How to avoid:** Use simple adjacency list pattern from Day 1 (projects table with id and parent_id for future nesting). Costs nothing now, supports natural expansion later
**Warning signs:** No foreign key patterns in schema; assumption that projects will never have nested structures; plans to "rebuild database in Phase 2"

## Code Examples

Verified patterns from official sources:

### Database Schema with Drizzle
```typescript
// Source: https://orm.drizzle.team/docs/sql-schema-declaration
// lib/db/schema.ts
import { pgTable, uuid, text, timestamp } from 'drizzle-orm/pg-core'

export const projects = pgTable('projects', {
  id: uuid('id').defaultRandom().primaryKey(),
  user_id: uuid('user_id').notNull(), // References auth.users (Supabase)
  name: text('name').notNull(),
  description: text('description'),
  client_name: text('client_name').notNull(),
  client_email: text('client_email'),
  created_at: timestamp('created_at').defaultNow().notNull(),
  updated_at: timestamp('updated_at').defaultNow().notNull(),
})

// Future-ready: Add parent_id for nested structures in Phase 2+
// parent_id: uuid('parent_id').references(() => projects.id, { onDelete: 'cascade' }),
```

### Supabase Client Configuration
```typescript
// Source: https://supabase.com/docs/guides/getting-started/quickstarts/nextjs
// lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          )
        },
      },
    }
  )
}
```

### TanStack Query Setup
```typescript
// Source: https://tanstack.com/query/latest
// app/layout.tsx (for Client Component usage in later phases)
'use client'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useState } from 'react'

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            refetchOnWindowFocus: false,
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}
```

### Complete Server Action with Validation
```typescript
// Source: https://nextjs.org/docs/app/guides/authentication
// actions/projects.ts
'use server'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { db } from '@/lib/db'
import { projects } from '@/lib/db/schema'
import { verifySession } from '@/lib/dal'
import { projectCreateSchema } from '@/lib/validations/project'
import { eq } from 'drizzle-orm'

export async function createProject(prevState: any, formData: FormData) {
  const session = await verifySession()

  const validated = projectCreateSchema.safeParse({
    name: formData.get('name'),
    description: formData.get('description'),
    client_name: formData.get('client_name'),
  })

  if (!validated.success) {
    return {
      errors: validated.error.flatten().fieldErrors,
    }
  }

  const [project] = await db
    .insert(projects)
    .values({
      ...validated.data,
      user_id: session.userId,
    })
    .returning({ id: projects.id })

  revalidatePath('/projects')
  redirect(`/projects/${project.id}`)
}

export async function updateProject(
  projectId: string,
  prevState: any,
  formData: FormData
) {
  const session = await verifySession()

  const validated = projectCreateSchema.safeParse({
    name: formData.get('name'),
    description: formData.get('description'),
    client_name: formData.get('client_name'),
  })

  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors }
  }

  // RLS ensures user can only update their own projects
  await db
    .update(projects)
    .set({
      ...validated.data,
      updated_at: new Date(),
    })
    .where(eq(projects.id, projectId))

  revalidatePath(`/projects/${projectId}`)
  return { success: true }
}

export async function deleteProject(projectId: string) {
  const session = await verifySession()

  // RLS ensures user can only delete their own projects
  await db.delete(projects).where(eq(projects.id, projectId))

  revalidatePath('/projects')
  redirect('/projects')
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Pages Router | App Router with Server Components | Next.js 13 (2022), stable in 14-15 | Eliminates client-side data fetching waterfalls; reduces bundle size 30%+ |
| Prisma ORM | Drizzle ORM | 2023-2024 shift | Smaller bundles, better edge/serverless support, code-first schema |
| NextAuth.js | Auth.js v5 / Supabase Auth | 2024-2025 | NextAuth rebranded to Auth.js; Supabase Auth provides simpler integrated solution |
| Custom API routes | Server Actions | Next.js 13 App Router | Simplifies mutations; no need for API route boilerplate |
| Client-side form validation only | Zod on both client and server | 2023+ standard | Type-safe validation; prevents bypass via API manipulation |
| Middleware for auth | Data Access Layer pattern | CVE-2025-29927 disclosure (March 2025) | Security vulnerability fix; performance improvement |

**Deprecated/outdated:**
- **Pages Router:** Still supported but App Router is default for new projects in 2026
- **getServerSideProps/getStaticProps:** Replaced by Server Component async/await and fetch with caching
- **NextAuth.js (pre-v5):** API routes pattern replaced by App Router integration in Auth.js v5
- **Relying solely on middleware for auth:** CVE-2025-29927 vulnerability; use DAL pattern instead

## Open Questions

Things that couldn't be fully resolved:

1. **Supabase vs Neon for PostgreSQL hosting**
   - What we know: Supabase provides auth + database + storage integrated; Neon offers better database performance/features
   - What's unclear: Performance comparison at scale for this specific use case (project management with future AI features)
   - Recommendation: Start with Supabase for integrated auth + RLS; evaluate Neon in Phase 5+ if database performance becomes bottleneck

2. **When to introduce TanStack Query**
   - What we know: Server Components handle initial data fetching; TanStack Query adds optimistic updates and background sync
   - What's unclear: Whether Phase 1 CRUD needs optimistic updates or if simple Server Actions + revalidatePath suffice
   - Recommendation: Defer TanStack Query to Phase 2+ unless user testing shows need for instant feedback on mutations

3. **Hard delete vs soft delete for projects**
   - What we know: Hard delete simpler; soft delete adds recovery features but cascade complexity
   - What's unclear: User expectation for project recovery (no user research yet)
   - Recommendation: Start with hard delete; add deleted_at column in later phase if users request recovery feature

4. **React Hook Form necessity**
   - What we know: Server Actions work with plain forms; React Hook Form adds client-side validation UX
   - What's unclear: Whether basic forms with Server Action validation provide acceptable UX
   - Recommendation: Start with plain forms + Server Actions; add React Hook Form in Phase 2 if validation UX feedback is poor

## Don't Build in Phase 1

Features to explicitly defer to later phases:

### Defer to Phase 2 (Component Design)
- Complex nested data models (phases → components → dependencies)
- Drag-and-drop reordering
- Rich text editing for descriptions
- Component libraries or templates

### Defer to Phase 3 (Planning & Sequencing)
- Dependency graphs
- Timeline visualization
- Gantt charts
- Critical path analysis

### Defer to Phase 4 (Governance)
- Multi-environment management
- Naming convention enforcement
- License tracking
- Environment variable management

### Defer to Phase 5 (Templates & AI)
- AI-suggested components
- Template library
- Natural language input
- Smart defaults based on project type

### Defer to Phase 6 (Branding)
- PDF export with custom branding
- Client-facing presentation mode
- Custom color schemes
- Logo uploads

### Defer to Phase 7 (Collaboration)
- Real-time multi-user editing
- Conflict resolution
- Version control/branching
- Comments and mentions
- Permissions (beyond owner-only)

### Defer to Phase 8 (Export)
- Word/PDF documentation generation
- Architecture diagram export
- API documentation
- Deployment scripts generation

**Phase 1 Success Criteria (Reminder):**
1. User can create a new project with name, description, and client info
2. User can edit existing project details
3. User can delete a project they created
4. Projects persist across sessions

**Build only what's needed for these criteria.** Every additional feature delays learning and adds technical debt.

## Sources

### Primary (HIGH confidence)
- [Next.js Authentication Documentation](https://nextjs.org/docs/app/guides/authentication) - Official authentication patterns
- [Next.js Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components) - Official component patterns
- [Supabase Row Level Security Documentation](https://supabase.com/docs/guides/database/postgres/row-level-security) - RLS patterns
- [TanStack Query Documentation](https://tanstack.com/query/latest) - Server state management
- [Drizzle ORM Documentation](https://orm.drizzle.team/docs/migrations) - Migration patterns
- [Next.js Project Structure](https://nextjs.org/docs/app/getting-started/project-structure) - Official folder structure

### Secondary (MEDIUM confidence)
- [Prisma vs Drizzle: Which ORM for Your Next.js Project? (2026)](https://designrevision.com/blog/prisma-vs-drizzle) - ORM comparison
- [React vs. Next.js: A 2026 Developer's Guide](https://sam-solutions.com/blog/react-vs-nextjs/) - Framework selection
- [Supabase Row Level Security: Complete Guide (2026)](https://designrevision.com/blog/supabase-row-level-security) - RLS patterns
- [Mastering CRUD Operations with Next.js and TanStack Query](https://medium.com/@sergey-bocharov/organizing-crud-operations-with-next-js-and-tanstack-query-63d53e539608) - CRUD patterns
- [Handling Forms in Next.js with React Hook Form, Zod, and Server Actions](https://medium.com/@techwithtwin/handling-forms-in-next-js-with-react-hook-form-zod-and-server-actions-e148d4dc6dc1) - Form validation
- [Modeling Hierarchical Tree Data in PostgreSQL](https://leonardqmarcq.com/posts/modeling-hierarchical-tree-data) - Database schema patterns
- [Building authentication in Next.js App Router: The complete guide for 2026](https://workos.com/blog/nextjs-app-router-authentication-guide-2026) - Auth patterns
- [React Server Components in Production: Best Practices for 2026](https://www.growin.com/blog/react-server-components/) - RSC patterns

### Tertiary (LOW confidence - WebSearch only)
- [From MVP to Scale-Up: The "Technical Debt" You Should Actually Keep](https://www.pragmaticcoders.com/blog/from-mvp-to-scale-up-the-technical-debt-you-should-actually-keep) - MVP strategy
- [Soft delete vs hard delete](https://appmaster.io/blog/soft-delete-vs-hard-delete) - Delete patterns

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Verified with official documentation (Next.js, Supabase, Drizzle, TanStack Query all have official docs confirming patterns)
- Architecture: HIGH - Official Next.js documentation provides App Router structure; multiple authoritative sources confirm Server Components + Server Actions pattern
- Pitfalls: MEDIUM-HIGH - CVE-2025-29927 verified from official sources; soft delete cascading issues verified in multiple sources; other pitfalls based on community consensus and official documentation warnings

**Research date:** 2026-02-20
**Valid until:** 2026-04-20 (60 days - stable ecosystem, but Next.js releases frequently)

**Key assumptions:**
- User has Node.js 18+ installed
- User will deploy to Vercel or similar platform supporting Next.js App Router
- PostgreSQL database acceptable (vs SQLite or NoSQL)
- Authentication required from Phase 1 (not deferred)
- Single-tenant data isolation acceptable for Phase 1 (no team/organization concept yet)
