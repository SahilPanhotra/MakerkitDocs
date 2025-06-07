-----------------
FILE PATH: kits\remix-fire\development\i18n-translations.mdoc

---
status: "published"

title: "Adding and updating locales in your Makerkit application"
label: Translations and Locales
order: 5
---


Much of the MakerKit's codebase uses translations using the package `next-i18next':

- this package allows you to create multi-language apps effortlessly
- it also helps easily renaming the Makerkit's entities according to your preferences: for example, renaming "organizations" to another entity such as "team" or "workspace"
- in a way, it helps write cleaner HTML

To add or update the default strings, you must use the locale files at `public/locales/{lang}.json`. For example, all the locale files for the English language are placed under `public/locales/en`.

{% alert type="warning" title="Restart the server" %}
When adding new keys, you'll need to restart the MakerKit server (`npm run dev`) to see the changes. Also, you may need to clear the browser cache since the client is hydrated with the translation files fetched from the JSON files.
{% /alert %}

The locales are split by functionality, page, or entity: as your application grows, you may not want to load every JSON file to reduce your bundle size.

For example, the translations for the authentication are placed in `auth.json,` while the ones used across every page are placed under `common.json.`

If you are unsure where to add some translations, simply add them to `common.json`.

{% alert type="info" title="All the translations are already loaded" %}
To simplify things, the MakerKit's starter uses all the available JSON files, and we recommend doing so until you have too many files.
{% /alert %}

### Adding new languages

By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:
1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **Remix i18n config**: We need to also add a new language to the Remix configuration at `app/i18n/i18next.config.ts`. Simply add your new language's code to the `locales` array.

The configuration will look like the below:

 ```js {% title="app/i18n/i18next.config.ts" %}
const i18Config = {
  fallbackLanguage: DEFAULT_LOCALE,
  supportedLanguages: [DEFAULT_LOCALE, 'es'],
  defaultNS: ['common', 'auth', 'organization', 'profile', 'subscription'],
  react: { useSuspense: false },
};
```

To add new locale files by default, add them to the `defaultNS` array.

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```

## Using the Language Switcher

The MakerKit kits come with a language switcher that allows users to switch between the available languages. This is not enabled by default, but you can easily add it anywhere in your application by importing the `LanguageSwitcherDropdown` component from `~/components/LanguageSwitcherDropdown`. 

It does not require any props, and it will automatically detect the available languages and display them in a dropdown.

If you want to customize the language switcher or use a different component, you can leverage the `useChangeLanguage` hook to change the language and store the selection as a cookie, which will then be used to set the default language.

-----------------
FILE PATH: kits\remix-fire\development\logging.mdoc

---
status: "published"
title: "Logging | Remix Firebase SaaS Kit"
label: Logging
order: 8
description: "Logging is a fundamental part for your SaaS to understand and
monitor the behavior of your code at runtime. Let's learn how to add logging
to your Makerkit application."
---


The Makerkit Boilerplate uses `pino` as logging library. Pino is simple and
lightweight, and it's used across the API functions to log important
information that helps you debug your code.

Generally speaking, you will find that we log every function, especially for
asynchronous operations that could fail for a number of reasons: network
issues, API exceptions, and so on.

So, how to log your API functions effectively? Our recommendation is to log
**before executing an operation** and then log the **result of the
operation**, without leaking important data.

Furthermore, we want to log information such as:
- which user is performing the operation?
- which organization is the user part of?
- any other information that can help you understand and debug what is
happening.

Let's assume you're writing an API function to write to your Firestore database:

```tsx
function addIntegrationHandler() {
  return writeToFirestore(data);
}
```

Let's rewrite the above by adding logging to your function:

```tsx
import logger from '~/core/logger';

async function addIntegrationHandler(
  userId: string,
  organizationId: number,
  integrationId: number,
) {
  // this is the context that every log will print out
  const loggingContext = {
    integrationId,
    organizationId,
    userId,
  };

  // Here we log what we're doing
  logger.log(loggingContext, `Adding new integration to organization`);

  try {
    await writeToFirestore(data);

    // Here we log that the result of the operation
    // was successful
    logger.log(loggingContext, `Integration successfully added`);

    // return successful response
    return res.json({
      integrationId,
      success: true
    });
  } catch (e) {
    // Here we log that the operation failed
    logger.error(loggingContext, `Encountered an error while adding integration`);

    // Logging errors can be okay but
    // ensure not to leak important information!
    logger.debug(e);

    // return 500
    return res.status(500).json({
      integrationId,
      success: false
    });
  }
}
```

If you use the filter `withExceptionFilter`, it will automatically
log eventual exceptions thrown by the function, which makes the try/catch
block optional.

NB: If you're using Vercel, logs are not persisted by default! [Check out the
Vercel integrations](https://vercel.com/integrations#logging) for adding persisting logs to your projects.


-----------------
FILE PATH: kits\remix-fire\development\user-roles.mdoc

---
status: "published"
title: "Extend User Roles | Remix Firebase SaaS Starter Kit"
description: "Learn how to extend the User Roles in the Remix Firebase SaaS Starter Kit"
label: User Roles
order: 9
---

The MakerKit Starter has three default roles:

```tsx
export enum MembershipRole {
  Member,
  Admin,
  Owner,
}
```

We use an `enum`, but you can convert this to an object if you need more granular permissions.

The permissions are hierarchical, which means that if we had a role with a lower level (`Readonly`), we would add it before `Member`:

```tsx
export enum MembershipRole {
  Readonly,
  Member,
  Admin,
  Owner,
}
```

When writing permissions between users, we can check if the user performing the action has a greater role than the target user.

You can extend the role above easily by adding your own, for example:

```tsx
export enum MembershipRole {
  Readonly,
  AccountManager,
  Owner,
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

Learn more about [using user roles in your permissions system](/docs/permissions).


-----------------
FILE PATH: kits\remix-fire\development\validating-api-input-zod.mdoc

---
status: "published"

title: "Validating the Remix API inputs with Zod and Typescript"
label: Validating API payload with Zod
order: 7
description: "Zod is a library for validating data with awesome support for
Typescript. Learn how to use it within your Makerkit project."
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

When we write an API endpoint, we first define the schema of the payload:

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

Now, let's write the body of the API handler that validates the body of the
function, which we expect to be equal to the `Body` interface.

```tsx
import { throwBadRequestException } from `~/core/http-exceptions`;

export const action: ActionFunction = async ({request}) => {
  try {
     // we can safely use data with the interface Body
    const data = await req.formData();
    const body = Object.fromEntries(data.entries());
    const bodyResult = await getBodySchema().parseAsync(body);
    const { displayName, email } = bodyResult.data;

    return sendInvite({ displayName, email });
  } catch(e) {
    return throwBadRequestException();
  }
}
```

You can also use `safeParse` if you prefer not to throw an error when the
validation fails:

```tsx
export const action: ActionFunction = async ({request}) => {
  const data = await req.formData();
  const body = Object.fromEntries(data.entries());
  const result = await getBodySchema().parseAsync(body);

  // we use result.success as a type guard
  // when false, we throw an exception
  if (!result.success) {
    return throwBadRequestException();
  }

  // TS correctly infers result.data now
  return sendInvite(result.data);
}
```

To learn more about validating data with Zod, we suggest you
check out [the Zod official documentation on GitHub](https://github.com/colinhacks/zod).


-----------------
FILE PATH: kits\remix-fire\getting-started\clone-repository.mdoc

---
status: "published"
title: Clone the Remix.js Firebase SaaS boilerplate repository
label: Clone the repository
order: 2
description: 'Learn how to clone the Remix.js Firebase repository'
---

If you have bought a license for MakerKit, you have access to all the
repositories built by the MakerKit team. In this document, we will learn how
to fetch and install the codebase.

### Requirements

To get started with the Remix and Firebase SaaS template, we need to ensure
you install the required software.

- Node.js (LTS recommended)
- Git
- Docker (optional but recommended)

### Cloning the repository

To get the codebase on your local machine, clone the repository with the
following command:

```
git clone git@github.com:makerkit/remix-firebase-saas-kit.git my-saas
```

The command above clones the repository in the folder `my-saas` which
you can rename it with the name of your project.

### Initializing Git

Now, run the following commands for:

1. Moving into the folder
2. Adding the remote to your own Git repository (if you haven't created a Git repository yet, you can do it later on)
3. Adding the original Makerkit repository as "upstream" so we can fetch updates from the main repository:

```
cd my-saas
git remote rm origin
git remote add origin <your-git-repository>
git remote add upstream git@github.com:makerkit/next-firebase-saas-kit.git
git add .
git commit -a -m "Initial Commit"
```

In this way, to fetch updates (after committing your files), simply run:

```
git pull upstream main
```

If it doesn't work, try with:

```
git pull upstream main --allow-unrelated-histories
```

You'll likely run into conflicts when running this command, so carefully choose the changes (sorry!).

### Installing the Node dependencies

Finally, we can install the NodeJS dependencies with `npm`:

```
npm i
```

While the application code is fully working, we now need to set up your Firebase
project.

So let's jump on to the next step!


-----------------
FILE PATH: kits\remix-fire\getting-started\coding-conventions.mdoc

---
status: "published"

title: SaaS template Coding Conventions
label: Coding Conventions
order: 5
description: 'An explanation of the coding conventions used in the boilerplate'
---


## React.FC vs React.FCC

Since React 18, `React.FC` no longer implies that components can accept a `children` prop. The alternative to that is using the interface `React.FC<React.PropsWithChildren<{}>>`, which is a bit long.

`React.FCC` is a shorthand that includes `children` as a possible property that works similarly to how it used to work. When a component needs `children`, `React.FCC` is preferred since it's faster to type.

## Exporting functions vs constants

We use both conventions in the boilerplate, i.e., exporting functions as components and constants.

My general rule is the following:

- if it's the exported component, use `const` typed with `React.FC` or `React.FCC`
- if it's not exported, use `function` since my personal preference is to define functions below the exported component

NB: sometimes, that may not be the case, and that's not necessarily intentional. The two constructs are essentially the same in most cases.

## Why is useCallback used in most components?

Most functions defined within a render function use `useCallback`. In my experience, this is the better default to avoid re-renders that I can't immediately explain.

Whether you want to adhere to this convention is totally up to you; just be aware of the potential issues.

## Interfaces, Types, and Inline 

I've gone through phases, so you may find some types defined as `type` and others as `interface`. I've since realized that I prefer `interface`, but you will find some instance that uses `type`.

Inline types are typically only defined when the type isn't reused anywhere. For example, when defining a function that accepts parameters as objects.

## Function parameters

I adhere to the principle that if a function accepts multiple parameters of the same type, then use an object. Unfortunately, Typescript can't save us if the types match.

Consider the example below:

```tsx
function makeDiv(width: number, height: number) {}

const height = 10;
const width = 40;

// correct according to TS, not in practice!
const div = makeDiv(height, width);
```

Now we rewrite it using an object:

```tsx
function makeDiv(params: { width: number, height: number}) {}

const height = 10;
const width = 40;

// Okay, now it looks better!
const div = makeDiv({ height, width });
```

## Custom React Hooks

The boilerplate uses Custom React Hooks when:

- fetching/writing using an API function
- fetching/writing using Firestore
- fetching/writing using Storage

It makes the code more understandable and reusable.

-----------------
FILE PATH: kits\remix-fire\getting-started\introduction.mdoc

---
status: "published"
title: 'Introduction to MakerKit: a SaaS boilerplate for Remix and Firebase'
label: 'Introduction'
order: 0
description: 'Introduction to the Remix and Firebase kit'
---

MakerKit is **the Remix.js boilerplate project for SaaS** built to get you
started on the right foot.

MakerKit is a fully SSR-rendered Remix application that uses Firebase for
authentication, database (with Firestore), and storage.

MakerKit is ideal for any SaaS application or dynamic websites such as walled Online Publications.

It particularly shines when you wish to provide uniform navigation between your marketing site, documentation, and application.

## What does the boilerplate provide?

### Tech Stack

We built MakerKit with some of the best technologies available today, such as
Remix, Tailwind CSS, and Firebase:

- **Scalable Remix** structure template, perfect for the most ambitious
projects
- Beautiful **Tailwind 3** CSS theme - with dark mode!
- **Full Firebase** setup with local emulators
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
- **MDX-powered Blog and Documentation** generators - already
**SEO-optimized** for you [Coming Soon]

### Marketing

Marketing matters! But don't get hung up on the technicalities. MakerKit provides:

- **Newsletter Sign-up form** for *ConvertKit*, but easy to adapt to more
providers
- **Google Analytics** script baked-in
- **Email Templates** you can write with React using [React.email]
(https://react.email)

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

MakerKit has plenty of resources for using both Remix and Firebase - and if you're stuck, you can always reach out to me for help.


-----------------
FILE PATH: kits\remix-fire\getting-started\running-the-application.mdoc

---
status: "published"

title: How to run a Remix and Firebase MakerKit application
label: Running the application
order: 3
description: 'Learn how to run a MakerKit application'
---


After installing the modules and the emulators, we can finally run the
application in development mode.

We need to execute two commands (and an optional one for Stripe):

1. **Remix Server**: the first command is for running the Remix server
2. **Firebase Emulators**: the second command is for running the Firebase
emulators
3. **Stripe CLI**: finally, the Stripe CLI is needed to dispatch webhooks to
our local server (optional, only needed when interacting with Stripe)

## About this Documentation

This documentation complements the Firebase one and is not meant to be a replacement. We recommend reading the Firebase documentation to get a better understanding of the Firebase concepts and how to use it.

### Running the Firebase Emulators

First, let's run the emulators. The command below runs the emulators for
the following products: Authentication, Firestore, and Storage.

```bash
npm run firebase:emulators:start
```

Additionally, it imports the default emulator's data from the folder `.
/firebase-seed`, which we set up upfront to run the Cypress tests.

After running the command above, you will be able to access the [Firebase
Emulators UI](https://firebase.google.com/docs/emulator-suite) at
[http://localhost:4000](http://localhost:4000).

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

## Saving the Firebase Emulator's data

After shutting the emulators down, the Firebase emulator clears all the records
and the users added during the session while the emulators are running.

Fortunately, we can save and restore as many dumps as we want. For example, this can be useful for saving and loading particular scenarios for testing.

To save the current snapshot, run the following command while the emulators are up and running:

```bash
npm run firebase:emulators:export
```

By default, MakerKit imports and exports to `./firebase-seed`, but you can change this to any other path.

### Always Save on exit

We don't particularly recommend this, but if you want to keep persisting data as you exit the emulators,
you can decorate the `firebase:emulators:start` with the following options:

```bash
firebase emulators:start --export-on-exit=./your-directory
```

Saving your emulators' state can be handy, but keep in mind that for testing reasons,
it's always best to create a snapshot of your data and use the same one so that your tests won't break if the data changes.

Of course, change `./your-directory` to your preferred path.


-----------------
FILE PATH: kits\remix-fire\getting-started\technical-details.mdoc

---
status: "published"
title: The technologies that make up a Remix.js Firebase application
label: Technical Details
order: 1
description: 'MakerKit uses Remix, Firebase, Firestore, Tailwind CSS, Zod,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
---

MakerKit is built primarily with Remix, Firebase, and Tailwind CSS.

Under the hood, there are many more libraries and architectural
decisions you should know.

### 1) A Scalable Remix Application
The codebase is entirely built with [Remix](https://remix.run).

The front end is fully built with React, leveraging React Hooks
and the best practices available today.

### 2) Built on top of Firebase

[Firebase](https://firebase.com) is one of the most popular *PaaS* out there,
with a very generous free tier and fairly affordable pricing for growing
applications.

MakerKit uses Firebase for Authentication, Firestore for storing your data,
Storage for storing files, and AppCheck to protect your endpoints against
possible attacks.

### 3) Theme built with Tailwind 3

The bulk of the application's theme, built with [Tailwind 3](https://tailwindcss.com/), is
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


-----------------
FILE PATH: kits\remix-fire\going-to-production\going-to-production-overview.mdoc

---
status: "published"
title: Checklist to ship a Remix.js Firebase SaaS application to Production
description: This checklist will help you ship your Remix.js Firebase SaaS application to production
label: Production Checklist
order: 0
---

Before going to production, it's essential to follow certain precautions to avoid failures.

## 1) Environment Variables

First, we must safely inject the correct environment variables into their relative environment. MakerKit has predefined templates that allow you to understand where each variable should be added.

Before publishing your SaaS, or if you are making a production release after
a change and this included an environment variable; ensure your CI provider is
up to date with the correct variable in the right environments.

Since the `.env` file will not be committed to your repository, you will
need to add the environment variables beforehand.

## 2) Firebase Security Rules (Firestore and Storage)

Whenever your Security rules need updating, you have to make sure to
propagate these changes and publish them from your Firebase Console.

## 3) Firestore Indexes

As you may know, the Firestore emulator does not fail when a new index is
needed, like its production version. This can be dangerous!

If you are setting up Makerkit for the first time, remember to set up the
an index for querying an invitation without knowing its organization.

### Create Index Exemption by clicking on the link below

You can do so using the following link (replace
"\[\[YOUR_PROJECT_ID\]\]" with your real project ID):

https://console.firebase.google.com/u/0/project/\[\[YOUR_PROJECT_ID\]\]/firestore/indexes/single-field?create_exemption=ClBwcm9qZWN0cy9zdG9yeWpveS00NmVlMy9kYXRhYmFzZXMvKGRlZ
mF1bHQpL2NvbGxlY3Rpb25Hcm91cHMvaW52aXRlcy9maWVsZHMvY29kZRACGggKBGNvZGUQAQ

If you have added other complex queries that need a Firestore index
exemption, **ensure you are testing thoroughly with an actual Firestore database
to avoid similar issues**.

## 4) Enable Authentication providers

If you have added authentication providers other than Google, make sure you
have enabled them from the Console and added the needed configuration so
that they work correctly.

For many of them, you will have to provide an app
ID or a key, so configure and test them in a production environment.

## 5) Enable your Stripe account Billing Status

Remember to complete your Stripe account's information and enable billing and to not forget it in `Testing Mode`.

Additionally, you will need to add your Stripe API keys to the environment variables to process payments. Ensure you have added the correct keys to the correct environment and that have selected all the required events to your webhook.

Please follow the [Stripe configuration guide](/docs/remix-fire/stripe-configuration) to configure your Stripe account.

## 6) Add your SMPT service credentials

You will need to add your SMTP service credentials to the environment variables to send emails. This is required for sending emails to users when they are invited to an organization.

To set the credentials, use the global configuration file `configuration.ts`.

## 7) Enable Google Analytics (Optional)

Understand how visitors find your website early on.

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

