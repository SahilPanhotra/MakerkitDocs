-----------------
FILE PATH: kits/remix-supabase/getting-started/running-the-application.mdoc

---
status: "published"

title: How to run a Remix and Supabase MakerKit application
label: Running the application
order: 3
description: 'Learn how to run a Remix and Supabase MakerKit application'
---


After installing the modules, we can finally run the
application in development mode.

We need to execute two commands (and an optional one for Stripe):

1. **Remix Server**: the first command is for running the Remix server
2. **Supabase Environment**: the second command is for running the Supabase
environment with Docker
3. **Stripe CLI**: finally, the Stripe CLI is needed to dispatch webhooks to
our local server (optional, only needed when interacting with Stripe)

## About this Documentation

This documentation complements the Supabase one and is not meant to be a replacement. We recommend reading the Supabase documentation to get a better understanding of the Supabase concepts and how to use it.

## Install and Run Docker

Before we can run the Supabase local environment, we need to run Docker, as Supabase uses it for running its local environment.

You can use Docker Desktop, Colima, OrbStack, or any other Docker-compatible solution.

### Adding the environment variables file

The kit comes with a template of what your `.env` file should look like named `.env.template`.

**Before you continue**: rename `.env.template` to `.env`, or copy its contents to `.env`.

**This file won't be committed to git**. When you deploy your production app, ensure you add the environment variables using your CI/Service.

NB: Remix does not use the `.env` file when bundling the application in production mode.

### Running the Supabase Environment

First, let's run the Supabase environment, which will spin up a local
instance using Docker. We can do this by running the following command:

```bash
npm run supabase:start
```

Additionally, it imports the default seed data. We use it this data to
populate the database with some initial data and execute the E2E tests.

After running the command above, you will be able to access the Supabase
Studio UI at [http://localhost:54323/](http://localhost:54323/).

### Adding the Supabase Keys to the Environment Variables
If this is the first time you run this command, we will need to get the
Supabase keys and add them to our local environment variables configuration
file `.env`.

```bash
> supabase start

Applying migration 20221215192558_schema.sql...
Seeding data supabase/seed.sql...
Started supabase local development setup.

         API URL: http://localhost:54321
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
    Inbucket URL: http://localhost:54324
      JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
        anon key: ****************************************************
service_role key: ****************************************************
```

Now, we need to copy the `anon key` and `service_role key` values and add
them to the `.env` file:

```
SUPABASE_ANON_KEY=****************************************************
SUPABASE_SERVICE_ROLE_KEY=****************************************************
```

### Building the Contentlayer (Blog and Documentation) documents

The codebase expects the Contentlayer documents to be built before running the Remix server. To do so, run the following command:

```
npm run contentlayer:build
```

If you want to build the documents every time you save a file, you can run the following command:

```
npm run contentlayer:watch
```

Running the `contentlayer:watch` command is the recommended way to work with the Contentlayer during development.

{% alert type="warning" %}
You may see an error when running the `contentlayer:build` command. This is a known issue in Node >= 18. You can safely ignore it, as long as the ".contentlayer" directory is created.
{% /alert %}

### Running the Remix Server

And now, the Remix server:

```bash
npm run dev
```

If everything goes well, your server should be running at
[http://localhost:3000](http://localhost:3000).

### Running the Stripe CLI

Run the Stripe CLI with the following command:

```bash
npm run stripe:listen
```

#### Add the Stripe Webhooks Key to your environment file

If this is the first time you run this command, you will need to copy the Webhooks key printed on the console and add it to your development environment variables file:

 ```bash {% title=".env.development" %}
STRIPE_WEBHOOKS_SECRET=<PASTE_KEY_HERE>
```


#### Signing In for the first time

You should now be able to sign in. To quickly get started, use the following credentials:

```
email = test@makerkit.dev
password = testingpassword
```

#### Email Confirmations

When signing up, Supabase sends an email confirmation to a testing account. You can access the InBucket testing emails [using the following link](http://localhost:54324/monitor), and can follow the links to complete the sign up process.




-----------------
FILE PATH: kits/remix-supabase/getting-started/technical-details.mdoc

---
status: "published"
title: The technologies that make up a Remix Supabase application
label: Technical Details
order: 1
description: 'MakerKit uses Remix, Supabase, Tailwind CSS, Zod, React Query,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
---

MakerKit is built primarily with Remix, Supabase, and Tailwind CSS.

Under the hood, there are many more libraries and architectural
decisions you should know.

### 1) A Scalable Remix Application
The codebase is entirely built with [Remix](https://remix.run).

The front end is fully built with React, leveraging React Hooks
and the best practices available today.

### 2) Built on top of Supabase

[Supabase](https://supabase.com) is an open source Firebase alternative,
 a great choice for a backend for a SaaS application.

MakerKit uses Supabase for Authentication, Database (Postgres) for storing data and Storage for storing files.

This kit **does not use Prisma**. We will release a Prisma version in the future.

### 3) Theme built with Tailwind 3

The bulk of the application's theme, built with [Tailwind 3](https://tailwindcss.com/), is
stored in three CSS files rather than HTML.

It may sound like a bad practice, but we approached this way to help you find and edit
the theme's CSS in one single place.

We designed the theme in light and dark, so you can choose to use both according to your user's OS settings or manage it using a local cookie.

### 4) Radix UI UI Components

All the kits use [Radix UI](https://radix-ui.com), arguably the best headless UI library for React.

The components are styled using Tailwind CSS. Many components are inspired by the template made by [Shad CN](https://ui.shadcn.com/). We recommend taking a look if you want to build your own components.

Some components are still using Headless UI, but will be phased out in the future.

### 5) Fully and Strictly Typed with Typescript

We wrote every single line of code with *strictly-typed
Typescript* both on the client and server side.

### 6) Payload validation with Zod

The API endpoints are **validated and protected with [Zod](https://github.com/colinhacks/zod)**, which also helps with strong-typing.

### 7) E2E testing with Cypress

We ship MakerKit with [Cypress](https://www.cypress.io/), likely the most
popular E2E testing framework.

We made lots of examples and utilities to change the tests as you develop
your application.

### 8) Linted with EsLint and formatted with Prettier

We lint the codebase with **a very strict EsLint configuration**, but
we distribute it with less strict params so that it won't be in your way
as you're experimenting with the starter.

We will show you how to add stricter parameters if that's your thing, but we do not recommend jumping head-in with it. It's much better to activate the stricter configuration once you have already shipped the product and are looking to stabilize it.

Furthermore, the codebase is **formatted with Prettier** automatically.

### 9) Multi-language by default with remix-i18n

We configured the application to be translated by default thanks to the
`remix-i18n` package.

All the application's strings are translated by default, so you can easily
extend it to other languages if necessary.

Using translation files can help you if you want to **translate your apps in multiple
languages**, but it also allows you to find the text strings you
want to replace.

In this way, you do not need to change strings in the codebase, but they're
**neatly organized in JSON files**.

### 10) Logging with Pino

MakerKit uses [Pino](https://github.com/pinojs/pino), a lightweight logging library.

API errors are logged with just enough information to help you debug
your API routes.

### 11) React Query

MakerKit uses Tanstack's React Query to coordinate data fetching and caching.


-----------------
FILE PATH: kits/remix-supabase/getting-started.mdoc

---
status: "published"
title: "Getting Started in Remix Supabase"
label: "Getting Started"
order: 1
description: "Remix Supabase is a boilerplate for building SaaS applications with Remix and Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/going-to-production/going-to-production-overview.mdoc

---
status: "published"
title: Checklist to ship a MakerKit application to Production | Remix Supabase SaaS Kit
description: This checklist will help you ship your Remix Supabase SaaS Kit application to production
label: Production Checklist
order: 0
---

Before going to production, it's essential to follow certain precautions to avoid failures.

## 1) Environment Variables

First, we must safely inject the correct environment variables into their
relative environment. MakerKit has predefined templates that allow you to understand where each variable should be added.

Before publishing your SaaS, or if you are making a production release after
a change and this included an environment variable; ensure your CI provider is
up to date with the correct variable in the right environments.

Since the `.env` file will not be committed to your repository, you will
need to add the environment variables beforehand.

[Check out our Environment Variables documentation](environment-variables) for more information.

## 2) Supabase Postgres Row Level Security (RLS) Policies

If you are using Supabase, you will need to ensure that your RLS policies
are set up correctly, and that the changes are propagated to the production
database.

## 3) Sync your database schema with your remote Supabase database

If you are using Supabase, you will need to ensure that your database schema is in sync with your remote Supabase database.

You can do this using the Supabase CLI: for more information, please refer to the [Supabase CLI documentation](https://supabase.com/docs/reference/cli/supabase-db-push).

More specifically, you need to run the following command:

```bash
supabase db push
```

You may need to link your Supabase project to your local Supabase CLI instance first.

After running this command, navigate to your Supabase project and check that the database schema is in sync with your local database schema.

## 4) Configuring Authentication URLs in Supabase

We need to add the app's URL to the Supabase Auth settings. To do this, navigate to the "Auth" tab in your project's dashboard and then navigate to the "URL Configuration" tab.

From here:
1. **Site URL**: set up the "Site URL" field using your Vercel URL
2. **Redirect URLs**: set the "Redirect URLs" field using the following URL: `https://<your-vercel-url>/auth/callback`

This is fundamental, as it will allow Supabase to redirect users back to our application after they log in.

{% img src="/assets/courses/supabse-auth-settings.webp" width="1688" height="1132" /%}

## 5) Enable your Stripe account Billing Status

Remember to complete your Stripe account's information and enable billing and to not forget it in `Testing Mode`.

Additionally, you will need to add your Stripe API keys to the environment variables to process payments. Ensure you have added the correct keys to the correct environment and that have selected all the required events to your webhook.

Please follow the [Stripe configuration guide](/docs/next-supabase-lite/stripe-configuration) to configure your Stripe account.

If you use Lemon Squeezy - the same applies.

## 6) Add your SMPT service credentials

You will need to add your SMTP service credentials to the environment variables to send emails. This is required for sending emails to users when they are invited to an organization.

To set the credentials, add them to the environment variables in the correct environment.

Remember to do the same for your email templates in Supabase - as the limits are extremely low in the free plan.

Please follow the [email configuration guide](/docs/remix-supabase/emails) to learn more about how to configure your SMTP service.

## 7) Follow the Supabase guide

Follow the [Supabase guide](https://supabase.com/docs/guides/platform/going-into-prod) to ensure you have followed all the steps to deploy your application to production.

## 8) Enable Sentry (Optional)

With 4000 events for free per month, we recommend that you use [Sentry.io](https://sentry.io) for
ensuring you can catch, analyze and fix any runtime errors in your application that your users will encounter.

## 9) Set up a Logging Service (Optional, highly recommended)

We recommend that you use a logging service to log any events that occur in your application. This will allow you to debug any issues that may occur in your application.

While this is optional, we highly recommend that you use a logging service. Logging is how you can debug issues in your application, and it is essential to have a logging service in place.

Some logging services that we recommend are:

- [Baselime](https://baselime.io)
- [Logflare](https://logflare.app)
- [Axiom](https://axiom.co)

The above can be installed with a one-click install in the Vercel dashboard (if you are using Vercel).

-----------------
FILE PATH: kits/remix-supabase/going-to-production.mdoc

---
status: "published"
title: "Going to Production in Remix Supabase (legacy)"
label: "Going to Production"
order: 14
description: "Learn how to prepare your application for shipping to production"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/how-to/api/api-routes.mdoc

---
status: "published"

title: "Adding API Routes in your Remix application"
label: "Adding API Routes"
order: 0
description: "API Routes allow you to create server endpoints in your Remix App that you can call from your client or from external sources"
---


API routes are a way to create a custom API endpoint for your application. You can use API routes to create RESTful endpoints that return JSON data.

Adding an API route to a Makerkit application is no different than any other Remix project. Let's see how.

## Creating an API Route

To create an API route, will export an `action` function from your routes.

Here's an example of an API route that returns a JSON response as response to a `GET` request.

 ```ts {% title="app/routes/hello._index.ts" %}
import { json } from "@remix-run/node";

export async function loader() {
  return json({ text: 'Hello' });
}
```

This is kinda simple, right? Let's see how we can use this API route in our application.

### Protecting API Routes

It's very likely that you want to ensure only authenticated users can access your API routes. To do that, you can use the `requireSession` function.

In the example below, we create a `POST` API route that requires the user to be authenticated. If the user is not authenticated, the user will be redirected away. Additionally, the `requireSession` function will also ensure the level access if the user opted in to multi factor authentication.

 ```ts {% title="app/routes/route._index.ts" %}
import { LoaderFunctionArgs } from "@remix-run/node";

import getSupabaseServerClient from '~/core/supabase/server-client';
import requireSession from '~/lib/user/require-session.server';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const session = await requireSession(client);

  // user is authenticated, do something here
}
```


-----------------
FILE PATH: kits/remix-supabase/how-to/api/call-api-routes-from-client.mdoc

---
status: "published"
title: "Calling API Routes from the client-side using React Query | Remix Supabase SaaS Kit"
label: "Calling API Routes from the client"
order: 1
description: "Learn how to use React Query to call your API route handlers from React components"
---

Makerkit includes the library React Query, a data-fetching React utility library.

This library is useful for making asynchronous requests from your React components. For example:

1. When interacting with the Supabase Client (DB, Storage, Auth, etc.)
2. When making requests to API Routes

Why is it useful? React Query makes it simpler to fetch and mutate data, caching and revalidating. Let's see how we can use it to make requests to our endpoints.

### Should I use React Query or Remix's useFetcher/useSubmit?

No exact answer - but here is what I do:

1. For actions that need to trigger a refresh of the UI, I use Remix's `useSubmit/useRefresh` hook.
2. For pulling data from the server-side as a result of user interactions, I use React Query.

Ultimately, you should use what you feel more comfortable with as both approaches are valid.

## Using React Query for fetching data from an API Route Handler

Let's assume we have a very simple API Route at `routes/resources.data.ts`:

 ```tsx {% title="app/routes/resources.data.ts" %}
import { json } from "@remix-run/node";

export async function loader() {
  return json({
    hello: "world"
  });
}
```

As you may know, we can call the endpoint `/resources/data` using a GET request and will receive the JSON response `{ "hello": "world" }`.

Now, we want to call this from our Remix application using React Query, and use
it within a React component.

We proceed building a React Hook with React Query which is able to fetch the data from our endpoint `/api/data`:

```tsx
import { useQuery } from '@tanstack/react-query';

export function useFetchData() {
  const key = '/api/data';

  return useQuery([key], async () => {
    return fetch(key).then(res => res.json())
  });
}
```

Now, we can call this endpoint from any React component:

```tsx
function MyComponent() {
  const { data, loading, error } = useFetchData();

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error...</p>;
  }

  return <div>{data.hello}</div>
}
```

## Mutating data with React Query

When it comes to mutating data, we can use the `useMutation` utility, designed to make mutations simple.

As you may know, mutations will likely use the method `POST`, `PUT`, `PATCH`, or `DELETE`. In these cases, the middleware in the kit will require us to send a CSRF token for security reasons.

As such, we provide a handy wrapper around `fetch`.

Let's write a very simple API Route that makes a mutation: we add a new record in the `tasks` table, and return the ID back to the client.

```tsx
import { ActionFunctionArgs, json } from "@remix-run/node";

export async function action(args: ActionFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const session = await requireSession(client);
  const task = args.request.json();

  const { data } = await client.from('tasks').insert({
    name: task.name,
    done: false,
    user_id: session.user.id,
  })
  .select('id')
  .single();

  return json({ id: data.id });
}
```

Now, let's write a React Hook that calls this endpoint using React Query.

```tsx
import { useMutation } from '@tanstack/react-query';
import useFetch from '~/core/hooks/use-fetch';

interface Task {
  name: string;
}

function useInsertTask() {
  const fetcher = useFetch();
  const path = '/api/task';

  return useMutation(
    path, async (data: Task) => {
      return fetcher({
        path,
        body: data,
        method: 'POST'
      });
    }
  );
}
```

What does `useFetch` do? This hook automatically inserts the CSRF token in our fetch requests, so that you don't have to.

If you prefer not using it, you can manually add the CSRF token instead:

```tsx
import { useMutation } from '@tanstack/react-query';
import useCsrfToken from '~/core/hooks/use-csrf-token';

interface Task {
  name: string;
}

function useInsertTask() {
  const csrfToken = useCsrfToken();
  const path = '/api/task';

  return useMutation((data: Task) => {
      return fetch(path, {
        body: JSON.stringify(data),
        method: 'POST',
        headers: {
          'x-csrf-token': csrfToken
        }
    });
  });
}
```

We can now call the mutation from a React component:

```tsx
function TaskForm() {
  const insertTaskMutation = useInsertTask();

  const onSubmit: React.FormEventHandler<HTMLFormElement> = (event) => {
    const name = new FormData(event.currentTarget).get('name') ?? 'No Name';
    const task = { name };

    return insertTaskMutation.mutateAsync(task);
  };

  return (
    <form onSubmit={onSubmit}>
      <input type={'name'} required />

      <button disabled={insertTaskMutation.loading}>
        Submit
      </button>
    </form>
  );
}
```


-----------------
FILE PATH: kits/remix-supabase/how-to/app/adding-app-pages.mdoc

---
status: "published"

title: "Adding pages to the application of your Makerkit Remix Supabase project"
label: Adding Pages
order: 1
description: "Learn how to add new pages to the application of your Makerkit Remix Supabase project."
---


To add a new page to the application site of your Remix project, you can create a new page in the `app/routes` directory under the layout `_app`. This means your new pages will follow the convention `_app.<page>.tsx`.

For example, the tasks page of the application is located at `app/routes/_app.tasks._index.tsx`.

### What does the `_app` layout do?

By "application", we mean the private pages of your project behind the authentication wall, such as the dashboard, the settings page, etc.

This directory contains the pages that are **only accessible to authenticated users**.

### Layout and Sidebar

Pages created within the layout `_app` will inherit the layout of the directory and will be displayed together with the Sidebar menu.

### Example - Creating a new page

For example, let's create a `Tasks` application located at `/tasks`. We will then create a new page at `_app.tasks._index.tsx`.

 ```tsx {% title="app/routes/_app.tasks._index.tsx" %}
import type {
  LoaderFunctionArgs,
  MetaFunction,
} from '@remix-run/node';

import { useLoaderData } from '@remix-run/react';
import { RectangleStackIcon } from '@heroicons/react/24/outline';

import { Trans } from 'react-i18next';

import AppHeader from '~/components/AppHeader';
import AppContainer from '~/components/AppContainer';

export const meta: MetaFunction = () => {
  return [
    {
      title: 'Tasks',
    },
  ];
};

export async function loader(args: LoaderFunctionArgs) {
  await requireSession(
    getSupabaseServerClient(args.request),
  );
  // return tasks
}

function TasksPage() {
  const { tasks } = useLoaderData<typeof loader>();

  return (
    <>
      <AppHeader>
        <span className={'flex space-x-2'}>
          <RectangleStackIcon className="w-6" />

          <span>
            <Trans i18nKey={'common:tasksTabLabel'} />
          </span>
        </span>
      </AppHeader>

      <AppContainer>
        {/* ... */}
      </AppContainer>
    </>
  );
}

export default TasksPage;
```

### Remember: always validate the session in every page

The parallel loading done by Remix to optimize the loading of the application means we cannot rely on parent routes to validate the session. This means that we need to validate the session in every page.

If your page needs authentication, add a loader and protect it with `requireSession`:

```tsx
export async function loader(
  args: LoaderFunctionArgs
) {
  await requireSession(
    getSupabaseServerClient(args.request),
  );
}
```

### Layout

Layout-wise, we added the following components:

1. An `AppHeader` component that will be used to render the header of the page. This component is located at `app/components/AppHeader.tsx`.
2. An `AppContainer` component that will be used to render the content of the page. This component is located at `app/components/AppContainer.tsx`.

You can omit these components if you don't need them.


-----------------
FILE PATH: kits/remix-supabase/how-to/app/guarding-pages.mdoc

---
status: "published"
title: "Guarding Pages | Remix Supabase SaaS Kit"
label: "Guarding Pages"
order: 4
description: "Learn how to guard pages in your Remix Supabase application"
---

As a server-side rendered application, Remix Supabase will always render the page on the server before sending it to the client.

**We assume you are in the context of a specific organization, as it is the primary use case for the Starter Kit.**

In the following example, we are loading the user role within the organization **that is currently selected**.

```tsx
import { LoaderFunctionArgs } from "@remix-run/node";

async function loader(args: LoaderFunctionArgs) {
  const data = await loadAppData(args);
  const userRole = data.role;

  // if the user is not an owner, redirect them to the home page
  if (userRole !== MembershipRole.Owner) {
    return redirect('/');
  }

  // return props to the page
}

async function OnlyOwnersPage() {
  const data = await useLoaderData<typeof loader>();
  // render the page
}
```

As you can see above:

1. We are loading the data required to render the page
2. We are checking the user role within the organization
3. If the user is not an owner, we redirect them to the home page (but feel free to change this)

## Guarding Application Pages with Organization Subscriptions

In this example, we want to make sure the Organization is subscribed to a plan before allowing the user to access the page.

To do so, we check the `subscription` property of the `Organization` object and redirect the user to the dashboard if the subscription is `active` or `trialing`.

```tsx
import { LoaderFunctionArgs, redirect } from "@remix-run/node";

async function loader(args: LoaderFunctionArgs) {
  const data = await loadAppData(args);
  const status = data.organization.subscription?.data?.status;

  // if the subscription is not active, redirect the user to the dashboard
  if (!isSubscriptionActive(status)) {
    return redirect('/');
  }

  // return props to the page
}

async function OnlySubscribersPage() {
  const data = await useLoaderData<typeof loader>();
  // render the page
}

function isSubscriptionActive(status: string | undefined) {
  return ['trialing', 'active'].includes(status);
}
```

-----------------
FILE PATH: kits/remix-supabase/how-to/app/updating-sidebar-menu.mdoc

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

