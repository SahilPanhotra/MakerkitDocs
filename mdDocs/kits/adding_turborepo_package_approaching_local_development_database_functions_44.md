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

---

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


-----------------
FILE PATH: kits\next-supabase-turbo\development\database-webhooks.mdoc

---
status: "published"
label: "Database Webhooks"
order: 6
title: "How to handle custom database webhooks in your Next.js Supabase application"
description: "Learn how to handle custom database webhooks in your Next.js Supabase application"
---

Database webhooks allow you to listen to changes in your database and trigger custom logic when a change occurs. This is useful for sending notifications, updating caches, or triggering other services.

Makerkit handles some webhooks by default for functionalities such as deleting a subscription following a user deletion, or sending emails after a user signs up.

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

-----------------
FILE PATH: kits\next-supabase-turbo\development\legal-pages.mdoc

---
status: "published"

label: "Legal Pages"
title: "Legal Pages in the Next.js Supabase Turbo Starter Kit"
description: "Learn how to create and update legal pages in the Next.js Supabase Turbo Starter Kit."
order: 8
---


Legal pages are defined in the `apps/web/app/(marketing)/(legal)` directory. 

Makerkit comes with the following legal pages:

1. Terms and Conditions
2. Privacy Policy
3. Cookie Policy

For obvious reasons, **these pages are empty and you need to fill in the content**.

Do yourself a favor and do not use ChatGPT to generate these pages.

### Using a CMS for legal pages

You can use a CMS to manage the content of the legal pages. To do this, use the [CMS Client](content/cms):

```tsx
import { createCmsClient } from '@kit/cms';

export async function MyPage() {
  const cms = await createCmsClient();

  const { title, content } = await cms.getContentBySlug({
    slug: `slug`,
    collection: `pages`
  });

  return (
    <div>
      <h1>{title}</h1>
      <div dangerouslySetInnerHTML={{ __html: content }} />
    </div>
  );
}
```

-----------------
FILE PATH: kits\next-supabase-turbo\development\loading-data-from-database.mdoc

---
status: "published"

label: "Loading data from the DB"
order: 4
title: "Learn how to load data from the Supabase database"
description: "In this page we learn how to load data from the Supabase database and display it in our Next.js application."
---


Now that our database supports the data we need, we can start loading it into our application. We will use the `@makerkit/data-loader-supabase-nextjs` package to load data from the Supabase database.

Please check the [documentation](https://github.com/makerkit/makerkit/tree/main/packages/data-loader/supabase/nextjs) for the `@makerkit/data-loader-supabase-nextjs` package to learn more about how to use it.

This nifty package allows us to load data from the Supabase database and display it in our server components with support for pagination.

In the snippet below, we will

1. Load the user's workspace data from the database. This allows us to get the user's account ID without further round-trips because the workspace is loaded by the user layout.
2. Load the user's tasks from the database.
3. Display the tasks in a table.
4. Use a search input to filter the tasks by title.

Let's take a look at the code:

```tsx
import { use } from 'react';

import { ServerDataLoader } from '@makerkit/data-loader-supabase-nextjs';

import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { Button } from '@kit/ui/button';
import { Heading } from '@kit/ui/heading';
import { If } from '@kit/ui/if';
import { Input } from '@kit/ui/input';
import { PageBody } from '@kit/ui/page';
import { Trans } from '@kit/ui/trans';

import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { withI18n } from '~/lib/i18n/with-i18n';

import { TasksTable } from './_components/tasks-table';
import { UserAccountHeader } from './_components/user-account-header';
import { loadUserWorkspace } from './_lib/server/load-user-workspace';

interface SearchParams {
  page?: string;
  query?: string;
}

export const generateMetadata = async () => {
  const i18n = await createI18nServerInstance();
  const title = i18n.t('account:homePage');

  return {
    title,
  };
};

function UserHomePage(props: { searchParams: SearchParams }) {
  const client = getSupabaseServerClient();
  const { user } = use(loadUserWorkspace());

  const page = parseInt(props.searchParams.page ?? '1', 10);
  const query = props.searchParams.query ?? '';

  return (
    <>
      <UserAccountHeader
        title={<Trans i18nKey={'common:homeTabLabel'} />}
        description={<Trans i18nKey={'common:homeTabDescription'} />}
      />

      <PageBody className={'space-y-4'}>
        <div className={'flex items-center justify-between'}>
          <div>
            <Heading level={4}>
              <Trans i18nKey={'tasks:tasksTabLabel'} defaults={'Tasks'} />
            </Heading>
          </div>

          <div className={'flex items-center space-x-2'}>
            <form className={'w-full'}>
              <Input
                name={'query'}
                defaultValue={query}
                className={'w-full lg:w-[18rem]'}
                placeholder={'Search tasks'}
              />
            </form>
          </div>
        </div>

        <ServerDataLoader
          client={client}
          table={'tasks'}
          page={page}
          where={{
            account_id: {
              eq: user.id,
            },
            title: {
              textSearch: query ? `%${query}%` : undefined,
            },
          }}
        >
          {(props) => {
            return (
              <div className={'flex flex-col space-y-8'}>
                <If condition={props.count === 0 && query}>
                  <div className={'flex flex-col space-y-2.5'}>
                    <p>
                      <Trans
                        i18nKey={'tasks:noTasksFound'}
                        values={{ query }}
                      />
                    </p>

                    <form>
                      <input type="hidden" name={'query'} value={''} />

                      <Button variant={'outline'} size={'sm'}>
                        <Trans i18nKey={'tasks:clearSearch'} />
                      </Button>
                    </form>
                  </div>
                </If>

                <TasksTable {...props} />
              </div>
            );
          }}
        </ServerDataLoader>
      </PageBody>
    </>
  );
}

export default withI18n(UserHomePage);
```

Let's break this down a bit, shall we:

1. We import the necessary components and functions.
2. We define the `SearchParams` interface to type the search parameters.
3. We define the `generateMetadata` function to generate the page metadata.
4. We define the `UserHomePage` component that loads the user's workspace and tasks from the database.
5. We define the `ServerDataLoader` component that loads the tasks from the database.
6. We render the tasks in a table and provide a search input to filter the tasks by title.
7. We export the `UserHomePage` component with the `withI18n` HOC. This helps bootstrap the i18n instance for the component.

### Tasks Table

Now, let's show the tasks table component:

```tsx
'use client';

import Link from 'next/link';

import { ColumnDef } from '@tanstack/react-table';
import { Pencil } from 'lucide-react';
import { useTranslation } from 'react-i18next';

import { Button } from '@kit/ui/button';
import { DataTable } from '@kit/ui/enhanced-data-table';

import { Database } from '~/lib/database.types';

type Task = Database['public']['Tables']['tasks']['Row'];

export function TasksTable(props: {
  data: Task[];
  page: number;
  pageSize: number;
  pageCount: number;
}) {
  const columns = useGetColumns();

  return (
    <div>
      <DataTable {...props} columns={columns} />
    </div>
  );
}

function useGetColumns(): ColumnDef<Task>[] {
  const { t } = useTranslation('tasks');

  return [
    {
      header: t('task'),
      cell: ({ row }) => (
        <Link
          className={'hover:underline'}
          href={`/home/tasks/${row.original.id}`}
        >
          {row.original.title}
        </Link>
      ),
    },
    {
      header: t('createdAt'),
      accessorKey: 'created_at',
    },
    {
      header: t('updatedAt'),
      accessorKey: 'updated_at',
    },
    {
      header: '',
      id: 'actions',
      cell: ({ row }) => {
        const id = row.original.id;

        return (
          <div className={'flex justify-end space-x-2'}>
            <Link href={`/home/tasks/${id}`}>
              <Button variant={'ghost'} size={'icon'}>
                <Pencil className={'h-4'} />
              </Button>
            </Link>
          </div>
        );
      },
    },
  ];
}
```

In this snippet, we define the `TasksTable` component that renders the tasks in a table. We use the `DataTable` component from the `@kit/ui/enhanced-data-table` package to render the table.

We also define the `useGetColumns` hook that returns the columns for the table. We use the `useTranslation` hook from the `react-i18next` package to translate the column headers.

