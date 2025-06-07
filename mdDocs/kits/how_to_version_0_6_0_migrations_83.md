-----------------
FILE PATH: kits/remix-supabase/how-to.mdoc

---
status: "published"
title: "How to in Remix Supabase"
label: "How to"
order: 16
description: "How to in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/migrations/version-0.6.0.mdoc

---
status: "published"

title: "Migrating to 0.6.0"
label: "Migrating to 0.6.0"
order: 0
---


The 0.6.0 release of the kit introduces a number of breaking changes. This guide will help you migrate your code to the new version.

NB: If you have cloned the repository after the 0.6.0 release, you can skip this guide.

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

Then, copy this script into your project named "migrate-0.6.js":

```js title="migrate-0.6.js
const replace = require('replace-in-file');
const files = ['app/**/*.tsx', 'app/**/*.css'];

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
node migrate-0.6.js
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
FILE PATH: kits/remix-supabase/migrations.mdoc

---
status: "published"
title: "Migrations in Remix Supabase"
label: "Migrations"
order: 11
description: "Migrations in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/organizations/extending-organizations.mdoc

---
status: "published"
title: Extending Organizations in Remix.js Supabase SaaS Starter Kit
description: Learn how to extend the organization's data model in your Remix.js Supabase SaaS Starter Kit application
label: Extending Organizations
order: 1
---

There are many ways you can extend the organization's data model in your applications: you will use this entity for the objects that are shared among the group's users.

For example, here is a popular scenario: we assume users can install integrations for their organizations. Integrations can have the following interface:

 ```tsx {% title="integration.ts" %}
enum IntegrationType {
  Zapier,
  Notion,
  // etc
}

interface Integration<IntegrationData = unknown> {
  type: IntegrationType;
  data: IntegrationData;
}
```

Then, you can extend the organization's interface and add an `integrations` property to `Organization`, and initialize it as an empty array `[]` when you create a new integration:

 ```tsx {% title="organization.ts" %}
interface Organization {
  // ...
  integrations: Integration[];
}
```

-----------------
FILE PATH: kits/remix-supabase/organizations/organizations-overview.mdoc

---
status: "published"

title: MakerKit's Organizations
label: Overview
order: 0
---


The Makerkit boilerplate's data model is modeled around *Organizations*. 

Organizations are groups of users, which can also be named Projects, Teams, and so on: it really depends on your application's domain.

The data model of an organization looks like the following:

 ```tsx {% title="organization.ts" %}
type UserId = string;

interface Organization {
  name: string;
  timezone?: string;
  logoURL?: string | null;
  subscription?: OrganizationSubscription;
  customerId?: string;

  members: Record<UserId, {
    role: number;
    user: Reference;
  }>;
}
```

### Renaming "Organizations" to another entity

Makerkit uses "organizations" to refer to a group of users. However, the name "organizations" may not make sense to your application's domain, even with a similar technical data model. 

For example, you may want to call them "Teams", or "Workspaces". In this case, we suggest you keep using "organization" as a technical concept (otherwise, you'd have to rename every instance) within the codebase while changing the labels in the UI using the i18n files at `public/locales`. 

To do so, simply rename all "Organization" instances according to your preferred choice.

-----------------
FILE PATH: kits/remix-supabase/organizations/permissions.mdoc

---
status: "published"
title: Managing User Permissions in Remix Supabase
label: User Permissions
description: "Learn how to write a simple permissions system based on the
users' role in your Makerkit applications using Remix Supabase"
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
FILE PATH: kits/remix-supabase/organizations/row-level-security.mdoc

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

-----------------
FILE PATH: kits/remix-supabase/organizations/subscription-permissions.mdoc

---
status: "published"
title: Subscription Permissions | Remix Supabase SaaS Kit
label: Subscription Permissions
order: 3
description: "Learn how to gate access to certain features or pagess based
on the organization's subscription"
---

Makerkit provides some utilities to retrieve the current organization's
subscription: you can use these utilities to allow or gate access to
specific features, pages, or to perform certain actions.

1. On the client side, we can use the hook `useIsSubscriptionActive`
2. On the server-side, we can use the functions
`isOrganizationSubscriptionActive` and `getOrganizationSubscription`

## Client Side Gating

Let's assume we want to allow only users who belong to paying organizations to
perform a certain action:

 ```tsx {% title="src/lib/organizations/permissions.ts" %}
function isAdmin(
  role: MembershipRole
) {
  return role === MembershipRole.Admin;
}
```

Now, combine this function with other conditions:

 ```tsx {% title="src/lib/organizations/permissions.ts" %}
export function useCreateNewThing(
  userRole: MembershipRole,
) {
  const isPayingUser = useIsSubscriptionActive();

  return isPayingUser && isAdmin(userRole);
}
```

As you may know, I recommend adding all the permissions logic to the file
`permissions.ts` (or similar files), and never inline it: this makes it easy to
store logic in one single place rather than scattered across the
application.

Inlining logic in our code will make it hard to track logic
down and debugging it.

Once defined out logic functions in `permissions.ts`, we can then use these
in our components to hide the sections that users do
not have access to:

```tsx
function Feature() {
  const userRole = useCurrentUserRole();
  const canCreateThing = useCreateNewThing(userRole);

  if (!canCreateThing) {
    return <div>Sorry, you do not have access to this feature. Subscribe?</div>
  }

  return <FeatureContainer />;
}
```

## Server Side Gating

When gating access on the server side, we can use two utilities that allow
us to get the organization's subscription by its ID.

### Using permissions utilities

When you need to check the user's subscription in your API functions, you can
use the functions `isOrganizationSubscriptionActive` and
`getOrganizationSubscription` exported from
`src/lib/server/organizations/memberships.ts`.

For example, let's assume we have an API handler that performs an action:

```tsx
export default async function action({ request }) {
  const organization = await parseOrganizationCookie(request);

  const isSubscriptionActive =
    await isOrganizationSubscriptionActive(organization);

  if (!isSubscriptionActive) {
    return throwForbiddenError();
  }

  // all good! perform action
}
```


-----------------
FILE PATH: kits/remix-supabase/organizations.mdoc

---
status: "published"
title: "Organizations in Remix Supabase (legacy)"
label: "Organizations"
order: 11
description: "Learn how to add organizations to your Remix Supabase application"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/payments/lemon-squeezy.mdoc

---
status: "published"
title: Use Lemon Squeezy with MakerKit | Remix Supabase
label: Migrate to Lemon Squeezy
order: 4
description: 'Lear how to migrate your Remix Supabase project to Lemon Squeezy'
---

If you prefer using Lemon Squeezy instead of Stripe with your Makerkit kit, we provide a different branch from the main repository that you can use.

The branch is called `main-ls`. If you want to use Lemon Squeezy, you can clone the repository and checkout the `main-ls` branch.

NB: the integration is in beta, and I am working on improving it.

## Setting up Lemon Squeezy

### Step 1: Create a Lemon Squeezy account

First, you need to create a Lemon Squeezy account. You can do so by visiting [https://lemonsqueezy.io](https://lemonsqueezy.com).

Get your store setup (start in Test mode, so you don't need to wait for activation).

### Step 2: Create a Product/Variant in Lemon Squeezy

You can see this guide for [setting up SaaS subscription plans in Lemon Squeezy](https://docs.lemonsqueezy.com/guides/tutorials/saas-subscription-plans).

If you want, set up a monthly/annual plan for your product, or just a single plan.

{% img src="/assets/images/posts/lemon-squeezy-plans.webp" width="1286" height="816" /%}

To replicate the basic Makerkit subscription plans, you can create a plan for each of the following: Basic, Pro and Premium plans.

To get started quickly, you can also set up a single plan for your product.

### Step 3: Create a Lemon Squeezy API key

Now, we need to setup an API Key. Go to your Lemon Squeezy settings page and click on the [API Keys tab](https://app.lemonsqueezy.com/products).

Click on the "Create API Key" button and give it a **very specific name**, so you won't confuse it with your production keys. Copy the API key and ensure to store it. We will be adding this to your Makerkit's local environment variables file.

{% img src="/assets/images/posts/lemon-squeezy-api-key-name.webp" width="946" height="614" /%}

Depending on which kit you're using, you will add this key to either `.env.local` (Next.js) or `.env` (Remix). This key is not supposed to be committed to your repository, so we add them safely to our local files.

Additionally, we need to add the `LEMON_SQUEEZY_STORE_ID` to the local environment variables file, and a secret string `LEMON_SQUEEZY_SIGNING_SECRET` that we use to verify the webhook signature.

Update them in your local environment variables file:

```
LEMON_SQUEEZY_API_KEY=<YOUR_KEY>
LEMON_SQUEEZY_STORE_ID=<YOUR_STORE_ID>
LEMONS_SQUEEZY_SIGNATURE_SECRET=<a random string>
```

#### Find your Lemon Squeezy Store ID

To find your Lemon Squeezy Store ID, visit your `Stores` page in the Lemon Squeezy dashboard.

You can do so by visiting the following URL: [https://app.lemonsqueezy.com/settings/stores](https://app.lemonsqueezy.com/settings/stores).

The ID is the number next to the store name.

Proceed by copying the Store ID into the `LEMON_SQUEEZY_STORE_ID` environment variable.

## Step 4: Configure Lemon Squeezy plans in the Makerkit configuration

Now, we need to configure the Lemon Squeezy plans in the Makerkit configuration, and enter the correct plan IDs.

To do so, open the `~/configuration.ts` file and edit the `stripe` object.

We can convert this to something more abstract, such as `subscriptions`, and add the relevant Lemon Squeezy product and variant IDs.

 ```ts {% title="src/configuration.ts" %}
{
  subscriptions: {
    products: [
      {
        name: 'Basic',
        description: 'Description of your Basic plan',
        badge: `Up to 20 users`,
        productId: 1, // <-- Lemon Squeezy product ID
        features: [
          'Basic Reporting',
          'Up to 20 users',
          '1GB for each user',
          'Chat Support',
        ],
        plans: [
          {
            name: 'Monthly',
            price: '$9',
            variantId: 1, // <-- Lemon Squeezy variant ID
          },
          {
            name: 'Yearly',
            price: '$90',
            variantId: 2, // <-- Lemon Squeezy variant ID
          },
        ],
      }
    ]
  }
}
```

**Note**: The `variantId` is the ID of the plan in Lemon Squeezy. The `productId` is the ID of the product.

#### Find your Lemon Squeezy product and variant IDs

You can find your Lemon Squeezy Product and Variant IDs using the Lemon Squeezy dashboard.

### Step 4: Creating the Webhook In Lemon Squeezy

We need to create a Webhook from the Lemon Squeezy Dashboard so that we can receive the events to our webhook handler.

You will be prompted for the following:

- URL: point it to the ngrok URL (see below). Ensure you point it to the `/resources/ls/webhook` endpoint.
- Secret: the secret `LEMON_SQUEEZY_SIGNING_SECRET` that you have set in the `.env` file
- Events: select the events that you want to receive. In our case, we want to receive the `subscription_created` and `subscription_updated` events.

{% img src="/assets/images/posts/lemon-squeezy-webhook-setup.webp" width="1118" height="1666" /%}

#### Testing Locally

To test the webhook handler locally, we can use the [ngrok](https://ngrok.com/) tool.

We add the following command to the `package.json` file:

```
"ngrok": "npx ngrok http 3000"
```

And you can run it with:

```bash
npm run ngrok
```

You can also use other alternatives, such as Localtunnel or Cloudflare Tunnel.

### Step 5: Go to production

Once you're ready to go to production, you can switch your Lemon Squeezy store to production mode.

Additionally, remember to set up the webhook in production mode as well, pointing to your production URL.

