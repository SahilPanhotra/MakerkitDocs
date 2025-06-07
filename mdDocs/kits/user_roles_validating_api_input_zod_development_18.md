-----------------
FILE PATH: kits/next-supabase/development/user-roles.mdoc

---
status: "published"
title: "Extend User Roles | Next.js Supabase SaaS Starter Kit"
description: "Learn how to extend the User Roles in the Next.js Supabase SaaS Starter Kit"
label: User Roles
order: 9
---

The MakerKit Starter has three default roles:

```tsx
enum MembershipRole {
  Member = 0,
  Admin = 1,
  Owner = 2,
}

export default MembershipRole;
```

We use an `enum`, but you can convert this to an object if you need more granular permissions.

The permissions are hierarchical, which means that if we had a role with a lower level (`Readonly`), we would add it before `Member`:

```tsx
export enum MembershipRole {
  Readonly = 0,
  Member = 1,
  Admin = 2,
  Owner = 3,
}
```

When writing permissions between users, we can check if the user performing the action has a greater role than the target user.

You can extend the role above easily by adding your own, for example:

```tsx
export enum MembershipRole {
  Readonly = 0,
  AccountManager = 1,
  Owner = 2,
}
```

Afterward, remember to add the name and descriptions of these roles in the translations file `common.json`:

```json
"roles": {
  "owner": {
    "label": "Owner",
    "description": "Can change any setting, invite new members and manage billing"
  },
  "accountmanager": {
    "label": "Account Manager",
    "description": "Can change some settings, invite members, perform disruptive actions"
  },
  "readonly": {
    "label": "Readonly",
    "description": "Can only read information"
  }
}
```

Learn more about [using user roles in your permissions system](permissions).

The role is stored in the database as a column in the `memberships` tabel.

-----------------
FILE PATH: kits/next-supabase/development/validating-api-input-zod.mdoc

---
status: "published"
title: "Validating the Next.js API inputs with Zod and Typescript | Next.js Supabase SaaS Kit"
label: Validating API payload with Zod
order: 7
description: "Zod is a library for validating data with awesome support for
Typescript. Learn how to use it within your Next.js Supabase project."
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
FILE PATH: kits/next-supabase/development.mdoc

---
status: "published"
title: "Development in Next.js Supabase"
label: "Development"
order: 6
description: "This section will teach you everything you need to know about developing with Next.js Supabase."
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/getting-started/clone-repository.mdoc

---
status: "published"
title: Clone the Next.js Supabase SaaS boilerplate repository
label: Clone the repository
order: 2
description: 'Learn how to clone the Next.js Supabase kit repository'
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

#### Cloning the repository

To get the codebase on your local machine using the original repository, clone the repository with the
following command:

```
git clone git@github.com:makerkit/next-supabase-saas-kit.git my-saas
```

The command above clones the repository in the folder `my-saas` which
you can rename it with the name of your project.

#### Initializing Git

Now, run the following commands for:

1. Moving into the folder
2. Point to your own Git repository

```
cd my-saas
git remote rm origin
git remote add origin <your-git-repository>
```

If you haven't created a Git repository yet, you can do it later on.

### Setting the Upstream repository, and fetching updates

Now, we can add the original Makerkit repository as "upstream" so we can fetch updates from the main repository:

```
git remote add upstream git@github.com:makerkit/next-supabase-saas-kit.git
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
FILE PATH: kits/next-supabase/getting-started/introduction.mdoc

---
status: "published"

title: 'Introduction to MakerKit: a SaaS boilerplate for Next.js and Supabase'
label: 'Introduction'
order: 0
description: 'Introduction to MakerKit: a SaaS starter built with Next.js and Supabase'
---


MakerKit is **the Next.js boilerplate project for SaaS** built to get you
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

- **Payments** with Stripe, and support for **subscriptions** (or, alternatively, using Lemon Squeezy)
- **Onboarding flow**, which allows your users to set up their accounts and
create their organization
- **Organizations**: users can create and edit organizations. An organization is a group of users. You can rename the `Organization` entity according to your domain (for example, teams, projects, etc.)
- **Team Members**: users can invite other users to join their organization and assign them a role
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
FILE PATH: kits/next-supabase/getting-started/navigating-codebase.mdoc

---
status: "published"

title: Navigating the Codebase
label: Navigating the Codebase
order: 4
description: "Learn how to navigate the Next.js Supabase codebase - so you can understand and extend the codebase to your needs."
---


In this section - we will take an overview of the Next.js Supabase codebase. We
will look at the different folders and files and what they do.

## The "supabase" folder

This is the folder generated by Supabase when initiating a new project. In this folder, Supabase stores the local configuration `config.toml`, the database migrations, the seed file and the tests.

Most importantly, you will be more interested in the `migrations` folder. This folder contains the database migrations that seed your database with the necessary tables and data when starting the Supabase server.

It's here where you will be editing or adding new migrations to your database.

## The "public" folder

In this folder you can store static files that you want to serve from your application. For example, you can store images, fonts, etc. in this folder.

## The "src" folder

The `src` folder contains the various folders and files:
- `core`
- `lib`
- `components`
- `app`
- `i18n`
- `configuration.ts`
- `database.types.ts`

### The "core" folder

The core folder contains the building blocks of your application, and the core utilities that you will use throughout your application.

It does not contain any business logic, but rather contains the building blocks that you will use to build your business logic.

#### Reusable components

The `core/ui` folder contains the reusable components that you will use throughout your application.

It's best to place here components not related to your app's business logic. For example, you can place here components like `Button`, `Input`, `Select`, etc.

### The "lib" folder

The `lib` folder contains business-logic related code - and it's here where you will be writing most of your code.

### The "components" folder

The `components` folder contains the various components that you will use throughout your application.

For example, here you can add components such as `UsersList`, `UserDetails`, `UserForm`, etc. If they relate to a feature, this is a good place to put them.

### The "app" folder

This is the Next.js routes folder. It contains the various routes that your
application will have.

### The "i18n" folder

This folder contains the settings and utilities for internationalization in your application.

### The "configuration.ts" file

This file contains the configuration for your application. It's here where you will configure your auth providers, your application's name, etc.

### The "database.types.ts" file

This file is automatically generated by Supabase. It contains the types for your database tables and we use it to add type-safety to our database queries.

-----------------
FILE PATH: kits/next-supabase/getting-started/running-the-application.mdoc

---
status: "published"

title: How to run a Next.js and Supabase MakerKit application
label: Running the application
order: 3
description: 'Learn how to run a Next.js and Supabase MakerKit application'
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
file `.env.development`.

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

You should now be able to sign in.

To quickly get started, use the following credentials:

```
email = test@makerkit.dev
password = testingpassword
```

#### Email Confirmations

When signing up, Supabase sends an email confirmation to a testing account using [InBucket](http://localhost:54324/monitor).

You can access the InBucket testing emails [using the following link](http://localhost:54324/monitor), and can follow the links to complete the sign up process.




-----------------
FILE PATH: kits/next-supabase/getting-started/technical-details.mdoc

---
status: "published"
title: The technologies that make up a Next.js Supabase Makerkit application
label: Technical Details
order: 1
description: 'MakerKit uses Next.js App Router, Supabase, Tailwind CSS, Zod, SWR,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
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

### 9) Multi-language by default using i18next

We configured the application to be translated by default thanks to the
`i18next` package.

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

### 11) SWR for data fetching and mutations

MakerKit uses Vercel's SWR for data fetching and mutations when used from the client-side, which is in very few places, as we mostly use Server Actions.


-----------------
FILE PATH: kits/next-supabase/getting-started.mdoc

---
status: "published"
title: "Getting Started in Next.js Supabase"
label: "Getting Started"
order: 1
description: "Next.js Supabase is a boilerplate for building SaaS applications with Next.js and Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/going-to-production/going-to-production-overview.mdoc

---
status: "published"

title: Checklist to ship a MakerKit application to Production
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

**Remember to do the same for your emails in Supabase** - as the limits are extremely low in the free plan and may not be enough for your application.

Please follow the [email configuration guide](/docs/next-supabase/emails) to learn more about how to configure your SMTP service.

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
FILE PATH: kits/next-supabase/going-to-production.mdoc

---
status: "published"
title: "Going to Production in Next.js Supabase (legacy)"
label: "Going to Production"
order: 14
description: "Learn how to prepare your application for shipping to production"
collapsible: true
collapsed: true
---

