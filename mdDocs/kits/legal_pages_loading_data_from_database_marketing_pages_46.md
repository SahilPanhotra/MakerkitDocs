-----------------
FILE PATH: kits\next-supabase-turbo\development\legal-pages.mdoc

---
status: "published"
label: "Legal Pages"
title: "Legal Pages in the Next.js Supabase Turbo Starter Kit"
description: "Learn how to create and update legal pages in the Next.js Supabase Turbo Starter Kit."
order: 8
---

{% sequence title="Legal pages in the Starter Kit" description="Learn how to create and update legal pages in the Next.js Supabase Turbo Starter Kit." %}

[Includes pages in the Starter Kit](#includes-pages-in-the-starter-kit)

[Using a CMS for legal pages](#using-a-cms-for-legal-pages)

{% /sequence %}

## Includes pages in the Starter Kit

Legal pages are defined in the `apps/web/app/(marketing)/(legal)` directory. 

Makerkit comes with the following legal pages:

1. **Terms and Conditions**: `apps/web/app/(marketing)/(legal)/terms-and-conditions.mdx`
2. **Privacy Policy**: `apps/web/app/(marketing)/(legal)/privacy-policy.mdx`
3. **Cookie Policy**: `apps/web/app/(marketing)/(legal)/cookie-policy.mdx`

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

In the snippet below, we will:

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

Let's break this down a bit:

1. We import the necessary components and functions.
2. We define the `SearchParams` interface to type the search parameters.
3. We define the `generateMetadata` function to generate the page metadata.
4. We define the `UserHomePage` component that loads the user's workspace and tasks from the database.
5. We define the `ServerDataLoader` component that loads the tasks from the database.
6. We render the tasks in a table and provide a search input to filter the tasks by title.
7. We export the `UserHomePage` component with the `withI18n` HOC. This helps bootstrap the i18n instance for the component.

### Displaying the tasks in a table

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

-----------------
FILE PATH: kits\next-supabase-turbo\development\marketing-pages.mdoc

---
status: "published"
label: "Marketing Pages"
title: "Marketing Pages in the Next.js Supabase Turbo Starter Kit"
description: "Learn how to create and update marketing pages in the Next.js Supabase Turbo Starter Kit."
order: 7
---

{% sequence title="How to create and update marketing pages" description="Learn how to create and update marketing pages in the Next.js Supabase Turbo Starter Kit." %}

[The marketing pages included in the Starter Kit](#the-marketing-pages-included-in-the-starter-kit)

[Adding a new marketing page](#adding-a-new-marketing-page)

[Customizing the layout](#customizing-the-layout)

[Adding a contact form](#adding-a-contact-form)

{% /sequence %}

Makerkit comes with pre-defined marketing pages to help you get started with your SaaS application. These pages are built with Next.js and Tailwind CSS and are located in the `apps/web/app/(marketing)` directory.

## The marketing pages included in the Starter Kit

Makerkit comes with the following marketing pages:

- Home Page
- Contact Form
- Pricing Page
- FAQ
- Contact Page (with a contact form)
- [Legal Pages (Terms of Service, Privacy Policy, Cookie Policy)](./legal-pages)

## Adding a new marketing page

To add a new marketing page to your Makerkit application, you need to follow these steps.

Create a folder in the `apps/web/app/(marketing)` directory with the path you want to use for the page. For example, to create a new page at `/about`, you would create a folder named `about`. Then, create the page file in the folder. For example, to create an `about` page, you would create an `page.tsx` file in the `about` folder.

```tsx
// apps/web/app/(marketing)/about/page.tsx
export default function AboutPage() {
  return <div></div>
}
```

### Customizing the layout

This page inherits the layout at `apps/web/app/(marketing)/layout.tsx`. You can customize the layout by editing this file - but remember that it will affect all marketing pages.

## Contact Form

To make the contact form work, you need to add the following environment variables:

```bash
CONTACT_EMAIL=
```

In this variable, you need to set the email address where you want to receive the contact form submissions. The sender will be the same as the one configured in your [mailing configuration](../emails/email-configuration).

-----------------
FILE PATH: kits\next-supabase-turbo\development\migrations.mdoc

---
status: "published"
label: "Migrations"
order: 1
title: "How to create new migrations and update the database schema in your Next.js Supabase application"
description: "Learn how to create new migrations and update the database schema in your Next.js Supabase application"
---

{% sequence title="How to create new migrations and update the database schema" description="Learn how to create new migrations and update the database schema in your Next.js Supabase application" %}

[Do not use Supabase Studio to make schema changes](#do-not-use-supabase-studio-to-make-schema-changes-(recommendation))

[Updating the Database Schema](#declarative-schema)

[Creating a migration file from a schema change](#creating-a-migration-file-from-a-schema-change)

[Pushing the migration to the remote Supabase instance](#pushing-the-migration-to-the-remote-supabase-instance)

[Applying changes to the database schema](#applying-changes-to-the-database-schema)

{% /sequence %}

Creating a schema for your data is one of the primary tasks when building a new application. In this guide, we'll walk through how to create new migrations and update the database schema in your Next.js Supabase application.

## Do not use Supabase Studio to make schema changes (recommendation)

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

## Creating a migration file from a schema change

Now, you want this schema to be applied to your local Supabase instance so you can start using it and test it out.

We can now use Supabase's diffing feature to generate a migration file for us.

```bash
pnpm --filter web run supabase:stop     # Stop the Supabase service
pnpm --filter web run supabase:db:diff -f <filename>  # Generate the migration file using the diffing feature
```

These two commands will respectively:

1. Stop the Supabase service
2. Generate the migration file using the diffing feature

The migration file will be created in the `apps/web/supabase/migrations` directory.

```bash
pnpm run supabase:web:start   # start supabase
pnpm run supabase:web:reset   # reset supabase
```

{% alert type="warning" title="Double check the migration file" %}
Make sure the migration file is correct before pushing it to the remote Supabase instance. The diffing tool can make mistakes and has various caveats. Always double check the generated migration file before pushing it to the remote Supabase instance.
{% /alert %}

Learn more about [the known caveats of the diffing tool](https://supabase.com/docs/guides/local-development/declarative-database-schemas#known-caveats).

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
FILE PATH: kits\next-supabase-turbo\development\permissions-and-roles.mdoc

---
status: "published"
label: 'RBAC: Roles and Permissions'
title: 'RBAC: Understanding roles and permissions in Next.js Supabase'
description: 'Learn how to implement roles and permissions in Next.js Supabase'
order: 6
---

{% sequence title="How to implement roles and permissions" description="Learn how to implement roles and permissions in Next.js Supabase" %}

[Roles and Permissions tables](#roles-and-permissions-tables)

[Hierarchy levels](#hierarchy-levels)

[Adding new permissions](#adding-new-permissions)

[Default roles and permissions](#default-roles-and-permissions)

[Adding new roles and permissions](#adding-new-roles-and-permissions)

[Using roles and permissions in RLS](#using-roles-and-permissions-in-rls)

[Using roles and permissions in application code](#using-roles-and-permissions-in-application-code)

[Using permissions client-side](#using-permissions-client-side)

{% /sequence %}

Makerkit implements granular RBAC for team members. This allows you to define roles and permissions for each team member - giving you fine-grained control over who can access what.

## Roles and Permissions tables

Makerkit implements two tables for roles and permissions:

- `roles` table: This table stores the roles for each team member.
- `role_permissions` table: This table stores the permissions for each role.

## The "role_permissions" table

The table `role_permissions` has the following schema:

- `id`: The unique identifier for the role permission.
- `role`: The role for the team member.
- `permission`: The permission for the role.

## The "roles" table

The `roles` table has the following schema:

- `name`: The name of the role. This must be unique.
- `hierarchy_level`: The hierarchy level of the role.

### Hierarchy levels
We can use `hierarchy_level` to define the hierarchy of roles. For example, an `admin` role can have a higher hierarchy level than a `member` role. This will help you understand if a role has more permissions than another role.

### Adding new permissions

To add new permissions, we can use an enum for permissions `app_permissions`:

- `app_permissions` enum: This enum stores the permissions for each role.

By default, Makerkit comes with two roles: `owner` and `member` - and the following permissions:

```sql
create type public.app_permissions as enum(
  'roles.manage',
  'billing.manage',
  'settings.manage',
  'members.manage',
  'invites.manage'
);
```

You can add more roles and permissions as needed.

### Default roles and permissions

The default roles are defined as follows:

1. Members with `owner` role have full access to the application.
2. Members with `member` role have the following permissions: `members.manage` and `invites.manage`.

### Adding new roles and permissions

To add new permissions, you can update the `app_permissions` enum:

```sql
-- insert new permissions
alter type public.app_permissions add value 'tasks.write';
alter type public.app_permissions add value 'tasks.delete';
commit;
```

In the above, we added two new permissions: `tasks.write` and `tasks.delete`.

You can assign these permissions to roles in the `role_permissions` table for fine-grained access control:

```sql
insert into public.role_permissions (role, permission) values ('owner', 'tasks.write');
insert into public.role_permissions (role, permission) values ('owner', 'tasks.delete');
```

Of course - you will need to enforce these permissions in your application code and RLS.

### Using roles and permissions in RLS

To check if a user has a certain permission on an account, we can use the function `has_permission` - which you can use in your RLS to enforce permissions.

In the below, we create an RLS policy `insert_tasks` on the `tasks` table to check if a user can insert a new task. We use `public.has_permission` to check if the current user has the permission `tasks.write`:

```sql
create policy insert_tasks on public.tasks
    for insert
    to authenticated
    with check (
        public.has_permission(auth.uid(), account_id, 'tasks.write'::app_permissions)
    );
```

And now we can also add a policy to check if a user can delete a task:

```sql
create policy delete_tasks on public.tasks
    for delete
    to authenticated
    using (
        public.has_permission(auth.uid(), account_id, 'tasks.delete'::app_permissions)
    );
```

### Using roles and permissions in application code

You can use the exact same function `has_permission` in your application code to check if a user has a certain permission. You will call the function with the Supabase RPC method:

```tsx
async function hasPermissionToInsertTask(userId: string, accountId: string) {
  const { data: hasPermission, error } = await client.rpc('has_permission', {
    user_id: userId,
    account_id: accountId,
    permission: 'tasks.write',
  });

  if (error || !hasPermission) {
    throw new Error(`User has no permission to insert task`);
  }
}
```

You can now use `hasPermissionToInsertTask` to check if a user has permission to insert a task anywhere in your application code - provided you obtain the user and account IDs.

You can use this function to gate access to certain pages, or verify the user permissions before performing some server-side requests.

Of course, it's always worth making sure RLS is enforced on the database level as well.

### Using permissions client-side

While checks must be done always server-side, it is useful to have the permissions available client-side for UI purposes. For example, you may want to hide a certain button if the user does not have the permission to perform an action.

We fetch the permissions as part of the [Account Workspace API](account-workspace-api) - which is available to the layout around the account routes.

This API fetches the permissions for the current user and account and makes them available to the client-side simply by passing it from a page to the client components that require it.

Let's assume you have a page, and you want to check if the user has the permission to write tasks:

```tsx
import { loadTeamWorkspace } from '~/home/[account]/_lib/server/team-account-workspace.loader';

interface TasksPageProps {
  params: Promise<{ account: string }>
}

export default function TasksPage(props: TasksPageProps) {
  const slug = (await props.params).account
  const data = await loadTeamWorkspace(slug);
  const permissions = data.account.permissions; // string[]

  const canWriteTasks = permissions.includes('tasks.write');

  return (
    <div>
      {canWriteTasks && <button>Create Task</button>}
      // other UI elements // ...
    </div>
  );
}
```

You can also pass the permissions list to the components that need it as a prop.

This way, you can gate access to certain UI elements based on the user's permissions.

```tsx
import { loadTeamWorkspace } from '~/home/[account]/_lib/server/team-account-workspace.loader';

interface TasksPageProps {
  params: Promise<{ account: string }>
}

export default function TasksPage(props: TasksPageProps) {
  const slug = (await props.params).account
  const data = await loadTeamWorkspace(slug);
  const permissions = data.account.permissions; // string[]

  return (
    <div>
      <TaskList permissions={permissions} />
    </div>
  );
}
```

Similarly, you can use the permissions to gate access to certain routes or pages.

```tsx
import { loadTeamWorkspace } from '~/home/[account]/_lib/server/team-account-workspace.loader';

interface TasksPageProps {
  params: Promise<{ account: string }>
}

export default function TasksPage(props: TasksPageProps) {
  const slug = (await props.params).account
  const data = await loadTeamWorkspace(slug);
  const permissions = data.account.permissions; // string[]

  if (!permissions.includes('tasks.read')) {
    return <ErrorPage message="You do not have permission to write tasks" />;
  }

  return (
    <div>
      <TaskList permissions={permissions} />
    </div>
  );
}
```

Easy as that!


-----------------
FILE PATH: kits\next-supabase-turbo\development\seo.mdoc

---
status: "published"
label: "SEO"
title: SEO - Improve your Next.js application's search engine optimization"
description:  "Learn how to improve your Makerkit application's search engine optimization (SEO)"
order: 10
---

{% sequence title="How to improve your Makerkit application's SEO" description="Learn how to improve your Makerkit application's search engine optimization (SEO)" %}

[Best Practices for SEO](#best-practices-for-seo)

[Adding pages to the sitemap](#adding-pages-to-the-sitemap)

[Adding your website to Google Search Console](#adding-your-website-to-google-search-console)

[Backlinks](#backlinks)

[Content](#content)

[Indexing and ranking take time](#indexing-and-ranking-take-time)

{% /sequence %}

SEO is an important part of building a website. It helps search engines understand your website and rank it higher in search results. In this guide, you'll learn how to improve your Makerkit application's search engine optimization (SEO).

## Best Practices for SEO

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

If you add more static pages to your website, you can add them to the `getPaths` function in the `apps/web/app/server-sitemap.xml/route.ts` file. For example, if you add a new page at `/about`, you can add it to the `getPaths` function like this:

 ```tsx {% title="apps/web/app/server-sitemap.xml/route.ts" %}
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


