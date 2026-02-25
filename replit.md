# Replit.md

## Overview

This is a **PC Hardware Bottleneck Calculator** — a full-stack web application that analyzes PC component configurations (CPU, GPU, RAM, resolution, purpose) and uses OpenAI's GPT-4o to determine bottleneck percentages and recommendations. It's styled as a dark-themed "gamer/hardware enthusiast" tool, similar to willitbottleneck.com. Users select their hardware from curated lists, submit for analysis, and see results with a circular gauge visualization. Recent queries are stored and displayed as community history.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (React + Vite)
- **Framework**: React 18 with TypeScript, bundled by Vite
- **Routing**: `wouter` for lightweight client-side routing (single page app with Home and 404)
- **State Management**: `@tanstack/react-query` for server state (data fetching, caching, mutations)
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives with Tailwind CSS
- **Animations**: `framer-motion` for page transitions, reveals, and gauge animations
- **Styling**: Tailwind CSS with CSS variables for theming; dark mode by default with neon purple/blue accent colors
- **Path aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`

### Backend (Express 5 + Node.js)
- **Framework**: Express 5 running on Node.js with TypeScript (via `tsx`)
- **API Design**: RESTful endpoints defined in `shared/routes.ts` as a typed contract object (`api`) with Zod schemas for input validation and response parsing — shared between client and server
- **AI Integration**: OpenAI GPT-4o (via Replit AI Integrations) for bottleneck analysis. Uses JSON mode (`response_format: { type: "json_object" }`) to get structured responses
- **Development**: Vite dev server runs as middleware in development; static files served in production from `dist/public`

### Database (PostgreSQL + Drizzle ORM)
- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Connection**: `node-postgres` (pg) Pool using `DATABASE_URL` environment variable
- **Schema location**: `shared/schema.ts` — shared between client and server
- **Main table**: `bottleneck_queries` — stores CPU, GPU, RAM, resolution, purpose, result percentage, bottleneck component, explanation, and timestamp
- **Additional tables**: `conversations` and `messages` tables exist in `shared/models/chat.ts` (from Replit AI integrations scaffolding, not core to the app)
- **Migrations**: Managed via `drizzle-kit push` (`npm run db:push`)
- **Storage pattern**: `server/storage.ts` implements an `IStorage` interface with `DatabaseStorage` class

### Shared Layer
- `shared/schema.ts` — Drizzle table definitions, Zod schemas, TypeScript types
- `shared/routes.ts` — API contract with paths, methods, input/output Zod schemas (used by both client hooks and server route handlers)

### Build Process
- **Dev**: `tsx server/index.ts` with Vite middleware for HMR
- **Production build**: `script/build.ts` — Vite builds the client to `dist/public`, esbuild bundles the server to `dist/index.cjs`. Server dependencies in an allowlist are bundled; others are external
- **Production start**: `node dist/index.cjs`

### Key Design Decisions
1. **Shared route contracts**: The `api` object in `shared/routes.ts` acts as a single source of truth for API paths and schemas, preventing client-server drift
2. **AI-powered analysis**: Rather than implementing a static formula, the app sends hardware configs to GPT-4o for nuanced bottleneck analysis with explanations
3. **Curated hardware lists**: CPU and GPU options are hardcoded in `client/src/data/hardware.ts` with benchmark scores and tier classifications, using combobox (Command) components for searchable selection
4. **Dark-first design**: The entire theme is built for dark mode with gaming aesthetics (neon colors, glass effects, gradient backgrounds)

## External Dependencies

### Required Services
- **PostgreSQL Database**: Connected via `DATABASE_URL` environment variable. Used for storing bottleneck query history. Must be provisioned before the app runs
- **OpenAI API** (via Replit AI Integrations): Uses `AI_INTEGRATIONS_OPENAI_API_KEY` and `AI_INTEGRATIONS_OPENAI_BASE_URL` environment variables. Powers the GPT-4o bottleneck analysis

### Replit Integrations (scaffolded but secondary)
The `server/replit_integrations/` and `client/replit_integrations/` directories contain pre-scaffolded modules for chat, audio/voice, image generation, and batch processing. These are not actively used by the bottleneck calculator but are available infrastructure:
- `chat/` — Conversation CRUD with OpenAI streaming
- `audio/` — Voice recording, playback, speech-to-text
- `image/` — Image generation via gpt-image-1
- `batch/` — Rate-limited batch processing utilities

### Key NPM Packages
- `drizzle-orm` + `drizzle-zod` + `drizzle-kit` — ORM and schema-to-Zod generation
- `openai` — OpenAI SDK
- `express` v5 — HTTP server
- `@tanstack/react-query` — Client data fetching
- `framer-motion` — Animations
- `wouter` — Client routing
- `zod` — Schema validation (shared between client and server)
- shadcn/ui ecosystem: Radix UI primitives, `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`