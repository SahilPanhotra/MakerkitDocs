-----------------
FILE PATH: kits/next-supabase/how-to/api/api-routes-vs-server-actions.mdoc

---
status: "published"
title: "API Routes vs Server Actions | Next.js Supabase SaaS Kit"
label: "API Routes vs Server Actions"
order: 2
description: "API routes and Server actions can be used for similar use cases. This guide will help you understand the differences between the two."
---

API Routes and Server Actions can indeed be used in similar ways: ultimately, they both allow you to write server-side code that can be called from the client-side. However, let's look at the use-cases for each of them to understand the differences.

#### TLDR

TLDR; Use API Routes when fetching data, and Server Actions when performing actions/mutations.

## API Routes

API Routes are more generic and a lower-level abstraction than Server Actions. They are a way to write server-side code that can be called from the client-side. They are a great way to expose a REST API, or to write server-side code that can be called from the client-side.

The downside of API Routes is that they need to be called using the Fetch API, which you can use directly, or through a library such as React Query or SWR. This kit uses SWR.

Generally speaking, API Routes should be preferred when fetching data - rather than performing actions/mutations.

## Server Actions

Unlike API Routes, Server Actions are a higher-level abstraction. They are a way to write server-side code that can be called from the client-side as POST requests.

They are particularly useful when you want to perform actions/mutations on the server-side, such as creating a new user, or updating a user's profile.

Thanks to utilities such as `revalidatePath` and `revalidateTag`, you can also use Server Actions to invalidate the cache of a specific page or tag (used with `fetch`).

-----------------
FILE PATH: kits/next-supabase/how-to/api/api-routes.mdoc

---
status: "published"

title: "Adding API Routes in your Next.js application"
label: "Adding API Routes"
order: 0
description: "API Routes allow you to create server endpoints in your Next.js App that you can call from your client or from external sources"
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
FILE PATH: kits/next-supabase/how-to/api/call-api-routes-from-client.mdoc

---
status: "published"
title: "Calling API Routes from the client-side using SWR | Next.js Supabase SaaS Kit"
label: "Calling API Routes from the client"
order: 1
description: "Learn how to use SWR to call your API route handlers from React components in the  Next.js Supabase SaaS Kit project"
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


-----------------
FILE PATH: kits/next-supabase/how-to/app/adding-app-pages.mdoc

---
status: "published"

title: "Adding pages to the application of your Makerkit Next.js Supabase project"
label: Adding Pages
order: 1
description: "Learn how to add new pages to the application of your Makerkit Next.js Supabase project."
---


To add a new page to the application site of your Next.js project, you can create a new page in the `src/app/dashboard` directory.

By "application", we mean the private pages of your project behind the authentication wall, such as the dashboard, the settings page, etc.

The vast majority of the "private" pages of your application will be located in the `src/app/dashboard/[organization]` directory.
This directory contains the pages that are **only accessible to authenticated users**, and that are **related to a specific organization**.

### Layout and Sidebar

Pages created within the directory `src/app/dashboard/[organization]` will inherit the layout of the directory and will be displayed together with the Sidebar menu.

### Example - Creating a new page

For example, let's create a `Tasks` application located at `src/app/dashboard/[organization]/tasks`. We will then create a new page at `src/app/dashboard/[organization]/tasks/page.tsx`.

 ```tsx {% title="src/app/dashboard/[organization]/tasks/page.tsx" %}
import {
  RectangleStackIcon,
} from '@heroicons/react/24/outline';

import Trans from '~/core/ui/Trans';
import { withI18n } from '~/i18n/with-i18n';

import AppHeader from '~/app/dashboard/[organization]/components/AppHeader';
import AppContainer from '~/app/dashboard/[organization]/components/AppContainer';

interface TasksPageParams {
  params: {
    organization: string;
  };
}

// we will implement this function later
async function loadTasksData({ organizationUid }: { organizationUid: string}) {
  // ...
}

async function TasksPage({ params }: TasksPageParams) {
  const { tasks } = await loadTasksData({
    organizationUid: params.organization,
  });

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

export default withI18n(
  TasksPage
);
```

Let's break down what we did here:

1. We created a new page at `src/app/dashboard/[organization]/tasks/page.tsx`. This page will be accessible at `/dashboard/[organization]/tasks`.
2. We created a `loadTasksData` function that will be used to load the data of the page. This is only a mock function for now, we will implement it later.
3. We created a `TasksPage` component that will be used to render the page. This component is a Next.js Server Page component, so it must be exported as default.
4. We wrapped the `TasksPage` component with the `withI18n` HOC. This HOC will ensure translations are loaded and available in the component.

### Translations

Always add the `withI18n` HOC to your pages to ensure translations are loaded and available in the component when using Server Components (pages or layouts).

### Layout

Layout-wise, we added the following components:

1. An `AppHeader` component that will be used to render the header of the page. This component is located at `src/app/dashboard/[organization]/components/AppHeader.tsx`.
2. An `AppContainer` component that will be used to render the content of the page. This component is located at `src/app/dashboard/[organization]/components/AppContainer.tsx`.

You can omit these components if you don't need them.

### Page Parameters

Because the `TasksPage` component is located at `src/app/dashboard/[organization]/tasks/page.tsx`, it will receive the following parameters:

```tsx
interface TasksPageParams {
  params: {
    organization: string;
  };
}
```

This means we can access the parameter `organization` in the `TasksPage` component:

```tsx
async function TasksPage({ params }: TasksPageParams) {
  const { tasks } = await loadTasksData({
    organizationUid: params.organization,
  });

  return (
    <>
      {/* ... */}
    </>
  );
}
```

The `organization` parameter is the organization UID of the organization the user is currently viewing. You can use this parameter to fetch the data related to the organization.

## Building pages outside of the organization context

If you need to build a page that is not related to a specific organization, you can create it in the `src/app/dashboard` directory.

-----------------
FILE PATH: kits/next-supabase/how-to/app/guarding-pages.mdoc

---
status: "published"
title: "Guarding Pages | Next.js Supabase SaaS Kit"
label: "Guarding Pages"
order: 4
description: "Learn how to guard pages in your Next.js Supabase application"
---

As a server-side rendered application, Next.js Supabase will always render the page on the server before sending it to the client.

**We assume you are in the context of a specific organization, as it is the primary use case for the Starter Kit.**

We can use the function `loadAppData` to get data required to load the main layout of the app located at `app/dashboard/[organization]`.

NB: you can call `loadAppData` multiple times because it uses caching.

In the following example, we are loading the user role within the organization **that is currently selected**.

```tsx
import { redirect } from "next/navigation";

import loadAppData from '~/lib/server/loaders/load-app-data';
import MembershipRole from '~/lib/organizations/types/membership-role';

async function OnlyOwnersPage() {
  const data = await loadAppData();
  const userRole = data.role;

  // if the user is not an owner, redirect them to the home page
  if (userRole !== MembershipRole.Owner) {
    redirect('/');
  }

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
import { redirect } from "next/navigation";
import loadAppData from '~/lib/server/loaders/load-app-data';

async function OnlySubscribersPage() {
  const data = await loadAppData();
  const status = data.organization.subscription?.data?.status;

  // if the subscription is not active, redirect the user to the dashboard
  if (!isSubscriptionActive(status)) {
    redirect('/');
  }

  // render the page
}

function isSubscriptionActive(status: string | undefined) {
  return ['trialing', 'active'].includes(status);
}
```

-----------------
FILE PATH: kits/next-supabase/how-to/app/updating-sidebar-menu.mdoc

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

const NAVIGATION_CONFIG = (organization: string) => ({
  items: [
    {
      label: 'common:dashboardTabLabel',
      path: getPath(organization, configuration.paths.appHome),
      Icon: ({ className }: { className: string }) => {
        return <Squares2X2Icon className={className} />;
      },
    },
    {
      label: 'common:tasksTabLabel',
      path: getPath(organization, 'tasks'),
      Icon: ({ className }: { className: string }) => {
        return <Square3Stack3DIcon className={className} />;
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
FILE PATH: kits/next-supabase/how-to/authentication/change-auth-strategy.mdoc

---
status: "published"
title: "How to change Authentication strategy | Next.js Supabase SaaS Kit"
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
FILE PATH: kits/next-supabase/how-to/authentication/setup-oauth.mdoc

---
status: "published"
title: "How to setup oAuth in the Next.js Supabase Starter"
label: Setup oAuth
order: 1
description: "Learn how to setup oAuth in the Next.js Supabase Starter"
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
FILE PATH: kits/next-supabase/how-to/contextual-data/organization.mdoc

---
status: "published"

title: "Fetching the selected Organization"
label:  "Fetching the selected Organization"
order: 1
description: "Learn how to fetch the selected organization from the backend and frontend."
---


There are several ways to fetch the current selected organization, depending on the use-case.

## Getting the selected Organization ID

Getting the selected organization ID is done using different ways depending on the context.

1. If you are in a Server Component under the `[organization]` route, the `organizationUid` is passed as a prop to the component. You can match the `organizationUid` with the `uuid` property of the organization.
2. If you're not, use the cookie utility `parseOrganziationCookie` to get the `organizationUid` from the cookie. This is scoped by user, so you will need to pass the `userId` as well.

### Using "parseOrganizationIdCookie" to get the Organization ID

You can use the `parseOrganizationIdCookie` function in:

- API Routes
- Server Actions
- Server Components

```tsx
import getSupabaseServerActionClient from '~/core/supabase/action-client';
import { parseOrganizationIdCookie } from '~/lib/server/cookies/organization.cookie';
import requireSession from '~/lib/user/require-session';

async function getOrganizationId() {
  const client = getSupabaseServerActionClient();
  const session = await requireSession(client);

  return parseOrganizationIdCookie(session.user.id);
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
const client = getSupabaseRouteHandlerClient();
const session = await requireSession(client);
const userId = session.user.id;

const organizationResponse = await getCurrentOrganization({
  organizationUid,
  userId,
});
```

## Fetching the selected Organization from the frontend

Within the `[organization]` context, the layout loader passes the current organization is passed from the backend to the frontend using the Context API. Therefore, we can require it using the `useCurrentOrganization` hook.

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




