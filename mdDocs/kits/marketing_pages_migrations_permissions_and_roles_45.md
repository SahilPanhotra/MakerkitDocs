-----------------
FILE PATH: kits\next-supabase-turbo\development\marketing-pages.mdoc

---
status: "published"

label: "Marketing Pages"
title: "Marketing Pages in the Next.js Supabase Turbo Starter Kit"
description: "Learn how to create and update marketing pages in the Next.js Supabase Turbo Starter Kit."
order: 7
---


Makerkit comes with pre-defined marketing pages to help you get started with your SaaS application. These pages are built with Next.js and Tailwind CSS and are located in the `apps/web/app/(marketing)` directory.

Makerkit comes with the following marketing pages:

- Home Page
- Contact Form
- Pricing Page
- FAQ
- Contact Page (with a contact form)

## Adding a new marketing page

To add a new marketing page to your Makerkit application, you need to follow these steps.

Create a folder in the `apps/web/app/(marketing)` directory with the path you want to use for the page. For example, to create a new page at `/about`, you would create a folder named `about`. Then, create the page file in the folder. For example, to create an `about` page, you would create an `page.tsx` file in the `about` folder.

```tsx
// apps/web/app/(marketing)/about/page.tsx
export default function AboutPage() {
  return <div></div>
}
```

This page inherits the layout at `apps/web/app/(marketing)/layout.tsx`. You can customize the layout by editing this file - but remember that it will affect all marketing pages.

## Contact Form

To make the contact form work, you need to add the following environment variables:

```bash
CONTACT_EMAIL=
```

In this variable, you need to set the email address where you want to receive the contact form submissions. The sender will be the same as the one configured in your [mailing configuration](email-configuration).

-----------------
FILE PATH: kits\next-supabase-turbo\development\migrations.mdoc

---
status: "published"
label: "Migrations"
order: 1
title: "How to create new migrations and update the database schema in your Next.js Supabase application"
description: "Learn how to create new migrations and update the database schema in your Next.js Supabase application"
---

Creating a schema for your data is one of the primary tasks when building a new application. In this guide, we'll walk through how to create new migrations and update the database schema in your Next.js Supabase application.

## A quick word on migrations

Supabase's hosted Studio is pretty great - but I don't think it should be used to perform schema changes. Instead, I recommend using your local Supabase Studio to make the changes and then generate the migration file. Then, you can push the migration to the remote Supabase instance.

At this point, you have two options:

1. create a migration with `pnpm --filter web supabase migration new <name>` and update the code manually
2. or, use the local Supabase Studio to make the changes and then run `pnpm --filter web supabase db diff -f <name>` which will generate the migration file for you. DOUBLY CHECK THE FILE!

Once you've tested it all and are happy with your local changes, push the migration to the remote Supabase instance with `pnpm --filter web supabase db push`.

Doing the opposite is also okay - but:

1. You're making changes against the production database - which is risky
2. You're not testing the changes locally - which is risky
3. You need to pull the changes from the remote Supabase instance to your local instance so they are in sync

## Creating a Migration

The first step towards building your app schema is to create a new migration.

To do so, run the command:

```
pnpm --filter web supabase migration new <name>
```

The migration will be generated at `apps/web/supabase/migrations`.

Once added some SQL commands, you need to reset the schema for it to take effect:

```
pnpm run supabase:web:reset
```

The schema is now populated! Yay!

### Generating Supabase types

Now that your schema is populated, you need to generate the types so that your Supabase client can work correctly.

Please run the following command:

```
pnpm run supabase:web:typegen
```

Your Supabase client will now correctly infer the types with your schema changes.

-----------------
FILE PATH: kits\next-supabase-turbo\development\permissions-and-roles.mdoc

---
status: "published"

label: 'RBAC: Roles and Permissions'
title: 'RBAC: Understanding roles and permissions in Next.js Supabase'
description: 'Learn how to implement roles and permissions in Next.js Supabase'
order: 6
---


Makerkit implements granular RBAC for team members. This allows you to define roles and permissions for each team member - giving you fine-grained control over who can access what.

Makerkit implements two tables for roles and permissions:

- `roles` table: This table stores the roles for each team member.
- `role_permissions` table: This table stores the permissions for each role.

The table `role_permissions` has the following schema:

- `id`: The unique identifier for the role permission.
- `role`: The role for the team member.
- `permission`: The permission for the role.

The `roles` table has the following schema:

- `name`: The name of the role. This must be unique.
- `hierarchy_level`: The hierarchy level of the role.

We can use `hierarchy_level` to define the hierarchy of roles. For example, an `admin` role can have a higher hierarchy level than a `member` role. This will help you understand if a role has more permissions than another role.

And an enum for permissions `app_permissions`:

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
    with check (
        public.has_permission(auth.uid(), account_id, 'tasks.write'::app_permissions)
    );
```

And now we can also add a policy to check if a user can delete a task:

```sql
create policy delete_tasks on public.tasks
    for delete
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

export default function TasksPage() {
  const data = await loadTeamWorkspace();
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

export default function TasksPage() {
  const data = await loadTeamWorkspace();
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

export default function TasksPage() {
  const data = await loadTeamWorkspace();
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


-----------------
FILE PATH: kits\next-supabase-turbo\development\writing-data-to-database.mdoc

---
status: "published"

label: "Writing data to Database"
order: 5
title: "Learn how to write data to the Supabase database"
description: "In this page we learn how to write data to the Supabase database in your Next.js app"
---


In this page, we will learn how to write data to the Supabase database in your Next.js app.

## Writing a Server Action to Add a Task

Server Actions are defined by adding `use server` at the top of the function or file. When we define a function as a Server Action, it will be executed on the server-side.

This is useful for various reasons:
1. By using Server Actions, we can revalidate data fetched through Server Components
2. We can execute server side code just by calling the function from the client side

In this example, we will write a Server Action to add a task to the database.

### Defining a Schema for the Task

We use Zod to validate the data that is passed to the Server Action. This ensures that the data is in the correct format before it is written to the database.

The convention in Makerkit is to define the schema in a separate file and import it where needed. We use the convention `file.schema.ts` to define the schema.

```tsx
import { z } from 'zod';

export const WriteTaskSchema = z.object({
  title: z.string().min(1),
  description: z.string().nullable(),
});
```

### Writing the Server Action to Add a Task

In this example, we write a Server Action to add a task to the database. We use the `revalidatePath` function to revalidate the `/home` page after the task is added.

```tsx
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

import { z } from 'zod';

import { getLogger } from '@kit/shared/logger';
import { requireUser } from '@kit/supabase/require-user';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

import { WriteTaskSchema } from '~/(dashboard)/home/(user)/_lib/schema/write-task.schema';

export async function addTaskAction(params: z.infer<typeof WriteTaskSchema>) {
  'use server';

  const task = WriteTaskSchema.parse(params);

  const logger = await getLogger();
  const client = getSupabaseServerClient();
  const auth = await requireUser(client);

  if (!auth.data) {
    redirect(auth.redirectTo);
  }

  logger.info(task, `Adding task...`);

  const { data, error } = await client
    .from('tasks')
    .insert({ ...task, account_id: auth.data.id });

  if (error) {
    logger.error(error, `Failed to add task`);

    throw new Error(`Failed to add task`);
  }

  logger.info(data, `Task added successfully`);

  revalidatePath('/home', 'page');

  return null;
}
```

Let's focus on this bit for a second:

```tsx
const { data, error } = await client
    .from('tasks')
    .insert({ ...task, account_id: auth.data.id });
```

Do you see the `account_id` field? This is a foreign key that links the task to the user who created it. This is a common pattern in database design.

Now that we have written the Server Action to add a task, we can call this function from the client side. But we need a form, which we define in the next section.

### Creating a Form to Add a Task

We create a form to add a task. The form is a React component that accepts a `SubmitButton` prop and an `onSubmit` prop.

```tsx
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@kit/ui/form';
import { Input } from '@kit/ui/input';
import { Textarea } from '@kit/ui/textarea';
import { Trans } from '@kit/ui/trans';

import { WriteTaskSchema } from '../_lib/schema/write-task.schema';

export function TaskForm(props: {
  task?: z.infer<typeof WriteTaskSchema>;
  onSubmit: (task: z.infer<typeof WriteTaskSchema>) => void;
  SubmitButton: React.ComponentType;
}) {
  const form = useForm({
    resolver: zodResolver(WriteTaskSchema),
    defaultValues: props.task,
  });

  return (
    <Form {...form}>
      <form
        className={'flex flex-col space-y-4'}
        onSubmit={form.handleSubmit(props.onSubmit)}
      >
        <FormField
          render={(item) => {
            return (
              <FormItem>
                <FormLabel>
                  <Trans i18nKey={'tasks:taskTitle'} />
                </FormLabel>

                <FormControl>
                  <Input required {...item.field} />
                </FormControl>

                <FormDescription>
                  <Trans i18nKey={'tasks:taskTitleDescription'} />
                </FormDescription>

                <FormMessage />
              </FormItem>
            );
          }}
          name={'title'}
        />

        <FormField
          render={(item) => {
            return (
              <FormItem>
                <FormLabel>
                  <Trans i18nKey={'tasks:taskDescription'} />
                </FormLabel>

                <FormControl>
                  <Textarea {...item.field} />
                </FormControl>

                <FormDescription>
                  <Trans i18nKey={'tasks:taskDescriptionDescription'} />
                </FormDescription>

                <FormMessage />
              </FormItem>
            );
          }}
          name={'description'}
        />

        <props.SubmitButton />
      </form>
    </Form>
  );
}
```

### Using a Dialog component to display the form

We use the Dialog component from the `@kit/ui/dialog` package to display the form in a dialog. The dialog is opened when the user clicks on a button.

```tsx
'use client';

import { useState, useTransition } from 'react';

import { PlusCircle } from 'lucide-react';

import { Button } from '@kit/ui/button';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@kit/ui/dialog';
import { Trans } from '@kit/ui/trans';

import { TaskForm } from '../_components/task-form';
import { addTaskAction } from '../_lib/server/server-actions';

export function NewTaskDialog() {
  const [pending, startTransition] = useTransition();
  const [isOpen, setIsOpen] = useState(false);

  return (
    <Dialog open={isOpen} onOpenChange={setIsOpen}>
      <DialogTrigger asChild>
        <Button>
          <PlusCircle className={'mr-1 h-4'} />
          <span>
            <Trans i18nKey={'tasks:addNewTask'} />
          </span>
        </Button>
      </DialogTrigger>

      <DialogContent>
        <DialogHeader>
          <DialogTitle>
            <Trans i18nKey={'tasks:addNewTask'} />
          </DialogTitle>

          <DialogDescription>
            <Trans i18nKey={'tasks:addNewTaskDescription'} />
          </DialogDescription>
        </DialogHeader>

        <TaskForm
          SubmitButton={() => (
            <Button>
              {pending ? (
                <Trans i18nKey={'tasks:addingTask'} />
              ) : (
                <Trans i18nKey={'tasks:addTask'} />
              )}
            </Button>
          )}
          onSubmit={(data) => {
            startTransition(async () => {
              await addTaskAction(data);
              setIsOpen(false);
            });
          }}
        />
      </DialogContent>
    </Dialog>
  );
}
```

We can now import `NewTaskDialog` in the `/home` page and display the dialog when the user clicks on a button.

Let's go back to the home page and add the component right next to the input filter:

```tsx {18}
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

    <NewTaskDialog />
  </div>
</div>
```

-----------------
FILE PATH: kits\next-supabase-turbo\development.mdoc

---
status: "published"
title: "Development in Next.js Supabase Turbo"
label: "Development"
order: 4
description: "This section will teach you everything you need to know about developing with Makerkit."
collapsible: true
collapsed: true
---

Your roadmap to building with Makerkit. Here's what you need to know about development:

1. [**Getting Started with Local Development**](/docs/next-supabase-turbo/approaching-local-development)
   Set up your local development environment and understand the development workflow.
2. [**Working with Turborepo**](/docs/next-supabase-turbo/adding-turborepo-app)
   - [Adding a Turborepo App](/docs/next-supabase-turbo/adding-turborepo-app)
   - [Adding a Turborepo Package](/docs/next-supabase-turbo/adding-turborepo-package)
3. [**Database Development**](/docs/next-supabase-turbo/database-schema)
   - [Database Schema](/docs/next-supabase-turbo/database-schema)
   - [Database Functions](/docs/next-supabase-turbo/database-functions)
   - [Database Webhooks](/docs/next-supabase-turbo/database-webhooks)
   - [Working with Migrations](/docs/next-supabase-turbo/migrations)
4. [**Data Operations**](/docs/next-supabase-turbo/loading-data-from-database)
   - [Loading Data from Database](/docs/next-supabase-turbo/loading-data-from-database)
   - [Writing Data to Database](/docs/next-supabase-turbo/writing-data-to-database)
5. [**Access Control**](/docs/next-supabase-turbo/permissions-and-roles)
   Learn about roles, permissions, and access control in your application.
6. [**Content Management**](/docs/next-supabase-turbo/marketing-pages)
   - [Marketing Pages](/docs/next-supabase-turbo/marketing-pages)
   - [Legal Pages](/docs/next-supabase-turbo/legal-pages)
   - [External Marketing Website](/docs/next-supabase-turbo/external-marketing-website)
   - [Search Engine Optimization](/docs/next-supabase-turbo/seo)

Ready to start building? Pick a topic above and dive in! 🚀

-----------------
FILE PATH: kits\next-supabase-turbo\emails\authentication-emails.mdoc

---
status: "published"
title: "Setting the Supabase Auth Email Templates in Production"
label: "Authentication Emails"
description: "Configure the Supabase authentication URLs and emails in the Next.js Supabase Starter Kit."
order: 4
---

Makerkit provides a set of email templates that you can use for replacing the standard Supabase Auth emails. Additionally, you can customize these templates to match your brand.

**Please update the auth emails using the [following documentation in Supabase](https://supabase.com/docs/guides/auth/auth-email-templates#redirecting-the-user-to-a-server-side-endpoint)**.

{% alert type="warning" %}
Failure to do so will result in hiccups in the authentication flow when users click on an email and get redirected to a different browser than the one they used to sign up due to how the PKCE flow works.
{% /alert %}

## Setting the Email Templates in Supabase

Why should you use our email templates?

1. They will use the token hash strategy, which remediates the issue of users being redirected to a different browser than the one they used to sign up.
2. They look better than the default Supabase templates and you can customize them to match your brand.

## Customizing the Email Templates

Please clone the templates repository locally and customize them to your liking:

1. Clone our Emails Starter at [https://github.com/makerkit/makerkit-emails-starter](https://github.com/makerkit/makerkit-emails-starter)
2. Customize the templates as you see fit
3. Export the templates with your own information
4. Replace the templates in the `apps/web/supabase/templates` folder
5. Update the email templates in your Supabase settings

Now your emails from Supabase Auth will look great and match your brand! 🎉

