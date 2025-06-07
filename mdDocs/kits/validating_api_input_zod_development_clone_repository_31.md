-----------------
FILE PATH: kits/next-supabase-lite/development/validating-api-input-zod.mdoc

---
status: "published"
title: "Validating the Next.js API inputs with Zod and Typescript | Next.js Supabase Lite SaaS Kit"
label: Validating API payload with Zod
order: 7
description: "Zod is a library for validating data with awesome support for
Typescript. Learn how to use it within your Next.js Supabase Lite SaaS Kit project."
---

[Zod](https://github.com/colinhacks/zod) is a Typescript library that helps us
secure our API endpoints by validating the payloads sent
from the client and also facilitating the typing of the payloads with
Typescript.

Using Zod is the first line of defense to validate the data sent against our
API: as a result, it's something we recommend you keep doing. It ensures we
write safe, resilient, and valid code.

All Makerkit's API routes are secured with Zod: in this document, we want
to explain the conventions used by the SaaS Boilerplate, and how to use it
for your API endpoints.

When we write an API endpoint or a Server Action, we first define the schema of the payload:

```tsx
function getBodySchema() {
  return z.object({
    displayName: z.string(),
    email: z.string().email(),
  });
}
```

This function represents the schema, which will validate the following
interface:

```tsx
interface Body {
  displayName: string;
  email: Email;
}
```

Now, let's write the body of a Server Action that validates the body of the
function, which we expect to be equal to the `Body` interface.

```tsx
'use server';

export async function serverAction(
  data: z.infer<
    ReturnType<typeof getBodySchema>
  >
) {
  // we can safely use data with the interface Body
  const bodyResult = await getBodySchema().parseAsync(data);
  const { displayName, email } = bodyResult;

  return sendInvite({ displayName, email });
}

function getBodySchema() {
  return z.object({
    displayName: z.string(),
    email: z.string().email(),
  });
}
```

You can also use `safeParse` if you prefer not to throw an error when the
validation fails:

```tsx
'use server';

export async function serverAction(
  data: z.infer<
    ReturnType<typeof getBodySchema>
  >
) {
  // we can safely use data with the interface Body
  const bodyResult = await getBodySchema().safeParseAsync(data);

  if (bodyResult.success === false) {
    return bodyResult.error;
  }

  const { displayName, email } = bodyResult.data;

  return sendInvite({ displayName, email });
}

function getBodySchema() {
  return z.object({
    displayName: z.string(),
    email: z.string().email(),
  });
}
```

The same applies to Route Handlers:

```tsx
import { NextRequest } from "next/server";

export async function POST(
  request: NextRequest
) {
  const body = await request.json();
  const bodyResult = await getBodySchema().safeParseAsync(body);

  if (bodyResult.success === false) {
    return bodyResult.error;
  }

  const { displayName, email } = bodyResult.data;

  return sendInvite({ displayName, email });
}

function getBodySchema() {
  return z.object({
    displayName: z.string(),
    email: z.string().email(),
  });
}
```

To learn more about validating data with Zod, we suggest you
check out [the Zod official documentation on GitHub](https://github.com/colinhacks/zod).


-----------------
FILE PATH: kits/next-supabase-lite/development.mdoc

---
status: "published"
title: "Development in Next.js Lite Supabase"
label: "Development"
order: 6
description: "This section will teach you everything you need to know about developing with Next.js Lite Supabase."
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/getting-started/clone-repository.mdoc

---
status: "published"
title: Clone the Next.js Supabase Lite SaaS boilerplate Repository
label: Clone the repository
order: 2
description: 'Learn how to clone the MakerKit repository'
canonical: '/docs/next-supabase/clone-repository'
---

If you have bought a license for MakerKit, you have access to all the
repositories built by the MakerKit team. In this document, we will learn how
to fetch and install the codebase.

### Requirements

To get started with the Next.js and Supabase SaaS template, we need to ensure
you install the required software.

- Node.js
- Git
- Docker

### Getting Started with MakerKit

#### Clone the repository

To get the codebase on your local machine using the original repository, clone the repository with the
following command:

```
git clone git@github.com:makerkit/next-supabase-saas-kit-lite.git my-saas
```

The command above clones the repository in the folder `my-saas` which
you can rename it with the name of your project.

#### Initializing Git

Now, run the following commands for:

1. Moving into the folder
2. Add the remote repository

```
cd my-saas
git remote rm origin
git remote add origin <your-git-repository>
```

If you haven't created a Git repository yet, you can do it later on.

### Setting the Upstream repository, and fetching updates

Now, we can add the original Makerkit repository as "upstream" so we can fetch updates from the main repository:

```
git remote add upstream git@github.com:makerkit/next-supabase-saas-kit-lite.git
```

To fetch updates, you can run the following command:

```
git pull upstream main
```

Sometimes, you'll likely run into conflicts when running this command, so carefully choose the changes (sorry!).

If you want to use the Lemon Squeezy branch, you'll need to switch to the `main-ls` branch:

```
git checkout main-ls
```

Of course, when pulling updates, you'll need to pull from the `main-ls` branch:

```
git pull upstream main-ls
```

### Installing the Node dependencies

Finally, we can install the NodeJS dependencies with `npm`:

```
npm i
```

While the application code is fully working, we now need to set up your Supabase
project.

So let's jump on to the next step!


-----------------
FILE PATH: kits/next-supabase-lite/getting-started/introduction.mdoc

---
status: "published"

title: 'Introduction to MakerKit: a SaaS boilerplate for Next.js and Supabase (Lite)'
label: 'Introduction'
order: 0
description: 'Introduction to MakerKit: a SaaS starter built with Next.js and Supabase (Lite)'
canonical: '/docs/next-supabase/introduction'
---


MakerKit is **the Next.js Supabase boilerplate project for SaaS** built to get you
started on the right foot.

The MakerKit kit is a fully-featured Next.js application that uses Supabase for
authentication, database (with Postgres), and storage.

This version of the kit uses the new App Router introduced with Next.js 13, which is built on top of the new React Server Components.

## What does the boilerplate provide?

### Tech Stack

We built MakerKit with some of the best technologies available today, such as
Next.js, Tailwind CSS, and Supabase:

- **Scalable Next.js RSC (app directory)** structure template, perfect for the most ambitious
projects
- Beautiful **Tailwind 3** CSS theme - with dark mode!
- **Full Supabase** setup, including Authentication, Database and Storage
- **Reusable UI components** as building blocks (which you can easily swap
with your own or your favorite library) based on [Radix UI](https://radix-ui.com)
- **SSR-compatible Authentication flow**, including 3rd-party providers (Google,
Facebook, Twitter, GitHub, etc.)

### Template Features

MakerKit is fully functioning from the beginning and comes with the
following features:

- **User Authentication** with Supabase Auth
- **Payments** with Stripe, and support for **subscriptions**
- **MDX-powered Blog and Documentation** generators - already **SEO-optimized** for you [Coming Soon]

### Marketing

Marketing matters! But don't get hung up on the technicalities. MakerKit provides:

- **Newsletter Sign-up form** for *ConvertKit*, but easy to adapt to more
providers
- **Email Templates** you can write with React using [React.email](https://react.email)

### Debugging and Testing

Never waste time chasing bugs again.

Assuming you have created a Sentry account, MakerKit can catch exceptions (and possible bugs) and  report them to Sentry whenever they happen in both client and server-side code:

- **Error Tracking** set up with Sentry
- Comprehensive **E2E Tests** suite with Cypress, which you can use and learn
from

### After you buy

You're not going to be left alone. We offer help and support:

- Access to the team for feedback, requests, and general support. **We're here to support you build your SaaS**.

This documentation introduces each of the points above and guides you through
so that you can easily adjust MakerKit to your application's domain.

MakerKit has plenty of resources for using both Next.js and Supabase - and if you're stuck, you can always reach out to me for help.


-----------------
FILE PATH: kits/next-supabase-lite/getting-started/running-the-application.mdoc

---
status: "published"
title: How to run a Next.js and Supabase Lite MakerKit application
label: Running the application
order: 3
description: 'Learn how to run a Next.js and Supabase Lite MakerKit application and get started developing'
canonical: '/docs/next-supabase/running-the-application'
---

After installing the modules, we can finally run the
application in development mode.

We need to execute two commands (and an optional one for Stripe):

1. **Next.js Server**: the first command is for running the Next.js server
2. **Supabase Environment**: the second command is for running the Supabase
environment with Docker
3. **Stripe CLI**: finally, the Stripe CLI is needed to dispatch webhooks to
our local server (optional, only needed when interacting with Stripe)

## About this Documentation

This documentation complements the Supabase one and is not meant to be a replacement. We recommend reading the Supabase documentation to get a better understanding of the Supabase concepts and how to use it.

## Install and Run Docker

Before we can run the Supabase local environment, we need to run Docker, as Supabase uses it for running its local environment.

You can use Docker Desktop, Colima, OrbStack, or any other Docker-compatible solution.

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

When running the command, we will see a message like this:

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
NEXT_PUBLIC_SUPABASE_ANON_KEY=****************************************************
SUPABASE_SERVICE_ROLE_KEY=****************************************************
```

### Running the Next.js Server

And now, the Next.js server:

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
FILE PATH: kits/next-supabase-lite/getting-started/technical-details.mdoc

---
status: "published"

title: The technologies that make up a MakerKit application
label: Technical Details
order: 1
description: 'MakerKit uses Next.js, Supabase, Tailwind CSS, Zod, React Query,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
canonical: '/docs/next-supabase/technical-details'
---


MakerKit is built primarily with Next.js, Supabase, and Tailwind CSS.

Under the hood, there are many more libraries and architectural
decisions you should know.

### 1) A Scalable Next.js Application
The codebase is entirely built with [Next.js](https://nextjs.org) using the experimental app directory and React Server Components.

The front end is fully built with React, leveraging React Hooks
and the best practices available today.

### 2) Built on top of Supabase

[Supabase](https://supabase.com) is an open source Firebase alternative,
 a great choice for a backend for a SaaS application.

MakerKit uses Supabase for Authentication, Database (Postgres) for storing data and Storage for storing files.

This kit **does not use Prisma**. We use Supabase's own client library.

### 3) Theme built with Tailwind CSS 3

The bulk of the application's theme, built with [Tailwind CSS 3](https://tailwindcss.com/), is
stored in three CSS files rather than HTML.

It may sound like a bad practice, but we approached this way to help you find and edit
the theme's CSS in one single place.

We designed the theme in light and dark, so you can choose to use both according to your user's OS settings or manage it using a local cookie.

### 4) Radix UI Components

All the kits use [Radix UI](https://radix-ui.com), arguably the best headless UI library for React.

The components are styled using Tailwind CSS. Many components are inspired by the template made by [Shad CN](https://ui.shadcn.com/). We recommend taking a look if you want to build your own components.

Some components are still using Headless UI, but will be phased out in the future.

### 5) Fully and Strictly Typed with Typescript

We wrote every single line of code with *strictly-typed Typescript* both on the client and server side.

### 6) Payload validation with Zod

The API endpoints are **validated and protected with [Zod](https://github.com/colinhacks/zod)**, which also helps with strong-typing.

### 7) E2E testing with Cypress

We ship MakerKit with [Cypress](https://www.cypress.io/), likely the most
popular E2E testing framework.

We made lots of examples and utilities to change the tests as you develop
your application.

### 8) Linted with EsLint and formatted with Prettier

We lint the codebase with **a very strict EsLint configuration**, but we distribute it with less strict params so that it won't be in your way as you're experimenting with the starter.

We will show you how to add stricter parameters if that's your thing, but we do not recommend jumping head-in with it. It's much better to activate the stricter configuration once you have already shipped the product and are looking to stabilize it.

Furthermore, the codebase is **formatted with Prettier** automatically.

### 9) Logging with Pino

MakerKit uses [Pino](https://github.com/pinojs/pino), a lightweight logging library.

API errors are logged with just enough information to help you debug
your API routes.

### 10) SWR for data fetching and mutations

MakerKit uses Vercel's SWR for data fetching and mutations when used from the client-side, which is in very few places, as we mostly use Server Actions.

-----------------
FILE PATH: kits/next-supabase-lite/getting-started.mdoc

---
status: "published"
title: "Getting Started in Next.js Lite Supabase"
label: "Getting Started"
order: 1
description: "Next.js Lite Supabase is a boilerplate for building SaaS applications with Next.js Lite and Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/going-to-production/going-to-production-overview.mdoc

---
status: "published"
title: Checklist to ship a MakerKit application to Production | Next.js Supabase Lite SaaS Kit
description: This checklist will help you ship your Next.js Supabase Lite SaaS application to production
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

Please follow the [email configuration guide](/docs/next-supabase-lite/emails) to learn more about how to configure your SMTP service.

## 7) Configure the Image Remote Patterns in your Next.js configuration

You will need to configure the image remote patterns in your Next.js configuration to allow the images to be served from the correct domain. This needs to match the domain of your Supabase Storage bucket domain - and other eventual domains you may be using to serve your images.

## 8) Follow the Supabase guide

Follow the [Supabase guide](https://supabase.com/docs/guides/platform/going-into-prod) to ensure you have followed all the steps to deploy your application to production.

## 9) Enable Sentry (Optional)

With 4000 events for free per month, we recommend that you use [Sentry.io](https://sentry.io) for
ensuring you can catch, analyze and fix any runtime errors in your application that your users will encounter.

## 10) Set up a Logging Service (Optional, highly recommended)

We recommend that you use a logging service to log any events that occur in your application. This will allow you to debug any issues that may occur in your application.

While this is optional, we highly recommend that you use a logging service. Logging is how you can debug issues in your application, and it is essential to have a logging service in place.

Some logging services that we recommend are:

- [Baselime](https://baselime.io)
- [Logflare](https://logflare.app)
- [Axiom](https://axiom.co)

The above can be installed with a one-click install in the Vercel dashboard (if you are using Vercel).

-----------------
FILE PATH: kits/next-supabase-lite/going-to-production.mdoc

---
status: "published"
title: "Going to Production in Next.js Lite Supabase (legacy)"
label: "Going to Production"
order: 14
description: "Learn how to prepare your application for shipping to production"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/how-to/api/api-routes-vs-server-actions.mdoc

---
status: "published"
title: "API Routes vs Server Actions | Next.js Supabase Lite SaaS Kit"
label: "API Routes vs Server Actions"
order: 2
description: "API routes and Server actions can be used for similar use cases. This guide will help you understand the differences between the two."
---

API Routes and Server Actions can indeed be used in similar ways: ultimately, they both allow you to write server-side code that can be called from the client-side. However, let's look at the use-cases for each of them to understand the differences.

#### TLDR

Use API Routes when fetching data, and Server Actions when performing actions/mutations.

## API Routes

API Routes are more generic and a lower-level abstraction than Server Actions. They are a way to write server-side code that can be called from the client-side. They are a great way to expose a REST API, or to write server-side code that can be called from the client-side.

The downside of API Routes is that they need to be called using the Fetch API, which you can use directly, or through a library such as React Query or SWR. This kit uses SWR.

Generally speaking, API Routes should be preferred when fetching data - rather than performing actions/mutations.

## Server Actions

Unlike API Routes, Server Actions are a higher-level abstraction. They are a way to write server-side code that can be called from the client-side as POST requests.

They are particularly useful when you want to perform actions/mutations on the server-side, such as creating a new user, or updating a user's profile.

Thanks to utilities such as `revalidatePath` and `revalidateTag`, you can also use Server Actions to invalidate the cache of a specific page or tag (used with `fetch`).

-----------------
FILE PATH: kits/next-supabase-lite/how-to/api/api-routes.mdoc

---
status: "published"

title: "Adding API Routes"
label: "Adding API Routes"
order: 0
description: ""
---


API routes are a way to create a custom API endpoint for your application. You can use API routes to create RESTful endpoints that return JSON data.

Adding an API route to a Makerkit application is no different than any other Next.js (App Router) project. Let's see how.

## Creating an API Route

To create an API route, you can use the file convention `route.ts` within the `app` directory. The file name will be the path name of the API route. For example, if you create a file called `hello/route.ts` inside the `app` directory, it will be accessible at `/hello`.

Here's an example of an API route that returns a JSON response as response to a `GET` request.

 ```ts {% title="app/hello/route.ts" %}
import { NextResponse } from "next/server";

export function GET() {
  return NextResponse.json({ text: 'Hello' });
}
```

This is kinda simple, right? Let's see how we can use this API route in our application.

### Protecting API Routes

It's very likely that you want to ensure only authenticated users can access your API routes. To do that, you can use the `requireSession` function.

In the example below, we create a `POST` API route that requires the user to be authenticated. If the user is not authenticated, the user will be redirected away. Additionally, the `requireSession` function will also ensure the level access if the user opted in to multi factor authentication.

 ```ts {% title="app/hello/route.ts" %}
import { NextRequest } from "next/server";

import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';
import requireSession from '~/lib/user/require-session';

export async function POST(req: NextRequest) {
  const client = getSupabaseRouteHandlerClient();
  const session = await requireSession(client);

  // user is authenticated, do something here
}
```

### CSRF Token

By default, all API routes are protected against CSRF attacks if they use a mutation method. This means that you need to send a CSRF token with every request.

To add a CSRF token to your request, you can use the `useCsrfToken` React hook. This function will return a CSRF token that you can use in your request.

POST requests without a CSRF token will return a `403` error. This can be disabled in the middleware `src/middleware.ts`.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/api/call-api-routes-from-client.mdoc

---
status: "published"
title: "Calling API Routes from the client-side using SWR | Next.js Supabase Lite SaaS Kit"
label: "Calling API Routes from the client"
order: 1
description: "Learn how to use SWR to call your Next.js Supabase Lite API route handlers from React components"
---

Makerkit includes the library SWR, a data-fetching React utility written by Vercel, the creators of Next.js.

This library is useful for making asynchronous requests from your React components. For example:

1. When interacting with the Supabase Client (DB, Storage, Auth, etc.)
2. When making requests to API Routes

Why is it useful? SWR makes it simpler to fetch and mutate data, caching and revalidating. Let's see how we can use it to make requests to our endpoints.

### Should I use SWR or Server Actions?

No exact answer - but here is what I do:

1. For pure mutations, I prefer Server Actions.
2. For pulling data from the server-side as a result of user interactions, I use SWR.

Ultimately, you should use what you feel more comfortable with as both approaches are valid.

## Using SWR for fetching data from an API Route Handler

Let's assume we have a very simple API Route at `/api/data`:

 ```tsx {% title="app/api/data/route.ts" %}
import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({
    hello: "world"
  });
}
```

As you may know, we can call the endpoint `/api/data` using a GET request and will receive the JSON response `{ "hello": "world" }`.

Now, we want to call this from our Next.js application using SWR, and use it within a React component.

We proceed building a React Hook with SWR which is able to fetch the data from our endpoint `/api/data`:

```tsx
import useSWR from 'swr';

export function useFetchData() {
  const key = '/api/data';

  return useSWR<{ hello: string }>([key], async () => {
    return fetch(key).then(res => res.json());
  });
}
```

Now, we can call this endpoint from any React component:

```tsx
function MyComponent() {
  const { data, isLoading, error } = useFetchData();

  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error...</p>;
  }

  return <div>{data.hello}</div>
}
```

## Mutating data with SWR

When it comes to mutating data, we can use the `swr/mutation` utility, designed to make mutations simple.

As you may know, mutations will likely use the method `POST`, `PUT`, `PATCH`, or `DELETE`. In these cases, the middleware in the kit will require us to send a CSRF token for security reasons.

As such, we provide a handy wrapper around `fetch`.

Let's write a very simple API Route that makes a mutation: we add a new record in the `tasks` table, and return the ID back to the client.

```tsx
import { NextRequest, NextResponse } from "next/server";

export async function POST(
  req: NextRequest
) {
  const client = getSupabaseRouteHandlerClient();
  const session = await requireSession(client);
  const task = req.json();

  const { data } = await client.from('tasks').insert({
    name: task.name,
    done: false,
    user_id: session.user.id,
  })
  .select('id')
  .single();

  return NextResponse.json({ id: data.id });
}
```

Now, let's write a React Hook that calls this endpoint using SWR.

```tsx
import useMutation from 'swr/mutation';
import useApiRequest from '~/core/hooks/use-api';

interface Task {
  name: string;
}

function useInsertTask() {
  const fetcher = useApiRequest();
  const path = '/api/task';

  return useMutation(
    path, async (_, data: { arg: Task }) => {
      return fetcher({
        path,
        body: data.arg,
        method: 'POST'
      });
    }
  );
}
```

What does `useApiRequest` do? This hook automatically inserts the CSRF token in our fetch requests, so that you don't have to.

If you prefer not using it, you can manually add the CSRF token instead:

```tsx
import useMutation from 'swr/mutation';
import useCsrfToken from '~/core/hooks/use-csrf-token';

interface Task {
  name: string;
}

function useInsertTask() {
  const csrfToken = useCsrfToken();
  const path = '/api/task';

  return useMutation(
    path, async (_, data: { arg: Task }) => {
      return fetch(path, {
        body: JSON.stringify(data.arg),
        method: 'POST',
        headers: {
          'x-csrf-token': csrfToken
        }
      });
    }
  );
}
```

We can now call the mutation from a React component:

```tsx
function TaskForm() {
  const insertTaskMutation = useInsertTask();

  const onSubmit: React.FormEventHandler<HTMLFormElement> = (event) => {
    const name = new FormData(event.currentTarget).get('name') ?? 'No Name';
    const task = { name };

    return insertTaskMutation.trigger(task);
  };

  return (
    <form onSubmit={onSubmit}>
      <input type={'name'} required />

      <button disabled={insertTaskMutation.isLoading}>
        Submit
      </button>
    </form>
  );
}
```


