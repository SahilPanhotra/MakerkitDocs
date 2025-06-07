-----------------
FILE PATH: kits/react-router-supabase-turbo/development/adding-turborepo-app.mdoc

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
FILE PATH: kits/react-router-supabase-turbo/development/adding-turborepo-package.mdoc

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
FILE PATH: kits/react-router-supabase-turbo/development/approaching-local-development.mdoc

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
FILE PATH: kits/react-router-supabase-turbo/development/database-functions.mdoc

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
FILE PATH: kits/react-router-supabase-turbo/development/database-schema.mdoc

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


-----------------
FILE PATH: kits/react-router-supabase-turbo/development/database-webhooks.mdoc

---
status: "published"

label: "Database Webhooks"
order: 6
title: "How to handle custom database webhooks in your React Router Supabase application"
description: "Learn how to handle custom database webhooks in your React Router Supabase application"
---


Database webhooks allow you to listen to changes in your database and trigger custom logic when a change occurs. This is useful for sending notifications, updating caches, or triggering other services.

Makerkit handles some webhooks by default for functionalities such as deleting a subscription following a user deletion, or sending emails after a user signs up.

You can extend this functionality by adding your own webhook handlers:

 ```tsx {% title="apps/web/app/routes/api/db/webhook.ts" %}
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
FILE PATH: kits/react-router-supabase-turbo/development/legal-pages.mdoc

---
status: "published"
label: "Legal Pages"
title: "Legal Pages in the React Router Supabase Turbo Starter Kit"
description: "Learn how to create and update legal pages in the React Router Supabase Turbo Starter Kit."
order: 9
---

Legal pages are defined in the `apps/web/app/routes/marketing` directory.

Makerkit comes with the following legal pages:

1. Terms and Conditions
2. Privacy Policy
3. Cookie Policy

For obvious reasons, **these pages are empty and you need to fill in the content**.

Do yourself a favor and do not use ChatGPT to generate these pages.

-----------------
FILE PATH: kits/react-router-supabase-turbo/development/marketing-pages.mdoc

---
status: "published"
label: "Marketing Pages"
title: "Marketing Pages in the React Router Supabase Turbo Starter Kit"
description: "Learn how to create and update marketing pages in the React Router Supabase Turbo Starter Kit."
order: 7
---

Makerkit comes with pre-defined marketing pages to help you get started with your SaaS application. These pages are built with React Router and Tailwind CSS and are located in the `apps/web/app/(marketing)` directory.

Makerkit comes with the following marketing pages:

- Home Page
- Contact Form
- Pricing Page
- FAQ
- Contact Page (with a contact form)

## Adding a new marketing page

To add a new marketing page to your Makerkit application, you need to follow these steps.

Create a file in the `apps/web/app/routes/marketing` directory:

```tsx
// apps/web/app/routes/marketing/about.tsx
export default function AboutPage() {
  return <div></div>
}
```

Then add the route to the `apps/web/app/routes.ts` file:

```tsx {% title="apps/web/app/routes.ts" %}
const marketingLayout = layout('routes/marketing/layout.tsx', [
  index('routes/marketing/index.tsx'),
  route('terms-of-service', 'routes/marketing/terms-of-service.tsx'),
  route('privacy-policy', 'routes/marketing/privacy-policy.tsx'),
  route('pricing', 'routes/marketing/pricing.tsx'),
  route('contact', 'routes/marketing/contact/index.tsx'),
  route('faq', 'routes/marketing/faq.tsx'),
  route('blog', 'routes/marketing/blog/index.tsx'),
  route('blog/:slug', 'routes/marketing/blog/$slug.tsx'),
  route('cookie-policy', 'routes/marketing/cookie-policy.tsx'),
  route('about', 'routes/marketing/about.tsx'),  // <-- add this line
  layout('routes/marketing/docs/layout.tsx', [
    route('docs', 'routes/marketing/docs/index.tsx'),
    route('docs/*', 'routes/marketing/docs/$slug.tsx'),
  ]),
]);
```

This page inherits the layout at `apps/web/app/routes/marketing/layout.tsx`. You can customize the layout by editing this file - but remember that it will affect all marketing pages.

## Contact Form

To make the contact form work, you need to add the following environment variables:

```bash
CONTACT_EMAIL=
```

In this variable, you need to set the email address where you want to receive the contact form submissions. The sender will be the same as the one configured in your [mailing configuration](../emails/email-configuration).

-----------------
FILE PATH: kits/react-router-supabase-turbo/development/migrations.mdoc

---
status: "published"
label: "Migrations"
order: 1
title: "How to create new migrations and update the database schema in your React Router Supabase application"
description: "Learn how to create new migrations and update the database schema in your React Router Supabase application"
---

Creating a schema for your data is one of the primary tasks when building a new application. In this guide, we'll walk through how to create new migrations and update the database schema in your Next.js Supabase application.

## A quick word on migrations

Supabase's hosted Studio is pretty great - but I don't think it should be used to perform schema changes. Instead, I recommend using your local Supabase Studio to make the changes and then generate the migration file. Then, you can push the migration to the remote Supabase instance.

## Declarative schema

Using Supabase's declarative schema, you can define your schema at `apps/web/supabase/schemas`. In here, we can create a new file called using whatever name you want: for example, if our schema is about `integrations`, we can create a file called `integrations.sql`.

The file should contain the SQL statements to create the schema. 

For example, here's a basic schema for an `integrations` table:

```sql
create table if not exists public.integrations (
  id uuid primary key default uuid_generate_v4(),
  name text not null,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);
```

Now, you want this schema to be applied to your local Supabase instance so you can start using it and test it out. We can now use Supabase's diffing feature to generate a migration file for us.

```bash
pnpm --filter web run supabase:stop
pnpm --filter web run supabase:db:diff
```

These two commands will respectively:

1. Stop the Supabase service
2. Generate the migration file

The migration file will be created in the `apps/web/supabase/migrations` directory. Now, we want to restart Supabase with the `no-backup` flag to avoid restoring the database from the backup.

```bash
pnpm run supabase:web:start --no-backup
```

## Pushing the migration to the remote Supabase instance

After you're happy with the changes, you can push the migration to the remote Supabase instance. This is so that the changes are reflected in the remote database and not only in your local Supabase Studio.

```bash
pnpm --filter web supabase db push
```

## Applying changes to the database schema

Whenever you want to apply changes to the database schema, you can use the same process as above.

0. Stop the Supabase service
1. Modify the schema in the `apps/web/supabase/schemas` directory
2. Generate the migration file using the diffing feature
3. Restart Supabase with the `no-backup` flag to test the changes locally
4. Push the migration to the remote Supabase instance

-----------------
FILE PATH: kits/react-router-supabase-turbo/development/seo.mdoc

---
status: "published"

label: "SEO"
title: SEO - Improve your React Router application's search engine optimization"
description:  "Learn how to improve your Makerkit application's search engine optimization (SEO)"
order: 10
---


SEO is an important part of building a website. It helps search engines understand your website and rank it higher in search results. In this guide, you'll learn how to improve your Makerkit application's search engine optimization (SEO).

Makerkit is already optimized for SEO out of the box. However, there are a few things you can do to improve your application's SEO:

1. **Content**: The most important thing you can do to improve your application's SEO is to **create high-quality content**. No amount of technical optimization can replace good content. Make sure your content is relevant, useful, and engaging - and make sure it's updated regularly.
2. **Write content helpful to your customers**: To write good content, the kit comes with a blog and documentation feature. You can use these features to create high-quality content that will help your website rank higher in search results - and help your customers find what they're looking for.
3. **Use the correct keywords**: Use descriptive titles and meta descriptions for your pages. Titles and meta descriptions are the first things users see in search results, so make sure they are descriptive and relevant to the content on the page. Use keywords that your customers are likely to search for.
4. **Optimize images**: Use descriptive filenames and alt text for your images. This helps search engines understand what the image is about and can improve your website's ranking in image search results.
5. **Website Speed**: This is much less important than it used to be, but it's still a factor. Make sure your website loads quickly and is mobile-friendly. You can use tools like Google's PageSpeed Insights to check your website's speed and get suggestions for improvement. Optimize all images and assets to reduce load times.
6. **Mobile Optimization**: Make sure your website is mobile-friendly. Google ranks mobile-friendly websites higher in search results. You can use Google's Mobile-Friendly Test to check if your website is mobile-friendly.
7. **Sitemap and Robots.txt**: Makerkit generates a sitemap and robots.txt file for your website. These files help search engines understand your website's structure and what pages they should index. Make sure to update the sitemap as you add new pages to your website.
8. **Backlinks**: Backlinks are links from other websites to your website. It's touted to be **the single most important factor in SEO** these days. The more backlinks you have from high-quality websites, the higher your website will rank in search results. You can get backlinks by creating high-quality content that other websites want to link to.

### Adding pages to the sitemap

Generally speaking, Google will find your pages without a sitemap as it follows the link in your website. However, you can add pages to the sitemap by adding them to the `apps/web/app/server-sitemap.xml/route.ts` route, which is used to generate the sitemap.

If you add more static pages to your website, you can add them to the `getPaths` function in the `apps/web/app/routes/sitemap/route.tsx` file. For example, if you add a new page at `/about`, you can add it to the `getPaths` function like this:

 ```tsx {% title="apps/web/app/routes/sitemap/route.tsx" %}
function getPaths() {
  const paths = [
    '/',
    '/faq',
    '/blog',
    '/docs',
    '/pricing',
    '/contact',
    '/cookie-policy',
    '/terms-of-service',
    '/privacy-policy',
    // add more paths here,
    '/about', // <-- add the new page here
  ];

  return paths.map((path) => {
    return {
      loc: new URL(path, appConfig.url).href,
      lastmod: new Date().toISOString(),
    };
  });
}
```

All the blog and documentation pages are automatically added to the sitemap. You don't need to add them manually.

### Adding your website to Google Search Console

Once you've optimized your website for SEO, you can add it to Google Search Console. Google Search Console is a free tool that helps you monitor and maintain your website's presence in Google search results.

You can use it to check your website's indexing status, submit sitemaps, and get insights into how Google sees your website.

The first thing you need to do is verify your website in Google Search Console. You can do this by adding a meta tag to your website's HTML or by uploading an HTML file to your website.

Once you've verified your website, you can submit your sitemap to Google Search Console. This will help Google find and index your website's pages faster.

Please submit your sitemap to Google Search Console by going to the `Sitemaps` section and adding the URL of your sitemap. The URL of your sitemap is `https://your-website.com/server-sitemap.xml`.

Of course, please replace `your-website.com` with your actual website URL.

### Backlinks

Backlinks are said to be the most important factor in modern SEO. The more backlinks you have from high-quality websites, the higher your website will rank in search results - and the more traffic you'll get.

How do you get backlinks? The best way to get backlinks is to create high-quality content that other websites want to link to. However, you can also get backlinks by:

1. **Guest posting**: Write guest posts for other websites in your niche. This is a great way to get backlinks and reach a new audience.
2. **Link building**: Reach out to other websites and ask them to link to your website. You can offer to write a guest post or provide a testimonial in exchange for a backlink.
3. **Social media**: Share your content on social media to reach a wider audience and get more backlinks.
4. **Directories**: Submit your website to online directories to get backlinks. Make sure to choose high-quality directories that are relevant to your niche.

All these methods can help you get more backlinks and improve your website's SEO - and help you rank higher in search results.

### Content

When it comes to internal factors, content is king. Make sure your content is relevant, useful, and engaging. Make sure it's updated regularly and optimized for SEO.

What should you write about? Most importantly, you want to think about how your customers will search for the problem your SaaS is solving. For example, if you're building a project management tool, you might want to write about project management best practices, how to manage a remote team, or how to use your tool to improve productivity.

You can use the blog and documentation features in Makerkit to create high-quality content that will help your website rank higher in search results - and help your customers find what they're looking for.

### Indexing and ranking take time

New websites can take a while to get indexed by search engines. It can take anywhere from a few days to a few weeks (in some cases, even months!) for your website to show up in search results. Be patient and keep updating your content and optimizing your website for search engines.

---

By following these tips, you can improve your Makerkit application's search engine optimization (SEO) and help your website rank higher in search results. Remember, SEO is an ongoing process, so make sure to keep updating your content and optimizing your website for search engines. Good luck!


