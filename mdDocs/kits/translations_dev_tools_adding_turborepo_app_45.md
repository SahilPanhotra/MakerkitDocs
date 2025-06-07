-----------------
FILE PATH: kits\next-supabase-turbo\dev-tools\translations.mdoc

---
status: "published"
description: "The Next.js Supabase Turbo Dev Tools allows you to edit translations and use AI to translate them."
title: "Translations Editor"
label: "Translations Editor"
order: 1
---

The Translations Editor is a tool that allows you to edit translations and use AI to translate them.

It's a simple editor that allows you to edit translations for your project.

{% img src="/assets/images/dev-tools-translations.webp" width="1000"
height="600" /%}

## Set Open AI API Key

First, you need to set the Open AI API Key in the `apps/e2e/.env.local` file.

```bash apps/e2e/.env.local
OPENAI_API_KEY=your-openai-api-key
```

Either make sure your key has access to the `gpt-4o-mini` model or set the `LLM_MODEL_NAME` environment variable to whichever model you have access to.

## Adding a new language

First, you need to add the language to the `apps/web/lib/i18n.settings.ts` file as described in the [Adding Translations](/docs/next-supabase-turbo/translations/adding-translations) documentation.

## Generate Translations with AI

The Translations Editor allows you to generate translations with AI.

You can use the AI to translate the translations for you by clicking the "Translate missing with AI" button.

## Editing Translations

Every time you update a translation, it will be saved automatically to the same file it's defined in.



-----------------
FILE PATH: kits\next-supabase-turbo\dev-tools.mdoc

---
status: "published"
title: "Developer Tools | Next.js Supabase Turbo"
label: "Developer Tools"
order: 17
description: "Using the Developer Tools to solve common issues in your application"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits\next-supabase-turbo\development\adding-turborepo-app.mdoc

---
status: "published"
label: "Adding a Turborepo app"
title: "Add a new Turborepo application to your Makerkit application | Next.js Supabase"
description: "Learn how to add a new Turborepo application to your Makerkit application"
order: 12
---

This is an **advanced topic** - you should only follow these instructions if you are placing a new app within your monorepo and want to keep pulling updates from the Makerkit repository.

In some ways - creating a new repository may be the easiest way to manage your application. However, if you want to keep your application within the monorepo and pull updates from the Makerkit repository, you can follow these instructions.

{% sequence title="How to add a new Turborepo application to your Makerkit application" description="Learn how to add a new Turborepo application to your Makerkit application" %}

[Creating a subtree](#creating-a-subtree)

[Creating a new application](#creating-a-new-application)

[Updating the new application](#updating-the-new-application)

{% /sequence %}

To pull updates into a separate application outside of `web` - we can use `git subtree`.

Basically, we will create a subtree at `apps/web` and create a new remote branch for the subtree. When we create a new application, we will pull the subtree into the new application. This allows us to keep it in sync with the `apps/web` folder.

### 1. Create a subtree

First, we need to create a subtree for the `apps/web` folder. We will create a new branch called `web` and create a subtree for the `apps/web` folder. We will create a branch named `web-branch` and create a subtree for the `apps/web` folder.

```bash
git subtree split --prefix=apps/web --branch web-branch
```

### 2. Create a new application

Now, we can create a new application in the `apps` folder. For example, let's create a new application called `api`.

Let's say we want to create a new app `pdf-chat` at `apps/pdf-chat` with the same structure as the `apps/web` folder (which acts as the template for all new apps).

```bash
git subtree add --prefix=apps/pdf-chat origin web-branch --squash
```

You should now be able to see the `apps/pdf-chat` folder with the contents of the `apps/web` folder.

### 3. Update the new application

When you want to update the new application, follow these steps:

#### Pull updates from the Makerkit repository

The command below will update all the changes from the Makerkit repository:

```bash
git pull upstream main
```

#### Push the web-branch updates

After you have pulled the updates from the Makerkit repository, you can split the branch again and push the updates to the `web-branch`:

```bash
git subtree split --prefix=apps/web --branch web-branch
```

Now, you can push the updates to the `web-branch`:

```bash
git push origin web-branch
```

#### Pull the updates to the new application

Now, you can pull the updates to the new application:

```bash
git subtree pull --prefix=apps/pdf-chat origin web-branch --squash
```


-----------------
FILE PATH: kits\next-supabase-turbo\development\adding-turborepo-package.mdoc

---
status: "published"
label: "Adding a Turborepo package"
title: "Add a new Turborepo package to your Makerkit application"
description: "Learn how to add a new Turborepo package to your Makerkit application"
order: 11
---

This is an **advanced topic** - you should only follow these instructions if you are sure you want to add a new package to your Makerkit application instead of adding a folder to your application in `apps/web`. You don't need to do this to add a new page or component to your application.

{% sequence title="How to add a new Turborepo package to your Makerkit application" description="Learn how to add a new Turborepo package to your Makerkit application" %}

[Adding a new package to your Makerkit application](#adding-a-new-package-to-your-makerkit-application)

[Adding dependencies to your package](#adding-dependencies-to-your-package)

[Exporting a module from a package](#exporting-a-module-from-a-package)

[Adding a new package to the next.config.mjs file](#adding-a-new-package-to-the-nextconfigmjs-file)

{% /sequence %}

To add a new package to your Makerkit application, you need to follow these steps.

First, enter the command below to create a new package in your Makerkit application:

```bash
turbo gen
```

Turborepo will ask you to enter the name of the package you want to create. Enter the name of the package you want to create and press enter.

If you don't want to add dependencies to your package, you can skip this step by pressing enter.

The command will have generated a new package under `packages` named `@kit/<package-name>`. If you named it `my-package`, the package will be named `@kit/my-package`.

Finally, to make fast refresh work when you make changes to the package, you need to add the package to the `next.config.mjs` file in the root of your Makerkit application `apps/web`.

```tsx
const INTERNAL_PACKAGES = [
  // all internal packages,
  '@kit/my-package',
];
```

## Exporting a module from a package

By default, the package exports a single module using the `index.ts` file. You can add more exports by creating new files in the package directory and exporting them from the `index.ts` file or creating export files in the package directory and adding them to the `exports` field in the `package.json` file.

### Exporting a module from index.ts

The easiest way to export a module from a package is to create a new file in the package directory and export it from the `index.ts` file.

```tsx
// packages/@kit/my-package/src/my-module.ts

export function myModule() {
  return 'Hello from my module';
}
```

Then, export the module from the `index.ts` file.

```tsx
// packages/@kit/my-package/src/index.ts
export * from './my-module';
```

### Exporting using the exports field in package.json

This can be very useful for tree-shaking. Assuming you have a file named `my-module.ts` in the package directory, you can export it by adding it to the `exports` field in the `package.json` file.

```json
{
  "exports": {
    ".": "./src/index.ts",
    "./my-module": "./src/my-module.ts"
  }
}
```

When to do this?

1. when exporting two modules that don't share dependencies to ensure better tree-shaking. For example, if your exports contains both client and server modules.
2. for better organization of your package


For example, create two exports `client` and `server` in the package directory and add them to the `exports` field in the `package.json` file.

```json
{
  "exports": {
    ".": "./src/index.ts",
    "./client": "./src/client.ts",
    "./server": "./src/server.ts"
  }
}
```

1. The `client` module can be imported using `import { client } from '@kit/my-package/client'`
2. The `server` module can be imported using `import { server } from '@kit/my-package/server'`

You can now use the package in your application by importing it using the package name.

```tsx
import { myModule } from '@kit/my-package';

console.log(myModule());
```

Et voilà! You have successfully added a new package to your Makerkit application. 🎉


-----------------
FILE PATH: kits\next-supabase-turbo\development\approaching-local-development.mdoc

---
status: "published"
label: "Starting developing your App"
order: 0
title: "How to approach local development | Next.js Supabase Turbo"
description: "Learn how to approach local development in your Next.js Supabase Turbo project"
---

In the previous sections, you learned how to clone the repository, set environment variables, customize the look and feel of the app, and some basic API that you'll be using throughout the app. 

In this section, you'll learn how to start developing your app locally.

Buckle up, we have a lot to do!

---

Generally speaking, you will be doing the following:

1. **Customization**: Set environment variables (application name, feature flags). This is a quick starting point that will start turning Makerkit into your very own app.
2. **Database**: Drawing and writing the database schema. Of course, your app will store data and have its schema. It's time to draw it.
3. **Routing**: Adding new routes. Your new pages will need routes - unless you can reuse the "home" pages of the accounts.
4. **Fetching Data**: Fetching data from your DB and displaying it onto the new routes.
5. **Writing Data** Adding new forms. You will need to add forms to create new data.

In 90% of cases, the above is what you will be doing. The remaining 10% is adding new (very specific) features, which are a bit more complex - and not relevant to Makerkit itself.

-----------------
FILE PATH: kits\next-supabase-turbo\development\database-functions.mdoc

---
status: "published"
label: "Database Functions"
order: 3
title: "Database Functions available in your Next.js Supabase schema"
description: "Learn the most useful database functions in your schema"
---

The database schema contains several functions that you can use so that you can extend your database with custom logic and RLS.

{% sequence title="How to use Database Functions" description="Learn how to use the functions in your schema" %}

[How to use Database Functions](#how-to-use-database-functions)

[Check if a user is the Owner of an Account](#check-if-a-user-is-the-owner-of-an-account)

[Check if a user is a Member of an Account](#check-if-a-user-is-a-member-of-an-account)

[Check if a user is a team member of an Account](#check-if-a-user-is-a-team-member-of-an-account)

[Check if an account has permissions to action another account](#check-if-an-account-has-permissions-to-action-another-account)

[Check Permissions](#check-permissions)

[Check if a configuration is set](#check-if-a-configuration-is-set)

[Check if an account has an active subscription](#check-if-an-account-has-an-active-subscription)

[Check if an account is a Super Admin](#check-if-an-account-is-a-super-admin)

[Check if an account is a MFA Compliant](#check-if-an-account-is-a-mfa-compliant)

[Check if an account signed in with MFA](#check-if-an-account-signed-in-with-mfa)

{% /sequence %}

## How to use Database Functions

To call the functions below, you can either:
1. **SQL** - Use them directly in your SQL schema
2. **RPC** - Use them using the Supabase's client RPC API

### Example of calling a function from SQL

```sql
select public.is_account_owner(account_id) from accounts;
```

### Example of calling a function from RPC

```tsx
const { data, error } = await supabase.rpc('is_account_owner', { account_id: '123' });
```


## Check if a user is the Owner of an Account

This function checks if the user is the owner of an account. It is used in the `accounts` table to restrict access to the account owner.

```sql
public.is_account_owner(account_id uuid)
```

This is `true` if the account is the user's account or if the user created a team account.

## Check if a user is a Member of an Account

This function checks if the user is a member of an account. It is used in the `accounts` table to restrict access to the account members.

```sql
public.has_role_on_account(
  account_id uuid,
  account_role varchar(50) default null
)
```

If the `account_role` is not provided, it will return `true` if the user is a member of the account. If the `account_role` is provided, it will return `true` if the user has the specified role on the account.

## Check if a user is a team member of an Account

Check if a user is a member of a team account. It is used in the `accounts` table to restrict access to the team members.

```sql
public.is_team_member(
  account_id uuid,
  user_id uuid
)
```

### Check if an account has permissions to action another account

This function checks if an account has permissions to action another account. It is used in the `accounts` table to restrict access to the account owner.

```sql
public.can_action_account_member(
  target_team_account_id uuid,
  target_user_id uuid
)
```

This function will:

1. check if the current user is the owner of the target account: if so, return `true`
2. check if the target user is the owner of the target account: if so, return `false`
3. compare the hierarchy of the roles between the two accounts: if the current user has a higher role than the target user, return `true`

This is useful to check if a user can remove another user from an account, or any other action that requires permissions.

### Check Permissions

Check if a user has a specific permission on an account.

```sql
public.has_permission(
  user_id uuid,
  account_id uuid,
  permission_name app_permissions
)
```

This function will return `true` if the user has the specified permission on the account.

The permissions are specified in the enum `app_permissions`. You can extend this enum to add more permissions.

### Check if a configuration is set

Check if a configuration is set in the `public.config` table.

```sql
public.is_set(
  field_name text
)
```

### Check if an account has an active subscription

Check if an account has an active subscription.

```sql
public.has_active_subscription(
  account_id uuid
)
```

This means that the subscription status is either `active` or `trialing`. In other words, the account billing is in good standing (eg. no unpaid invoice)

This is important because just checking if the subscription exists is not enough. You need to check if the subscription is active, as the status can vary (eg. `canceled`, `incomplete`, `incomplete_expired`, `past_due`, `unpaid`)

### Check if an account is a Super Admin

Check if an account is a Super Admin.

```sql
public.is_super_admin()
```

The functions requires:
- The user is authenticated
- The user has the `super_admin` role
- The user is currently signed in with MFA

### Check if an account is a MFA Compliant

Check if an account is a MFA Compliant - eg:

1. The user enabled MFA and is currently signed in with MFA
2. The user did not enable MFA and is currently signed in with AAL1

```sql
public.is_mfa_compliant()
```

This is useful for ensuring that:
1. Users with MFA comply with the MFA policy
2. Users without MFA can continue to access the application without MFA

Use the below `public.is_aal2` function to force a user to use MFA instead.

### Check if an account signed in with MFA

Check if an account signed in with MFA (AAL2) or not. This is useful when you want to restrict access to certain tables based on whether the user is signed in with MFA or not.

```sql
public.is_aal2()
```

-----------------
FILE PATH: kits\next-supabase-turbo\development\database-schema.mdoc

---
status: "published"
label: "Database Schema"
order: 2
title: "How to create new migrations and update the database schema in your Next.js Supabase application"
description: "Learn how to create new migrations and update the database schema in your Next.js Supabase application"
---

{% sequence title="Steps to create a new migration" description="Learn how to create new migrations and update the database schema in your Next.js Supabase application" %}

[Permissions](#permissions)

[RLS Policies](#rls-policies)

[Enabling MFA compliance](#enable-mfa-compliance)

[Enabling access to Super Admins only](#enable-access-to-super-admins-only)

[Resetting Migrations](#resetting-migrations)

{% /sequence %}

After creating your migration, it's time to add the required code.

In our example, we create the schema for a simple tasks application.

### Permissions

Makerkit defines a set of default permissions in an enum named `public.app_permissions`.

To add more permissions for your app, please update the enum:

```sql
-- insert new permissions
alter type public.app_permissions add value 'tasks.write';
alter type public.app_permissions add value 'tasks.delete';
commit;
```

In the case above, we added the permissions `tasks.write` and `tasks.delete`. We can use these in our RLS rules to make sure
the permissions are able to restrict access.

### Tasks Table

Let's now add the new `tasks` table

```sql
-- create tasks table
create table if not exists public.tasks (
  id uuid primary key default gen_random_uuid(),
  title varchar(500) not null,
  description varchar(50000),
  done boolean not null default false,
  account_id uuid not null references public.accounts(id),
  created_at timestamp with time zone not null default now(),
  updated_at timestamp with time zone not null default now()
);

grant select, insert, update, delete on table public.tasks to
    authenticated, service_role;
```

Let's explain:
1. `uuid` is a primary key generated automatically
2. `title` is a text field constrained to 500 chars. `not null` makes sure it cannot be null.
3. `description` is a text field constrained to 50000
4. `done` is a boolean field
5. `account_id` is the account that owns the task

We then add the required permissions to the roles `authenticated` and `service_role`.

Anonymous users have no access to this table.

### Accounts

Accounts are the primary entities of our schema. An account can be a user or a team.

We can connect an entity to the account that owns it using a foreign key

```sql
account_id uuid not null references public.accounts(id)
```

### Enabling RLS

When you create a new table, always enable RLS.

```sql
-- enable row level security
alter table tasks enable row level security;
```

## RLS Policies

RLS Policies are fundamental to protect our tables.

We insert an RLS policy for each action: `select`, `insert`, `update` and `delete`.

### Enable RLS for a table

In the majority of cases, you want to enable RLS for a table after creating it:

```sql
alter table public.tasks enable row level security;
```

### Selecting Tasks

When writing an RLS policy for selecting data from the `tasks` table, we need to make sure the user is the owner of the task or has the required permission.

Generally speaking, entities belong to `public.accounts` - and we can use the `account_id` to check if the user is the owner.

```sql
create policy select_tasks on public.tasks
    for select
    to authenticated
    using (
      account_id = (select auth.uid()) or
      public.has_role_on_account(account_id)
    );
```

Did you know that an account can be a user or a team? We can use the `public.has_role_on_account` function to check if the user has a role on the account.

Therefore, this RLS policy works in both ways:

1. if the user is the owner of the task - we check that the `account_id` is equal to the `auth.uid()`
2. if the user has a role on the account - we check that the user has a role on the account

### Inserting Tasks

When writing an RLS policy for inserting data into the `tasks` table, we need to make sure the user is the owner of the task or has the required permission.

```sql
create policy insert_tasks on public.tasks
    for insert
    with check (
        account_id = (select auth.uid()) or
        public.has_permission(auth.uid(), account_id, 'tasks.write'::app_permissions)
    );
```

In th above, we check if the user is the owner of the task or has the `tasks.write` permission.

1. If the `account_id` is equal to the `auth.uid()` - the account is personal - so permissions are not required
2. If the user has the `tasks.write` permission - the user can insert the task

### Updating Tasks

When writing an RLS policy for updating data in the `tasks` table, we need to make sure the user is the owner of the task or has the required permission.

```sql
create policy update_tasks on public.tasks
    for update
    using (
        account_id = (select auth.uid()) or
        public.has_permission(auth.uid(), account_id, 'tasks.write'::app_permissions)
    )
    with check (
        account_id = (select auth.uid()) or
        public.has_permission(auth.uid(), account_id, 'tasks.write'::app_permissions)
    );
```

Did you know that we need to add the `using` and `with check` clauses?

- `using` is used to filter the rows that the user can update
- `with check` is used to check if the user can update the row

In the above, we check if the user is the owner of the task or has the `tasks.write` permission.

### Deleting Tasks

When writing an RLS policy for deleting data from the `tasks` table, we need to make sure the user is the owner of the task or has the required permission.

```sql
create policy delete_tasks on public.tasks
    for delete
    using (
        account_id = (select auth.uid()) or
        public.has_permission(auth.uid(), account_id, 'tasks.delete'::app_permissions)
    );
```

In the above, we check if the user is the owner of the task or has the `tasks.delete` permission.

Our schema is now complete! Yay! 🎉

## Enable MFA compliance

We can use the function `public.is_mfa_compliant` to enforce MFA compliance for our application.

```sql
create policy restrict_mfa_tasks
    on public.tasks
    as restrictive
    to authenticated
    using (public.is_mfa_compliant());
```

In the above, we restrict access to the `tasks` table to users who are MFA compliant. Uses who enabled MFA must be signed in with MFA in order to access the table.

Instead, users who did not enable MFA can keep accessing the table without MFA.

## Enable access to Super Admins only

We can use the function `public.is_super_admin` to restrict access to Super Admins only.

We assume that we're protecting a table named `logs` and allow `select` access to Super Admins only.

```sql
create policy restrict_logs_super_admins
    on public.logs
    as restrictive
    for select
    to authenticated
    using (public.is_super_admin());
```

- By using the `public.is_super_admin` function, we ensure that only Super Admins can access the `logs` table.
- By using the `restrictive` policy, we ensure that this access restriction is enforced, irrespective of other policies.

## Resetting Migrations

When adding a new schema, we need to reset the migrations.

```bash
pnpm run supabase:web:reset
```

Then, we generate the new types using the following command:

```bash
pnpm run supabase:web:typegen
```

You can now use the new types in your application when using the Supabase client.


-----------------
FILE PATH: kits\next-supabase-turbo\development\database-webhooks.mdoc

---
status: "published"
label: "Database Webhooks"
order: 6
title: "How to handle custom database webhooks in your Next.js Supabase application"
description: "Learn how to handle custom database webhooks in your Next.js Supabase application"
---

{% sequence title="How to handle custom database webhooks" description="Learn how to handle custom database webhooks in your Next.js Supabase application" %}

[What are Database Webhooks?](#what-are-database-webhooks)

[How to handle custom database webhooks](#how-to-handle-custom-database-webhooks)

{% /sequence %}

## What are Database Webhooks?

Database webhooks allow you to listen to changes in your database and trigger custom logic when a change occurs. This is useful for sending notifications, updating caches, or triggering other services.

Makerkit handles some webhooks by default for functionalities such as deleting a subscription following a user deletion, or sending emails after a user signs up.

## How to handle custom database webhooks

You can extend this functionality by adding your own webhook handlers:

 ```tsx {% title="apps/web/app/api/db/webhook/route.ts" %}
import {
  getDatabaseWebhookHandlerService,
} from '@kit/database-webhooks';

/**
 * @name POST
 * @description POST handler for the webhook route that handles the webhook event
 */
export async function POST(request: Request) {
  const service = getDatabaseWebhookHandlerService();

  try {
    // handle the webhook event
    await service.handleWebhook(request, {
      handleEvent(change) {
        if (change.type === 'INSERT' && change.table === 'invitations') {
          // do something with the invitation
        }
      },
    });

    return new Response(null, { status: 200 });
  } catch {
    return new Response(null, { status: 500 });
  }
}
```

As you can see above - the `handleEvent` function is where you can add your custom logic to handle the webhook event. In this example, we check if the event is an `INSERT` event on the `invitations` table and then do something with the invitation.

The `change` object is of type `RecordChange` and contains the following properties:

```tsx
import { Database } from '@kit/supabase/database';

export type Tables = Database['public']['Tables'];

export type TableChangeType = 'INSERT' | 'UPDATE' | 'DELETE';

export interface RecordChange<
  Table extends keyof Tables,
  Row = Tables[Table]['Row'],
> {
  type: TableChangeType;
  table: Table;
  record: Row;
  schema: 'public';
  old_record: null | Row;
}
```

You may need to cast the type manually:

```tsx
type AccountChange = RecordChange<'accounts'>;
```

Now, the `AccountChange` type can be used to type the `change` object in the `handleEvent` function and is typed to the `accounts` table.

-----------------
FILE PATH: kits\next-supabase-turbo\development\external-marketing-website.mdoc

---
status: "published"

label: "External Marketing Website"
title: "External Marketing Website in the Next.js Supabase Turbo Starter Kit"
description: "Learn how to configure Makerkit to work with an external marketing website in the Next.js Supabase Turbo Starter Kit."
order: 9
---


Many teams prefer to create an external marketing website for their SaaS application. This allows them to have more control over the design and content of the website. For example, using services such as Framer, Webflow, or Wordpress.

In this case, we have to redirect all marketing pages to the external marketing website. You can do so tweaking the middleware in the `apps/web/middleware.ts` file.

Take the list of all your marketing pages, and then add a middleware to redirect all those pages to the external marketing website.

```tsx
import { NextRequest, NextResponse } from 'next/server';

export function middleware(req: NextRequest) {
  if (isMarketingPage(req)) {
    return NextResponse.redirect('https://your-external-website.com' + req.nextUrl.pathname);
  }

  // leave the rest of the middleware unchanged
}

function isMarketingPage(req: NextRequest) {
  const marketingPages = [
    '/pricing',
    '/faq',
    '/contact',
    '/about',
    '/home',
    '/privacy-policy',
    '/terms-and-conditions',
    '/cookie-policy',
  ];

  return marketingPages.includes(req.nextUrl.pathname);
}
```

Should you add a new marketing page, you need to update the `isMarketingPage` function with the new page path.

