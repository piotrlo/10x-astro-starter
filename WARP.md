# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Quick Start Commands

### Development
```bash
# Start dev server (runs on port 3000)
bun run dev

# Build for production
bun run build

# Preview production build
bun run preview
```

### Code Quality
```bash
# Run linter
bun run lint

# Fix linting issues automatically
bun run lint:fix

# Format code with Prettier
bun run format
```

### Testing
Note: This project currently does not have tests configured. When adding tests, follow the user's preference to write tests in `.test.ts` files using Bun's test runner (`bun test`).

## Tech Stack

- **Astro 5** - Server-side rendering framework (output mode: `server`)
- **React 19** - Interactive components only (no "use client" directives)
- **TypeScript 5** - Strict type checking enabled
- **Tailwind CSS 4** - Utility-first styling with CSS variables
- **Shadcn/ui** - Component library (New York style)
- **Node.js Adapter** - Standalone server mode
- **Supabase** - Backend/database (when configured)

## Project Architecture

### Rendering Strategy
This is a **server-side rendered (SSR)** Astro application. The configuration uses `output: "server"` with the Node.js adapter in standalone mode. API routes require `export const prerender = false`.

### Component Architecture
- **Astro components** (`.astro`) - Use for static content and layouts
- **React components** (`.tsx`) - Use ONLY when interactivity is needed
- Never use Next.js-specific directives like "use client"

### Directory Structure
```
src/
├── layouts/           # Astro layouts
├── pages/             # Astro pages (file-based routing)
│   └── api/           # API endpoints (uppercase handlers: GET, POST)
├── middleware/        # Astro middleware (index.ts)
├── components/        # UI components
│   ├── ui/            # Shadcn/ui components
│   └── hooks/         # Custom React hooks
├── lib/               # Services and utilities
│   ├── services/      # Business logic extracted from routes
│   └── utils.ts       # Helper functions (cn() for class merging)
├── db/                # Supabase clients and types
├── types.ts           # Shared types (Entities, DTOs)
└── assets/            # Internal static assets

public/                # Public assets
```

### Path Aliases
- `@/*` maps to `./src/*`
- Import example: `import { cn } from "@/lib/utils"`

## Development Patterns

### Error Handling
- Handle errors and edge cases at the beginning of functions
- Use early returns for error conditions
- Place happy path last for improved readability
- Avoid unnecessary else statements (use if-return pattern)
- Use guard clauses for preconditions

### Type Safety
- Use Zod for runtime validation, especially in API routes
- Use explicit type declarations
- Leverage TypeScript's strict mode

### State Management
- Extract logic into custom hooks in `src/components/hooks`
- Use `React.memo()` for expensive components that render often with same props
- Use `useCallback` for event handlers passed to child components to prevent unnecessary re-renders
- Use `useMemo` for expensive calculations to avoid recomputation on every render
- Use `useTransition` for non-urgent state updates to keep UI responsive
- Use `useOptimistic` for optimistic UI updates in forms
- Utilize `React.lazy()` and `Suspense` for code-splitting and performance optimization
- Implement `useId()` for generating unique IDs for accessibility attributes

### Styling with Tailwind
- Use arbitrary values with square brackets: `w-[123px]`
- Use responsive variants: `sm:`, `md:`, `lg:`
- Use state variants: `hover:`, `focus-visible:`, `active:`
- Use dark mode variant: `dark:`
- Organize custom styles with `@layer` directive
- Access theme values with `theme()` function
- Use `cn()` utility from `@/lib/utils` for conditional classes

### Accessibility
- Use ARIA landmarks (main, navigation, search)
- Set `aria-expanded` and `aria-controls` for expandable content
- Use `aria-live` regions for dynamic content
- Apply `aria-label` or `aria-labelledby` for non-visible labels
- Use `aria-describedby` for form descriptions
- Implement `aria-current` for navigation states
- Use `useId()` for generating unique IDs in React

### Astro-Specific
- Use View Transitions API for smooth page transitions
- Use Content Collections for type-safe content
- Use Server Endpoints for API routes (uppercase handlers: GET, POST)
- API routes need `export const prerender = false`
- Extract business logic to `src/lib/services`
- Use `Astro.cookies` for cookie management
- Use `import.meta.env` for environment variables
- Use image optimization with Astro Image integration

### Backend/Database (Supabase)
- Access Supabase via `context.locals.supabase` in routes
- Do NOT import `supabaseClient` directly
- Use `SupabaseClient` type from `src/db/supabase.client.ts`
- Validate all data with Zod schemas

#### Supabase File Structure
```
src/
├── db/
│   ├── supabase.client.ts    # Supabase client initialization
│   └── database.types.ts      # Auto-generated DB types
├── middleware/
│   └── index.ts               # Adds supabase to context.locals
└── env.d.ts                   # TypeScript environment definitions
```

#### Required Environment Variables
- `SUPABASE_URL` - Your Supabase project URL
- `SUPABASE_KEY` - Your Supabase anon/public key

#### Supabase Initialization Files

**src/db/supabase.client.ts:**
```typescript
import { createClient } from '@supabase/supabase-js';
import type { Database } from '../db/database.types.ts';

const supabaseUrl = import.meta.env.SUPABASE_URL;
const supabaseAnonKey = import.meta.env.SUPABASE_KEY;

export const supabaseClient = createClient<Database>(supabaseUrl, supabaseAnonKey);
```

**src/middleware/index.ts:**
```typescript
import { defineMiddleware } from 'astro:middleware';
import { supabaseClient } from '../db/supabase.client.ts';

export const onRequest = defineMiddleware((context, next) => {
  context.locals.supabase = supabaseClient;
  return next();
});
```

**src/env.d.ts:**
```typescript
/// <reference types="astro/client" />

import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from './db/database.types.ts';

declare global {
  namespace App {
    interface Locals {
      supabase: SupabaseClient<Database>;
    }
  }
}

interface ImportMetaEnv {
  readonly SUPABASE_URL: string;
  readonly SUPABASE_KEY: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

## Shadcn/ui Components

This project uses Shadcn/ui for UI components - beautifully designed, accessible components that can be customized.

### Configuration
- **Style variant:** "new-york"
- **Base color:** "neutral"
- **CSS variables:** Enabled for theming
- **Icon library:** Lucide React
- **RSC mode:** Disabled (we use React with Astro, not Next.js)

### Finding Installed Components
Components are available in `src/components/ui/` directory.

### Using Components
Import components using the configured `@/` alias:

```tsx
import { Button } from "@/components/ui/button"
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
```

Example usage:
```tsx
<Button variant="outline">Click me</Button>

<Card>
  <CardHeader>
    <CardTitle>Card Title</CardTitle>
    <CardDescription>Card Description</CardDescription>
  </CardHeader>
  <CardContent>
    <p>Card Content</p>
  </CardContent>
  <CardFooter>
    <p>Card Footer</p>
  </CardFooter>
</Card>
```

### Installing Additional Components
Many other components are available but not currently installed. Full list: https://ui.shadcn.com/r

To install a new component:
```bash
bun x shadcn@latest add [component-name]
```

Example:
```bash
bun x shadcn@latest add accordion
```

**Important:** Use `shadcn@latest`, not the deprecated `shadcn-ui@latest`

Popular components include:
- Accordion, Alert, AlertDialog, AspectRatio, Avatar
- Calendar, Checkbox, Collapsible, Command, ContextMenu
- DataTable, DatePicker, Dropdown Menu
- Form, Hover Card, Menubar, Navigation Menu
- Popover, Progress, Radio Group, ScrollArea
- Select, Separator, Sheet, Skeleton, Slider
- Switch, Table, Textarea, Sonner (Toast), Toggle, Tooltip

## Database Migrations (Supabase)

This project uses Supabase CLI for database migrations.

### Creating Migration Files

Migration files must be placed in `supabase/migrations/` and follow this naming convention:

**Format:** `YYYYMMDDHHmmss_short_description.sql`

Where:
- `YYYY` - Four-digit year (e.g., 2024)
- `MM` - Two-digit month (01-12)
- `DD` - Two-digit day (01-31)
- `HH` - Two-digit hour in 24h format (00-23)
- `mm` - Two-digit minute (00-59)
- `ss` - Two-digit second (00-59)
- Use UTC time

Example: `20240906123045_create_profiles.sql`

### SQL Guidelines for Migrations

**General Rules:**
- Write all SQL in lowercase
- Include header comment with migration metadata
- Add thorough comments explaining each step
- Add copious comments for destructive operations (truncate, drop, alter)

**Security (RLS):**
- ALWAYS enable Row Level Security (RLS) when creating tables
- Create granular RLS policies (one per operation: select, insert, update, delete)
- Create separate policies for each role (`anon` and `authenticated`)
- DO NOT combine policies even if functionality is the same
- For public tables, policies can return `true`
- Include comments explaining rationale of each policy

**Example Migration Structure:**
```sql
-- Migration: Create user profiles table
-- Purpose: Store user profile information
-- Affected: New table 'profiles'
-- Dependencies: auth.users

-- Create the profiles table
create table public.profiles (
  id uuid references auth.users on delete cascade not null primary key,
  username text unique,
  avatar_url text,
  created_at timestamptz default now() not null,
  updated_at timestamptz default now() not null
);

-- Enable RLS
alter table public.profiles enable row level security;

-- Policy: Allow public read access to profiles
create policy "profiles_select_anon"
  on public.profiles for select
  to anon
  using (true);

-- Policy: Allow authenticated users to read all profiles
create policy "profiles_select_authenticated"
  on public.profiles for select
  to authenticated
  using (true);

-- Policy: Allow users to insert their own profile
create policy "profiles_insert_authenticated"
  on public.profiles for insert
  to authenticated
  with check (auth.uid() = id);

-- Policy: Allow users to update their own profile
create policy "profiles_update_authenticated"
  on public.profiles for update
  to authenticated
  using (auth.uid() = id);
```

### Prerequisites Before Creating Supabase Integration
- `/supabase/config.toml` must exist
- `/src/db/database.types.ts` must exist with correct type definitions
- `@supabase/supabase-js` package must be installed

## Git Hooks

### Pre-commit Hook
Automatically runs `lint-staged` which:
- Runs `eslint --fix` on `*.{ts,tsx,astro}` files
- Runs `prettier --write` on `*.{json,css,md}` files

Note: Per user preferences, build verification and tests should be run on pre-push, not pre-commit.

## Linting Configuration

The project uses ESLint flat config with:
- TypeScript strict + stylistic rules
- React rules (with React Compiler plugin)
- React Hooks rules
- JSX accessibility rules (jsx-a11y)
- Astro plugin rules
- Prettier integration

Key lint rules:
- `no-console: "warn"` - Warnings for console statements
- `react-compiler/react-compiler: "error"` - React Compiler optimization enforcement
- `react/react-in-jsx-scope: "off"` - React import not required

## Code Documentation

- Use JSDoc comments (`/** ... */`) for functions, types, and exports
- Document function parameters and return values
- Write comments in English
- Avoid inline comments (`//`) in code body

## Coding Conventions

- Use functional components with hooks (no class components)
- Keep functions small and single-purpose
- Use meaningful, descriptive names
- Use declarative patterns (map/filter/reduce over loops)
- Prefer composition over inheritance
- Avoid premature optimization
- Keep state minimal, derive data when possible
- Use immutable patterns

## Important Notes

- Always use `bun` instead of `node` or `npm` per user preference
- Commit messages must be in English
- Server runs on port 3000 by default
- React version 19 is configured with new JSX transform
- Shadcn/ui is configured with "new-york" style and Lucide icons
- No RSC (React Server Components) - this is Astro + React, not Next.js
