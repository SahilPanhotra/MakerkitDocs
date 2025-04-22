-----------------
FILE PATH: kits\react-router-supabase-turbo\data-fetching\supabase-clients.mdoc

---
title: "Using the Supabase Clients in React Router Supabase Turbo"
label: "Supabase Clients"
description: "Learn how to use the Supabase Clients in React Router Supabase Turbo"
position: 0
---

Before diving into the various ways we can communicate with the server, we need to introduce how we communicate with Supabase, which is hosting the database and therefore the source of our data.

Depending on where your code runs, you may need to use different clients to interact with Supabase. This is due to how cookies are set differently in various parts of the React Router application.

You can use 3 different clients to interact with Supabase:

1. **Browser** - This runs in the browser and is used to interact with Supabase from the client
2. **Server** - This runs in the server and is used to interact with Supabase from the server
3. **Server Admin** - This runs in the server and is used to interact with Supabase from the server, but with admin privileges

## Browser

To use the browser client, use the `useSupabase` hook:

```tsx
import { useSupabase } from '@kit/supabase/hooks/use-supabase';

export default function Home() {
  const supabase = useSupabase()

  return (
    <div>
      <h1>Supabase Browser Client</h1>
      <button onClick={() => supabase.auth.signOut()}>Sign Out</button>
    </div>
  )
}
```

## Server Client

To use the server actions client, use the `useSupabase` hook:

```tsx
import { LoaderFunctionArgs } from 'react-router';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const supabase = getSupabaseServerClient()

  const { data, error } = await supabase.from('tasks').select('*');

  return {
    data,
  };
}
```

## Server Admin Client

To use the Server Admin client, use the `getSupabaseServerAdminClient` function:

```tsx
import { LoaderFunctionArgs } from 'react-router';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

export async function loader(args: LoaderFunctionArgs) {
  const supabase = getSupabaseServerAdminClient()

  const { data, error } = await supabase.from('tasks').select('*');

  return {
    data,
  };
}
```

NB: only use this client when you need to perform admin operations, such as bypassing row-level security. Do not use this client for regular operations.

-----------------
FILE PATH: kits\react-router-supabase-turbo\data-fetching.mdoc

---
status: "published"
title: "Data Fetching in React Router Supabase Turbo"
label: "Data Fetching"
order: 5
description: "Data Fetching in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Here's a draft of the Data Fetching pillar page for Makerkit:

# Data Fetching in Makerkit

Makerkit provides a robust approach to fetching and managing data in your React Router Supabase application. Whether you're building server components, handling real-time updates, or managing client-side state, Makerkit offers patterns and utilities to make data fetching clean and efficient.

## Data Loading Patterns

Common patterns for loading data include:

- Server Component data fetching for initial page loads
- React Query for client-side data management
- Server Actions for mutations
- CSRF and Captcha protection for API Routes


-----------------
FILE PATH: kits\react-router-supabase-turbo\development\adding-pages.mdoc

---
status: "published"
label: "Adding Pages"
title: "Adding Pages in the React Router Supabase Turbo Starter Kit"
description: "Learn how to create new pages in the React Router Supabase Turbo Starter Kit."
order: 8
---

Adding new pages to your React Router Supabase Turbo Starter Kit application involves understanding the routing structure and following some conventions. This guide will walk you through creating new pages in both the marketing section and authenticated application areas.

## Understanding the Router Structure

The starter kit uses React Router and organizes routes in a file-based structure. Looking at the `apps/web/app/routes.ts` file, you can see how routes are grouped:

- `rootRoutes`: Static utility routes like robots.txt, sitemap, etc.
- `apiRoutes`: Backend API endpoints
- `marketingLayout`: Public-facing pages (homepage, blog, etc.)
- `authLayout`: Authentication-related pages
- `adminLayout`: Admin portal pages
- `userAccountLayout`: User personal account pages
- `teamAccountLayout`: Team account pages

## Route Configuration Format

Routes are defined using these helper functions:

```typescript
route('path', 'component-path')
layout('layout-component', [child-routes])
index('index-component')
```

## Adding a New Page to the Marketing Section

Let's create a new "Features" page in the marketing section:

1. **Create a new component file**:

```tsx
// apps/web/app/routes/marketing/features.tsx
import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { SitePageHeader } from '~/routes/marketing/_components/site-page-header';
import type { Route } from '~/types/app/routes/marketing/+types/features';

export const meta = ({ data }: Route.MetaArgs) => {
  return [
    {
      title: data?.title,
    },
  ];
};

export const loader = async function ({ request }: Route.LoaderArgs) {
  const { t } = await createI18nServerInstance(request);

  return {
    title: t('marketing:features'),
    subtitle: t('marketing:featuresDescription'),
  };
};

export default function FeaturesPage(props: Route.ComponentProps) {
  const data = props.loaderData;

  return (
    <div>
      <SitePageHeader title={data.title} subtitle={data.subtitle} />

      <div className={'container mx-auto py-8'}>
        <div className="grid grid-cols-1 gap-8 md:grid-cols-2 lg:grid-cols-3">
          {/* Feature cards */}
          <FeatureCard
            title="Multi-tenant Support"
            description="Support for multiple organizations and team accounts"
          />
          <FeatureCard
            title="Authentication"
            description="Various authentication methods including email, password, and OAuth"
          />
          <FeatureCard
            title="Billing Integration"
            description="Seamless integration with popular payment providers"
          />
        </div>
      </div>
    </div>
  );
}

function FeatureCard({ title, description }: { title: string; description: string }) {
  return (
    <div className="rounded-lg border p-6">
      <h3 className="text-xl font-semibold">{title}</h3>
      <p className="text-muted-foreground mt-2">{description}</p>
    </div>
  );
}
```

2. **Update the routes configuration** in `apps/web/app/routes.ts`:

```typescript
const marketingLayout = layout('routes/marketing/layout.tsx', [
  index('routes/marketing/index.tsx'),
  route('terms-of-service', 'routes/marketing/terms-of-service.tsx'),
  route('privacy-policy', 'routes/marketing/privacy-policy.tsx'),
  route('pricing', 'routes/marketing/pricing.tsx'),
  route('contact', 'routes/marketing/contact/index.tsx'),
  route('features', 'routes/marketing/features.tsx'), // Add your new route here
  route('faq', 'routes/marketing/faq.tsx'),
  // ...other routes
]);
```

3. **Update navigation** in `apps/web/app/routes/marketing/_components/site-navigation.tsx`:

```typescript
const links = {
  Blog: {
    label: 'marketing:blog',
    path: '/blog',
  },
  Features: {    // Add this new item
    label: 'marketing:features',
    path: '/features',
  },
  Docs: {
    label: 'marketing:documentation',
    path: '/docs',
  },
  // ...other links
};
```

4. **Add translations** (optional) by updating your i18n files with the new labels.

## Adding a Page to the Authenticated App Area

Let's add a "Reports" page for authenticated users:

1. **Create the component file**:

```tsx
// apps/web/app/routes/home/user/reports.tsx
import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';
import { PageBody } from '@kit/ui/page';
import { Trans } from '@kit/ui/trans';

import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { requireUserLoader } from '~/lib/require-user-loader';
import type { Route } from '~/types/app/routes/home/user/+types/reports';

import { HomeLayoutPageHeader } from './_components/home-page-header';

export const meta = ({ data }: Route.MetaArgs) => {
  return [
    {
      title: data?.title,
    },
  ];
};

export const loader = async (args: Route.LoaderArgs) => {
  // require user
  await requireUserLoader(args.request);

  const i18n = await createI18nServerInstance(args.request);
  const title = i18n.t('account:reportsTab');

  return {
    title,
  };
};

export default function UserReportsPage(props: Route.ComponentProps) {
  return (
    <>
      <HomeLayoutPageHeader
        title={<Trans i18nKey={'account:reportsTabLabel'} />}
        description={<AppBreadcrumbs />}
      />

      <PageBody>
        <div className="flex w-full flex-col space-y-6">
          <h1 className="text-2xl font-semibold">Reports</h1>
          <p className="text-muted-foreground">
            View usage statistics and reports for your account.
          </p>

          {/* Add your reports components here */}
          <div className="grid grid-cols-1 gap-4 lg:grid-cols-2">
            <ReportCard title="Usage Statistics" />
            <ReportCard title="Activity Log" />
          </div>
        </div>
      </PageBody>
    </>
  );
}

function ReportCard({ title }: { title: string }) {
  return (
    <div className="rounded-md border p-4">
      <h2 className="text-lg font-medium">{title}</h2>
      <div className="h-32 w-full bg-gray-100 dark:bg-gray-800 mt-2 flex items-center justify-center">
        <span className="text-muted-foreground">Report data will appear here</span>
      </div>
    </div>
  );
}
```

2. **Update the routes configuration** in `apps/web/app/routes.ts`:

```typescript
const userAccountLayout = layout('routes/home/user/layout.tsx', [
  route('home', 'routes/home/user/index.tsx'),
  route('home/settings', 'routes/home/user/settings.tsx'),
  route('home/billing', 'routes/home/user/billing.tsx'),
  route('home/billing/return', 'routes/home/user/billing-return.tsx'),
  route('home/reports', 'routes/home/user/reports.tsx'), // Add your new route here
]);
```

3. **Update navigation configuration** in `~/config/personal-account-navigation.config.ts`:

```typescript
// Add this to the routes array
{
  label: 'account:reportsTabLabel',
  Icon: <BarChart className={'h-4'} />,
  path: '/home/reports',
  end: true,
},
```

## Creating Nested Routes

For more complex scenarios, you might need nested routes with their own layout:

1. **Create a layout component**:

```tsx
// apps/web/app/routes/home/user/reports/layout.tsx
import { Outlet } from 'react-router';
import { PageHeader } from '@kit/ui/page';
import { Tabs, TabsList, TabsTrigger } from '@kit/ui/tabs';

export default function ReportsLayout() {
  return (
    <div className="flex flex-col space-y-4">
      <PageHeader>
        <Tabs defaultValue="overview">
          <TabsList>
            <TabsTrigger value="overview" href="/home/reports">Overview</TabsTrigger>
            <TabsTrigger value="activity" href="/home/reports/activity">Activity</TabsTrigger>
            <TabsTrigger value="usage" href="/home/reports/usage">Usage</TabsTrigger>
          </TabsList>
        </Tabs>
      </PageHeader>

      <Outlet />
    </div>
  );
}
```

2. **Create child route components**:

```tsx
// apps/web/app/routes/home/user/reports/activity.tsx
export default function ActivityReport() {
  return <div>Activity report content here</div>;
}
```

3. **Update the routes configuration** with a nested structure:

```typescript
const reportsLayout = layout('routes/home/user/reports/layout.tsx', [
  index('routes/home/user/reports/index.tsx'),
  route('activity', 'routes/home/user/reports/activity.tsx'),
  route('usage', 'routes/home/user/reports/usage.tsx'),
]);

const userAccountLayout = layout('routes/home/user/layout.tsx', [
  // ... existing routes
  reportsLayout, // Add the nested layout
]);
```

## Best Practices

1. **Route Organization**:
- Group related routes under shared layouts
- Use descriptive naming for route paths and components
2. **UI Consistency**:
- Reuse existing components like `PageHeader`, `PageBody`, etc.
- Follow the established UI patterns for each section
3. **Authentication**:
- Use `requireUserLoader` for authenticated routes
- Consider permissions for team/admin routes
4. **i18n Support**:
- Always use translation keys for user-visible text
- Structure translation keys logically (e.g., `marketing:features`)
5. **Type Safety**:
- Create proper type definitions for loaders and components
- Use the existing Route types pattern

By following these conventions, you can seamlessly add new pages to the starter kit while maintaining consistent structure and behavior.

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\adding-turborepo-app.mdoc

---
status: "published"
label: "Adding a Turborepo application"
title: "Add a new Turborepo application to your Makerkit application | React Router Supabase"
description: "Learn how to add a new Turborepo application to your Makerkit application"
order: 12
---

This is an **advanced topic** - you should only follow these instructions if you are placing a new app within your monorepo and want to keep pulling updates from the Makerkit repository.

In some ways - creating a new repository may be the easiest way to manage your application. However, if you want to keep your application within the monorepo and pull updates from the Makerkit repository, you can follow these instructions.

---

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
FILE PATH: kits\react-router-supabase-turbo\development\adding-turborepo-package.mdoc

---
status: "published"
label: "Adding a Turborepo package"
title: "Add a new Turborepo package to your React Router Supabase SaaS Kit"
description: "Learn how to add a new Turborepo package to your Makerkit application"
order: 11
---

This is an **advanced topic** - you should only follow these instructions if you are sure you want to add a new package to your Makerkit application instead of adding a folder to your application in `apps/web`. You don't need to do this to add a new page or component to your application.

---

To add a new package to your Makerkit application, you need to follow these steps.

First, enter the command below to create a new package in your Makerkit application:

```bash
turbo gen
```

Turborepo will ask you to enter the name of the package you want to create. Enter the name of the package you want to create and press enter.

If you don't want to add dependencies to your package, you can skip this step by pressing enter.

The command will have generated a new package under `packages` named `@kit/<package-name>`. If you named it `my-package`, the package will be named `@kit/my-package`.

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
FILE PATH: kits\react-router-supabase-turbo\development\approaching-local-development.mdoc

---
status: "published"
label: "Starting developing your App"
order: 0
title: "How to approach local development | React Router Supabase Turbo"
description: "Learn how to approach local development in your React Router Supabase Turbo project"
---

In the previous sections, you learnt how to clone the repository, set environment variables, customize the look and feel of the app, and some basic API that you'll be using throughout the app. In this section, you'll learn how to start developing your app locally.

Buckle up, we have a lot to do!

Generally speaking, you will be doing the following:

1. **Customization**: Set environment variables (application name, feature flags). This is a quick starting point that will start turning Makerkit into your very own app.
2. **Database**: Drawing and writing the database schema. Of course, your app will store data and have its schema. It's time to draw it.
3. **Routing**: Adding new routes. Your new pages will need routes - unless you can reuse the "home" pages of the accounts.
4. **Fetching Data**: Fetching data from your DB and display it onto the new routes.
5. **Writing Data** Adding new forms. You will need to add forms to create new data.

In 90% of cases, the above is what you will be doing. The remaining 10% is adding new (very specific) features, which is a bit more complex - and not relevant to Makerkit itself.

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\database-functions.mdoc

---
status: "published"

label: "Database Functions"
order: 3
title: "Database Functions available in your React Router Supabase schema"
description: "Learn the most useful database functions in your schema"
---


The database schema contains several functions that you can use so that you can extend your database with custom logic and RLS.

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

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\database-schema.mdoc

---
status: "published"
label: "Database Schema"
order: 2
title: "How to create new migrations and update the database schema in your React Router Supabase application"
description: "Learn how to create new migrations and update the database schema in your React Router Supabase application"
---

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

### RLS Policies

RLS Policies are fundamental to protect our tables.

We insert an RLS policy for each action: `select`, `insert`, `update` and `delete`.

#### Selecting Tasks

When writing an RLS policy for selecting data from the `tasks` table, we need to make sure the user is the owner of the task or has the required permission.

Generally speaking, entities belong to `public.accounts` - and we can use the `account_id` to check if the user is the owner.

```sql
create policy select_tasks on public.tasks
    for select
    to authenticated
    using (
      account_id = auth.uid() or
      public.has_role_on_account(account_id)
    );
```

Did you know that an account can be a user or a team? We can use the `public.has_role_on_account` function to check if the user has a role on the account.

Therefore, this RLS policy works in both ways:

1. if the user is the owner of the task - we check that the `account_id` is equal to the `auth.uid()`
2. if the user has a role on the account - we check that the user has a role on the account

#### Inserting Tasks

When writing an RLS policy for inserting data into the `tasks` table, we need to make sure the user is the owner of the task or has the required permission.

```sql
create policy insert_tasks on public.tasks
    for insert
    with check (
        account_id = auth.uid() or
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
        account_id = auth.uid() or
        public.has_permission(auth.uid(), account_id, 'tasks.write'::app_permissions)
    )
    with check (
        account_id = auth.uid() or
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
        account_id = auth.uid() or
        public.has_permission(auth.uid(), account_id, 'tasks.delete'::app_permissions)
    );
```

In the above, we check if the user is the owner of the task or has the `tasks.delete` permission.

Our schema is now complete! Yay! 🎉

### Resetting Migrations

When adding a new schema, we need to reset the migrations.

```bash
pnpm run supabase:web:reset
```

Then, we generate the new types using the following command:

```bash
pnpm run supabase:web:typegen
```

You can now use the new types in your application when using the Supabase client.


