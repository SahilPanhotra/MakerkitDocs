-----------------
FILE PATH: kits\next-supabase-turbo\plugins\waitlist-plugin.mdoc

---
status: "published"
title: 'Add a Waitlist to the Next.js Supabase SaaS Starter kit'
label: 'Waitlist'
order: 1
description: 'Add a waitlist to your Next.js Supabase SaaS Starter kit to collect emails from interested users.'
---

In this guide, you will learn how to add a waitlist to your Next.js Supabase SaaS Starter kit to collect emails from interested users. This feature is useful for building an audience before launching your product.

This plugin allows you to create a waitlist for your app. Users can sign up for the waitlist and receive an email when the app is ready.

### How it works

1. You disable sign up in your app from Supabase. This prevents any user from using the public API to sign up.
2. We create a new table in Supabase called `waitlist`. Users will sign up for the waitlist and their email will be stored in this table.
3. When you want to enable a sign up for a user, mark users as `approved` in the `waitlist` table.
4. The database trigger will create a new user in the `auth.users` table and send an email to the user with a link to set their password.
5. The user can now sign in to the app and update their password.
6. User gets removed from the waitlist as soon as the email is sent.

### Installation

#### Get the plugin using the CLI

Please run the following command in your terminal:

```bash
npx @makerkit/cli@latest plugins install waitlist
```

After completed, the CLI will install the plugin at `packages/plugins/waitlist`.

#### Add the plugin to your main app

Now, install the plugin from your main app by adding the following to your `package.json` file:

 ```json {% title="apps/web/package.json" %}
{
  "dependencies": {
    "@kit/waitlist": "workspace:*"
  }
}
```

And then run `pnpm install` to install the plugin.

#### Add migrations to your app

From the folder `packages/plugins/waitlist/migrations`, copy the migrations to your app's migrations folder at `apps/web/supabase/migrations`.

This will add the `waitlist` table to your database.

Now, re-run the migrations:

```bash
pnpm run supabase:web:reset
pnpm run supabase:web:typegen
```

You can now use the waitlist plugin in your app.

#### Replace the sign up form

Replace your sign up form with the waitlist form in your app (retain the existing imports):

```tsx
import { WaitlistSignupForm } from '@kit/waitlist/client';

function SignUpPage({ searchParams }: Props) {
  const inviteToken = searchParams.invite_token;

  const signInPath =
    pathsConfig.auth.signIn +
    (inviteToken ? `?invite_token=${inviteToken}` : '');

  return (
    <>
      <Heading level={4}>
        <Trans i18nKey={'auth:signUpHeading'} />
      </Heading>

      <WaitlistSignupForm inviteToken={inviteToken} />

      <div className={'justify-centers flex'}>
        <Button asChild variant={'link'} size={'sm'}>
          <Link href={signInPath}>
            <Trans i18nKey={'auth:alreadyHaveAnAccount'} />
          </Link>
        </Button>
      </div>
    </>
  );
}

export default withI18n(SignUpPage);
```

#### Adding the Database Webhook to listen for new signups

Let's extend the DB handler at `apps/web/api/db/webhook/route.ts`. This handler will listen for new signups and send an email to the user:

```tsx
import { getDatabaseWebhookHandlerService } from '@kit/database-webhooks';
import { enhanceRouteHandler } from '@kit/next/routes';

import appConfig from '~/config/app.config';
import pathsConfig from '~/config/paths.config';

/**
 * @name POST
 * @description POST handler for the webhook route that handles the webhook event
 */
export const POST = enhanceRouteHandler(
  async ({ request }) => {
    const service = getDatabaseWebhookHandlerService();

    try {
      // handle the webhook event
      await service.handleWebhook(request, {
        async handleEvent(payload) {
          if (payload.table === 'waitlist' && payload.record.approved) {
            const { handleApprovedUserChange } = await import(
              '@kit/waitlist/server'
            );

            const inviteToken = payload.record.invite_token;
            const redirectToUrl = new URL(pathsConfig.auth.passwordUpdate, appConfig.url);

            if (inviteToken) {
              const next = encodeURI(pathsConfig.app.joinTeam + '?invite_token=' + inviteToken);
              redirectToUrl.searchParams.append('callback', next);
            }

            const redirectTo = redirectToUrl.toString();

            await handleApprovedUserChange({
              email: payload.record.email,
              redirectTo,
            });
          }
        },
      });

      // return a successful response
      return new Response(null, { status: 200 });
    } catch {
      // return an error response
      return new Response(null, { status: 500 });
    }
  },
  {
    auth: false,
  },
);

```

#### Adding the Trigger to the Database

We need to add a trigger to the `waitlist` table to listen for updates and send a webhook to the app when a user is approved.

During development, you can simply add the webhook to your seed file `apps/web/supabase/seed.sql`:

```sql
create trigger "waitlist_approved_update" after update
on "public"."waitlist"
for each row
when (new.approved = true)
execute function "supabase_functions"."http_request"(
  'http://host.docker.internal:3000/api/db/webhook',
  'POST',
  '{"Content-Type":"application/json", "X-Supabase-Event-Signature":"WEBHOOKSECRET"}',
  '{}',
  '5000'
);
```

The above creates a trigger that listens for updates to the `waitlist` table and sends a POST request to the webhook route.

**Note**: You need to add this trigger to your production database as well. You will replace your `WEBHOOKSECRET` with the secret you set in your `.env` file and the `host.docker.internal:3000` with your production URL.
Just like you did for the other existing triggers.

#### Approving users

Simply update the `approved` column in the `waitlist` table to `true` to approve a user. You can do so from the Supabase dashboard or by running a query.

Alternatively, run an update based on the created_at timestamp:

```sql
update public.waitlist
set approved = true
where created_at < '2024-07-01';
```

#### Email Templates and URL Configuration

Please make sure to [edit the email template](https://makerkit.dev/docs/next-supabase-turbo/authentication-emails) in your Supabase account.
The default email in Supabase does not support PKCE and therefore does not work. By updating it - we replace the existing strategy with the token-based strategy - which the `confirm` route in Makerkit can support.

Additionally, [please add the following URL to your Supabase Redirect URLS allow list](https://supabase.com/docs/guides/auth/redirect-urls):

```
<your-url>/password-reset
```

This will allow Supabase to redirect users to your app to set their password after they click the email link.

If you don't do this - the email links will not work.

#### Translations

Please add the following translations to your `auth.json` translation:

```json
{
  "waitlist": {
    "heading": "Join the Waitlist for Early Access",
    "submitButton": "Join Waitlist",
    "error": "Ouch, we couldn't add you to the waitlist. Please try again.",
    "errorDescription": "We couldn't add you to the waitlist. If you have already signed up, you are already on the waitlist.",
    "success": "You're on the waitlist!",
    "successDescription": "We'll let you know when you can sign up."
  }
}
```

### Tailwind CSS Styles

If using TailwindCSS v3, please update the Tailwind CSS styles to include the new folder:

```js { % title="tooling/tailwind/index.ts" %}
export default {
  darkMode: ['class'],
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/waitlist/src/**/*.tsx'  // <-- add this line
    '!**/node_modules',
  ],
  // ...
};
```

The above is not required if you are using TailwindCSS v4.

---

Easy peasy! Now you have a waitlist for your app.

-----------------
FILE PATH: kits\next-supabase-turbo\plugins.mdoc

---
status: "published"
title: "Plugins in Next.js Supabase Turbo"
label: "Plugins"
order: 14
description: "Plugins in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Extend your application's functionality with our plugin system:

1. [**Getting Started with Plugins**](/docs/next-supabase-turbo/installing-plugins)
   Learn how to install and manage plugins in your application.
2. Available Plugins
   - [Chatbot Plugin](/docs/next-supabase-turbo/plugins/chatbot-plugin)
   - [Feedback Plugin](/docs/next-supabase-turbo/plugins/feedback-plugin)
   - [Text Editor Plugin](/docs/next-supabase-turbo/plugins/text-editor-plugin)
   - [Waitlist Plugin](/docs/next-supabase-turbo/plugins/waitlist-plugin)
   - [Testimonials Plugin](/docs/next-supabase-turbo/plugins/testimonials-plugin)
   - [Roadmap Plugin](/docs/next-supabase-turbo/plugins/roadmap-plugin)


-----------------
FILE PATH: kits\next-supabase-turbo\recipes\drizzle-supabase.mdoc

---
status: "published"
title: "Using Drizzle as a client for interacting with Supabase"
label: "Drizzle"
order: 6
description: "Learn how to add Drizzle to your Next.js Supabase Turbo project and use it to interact with Supabase"
---

By default, the kit interacts with Supabase using the plain Supabase client. 

This guide will walk you through adding [Drizzle ORM](https://orm.drizzle.team/) support to your Makerkit project. Drizzle is a TypeScript ORM that provides type-safe database queries with great developer experience.

This guide is an adaptation of the Drizzle guide for Next.js applications. [The original guide can be found here](https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase).

Please refer to the [Drizzle documentation](https://orm.drizzle.team/docs/overview) for more advanced usage and configuration options.

## Prerequisites
- A working Makerkit project
- Basic understanding of TypeScript and SQL
- Supabase project set up

## Step 1: Install Dependencies

First, let's add the required dependencies to the `@kit/supabase` package:

```bash
pnpm --filter "@kit/supabase" add jwt-decode postgres
pnpm --filter "@kit/supabase"add -D drizzle-orm drizzle-kit
```

We added the required dependencies to the `@kit/supabase` package.

## Step 2: Create Drizzle Configuration

Create a new file `packages/supabase/drizzle.config.js`:

```javascript {% title="packages/supabase/drizzle.config.js" %}
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/schema.ts',
  out: './src/drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    // This will use local development DB by default
    url: process.env.DATABASE_URL ?? 'postgresql://postgres:postgres@127.0.0.1:54322/postgres'
  },
  schemaFilter: ['public'],
  verbose: true,
  strict: true,
});
```

We will use the Drizzle Kit CLI to generate a schema out of the Supabase database. This will in turn create the `schema.ts` file in the `out` directory.

The `schema.ts` file will contain the TypeScript types matching your database schema that Drizzle uses to infer the TS types for your database queries.

## Step 3: Add Scripts to package.json

Update `packages/supabase/package.json`:

```json {% title="packages/supabase/package.json" %}
{
  "scripts": {
    // ... existing scripts
    "drizzle": "drizzle-kit",
    "pull": "drizzle-kit pull --config drizzle.config.js"
  },
  "exports": {
    // Add these new exports
    "./drizzle-client": "./src/clients/drizzle-client.ts",
    "./drizzle-schema": "./src/drizzle/schema.ts"
  }
}
```

These commands will help us pull the schema from the Supabase database and generate the `schema.ts` file.

## Step 4: Create Drizzle Client

Create a new file `packages/supabase/src/clients/drizzle-client.ts`:

```typescript {% title="packages/supabase/src/clients/drizzle-client.ts" %}
import 'server-only';

import { DrizzleConfig, sql } from 'drizzle-orm';
import { drizzle } from 'drizzle-orm/postgres-js';
import { JwtPayload, jwtDecode } from 'jwt-decode';
import postgres from 'postgres';
import { z } from 'zod';

import * as schema from '../drizzle/schema';

// Config validation
const SUPABASE_DATABASE_URL = z
  .string({
    description: `The URL of the Supabase database. Please provide the variable SUPABASE_DATABASE_URL.`,
    required_error: 'The environment variable SUPABASE_DATABASE_URL is required',
  })
  .url()
  .parse(process.env.SUPABASE_DATABASE_URL!);

const config = {
  casing: 'snake_case',
  schema,
} satisfies DrizzleConfig<typeof schema>;

// Admin client bypasses RLS
const adminClient = drizzle({
  client: postgres(SUPABASE_DATABASE_URL, { prepare: false }),
  ...config,
});

// RLS protected client
const rlsClient = drizzle({
  client: postgres(SUPABASE_DATABASE_URL, { prepare: false }),
  ...config,
});

export function getDrizzleSupabaseAdminClient() {
  return adminClient;
}

export async function getDrizzleSupabaseClient() {
  const client = getSupabaseServerClient();
  const { data } = await client.auth.getSession();
  const accessToken = data.session?.access_token ?? '';
  const token = decode(accessToken);

  const runTransaction = ((transaction, config) => {
    return rlsClient.transaction(async (tx) => {
      try {
        // Set up Supabase auth context
        await tx.execute(sql`
          select set_config('request.jwt.claims', '${sql.raw(
            JSON.stringify(token),
          )}', TRUE);
          select set_config('request.jwt.claim.sub', '${sql.raw(
            token.sub ?? '',
          )}', TRUE);
          set local role ${sql.raw(token.role ?? 'anon')};
        `);

        return await transaction(tx);
      } finally {
        // Clean up
        await tx.execute(sql`
          select set_config('request.jwt.claims', NULL, TRUE);
          select set_config('request.jwt.claim.sub', NULL, TRUE);
          reset role;
        `);
      }
    }, config);
  }) as typeof rlsClient.transaction;

  return {
    runTransaction,
  };
}

function decode(accessToken: string) {
  try {
    return jwtDecode<JwtPayload & { role: string }>(accessToken);
  } catch {
    return { role: 'anon' } as JwtPayload & { role: string };
  }
}
```

## Step 5: Generate the Schema

Now we'll generate the Drizzle schema from your existing Supabase database:

```bash
pnpm --filter "@kit/supabase" drizzle pull
```

This will create a `schema.ts` file in your `out` directory with the TypeScript types matching your database schema.

You should see the following output:

```
Pulling from ['public'] list of schemas

Using 'postgres' driver for database querying
[✓] 14  tables fetched
[✓] 104 columns fetched
[✓] 9   enums fetched
[✓] 18  indexes fetched
[✓] 23  foreign keys fetched
[✓] 28  policies fetched
[✓] 3   check constraints fetched
[✓] 2   views fetched

[✓] Your SQL migration file ➜ src/drizzle/0000_easy_black_panther.sql 🚀
[✓] Your schema file is ready ➜ src/drizzle/schema.ts 🚀
[✓] Your relations file is ready ➜ src/drizzle/relations.ts 🚀
```

### Adding the auth schema to the schema

After generating the schema, please add the following code at the top of your `schema.ts` file:

1. Import the `pgSchema` function from Drizzle
2. Add the `auth` schema to the schema, which we need to reference the `auth.users` table in Supabase. This is required because we only pull the public schema by default, however some tables reference the `auth` schema.

```ts {% title="packages/supabase/src/drizzle/schema.ts" %}
const authSchema = pgSchema('auth');

const usersInAuth = authSchema.table('users', {
  id: uuid('id').primaryKey(),
});
```

### Disable linting on schema.ts

Since this is a generated file, we want to disable linting on it. To do so, add this line to the top of the file:

```ts {% title="packages/supabase/src/drizzle/schema.ts" %}
/* eslint-disable */
```

## Step 6: Using the Drizzle Client

Here's an example of how to use the Drizzle client in a Next.js page:

```typescript
import { use } from 'react';
import { getDrizzleSupabaseClient } from '@kit/supabase/drizzle-client';
import { accounts } from '@kit/supabase/drizzle-schema';

function MyComponent() {
  const client = use(getDrizzleSupabaseClient());

  // Example query using transactions
  const data = use(client.runTransaction((tx) => {
    return tx.select().from(accounts);
  }));

  // Rest of your component code
}
```

Alternatively, in a Server Action:

```typescript
'use server';

import { getDrizzleSupabaseClient } from '@kit/supabase/drizzle-client';
import { accounts } from '@kit/supabase/drizzle-schema';

export const getAccountsAction = enhanceAction(async (data) => {
  const client = await getDrizzleSupabaseClient();

  // Fetch accounts
  const data = await client.runTransaction((tx) => {
    return tx.select().from(accounts);
  });

  // Return the data
  return data;
},
{
  auth: true
});
```

### Using the Admin Client

If you need elevated permissions, you can use the `getDrizzleSupabaseAdminClient()` function:

```typescript
import { use } from 'react';
import { getDrizzleSupabaseAdminClient } from '@kit/supabase/drizzle-client';
import { accounts } from '@kit/supabase/drizzle-schema';

function MyComponent() {
  const client = getDrizzleSupabaseAdminClient();

  // Example query using transactions
  const data = await client.select().from(accounts);

  // Rest of your component code
}
```

## Important Notes

1. **RLS Support**: The Drizzle client respects Supabase Row Level Security (RLS) by passing the user's JWT token and role in transactions.
2. **Client Types**: There are two client types:
   - `getDrizzleSupabaseAdminClient()` - bypasses RLS (use with caution)
   - `getDrizzleSupabaseClient()` - respects RLS (use for regular user operations)
3. **Environment Variables**: Make sure to set `SUPABASE_DATABASE_URL` in your environment variables:
   ```bash
   SUPABASE_DATABASE_URL=postgres://user:pass@host:port/database
   ```

You can only use the Drizzle client in a Server environment - therefore in Server Components, Server Actions and API Routes. It will fail if you bundle it within a client-side component.

### Keep the SUPABASE_DATABASE_URL variable confidential

The variable `SUPABASE_DATABASE_URL` is required for the Drizzle client to work correctly. You will find this in the Supabase dashboard under the project settings.

{% alert type="warning" title="Do not commit the SUPABASE_DATABASE_URL variable to the repository" %}
Please keep the variable `SUPABASE_DATABASE_URL` confidential. It contains the DB credentials and should not be committed to the repository. Therefore, only add it to the `.env.development` file for development purposes containing the local DB credentials.
{% /alert %}

When in production, please only add the `SUPABASE_DATABASE_URL` variable using your hosting provider's environment variables.

## Next Steps

While the core kit will keep using the plain Supabase client, from here on you can use the Drizzle client in your application.

If you want to use the Drizzle utilities for migrations, you may need to place the `src/drizzle` folder in the root of your project and replace the path in the `drizzle.config.js` file pointing to the `apps/web` folder.data

However, please remember to keep up to date the `drizzle-schema` export in the `@kit/supabase` package, since you want to be able to use the Drizzle schema throughout the packages.

## Benefits of Using Drizzle

- Type-safe database queries
- Better developer experience with autocomplete
- SQL-like query builder
- Transaction support
- Automatic schema generation from your existing database

Please refer to the [Drizzle documentation](https://orm.drizzle.team/docs/overview) for more advanced usage and configuration options.


-----------------
FILE PATH: kits\next-supabase-turbo\recipes\onboarding-checkout.mdoc

---
status: "published"
title: 'Creating an Onboarding and Checkout flows'
label: 'Onboarding Checkout'
order: 2
description: 'Learn how to create an onboarding and checkout flow in the Next.js Supabase Starter Kit.'
---

One popular request from customers is to have a way to onboard new users and guide them through the app, and have customers checkout before they can use the app.

In this guide, we'll show you how to create an onboarding and checkout flow in the Next.js Supabase Starter Kit.

In this guide, we will cover:

1. Creating an onboarding flow when a user signs up.
2. Creating a multi-step form to have customers create a new Team Account
3. Creating a checkout flow to have customers pay before they can use the app.
4. Use Webhooks to update the user's record after they have paid.

Remember: you can customize the onboarding and checkout flow to fit your app's needs. This is a starting point, and you can build on top of it.

NB: **please make sure you have pullest the latest changes from the main branch before you start this guide.**

## Step 0: Adding an Onboarding Table

Before we create the onboarding flow, let's add a new table to store the onboarding data.

Create a new migration using the following command:

```bash
pnpm --filter web supabase migration new onboarding
```

This will create a new migration file at `apps/web/supabase/migrations/<timestamp>_onboarding.sql`. Open this file and add the following SQL code:

 ```sql {% title="apps/web/supabase/migrations/<timestamp>_onboarding.sql" %}
create table if not exists public.onboarding (
  id uuid primary key default uuid_generate_v4(),
  account_id uuid references public.accounts(id) not null unique,
  data jsonb default '{}',
  completed boolean default false,
  created_at timestamp with time zone default current_timestamp,
  updated_at timestamp with time zone default current_timestamp
);

revoke all on public.onboarding from public, service_role;

grant select, update, insert on public.onboarding to authenticated;
grant select, delete on public.onboarding to service_role;

alter table onboarding enable row level security;

create policy read_onboarding
    on public.onboarding
    for select
    to authenticated
    using (account_id = (select auth.uid()));

create policy insert_onboarding
    on public.onboarding
    for insert
    to authenticated
    with check (account_id = (select auth.uid()));

create policy update_onboarding
    on public.onboarding
    for update
    to authenticated
    using (account_id = (select auth.uid()))
    with check (account_id = (select auth.uid()));
```

This migration creates a new `onboarding` table with the following columns:
- `id`: A unique identifier for the onboarding record.
- `account_id`: A foreign key reference to the `accounts` table.
- `data`: A JSONB column to store the onboarding data.
- `completed`: A boolean flag to indicate if the onboarding is completed.
- `created_at` and `updated_at`: Timestamps for when the record was created and updated.

The migration also sets up row-level security policies to ensure that users can only access their own onboarding records.

Update your DB schema by running the following command:

```bash
pnpm run supabase:web:reset
```

And update your DB types by running the following command:

```bash
pnpm run supabase:web:typegen
```

Now that we have the `onboarding` table set up, let's create the onboarding flow.

## Step 1: Create the Onboarding Page

First, let's create the main onboarding page. This will be the entry point for our onboarding flow.

Create a new file at `apps/web/app/onboarding/page.tsx`:

 ```tsx {% title="apps/web/app/onboarding/page.tsx" %}
import { AppLogo } from '~/components/app-logo';
import { OnboardingForm } from './_components/onboarding-form';

function OnboardingPage() {
  return (
    <div className="flex h-screen flex-col items-center justify-center space-y-16">
      <AppLogo />

      <div>
        <OnboardingForm />
      </div>
    </div>
  );
}

export default OnboardingPage;
```

This page is simple. It displays your app logo and the `OnboardingForm` component, which we'll create next.

## Step 2: Create the Onboarding Form Schema

Before we create the form, let's define its schema. This will help us validate the form data.

Create a new file at `apps/web/app/onboarding/_lib/onboarding-form.schema.ts`:

 ```typescript {% title="apps/web/app/onboarding/_lib/onboarding-form.schema.ts" %}
import { z } from 'zod';

export const OnboardingFormSchema = z.object({
  profile: z.object({
    name: z.string().min(1).max(255),
  }),
  team: z.object({
    name: z.string().min(1).max(255),
  }),
  checkout: z.object({
    planId: z.string().min(1),
    productId: z.string().min(1),
  }),
});
```

This schema defines the structure of our onboarding form. It has three main sections: profile, team, and checkout.

## Step 3: Create the Onboarding Form Component

Now, let's create the main `OnboardingForm` component. This is where the magic happens!

Create a new file at `apps/web/app/onboarding/_components/onboarding-form.tsx`:

 ```tsx {% title="apps/web/app/onboarding/_components/onboarding-form.tsx" %}
'use client';

import { useCallback, useRef, useState } from 'react';

import { createPortal } from 'react-dom';

import dynamic from 'next/dynamic';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

import { PlanPicker } from '@kit/billing-gateway/components';
import { Button } from '@kit/ui/button';
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
} from '@kit/ui/form';
import { If } from '@kit/ui/if';
import { Input } from '@kit/ui/input';
import {
  MultiStepForm,
  MultiStepFormContextProvider,
  MultiStepFormHeader,
  MultiStepFormStep,
  useMultiStepFormContext,
} from '@kit/ui/multi-step-form';
import { Stepper } from '@kit/ui/stepper';

import billingConfig from '~/config/billing.config';
import { OnboardingFormSchema } from '~/onboarding/_lib/onboarding-form.schema';
import { submitOnboardingFormAction } from '~/onboarding/_lib/server/server-actions';

const EmbeddedCheckout = dynamic(
  async () => {
    const { EmbeddedCheckout } = await import('@kit/billing-gateway/checkout');

    return {
      default: EmbeddedCheckout,
    };
  },
  {
    ssr: false,
  },
);

export function OnboardingForm() {
  const [checkoutToken, setCheckoutToken] = useState<string | undefined>(
    undefined,
  );

  const form = useForm({
    resolver: zodResolver(OnboardingFormSchema),
    defaultValues: {
      profile: {
        name: '',
      },
      team: {
        name: '',
      },
      checkout: {
        planId: '',
        productId: '',
      },
    },
    mode: 'onBlur',
  });

  const onSubmit = useCallback(
    async (data: z.infer<typeof OnboardingFormSchema>) => {
      try {
        const { checkoutToken } = await submitOnboardingFormAction(data);

        setCheckoutToken(checkoutToken);
      } catch (error) {
        console.error('Failed to submit form:', error);
      }
    },
    [],
  );

  const checkoutPortalRef = useRef<HTMLDivElement>(null);

  if (checkoutToken) {
    return (
      <EmbeddedCheckout
        checkoutToken={checkoutToken}
        provider={billingConfig.provider}
        onClose={() => setCheckoutToken(undefined)}
      />
    );
  }

  return (
    <div
      className={
        'w-full rounded-lg p-8 shadow-sm duration-500 animate-in fade-in-90 zoom-in-95 slide-in-from-bottom-12 lg:border'
      }
    >
      <MultiStepForm
        className={'space-y-8 p-1'}
        schema={OnboardingFormSchema}
        form={form}
        onSubmit={onSubmit}
      >
        <MultiStepFormHeader>
          <MultiStepFormContextProvider>
            {({ currentStepIndex }) => (
              <Stepper
                variant={'numbers'}
                steps={['Profile', 'Team', 'Complete']}
                currentStep={currentStepIndex}
              />
            )}
          </MultiStepFormContextProvider>
        </MultiStepFormHeader>

        <MultiStepFormStep name={'profile'}>
          <ProfileStep />
        </MultiStepFormStep>

        <MultiStepFormStep name={'team'}>
          <TeamStep />
        </MultiStepFormStep>

        <MultiStepFormStep name={'checkout'}>
          <If condition={checkoutPortalRef.current}>
            {(portalRef) => createPortal(<CheckoutStep />, portalRef)}
          </If>
        </MultiStepFormStep>
      </MultiStepForm>

      <div className={'p-1'} ref={checkoutPortalRef}></div>
    </div>
  );
}

function ProfileStep() {
  const { nextStep, form } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex flex-col space-y-6'}>
        <div className={'flex flex-col space-y-2'}>
          <h1 className={'text-xl font-semibold'}>Welcome to Makerkit</h1>

          <p className={'text-sm text-muted-foreground'}>
            Welcome to the onboarding process! Let&apos;s get started by
            entering your name.
          </p>
        </div>

        <FormField
          render={({ field }) => {
            return (
              <FormItem>
                <FormLabel>Your Name</FormLabel>

                <FormControl>
                  <Input {...field} placeholder={'Name'} />
                </FormControl>

                <FormDescription>Enter your full name here</FormDescription>
              </FormItem>
            );
          }}
          name={'profile.name'}
        />

        <div className={'flex justify-end'}>
          <Button onClick={nextStep}>Continue</Button>
        </div>
      </div>
    </Form>
  );
}

function TeamStep() {
  const { nextStep, prevStep, form } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex w-full flex-col space-y-6'}>
        <div className={'flex flex-col space-y-2'}>
          <h1 className={'text-xl font-semibold'}>Create Your Team</h1>

          <p className={'text-sm text-muted-foreground'}>
            Let&apos;s create your team. Enter your team name below.
          </p>
        </div>

        <FormField
          render={({ field }) => {
            return (
              <FormItem>
                <FormLabel>Your Team Name</FormLabel>

                <FormControl>
                  <Input {...field} placeholder={'Name'} />
                </FormControl>

                <FormDescription>
                  This is the name of your team.
                </FormDescription>
              </FormItem>
            );
          }}
          name={'team.name'}
        />

        <div className={'flex justify-end space-x-2'}>
          <Button variant={'ghost'} onClick={prevStep}>
            Go Back
          </Button>

          <Button onClick={nextStep}>Continue</Button>
        </div>
      </div>
    </Form>
  );
}

function CheckoutStep() {
  const { form, mutation } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex w-full flex-col space-y-6 lg:min-w-[55rem]'}>
        <div className={'flex flex-col space-y-2'}>
          <PlanPicker
            pending={mutation.isPending}
            config={billingConfig}
            onSubmit={({ planId, productId }) => {
              form.setValue('checkout.planId', planId);
              form.setValue('checkout.productId', productId);

              mutation.mutate();
            }}
          />
        </div>
      </div>
    </Form>
  );
}
```

This component creates a multi-step form for the onboarding process. It includes steps for profile information, team creation, and plan selection.

## Step 4: Create the Server Action

Now, let's create the server action that will handle the form submission.

Create a new file at `apps/web/app/onboarding/_lib/server/server-actions.ts`:

 ```typescript {% title="apps/web/app/onboarding/_lib/server/server-actions.ts" %}
'use server';

import { redirect } from 'next/navigation';

import { createBillingGatewayService } from '@kit/billing-gateway';
import { enhanceAction } from '@kit/next/actions';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

import appConfig from '~/config/app.config';
import billingConfig from '~/config/billing.config';
import pathsConfig from '~/config/paths.config';
import { OnboardingFormSchema } from '~/onboarding/_lib/onboarding-form.schema';

export const submitOnboardingFormAction = enhanceAction(
  async (data, user) => {
    const logger = await getLogger();

    logger.info({ userId: user.id }, `Submitting onboarding form...`);

    const isOnboarded = user.app_metadata.onboarded === true;

    if (isOnboarded) {
      logger.info(
        { userId: user.id },
        `User is already onboarded. Redirecting...`,
      );

      redirect(pathsConfig.app.home);
    }

    const client = getSupabaseServerClient();

    const createTeamResponse = await client
      .from('accounts')
      .insert({
        name: data.team.name,
        primary_owner_user_id: user.id,
        is_personal_account: false,
      })
      .select('id')
      .single();

    if (createTeamResponse.error) {
      logger.error(
        {
          error: createTeamResponse.error,
        },
        `Failed to create team`,
      );

      throw createTeamResponse.error;
    } else {
        logger.info(
            { userId: user.id, teamId: createTeamResponse.data.id },
            `Team created. Creating onboarding data...`,
        );
    }

    const response = await client.from('onboarding').upsert(
      {
        account_id: user.id,
        data: {
          userName: data.profile.name,
          teamAccountId: createTeamResponse.data.id,
        },
        completed: true,
      },
      {
        onConflict: 'account_id',
      },
    );

    if (response.error) {
      throw response.error;
    }

    logger.info(
      { userId: user.id, teamId: createTeamResponse.data.id },
      `Onboarding data created. Creating checkout session...`,
    );

    const billingService = createBillingGatewayService(billingConfig.provider);

    const { plan } = getPlanDetails(
      data.checkout.productId,
      data.checkout.planId,
    );

    const returnUrl = new URL('/onboarding/complete', appConfig.url).href;

    const checkoutSession = await billingService.createCheckoutSession({
      returnUrl,
      customerEmail: user.email,
      accountId: createTeamResponse.data.id,
      plan,
      variantQuantities: [],
      metadata: {
        source: 'onboarding',
        userId: user.id,
      },
    });

    return {
      checkoutToken: checkoutSession.checkoutToken,
    };
  },
  {
    auth: true,
    schema: OnboardingFormSchema,
  },
);

function getPlanDetails(productId: string, planId: string) {
  const product = billingConfig.products.find(
    (product) => product.id === productId,
  );

  if (!product) {
    throw new Error('Product not found');
  }

  const plan = product?.plans.find((plan) => plan.id === planId);

  if (!plan) {
    throw new Error('Plan not found');
  }

  return { plan, product };
}
```

This server action handles the form submission, inserts the onboarding data into Supabase, create a team (so we can assign it a subscription), and creates a checkout session for the selected plan.

Once the checkout is completed, the user will be redirected to the `/onboarding/complete` page. This page will be created in the next step.

## Step 6: Enhancing the Stripe Webhook Handler

This change extends the functionality of the Stripe webhook handler to complete the onboarding process after a successful checkout. Here's what's happening:

In the `handleCheckoutSessionCompleted` method, we add new logic to handle onboarding-specific actions.

First, define the `completeOnboarding` function to process the onboarding data and create a team based on the user's input.

NB: this is valid for Stripe, but you can adapt it to any other payment provider.

 ```typescript {% title="packages/billing/stripe/src/services/stripe-webhook-handler.service.ts" %}
async function completeOnboarding(accountId: string) {
  const logger = await getLogger();
  const adminClient = getSupabaseServerAdminClient();

  logger.info(
    { accountId },
    `Checkout comes from onboarding. Processing onboarding data...`,
  );

  const onboarding = await adminClient
    .from('onboarding')
    .select('*')
    .eq('account_id', accountId)
    .single();

  if (onboarding.error) {
    logger.error(
      { error: onboarding.error, accountId },
      `Failed to retrieve onboarding data`,
    );

    // if there's an error, we can't continue
    return;
  } else {
    logger.info({ accountId }, `Onboarding data retrieved. Processing...`);

    const data = onboarding.data.data as {
      userName: string;
      teamAccountId: string;
    };

    const teamAccountId = data.teamAccountId;

    logger.info(
      { userId: accountId, teamAccountId },
      `Assigning membership...`,
    );

    const assignMembershipResponse = await adminClient
      .from('accounts_memberships')
      .insert({
        account_id: teamAccountId,
        user_id: accountId,
        account_role: 'owner',
      });

    if (assignMembershipResponse.error) {
      logger.error(
        {
          error: assignMembershipResponse.error,
        },
        `Failed to assign membership`,
      );
    } else {
      logger.info({ accountId }, `Membership assigned. Updating account...`);
    }

    const accountResponse = await adminClient
      .from('accounts')
      .update({
        name: data.userName,
      })
      .eq('id', accountId);

    if (accountResponse.error) {
      logger.error(
        {
          error: accountResponse.error,
        },
        `Failed to update account`,
      );
    } else {
      logger.info(
        { accountId },
        `Account updated. Cleaning up onboarding data...`,
      );
    }

    // set onboarded flag on user account
    const updateUserResponse = await adminClient.auth.admin.updateUserById(
      accountId,
      {
        app_metadata: {
          onboarded: true,
        },
      },
    );

    if (updateUserResponse.error) {
      logger.error(
        {
          error: updateUserResponse.error,
        },
        `Failed to update user`,
      );
    } else {
      logger.info({ accountId }, `User updated. Cleaning up...`);
    }

    // clean up onboarding data
    const deleteOnboardingResponse = await adminClient
      .from('onboarding')
      .delete()
      .eq('account_id', accountId);

    if (deleteOnboardingResponse.error) {
      logger.error(
        {
          error: deleteOnboardingResponse.error,
        },
        `Failed to delete onboarding data`,
      );
    } else {
      logger.info(
        { accountId },
        `Onboarding data cleaned up. Completed webhook handler.`,
      );
    }
  }
}
```

Now, we handle this function in the `handleCheckoutSessionCompleted` method, right before the `onCheckoutCompletedCallback` is called.

 ```typescript {% title="packages/billing/stripe/src/services/stripe-webhook-handler.service.ts" %}
// ...

const subscriptionData =
  await stripe.subscriptions.retrieve(subscriptionId);

const metadata = subscriptionData.metadata as {
  source: string;
  userId: string;
} | undefined;

// if the checkout comes from onboarding
// we need to complete the onboarding process
if (metadata?.source === 'onboarding') {
  const userId = metadata.userId;

  await completeOnboarding(userId);
}

return onCheckoutCompletedCallback(payload);
```

This enhanced webhook handler completes the onboarding process by creating a team account, updating the user's personal account, and marking the user as "onboarded" in their Supabase user metadata.

## Step 7: Handling the Onboarding Completion Page

The checkout will rediret to the `/onboarding/complete` page after the onboarding process is completed. This is because there is no telling when the webhook will be triggered.

In this page, we will start fetching the user data and verify if the user has been onboarded.

If the user has been onboarded correctly, we will redirect the user to the `/home` page. If not, we will show a loading spinner and keep checking the user's onboarded status until it is true.

 ```tsx {% title="/apps/web/app/onboarding/complete/page.tsx" %}
'use client';

import { useRef } from 'react';

import { useQuery } from '@tanstack/react-query';

import { useSupabase } from '@kit/supabase/hooks/use-supabase';
import { LoadingOverlay } from '@kit/ui/loading-overlay';

import pathsConfig from '~/config/paths.config';

export default function OnboardingCompletePage() {
  const { error } = useCheckUserOnboarded();

  if (error) {
    return (
      <div className={'flex flex-col items-center justify-center'}>
        <p>Something went wrong...</p>
      </div>
    );
  }

  return <LoadingOverlay>Setting up your account...</LoadingOverlay>;
}

/**
 * @description
 * This function checks if the user is onboarded
 * If the user is onboarded, it redirects them to the home page
 * it retries every second until the user is onboarded
 */
function useCheckUserOnboarded() {
  const client = useSupabase();
  const countRef = useRef(0);
  const maxCount = 10;
  const error = countRef.current >= maxCount;

  useQuery({
    queryKey: ['onboarding-complete'],
    refetchInterval: () => (error ? false : 1000),
    queryFn: async () => {
      if (error) {
        return false;
      }

      countRef.current++;

      const response = await client.auth.getUser();

      if (response.error) {
        throw response.error;
      }

      const onboarded = response.data.user.app_metadata.onboarded;

      // if the user is onboarded, redirect them to the home page
      if (onboarded) {
        return window.location.assign(pathsConfig.app.home);
      }

      return false;
    },
  });

  return {
    error,
  };
}
```

This page

## What This Means for Your Onboarding Flow

With these changes, your onboarding process now includes these additional steps:

1. When a user completes the checkout during onboarding, it triggers this enhanced webhook handler.
2. The handler retrieves the onboarding data that was saved earlier in the process.
3. It creates a new team account with the name provided during onboarding.
4. It updates the user's personal account with their name.
5. Finally, it marks the user as "onboarded" in their Supabase user metadata.

This completes the onboarding process, ensuring that all the information collected during onboarding is properly saved and the user's account is fully set up.

## Step 8: Update Your App's Routing

To integrate this onboarding flow into your app, you'll need to update your routing logic.

We can do this in the middleware, in the logic branch that handles the `/home` route. If the user is not logged in, we'll redirect them to the sign-in page. If the user is logged in but has not completed onboarding, we'll redirect them to the onboarding flow.

Update the `apps/web/app/middleware.ts` file:

 ```typescript {% title="apps/web/app/middleware.ts" %}
{
  pattern: new URLPattern({ pathname: '/home/*?' }),
  handler: async (req: NextRequest, res: NextResponse) => {
    const {
      data: { user },
    } = await getUser(req, res);

    const origin = req.nextUrl.origin;
    const next = req.nextUrl.pathname;

    // If user is not logged in, redirect to sign in page.
    if (!user) {
      const signIn = pathsConfig.auth.signIn;
      const redirectPath = `${signIn}?next=${next}`;

      return NextResponse.redirect(new URL(redirectPath, origin).href);
    }

    // verify if user has completed onboarding
    const isOnboarded = user?.app_metadata.onboarded;

    // If user is logged in but has not completed onboarding,
    if (!isOnboarded) {
      return NextResponse.redirect(new URL('/onboarding', origin).href);
    }

    const supabase = createMiddlewareClient(req, res);

    const requiresMultiFactorAuthentication =
      await checkRequiresMultiFactorAuthentication(supabase);

    // If user requires multi-factor authentication, redirect to MFA page.
    if (requiresMultiFactorAuthentication) {
      return NextResponse.redirect(
        new URL(pathsConfig.auth.verifyMfa, origin).href,
      );
    }
  },
}
```

This code checks if the user is logged in and has completed onboarding. If the user has not completed onboarding, they are redirected to the onboarding flow.

Also add the following pattern to make sure the user is authenticated when visiting the `/onboarding` route:

```typescript {% title="apps/web/app/middleware.ts" %}
{
  pattern: new URLPattern({ pathname: '/onboarding/*?' }),
  handler: async (req: NextRequest, res: NextResponse) => {
    const {
      data: { user },
    } = await getUser(req, res);

    // If user is not logged in, redirect to sign in page.
    if (!user) {
      const signIn = pathsConfig.auth.signIn;
      const redirectPath = `${signIn}?next=${next}`;

      return NextResponse.redirect(new URL(redirectPath, origin).href);
    }

    const supabase = createMiddlewareClient(req, res);

    const requiresMultiFactorAuthentication =
      await checkRequiresMultiFactorAuthentication(supabase);

    // If user requires multi-factor authentication, redirect to MFA page.
    if (requiresMultiFactorAuthentication) {
      return NextResponse.redirect(
        new URL(pathsConfig.auth.verifyMfa, origin).href,
      );
    }
  },
}
```

### Marking invited users as onboarded

When a user gets invited to a team account, you don't want to show them the onboarding flow again. You can use the `onboarded` property to mark the user as onboarded.

Update the `packages/features/team-accounts/src/server/actions/team-invitations-server-actions.ts` server action `acceptInvitationAction` at line 125 (eg. before increasing seats):

```tsx
// mark user as onboarded
await adminClient.auth.admin.updateUserById(user.id, {
  app_metadata: {
    onboarded: true,
  },
});
```

In this way, the user will be redirected to the `/home` page after accepting the invite.

### Considerations

Remember, the user can always unsubscribe from the plan they selected during onboarding. You should handle this case in your billing system if your app always requires a plan to be active.

## Conclusion

That's it! You've now added an onboarding flow to your Makerkit Next.js Supabase Turbo project.

This flow includes:

1. A profile information step
2. A team creation step
3. A plan selection step
4. Integration with your billing system

Remember to style your components, handle errors gracefully, and test thoroughly. Happy coding! 🚀


