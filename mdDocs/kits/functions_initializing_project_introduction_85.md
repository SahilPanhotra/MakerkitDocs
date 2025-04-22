-----------------
FILE PATH: kits\remix-supabase\tutorials\functions.mdoc

---
status: "published"

title: "Functions you need to know"
label: "Functions you need to know"
order: 17
description: "Learn the most important functions you need to know to use the Remix Supabase kit effectively."
---


This is an extremely important section of the documentation: it will teach you the most important functions you need to know to use the Remix Supabase kit effectively.

We will cover both client-side and server-side functions.

## Client-Side Functions

### Supabase Hooks

The Remix Supabase kit uses the client Supabase SDK extensively throughout the
kit. You can find the full documentation of the Supabase SDK [here](https://supabase.io/docs/reference/javascript/supabase-client).

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

This hook will allow you to get the current user organization. It is populated with the data fetched server-side in the `(app)` layout, and is injected using the Context API.

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

### 1. "getSupabaseServerClient": getting the server-side Supabase client

This function will return a server-side Supabase client, which you can use to interact with Supabase server-side.

```ts
import getSupabaseServerClient from '~/core/supabase/server-client';

export function  GET() {
  const supabase = getSupabaseServerClient();

  // Do something with the supabase client
}
```

if you want to use a Service Role key and use admin permissions, you can pass the `admin` option to the `getSupabaseServerClient` function:

```ts
import getSupabaseServerClient from '~/core/supabase/server-client';

export function  GET() {
  const supabase = getSupabaseServerClient({
    admin: true
  });

  // Do something with the supabase client
}
```

### 2. "requireSession": Getting (and requiring) the current user session

This function will return the current user session data, and will also require the user to be logged in.

- If the user is not logged in: they will be redirected to the sign-in page.
- If the user is signed in, but requires MFA: they will be redirected to the MFA verification page.

It returns a Supabase Auth `Session` instance:

```ts
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function GET() {
  const supabase = getSupabaseServerClient();
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
  const supabase = getSupabaseServerClient();
  const response = await getCurrentOrganization(supabase);

  //
}
```

Using a user ID:

```ts
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

export async function GET() {
  const supabase = getSupabaseServerClient();
  const response = await getCurrentOrganization(supabase, {
    userId: '123'
  });
}
```

Using an organization ID:

```ts
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

export async function GET() {
  const supabase = getSupabaseServerClient();
  const response = await getCurrentOrganization(supabase, {
    userId: '123',
    organizationId: '456'
  });
}
```

-----------------
FILE PATH: kits\remix-supabase\tutorials\initializing-project.mdoc

---
status: "published"
title: "Setting up the Remix Supabase Kit project"
label: Initial Setup
order: 1
description: "This section describes how to set up your development environment to use the Remix Supabase SaaS template, from forking the repository to installing the Node modules"
---

## Initializing the Project

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
git remote add upstream git@github.com:makerkit/remix-supabase-saas-kit.git
```

### 2. Cloning the Repository

Alternatively, assuming you have accepted the invites and have access to the repository, open your terminal and run this command (replace `tasks-app` with your name of choice):

```
git clone git@github.com:makerkit/remix-supabase-saas-kit.git tasks-app
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
git remote add upstream git@github.com:makerkit/remix-supabase-saas-kit.git
git add .
git commit -a -m "Initial Commit"
```

By adding the Makerkit's repository as `upstream` remote, you can fetch updates (after committing your files) by running the following command:

```
git pull upstream main --allow-unrelated-histories
```

If you have any conflicts, you can resolve them manually and then commit the changes. If the merge conflicts are too complex, accept your changes and discard the Makerkit's changes - then, you can manually merge the changes by looking at the diff in GitHub.

#### Remember to keep your forked repository up-to-date!

As updates are released almost daily, remember to pull updates often!

---


Perfect! Now you can fire up your IDE and open the `tasks-app` project we just created.

-----------------
FILE PATH: kits\remix-supabase\tutorials\introduction.mdoc

---
status: "published"

title: "Building a SaaStasks application with Makerkit"
label: Introduction
order: 0
description: "Get a head start on your app development with our Makerkit SaaS boilerplate, built with Remix and Supabase. Follow our step-by-step guide for an easy setup!"
---


This tutorial is a comprehensive guide to getting started with the Remix and
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
FILE PATH: kits\remix-supabase\tutorials\marketing-site-pages.mdoc

---
status: "published"

title: "Adding pages to the Marketing Site"
label: "Adding pages to the Marketing Site"
order: 16
description: "Learn how to add pages to the Marketing Site of your Remix Supabase application, and how to add them to the navigation menu."
---


Adding pages to the Marketing Site is easy.

The marketing/site pages are placed under the prefix `_site`
directory, and are part of the `site` layout.

 ```tsx {% title="app/routes/_site.about.tsx" %}
import type { MetaFunction } from '@remix-run/node';

import configuration from '~/configuration';
import Hero from '~/core/ui/Hero';
import Container from '~/core/ui/Container';
import SubHeading from '~/core/ui/SubHeading';

export const meta: MetaFunction = () => {
  return {
    title: `About - ${configuration.site.siteName}`,
  };
};

const About = () => {
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

          <div>
            <!-- Add your content here -->
          </div>
        </div>
      </Container>
    </div>
  );
};

export default About;
```

### Adding pages to the navigation menu

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
FILE PATH: kits\remix-supabase\tutorials\onboarding-flow.mdoc

---
status: "published"
title: "Onboarding Flow | Remix Supabase SaaS Kit"
label: Onboarding Flow
order: 8
description: "Learn how to onboard your users to your app using the Onboarding Flow in the Remix Supabase SaaS Kit"
---

After sign-up, users are redirected to the onboarding flow. Here, you can ask users questions, configure their accounts or simply show them around.

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


-----------------
FILE PATH: kits\remix-supabase\tutorials\project-configuration.mdoc

---
status: "published"
title: "Project Configuration | Remix Supabase"
label: Project Configuration
order: 4
description: "Learn how to set up up your Remix Supabase project's configuration"
---

Makerkit can be configured from a single place: the `src/configuration.ts` object. This file exports a constant we use throughout the project to read our application's settings.

{% alert type="info" title="Use this file to configure your project" %}

  We recommend adding additional configuration to this file rather than reading it directly from the environment variables. For example, assuming you rename one of your environment variables, you will only need to update it in one place. Additionally, it helps maintain your codebase more explicit and understandable.
{% /alert %}

We retrieve some of this file's configuration using **environment
variables**: these help us swap and tweak our settings based on which environment we're using.

Here is what the configuration file looks like:

 ```tsx {% title="src/configuration.ts" %}
import getEnv from '~/core/get-env';
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

const env = getEnv() ?? {};
const production = env.NODE_ENV === 'production';

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
    siteUrl: env.SITE_URL,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: env.DEFAULT_LOCALE,
  },
  auth: {
    // ensure this is the same as your Supabase project
    requireEmailConfirmation: true,
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
  environment: env.ENVIRONMENT,
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    signInMfa: '/auth/verify',
    signInFromLink: '/auth/link',
    callback: '/auth/callback',
    onboarding: `/onboarding`,
    appHome: '/dashboard',
    settings: {
      profile: '/settings/profile',
      authentication: '/settings/profile/authentication',
      email: '/settings/profile/email',
      password: '/settings/profile/password',
    },
    api: {
      checkout: `/resources/stripe/checkout`,
      billingPortal: `/resources/stripe/portal`,
      organizations: {
        create: `/resources/organizations/create`,
        transferOwnership: `/resources/organizations/transfer-ownership`,
        members: `/resources/organizations/members`,
      },
    },
  },
  email: {
    host: 'smtp.ethereal.email',
    port: 587,
    user: 'gracie.hand58@ethereal.email',
    password: '5hr3DDGGc2bTbu7tJR',
    senderAddress: 'MakerKit Team <info@makerkit.dev>',
  },
  sentry: {
    dsn: env.SENTRY_DSN,
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
FILE PATH: kits\remix-supabase\tutorials\project-structure.mdoc

---
status: "published"
title: "Project Structure | Remix Supabase SaaS Kit"
label: Project Structure
order: 2
description: "A quick overview of the project structure and how to navigate it in Remix Supabase SaaS Kit."
---

When inspecting the project structure, you will find something similar to the below:

```
tasks-app
├── README.md
├── @types
├── src
│   ├── components
│   ├── core
│   ├── lib
│   └── routes
        ...
│       └── root.tsx
├── package-lock.json
├── package.json
├── public
│   └── favicon.ico
└── tsconfig.json
```

NB: we omitted a lot of the files for simplicity.

Let's take a deeper look at the structure we see above:

- `src/core`: this folder contains reusable building blocks of the template that **are not related to any domain**. This includes components, React hooks, functions, and so on.
- `src/components`: this folder contains the application's components, including those that belong to your application's domain. For example, we can add the `tasks` components to `src/components/tasks`.
- `src/lib`: this folder contains the business logic of your application's
domain. For example, we can add the `tasks` hooks to write and fetch data
from Supabase in `src/lib/tasks`.
- `src/routes`: this is Remix folder where we add our application
routes.

Don't worry if this isn't clear yet! We'll explain where we place our files in the sections ahead.

-----------------
FILE PATH: kits\remix-supabase\tutorials\routing.mdoc

---
status: "published"
title: "Routing | Remix Supabase SaaS Kit"
label: "Routing"
order: 9
description: "Learn the basics of routing in the Remix Supabase SaaS Boilerplate"
---

It's time to work on our application's value proorder: adding and tracking tasks! This is likely the most exciting part for you because **it's where you get to change things and add your SaaS features to the template**.

### Routing Structure

Before getting started, let's take a look at the default page structure of the boilerplate is the following.

{% alert type="info" %}
The routing structure below has been updated to using the new Remix v2 convention
{% /alert %}

```txt {16-17}
├── routes
  └── $.tsx
  └── _app.dashboard.tsx
  └── _app.settings._index.tsx
  └── _app.settings.organization._index.tsx
  └── _app.settings.organization.members._index.tsx
  └── _app.settings.organization.members.invite._index.tsx
  └── _app.settings.organization.tsx
  └── _app.settings.profile._index.tsx
  └── _app.settings.profile.authentication.tsx
  └── _app.settings.profile.email.tsx
  └── _app.settings.profile.password.tsx
  └── _app.settings.profile.tsx
  └── _app.settings.subscription._index.tsx
  └── _app.settings.tsx
  └── _app.tasks.$task.tsx
  └── _app.tasks._index.tsx
  └── _app.tsx
  └── _invite.tsx
  └── _site._index.tsx
  └── _site.about.tsx
  └── _site.faq.tsx
  └── _site.pricing.tsx
  └── _site.tsx
  └── admin._index.tsx
  └── admin.organizations.$id.members.tsx
  └── admin.organizations._index.tsx
  └── admin.tsx
  └── admin.users.$id.ban.ts
  └── admin.users.$id.impersonate.tsx
  └── admin.users.$id.reactivate.ts
  └── admin.users.$id.tsx
  └── admin.users._index.tsx
  └── auth.callback.tsx
  └── auth.link.tsx
  └── auth.password-reset.tsx
  └── auth.sign-in.tsx
  └── auth.sign-up.tsx
  └── auth.tsx
  └── auth.verify.tsx
  └── healthcheck.ts
  └── invite.$code.tsx
  └── onboarding._index.tsx
  └── resources.organizations.create.ts
  └── resources.organizations.members.$member.tsx
  └── resources.organizations.transfer-ownership.ts
  └── resources.stripe.checkout.tsx
  └── resources.stripe.portal.ts
  └── resources.stripe.webhook.ts
```

The routes are split in the following way:

1. The website's pages are placed under the prefix `_site`
2. The auth pages are placed under the prefix `_auth`
3. The internal pages (behind auth) pages are placed under the prefix `_app`

Some pages in the "middle" are placed outside `_app`, such as the Invites page and the Onboarding flow. These require custom handling.

### Setting the application's Home Page

By default, the application's home page is `/dashboard`; every time the user
logs in, they're redirected to the page `src/routes/_app.dashboard._index.tsx`.

You can update the above by setting the application's home page path at `configuration.paths.appHome`.

### Routing

Ok, so we want to add two pages to our application:

1. **Tasks List**: A page to list all our tasks
2. **Task Detail**: A page specific to the selected task

1. **List Page**: we create a page `page.tsx`, which is accessible at the path `/tasks`
3. **Task Page**: we create a page `$task.tsx`, which is accessible at the path
`/tasks/<taskID>` where `taskID` is a dynamic variable that refers to the actual ID of the task

```txt
├── routes
  └── _app.tasks._index.tsx
  └── _app.tasks.$task.tsx
```

### Adding Functionalities to your application

To add new functionalities to your application (in our case, tasks management), usually, you'd need the following things:
1. First, we want to define our data model
2. Once we're happy with the data model, we can create our Supabase hooks to
write new tasks and then fetch the ones we created
3. Then, we import and use our hooks within the components
4. Finally, we add the components to the pages

### Adding page links to the Navigation Menu

#### Updating the Top header Navigation

To update the navigation menu, we need to update the `NAVIGATION_CONFIG`
object in `src/navigation-config.tsx`.

```ts
import { Squares2X2Icon, Cog8ToothIcon } from "@heroicons/react/24/outline";

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
      label: 'common:settingsTabLabel',
      path: '/settings',
      Icon: ({ className }: { className: string }) => {
        return <Cog8ToothIcon className={className} />;
      },
    },
  ],
};
```

To add a new link to the header menu, we can add the following item in the `NAVIGATION_CONFIG` object:

```tsx
import { Squares2X2Icon } from "@heroicons/react/24/outline";

{
  label: 'common:tasksTabLabel',
  path: '/tasks',
  Icon: ({ className }: { className: string }) => {
    return <Squares2X2Icon className={className} />;
  },
},
```

The result will be similar to the images below:

{% img src="/assets/images/posts/sidebar-layout-link.webp" width="1280" height="984" /%}

{% alert type="warning" %}
You may need to restart your development server to see the changes, and/or refresh the browser's cache for the changes to be visible.
{% /alert %}

-----------------
FILE PATH: kits\remix-supabase\tutorials\running-project.mdoc

---
status: "published"

title: "Running the App"
label: Running the App
order: 3
description: "Learn how to run the app locally for development and testing."
---


## Running the project

When we run the stack of a Makerkit application, we will need to run a few commands before:
1. **Remix**: First, we need to run the Remix development server
2. **Supabase**: We need to run the Supabase local environment using Docker
3. **Stripe CLI** (optional): If you plan on interacting with Stripe, you are also required to run the Stripe CLI, which allows us to test the webhooks from Stripe against our local server.

### Why do we need to run the Supabase local environment?

The guide below will show you how to run the application locally. But why locally? Developing locally has many advantages over developing on a remote server:

**Speedy Development:** Local development lets you work without the hindrance of network latency or internet disruptions.

**Simplified Collaboration:** It's typically easier to collaborate with teammates when developing locally on the same project.

**Cost Efficiency:** Supabase offers a generous free tier, including two free projects. However, if you need more than two, local development comes into play. You can create countless local projects and establish links with live projects at your convenience for launch.

**Code-Based Configuration:** Any modifications to your tables via the Dashboard aren't reflected in code. Adopting local development workflows allows you to maintain all table schemas in code.

Once you are ready to deploy your application, you can do so by following
the [production checklist guide](/docs/remix-supabase/going-to-production-overview).

Fore more information, check out the [Supabase documentation](https://supabase.com/docs/guides/cli/local-development).

### Adding the environment variables file

The kit comes with a template of what your `.env` file should look like named `.env.template`.

**Before you continue**: rename `.env.template` to `.env`, or copy its contents to `.env`.

**This file won't be committed to git**. When you deploy your production app, ensure you add the environment variables using your CI/Service.

NB: Remix does not use the `.env` file when bundling the application in production mode.

### Building the Content using Contentlayer

Before running the application, we need to build the content using Contentlayer. To do so, run the following command:

```
npm run contentlayer:build
```

### Running the Remix development server

Run the following command from your IDE or from your terminal:

```
npm run dev
```

This command will start the Remix server at [localhost:3000](http://localhost:3000). If you navigate to this page, you should be able to see the landing page of your application:

{% img src="/assets/images/posts/tutorial-landing-page.webp" width="2856" height="1972" /%}

### Running the Supabase Local environment

To run the Supabase local environment, we need to first run Docker. Then,
open a new terminal (or, better, from your IDE) and run the following
command:s

```
npm run supabase:start
```

If everything is working correctly, you will see the output below:

```
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

When you need to stop the development environment, you can run the following command:

```
npm run supabase:stop
```

#### Supabase Studio UI

To access the Supabase Studio UI, open a new tab in your browser and navigate to [http://localhost:54323](http://localhost:54323).

{% alert type="info" %}
Makerkit's template adds some data to your project by default for testing reasons. In fact, the data you see is used to run the Cypress E2E tests. This is why you'll see some pre-populated data in your project.
{% /alert %}

### Running the Stripe CLI (optional)

Optionally, if you want to run Stripe locally (e.g., sending webhooks to your local server), we need two more steps.

1. Have a Stripe account
2. Have Docker installed and running
3. Run the Stripe CLI (or install Stripe on your OS)

To run the CLI, run the following command:

```
npm run stripe:listen
```

When running the command above for the first time, **Stripe will ask you to login**. You can do so by following the instructions in the terminal.

After signing in, run the command again.

```
npm run stripe:listen
```

The above command runs the Stripe CLI and will route webhooks coming from Stripe to your local endpoint. For example, in the Makerkit starter, this endpoint is `/resources/stripe/webhook`.

When running the command, it will print a webhook key used to sign the messages from Stripe. You will need to add this key to your local environment variables file as below:

```
STRIPE_WEBHOOK_SECRET=<KEY>
```

The webhook printed should not change, so you may only need to do this the first time.

### Signing In for the first time

You should now be able to sign in. To quickly get started, use the following credentials:

```
email = test@makerkit.dev
password = testingpassword
```

### Email Confirmations for Testing with InBucket

When signing up, Supabase sends an email confirmation to a testing account.

You can access the InBucket testing emails [at http://localhost:54324/monitor](http://localhost:54324/monitor), and can follow the links to complete the sign up process.

InBucket is started automatically when you run `npm run supabase:start`.

