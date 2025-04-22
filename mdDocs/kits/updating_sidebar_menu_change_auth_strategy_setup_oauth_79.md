-----------------
FILE PATH: kits\remix-supabase\how-to\app\updating-sidebar-menu.mdoc

---
status: "published"

title: "Updating the sidebar menu of your Makerkit SaaS application"
label: Updating the Sidebar menu
order: 1
description: "Learn how to update the sidebar menu of your Makerkit landing pages"
---


The Sidebar of your Makerkit SaaS application is defined in the `src/navigation.config.tsx` configuration file.

This allows you to easily add, remove, or update the menu entries of your application and automatically generate a responsive sidebar menu for your users.

### Adding a new menu entry to the Sidebar

To add a new menu entry to the sidebar, simply add a new entry to the `NAVIGATION_CONFIG.items` array in the `src/navigation.config.tsx` file.

A menu entry is defined by the following properties:
```tsx
{
  label: string;
  path: string;
  Icon: (props: { className: string }) => JSX.Element;
}
```

- `label`: The label of the menu entry. This label will be displayed in the sidebar of your application. This can be either a reference to a translation key or a string.
- `path`: The path of the menu entry. This path will be used to generate the `href` attribute of the menu entry.
- `Icon`: The icon of the menu entry. This icon will be displayed in the sidebar of your application.

Consider the default schema below:

```tsx
import configuration from '~/configuration';

import {
  Cog8ToothIcon,
  Square3Stack3DIcon,
  Squares2X2Icon,
} from '@heroicons/react/24/outline';

const NAVIGATION_CONFIG = {
  items: [
    {
      label: 'common:dashboardTabLabel',
      path: configuration.paths.appHome,
      Icon: ({ className }: { className: string }) => {
        return <Squares2X2Icon className={className} />;
      },
    },
    {
      label: 'common:tasksTabLabel',
      path: '/tasks',
      Icon: ({ className }: { className: string }) => {
        return <Square3Stack3DIcon className={className} />;
      },
    },
    {
      label: 'common:settingsTabLabel',
      path: '/settings',
      Icon: ({ className }: { className: string }) => {
        return <Cog8ToothIcon className={className} />;
      },
    },
  ],
};

export default NAVIGATION_CONFIG;
```

Now, let's add a new menu entry to the sidebar:

```tsx {32-37}
import configuration from '~/configuration';

import {
  Cog8ToothIcon,
  Square3Stack3DIcon,
  Squares2X2Icon,
} from '@heroicons/react/24/outline';

const NAVIGATION_CONFIG = {
  items: [
    {
      label: 'common:dashboardTabLabel',
      path: configuration.paths.appHome,
      Icon: ({ className }: { className: string }) => {
        return <Squares2X2Icon className={className} />;
      },
    },
    {
      label: 'common:tasksTabLabel',
      path: '/tasks',
      Icon: ({ className }: { className: string }) => {
        return <Square3Stack3DIcon className={className} />;
      },
    },
    {
      label: 'common:settingsTabLabel',
      path: '/settings',
      Icon: ({ className }: { className: string }) => {
        return <Cog8ToothIcon className={className} />;
      },
    },
    {
      label: 'common:myNewTabLabel',
      path: '/my-new-tab',
      Icon: ({ className }: { className: string }) => {
        return <Cog8ToothIcon className={className} />;
      },
    },
  ],
};
```

As you can see, we added a new menu entry to the `NAVIGATION_CONFIG.items` array. The change will be automatically reflected in the sidebar of your application.

-----------------
FILE PATH: kits\remix-supabase\how-to\authentication\change-auth-strategy.mdoc

---
status: "published"
title: "How to change Authentication strategy | Remix Supabase SaaS Kit"
label: Change Authentication strategy
order: 0
description: "Learn how to change the authentication strategy in Makerkit. Choose between email and password, phone number, email link, and OAuth."
---

Makerkit supports various authentication strategies supported by Supabase. We can customize these using the global configuration at `src/configurationt.ts`.

In the configuration file, you can find the following code:

```ts
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

// in your configuration JSON
auth: {
  // ensure this is the same as your Supabase project. By default - it's true
  requireEmailConfirmation:
    process.env.NEXT_PUBLIC_REQUIRE_EMAIL_CONFIRMATION === 'true',
  // NB: Enable the providers below in the Supabase Console
  // in your production project
  providers: {
    emailPassword: true,
    phoneNumber: false,
    emailLink: false,
    oAuth: ['google'] as Provider[]
  },
},
```

We will need to edit this file to change the authentication strategy.

### Enabling Email and Password Authentication

This is the default authentication strategy. It allows users to sign up and sign in using their email and password.

To enable this strategy, set `providers.emailPassword` to `true`.

```tsx
providers: {
  emailPassword: false,
  phoneNumber: true,
  emailLink: false,
  oAuth: ['google'] as Provider[],
},
```

### Enabling Phone Number Authentication

This strategy allows users to sign up and sign in using their phone number.

To enable this strategy, set `providers.phoneNumber` to `true`.

```tsx
providers: {
  emailPassword: false,
  phoneNumber: true,
  emailLink: false,
  oAuth: ['google'] as Provider[],
},
```

### Enabling Email Link Authentication

This strategy allows users to sign up and sign in using their email address. A link will be sent to their email address to verify their identity.

To enable this strategy, set `providers.emailLink` to `true`.

```tsx
providers: {
  emailPassword: false,
  phoneNumber: trfalseue,
  emailLink: true,
  oAuth: ['google'] as Provider[]
},
```

### Enabling OAuth Authentication

This strategy allows users to sign up and sign in using their social media accounts. You can enable multiple OAuth providers.

To enable this strategy, set `providers.oAuth` to an array of OAuth providers.

```tsx
providers: {
  emailPassword: false,
  phoneNumber: false,
  emailLink: false,
  oAuth: ['google', 'facebook'] as Provider[]
},
```

The interface `Provider` is very important as it tells Makerkit which OAuth provider to use. You can find the list of [supported OAuth providers here](https://supabase.io/docs/guides/auth#third-party-logins).

## Can I use multiple authentication strategies?

Yes, you can use multiple authentication strategies. For example, you can enable email and password authentication and phone number authentication at the same time.

```tsx
providers: {
  emailPassword: true,
  phoneNumber: true,
  emailLink: false,
  oAuth: ['google', 'facebook'] as Provider[]
},
```

With that said, the UI is not designed to support multiple authentication strategies. **You will need to customize the UI to support multiple authentication strategies**.


-----------------
FILE PATH: kits\remix-supabase\how-to\authentication\setup-oauth.mdoc

---
status: "published"

title: "How to setup oAuth in the Remix Supabase Starter"
label: Setup oAuth
order: 1
description: "Learn how to setup oAuth in the Remix Supabase Starter"
---


For oAuth - you can enable multiple providers. For example, you can enable Google and Facebook authentication at the same time by adding the providers to the `auth.providers.oAuth` array in the configuration file at `src/configuration.ts`.

You must configure the oAuth providers in three places:
1. **Supabase** - you need to setup an oAuth application in Supabase.
2. **Provider** - you need to setup an oAuth application in the provider (for example, Google Auth)
3. **Makerkit Configuration** - you need to add the provider to the `auth.providers.oAuth` array in the configuration file at `src/configuration.ts`.

Below is an example of how to enable Google and Facebook authentication:

 ```tsx {% title="src/configuration.ts" %} {5}
providers: {
  emailPassword: false,
  phoneNumber: false,
  emailLink: false,
  oAuth: ['google', 'facebook'] as Provider[]
},
```

The interface `Provider` is very important as it tells Makerkit which OAuth provider to use. You can find the list of [supported OAuth providers here](https://supabase.io/docs/guides/auth#third-party-logins).

By default, Makerkit provides logos for the most famous oAuth providers, but not all of them. To add a logo for an oAuth provider, please update the `src/core/ui/AuthProviderLogo.tsx` file.

You will provide a string or a React node for the logo by specifying the provider name as the key of the object.

 ```tsx {% title="src/core/ui/AuthProviderLogo.tsx" %}
function getOAuthProviderLogos(): Record<string, string | React.ReactNode> {
  return {
    password: <AtSymbolIcon className={'h-[22px] w-[22px]'} />,
    phone: <DevicePhoneMobileIcon className={'h-[22px] w-[22px]'} />,
    google: '/assets/images/google.webp',
    facebook: '/assets/images/facebook.webp',
    twitter: '/assets/images/twitter.webp',
    github: '/assets/images/github.webp',
    microsoft: '/assets/images/microsoft.webp',
    apple: '/assets/images/apple.webp',
  };
}
```

### How to Setup an oAuth Provider

Most of the setup is done in two places:

1. **Supabase** - you need to setup an oAuth application in Supabase.
2. **Provider** - you need to setup an oAuth application in the provider (for example, Google Auth)

To know more about how to setup an oAuth provider, please read the [Supabase documentation](https://supabase.io/docs/guides/auth#third-party-logins).

-----------------
FILE PATH: kits\remix-supabase\how-to\contextual-data\organization.mdoc

---
status: "published"

title: "Fetching the selected Organization"
label:  "Fetching the selected Organization"
order: 1
description: "Learn how to fetch the selected organization from the backend and frontend."
---


There are several ways to fetch the current selected organization, depending on the use-case.

### Using "parseOrganizationIdCookie" to get the Organization ID

You can use the `parseOrganizationIdCookie` function in:

- Remix Loaders
- Remix Actions

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSupabaseServerClient from '~/core/supabase/server-client';
import requireSession from '~/lib/user/require-session.server';

import {
  parseOrganizationIdCookie,
} from '~/lib/server/cookies/organization.cookie';

async function getOrganizationId(request: Request) {
  const client = getSupabaseServerClient(request);
  const session = await requireSession(client);

  return parseOrganizationIdCookie(request, session.user.id);
}

export async function loader(args: LoaderFunctionArgs) {
  const organizationId = await getOrganizationId(args.request);

  return json({ organizationId });
}
```

This can be nullish, so remember to check for that.

## Fetching the selected Organization from the backend

To do so, use the function `getCurrentOrganization`, which also takes care of redirecting the user to the login page if they are not signed in, and checking the correct MFA access level if the user opted in.

You can use this function in:
- API Routes
- Server Actions
- Server Components

This function requires the current user ID and the (optionally) organization UID.

1. Get the current user ID from the session: we use the `requireSession` helper to do so.
2. Get the organization: we use the `getCurrentOrganization` helper to do so using the user ID and the organization UID.

```tsx
import { ActionFunctionArgs } from '@remix-run/node';

import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';
import getSupabaseServerClient from '~/core/supabase/server-client';
import requireSession from '~/lib/user/require-session.server';

import {
  parseOrganizationIdCookie,
} from '~/lib/server/cookies/organization.cookie';

export async function action({ request }: ActionFunctionArgs) {
  const client = getSupabaseServerClient();
  const session = await requireSession(client);
  const userId = session.user.id;
  const organizationId = await parseOrganizationIdCookie(request, userId);

  const organizationResponse = await getCurrentOrganization(client, {
    organizationId,
    userId,
  });
}
```

## Fetching the selected Organization from the frontend

Within the `_app` layout, the layout loader passes the current organization is passed from the backend to the frontend using the Context API. Therefore, we can require it using the `useCurrentOrganization` hook.

### Getting the Organization with "useCurrentOrganization"

To retrieve the signed in user from the frontend, you can use the `useUserSession` hook:

```tsx
import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';

const organization = useCurrentOrganization();
```

This is a React hook and can only be used inside a React component.

### Getting if the Organization subscription status is set to "active" on the frontend

You can use the `useIsSubscriptionActive` hook to get the subscription status of the current organization. You may want to use this function to gate access to certain parts of the app.

```tsx
import useCurrentOrganization from '~/lib/organizations/hooks/use-is-subscription-active';

const isActive = useIsSubscriptionActive();
```

This is a React hook and can only be used inside a React component.




-----------------
FILE PATH: kits\remix-supabase\how-to\contextual-data\user.mdoc

---
status: "published"

title: "Fetching the signed in User"
label:  "Fetching the signed in User"
order: 0
description: "Learn how to fetch the signed in user from the backend and frontend."
---


There are several ways to fetch the signed in user, depending on the use-case.

## Fetching the signed in user from the backend

To do so, use the function `requireSession`, which also takes care of redirecting the user to the login page if they are not signed in, and checking the correct MFA access level if the user opted in.

You can use this function in:
- API Routes
- Server Actions
- Server Components

### Using "requireSession" in API Routes

When using API routes, you can retrieve the signed in user using the `requireSession` function.

```tsx
import getSupabaseServerClient from '~/core/supabase/server-client';
import requireSession from '~/lib/user/require-session.server';

export async function POST() {
  const client = getSupabaseServerClient();
  const session = await requireSession(client);
}
```

### Using "requireSession" in Loaders

When using Remix Loaders, you can retrieve the signed in user using the `requireSession` function.

```tsx
import { LaaderFunctionArgs } from '@remix-run/node';

import getSupabaseServerClient from '~/core/supabase/server-client';
import requireSession from '~/lib/user/require-session.server';

export async function loader(args: LaaderFunctionArgs) {
  const client = getSupabaseServerClient();
  const session = await requireSession(client);

  // use session here
}
```

### Using "requireSession" Actions

You can use the `requireSession` function in Remix Actions to retrieve the signed in user.

```tsx
import { ActionFunctionArgs } from '@remix-run/node';

export async function action(args: ActionFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const session = await requireSession(client);

  // use session here
}
```

## Fetching the signed in user from the frontend

When in the scope of the `_app` layout, the app automatically injects the user session in the context layout.

This includes the user ID, email, and other authentication data, as well as the user's Supabase DB record.

### Getting the user Session with "useUserSession"

To retrieve the signed in user from the frontend, you can use the `useUserSession` hook:

```tsx
import useUserSession from '~/core/hooks/use-user-session';

const userSession = useUserSession();
```

This is a React hook and can only be used inside a React component.

### Getting the user ID with "useUserId"

To retrieve the signed in user ID from the frontend, you can use the `useUserId` hook:

```tsx
import useUserId from '~/core/hooks/use-user-id';

const userId = useUserId();
```

This is a React hook and can only be used inside a React component.

### Getting the user role within the organization with "useUserRole"

To retrieve the signed in user role of the current organization from the frontend, you can use the `useUserRole` hook:

```tsx
import useCurrentUserRole from '~/core/hooks/use-current-user-role';

const role = useCurrentUserRole();
```

This is a React hook and can only be used inside a React component.


-----------------
FILE PATH: kits\remix-supabase\how-to\data-fetching\reading-record.mdoc

---
status: "published"

title: "Reading Records from Postgres | Remix Supabase Kit"
label: "Reading Records from Postgres"
order: 0
description: "Learn how to read records from Supabase using your Remix Supabase
application"
---


In Makerkit, we define all the queries in their own file (depending on their feature) named `queries.ts`.

Assuming we have a table named `tasks` with the following schema:

```sql
create table tasks (
  id serial primary key,
  name text,
  organization_id integer references organizations not null,
  due_date timestamp,
  description text,
  done boolean default false
);
```

We can define a query to read a single task from the table:

```tsx
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '~/database.types';

export function getTask(client: SupabaseClient<Database>, id: number) {
  return client
    .from('tasks')
    .select(
      `
      id,
      name,
      organizationId: organization_id,
      dueDate: due_date,
      description,
      done
    `,
    )
    .eq('id', id)
    .single();
}
```

The `single` modifier will return a single record from the database. If no record is found, it will throw an error. If you want to return `null` instead, you can use the `maybeSingle` modifier.

## Usage

You can now use the function above anywhere in the following layers:

1. Loader functions
2. Action Functions
3. React Client Components

In fact, we can import the Supabase client both on the browser and on the client.

### Backend

Assuming we want to read a task from a Server component, such as the Task page, we can do the following:

```tsx
import { LoaderFunctionArgs, json, redirect } from '@remix-run/node';

const TaskPage = () => {
  const { task } = useLoaderData<typeof laoder>();

  return (
    <>
      <h1>{task.name}</h1>
      <p>{task.description}</p>
    </>
  );
};

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const taskId = args.params.taskId;
  const { data: task } = await getTask(client, Number(taskId));

  if (!task) {
    return redirect('/dashboard');
  }

  return json({
    task,
  });
}

export default TaskPage;
```

### Frontend

You can also choose to read the task from the frontend. This is useful if you want to use the data in a React component.

To do so, we use the **Supabase Web SDK client** using the hook `useSupabase` and combine it with React Query to fetch the data.

This is useful because you can fetch data directly from the client, which is extremely useful in scenarios such as loading data on-demand resulting from user interactions (click a table row, opening a modal, etc.)

```tsx
import useQuery from "@tanstack/react-query";

function useFetchTask({ id }: { id: number }) {
  const supabase = useSupabase();
  const key = ['tasks', id];

  return useQuery([key], async () => {
    return getTask(supabase, id);
  });
}
```

We can then use the hook in a React component:

```tsx
function TaskComponent({ id }: { id: number }) {
  const { data: task, loading, error } = useFetchTask({ id });

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <>
      <h1>{task.name}</h1>
      <p>{task.description}</p>
    </>
  );
}
```

-----------------
FILE PATH: kits\remix-supabase\how-to\database\generate-database-types.mdoc

---
status: "published"

title: "How to generate your Supabase Database types"
label: "Generating Database Types"
order: 2
description: "Learn how to generate your Supabase Database types using the typegen command"
---


When you change your Database schema, you need to also remember to update the Typescript types that the Supabase Client can use to strong-type your queries and mutations.

Following a change in the schema, you need to do two things:

1. **Reset the Database**: if you have changed the schema manually, e.g. updating the SQL code, you need to either reset or restart your Supabase instance. You can do so using the command `npm run supabase:db:reset`
2. **Generate the new type definitions**: now, you can run the command `npm run typegen` to generate the types. The generated file will be available at `src/database.types.ts`

That's it! Always remember to update your schema types after changing your database schema.

-----------------
FILE PATH: kits\remix-supabase\how-to\database\resetting-local-database.mdoc

---
status: "published"
title: "How to reset your local Database | Remix Supabase SaaS Kit"
label: "Resetting the local DB"
order: 1
description: "Learn how to reset the seed data in your Remix Supabase Database"
---

Resetting the DB can be useful when updating the schema of your database using the SQL schema files instead of the Supabase Studio UI.

To reflect the new changes, and/or resetting the database to the local seed, run the following command:

```
npm run supabase:db:reset
```

The database is now in the same state as when you started your Supabase server.

**Important**: users created during the session will be reset, which means your application may be in a conflicting state. Log out or delete the cookies to restart from scratch.

-----------------
FILE PATH: kits\remix-supabase\how-to\database\seed-data.mdoc

---
status: "published"

title: "How to seed data in your Supabase application"
label: "Seeding Local Data"
order: 0
description: "Learn how to seed data in your Supabase Database"
---


The Makerkit repository adds some seed data by default for running the E2E tests.

To update the seed data, you will update the SQL file `supabase/seed.sql`. This file contains the SQL code that seeds the database everytime you start the Supabase Server using `npm run supabase:start`.

### Adding data to the seed file

If you'd like to extend this data with your own, update the data in your local environment, either by using your application or using the Supabase Studio interface.

Then, you can run the `diff` command:

```
supabase diff
```

This command will output the SQL code with the difference between the seed data and the current local DB data. You can copy and paste the outputted data into your `seed.sql` file.

The next database restarts will use the new data inserted.


-----------------
FILE PATH: kits\remix-supabase\how-to\introduction.mdoc

---
status: "published"

title: "Introduction to the How to section of the Makerkit Remix Supabase documentation"
label: Introduction
order: 0
description: "Here is a brief introduction to the How to section of the Makerkit Remix Supabase documentation."
---


The "How to" documentation of Makerkit Remix Supabase is a collection of tutorials that will help you to build your own web application using Remix and Supabase using small and practical examples.

## What you will learn


1. Setting up your Makerkit Remix Supabase project
2. Authentication
3. Updating the marketing pages of your SaaS
4. Interacting with the Database
5. Updating the Marketing pages of your SaaS
6. Updating the Internal pages of your SaaS
7. Setting up Payments
8. Updating Translations
9. ... and a lot more.

This documentation is under development and will be updated regularly. If you have any questions, please contact me.

-----------------
FILE PATH: kits\remix-supabase\how-to\payments\configuring-plans.mdoc

---
status: "published"

title: "Configuring your SaaS Stripe Plans in your Makerkit Remix Supabase App"
label: Configuring Plans
order: 0
description: "Learn how to configure the Stripe plans in your SaaS application with Makerkit"
---


When taking payments from Stripe, you're required to configure your subscription's plans.

Your plans need to be configured in two places:

1. **Stripe** - first, you need to define your **products** and **plans** in Stripe itself. This requires you to have an account in Stripe.
2. **Configuration**: - then, you will copy your plans data in the Makerkit configuration, which is used to render the Pricing table and display the prices metadata

### Pricing Table Configuration

Below is the default Pricing Table configuration of the kits. The `PricingTable` component will automatically generate the pricing table based on the Stripe plans you have created and added to the configuration (see below).

We have 3 products (Basic, Pro and Premium), each with 2 plans (monthly, yearly). By default, the Pro plan is set as the recommended plan.

Of course, the below is simply an example. You will need to customize this according to your application's plans.

```tsx
stripe: {
  products: [
    {
      name: 'Basic',
      description: 'Description of your Basic plan',
      badge: `Up to 20 users`,
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
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$90',
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
        },
      ],
    },
    {
      name: 'Pro',
      badge: `Most Popular`,
      recommended: true,
      description: 'Description of your Pro plan',
      features: [
        'Advanced Reporting',
        'Up to 50 users',
        '5GB for each user',
        'Chat and Phone Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$29',
          stripePriceId: 'pro-plan-mth',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$200',
          stripePriceId: 'pro-plan-yr'
        },
      ],
    },
    {
      name: 'Premium',
      description: 'Description of your Premium plan',
      badge: ``,
      features: [
        'Advanced Reporting',
        'Unlimited users',
        '50GB for each user',
        'Account Manager',
      ],
      plans: [
        {
          name: '',
          price: 'Contact us',
          stripePriceId: '',
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ],
}
```

#### Update the Stripe Price IDs values!

Please replace the property `stripePriceId` with the real price ID from the
Stripe Dashboard.

<div>
  {% alert type="warning" title="Update your real Stripe Price IDs" %}
      The properties "stripePriceId" are placeholders. You will need to replace them with the IDs of your plans.

{% /alert %}
</div>

### Additional Configuration

You can also configure the following properties on each `product`:

- `recommended` - set to `true` if you want to highlight a plan as recommended
- `badge` - set to a string if you want to display a badge next to the plan name

You can also configure the following properties on each `plan`:

- `price` - any string that you want to display as the price (e.g. `$9.99` or `Free`)
- `label` - set to a custom string if you want to display a button label
- `href` - set to a custom URL if you want to link to a custom page (for example, a contact page)
- `trialPeriodDays` - set to the number of days you want to offer a free trial for


-----------------
FILE PATH: kits\remix-supabase\how-to\payments\stripe-webhooks-locally.mdoc

---
status: "published"

title: "Running the Stripe Webhook locally"
label: Running the Stripe Webhook locally
order: 1
description: "Want to test and run the Stripe Webhook locally? This guide will show you how to do it."
---


When testing webhooks sent by Stripe, it's useful to redirect them to our local webhook endpoint so that we can test them locally.

Makerkit has a built-in NPM script that starts the Stripe CLI and redirects webhooks to our local endpoint.

## Prerequisites

The only prerequisite is to have Docker installed and running on your machine.

## Running the Stripe Webhook locally

To run the Stripe Webhook locally, run the following command:

```bash
npm run stripe:listen
```

The first time you run this command, it will ask you to login to your Stripe account. Follow the instructions on the screen to do so.

Once you're logged in, run the command again:

```bash
npm run stripe:listen
```

This time, the Stripe CLI will start and redirect webhooks to our local endpoint.

You can now test your webhooks locally when testing your Stripe integration with Makerkit 🎉


-----------------
FILE PATH: kits\remix-supabase\how-to\payments\using-lemon-squeezy.mdoc

---
status: "published"
title: "Using Lemon Squeezy instead of Stripe in your Remix Supabase application"
label: Using Lemon Squeezy instead of Stripe
order: 4
description: "How to use Lemon Squeezy instead of Stripe in your Remix Supabase application"
---

To use Lemon Squeezy instead of Stripe, you should use the branch `main-ls` to develop your SaaS.

The Lemon Squeezy configuration is slightly different from the Stripe configuration as it accepts a variant ID instead of a price ID.

```tsx
subscriptions: {
  plans: [
    {
      name: 'Basic',
      description: 'Description of your Basic plan',
      badge: `Up to 20 users`,
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
          variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$90',
           variantID: '<your variant ID>',
        },
      ],
    },
    {
      name: 'Pro',
      badge: `Most Popular`,
      recommended: true,
      description: 'Description of your Pro plan',
      features: [
        'Advanced Reporting',
        'Up to 50 users',
        '5GB for each user',
        'Chat and Phone Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$29',
           variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$200',
          variantID: '<your variant ID>',
        },
      ],
    },
    {
      name: 'Premium',
      description: 'Description of your Premium plan',
      badge: ``,
      features: [
        'Advanced Reporting',
        'Unlimited users',
        '50GB for each user',
        'Account Manager',
      ],
      plans: [
        {
          name: '',
          price: 'Contact us',
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ]
}
```

Please refer to the [complete Lemon Squeezy documentation](/docs/next-supabase/lemon-squeezy) for more details.

