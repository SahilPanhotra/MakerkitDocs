-----------------
FILE PATH: kits\next-supabase-turbo\installation\llms.mdoc

---
status: "published"
title: "LLM rules for Cursor and Windsurf | Next.js Supabase"
label: "LLMs rules"
order: 11
description: "Use these rules for Cursor and Windsurf to generate the best possible context for your LLMs"
---

The following content is a guideline for [Cursor](https://www.cursor.com) and [Codeium Windsurf](https://codeium.com) to help you generate the best possible context for your LLMs. It should also work well for other coding tools, such as Windsurf.

1. **Cursor**: create a file `.cursorrules` file in your project root and paste the rules below.
2. **Windsurf**: create a file `.windsurfrules` file in your project root and paste the rules at the bottom of this page.

**Please Remember**: These tools are brilliant, but not perfect. Always double check the results and make sure they are relevant and useful.

## Cursor Rules for Makerkit

Cursor doesn't have a specified limit, so we recommend using the following content:

````md title=".cursorrules"
# Makerkit Guidelines

You are an expert programming assistant focusing on:

- Expertise: React, Supabase, TypeScript, Next.js 15, Shadcn UI, Tailwind CSS in a Turborepo project
- Focus: Code clarity, Readability, Best practices, Maintainability
- Style: Expert level, factual, solution-focused
- Libraries: TypeScript, React Hook Form, React Query, Zod, Lucide React

## Application Scope

<INSERT_HERE_YOUR_APPLICATION_SCOPE_HERE>

## Project Structure

The below is the Makerkit's Next.js App Router structure.

```
- apps
-- web
--- app
---- home        # protected routes
------ (user)    # user workspace
------ [account] # team workspace
---- (marketing) # marketing pages
---- auth        # auth pages
--- components   # global components
--- config       # global config
--- lib          # global utils
--- content      # markdoc content
--- styles
--- supabase     # supabase root
```

## Configuration

- When adding paths under the user workspace, update the file `apps/web/config/personal-account-navigation.config.tsx`
- When adding paths under the team workspace, update the file `apps/web/config/team-account-navigation.config.tsx`

## Database

- Supabase uses Postgres
- We strive to create a safe, robust, performant schema
- Accounts are the general concept of a user account, defined by the having the same ID as Supabase Auth's users (personal). They can be a team account or a personal account.
- Generally speaking, other tables will be used to store data related to the account. For example, a table `notes` would have a foreign key `account_id` to link it to an account.
- Using RLS, we must ensure that only the account owner can access the data. Always write safe RLS policies and ensure that the policies are enforced.
- Unless specified, always enable RLS when creating a table. Propose the required RLS policies ensuring the safety of the data.
- Always consider any required constraints and triggers are in place for data consistency
- Always consider the compromises you need to make and explain them so I can make an educated decision. Follow up with the considerations make and explain them.
- Always consider the security of the data and explain the security implications of the data.
- Always use Postgres schemas explicitly (e.g., `public.accounts`)
- Data types should always be inferred using the `Database` types from `@kit/supabase/database`

```tsx
import type { Tables } from '@kit/supabase/database';
type Bookmark = Tables<'bookmarks'>;
```

## Personal Account Context

The user/personal account context in the application lives under the path `app/home/(user)`. Under this context, we identify the user using Supabase Auth.

We can use the `requireUserInServerComponent` to retrieve the relative Supabase User object and identify the user.

### Client Components

In a Client Component, we can access the `UserWorkspaceContext` and use the `user` object to identify the user.

We can use it like this:

```tsx
import { useUserWorkspace } from '@kit/accounts/hooks/use-user-workspace';
```

## Team Account Context

The team account context in the application lives under the path `app/home/[account]`. The `[account]` segment is the slug of the team account, from which we can identify the team.

### Accessing the Account Workspace Data in Client Components

The data fetched from the account workspace API is available in the team context. You can access this data using the useAccountWorkspace hook.

```tsx
'use client';

import { useTeamAccountWorkspace } from '@kit/team-accounts/hooks/use-team-account-workspace';

export default function SomeComponent() {
  const { account, user, accounts } = useTeamAccountWorkspace();
  // use account, user, and accounts
}
```

The useTeamAccountWorkspace hook returns the same data structure as the loadTeamWorkspace function.

NB: the hooks is not to be used is Server Components, only in Client Components. Additionally, this is only available in the pages under /home/[account] layout.

### Team Pages

These pages are dedicated to the team account, which means they are only accessible to team members. To access these pages, the user must be authenticated and belong to the team.

## UI Components

Reusable UI components are defined in the "packages/ui" package named "@kit/ui".
By exporting the component from the "exports" field, we can import it using the "@kit/ui/{component-name}" format.

### Code Standards

- Files
  - Always use kebab-case
- Naming
  - Functions/Vars: camelCase
  - Constants: UPPER_SNAKE_CASE
  - Types/Classes: PascalCase
- TypeScript
  - Prefer types over interfaces
  - Use type inference whenever possible
  - Avoid any, any[], unknown, or any other generic type
  - Use spaces between code blocks to improve readability

### Styling

- Styling is done using Tailwind CSS. We use the "cn" function from the "@kit/ui/utils" package to generate class names.
- Avoid fixes classes such as "bg-gray-500". Instead, use Shadcn classes such as "bg-background", "text-secondary-foreground", "text-muted-foreground", etc.

### Data Fetching

- In a Server Component context, please use the Supabase Client directly for data fetching
- In a Client Component context, please use the `useQuery` hook from the "@tanstack/react-query" package

Data Flow works in the following way:

1. Server Component uses the Supabase Client to fetch data.
2. Data is rendered in Server Components or passed down to Client Components when absolutely necessary to use a client component (e.g. when using React Hooks or any interaction with the DOM).

```tsx
import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const { data, error } = await client.from('notes').select('*');

  // use data
}
```

or pass down the data to a Client Component:

```tsx
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export default function ServerComponent() {
  const supabase = getSupabaseServerClient();
  const { data, error } = await supabase.from('notes').select('*');

  if (error) {
    return <SomeErrorComponent error={error} />;
  }

  return <SomeClientComponent data={data} />;
}
```

#### Supabase Clients

- In a Server Component context, use the `getSupabaseServerClient` function from the "@kit/supabase/server-client" package.
- In a Client Component context, use the `useSupabase` hook from the "@kit/supabase/hooks/use-supabase" package.

##### Admin Actions

Only in rare cases suggest using the Admin client `getSupabaseServerAdminClient` when needing to bypass RLS from the package `@kit/supabase/server-admin-client`.

#### React Query

When using `useQuery`, make sure to define the data fetching hook. Create two components: one that fetches the data and one that displays the data.

## Server Actions

- For Data Mutations from Client Components, always use Server Actions
- Always name the server actions file as "server-actions.ts"
- Always name exported Server Actions suffixed as "Action", ex. "createPostAction"
- Always use the `enhanceAction` function from the "@kit/supabase/actions" package.

```tsx
'use server';

import { z } from 'zod';

import { enhanceAction } from '@kit/next/actions';

const ZodSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export const myServerAction = enhanceAction(
  async function (data, user) {
    // 1. "data" is already a valid ZodSchema and it's safe to use
    // 2. "user" is the authenticated user

    // ... your code here
    return {
      success: true,
    };
  },
  {
    auth: true,
    schema: ZodSchema,
  },
);
```

## Route Handler / API Routes

- Use Route Handlers when data fetching from Client Components
- To create API routes (route.ts), always use the `enhanceRouteHandler` function from the "@kit/supabase/routes" package.

```tsx
import { NextResponse } from 'next/server';

import { z } from 'zod';

import { enhanceRouteHandler } from '@kit/next/routes';

const ZodSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export const POST = enhanceRouteHandler(
  async function ({ body, user, request }) {
    // 1. "body" is already a valid ZodSchema and it's safe to use
    // 2. "user" is the authenticated user
    // 3. "request" is NextRequest
    // ... your code here
    return NextResponse.json({
      success: true,
    });
  },
  {
    schema: ZodSchema,
  },
);

// example of unauthenticated route (careful!)
export const GET = enhanceRouteHandler(
  async function ({ user, request }) {
    // 1. "user" is null, as "auth" is false and we don't require authentication
    // 2. "request" is NextRequest
    // ... your code here
    return NextResponse.json({
      success: true,
    });
  },
  {
    auth: false,
  },
);
```

Consider logging asynchronous requests using the `@kit/shared/logger` package in a structured way to provide context to the logs in both server actions and route handlers.

```tsx
const ctx = {
  name: 'my-server-action', // use a meaningful name
  userId: user.id, // use the authenticated user's ID
};

logger.info(ctx, 'Request started...');

const { data, error } = await supabase.from('notes').select('*');

if (error) {
  logger.error(ctx, 'Request failed...');
  // handle error
} else {
  logger.info(ctx, 'Request succeeded...');
  // use data
}
```

### Related APIs

When required, use the following APIs.

1. Personal Account APIs:

```tsx
import { createAccountsApi } from '@kit/accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const api = createAccountsApi(client);
  // use api
}
```

2. Team Account APIs:

```tsx
import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { createTeamAccountsApi } from '@kit/team-accounts/api';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const api = createTeamAccountsApi(client);
  // use api
}
```

3. Auth API:

```tsx
import { redirect } from 'next/navigation';

import { requireUser } from '@kit/supabase/require-user';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const auth = await requireUser(client);
  // check if the user needs redirect
  if (auth.error) {
    redirect(auth.redirectTo);
  }
  // user is authed!
  const user = auth.data;
}
```

4. Billing API:

```tsx
import { createBillingGatewayService } from '@kit/billing-gateway';

const service = createBillingGatewayService('stripe');
```

## Creating Pages

When creating new pages ensure:

1. The page is exported using `withI18n` to enable i18n.
2. The page has the required and correct metadata using the `metadata` or `generateMetadata` function.
3. Don't worry about authentication, it will be handled automatically.

```tsx
import { Metadata } from 'next';

import { withI18n } from '~/lib/i18n/with-i18n';

export const metadata: Metadata = {
  title: 'A page title',
  description: 'A page description',
};

function Page() {
  // ...
}

export default withI18n(Page);
```

## Forms

- Use React Hook Form for form validation and submission.
- Use Zod for form validation.
- Use the `zodResolver` function to resolve the Zod schema to the form.

Follow the example below to create all forms:

### Define the schema

Zod schemas should be defined in the `schema` folder and exported, so we can reuse them across a Server Action and the client-side form:

```tsx
// _lib/schema/create-note.schema.ts
import { z } from 'zod';

export const CreateNoteSchema = z.object({
  title: z.string().min(1),
  content: z.string().min(1),
});
```

### Create the Server Action

```tsx
// _lib/server/server-actions.ts
'use server';

import { z } from 'zod';
import { enhanceAction } from '@kit/next/actions';
import { CreateNoteSchema } from '../schema/create-note.schema';

const CreateNoteSchema = z.object({
  title: z.string().min(1),
  content: z.string().min(1),
});

export const createNoteAction = enhanceAction(
  async function (data, user) {
    // 1. "data" has been validated against the Zod schema, and it's safe to use
    // 2. "user" is the authenticated user

    // ... your code here
    return {
      success: true,
    };
  },
  {
    auth: true,
    schema: CreateNoteSchema,
  },
);
```

Then create a client component to handle the form submission:

```tsx
// _components/create-note-form.tsx
'use client';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@kit/ui/form';

import { CreateNoteSchema } from '../_lib/schema/create-note.schema';

export function CreateNoteForm() {
  const [pending, startTransition] = useTransition();

  const form = useForm({
    resolver: zodResolver(CreateNoteSchema),
    defaultValues: {
      title: '',
      content: '',
    },
  });

  const onSubmit = (data) => {
    startTransition(async () => {
      try {
        await createNoteAction(data);
      } catch {
        // handle error
      }
    });
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <Form {...form}>
        <FormField
          name={'title'}
          render={({ field }) => (
            <FormItem>
              <FormLabel>
                <span className={'text-sm font-medium'}>Title</span>
              </FormLabel>

              <FormControl>
                <input
                  type={'text'}
                  className={'w-full'}
                  placeholder={'Title'}
                  {...field}
                />
              </FormControl>

              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          name={'content'}
          render={({ field }) => (
            <FormItem>
              <FormLabel>
                <span className={'text-sm font-medium'}>Content</span>
              </FormLabel>

              <FormControl>
                <textarea
                  className={'w-full'}
                  placeholder={'Content'}
                  {...field}
                />
              </FormControl>

              <FormMessage />
            </FormItem>
          )}
        />

        <button disabled={pending} type={'submit'} className={'w-full'}>
          Submit
        </button>
      </Form>
    </form>
  );
}
```

Always use `@kit/ui` for writing the UI of the form.

## Error Handling

- Logging using the `@kit/shared/logger` package
- Don't swallow errors, always handle them appropriately
- Handle promises and async/await gracefully
- Consider the unhappy path and handle errors appropriately
- Context without sensitive data

## Translations

- Use the `Trans` component from the `@kit/ui/trans` package to translate strings.
- Translations are placed in the `apps/web/public/locales/{language}/{namespace}.json` path.
- To add a new language, update the `languages` array in the `apps/web/lib/i18n/i18n.settings.ts` file.
- Consider adding a new namespace for a new functionality/page.
````

## Windsurf Rules for Makerkit

Windsurf has a limit of 6000 characters per file, so we recommend using the following content:

````md title=".windsurfrules"
# Makerkit Guidelines

## Project Stack
- Framework: Next.js 15 App Router, TypeScript, React, Node.js
- Backend: Supabase with Postgres
- UI: Shadcn UI, Tailwind CSS
- Key libraries: React Hook Form, React Query, Zod, Lucide React
- Focus: Code clarity, Readability, Best practices, Maintainability

## Project Structure
```
/apps/web/
  /app
    /home          # protected routes
      /(user)      # user workspace
      /[account]   # team workspace
    /(marketing)   # marketing pages
    /auth         # auth pages
  /components     # global components
  /config        # global config
  /lib           # global utils
  /content       # markdoc content
  /supabase      # supabase root
```

## Core Principles

### Data Flow
1. Server Components
  - Use Supabase Client directly via `getSupabaseServerClient`
  - Handle errors with proper boundaries
  - Example:
  ```tsx
  async function ServerComponent() {
    const client = getSupabaseServerClient();
    const { data, error } = await client.from('notes').select('*');
    if (error) return <ErrorComponent error={error} />;
    return <ClientComponent data={data} />;
  }
  ```

2. Client Components
  - Use React Query for data fetching
  - Implement proper loading states
  - Example:
  ```tsx
  function useNotes() {
    const { data, isLoading } = useQuery({
      queryKey: ['notes'],
      queryFn: async () => {
        const { data } = await fetch('/api/notes');
        return data;
      }
    });
    return { data, isLoading };
  }
  ```

### Server Actions
- Name files as "server-actions.ts" in `_lib/server` folders
- Export with "Action" suffix
- Use `enhanceAction`. Use a single parameter for data
- Use `revalidatePath` to refresh the data after a submission
- Example:
```tsx
'use server';

import { enhanceAction } from '@kit/next/actions';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export const createNoteAction = enhanceAction(
  async function (data, user) {
    const client = getSupabaseServerClient();
    const { error } = await client
      .from('notes')
      .insert({ ...data, user_id: user.id });

    if (error) throw error;
    return { success: true };
  },
  {
    auth: true,
    schema: NoteSchema,
  }
);
```

From client components:

```tsx
await createNoteAction(data);
```

### Route Handlers

- Use `enhanceRouteHandler` to wrap route handlers
- Use Route Handlers when data fetching from Client Components

```tsx
import { enhanceRouteHandler } from '@kit/next/routes';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export const POST = enhanceRouteHandler(
  async ({ request, data, user }) => {
    const client = getSupabaseServerClient();
    // ....
  },
  {
    auth: true,
    schema: NoteSchema,
  }
);
```

## Database & Security

### RLS Policies
- Strive to create a safe, robust, secure and consistent database schema
- Always consider the compromises you need to make and explain them so I can make an educated decision. Follow up with the considerations make and explain them.
- Enable RLS by default and propose the required RLS policies
- `public.accounts` are the root tables for the application
- Implement cascading deletes when appropriate
- Ensure strong consistency considering triggers and constraints
- Always use Postgres schemas explicitly (e.g., `public.accounts`)
- Data types should always be inferred using the `Database` types from `@kit/supabase/database`
```tsx
import type { Tables } from '@kit/supabase/database';
type Bookmark = Tables<'bookmarks'>;
```

## Forms Pattern

### 1. Schema Definition
```tsx
// schema/note.schema.ts
import { z } from 'zod';

export const NoteSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(1),
  category: z.enum(['work', 'personal']),
});
```

### 2. Form Component
```tsx
'use client';

export function NoteForm() {
  const [pending, startTransition] = useTransition();

  const form = useForm({
    resolver: zodResolver(NoteSchema),
    defaultValues: { title: '', content: '', category: 'personal' }
  });

  const onSubmit = (data: z.infer<typeof NoteSchema>) => {
    startTransition(async () => {
      try {
        await createNoteAction(data);
        form.reset();
      } catch (error) {
        // Handle error
      }
    });
  };

  return (
    <Form {...form}>
      <FormField name="title" render={({ field }) => (
        <FormItem>
          <FormLabel>Title</FormLabel>
          <FormControl>
            <Input {...field} />
          </FormControl>
          <FormMessage />
        </FormItem>
      )} />
      {/* Other fields */}
    </Form>
  );
}
```

## Error Handling

- Consider logging asynchronous requests in server code using the `@kit/shared/logger`
- Handle promises and async/await gracefully
- Consider the unhappy path and handle errors appropriately

### Structured Logging
```tsx
const ctx = {
  name: 'create-note',
  userId: user.id,
  noteId: note.id
};

logger.info(ctx, 'Creating new note...');

try {
  await createNote();
  logger.info(ctx, 'Note created successfully');
} catch (error) {
  logger.error(ctx, 'Failed to create note', { error });
  throw error;
}
```

## Context Management

In client components, we can use the `useUserWorkspace` hook to access the user's workspace data.

### Personal Account
```tsx
'use client';

function PersonalDashboard() {
  const { workspace, user } = useUserWorkspace();
  if (!workspace) return null;

  return (
    <div>
      <h1>Welcome, {user.email}</h1>
      <SubscriptionStatus status={workspace.subscription_status} />
    </div>
  );
}
```

### Team Account
In client components, we can use the `useTeamAccountWorkspace` hook to access the team account's workspace data. It only works under the `/home/[account]` route.

```tsx
'use client';

function TeamDashboard() {
  const { account, user } = useTeamAccountWorkspace();

  return (
    <div>
      <h1>{account.name}</h1>
      <RoleDisplay role={account.role} />
      <PermissionsList permissions={account.permissions} />
    </div>
  );
}
```

## UI Components

- Reusable UI components are defined in the "packages/ui" package named "@kit/ui".
- By exporting the component from the "exports" field, we can import it using the "@kit/ui/{component-name}" format.

## Creating Pages

When creating new pages ensure:
- The page is exported using `withI18n(Page)` to enable i18n.
- The page has the required and correct metadata using the `metadata` or `generateMetadata` function.
- Don't worry about authentication, it's handled in the middleware.
````

You can add more rules to Windsurf using the global rules, which are a set of rules that are applied to all projects:

```mdx
I use Typescript, React, Next.js App Router, Zod and Shadcn UI.

## Typescript
- My Typescript is always typed, but never returns explicit types.
- My Typescript is strict.
- I write small, self-contained, pure functions. I avoid mutations.
- My code is simple, self-explanatory, and readable.
- I write spaces between code blocks to make it easier to read.
- Prefer types over interfaces
- Use type inference whenever possible
- Avoid any, any[], unknown, or any other generic type
- Use spaces between code blocks to improve readability

## React
- I write small and reusable React components. I only use `useState` when required. If I do, I make sure to use a few as possible and group state together to avoid unnecessary re-renders.
- I write small and reusable React components. I only use `useEffect` when required, which means rarely.
- Components are written with clear intent and purpose, use props to pass data from parent to child and are reusable across the app.
```

## Additional context for LLMs

You can generate a set of Markdown files using our little script and the [open source Makerkit documentation](https://github.com/makerkit/documentation).

You can then use the Markdown files as a knowledge base for your LLMs, such as Cursor, ChatGPT or Claude.

Clone the repository, then run the following command:

```bash
node index.js kits/next-supabase-turbo
```

To ensure a more precise context, always scope the generation for the kit you are interested in. For example, to generate the markdown files for the Remix Supabase Kit, run the following command:

```bash
node index.js kits/remix-supabase-turbo
```

You can also do it for the courses by running the following command:

```bash
node index.js courses
```

This will generate markdown files in the dist folder.

You can configure how many words per file by passing a second argument to the script, e.g.:

```bash
node index.js kits 4000
```

By default, it will generate 5000 words per file.

-----------------
FILE PATH: kits\next-supabase-turbo\installation\migration-from-makerkit-v1.mdoc

---
status: "published"
title: "Guidelines for migrating from Makerkit v1 | Next.js Supabase SaaS Starter Kit"
label: "Migrating from v1"
order: 9
description: "Guidelines for migrating from Makerkit v1 to Next.js SaaS Boilerplate."
---

🚀 Welcome to your journey from Makerkit v1 to the Next.js SaaS Boilerplate! 

This guide is designed to help you understand the changes between the two versions and navigate your project migration to the new v2. Whether you're a seasoned developer or just starting out, we've got you covered!

Here's a roadmap of the steps you'll take:

1. **Bootstrap a new project**: Kickstart a new project using the Next.js SaaS Boilerplate v2 🎉
2. **Update Schema**: Tweak the Postgres schema foreign key references, and integrate your existing tables 🧩
3. **Move files from older app**: Transport your files to the new app structure 📦
4. **Update imports to existing files**: Refresh imports to align with the new file structure 🔄
5. **Update configuration**: Modify the configuration files to match the new schemas ⚙️

## 1. Bootstrap a new project

The Next.js SaaS Boilerplate is a fresh take on Makerkit v1. You'll need to create a new project using this innovative boilerplate. Follow the [installation guide](clone-repository) to get your new project up and running in no time!

## 2. Update Schema

The schema in the Next.js SaaS Boilerplate has evolved significantly from Makerkit v1. You'll need to update your Postgres schema to match the new one.

Previously, you'd have a foreign key set to the organization ID:

```sql
organization_id bigint not null references organizations(id) on delete cascade,
```

Now, you'll have a foreign key set to the account ID as a UUID:

```sql
account_id uuid not null references public.accounts(id) on delete cascade,
```

In v2, an account can be both a user or an organization. So, instead of referencing the organization ID as in v1, you'll now reference the account ID. 

## 3. Move files from older app

In v2, you have the flexibility to add these files to either the "user scope" (`/home`) or the "account scope" (`/home/[account]`). Choose the one that best suits your project needs.

## 4. Update imports to existing files

You'll need to update the imports to your existing files to match the new file structure. This applies to all the components and utilities that you imported from Makerkit. For instance, a button previously imported from `~/core/ui/Button`, will now be imported from `@kit/ui/button`.

## 5. Update configuration

Lastly, you'll need to update the configuration files to match the new schemas. The configuration is now split across various files at `apps/web/config`. Pay special attention to the billing schema, which is now more versatile (and a bit more complex).

Ready to get started? Let's dive in! 🏊‍♀️

-----------------
FILE PATH: kits\next-supabase-turbo\installation\navigating-codebase.mdoc

---
status: "published"

title: "Navigating the your Next.js Supabase Turbo Starter Kit codebase"
label: "Navigating the Codebase"
order: 7
description: "Learn how to navigate the codebase of the Next.js Supabase Turbo Starter Kit. Understand the project structure and how to update the codebase."
---


As mentioned before, this SaaS Starter Kit is modular, thanks to Turborepo - which makes it easy to manage multiple packages in a single repository.

## Project Structure

The main directories in the project are:
1. `apps` - the location of the Next.js app
2. `packages` - the location of the shared code and the API

### `apps` Directory

This is where the app (or the apps, should you add more) lives. The main app is a Next.js app, which is a React framework that allows you to build static and server-rendered web applications.

The Next.js app defines:

1. the configuration of the project
2. the routes of the application

### `packages` Directory

This is where the shared code and the API live. The shared code is a collection of functions that are used by both the Next.js app and the API. The API is a serverless function that is used to interact with the Supabase database.

The shared code defines:

1. shared libraries (Supabase, Mailers, CMS, Billing, etc.)
2. shared features (admin, accounts, account, auth, etc.)
3. UI components (buttons, forms, modals, etc.)

All apps can use and reuse the API exported from the `packages` directory. This makes it easy to have one, or many apps in the same codebase, sharing the same code.

## Diving into the Codebase

The codebase is structured in a way that makes it easy to navigate and update. Here are some of the key files and directories you should be aware of:

```
- apps
---
 web
---
-- app
---
-- components
---
-- config
---
-- lib
---
-- content
---
-- styles
---
-- supabase
```


### Diving into `apps/web`

The `apps/web` directory is where the Next.js app lives. Here are some of the key directories and files you should be aware of:

1. `app` - this is where the main app lives. This is where you define the routes of the application.
2. `components` - this is where you define the shared components of the application.
3. `config` - this is where you define the configuration of the application.
4. `lib` - this is where you define the shared libraries of the application.
5. `content` - this is where you define the content of the application (by default, uses Keystatic Markdoc files)
6. `styles` - this is where you define the styles of the application.
7. `supabase` - this is where you define the Supabase configuration, migrations and tests

### Diving into `app`

The `app` directory is where the routing of the application is defined. Here are some of the key files you should be aware of:

```
- app
---
 home
---
 (marketing)
---
 auth
---
 join
---
 admin
---
 update-password
---
 server-sitemap.xml
```

Let's break down the directories:

1. `home` - this is where the internal home of the application lives. This is where you define the routes when the user is logged in.
2. `(marketing)` - this is where the marketing pages of the application live. This is where you define the routes when the user is not logged in. It's a pathless route, so you don't see it in the URL.
3. `auth` - this is where the authentication pages of the application live. This is where you define the routes for the login, signup, and forgot password pages.
4. `join` - this is where the join pages of the application live. This route is the route where the user can join a team account following an invitation.
5. `admin` - this is where the admin pages of the application live. This is where you define the routes for the super admin pages.
6. `update-password` - this is where the update password pages of the application live. This is the route the user is redirected to after resetting their password.
7. `server-sitemap.xml` - this is where the sitemap of the application is defined.

### Diving into `home`

The `home` directory is where the internal dashboard of the application lives. Here are some of the key files you should be aware of:

```
---
 home
---
---
 (user)
---
---
 [account]
```

Let's break down the directories:

1. `(user)` - this is where the user pages of the application live. This is where you define the routes for the personal account pages.
2. `[account]` - this is where the account pages of the application live. This is where you define the routes for the team account pages.

The `home` path allows us to separate the marketing pages from the internal dashboard pages.

So the user home page would be `/home/user` and the account home page would be `/home/[account]`.






