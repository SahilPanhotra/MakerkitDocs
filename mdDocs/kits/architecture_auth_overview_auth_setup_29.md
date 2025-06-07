-----------------
FILE PATH: kits/next-supabase-lite/architecture.mdoc

---
status: "published"
title: "Architecture in Next.js Lite Supabase"
label: "Architecture"
order: 8
description: "Architecture in Next.js Lite Supabase"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits/next-supabase-lite/authentication/auth-overview.mdoc

---
status: "published"

title: Setting up Supabase and Next.js authentication with MakerKit
label: Auth Overview
order: 0
description: 'MakerKit uses Supabase Authentication to allow access to your Next.js application using oAuth providers and email/password.'
canonical: '/docs/next-supabase/auth-overview'
---


MakerKit uses Supabase to manage authentication within your application.

By default, every kit comes with the following built-in authentication methods:
- **Email/Password** - we added, by default, the traditional way of signing in
- **Third Party Providers** - we also added by default Google Auth sign-in
- **Email Links**
- **Phone Number**

You're free to add (or remove) any of the methods supported by Supabase's
Authentication: we will see how.

This documentation will help you with the following:
 - **Setup** - setting up your Supabase project
 - **SSR** - use SSR to persist your users' authentication, adding new
providers
 - **Customization** - an overview of how MakerKit works so that you can adapt
it to your own application's needs

## Configuration

The way you want your users to authenticate can be driven via configuration.

If you open the global configuration at `src/configuration.ts`, you'll find
the `auth` object:

 ```tsx {% title="configuration.ts" %}
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

auth: {
  requireEmailConfirmation: false,
  providers: {
    emailPassword: true,
    phoneNumber: false,
    emailLink: false,
    oAuth: ['google'] as Provider[],
  },
}
```

As you can see, the `providers` object can be configured to only display the
auth methods we want to use.

1. For example, by setting both `phoneNumber` and `emailLink` to `true`, the
authentication pages will display the `Email Link` authentication
 and the `Phone Number` authentication forms.
2. Instead, by setting `emailPassword` to `false`, we will remove the
`email/password` form from the authentication and user profile pages.

{% alert type="warning" %}
Remember that you will always need to enable and configure the authentication
  methods you want to use in your Supabase project.
{% /alert %}

## Requiring Email Verification

This setting needs to match what you have set up in Supabase. If you require email confirmation before your users can sign in, you will have to flip the following flag in your configuration:

```ts
auth: {
  requireEmailConfirmation: false,
}
```

When the flag is set to `true`, the user will not be redirected to the onboarding flow, but will instead see a successful alert asking them to confirm their email. After confirmation, they will be able to sign in.

When the flag is set to `false`, the application will redirect them directly to the onboarding flow.

## Emails sent by Supabase

Supabase spins up an [InBucket](http://localhost:54324/) instance where all the emails are sent: this is where you can find emails related to password reset, sign-in links, and email verifications.

To access the InBucket instance, you can go to the following URL: [http://localhost:54324/](http://localhost:54324/). Save this URL, you will use it very often.

-----------------
FILE PATH: kits/next-supabase-lite/authentication/auth-setup.mdoc

---
status: "published"

title: Setting up Supabase authentication with MakerKit
label: Setup
order: 1
description: 'Learn how to setup Supabase authentication with MakerKit.'
---


Supabase needs a few settings to be configured in their Dashboard to work correctly. This guide will walk you through the steps to get your Supabase authentication setup.

{% alert type="warning" %}
Skipping this step will result in your users not being able to login or sign up.
{% /alert %}

## Authentication URLs

The first thing you need to do is to set the authentication URLs in the Supabase Dashboard. These URLs are used to redirect users to the correct page after they have logged in or signed up.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Authentication** tab.
4. Click on **URL Configuration**.
5. Add your Site URL to the **Site URL** field. This is the URL of your MakerKit site (e.g. `https://my-site.com`).
6. Add your Redirect URLs to the **Redirect URLs** field. You need to add at least two URLs: This is the URL of your MakerKit site with `/auth/callback` appended to it (e.g. `https://my-site.com/auth/callback`) and another for redirecting users to their password reset page (e.g. `https://my-site.com/settings/profile/password`).

### Troubleshooting

If you are having issues with authentication, ensure that the Site URL and Redirect URLs are correct. If you are using a custom domain, ensure that you are using the correct domain in the Site URL and Redirect URLs.

{% alert type="warning" %}
NB: if your domain includes "www", ensure you include it in the Site URL and Redirect URLs. If your domain does not
  include "www", ensure you do not include it in the Site URL and Redirect URLs. If these do not match, your users will
  not be able to login.
{% /alert %}

If something is still not working, please open a support ticket with any useful information (such as server logs).

## Custom SMTP (optional, but recommended)

If you want to send emails from your own domain, you can configure your SMTP settings in the Supabase Dashboard.

This is optional, but recommended if you want to send emails from your own domain.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Project Settings** tab.
4. Click on **Auth**.
5. Tweak the `SMTP Settings` settings to your liking according to your provider's documentation.

-----------------
FILE PATH: kits/next-supabase-lite/authentication.mdoc

---
status: "published"
title: "Authentication in Next.js Lite Supabase (legacy)"
label: "Authentication"
order: 4
description: "Learn how to set up authentication in your Next.js Lite Supabase application"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/blog-and-docs/blog.mdoc

---
status: "published"
title: Add Blog posts to your Next.js Supabase Lite application
label: Blog
order: 1
description: 'Learn how to add blog posts in your MakerKit Next.js Supabase Lite application'
---

We can add your website's blog posts within the `src/content/posts` folder.

## Add a new blog post

To add a new blog post, create a new markdown file within the `src/content/posts` folder. The file name should be the slug of the blog post. For example, if you want to create a blog post with the slug `hello-world`, create a file called `hello-world.mdx`.

For example, the following file is a blog post with the slug `lorem-ipsum`:

```mdx
---

title: Lorem Ipsum
date: 2023-09-23
live: true
description: "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."
image: "/assets/images/posts/lorem-ipsum.webp"
---

```

## Blog post metadata

Each blog post should have the following metadata:

- `title`: The title of the blog post
- `date`: The date of the blog post
- `live`: Whether the blog post should be visible on the website
- `description`: The description of the blog post
- `image`: The image of the blog post

## Blog post content

The content of the blog post should be written in markdown. You can use the [MDX](https://mdxjs.com/) syntax to add React components within the markdown content.

Additionally, we included multiple components that you can use to add content to your blog post:

- `Image`: Add an image to your blog post (a wrapper around the `next/image` component)
- `Video`: Add a video to your blog post. This is lazy-loaded and includes a loading spinner while the video is loading
- `Alert`: Add an alert to your blog post
- `LazyRender`: Lazy-load a React component within your blog post
- `TweetEmbed`: Embed a tweet within your blog post

-----------------
FILE PATH: kits/next-supabase-lite/blog-and-docs/documentation.mdoc

---
status: "published"
title: Add a Documentation website to your Next.js Supabase Lite application
label: Documentation
order: 2
description: 'Learn how to add documentation pages in your Next.js Supabase Lite application'
---

The Next.js Supabase Starter Kit comes with a documentation generator that you can use to create a documentation website for your SaaS application.

{% img src="/assets/images/posts/docs-index.webp" alt="Documentation Index Page" width="2796" height="1668" /%}

The documentation (like the blog) is built using the amazing [Contentlayer](https://contentlayer.dev). Take a look at the documentation if you're planning on extending the functionality of the documentation.

### Content Folder

Every kit now has a `content` folder, which contains the Markdown files for the documentation. The `content` folder is organized into sections, which are then rendered as pages on the documentation site.

Sections can be nested, so you can have a section with sub-sections. The sub-sections are then rendered as sub-pages.

### Initial Section page

Every section page can have an `index.mdx` file, which is the initial page for the section. This is useful if you want to have a landing page for the section, which then links to the sub-sections.

{% img src="/assets/images/posts/docs-section-page.webp" alt="Documentation Section Page" width="2818" height="1800" /%}

### Ordering

To organize the order of the sections, you shall prefix the folder name with a number, such as `01-getting-started`. The sections are then sorted alphabetically, so you can use numbers to control the order.

{% img src="/assets/images/posts/docs-ordering.webp" alt="Documentation Ordering" width="754" height="674" /%}

For example, the `content` folder for the Next.js Supabase kit looks like this:

```bash
- content
  - docs
    - 01-getting-started
      - index.mdx
      - 01-installation.mdx
      - 02-configuration.mdx
        - 01-supabase.mdx
        - 02-firebase.mdx
      - 03-authentication.mdx
      - 04-database.mdx
      - 05-storage.mdx
      - 06-usage.mdx
      - 07-deployment.mdx
        - 01-vercel.mdx
        - 02-netlify.mdx
      - 08-questions.mdx
    - 02-faq
      - index.mdx
      - 01-what-is-supabase.mdx
```

As you can see, we can use nested folders to organize the content. The `01-getting-started` folder is the first section, and it has an `index.mdx` file, which is the initial page for the section. The `02-configuration` folder is the second section, and it has a sub-section `01-supabase` and `02-firebase`.

### Frontmatter

Every Markdown file can have a frontmatter, which is a YAML block at the top of the file. The frontmatter is used to define the title of the page, and the description of the page.

For example, the `index.mdx` file for the `01-getting-started` section looks like this:

```mdx
---

title: "Getting Started with the Makerkit SaaS Starter Kits"
label: "Getting Started"
description: "Learn how to get started with the Makerkit SaaS Starter Kits"
---

```

The `title` is the title of the page, which is used for the `title` tag in the HTML code. The `label` is the label of the page, which is used in the sidebar navigation (where it's likely you will want a less SEO-focused label). The `description` is the description of the page, which is used in the meta tags.

### Content

The content of the Markdown file is written in Markdown, and is rendered as HTML. You can use Markdown to write the content of the page.

Just like the blog post, you can use the following components to render the content:

- `Image`: Add an image to your blog post (a wrapper around the `next/image` component)
- `Video`: Add a video to your blog post. This is lazy-loaded and includes a loading spinner while the video is loading
- `Alert`: Add an alert to your blog post
- `LazyRender`: Lazy-load a React component within your blog post
- `TweetEmbed`: Embed a tweet within your blog post

### Navigation

The navigation for the documentation is automatically generated based on the folder structure of the `content` folder. Furthermore, page links appear at the bottom of the page, so you can easily navigate to the previous or

-----------------
FILE PATH: kits/next-supabase-lite/blog-and-docs.mdoc

---
status: "published"
title: "Blog and Docs in Next.js Lite Supabase"
label: "Blog and Docs"
order: 8
description: "Blog and Docs in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/configuration/configuration.mdoc

---
status: "published"

title: "Define the global configuration of your MakerKit SaaS application
with Next.js and Supabase Lite"
label: "Global Configuration"
order: 0
description: "Learn how to define the global configuration of a MakerKit
application with Next.js and Supabase Lite"
canonical: '/docs/next-supabase/configuration'
---


We store the global configuration of a MakerKit application in `/configuration.ts`.

Within any application file, we can use the path `~/configuration` to
import it from any other file.

{% alert type="info" %}
You do not need any changes to start developing your application. Feel free
to complete the configuration once you want to deploy the app for the first
time.
{% /alert %}

The configuration has the following structure:

 ```typescript {% title="app/configuration.ts" %}
export default {
  site: {
    name: '',
    description: '',
    themeColor: '#ffffff',
    themeColorDark: '#0a0a0a',
    siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
    siteName: '',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
  },
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    emailLinkSignIn: '/auth/link',
    appHome: '/tasks',
    settings: {
      profile: '/settings/profile',
      authentication: '/settings/profile/authentication',
      email: '/settings/profile/email',
      password: '/settings/profile/password',
    },
  },
  auth: {
    requireEmailConfirmation:
      process.env.NEXT_PUBLIC_REQUIRE_EMAIL_CONFIRMATION === 'true',
    // NB: Enable the providers below in the Supabase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: ['google'],
    },
  },
  email: {
    host: '',
    port: 0,
    user: '',
    password: '',
  },
  environment: process.env.NEXT_PUBLIC_ENVIRONMENT,
  production: process.env.NODE_ENV === 'production',
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Light,
  stripe: {
    products: [
      {
        name: 'Basic',
        description: 'Describe your basic plan',
        plans: [
          {
            price: '$249/year',
            stripePriceId: '<STRIPE_PRICE_ID>',
          }
        ],
      },
      {
        name: 'Pro',
        description: 'Describe your pro plan',
        plans: [
          {
            price: '$249/year',
            stripePriceId: '<STRIPE_PRICE_ID>',
          }
        ],
      },
    ],
  },
};
```

These values are used throughout the application instead of being hardcoded into the codebase.

This file will not be committed (at least not the "secret" variable). So instead, you should define those variables within your favorite CI/CD.


-----------------
FILE PATH: kits/next-supabase-lite/configuration/environment-variables.mdoc

---
status: "published"

title: "Environment Variables"
label: Environment Variables
order: 2
description: "Learn how to use environment variables in your Next.js Supabase project."
---


The starter project comes with three different environment variables files:

1. **.env**: this is a shared environment file. Here, you can add variables shared across all the environments.
2. **.env.development**: the configuration file loaded when running the `dev` command
 - Use this file for your **development configuration**.
3. **.env.production**: the configuration file loaded when running the `build` command locally or deployed to Vercel.
 - Use this file for your **actual project's configuration** (without private keys!).
4. **.env.test**: this environment file is loaded when running the Cypress E2E tests. You would rarely need to use this.

Additionally, you can add another environment file named `.env.local`: this file is never added to Git and can be used to store the secrets you need during development.

{% alert type="warning" %}
<span className='block'>
    Never ever add secrets to your `.env` files. It's not safe, and you risk leaking confidential data.
  </span>

  <span>
    Instead, add your secrets using your Vercel Console, or use the `.env.local` file during development.
  </span>
{% /alert %}


## Environment Variables

Makerkit provides templates for configuring your environment variables correctly and the **minimum** environment variables to run your local environment.

To push your project to production, **you must fill these variables by creating your Supabase project** and adding the required values.

The production environment variables set your application up for the Supabase environment:

```
DEFAULT_LOCALE=en
SITE_URL=http://localhost:3000

# SUPABASE
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# STRIPE
STRIPE_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# EMAIL -
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

{% alert type="warning" title="You must add these variables to your production environment" %}

  <span>
    You can safely add public or non-secret variables to your <code>.env.production</code> file for your production environment. Instead, add your secrets using your Vercel Console, or use the <code>.env.local</code> file during development.
  </span>
{% /alert %}

#### Use your CI/CD to set environment variables for your production environment

Some of the variables above are secret. You must add these from your CI/CD environment:

```
# SUPABASE
SUPABASE_SERVICE_ROLE_KEY=

# STRIPE
STRIPE_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=

# EMAIL -
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

You can safely add public or non-secret variables to your `.env.production` file for your production environment.


## Website Configuration

The following variables are used to configure your website:

### DEFAULT_LOCALE

The default locale for your application. This is used by the `i18next` library to set the default locale for your application.

### SITE_URL

The URL of your application. This is used across the application to generate links to your application.

## Supabase Configuration

The following variables are used to configure your Supabase project:

### NEXT_PUBLIC_SUPABASE_URL

The URL of your Supabase project. This is used by the Supabase client to connect to your Supabase project. It is a public variable.

### NEXT_PUBLIC_SUPABASE_ANON_KEY

The anonymous key of your Supabase project. This is used by the Supabase client to connect to your Supabase project. It is a public variable.

### SUPABASE_SERVICE_ROLE_KEY

The service role key of your Supabase project. This is used by the Supabase client to connect to your Supabase project. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

## Stripe Configuration

The following variables are used to configure your Stripe project:

### STRIPE_WEBHOOK_SECRET

The webhook secret of your Stripe project (also referred to as Signing Secret within the Stripe dashboard). This is used by the Stripe client to connect to your Stripe project. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### STRIPE_SECRET_KEY

The secret key of your Stripe project. This is used by the Stripe client to connect to your Stripe project. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

The publishable key of your Stripe project. This is used by the Stripe client to connect to your Stripe project. It is a public variable and it's used to create the embedded checkout with the Stripe SDK.

## Email Configuration

The following variables are used to configure your email provider:

### EMAIL_HOST

The SMTP host of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_PORT

The SMTP port of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_USER

The SMTP user of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_PASSWORD

The SMTP password of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_SENDER

The sender of your emails. This is used by the email client to send emails to your users. It is a public variable. Use the format `Sender Name <email@app.com>`.

-----------------
FILE PATH: kits/next-supabase-lite/configuration.mdoc

---
status: "published"
title: "Configuration in Next.js Lite Supabase"
label: "Configuration"
order: 2
description: "Configuration in Next.js Lite Supabase"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits/next-supabase-lite/data-fetching/api-routes.mdoc

---
status: "published"

title: "API Routes"
label: "API Routes"
order: 4
description: "Learn how to create and manage API routes in your Next.js application."
---


To use API Routes in your Next.js Project, you can create a `route.ts` file within the `app` directory, which will be treated automatically as an API Route.

To export an handler, we need to export the HTTP verb we want to handle, such as `GET`, `POST`, `PUT`, etc.

 ```ts {% title="app/api/hello/route.ts" %}
import { NextRequest, NextResponse } from "next/server";

export async function GET(
  req: NextRequest
) {
  return NextResponse.json({ hello: "world" });
}
```

{% alert type="warning" %}
Don't confuse API Routes in the App Directory with API Routes in the Pages Directory.

  They are different and the signature doesn't match. Copy/pasting examples from the pages directory will not work.
{% /alert %}

## Calling API Routes

To call API Routes, you have two ways:
1. Use our utility function that wraps `fetch` and automatically inserts the CSRF token for you
2. Manually call the `fetch` function and insert the CSRF token yourself

The Middleware will reject every mutation request that does not add a CSRF token to the request. You can disable this, but it is not recommended.

### Using the Utility Function

By using the `useApiRequest` utility function, you can easily call API Routes from your application. In the example below, we use both the `useApiRequest` and `useSWR` hooks to fetch data from an API Route.

```tsx
import useSWRMutation from 'swr/mutation';

import configuration from '~/configuration';
import useApiRequest from '~/core/hooks/use-api';
import useRefresh from '~/core/hooks/use-refresh';

interface Params {
  membershipId: number;
}

const path = configuration.paths.api.organizations.transferOwnership;

function useTransferOrganizationOwnership() {
  const fetcher = useApiRequest<void, Params>();
  const refresh = useRefresh();
  const key = ['organizations', 'transfer-ownership'];

  return useSWRMutation(
    key,
    (_, { arg }: { arg: Params }) => {
      return fetcher({
        path,
        method: `PUT`,
        body: {
          membershipId: arg.membershipId,
        },
      });
    },
    {
      onSuccess: refresh,
    }
  );
}

export default useTransferOrganizationOwnership;
```

To use the `useTransferOrganizationOwnership` hook, we can simply call it from our component.

```tsx
function Component() {
  const { trigger } = useTransferOrganizationOwnership();

  // use trigger to call the mutation
}
```

Using `useSWRMutation` is not required, but it is recommended.

## CSRF Check

By default, the CSRF check is enabled. This means that every mutation request must include a CSRF token.

If you use the utility `useApiRequest` function, this is handled automatically for you. If you use `fetch` directly, you must add the CSRF token yourself.

To add the CSRF token, you can use the `useCsrfToken` hook.

```tsx
import useCsrfToken from '~/core/hooks/use-csrf-token';

function MyComponent() {
  const { csrfToken } = useCsrfToken();

  // use csrfToken in your fetch request

  return (
    <button onClick={() => {
      fetch('/api/my-route', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-CSRF-Token': csrfToken,
        },
      });
    }}>Click</button>
  );
}
```


-----------------
FILE PATH: kits/next-supabase-lite/data-fetching/fetching-data-supabase.mdoc

---
status: "published"

title: "Fetching data from Supabase"
label: "Fetching data from Supabase"
order: 1
description: "Learn how fetch data from your Supabase database in Makerkit and Next.js"
canonical: '/docs/next-supabase/data-fetching'
---


When fetching data from the Supabase Postgres Database, we use the Supabase
JS client.

To make data-fetching from Supabase more reusable, our convention is to
create a custom React hook for each query we make.

For example, let's take a look at the hook to fetch an organization from
Supabase.

First, we want to write a query that receives two parameters:
1. a Supabase client interface
2. an organization ID

 ```tsx {% title="organizations/queries.ts" %}
export function getOrganizationById(
  client: Client,
  organizationId: number
) {
  return client
    .from('organizations')
    .select(`
      id,
      name,
      logoURL: logo_url
    `)
    .eq('id', organizationId)
    .throwOnError()
    .single();
}
```

It's important that we pass the client as a parameter so that we can use the
function in both the browser and the server.

Now, if we want to use a React Hook to fetch the organization from one of
our components, we can use the `useSWR` hook from `swr`:

```tsx
import useSupabase from '~/core/hooks/use-supabase';
import { getOrganizationById } from '~/lib/organizations/database/queries';
import useSWR from 'swr';

function useOrganizationQuery(
  organizationId: number
) {
  const client = useSupabase();
  const key = ['organization', organizationId];

  return useSWR(key, async () => {
    return getOrganizationById(client, organizationId).then(
      (result) => result.data
    );
  });
}

export default useOrganizationQuery;
```
---


There are two ways to use this function:

1. **Server**: directly, in a server-side function such as: a Route Handler, a Server Action, a Server Component, or a Middleware
2. **Client**: indirectly, in a React Hook in a client-side component

### Retrieving data in a server-side function

Let's see some scenarios where we can use this function in a server-side.

#### Route Handler

When you are fetching data from a Route Handler, you can use the `getSupabaseRouteHandlerClient` function.

 ```tsx {% title="pages/api/organizations/[id]/route.ts" %}

import { getOrganizationById } from '~/lib/organizations/database/queries';
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';
import { NextRequest, NextResponse } from "next/server";

export const GET = async (
  req: NextRequest,
  { params }: { params: { id: string } }
) => {
  const client = getSupabaseRouteHandlerClient();
  const { data } = await getOrganizationById(client, params.id);

  return NextResponse.json(data);
};
```

### Server Action

When you are performing mutations using React Server Actions, you can use the `getSupabaseServerActionClient` function to read data from the Supabase database.

```tsx
'use server';

import getSupabaseServerActionClient from '~/core/supabase/action-client';
import { getOrganizationById } from '~/lib/organizations/database/queries';

export async function getOrganizationByIdAction(
  organizationId: number
) {
  const client = getSupabaseServerActionClient();
  const { data } = await getOrganizationById(client, organizationId);

  return data;
}
```

### Server Component

When you are fetching data from a Server Component, you can use the `getSupabaseServerComponentClient` function.

```tsx
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';
import { getOrganizationById } from '~/lib/organizations/database/queries';

interface Params {
  params: {
    id: string;
  };
}

async function OrganizationPage({ params }: Params) {
  const client = getSupabaseServerComponentClient();
  const organization = await getOrganizationById(client, params.id);

  return <div>{organization.name}</div>;
}
```

### Retrieving data using a React hook

Now that we have a hook to fetch an organization, we can use it in our
components to retrieve the data.

```tsx
import { useOrganizationQuery } from './use-organization-query';

function OrganizationCard({ organizationId }) {
  const {
    data: organization,
    isLoading,
    error
  } = useOrganizationQuery(organizationId);

  /* data is loading */
  if (isLoading) {
    return <div>Loading...</div>
  }

  /* request errored */
  if (error) {
    return <div>Error!</div>
  }

  /* request successful, we can access "organization" */
  return <div>{organization.name}</div>;
}
```


