-----------------
FILE PATH: kits/next-supabase/how-to/troubleshooting/supabase-not-stopping.mdoc

---
status: "published"

title: "How to fix: Supabase is not stopping"
label: "Supabase is not stopping"
order: 1
description: "Learn why Supabase may not be able to stop in certain cases and how to fix it."
---


Sometimes, Supabase may not be able to stop. In most cases, this happens because you have multiple projects running, and are trying to stop it from the wrong project.

Try to run the following command to discover the projects that are running in Docker:

```bash
docker ps
```

If you want to kill them all at once, also making Supabase stop, you can run the following command:

```bash
docker kill $(docker ps -q)
```

NB: this command will shut down **all running Docker containers**.

-----------------
FILE PATH: kits/next-supabase/how-to/troubleshooting/tables-not-found.mdoc

---
status: "published"
title: "How to fix: Supabase tables not found | Next.js Supabase SaaS Kit"
label: "Tables/Functions not found"
order: 2
description: "How to fix the error: Tables/Functions not found when running Supabase locally or remotely."
---

When you try to start your Makerkit SaaS and try to navigate to your application, you may hit an error that says that the SQL tables or functions were not found.

This can happen for two reasons:

1. You're running Supabase locally and the migrations were not applied correctly.
2. You're running Supabase remotely and the migrations were not pushed to Supabase.
3. Something else is wrong with your Supabase instance. In case contact me or the Supabase team.

## This is happening when running Supabase locally

If this is happening when running Supabase locally, it means that the migrations weren't applied correctly. In this case, I recommend restarting Supabase and running the migrations again.

```
npm run supabase:stop
npm run supabase:start
```

### This is happening in the Supabase Cloud Instance

Instead - if this is happening in your remote instance, it means you did not push your migrations to the remote instance. In this case, I recommend pushing your migrations to the remote instance.

To push our database to Supabase, we need to link the Supabase CLI to our project.

You'll need to grab your project's reference ID from the URL: it's the part that comes after `app.supabase.io/` in the URL or the segment that comes before `supabase.co/` in the URL, such as `*******.supabase.co`.

We can do this by running the following command:

```bash
./node_modules/.bin/supabase link --project-ref **************
```

When prompted for the database password, enter the password you created when creating your project.

At this point, we can proceed to push our database to Supabase. Run the following command to push our database to Supabase:

```bash
./node_modules/.bin/supabase db push
```

The output should look like this:

```bash
Applying migration 20230705082911_schema.sql...
Finished supabase db push.
```

To verify that our database was pushed successfully, navigate to your project's dashboard and verify the tables were created.

-----------------
FILE PATH: kits/next-supabase/how-to/ui/dark-theme.mdoc

---
status: "published"
title: "Setting up the dark theme in Next.js Supabase"
label: Dark Theme
order: 1
description: "How to set up the dark theme in Next.js Supabase"
---

Makerkit supports by default a dark theme. With that said, you can decide to opt-out of the dark theme and use the light theme only instead using the configuration below.

## Using the dark theme by default

To use the dark theme by default, you need to edit the following configuration to your `src/configuration.ts` file:

 ```tsx {% title="src/configuration.ts" %}
{
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
}
```

Users can still opt-out of the dark theme by using the theme switcher in the top right corner of the page.

## Using only the dark theme

To switch to the dark theme, you need to edit the following configuration to your `src/configuration.ts` file:

 ```tsx {% title="src/configuration.ts" %}
{
  features: {
    enableThemeSwitcher: false,
  },
  theme: Themes.Dark,
}
```

## Disabling the dark theme

To disable the dark theme and only use the light theme - you need to edit the following configuration to your `src/configuration.ts` file:

 ```tsx {% title="src/configuration.ts" %}
{
  features: {
    enableThemeSwitcher: false,
  },
  theme: Themes.Light,
}
```

In the above, we flipped the `features.enableThemeSwitcher` flag to `false` and
set the `theme` to `Themes.Light`.

-----------------
FILE PATH: kits/next-supabase/how-to/ui/theming.mdoc

---
status: "published"
title: Theming | Next.js Supabase SaaS Kit
label: Theming
order: 1
description: "Discover how to apply the ShadCN UI theme to your Next.js Supabase SaaS Kit project."
---

Makerkit's theming is defined in two places:

1. the `global.css` file
2. the `tailwind.config.js` file.

## Colors

To update the theme's colors, please update the `global.css` file in your project and update the CSS variables with your color palette.

### Using the ShadCN themes

The ShadCN UI themes are predefined themes that you can use in your Makerkit project too.

You can [copy/paste the ShadCN themes](https://ui.shadcn.com/themes) into your `global.css` file and update the CSS variables to use the Makerkit color palette.

Next - you need to update the `tailwind.config.js` file in your project and update the `colors` object. These should match the CSS variables you have defined in the `global.css` file.

### Updating the Tailwind configuration

Additionally, we need to update the Tailwind configuration: since Makerkit uses a color palette based on Tailwind's palette system, we need to update the `tailwind.config.js` file in your project and update the `colors` object.

For example, since the default color for the `primary` color is `violet.500`, we need to update the `colors` object to:

```js
primary: {
  DEFAULT: 'hsl(var(--primary))',
  foreground: 'hsl(var(--primary-foreground))',
  ...colors.violet,
}
```

The code above spreads the `violet` color palette into the `primary` object, which means that the `primary` color will be the `violet.500` color.

The variable `--primary` is the hsl value of the `violet.500` color.

Additionally, we can access other colors of the `primary` palette using `primary-500`, `primary-600`, etc. This helps us to access the color palette in a more semantic way.

### Dark Mode

We need to do the same for the dark mode. Makerkit has historically used the `dark` color palette convention to define the dark mode colors.

By default, Makerkit uses `slate` as the dark mode color palette, so we need to update the `colors` object to:

```js
dark: {
  ...colors.slate,
  DEFAULT: colors.slate[950],
  foreground: colors.slate[100],
}
```

As you can see, we spread the `slate` color palette into the `dark` object, which means that the `dark` color will be the `slate.950` color.

If you were to use another color such as `zinc`, you would need to update the `colors` object to:

```js
dark: {
  ...colors.zinc,
  DEFAULT: colors.zinc[950],
  foreground: colors.zinc[100],
}
```

## Icons

While ShadCN UI uses `lucide` as their icons library, Makerkit has historically used `heroicons`. We have configured ShadCN UI to use `heroicons` instead of `lucide` in all Makerkit projects, but for new components, you will need to use `lucide` instead if you don't want to make changes. As such, please install `lucide` as a dependency in your project.

-----------------
FILE PATH: kits/next-supabase/how-to.mdoc

---
status: "published"
title: "How to in Next.js Supabase"
label: "How to"
order: 16
description: "How to in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/migrations/version-0.2.0.mdoc

---
status: "published"

title: "Migrating to 0.2.0"
label: " Migrating to 0.2.0"
order: 0
---


The 0.2.0 release of the kit introduces a number of breaking changes. This guide will help you migrate your code to the new version.

NB: If you have cloned the repository after the 0.2.0 release, you can skip this guide.

---


The changes are being introduced to make the kit compatible with the changes introduced to Next.js 13.4.0. This release introduced the following changes:
- **1. Client-side caching**: pages are cached for 30 seconds by default. This introduced bugs into the kit, where the current organization was not being updated when the user switched organizations and navigating to a cached page. We mitigate this by using dynamic routing using the organization UUID as part of the URL.
- **2. Server Actions**: since they're easier to use than traditional API Route, we updated all the mutations to use server actions.

## A new "dashboard" folder

Underneath the `app` folder, we added an additional prefix for routes that are organization-specific, called `dashboard`. The path `/dashboard/{uid}` is now the home page.

- **1. Move folders to app/dashboard**: To migrate to this new folder, you will be moving all the folders currently inside `app/(app)` to `app/dashboard`.

- **2. Update imports**: You will also need to update all the imports to reflect the new folder structure. To find all the imports, you can run the following command in your terminal:

```bash
npm run typechecker
```

This will show you all the files that need to be updated since they'll be throwing an error.

## Organizations now are referenced publicly by UUID

We added a UUID property to the organization so they can be part of the URL, without tipping of its sequential number such as the ID.

The migration will add the UUID to all the organizations in your database.

### Getting the organization UUID from the URL

Since the organization UUID is now part of the URL, you no longer need to use the cookie to get the current organization.

This parameter is accessible from all the pages and layouts in which it is defined. To access it, you can use the following code:

```tsx
interface LayoutProps {
  params: {
    organization: string;
  }
}

function MyLayout(props: LayoutProps) {
  const organizationUid = props.params.organization;

  return (
    <div>
      <h1>Organization UID: {organizationUid}</h1>
    </div>
  )
}
```

**The same also applies to pages**.

You can use the organization UUID to fetch the organization data from the API by matching the property `organization.uuid` with the UUID in the URL.

For example, below is the code to fetch the organization data:

```tsx
export function getOrganizationByUid(client: SupabaseClient, uid: string) {
  return client
    .from('organizations')
    .select(`
      *
    `)
    .eq('uuid', uid)
    .maybeSingle();
}
```

## Migrating the Tasks Demo

Here are the steps I took to update the tasks demo.

### Pull the changes from the upstream repository

Run the following command in your terminal (requires you setup the upstream repository):

```bash
git pull upstream main
```

### Resolve conflicts

You will need to resolve the conflicts for several files.

I got conflicts in two files:

{% img src="/assets/images/docs/merge-conflcts-0.2.0-migration.webp" height="1026" width="1582" /%}

You will have to adjust the code to match the new folder structure. The navigation configuration now adds the organization UID to the URLs:

```tsx
const NAVIGATION_CONFIG = (organization: string) => ({
  items: [
    {
      label: 'common:dashboardTabLabel',
      path: getPath(organization, ''),
      Icon: ({ className }: { className: string }) => {
        return <Squares2X2Icon className={className} />;
      },
      end: true,
    },
    {
      label: 'common:settingsTabLabel',
      path: getPath(organization, 'settings'),
      Icon: ({ className }: { className: string }) => {
        return <Cog8ToothIcon className={className} />;
      },
    },
  ],
});
```

The easier way is to accept the changes from the upstream repository and then manually update the code.

### Move the folders to the new location

Your files that were added on top of the core kit, will have stayed at `(app)`. The Makerkit files, instead, have been moved to `(app)/dashboard`. You will now need to move them to `(app)/dashboard` as well.

```
- src
  - app
    - (app) <-- move to dashboard below
  - dashboard
    -
```

### Running the Typechecker to find all the imports that need to be updated

When running the typechecker, I get the following errors:

```
npm run typecheck

> next-supabase-saas-kit@0.2.0 typecheck
> tsc -b && tsc -b cypress

src/app/dashboard/[organization]/tasks/page.tsx:8:23 - error TS2307: Cannot find module '~/app/(app)/components/AppHeader' or its corresponding type declarations.

8 import AppHeader from '~/app/(app)/components/AppHeader';
                        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/app/dashboard/[organization]/tasks/page.tsx:10:26 - error TS2307: Cannot find module '~/app/(app)/components/AppContainer' or its corresponding type declarations.

10 import AppContainer from '~/app/(app)/components/AppContainer';
                            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/app/dashboard/[organization]/tasks/[task]/page.tsx:10:23 - error TS2307: Cannot find module '~/app/(app)/components/AppHeader' or its corresponding type declarations.

10 import AppHeader from '~/app/(app)/components/AppHeader';
                         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/app/dashboard/[organization]/tasks/[task]/page.tsx:11:26 - error TS2307: Cannot find module '~/app/(app)/components/AppContainer' or its corresponding type declarations.

11 import AppContainer from '~/app/(app)/components/AppContainer';
                            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/app/dashboard/[organization]/tasks/new/page.tsx:3:23 - error TS2307: Cannot find module '~/app/(app)/components/AppHeader' or its corresponding type declarations.

3 import AppHeader from '~/app/(app)/components/AppHeader';
                        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/app/dashboard/[organization]/tasks/new/page.tsx:5:26 - error TS2307: Cannot find module '~/app/(app)/components/AppContainer' or its corresponding type declarations.

5 import AppContainer from '~/app/(app)/components/AppContainer';
```

To fix this, I was able to search and replace all the occurrences of `(app)` with `dashboard/[organization]`.

🎉 And that's it! You should now be able to run the app and see the new changes.


-----------------
FILE PATH: kits/next-supabase/migrations/version-0.3.0.mdoc

---
status: "published"

title: "Migrating to 0.3.0"
label: "Migrating to 0.3.0"
order: 1
---


The 0.3.0 release of the kit introduces a number of breaking changes. This guide will help you migrate your code to the new version.

NB: If you have cloned the repository after the 0.3.0 release, you can skip this guide.

---


The changes impact the following areas:

1. Tailwind CSS classes
2. Installed Packages

## Tailwind CSS classes

We have updates the Tailwind CSS classes using a new convention named `dark`, which is used for dark mode. This is a breaking change, as it will impact the way you use the Tailwind CSS classes.

Before, we used to have a `black` color - but it is no longer available. Instead, we have a `dark` color, which is used for dark mode. This means that you will need to update your code to use the new `dark` color.

Additionally, the new color uses a different level of opacity, which is more suitable to the standard Tailwind CSS convention, and is more consistent with the other colors.

By default, the `dark` color is an alias for `colors.gray` from the Tailwind CSS palette. You can change it by updating the `colors.dark` property in the `tailwind.config.js` file.

For example, you can provide your own palette, or switch to another dark color such as `colors.slate`.

 ```js {% title="tailwind.config.js" %}
const colors = require('tailwindcss/colors');

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./app/**/*.{ts,tsx,jsx,js}'],
  darkMode: 'class',
  theme: {
    extend: {
      fontFamily: {
        serif: ['serif'],
        heading: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
        sans: [
          'system-ui',
          'BlinkMacSystemFont',
          'Inter',
          'Segoe UI',
          'Roboto',
          'Ubuntu',
        ],
        monospace: [`SF Mono`, `ui-monospace`, `Monaco`, 'Monospace'],
      },
      colors: {
        primary: {
          ...colors.indigo,
          contrast: '#fff',
        },
        dark: colors.slate,
      },
    },
  },
};
```

### Automating the migration

To automate the migration, you can use the following script. First, install the package (either globally or locally):

```bash
npm i replace-in-file
```

Then, copy this script into your project named "migrate-0.3.js":

```js title="migrate-0.3.js
const replace = require('replace-in-file');
const files = ['src/**/*.tsx', 'src/**/*.css'];

const promise = () =>
  new Promise((resolve, reject) => {
    replace({
      ...getConfigs(),
      files,
    })
      .then((results) => {
        results.forEach((result) => {
          if (result.hasChanged) {
            console.log('File changed:', result.file);
          }
        });

        resolve();
      })
      .catch((error) => {
        console.error('Error occurred:', error);

        reject(error);
      });
  });

(async () => {
  console.log(`Replacing classes in "${files}" ...`);
  await promise();
  console.log('Done!');
})();

function getConfigs() {
  return {
    from: [
      /black-600/g,
      /black-500/g,
      /black-400/g,
      /black-300/g,
      /black-200/g,
      /black-100/g,
      /black-50/g,
      "'classnames'",
    ],
    to: [
      'dark-900',
      'dark-900',
      'dark-800',
      'dark-700',
      'dark-600',
      'dark-500',
      'dark-400',
      "'clsx'",
    ],
  };
}
```

Then, run the script:

```bash
node migrate-0.3.js
```

This will replace all the classes in your project.

## Packages

### Replaced "classnames" with "clsx"

Additionally, we replaced the package `classNames` with a newer, smaller, faster and maintained package named `clsx`. This is a breaking change, as it will impact the way you import the package.

The command above will also replace the import statement for you. However, if you want to do it manually, rename every instance of `'classnames'` with `'clsx'` in your project.

### Removed "Headless UI"

We have removed the package `headlessui` from the kit. It's slowly maintained and we don't use it anymore.

This is a breaking change, as it will impact the way you import the package.

If you have components using `headlessui`, please keep it in your project. Otherwise, you can remove it.

## Conclusion

This guide should help you migrate your project to the new version of the kit. If you have any questions, please get in touch with me.

-----------------
FILE PATH: kits/next-supabase/migrations.mdoc

---
status: "published"
title: "Migrations in Next.js Supabase"
label: "Migrations"
order: 11
description: "Migrations in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/organizations/organization-hooks.mdoc

---
status: "published"
title: 'Organization React Hooks | Next.js Supabase SaaS Kit'
label: Custom React Hooks
order: 10
description: 'Reference for the organizations React hooks in your Next.js Supabase SaaS Kit'
---

## useCurrentOrganization

This hook returns the current organization. It's a wrapper around the `OrganizationContext` context. The returned value is an `Organization` object.

```tsx
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const organzation = useCurrentOrganization();
```

## useIsSubscriptionActive

This hook returns a boolean indicating whether the current organization has
an active subscription. It's a shortcut around the `useCurrentOrganization`
hook that checks if the `status` property is set to `active` or `trialing`.

```tsx
import { useIsSubscriptionActive } from '~/lib/organizations/hooks/use-is-subscription-active';

const isSubscriptionActive = useIsSubscriptionActive();
```

## Other Organization Hooks

The kit has many more hooks that are used internally: they are not meant to be used directly unless you want to update the logic or fix a bug.

All the hooks are available in the `~/lib/organizations/hooks` directory.

-----------------
FILE PATH: kits/next-supabase/organizations/organizations-overview.mdoc

---
status: "published"

title: MakerKit's Organizations
label: Overview
order: 0
---


The Makerkit boilerplate's data model is modeled around *Organizations*. 

Organizations are groups of users, which can also be named Projects, Teams, and so on: it really depends on your application's domain.

The data model of an organization looks like the following:

```sql
create table organizations (
  id bigint generated always as identity primary key,
  name text not null,
  logo_url text,
  created_at timestamptz not null default now()
);
```

As you can see it's a very simple data model, with just a name and a logo, but you can extend it as you wish.

### Renaming "Organizations" to another entity

Makerkit uses "organizations" to refer to a group of users. However, the name "organizations" may not make sense to your application's domain, even with a similar technical data model. 

For example, you may want to call them "Teams", or "Workspaces". In this case, we suggest you keep using "organization" as a technical concept (otherwise, you'd have to rename every instance) within the codebase while changing the labels in the UI using the i18n files at `public/locales`. 

To do so, simply rename all "Organization" instances according to your preferred choice.

-----------------
FILE PATH: kits/next-supabase/organizations/permissions.mdoc

---
status: "published"
title: Managing User Permissions in Next.js Supabase
label: User Permissions
description: "Learn how to write a simple permissions system based on the users' role in your Makerkit applications using Next.js Supabase"
order: 2
---

Most permissions are written in a single file at `src/lib/organizations/permissions.ts`.

Here, you can find some of the examples used in the boilerplate so that you can start writing your own.

Why are permissions written in a single file? Because it's easy to write inline logic and lose track of it. Therefore, we will write all the business logic within the same file and encapsulated as simple functions.

Let's take a look at a simple permission function in the boilerplate:

 ```tsx {% title="src/lib/organizations/permissions.ts" %}
/**
 *
 * @param currentUserRole The current logged-in user
 * @param targetUser The role of the target of the action
 * @description Checks if a user can perform actions (such as update a role) of another user
 * @name canUpdateUser
 */
export function canUpdateUser(
  currentUserRole: MembershipRole,
  targetUser: MembershipRole
) {
  return currentUserRole > targetUser;
}
```

The function takes two parameters: the current user's role and the target user's role, and checks if the current user can update the target user's role (or anything).

Now, we can use the function above with the `IfHasPermissions` component to display or hide some parts of the application. This component automatically injects the current user's role, such as below:

```tsx
<IfHasPermissions
  condition={(currentUserRole) =>
    canInviteUser(currentUserRole, targetUserRole)
  }
>
  <InviteUserComponent />
</IfHasPermissions>
```

The `InviteUserComponent` component will be displayed if the condition is truthy.

Otherwise, you can use these functions throughout the application on both the client and the server.


-----------------
FILE PATH: kits/next-supabase/organizations/row-level-security.mdoc

---
status: "published"

title: "Common Row-Level-Security Policies you can apply to your data"
label: "RLS Policies"
description: "To protect your data, you can apply row-level-security policies to your data. This article lists some common policies you can apply to your data"
order: 4
---


Supabase uses RLS policies to protect your data. RLS are useful for two main reasons:
1. Supabase exposes your database directly to your clients. This means that your clients can query your database directly. RLS policies allow you to restrict what data they can access.
2. You can use SQL policies to apply access rules to your data. This helps you to keep your data secure without having to write any additional code.

In this document, we'll list some common RLS policies you can apply to your data in Makerkit.

You can copy and paste some of this (provided the table names match) into your database so that you can get started quickly.

## Common RLS Policies

### Restricting access to a table based on a column

This policy will restrict access to a table based on a column. In this example, we're restricting access to the `users` table based on the `id` column.

```sql
create policy "Restrict access to user tasks to their own tasks"
  on tasks
  for select
  to authenticated
  using (user_id = auth.uid());
```

### Restricting write access to a table based on a column

This policy will restrict write access to a table based on a column. In this example, we're restricting write access to the `users` table based on the `id` column.

```sql
create policy "Restrict write access to authenticated users"
  on users
  for insert
  to authenticated
  with check (id = auth.uid());
```

Truthfully, this is not a very useful policy. It's unlikely that you'll want to restrict access to a table based on the user ID in a multi-tenant application. However, it's a good example if you store per-user data in a table.

#### Restrictive Policies

If you want these rules to be `restrictive` (i.e. they apply to all queries), then you can add `as restrictive`:

```sql
create policy "Restrict write access to authenticated users"
  on users
  as restrictive;
  for insert
  to authenticated
  with check (id = auth.uid())
```

This means that if you have 2 `select` policies, one `restrictive` and one `permissive`, then the `restrictive` policy will take precedence. If the `restrictive` policy fails, then the `permissive` policy will not apply.

This is useful when a policy needs to apply to all queries.

### Restricting access to a table based on a column and a role

In Makerkit - multi-tenancy is achieved using `organizations`. As such, you'll want to restrict access to data based on the `organization_id` column.

Let's create a policy that restricts access to the `tasks` table based on the `organization_id` column - taking into consideration the user's role in the organization.

User roles are defined using a hierarchy. The hierarchy is as follows:
1. Owner (role = 2)
2. Admin (role = 1)
3. Member (role = 0)

You can update this as you see fit - but this is the default hierarchy. Remember that the `role` column is stored in the `memberships` table and needs to be updates should you wish to change the hierarchy.


#### Create the function to get the user's role

```sql
create or replace function get_current_user_role(org_id bigint)
returns int as $$
declare
    user_role int;
begin
    select role
        from memberships
        where organization_id = org_id and user_id = auth.uid()
        into user_role;

    if user_role is null then
        raise exception 'User is not a member of the organization';
    end if;

    return user_role;
end;
$$ language plpgsql;
```

#### Create the policies

Now that we have the function to get the user's role, we can create the policies. The policy state that **only the owner can insert, update, or delete tasks**.

```sql
create or replace policy "Only the owner can update tasks"
    on tasks
    as restrictive
    for update
    to authenticated
    using (
        get_current_user_role(organization_id) >= 1
    ) with check (
        get_current_user_role(organization_id) >= 1
    );

create or replace policy "Only the owner can delete tasks"
    on tasks
    as restrictive
    for delete
    to authenticated
    using (
        get_current_user_role(organization_id) >= 1
    );
```

Let's now allow every organization member to read tasks.

We can use the built-in `current_user_is_member_of_organization` function to check if the user is a member of the organization.

```sql
create or replace policy "Only the owner can delete tasks"
    on tasks
    as restrictive
    for select
    to authenticated
    using (
        current_user_is_member_of_organization(organization_id)
    );
```

NB: when using a `select` policy, you don't need to use `with check`. This is because the `using` clause is sufficient to restrict access to the data. With that said, remember than failing policies won't throw an error - they'll just return no data.

We can also allow every member to create tasks within their organization using the `insert` policy:

```sql
create or replace policy "Only the owner can insert tasks"
    on tasks
    as restrictive
    for insert
    to authenticated
    with check (
        current_user_is_member_of_organization(organization_id)
    );
```

## Subscriptions and RLS

RLS policies are applied to subscriptions. This means that if you have a subscription that returns data that the user doesn't have access to - then the subscription will fail.

Let's see how we can use RLS for restricting access to subscriptions.

This is **very dependent on your use case**. You'll need to think about how you want to restrict access to your data. But for the sake of this example, let's assume that we want to restrict how many tasks a user can write to the database.

### Storing your Plans in the database

First, we'll need to store our plans in the database. We'll create a `plans` table that stores the `id`, `name`, and `max_tasks` for each plan.

You have two options here when talking about Stripe:
1. assign a plan to a Product ID
2. assign a plan to a Price ID

If you need different quotas for monthly and yearly plans - then you'll need to assign a plan to a Price ID. If you don't need different quotas for monthly and yearly plans - then you can assign a plan to a Product ID.

In the example below, we're assigning a plan to a Price ID.

```sql
create table plans (
  name text not null,
  price_id text not null,
  task_quota int not null,
  primary key (product_id)
);
```

We also allow read access to this table for all authenticated users:

```sql
create policy "Allow all authenticated users to read plans"
    on plans
    as restrictive
    for select
    to authenticated
    using (true);
```

#### A function to get the organization's subscription

Let's create an SQL function that returns the organization's subscription:

```sql
create or replace function get_active_subscription(org_id bigint)
returns table (
  period_starts_at timestamptz,
  period_ends_at timestamptz,
  price_id text,
  "interval" text
) as $$
begin
    return query select subscriptions.period_starts_at, subscriptions.period_ends_at, subscriptions.price_id, subscriptions."interval" from public.subscriptions
    join organizations_subscriptions on subscriptions.id = organizations_subscriptions.subscription_id
    where organizations_subscriptions.organization_id = org_id and (subscriptions.status = 'active' or subscriptions.status = 'trialing');
end;
$$ language plpgsql;
```

#### A function to get the organization's task count

Let's create an SQL function that returns the organization's task count:

```sql
create or replace function get_organization_task_count(org_id bigint)
returns int as $$
declare
    task_count int;
begin
    select count(*)
        from tasks
        where organization_id = org_id
        into task_count;

    return task_count;
end;
$$ language plpgsql;
```

#### A function to check if the organization can create a task

Let's create an SQL function that returns `true` if the organization can create a task:

```sql
create or replace function organization_can_create_task(org_id bigint)
returns boolean as $$
declare
    task_count int;
    organization_price_id text;
    plan_task_quota int;
begin
    select get_organization_task_count(org_id)
        into task_count;

    select price_id
    into organization_price_id
    from get_active_subscription(org_id);

    if organization_price_id is null then
        raise exception 'Organization does not have an active subscription';
    end if;

    select task_quota from plans where price_id = organization_price_id into plan_task_quota;

    return task_count < plan_task_quota;
end;
$$ language plpgsql;
```

Here we store the total tasks count in a variable:

```
select get_organization_task_count(org_id) into task_count;
```

Then we get the organization's price ID:

```
select price_id
    into organization_price_id
    from get_active_subscription(org_id);
```

If the organization doesn't have an active subscription - then we throw an error:

```
if organization_price_id is null then
    raise exception 'Organization does not have an active subscription';
end if;
```

What if you wanted to provide a default plan for organizations that don't have an active subscription? You could do something like this for allowing 10 tasks for organizations that don't have an active subscription:

```
if organization_price_id is null then
  return task_count < 10;
end if;
```

Then we get the plan's task quota:

```
select task_quota from plans where price_id = organization_price_id into plan_task_quota;
```

Finally, we check that the count of tasks is less than the plan's task quota:

```
return task_count < plan_task_quota;
```

#### Wrapping it all up in a policy

We can now write a policy that restricts access to the `tasks` table based on the organization's task quota:

```sql
create or replace policy "Only allow organizations to create tasks if they have enough quota"
    on tasks
    as restrictive
    for insert
    to authenticated
    with check (
        organization_can_create_task(organization_id)
    );
```

## You don't always need RLS

Sometimes - you don't need RLS. As long as RLS is enabled on the table - you can think about not applying any RLS policies.

Instead - you can manage access to the data using traditional server-side code. This is useful if you want to apply more complex access rules to your data or when the data is not directly updated by the client - but instead by a server-side process (such as a cron job or a webhook).

## Restricting Functions

Sometimes - you need a security definer functions. This is useful for bypassing RLS policies. But it's risky.

If you want to run a function - making sure only the admin service role ca call it, check the role of the user calling the function:

```sql
create or replace function create_task(user_id bigint, name text)
returns tasks as $$
begin
    if current_setting('role') != 'service_role' then
        raise exception 'Only admins can call this function';
    end if;

    insert into tasks (user_id, name) values (user_id, name);

    return (select * from tasks where id = currval('tasks_id_seq'));
end;
$$ language plpgsql security definer search_path = public;
```

This function will throw an error if it's called from the client. You can then call this function from the server using a service role key and ensure that the user has enough quota to create a task server-side before calling this function.

