-----------------
FILE PATH: kits\next-supabase\tutorials\deploying-production.mdoc

---
status: "published"

title: "Deploying to Production"
label: "Deploying to Production"
order: 16
description: "Learn how to deploy your Next.js Supabase app to your hosting provider."
---


Before deploying to production or any remote server, you need to [follow all the steps outlined in the documentation](/docs/next-supabase/going-to-production-overview).

**Much of this work needs to be done externally**, not in the Makerkit codebase.

1. **Supabase**: Create a new Supabase project, and add the relevant environment variables to your project
2. **Link project with the CLI**: Link the Supabase project with your local CLI
3. **Remote Database**: Deploy the database schema and migrations to your Supabase project
4. **Environment Variables**: Ensure that your environment variables are set correctly
5. **Auth**: Enable the authentication providers you want to use from the Supabase Console
6. **SMTP**: Set up an SMTP server to send emails for team invites - and also do the same in Supabase Console to avoid their very low limits and better deliverability
7. **Payments**: Set up your Stripe or Lemon Squeezy accounts and add the relevant environment variables
8. **Deployment**: Finally, deploy your application to your hosting provider (Vercel, Firebase, Netlify, etc.): please follow the instructions provided by your hosting provider to deploy a Next.js application.

-----------------
FILE PATH: kits\next-supabase\tutorials\environment-variables.mdoc

---
status: "published"

title: "Environment Variables"
label: Environment Variables
order: 5
description: "Learn how to use environment variables in your Next.js project."
---


The starter project comes with two different environment variables files:
1. **.env**: the main environment file - use this for variables that are common to all environments - and that are not secrets.
2. **.env.development**: the development environment file (used when running the project locally in development mode) - when they're specific to the development environment.
3. **.env.production**: the production environment file (used when deploying or running the build command) - when they're specific to the production environment.
4. **.env.local**: this environment file is loaded when running the project locally. It won't be committed to Git. Useful for adding secrets that you don't want to commit to Git.
5. **.env.test**: this environment file is loaded when running the Cypress E2E tests. You would rarely need to use this.


-----------------
FILE PATH: kits\next-supabase\tutorials\functions.mdoc

---
status: "published"

title: "Functions you need to know"
label: "Functions you need to know"
order: 15
description: "Learn the most important functions you need to know to use the Next.js Supabase kit effectively."
---


This is an extremely important section of the documentation: it will teach you the most important functions you need to know to use the Next.js Supabase kit effectively.

We will cover both client-side and server-side functions.

## Client-Side Functions

### Supabase Hooks

The Next.js Supabase kit uses the client Supabase SDK extensively throughout the kit. You can find the full documentation of the Supabase SDK [here](https://supabase.io/docs/reference/javascript/supabase-client).

To interact with the client-side Supabase SDK, you can use the hook `useSupabase`.

```tsx
import useSupabase from '~/core/hooks/use-supabase';

function Component() {
  // This is the Supabase client
  const supabase = useSupabase();
}
```

Using the `useSupabase` hook, you can access the Supabase client anywhere in your application, and it will be initialized with the correct Supabase URL and Supabase key that you have set in your environment variables.

#### Server Side Supabase Clients
There are other variations for use on the server-side, but of course these are not hooks since you will be using them outside of React components.

We will take a look at server-side Supabase functions in the next sections.

### Makerkit Hooks

#### 1. "useCurrentOrganization": getting the current user organization

This hook will allow you to get the current user organization. It is populated with the data fetched server-side in the `dashboard/[organization]` layout, and is injected using the Context API.

For example, let's see how we can use this hook to get the current user organization name:

```ts
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

export function useCurrentOrganizationName() {
  const organization = useCurrentOrganization();

  return organization?.name;
}
```

There are a variety of other hooks that you can use to get the current user organization data. You can find them in the `~/lib/organizations/hooks` folder.

#### 2. "useCurrentUserRole": getting the current user role within the current organization

In case you need to access the current user role within the current organization, you can use the `useCurrentUserRole` hook.

For example, let's see how we can use this hook to get the current user role within the current organization:

```tsx
import { useCurrentUserRole } from '~/lib/organizations/hooks/use-current-user-role';

function MyComponent() {
  const role = useCurrentUserRole();

  return (
    <div>
      <p>Current user role: {role}</p>
    </div>
  );
}
```

#### 3. "useUserSession": getting the current user session information

In case you need to access the current user session data, you can use the `useUserSession` hook.

This hook will return the current user session data, which includes the following information:
- `auth`: the data that comes from Supabase Auth
- `data`: the data that comes from the user's Supabase document

```tsx
export interface UserSession {
  auth: AuthUser | undefined;
  data: UserData | undefined;
}
```

For example, let's see how we can use this hook to get the current user ID:

```tsx
import { useUserSession } from '~/core/hooks/use-user-session';

function MyComponent() {
  const userSession = useUserSession();
  const userId = userSession?.auth?.uid;

  return (
    <div>
      <p>Current user ID: {userId}</p>
    </div>
  );
}
```

#### 4. "useIsSubscriptionActive": getting the current organization subscription status and checking if it's active

A subscription is considered active if it's either `active` or `trialing`. The `useIsSubscriptionActive` hook will return `true` if the current organization is on any paid subscription, regardless of plan.

This can be useful if you want to display a message to the user if they are on a paid subscription, or if you want to restrict access to certain pages.

You can customize this hook to fit your needs.

```tsx
import type { Stripe } from 'stripe';
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const ACTIVE_STATUSES: Stripe.Subscription.Status[] = ['active', 'trialing'];

/**
 * @name useIsSubscriptionActive
 * @description Returns whether the organization is on any paid
 * subscription, regardless of plan.
 */
function useIsSubscriptionActive() {
  const organization = useCurrentOrganization();
  const status = organization?.subscription?.status;

  if (!status) {
    return false;
  }

  return ACTIVE_STATUSES.includes(status);
}

export default useIsSubscriptionActive;
```

## Server-Side Functions

### 1. Supabase Server clients

Supabase provides different server clients depending on where they are used. This is because the cookie handling is usually different.

1. `getSupabaseServerComponentClient`: getting the server-side Supabase client for use in Server Components
2. `getSupabaseRouteHandlerClient`: getting the server-side Supabase client for use in API Route Handlers
3. `getSupabaseServerActionClient`: getting the server-side Supabase client for use in Server Actions

This function will return a server-side Supabase client, which you can use to interact with Supabase server-side.

```ts
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';

export function  GET() {
  const supabase = getSupabaseRouteHandlerClient();

  // Do something with the supabase client
}
```

if you want to use a Service Role key and use admin permissions, you can pass the `admin` option to the `getSupabaseRouteHandlerClient` function:

```ts
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';

export function  GET() {
  const supabase = getSupabaseRouteHandlerClient({
    admin: true
  });

  // Do something with the supabase client
}
```

If you want to use the client in a Server Component:

```tsx
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

function PageComponent() {
  const client = getSupabaseServerComponentClient();
}
```

If you want to use the Supabase client in a Server Action:

```tsx
'use server';

import getSupabaseServerActionClient from '~/core/supabase/server-action-client';

export async function serverAction() {
  const client = getSupabaseServerActionClient();
}
```

### 2. "requireSession": Getting (and requiring) the current user session

This function will return the current user session data, and will also require the user to be logged in.

- If the user is not logged in: they will be redirected to the sign-in page.
- If the user is signed in, but requires MFA: they will be redirected to the MFA verification page.

It returns a Supabase Auth `Session` instance:

```ts
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';

export async function GET() {
  const supabase = getSupabaseRouteHandlerClient();
  const session = await requireSession(supabase);

  //
}
```

Use it across layouts, route handlers, etc. every time you want to validate the user is signed in.

### 3. "getCurrentOrganization": getting the current user organization

This function returns the current user organization. It accepts two optional fields:
- `userId`: the user ID to get the organization for. If not provided, it will use the current user ID.
- `organizationId`: the organization ID to get. If not provided, it will use the current organization ID. If neither is found, it will fetch the first organization for the user.

And returns the organization data:

 ```tsx {% title="src/lib/organizations/database/queries.ts" %}
export type UserOrganizationData = {
  role: MembershipRole;
  organization: Organization & {
    subscription?: {
      customerId: Maybe<string>;
      data: OrganizationSubscription;
    };
  };
};
```

Usage:

```ts
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

export async function GET() {
  const response = await getCurrentOrganization();

  //
}
```

Using a user ID:

```ts
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

export async function GET() {
  const response = await getCurrentOrganization({
    userId: '123'
  });
}
```

Using an organization ID:

```ts
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

export async function GET() {
  const response = await getCurrentOrganization({
    userId: '123',
    organizationId: '456'
  });
}
```

-----------------
FILE PATH: kits\next-supabase\tutorials\initializing-project.mdoc

---
status: "published"
title: "Set up the Next.js Supabase SaaS Kit project"
label: Initial Setup
order: 1
description: "This section describes how to set up your development environment to use the Next.js Supabase SaaS template, from forking the repository to installing the Node modules"
---

You have two ways to initialize the project:

1. Forking it from GitHub, or
2. Cloning it using Git, or
3. Using the Makerkit CLI (coming soon!)

### 1. Forking the Repository

Fork this repository by clicking on the "Fork" button on the top-right
corner in GitHub for the repository you want to use. Please be aware that by forking the repository, the Makerkit organization has read-only access to your forked repository.

Once you have forked the repository, clone it locally. Then, set the
upstream repository to the original repository, so you can pull updates
when needed:

```
git remote add upstream git@github.com:makerkit/next-supabase-saas-kit.git
```

### 2. Cloning the Repository

Alternatively, **assuming you have accepted the invites and have access to the repository**, open your terminal and run this command (replace `tasks-app` with your name of choice):

```
git clone git@github.com:makerkit/next-supabase-saas-kit.git tasks-app
```

Once completed, we'll change into the `tasks-app` directory, and then we will install the Node modules:

```
cd tasks-app
npm i
```

### 3. Using the Makerkit CLI (coming soon!)

You can use the Makerkit CLI to create a new project based on this template. The CLI will ask you a few questions, and then it will create a new repository for you, and it will clone it locally.

Run the following command to run the CLI:

```
npx @makerkit/cli new
```

The CLI will ask you a few questions, and then it will create a new repository for you, and automatically install the Node modules.

### Reinitialize Git

As the Git repository's remote points to Makerkit's original repository, you can re-initialize (optionally!) the Git repository by running the following commands:

```
git remote rm origin
```

Now you have a clean Git repository.

### Setting the Upstream Repository

Then, we set the Makerkit repository as `upstream` - so we can pull updates when needed:

```
git remote add upstream git@github.com:makerkit/next-supabase-saas-kit.git
git add .
git commit -a -m "Initial Commit"
```

By adding the Makerkit's repository as `upstream` remote, you can fetch updates (after committing your files) by running the following command:

```
git pull upstream main --allow-unrelated-histories
```

If you have any conflicts, you can resolve them manually and then commit the changes. If the merge conflicts are too complex, accept your changes and discard the Makerkit's changes - then, you can manually merge the changes by looking at the diff in GitHub.

#### Remember to keep your forked repository up-to-date!

NB: as updates are released almost daily, remember to pull updates often!

---


Perfect! Now you can fire up your IDE and open the `tasks-app` project we just created.

-----------------
FILE PATH: kits\next-supabase\tutorials\introduction.mdoc

---
status: "published"

title: "Building a SaaS tasks application with Makerkit"
label: Introduction
order: 0
description: "Get a head start on your app development with our Makerkit SaaS boilerplate, built with Next.js and Supabase. Follow our step-by-step guide for an easy setup!"
---


This tutorial is a comprehensive guide to getting started with the Next.js and
Supabase SaaS template to build a basic tasks application, from fetching the repository to deploying
the application.

By the end, you'll have a fully working application with the minimum basics a SaaS needs:

1. Home Page and marketing pages (blog, pricing, FAQ)
2. Authentication (sign in, sign up, password reset)
3. Payments with Stripe
4. Profile Settings
5. Organization Settings
6. Tasks management with paginated Table and CRUD operations
7. Super Admin

## Prerequisites

To get started, you're going to need some things installed:

1. Git
2. Node.js version (LTS or greater)
3. npm 7 or greater
4. Docker - up and running
5. Your favorite code editor (VSCode, WebStorm, Zed, etc.)
6. To deploy your application, we recommend Vercel (but you can use any other provider)
7. A Stripe Account

Experience with React, TypeScript/JavaScript, and Supabase would be
advantageous but not strictly required. The codebase can also serve as a way to learn these topics more in-depth.

If you have all the above installed, let's get started!

-----------------
FILE PATH: kits\next-supabase\tutorials\marketing-site-pages.mdoc

---
status: "published"
title: "Adding pages to the Marketing Site | Next.js Supabase SaaS Kit"
label: "Adding pages to the Marketing Site"
order: 15
description: "Learn how to add pages to the Marketing Site of your Next.js Supabase application, and how to add them to the navigation menu."
---

Adding pages to the Marketing Site is easy.

The marketing/site pages are placed under the `app/(site)` directory, and are part of the `site` layout.

 ```tsx {% title="app/(site)/about/page.tsx" %}
import Hero from '~/core/ui/Hero';
import Container from '~/core/ui/Container';
import SubHeading from '~/core/ui/SubHeading';

export const metadata = {
  title: 'About',
};

const AboutPage = () => {
  return (
    <div>
      <Container>
        <div className={'flex flex-col space-y-14'}>
          <div className={'flex flex-col items-center'}>
            <Hero>About us</Hero>

            <SubHeading>
              We are a team of passionate developers and designers who love to
              build great products.
            </SubHeading>
          </div>
        </div>

        <div>
          <!-- Content -->
        </div>
      </Container>
    </div>
  );
};

export default AboutPage;
```

### Adding the page to the site's navigation menu

To update the site's navigation menu, you can visit the `SiteNavigation.tsx` component and add a new entry to the menu.

By default, you will see the following object `links`:

 ```tsx {% title="components/SiteNavigation.tsx" %}
const links: Record<string, Link> = {
  Blog: {
    label: 'Blog',
    path: '/blog',
  },
  Docs: {
    label: 'Docs',
    path: '/docs',
  },
  Pricing: {
    label: 'Pricing',
    path: '/pricing',
  },
  FAQ: {
    label: 'FAQ',
    path: '/faq',
  },
};
```

The menu is defined in the render function:

```tsx
<NavigationMenu>
  <NavigationMenuItem link={links.Blog} />
  <NavigationMenuItem link={links.Docs} />
  <NavigationMenuItem link={links.Pricing} />
  <NavigationMenuItem link={links.FAQ} />
</NavigationMenu>
```

Assuming we want to add a new menu entry, say `About`, we would first add the link object:

```tsx
About: {
  label: 'About',
  path: '/about',
},
```

And then we update the menu:

```tsx
<NavigationMenu>
  <NavigationMenuItem link={links.About} />
  ...
</NavigationMenu>
```

-----------------
FILE PATH: kits\next-supabase\tutorials\onboarding-flow.mdoc

---
status: "published"

title: "Onboarding Flow"
label: Onboarding Flow
order: 7
description: "Learn how to onboard your users to your app using the Onboarding Flow."
---


After sign-up, users are redirected to the onboarding flow.

Here, you can ask users questions, configure their accounts or simply show them around.

By default, Makerkit adds one single step, where it asks users to **create an organization**.

{% img src="/assets/images/posts/onboarding-flow-tutorial.webp" width="2940" height="1972" /%}

Of course, you can extend it and add as many steps as you wish. I would recommend replacing the image on the right-hand side with a screenshot of your application, a video, or something that can help users understand what they are signing up for.

### What happens when the user submits the form?

After the user submits the form, the API will receive the request and:

1. create the user Supabase record in the `users` table
2. create the organization Supabase record in the `organizations` table
3. create a membership between the user and the organization in the
`memberships` table and assign the user the role `owner`

I encourage you to visit the [Local Supabase Studio UI](http://localhost:54323) and see the data created.

### What is an "Organization"?

Organizations are groups of users.

You can call them projects, teams, classrooms, or whatever feels suitable for your domain. But, generally speaking, organizations are the backbone of the data model because it's where we store most of the data shared among users.

Users can:

1. create new Organizations
2. be invited to other Organizations
3. switch between organizations using a dropdown

### Can I remove this step?

No, **but you can skip it** by automatically submitting the form.

Yes, you can follow this guide for [removing the onboarding flow and organizations](/recipes/removing-organizations). It is for the Firebase version, but the same principles apply.

Alternatively - you can use the Lite Kit - which doesn't have an onboarding flow and organizations.

-----------------
FILE PATH: kits\next-supabase\tutorials\project-configuration.mdoc

---
status: "published"
title: "Project Configuration | Next.js Supabase"
label: Project Configuration
order: 4
description: "Learn how to set up up your project's configuration"
---

Makerkit can be configured from a single place: the `src/configuration.ts` object. This file exports a constant we use throughout the project to read our application's settings.

{% alert type="info" title="Use this file to configure your project" %}
  We recommend adding additional configuration to this file rather than reading it directly from the environment variables. For example, assuming you rename one of your environment variables, you will only need to update it in one place. Additionally, it helps maintain your codebase more explicit and understandable.
{% /alert %}

We retrieve some of this file's configuration using **environment
variables**: these help us swap and tweak our settings based on which environment we're using.

Here is what the configuration file looks like:

 ```tsx {% title="src/configuration.ts" %}
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

const production = process.env.NODE_ENV === 'production';

enum Themes {
  Light = 'light',
  Dark = 'dark',
}

const configuration = {
  site: {
    name: 'Awesomely - Your SaaS Title',
    description: 'Your SaaS Description',
    themeColor: '#ffffff',
    themeColorDark: '#0a0a0a',
    siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
  },
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
      oAuth: ['google'] as Provider[],
    },
  },
  production,
  environment: process.env.NEXT_PUBLIC_ENVIRONMENT,
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    signInMfa: '/auth/verify',
    onboarding: `/onboarding`,
    appPrefix: '/dashboard',
    appHome: '/dashboard',
    authCallback: '/auth/callback',
    settings: {
      profile: 'settings/profile',
      authentication: 'settings/profile/authentication',
      email: 'settings/profile/email',
      password: 'settings/profile/password',
    },
  },
  email: {
    host: 'smtp.ethereal.email',
    port: 587,
    user: 'fabian96@ethereal.email',
    password: 'puBhygwkTP7DtTSCSz',
    senderAddress: 'MakerKit Team <info@makerkit.dev>',
  },
  sentry: {
    dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  },
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
          },
          {
            name: 'Yearly',
            price: '$90',
            stripePriceId: '<price_id>',
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
            stripePriceId: '<price_id>',
          },
          {
            name: 'Yearly',
            price: '$200',
            stripePriceId: '<price_id>',
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
  },
};

export default configuration;
```

Feel free to extend this configuration to your needs. For example, you could new properties to the `site` object to store your social media handles, or add new properties to the `paths` object to store the paths to your pages.


-----------------
FILE PATH: kits\next-supabase\tutorials\routing.mdoc

---
status: "published"
title: "Routing | Next.js Supabase SaaS Kit"
label: "Routing"
order: 10
description: "Learn the basics of routing in the Next.js Supabase SaaS Boilerplate"
---

Before getting started with building our Tasks Pages, let's take a look at the default page structure of the boilerplate:

```txt
├── app
  └── layout.tsx

  └── onboarding

  (site)
    └── faq
    └── pricing

  └── auth
    └── link
    └── password-reset
    └── sign-in
    └── sign-up

  └── invite
    └── [code]
      └── page.tsx

  └── dashboard
    └── [organization]
      └── page.tsx

      └── settings
        └── organization
          └── members
            └── page.tsx
            └── invite
              └── page.tsx

        └── profile
          └── index
          └── email
          └── password
          └── authentication

        └── subscription

  └── page.tsx
```

The routes are split in the following way:

1. The website's pages are placed under `(site)`
2. The auth pages are placed under `auth`
3. The internal pages (behind auth) that are specific to organizations are placed under `dashboard/[organization]`
4. Some pages in the "middle" are placed outside `dashboard/[organization]`, such as the invites page and the onboarding flow.

### Setting the application's Home Page

By default, the application's home page is `/dashboard/[organization]`; every time the user logs in, they're redirected to the page `src/app/dashboard/[organization]/page.tsx`.

The path `dashboard` is also defined as `appPrefix` in the configuration file `src/configuration.tsx`: you can update this and rename the folder if you want to change the path.

To update the application's home page, you can update the `configuration.paths.appHome` variable in `src/configuration.tsx`.

### Demo: Tasks Application

Our demo application will be a simple tasks management application. We will be able to create tasks, list them, mark them as completed, and delete them.

Here is a quick demo of what we will build:

{% video src="/assets/images/posts/tasks-demo.mp4" /%}

### Routing

Ok, so we want to add three pages to our application:

1. **Tasks List**: A page to list all our tasks
2. **Task Detail**: A page specific to the selected task

To create these two pages, we will create a folder named `tasks` at `src/app/dashboard/[organization]/tasks`.

In the folder `src/app/dashboard/[organization]/tasks` we will create three `Page Components`:

1. The tasks list page at `tasks/page.tsx`
2. The task's own detail page at `tasks/[task]/page.tsx`.

```
├── app
  └── dashboard/[organization]
    └── tasks
      └── page.tsx

      └── [task]
        └── page.tsx
```

### Adding Functionalities to your application

To add new functionalities to your application (in our case, tasks management), usually, you'd need the following things:

1. **Data Model**: first, we want to define the data model of the feature you want to add
2. **Supabase Queries/Mutations**: once we're happy with the data model, we can create our Supabase DB queries/mutations
3. **Build Feature components**: then we can build the components of the feature (forms, lists, etc.)
4. **Add feature components into the pages**: finally, we add the components to the pages

### Adding page links to the Navigation Menu

#### Updating the Top header Navigation

To update the navigation menu, we need to update the `NAVIGATION_CONFIG` object in `src/navigation-config.tsx`. This object contains the links to the pages that are displayed in the top header menu.

1. The `label` is the text that will be displayed in the menu - it's a translation key that you can find in `locales/[lang]/common.json`
2. The `path` is the path to the page

```ts
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
      label: 'common:tasksTabLabel',
      path: getPath(organization, 'tasks'),
      Icon: ({ className }: { className: string }) => {
        return <RectangleStackIcon className={className} />;
      },
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

function getPath(organizationId: string, path: string) {
  const appPrefix = configuration.paths.appPrefix;

  return [appPrefix, organizationId, path].filter(Boolean).join('/');
}
```

To add a new link to the header menu, follow the same convention:

1. Add a new item to the `items` array
2. Add the `label` and `path` properties
3. Make sure the `label` is a translation key that you can find in `locales/[lang]/common.json`
4. Add the `Icon` property if you want to display an icon next to the label
5. Make sure the `path` is correct

The result will be similar to the images below:

{% img src="/assets/images/posts/sidebar-layout-link.webp" width="1280" height="984" /%}

---


In the next sections we will go into more detail about how to use and configure Application Pages.

