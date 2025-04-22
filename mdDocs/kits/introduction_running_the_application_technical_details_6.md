-----------------
FILE PATH: kits\next-fire\getting-started\introduction.mdoc

---
status: "published"

title: 'Introduction to MakerKit: a SaaS boilerplate for Next.js and Firebase'
label: 'Introduction'
order: 0
description: 'Introduction to MakerKit: a SaaS starter built with Next.js and Firebase'
---

MakerKit is **the Next.js boilerplate project for SaaS** built to get you started on the right foot.

MakerKit is a fully SSR-rendered Next.js application that uses Firebase for authentication, database (with Firestore), and storage. MakerKit is ideal for any SaaS application or dynamic websites such as walled Online Publications.

It particularly shines when you wish to provide uniform navigation between your marketing site, documentation, and internal application.

## What does the boilerplate provide?

### Tech Stack

We built MakerKit with some of the best technologies available today, such as
Next.js, Tailwind, and Firebase:

- **Scalable Next.js** structure template, perfect for the most ambitious
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
**SEO-optimized** for you

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

MakerKit has plenty of resources for using both Next.js and Firebase - and if you're stuck, you can always reach out to me for help.


-----------------
FILE PATH: kits\next-fire\getting-started\running-the-application.mdoc

---
status: "published"

title: How to run a Next.js and Firebase MakerKit application
label: Running the application
order: 3
description: 'Learn how to run a MakerKit application'
---

After installing the modules and the emulators, we can finally run the
application in development mode.

We need to execute two commands (and an optional one for Stripe):

1. **Next.js Server**: the first command is for running the Next.js server
2. **Firebase Emulators**: the second command is for running the Firebase
emulators
3. **Stripe CLI**: finally, the Stripe CLI is needed to dispatch webhooks to
our local server (optional, and only needed when interacting with Stripe)

## About this Documentation

This documentation complements the Firebase one and is not meant to be a replacement. We recommend reading the Firebase documentation to get a better understanding of the Firebase concepts and how to use it.

### Running the Firebase Emulators

First, let's run the emulators.

The command below runs the emulators for
the following products: Authentication, Firestore, and Storage.

```bash
npm run firebase:emulators:start
```

Additionally, it imports the default emulator's data from the folder `.
/firebase-seed`, which we set up upfront to run the Cypress tests.

After running the command above, you will be able to access the [Firebase
Emulators UI](https://firebase.google.com/docs/emulator-suite) at
[http://localhost:4000](http://localhost:4000).

### Running the Next.js Server

And now, the Next server:

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
FILE PATH: kits\next-fire\getting-started\technical-details.mdoc

---
status: "published"
title: The technologies that make up a Next.js Firebase application
label: Technical Details
order: 1
description: 'MakerKit uses Next.js, Firebase, Firestore, Tailwind CSS, Zod,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
---

MakerKit is built primarily with Next.js, Firebase, and Tailwind CSS.

Under the hood, there are many more libraries and architectural
decisions you should know.

### 1) A Scalable Next.js Application
The codebase is entirely built with [Next.js](https://nextjs.org) (version 13),
a popular React meta-framework created by the fine folks at [Vercel](https://vercel.com).

The front end is fully built with React, leveraging React Hooks
and the best practices available today.

The boilerplate is currently using the `pages` folder structure.

### 2) Built on top of Firebase

[Firebase](https://firebase.com) is one of the most popular *PaaS* out there,
with a very generous free tier and fairly affordable pricing for growing
applications.

MakerKit uses Firebase for Authentication, Firestore for storing your data,
Storage for storing files, and AppCheck to protect your endpoints against
possible attacks.

### 3) Theme built with Tailwind CSS

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

### 9) Multi-language by default with next-i18n

We configured the application to be translated by default thanks to the
[next-i18n](http://next-i18next.com) package.

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
FILE PATH: kits\next-fire\getting-started.mdoc

---
status: "published"
title: "The technologies that make up a Next.js Firebase Makerkit application"
label: "Getting Started"
order: 1
description: 'MakerKit uses Next.js, Firebase, Firestore, Tailwind CSS, Zod,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
collapsible: true
collapsed: true
---

MakerKit is built primarily with Next.js, Firebase, and Tailwind CSS.

Under the hood, there are many more libraries and architectural
decisions you should know.

### 1) A Scalable Next.js Application
The codebase is entirely built with [Next.js](https://nextjs.org) (version 13),
a popular React meta-framework created by the fine folks at [Vercel](https://vercel.com).

The front end is fully built with React, leveraging React Hooks
and the best practices available today.

The boilerplate is currently using the `pages` folder structure.

### 2) Built on top of Firebase

[Firebase](https://firebase.com) is one of the most popular *PaaS* out there,
with a very generous free tier and fairly affordable pricing for growing
applications.

MakerKit uses Firebase for Authentication, Firestore for storing your data,
Storage for storing files, and AppCheck to protect your endpoints against
possible attacks.

### 3) Theme built with Tailwind CSS

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

### 9) Multi-language by default with next-i18n

We configured the application to be translated by default thanks to the
[next-i18n](http://next-i18next.com) package.

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
FILE PATH: kits\next-fire\going-to-production\environment-variables.mdoc

---
status: "published"

title: "Setting the correct environment variables of a Next.js and Firebase
application"
label: Environment Variables
order: 1
---

Before going live to production, setting the correct environment
variables for your production environment is crucial. 

As we've already seen before, you can organize your environment variables into three files:

- `.env`: here, you can add variables that are shared across environments
- `.env.local`: here go the variables that should be limited to your own machine, and it's recommended not to check this file in your VCS
- `.env.development`: here, you can add the variables that should only be used during development
- `.env.production`: here, you add the variables that should be used in your production environment

Typically, you would never place private data in your `.env.production` file because that would cause your data to be leaked.

You should set any value that is not secret or confidential (such as your Firebase project's API Key or the feature flag's value) in the `.env.production` file.

Alternatively, suppose you are using variables such as the project's private key or any value that is supposed to be secret: in that case, you should ensure to inject these values from a secure environment, for example, your CI or your Vercel console.

## Shared Variables

As mentioned above, we can add variables shared across environments in the `.env` configuration file. For example, MakerKit's `.env` template looks like the below:

```
# firebase
DEFAULT_LOCALE=en
```

## Local Variables

To use data only limited to your machine, use a `.env.local` configuration file. Do not check this file in your VCS: therefore, you can use it to set private variables:

```
SERVICE_ACCOUNT_PRIVATE_KEY=
SECRET_KEY_THAT_SHOULD_NOT_BE_EXPOSED=
```

## Development Variables

In our `.env.development` configuration, we will set variables such as the Firebase Emulator's configuration that will only run when running `next dev`, e.g., when we're developing the application:

```
NEXT_PUBLIC_EMULATOR=true
FIRESTORE_EMULATOR_HOST=localhost:8080
FIREBASE_AUTH_EMULATOR_HOST=localhost:9099
FIREBASE_STORAGE_EMULATOR_HOST=localhost:9199
FIREBASE_PUBSUB_EMULATOR_HOST=localhost:8085
NEXT_PUBLIC_FIREBASE_EMULATOR_HOST=localhost
NEXT_PUBLIC_FIRESTORE_EMULATOR_PORT=8080
NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_PORT=9099
NEXT_PUBLIC_FIREBASE_STORAGE_EMULATOR_PORT=9199
FEATURE_FLAG=true
```


## Production Variables

We will set variables for the production application in our `.env.production` configuration. 

Rename the file `.env.production.template` to `.env.production` and add the required variables after creating your Firebase and Vercel projects:

This needs to be filled in before you push your application to your CI:

```
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=<your-website-domain>
NEXT_PUBLIC_SITE_URL=
NEXT_PUBLIC_APPCHECK_KEY=

SERVICE_ACCOUNT_CLIENT_EMAIL=
GCLOUD_PROJECT=

## SECRET KEYS ARE BEST ADDED TO YOUR CI
SERVICE_ACCOUNT_PRIVATE_KEY=

NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

Careful! Do not add sensitive data here. Instead, add it from your CI or Vercel Console.

Please replace `<your-website-domain>` in the variable `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` with your own website domain. This is fundamental for the kit to work with Safari.

-----------------
FILE PATH: kits\next-fire\going-to-production\publish-firebase-security-rules.mdoc

---
status: "published"

title: "Publish the Firebase Security Rules variables before going to production"
label: Security Rules
order: 2
---

When running the Firebase Emulator, you can update your Firebase Firestore and Storage security rules by changing the relative files `firestore.rules` and `storage.rules`.

Before deploying your application to your users, you need to make sure that
you're also publishing the security rules from your Firebase Console.

### Storage Security Rules

To publish your Firebase Storage security rules, navigate to the Firebase
Console.

From the main sidebar, click on "Storage". Afterward, click on the "Rules" tab:

{% img width="955" height="719" src="/assets/images/docs/storage-rules-tab.webp" /%}

Afterward, paste the content from `storage.rules` to the text editor, and
click "Publish".

{% img width="758" height="628" src="/assets/images/docs/storage-rules-editor.webp" /%}

### Firestore Security Rules

To update your Firestore Security rules, follow the same steps as for the
Storage rules explained above.

Of course, the only difference is navigating to the Firestore tab, and
choosing the content of `firestore.rules`.


-----------------
FILE PATH: kits\next-fire\going-to-production.mdoc

---
status: "published"
title: Checklist to ship a Next.js Firebase SaaS application to Production
description: This checklist will help you ship your Next.js Firebase SaaS application to production
label: Production Checklist
order: 12
collapsible: true
collapsed: true
---

Before going to production, it's essential to follow certain precautions to avoid failures.

## 1) Environment Variables

First, we must safely inject the correct environment variables into their relative environment. MakerKit has predefined templates that allow you to understand where each variable should be added.

Before publishing your SaaS, or if you are making a production release after
a change and this included an environment variable; ensure your Vercel
Console is up to date with the correct variable in the right environments.

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


Please follow the [payments configuration guide](/docs/next-fire/stripe-configuration) to learn more about how to configure your Stripe account.

## 6) Add your SMPT service credentials

You will need to add your SMTP service credentials to the environment variables to send emails. This is required for sending emails to users when they are invited to an organization.

Please follow the [email configuration guide](/docs/next-fire/emails) to learn more about how to configure your SMTP service.

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
## 10) Deploy to Production

Once you have completed all the steps above, you can deploy your application to production. Since a Makerkit application is a simple Next.js application, [the Next.js official deployment guide is the best place to start](https://nextjs.org/docs/deployment).

-----------------
FILE PATH: kits\next-fire\how-to\api\api-routes.mdoc

---
status: "published"

title: "Adding API Routes"
label: "Adding API Routes"
order: 0
description: ""
---

API routes are a way to create a custom API endpoint for your application. You can use API routes to create RESTful endpoints that return JSON data.

Adding an API route to a Makerkit application is no different than any other Next.js (Pages Router) project. Let's see how.

## Creating an API Route

To create an API route, create a file inside the `pages/api` directory. The file name will be the path name of the API route. For example, if you create a file called `hello.ts` inside the `pages/api` directory, it will be accessible at `/api/hello`.

Here's an example of an API route that returns a JSON response:

 ```ts {% title="pages/api/hello.ts" %}
import { NextApiRequest, NextApiResponse } from "next";

export default (req: NextApiRequest, res: NextApiResponse) => {
  res.status(200).json({ text: 'Hello' })
}
```

This is kinda simple, right? Let's see how we can use this API route in our application.

### Protecting API Routes

It's very likely that you want to ensure only authenticated users can access your API routes. To do that, you can use the `withAuthedUser` utility to wrap your API route handler.

 ```ts {% title="pages/api/hello.ts" %}
import { NextApiRequest, NextApiResponse } from "next";
import { withAuthedUser } from '~/core/middleware/with-authed-user';

function helloWorldHandler(req: NextApiRequest, res: NextApiResponse) {
  res.status(200).json({ text: 'Hello' })
}

export default withAuthedUser(
  helloWorldHandler
);
```

When using the `withAuthedUser` HOC, the `req.firebaseUser` object will be available in your API route handler. This object contains the user's information from their Firebase Auth account.

You can use this object to retrieve data from your database, or to perform other actions, based on the user's information.

#### Using a Pipe for API Routes

We can pipe multiple middlewares together to create a pipeline for our API routes. For example, we can use the `withAuthedUser` middleware to ensure only authenticated users can access our API routes, and then use the `withCsrf` middleware to ensure only users with a specific role can access the API route.

 ```ts {% title="pages/api/hello.ts" %} {14-18}
import { NextApiRequest, NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withPipe } from '~/core/middleware/with-pipe';

function helloWorldHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  res.status(200).json({ text: 'Hello' })
}

export default withPipe(
  withAuthedUser,
  withMethodsGuard(['GET', 'POST']),
  helloWorldHandler,
);
```

### Checking CSRF Tokens

You can use the `withCsrf` HOC to ensure only users with a valid CSRF token can access your API routes.

The CSRF token must be sent in the `x-csrf-token` header of the request. If the token is not present, or if it's invalid, the request will be rejected.

 ```ts {% title="pages/api/hello.ts" %} {18}
import { NextApiRequest, NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withPipe } from '~/core/middleware/with-pipe';
import withCsrf from "./with-csrf";

function helloWorldHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  res.status(200).json({ text: 'Hello' })
}

export default withPipe(
  withAuthedUser,
  withMethodsGuard(['POST']),
  withCsrf(),
  helloWorldHandler,
);
```

If you pass the CSRF token in different ways, you can pass a function to the `withCsrf` HOC to retrieve the token from the request.

 ```tsx {% title="pages/api/hello.ts" %} {12}
import { NextApiRequest, NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withPipe } from '~/core/middleware/with-pipe';
import withCsrf from "./with-csrf";

function helloWorldHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  await withCsrf(req, () => req.body.csrfToken);

  res.status(200).json({ text: 'Hello' })
}

export default withPipe(
  withAuthedUser,
  withMethodsGuard(['POST']),
  helloWorldHandler,
);
```

