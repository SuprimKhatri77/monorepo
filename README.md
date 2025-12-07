# Monorepo Setup Guide

A comprehensive guide for setting up a PNPM workspace monorepo with Next.js, Express API, NX build system, and development tooling.

## Prerequisites

- Node.js (v18 or higher recommended)
- PNPM package manager
- Git (optional)

## Initial Setup

### 1. Initialize PNPM Project

```bash
pnpm init
```

### 2. Configure Root Package

Edit `package.json` in the root directory:

- Remove `"main": "index.js"`
- Add `"private": true`

This prevents accidental publication of the root package.

### 3. Configure PNPM Workspace

Create `pnpm-workspace.yaml` in the root:

```yaml
packages:
  - "apps/*"
  - "libs/*"
```

This defines the monorepo structure with separate directories for applications and shared libraries.

### 4. Create Directory Structure

```bash
mkdir apps libs
```

### 5. Configure Git Ignore

Create `.gitignore` in the root:

```gitignore
# Dependencies
node_modules/
.pnp
.pnp.*
.yarn/*
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/versions

# PNPM
.pnpm-debug.log*
pnpm-lock.yaml

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*
*.log

# Build outputs
dist/
**/dist/
build/
.next/
out/

# NX cache
.nx/cache
.nx/workspace-data

# Environment variables
.env
.env.*
!.env.example

# Misc
.DS_Store
*.pem

# TypeScript
*.tsbuildinfo
next-env.d.ts

# Vercel
.vercel

# IDE
.vscode/
.idea/

# Additional NX files
.cursor/rules/nx-rules.mdc
.github/instructions/nx.instructions.md
```

### 6. Initialize Git Repository (Optional)

```bash
git init
```

## Application Setup

### 7. Create Next.js Application

```bash
cd apps/
mkdir web
cd web
pnpm create next-app@latest .
```

Follow the Next.js setup prompts according to your preferences.

### 8. Create Express API

```bash
cd apps/
mkdir api
cd api
pnpm init
```

Install dependencies:

```bash
# Production dependencies
pnpm add express dotenv cors

# Development dependencies
pnpm add -D @types/cors @types/express tsx typescript
```

### 9. Configure API Scripts

Add to `apps/api/package.json`:

```json
"scripts": {
  "build": "rm -rf ./dist && tsc",
  "tsc": "tsc --noEmit",
  "dev": "tsx watch src/index.ts",
  "start": "node dist/src/index.js"
}
```

- `build`: Cleans the dist folder and compiles TypeScript
- `tsc`: Type-checks without emitting files
- `dev`: Runs the server in watch mode for development
- `start`: Runs the compiled production server

## TypeScript Configuration

### 10. Install Root TypeScript

```bash
cd ../../  # Return to root
pnpm add -D typescript -w
```

The `-w` flag installs to the workspace root.

### 11. Create Base TypeScript Config

Create `tsconfig.json` in root:

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "CommonJS",
    "strict": true,
    "skipLibCheck": true
  },
  "exclude": ["node_modules", "dist"]
}
```

This serves as the base configuration for all packages.

### 12. Configure API TypeScript

Create `tsconfig.json` in `apps/api`:

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": ".",
    "esModuleInterop": true
  }
}
```

This extends the base config with API-specific settings.

### 13. Add Type Checking to Next.js

Add to `apps/web/package.json` scripts:

```json
"tsc": "tsc --noEmit"
```

## NX Build System Integration

### 14. Initialize NX

Run from the root directory:

```bash
pnpm dlx nx@latest init
```

#### Setup Wizard Choices:

**a) Setup Type**

- Choose: `Guided` (use arrow keys and press Enter)

**b) Script Execution Order**

- Select: `dev` and `build`
- These scripts require dependencies to be built first
- Use arrow keys to navigate, Space to select, Enter to confirm

**c) Cacheable Scripts**

- Select: `build` and `tsc`
- These produce deterministic outputs and can be cached
- Use arrow keys to navigate, Space to select, Enter to confirm

**d) Build Script Outputs**

- Enter: `.next, dist, build`
- These are the directories where build artifacts are generated

**e) TSC Script Outputs**

- Leave blank and press Enter
- Type checking doesn't produce cacheable outputs

**f) Plugin Selection**

- Press Enter to skip (or select plugins as needed)
- Plugins can be added later based on requirements

**g) AI Agent Setup**

- Press Enter to skip
- Optional AI-powered development tools

**h) CI/CD Features**

- Press Enter to choose later
- Optional remote caching and self-healing CI features

## Code Formatting

### 15. Install Prettier

```bash
pnpm add -D prettier -w
```

### 16. Add Formatting Scripts

Add to root `package.json`:

```json
"scripts": {
  "format": "prettier \"{apps,libs}/**/*.{ts,tsx,js,json}\" --ignore-path .gitignore",
  "format:check": "pnpm format --check",
  "format:write": "pnpm format --write"
}
```

- `format:check`: Validates code formatting
- `format:write`: Automatically fixes formatting issues

### 17. Configure Prettier

Create `.prettierrc` in root:

```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "semi": false
}
```

## Monorepo Scripts

### 18. Add Workspace Scripts

Add to root `package.json`:

```json
"scripts": {
  "build": "pnpm exec nx run-many --target=build",
  "tsc": "pnpm exec nx run-many --target=tsc",
  "lint": "pnpm exec nx run-many --target=lint",
  "validate": "pnpm format:write && pnpm tsc && pnpm lint && pnpm build"
}
```

- `build`: Builds all projects in the workspace
- `tsc`: Type-checks all projects
- `lint`: Lints all projects
- `validate`: Runs complete validation pipeline (format, type-check, lint, build)

## Next Steps

Your monorepo is now configured! You can:

1. Create your Express API entry point at `apps/api/src/index.ts`
2. Start developing with `pnpm --filter web dev` for Next.js
3. Start the API with `pnpm --filter api dev`
4. Run `pnpm validate` before committing to ensure code quality

## Useful Commands

```bash
# Install dependencies for all packages
pnpm install

# Run a command in a specific package
pnpm --filter <package-name> <command>

# Add a dependency to a specific package
pnpm --filter <package-name> add <dependency>

# Run validate before pushing
pnpm validate
```

## Git Hooks and Code Quality

### 19. Setup Husky for Git Hooks

```bash
pnpm dlx husky init && pnpm install
```

This creates a `.husky` folder with pre-configured git hooks. Open `.husky/pre-commit` and replace `pnpm test` or `npm test` with:

```bash
pnpm validate
```

This ensures code quality checks run before every commit.

### 20. Install Lint-Staged

```bash
pnpm add -D lint-staged -w
```

Lint-staged runs commands only on staged files, improving performance.

### 21. Configure Lint-Staged

Create `lint-staged.config.js` in root:

```javascript
module.exports = {
  "*.{ts,tsx,js,json}": (filename) => ["pnpm validate"],
};
```

### 22. Update Pre-Commit Hook

Edit `.husky/pre-commit`:

- Remove `pnpm validate`
- Add `pnpm lint-staged`

This runs validation only on staged files instead of the entire codebase.

## tRPC Server Setup

### 23. Initialize tRPC Server Library

```bash
cd libs
mkdir trpc-server
cd trpc-server
pnpm init
pnpm add @trpc/server
```

Update `libs/trpc-server/package.json`:

- Change `"name"` to `"@repo/trpc-server"` (or use your root package name, e.g., `"@foundation-trpc/trpc-server"`)

**Note:** Use consistent naming across your monorepo. If your root is named `@foundation-trpc`, use `@foundation-trpc/package-name` for all packages.

### 24. Configure TypeScript for tRPC Server

Create `tsconfig.json` in `libs/trpc-server`:

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": ".",
    "jsx": "react-jsx",
    "esModuleInterop": true,
    "declaration": true,
    "skipLibCheck": true,
    "declarationMap": true,
    "composite": true
  },
  "include": ["src/**/*"]
}
```

Key settings:

- `declaration`: Generates `.d.ts` files for TypeScript consumers
- `composite`: Enables project references for better build performance
- `declarationMap`: Enables IDE navigation to source files

### 25. Configure Package Entry Points

Add to `libs/trpc-server/package.json`:

```json
{
  "main": "./dist/src/index.js",
  "types": "./dist/src/index.d.ts",
  "scripts": {
    "build": "rm -rf ./dist && tsc",
    "dev": "tsc --watch"
  }
}
```

- `main`: Entry point for JavaScript imports
- `types`: Entry point for TypeScript type definitions
- `dev`: Watch mode for development

### 26. Create tRPC Server Entry Point

Create `libs/trpc-server/src/index.ts`:

```typescript
import { appRouter } from "./routers/index";
import { createExpressMiddleware } from "@trpc/server/adapters/express";
import { createContextFactory } from "./context";
import { auth } from "@foundation-trpc/auth";
import { db } from "@foundation-trpc/db";

const createContext = createContextFactory({
  auth,
  db,
});

export const trpcExpress = createExpressMiddleware({
  router: appRouter,
  createContext,
});

// Re-export for convenient imports
export { appRouter } from "./routers/index";
export type { AppRouter } from "./routers/index";
```

This allows clean imports: `import { appRouter } from '@foundation-trpc/trpc-server'`

### 27. Create Context Factory

Create `libs/trpc-server/src/context.ts`:

```typescript
import { CreateExpressContextOptions } from "@trpc/server/adapters/express";
import type { DATABASE } from "@foundation-trpc/db";
import type { AUTH } from "@foundation-trpc/auth";
import { AppContext } from "@foundation-trpc/types";

export const createContextFactory = (deps: { db: DATABASE; auth: AUTH }) => {
  return async ({
    req,
    res,
  }: CreateExpressContextOptions): Promise<AppContext> => {
    console.log("Context factory triggered");

    return {
      req,
      res,
      db: deps.db,
      auth: deps.auth,
    };
  };
};

export type Context = AppContext;
```

**Installing Workspace Dependencies:**

To add other workspace packages (e.g., `@foundation-trpc/db`):

```bash
pnpm add @foundation-trpc/db --workspace
```

**Important:** Always build the library first before installing it elsewhere:

```bash
cd libs/db
pnpm build
cd ../trpc-server
pnpm add @foundation-trpc/db --workspace
```

**Example Library Package Structure:**

```json
{
  "name": "@foundation-trpc/db",
  "version": "1.0.0",
  "main": "./dist/src/index.js",
  "types": "./dist/src/index.d.ts",
  "scripts": {
    "build": "rm -rf ./dist && tsc"
  }
}
```

### 28. Configure tRPC Instance

Create `libs/trpc-server/src/trpc.ts`:

```typescript
import { initTRPC, TRPCError } from "@trpc/server";
import { Context } from "./context";
import { ProtectedContext } from "@foundation-trpc/types";
import { fromNodeHeaders } from "@foundation-trpc/auth";

export const t = initTRPC.context<Context>().create();

export const router = t.router as typeof t.router;
export const publicProcedure = t.procedure as typeof t.procedure;

// Protected procedure with authentication middleware
export const protectedProcedure = t.procedure.use(async ({ ctx, next }) => {
  // Session validation using your auth provider (e.g., Better Auth)
  const headers = fromNodeHeaders(ctx.req.headers);
  const result = await ctx.auth.api.getSession({
    headers,
  });

  if (!result || !result.session) {
    throw new TRPCError({
      code: "UNAUTHORIZED",
      message: "User not logged in.",
    });
  }

  return next({
    ctx: {
      ...ctx,
      session: {
        ...result.session,
        user: result.user,
      },
    } as ProtectedContext,
  });
}) as typeof t.procedure;
```

**Note:** Type errors on `t` are expected and can be safely ignored.

### 29. Create Route Handlers

Create `libs/trpc-server/src/routers/test.router.ts`:

```typescript
import { protectedProcedure, publicProcedure, router } from "../trpc";

export const testRoutes = router({
  test: publicProcedure
    .input(/* your input schema */)
    .output(/* output response schema */)
    .query(async ({ input, ctx }) => {
      // Access context (db, auth, req, res) and input here
      return { message: "Hello world" };
    }),

  // Add more routes as needed
});
```

### 30. Merge All Routers

Create `libs/trpc-server/src/routers/index.ts`:

```typescript
import { router } from "../trpc";
import { testRoutes } from "./test.router";
import { inferRouterOutputs } from "@trpc/server";

export const appRouter = router({
  test: testRoutes,
  // Add more route namespaces here
});

export type AppRouter = typeof appRouter;
export type AppRouterType = inferRouterOutputs<AppRouter>;
```

This creates a unified router with type-safe outputs for use across your application.

## Integrating tRPC with Express API

### 31. Install tRPC Server in API

First, build the tRPC server library:

```bash
cd libs/trpc-server
pnpm build
```

Then install it in the API:

```bash
cd ../../apps/api
pnpm add @foundation-trpc/trpc-server --workspace
```

Add to `apps/api/src/index.ts`:

```typescript
import { trpcExpress } from "@foundation-trpc/trpc-server";

// Configure CORS if not already done
app.use("/trpc", trpcExpress);
```

**Important:** If you have `db` and `auth` libraries requiring environment variables, ensure all secrets are added to `apps/api/.env` to avoid runtime errors.

## tRPC Client Setup

### 32. Initialize tRPC Client Library

```bash
cd libs
mkdir trpc-client
cd trpc-client
pnpm init
pnpm add @foundation-trpc/trpc-server --workspace
pnpm add @trpc/client @trpc/tanstack-react-query @tanstack/react-query @trpc/react-query
```

Configure the same `tsconfig.json` and package.json properties as the tRPC server (see steps 24-25).

### 33. Create Server Component Client

Install peer dependencies:

```bash
pnpm add react --save-peer
pnpm add -D @types/react
pnpm add next --save-peer
```

Create `libs/trpc-client/src/index.ts`:

```typescript
import { createTRPCProxyClient, httpBatchLink } from "@trpc/client";
import type { AppRouter } from "@foundation-trpc/trpc-server";
import type { createTRPCProxyClient as CreateTRPCProxyClient } from "@trpc/client";
import { cookies } from "next/headers";

export const trpc: ReturnType<typeof CreateTRPCProxyClient<AppRouter>> =
  createTRPCProxyClient<AppRouter>({
    links: [
      httpBatchLink({
        url: `${process.env.NEXT_PUBLIC_API_URL}/trpc`,
        async headers() {
          const cookieStore = await cookies();
          const cookieString = cookieStore
            .getAll()
            .map((cookie) => `${cookie.name}=${cookie.value}`)
            .join("; ");

          console.log("cookies being sent:", cookieString);

          return {
            cookie: cookieString,
          };
        },
        fetch(url, options) {
          return fetch(url, {
            ...options,
            credentials: "include",
            cache: "no-store",
          });
        },
      }),
    ],
  });

export { trpcClient } from "./client";
export { Provider } from "./Provider";
```

This creates a type-safe client for Next.js Server Components with cookie forwarding.

### 34. Create React Query Client

Create `libs/trpc-client/src/client.ts`:

```typescript
import { AppRouter } from "@foundation-trpc/trpc-server";
import { createTRPCReact } from "@trpc/react-query";
import type { CreateTRPCReact } from "@trpc/react-query";

export const trpcClient: CreateTRPCReact<AppRouter, unknown> =
  createTRPCReact<AppRouter>();
```

### 35. Create Provider Component

Create `libs/trpc-client/src/Provider.tsx`:

```typescript
"use client";

import { trpcClient as trpc } from "./client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { httpBatchLink } from "@trpc/client";
import React, { useState } from "react";

export const Provider = ({ children }: { children: React.ReactNode }) => {
  const [queryClient] = useState(() => new QueryClient());
  const baseUrl = process.env.NEXT_PUBLIC_API_URL || "http://localhost:5000";

  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: `${baseUrl}/trpc`,
          fetch(url, options) {
            return fetch(url, { ...options, credentials: "include" });
          },
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </trpc.Provider>
  );
};
```

### 36. Build Libraries and Restart TypeScript

```bash
cd libs/trpc-client
pnpm build

# Restart your TypeScript server in your IDE
# VS Code: Cmd/Ctrl + Shift + P → "TypeScript: Restart TS Server"
```

## Integrating tRPC with Next.js

### 37. Install and Configure Provider

Install in Next.js app:

```bash
cd apps/web
pnpm add @foundation-trpc/trpc-client --workspace
```

Wrap your app with the Provider in `app/layout.tsx`:

```typescript
import { Provider } from "@foundation-trpc/trpc-client";

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body>
        <Provider>{children}</Provider>
      </body>
    </html>
  );
}
```

### 38. Using tRPC in Components

**Server Components:**

```typescript
import { trpc } from "@foundation-trpc/trpc-client";

const data = await trpc.test.test.query();
```

**Client Components:**

```typescript
"use client";
import { trpcClient } from "@foundation-trpc/trpc-client";

const { data } = trpcClient.test.test.useQuery();
```

### 39. Package Naming Consistency

Update package names in `apps/api/package.json` and `apps/web/package.json`:

```json
{
  "name": "@foundation-trpc/api"
}
```

```json
{
  "name": "@foundation-trpc/web"
}
```

## Optimized Build Scripts

### 40. Final Root Package Scripts

Update root `package.json` scripts:

```json
{
  "scripts": {
    "format": "prettier \"{apps,libs}/**/*.{ts,tsx,js,json}\" --ignore-path .gitignore",
    "format:check": "pnpm format --check",
    "format:write": "pnpm format --write",
    "build:libs": "nx run-many --target=build --projects=@foundation-trpc/types,@foundation-trpc/db,@foundation-trpc/auth,@foundation-trpc/core,@foundation-trpc/trpc-server,@foundation-trpc/trpc-client",
    "build:api": "pnpm build:libs && pnpm --filter @foundation-trpc/api build",
    "build:web": "pnpm build:libs && pnpm --filter @foundation-trpc/web build",
    "build:apps": "nx run-many --target=build --projects=@foundation-trpc/api,@foundation-trpc/web",
    "build": "pnpm build:libs && pnpm build:apps",
    "tsc": "pnpm exec nx run-many --target=tsc",
    "lint": "pnpm exec nx run-many --target=lint",
    "validate": "pnpm format:write && pnpm tsc && pnpm lint && pnpm build",
    "prepare": "husky",
    "dev": "nx run-many --target=dev --projects=@foundation-trpc/types,@foundation-trpc/core,@foundation-trpc/trpc-server,@foundation-trpc/api --parallel",
    "start:api": "pnpm --filter @foundation-trpc/api start",
    "start:web": "pnpm --filter @foundation-trpc/web start"
  }
}
```

Key scripts:

- `build:libs`: Builds all shared libraries in dependency order
- `build:api` / `build:web`: Builds applications with their dependencies
- `dev`: Runs development mode for API and libraries in parallel
- `validate`: Complete validation pipeline for CI/CD

## Deployment Configuration

### 41. Deploy API to Render

**Settings:**

- Root Directory: Leave as is (monorepo root)
- Build Command:
  ```bash
  pnpm install && pnpm build:libs --skip-nx-cache && pnpm --filter @foundation-trpc/api build
  ```
- Start Command:
  ```bash
  pnpm start:api
  ```
- Add all required environment variables

The `--skip-nx-cache` flag ensures fresh builds in CI/CD environments.

### 42. Deploy Next.js to Render

**Settings:**

- Root Directory: Leave as is (monorepo root)
- Build Command:
  ```bash
  pnpm install && pnpm build:libs --skip-nx-cache && pnpm --filter @foundation-trpc/web build
  ```
- Start Command:
  ```bash
  pnpm start:web
  ```

### 43. Deploy Next.js to Vercel

**Project Settings:**

**Environment Variables:**

```
ENABLE_EXPERIMENTAL_COREPACK=1
NODE_VERSION=22.x
# Add your other environment variables
```

**Build & Development Settings:**

- Root Directory: `apps/web`
- Build Command:
  ```bash
  cd ../.. && pnpm build:libs --skip-nx-cache && pnpm --filter @foundation-trpc/web build
  ```
- Install Command:
  ```bash
  corepack enable && corepack prepare pnpm@10.21.0 --activate && pnpm install
  ```
- Output Directory: Leave as default (`.next`)
- Development Command: Leave as default

**Note:** Corepack ensures Vercel uses your specific PNPM version, avoiding version mismatch issues.

## Pro Tips

### Library Development Best Practices

1. **Consistent Structure**: Use the same structure for all libraries:

   - Create `src/index.ts` as the main export point
   - Export everything through index for clean imports
   - Use consistent `package.json` configuration

2. **Standard Library Package.json:**

   ```json
   {
     "name": "@foundation-trpc/library-name",
     "version": "1.0.0",
     "main": "./dist/src/index.js",
     "types": "./dist/src/index.d.ts",
     "scripts": {
       "build": "rm -rf ./dist && tsc",
       "dev": "tsc --watch"
     }
   }
   ```

3. **TypeScript Configuration Template:**

   ```json
   {
     "extends": "../../tsconfig.json",
     "compilerOptions": {
       "outDir": "dist",
       "rootDir": ".",
       "jsx": "react-jsx",
       "esModuleInterop": true,
       "declaration": true,
       "skipLibCheck": true,
       "declarationMap": true,
       "composite": true
     },
     "include": ["src/**/*"]
   }
   ```

4. **Build Before Install**: Always build a library before installing it as a workspace dependency to ensure TypeScript can find type definitions.

5. **Context Types Library**: Consider creating a separate `@foundation-trpc/types` library for shared types like `AppContext` and `ProtectedContext` that multiple packages need to reference.

## Troubleshooting

### Common Issues

1. **TypeScript can't find types**: Build the library first, then reinstall it
2. **Environment variables not working in API**: Ensure `.env` exists in `apps/api/`
3. **tRPC context errors**: Verify all dependencies (db, auth) are built and installed
4. **Vercel build fails**: Check PNPM version matches between local and Vercel

### Useful Development Commands

```bash
# Build all libraries at once
pnpm build:libs

# Build and watch a specific library
pnpm --filter @foundation-trpc/trpc-server dev

# Clear NX cache
nx reset

# Check for type errors across workspace
pnpm tsc

# Run development servers in parallel
pnpm dev
```
