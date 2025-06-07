-----------------
FILE PATH: kits/next-fire/how-to/api/call-api-routes-from-client.mdoc

---
status: "published"
title: "Calling API Routes from the client-side using SWR | Next.js Firebase SaaS Kit"
label: "Calling API Routes from the client"
order: 2
description: "Learn how to use SWR to call your Next.js Firebase API route handlers from React components"
---

Makerkit includes the library SWR, a data-fetching React utility written by Vercel, the creators of Next.js.

This library is useful for making asynchronous requests from your React components - primarily to API Routes.

Why is it useful? SWR makes it simpler to fetch and mutate data, caching and revalidating. Let's see how we can use it to make requests to our endpoints.

## Using SWR for fetching data from an API Route Handler

Let's assume we have a very simple API Route at `/api/data`:

 ```tsx {% title="app/api/data.ts" %}
import { NextApiRequest, NextApiResponse } from "next";

export default async function apiHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  return res.json({
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

Let's write a very simple API Route that makes a mutation: we add a new record in the `tasks` collection, and return the ID back to the client.

```tsx
import { NextApiRequest, NextApiResponse } from "next";

import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';
import { withPipe } from '~/core/middleware/with-pipe';
import { withCsrf } from '~/core/middleware/with-csrf';
import { withAuthedUser } from '~/core/middleware/with-authed-user';

async function apiHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const firestore = getRestFirestore();
  const collection = firestore.collection('tasks');

  const data = await collection.add({
    name: req.body.name
  });

  return res.json({ id: data.id });
}

export default withPipe(
  withAuthedUser,
  withCsrf(),
  apiHandler,
);
```

Now, let's write a React Hook that calls this endpoint using SWR.

```tsx
import useMutation from 'swr/mutation';
import { useApiRequest } from '~/core/hooks/use-api';

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
import { useCsrfToken } from "~/core/firebase/hooks/use-csrf-token";

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
FILE PATH: kits/next-fire/how-to/api/csrf-token.mdoc

---
status: "published"

title: "Checking CSRF Tokens"
label: "Checking CSRF Tokens"
order: 3
description: "How to check CSRF tokens in your Next.js Firebase API routes using the \"withCsrf\" HOC."
---

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

async function helloWorldHandler(
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

-----------------
FILE PATH: kits/next-fire/how-to/api/guarding-api-routes.mdoc

---
status: "published"

title: "Guarding an API Route"
label: "Guarding an API Route"
order: 1
description: ""
---

It's very likely that you want to ensure only authenticated users can access your API routes. To do that, you can use the `withAuth` HOC to wrap your API route handler.

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

## Guarding an API Route with a Custom Middleware

If you want to use a custom middleware to guard your API route, you can do so fairly easily by using a function that accepts a handler function and returns a handler function.

### Example: Guarding an API Route based on the User's Role

Let's say we want to create a function that guards an API route based on the user's role. We can do that by creating a function that accepts a role as an argument, and returns a handler function that checks the user's role before executing the handler function.

**Important**: this middleware requires that the `withAuthedUser` middleware is used before it - otherwise, the `req.firebaseUser` object will not be available.

 ```ts {% title="core/middleware/with-role.ts" %}
import { NextApiRequest,NextApiResponse } from "next";
import { MembershipRole } from '~/lib/organizations/types/membership-role';
import { getCurrentOrganization } from '~/lib/server/organizations/get-current-organization';

export function withRole(role: MembershipRole) {
  return async function(req: NextApiRequest, res: NextApiResponse) {
    const userId = req.firebaseUser.uid;
    const currentOrganizationId = req.cookies.organizationId;

    const organization =
      await getCurrentOrganization(userId, currentOrganizationId);

    const currentUserRole = organization.members[userId].role;

    if (currentUserRole !== role) {
      return res.status(403).json({
        error: 'You do not have permission to access this resource.'
      });
    }
  };
}
```

Now, we can use this middleware in our API route handler.

 ```ts {% title="pages/api/hello.ts" %}
import { NextApiRequest, NextApiResponse } from "next";
import { MembershipRole } from '~/lib/organizations/types/membership-role';

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withRole } from '~/core/middleware/with-role';
import { withPipe } from '~/core/middleware/with-pipe';

function helloWorldHandler(req: NextApiRequest, res: NextApiResponse) {
  res.status(200).json({ text: 'Hello' })
}

export default withPipe(
  withAuthedUser,
  withRole(MembershipRole.Admin),
  helloWorldHandler,
);
```

### Example: Guarding an API Route based on the Organization's Status

Let's say we want to create a function that guards an API route based on the organization's status.

**Important**: this middleware requires that the `withAuthedUser` middleware is used before it - otherwise, the `req.firebaseUser` object will not be available.

 ```ts {% title="core/middleware/with-active-subscription.ts" %}
import { NextApiRequest,NextApiResponse } from "next";
import { getCurrentOrganization } from '~/lib/server/organizations/get-current-organization';

export function withActiveSubscription() {
  return async function(req: NextApiRequest, res: NextApiResponse) {
    const userId = req.firebaseUser.uid;
    const currentOrganizationId = req.cookies.organizationId;
    const organization =
      await getCurrentOrganization(userId, currentOrganizationId);

    const status = organization.subscription?.status;
    const isActive = ['active', 'trialing'].includes(status);

    if (!isActive) {
      return res.status(403).json({
        error: 'You do not have permission to access this resource.'
      });
    }
  };
}
```

Now, we can use this middleware in our API route handler.

 ```ts {% title="pages/api/hello.ts" %}
import { NextApiRequest, NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withActiveSubscription } from '~/core/middleware/with-active-subscription';
import { withPipe } from '~/core/middleware/with-pipe';

function helloWorldHandler(req: NextApiRequest, res: NextApiResponse) {
  res.status(200).json({ text: 'Hello' })
}

export default withPipe(
  withAuthedUser,
  withActiveSubscription,
  helloWorldHandler,
);
```


-----------------
FILE PATH: kits/next-fire/how-to/app/adding-app-pages.mdoc

---
status: "published"

title: "Adding pages to the application of your Makerkit Next.js Firebase project"
label: Adding Pages
order: 1
description: "Learn how to add new pages to the application of your Makerkit Next.js Firebase project."
---


To add a new page to the application site of your Next.js project, you can create a new page in the `pages` directory and using the required `getServerSideProps` function.

By "application", we mean the private pages of your project behind the authentication wall, such as the dashboard, the settings page, etc.

As such - we need to make sure to protect these pages from being accessed by unauthorized users. To do so, we can use the `withAppProps` function, which will redirect the user to the login page if they are not authenticated.

### Adding a new page to the app

For example, if you want to add a page at `/tasks`, you can create a new file at `pages/about.tsx` with the following content:

```tsx
import { GetServerSidePropsContext } from 'next';
import { withAppProps } from '~/lib/props/with-app-props';
import RouteShell from '~/components/RouteShell';

const Tasks = () => {
  return (
    <RouteShell title={'Tasks'}>
     Tasks...
    </RouteShell>
  );
};

export default Tasks;

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  return await withAppProps(ctx);
}
```

<div>
  {% alert type="warning" title="Always use withAppProps" %}
      It is important to always use the <code>withAppProps</code> function in your <code>getServerSideProps</code> function, as it will make sure that the user is authenticated before accessing the page.

{% /alert %}
</div>

---



Let's break down what's happening here:

1. **withAppProps**: This function will retrieve all the required data and send it to the page as props. It will also redirect the user to the login page if they are not authenticated.
2. **RouteShell**: This component is a wrapper that will add the required layout to the page, including the navigation bar, the sidebar, the footer, etc. It will also initialize Firebase Firestore.

-----------------
FILE PATH: kits/next-fire/how-to/app/guarding-pages.mdoc

---
status: "published"
title: "Guarding Pages in the Next.js Firebase SaaS kit"
label: "Guarding Pages"
order: 4
description: "Learn how to guard pages in your Next.js Firebase application"
---

As a server-side rendered application, Next.js Firebase will always render the page on the server before sending it to the client. This means that you can use the `getServerSideProps` function to guard pages from being accessed by unauthorized users.

As shown before, we will reuse the `withAppProps` function to make sure that we initialize the required data before rendering the page.

In the example below:
1. We use the `withAppProps` function to initialize the required data
2. We check if the user can access the page
3. If the user can't access the page, we redirect them to the dashboard

This is a common pattern that you can use to guard pages in your application and redirect users to the dashboard if they are not authorized to access the page.

```tsx
import { GetServerSidePropsContext } from "next";
import { withAppProps } from '~/lib/props/with-app-props';

function Page() {
  // render the page
}

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  const appProps = await withAppProps(ctx);

  // We augment the props with the data we want
  // to pass to the client
  // replace the below with your own logic
  const canAccessPage = await checkUserCanAccessPage();

  if (!canAccessPage) {
    return {
      redirect: {
        destination: '/dashboard',
        permanent: false,
      },
    }
  }

  return appProps;
}
```

## Guarding Application Pages with User Roles

Let's make a more complete example by checking the user role and redirecting them to the dashboard if they are not an admin.

```tsx
import { GetServerSidePropsContext } from "next";
import { withAppProps } from '~/lib/props/with-app-props';
import { MembershipRole } from '~/lib/organizations/types/membership-role';

function Page() {
  // render the page
}

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  const appProps = await withAppProps(ctx);

  // get user role within the organization
  const userRole = appProps.organization.members[appProps.user.id].role;

  if (!canAccessPage(userRole)) {
    return {
      redirect: {
        destination: '/dashboard',
        permanent: false,
      },
    }
  }

  return appProps;
}

function canAccessPage(role: MembershipRole) {
  return role >= MembershipRole.Admin;
}
```

## Guarding Application Pages with Organization Subscriptions

In this example, we want to make sure the Organization is subscribed to a plan before allowing the user to access the page.

To do so, we check the `subscription` property of the `Organization` object and redirect the user to the dashboard if the subscription is `active` or `trialing`.

```tsx
import { GetServerSidePropsContext } from "next";
import { withAppProps } from '~/lib/props/with-app-props';
import { OrganizationSubscription } from '~/lib/organizations/types/organization-subscription';

function Page() {
  // render the page
}

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  const appProps = await withAppProps(ctx);
  const subscription = appProps.organization.subscription;

  if (!canAccessPage(subscription)) {
    return {
      redirect: {
        destination: '/dashboard',
        permanent: false,
      },
    }
  }

  return appProps;
}

function canAccessPage(subscription: OrganizationSubscription) {
  return ['active', 'trialing'].includes(subscription.status);
}
```


-----------------
FILE PATH: kits/next-fire/how-to/app/passing-data-server-to-client.mdoc

---
status: "published"

title: "Passing data from the server to the client in your Next.js Pages"
label: Passing data from server to client
order: 3
description: "Being a server-side rendered app, we can pass data from the server to the client whenever the user visits the page. Here is how"
---


Passing data from the server to a client during SSR is a common task.

In Makerkit, you can use the following pattern to pass data from the server to the client.

1. We use `withAppProps` to initialize the data on the server and making the required checks.
2. We augment the `props` property returned by `withAppProps` with the data we want to pass to the client.

```tsx
import { GetServerSidePropsContext } from "next";
import { withAppProps } from '~/lib/props/with-app-props';

type Data = {
  // ...
}

function Page({ data }: { data: Data }) {
  // you can use the data here
}

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  const { props } = await withAppProps(ctx);

  // We augment the props with the data we want
  // to pass to the client
  // replace the below with your own logic
  const data = await getDataFromServer();

  return {
    props: {
      ...props,
      data,
    }
  };
}
```

As you can see - we augment the `props` property returned by `withAppProps` with the data we want to pass to the client.

The property `data` will then be available in the `props` property of the page component `Page`.

```tsx
type Data = {
  // ...
}

interface PageParams {
  data: Data
}

function Page({ data }: { data: Data }) {
  // you can use the data here
}

export default Page;
```

-----------------
FILE PATH: kits/next-fire/how-to/app/updating-sidebar-menu.mdoc

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
FILE PATH: kits/next-fire/how-to/authentication/change-auth-strategy.mdoc

---
status: "published"
title: "How to change Authentication strategy | Next.js Firebase SaaS Kit"
label: Change Authentication strategy
order: 0
description: "Learn how to change the authentication strategy in Makerkit. Choose between email and password, phone number, email link, and OAuth."
---

Makerkit supports various authentication strategies supported by Firebase. We can customize these using the global configuration at `src/configurationt.ts`.

In the configuration file, you can find the following code:

```ts
auth: {
  // Enable MFA. You must upgrade to GCP Identity Platform to use it.
  // see: https://cloud.google.com/identity-platform/docs/product-comparison
  enableMultiFactorAuth: true,
  // When enabled, users will be required to verify their email address
  // before being able to access the app
  requireEmailVerification:
    process.env.NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION === 'true',
  // NB: Enable the providers below in the Firebase Console
  // in your production project
  providers: {
    emailPassword: true,
    phoneNumber: false,
    emailLink: false,
    oAuth: [GoogleAuthProvider],
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
  oAuth: [GoogleAuthProvider],
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
  oAuth: [GoogleAuthProvider],
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
  oAuth: [GoogleAuthProvider],
},
```

### Enabling OAuth Authentication

This strategy allows users to sign up and sign in using their social media accounts. You can enable multiple OAuth providers.

To enable this strategy, set `providers.oAuth` to an array of OAuth providers.

```tsx
import { GoogleAuthProvider, FacebookAuthProvider } from 'firebase/auth';

providers: {
  emailPassword: false,
  phoneNumber: false,
  emailLink: false,
  oAuth: [GoogleAuthProvider, FacebookAuthProvider],
},
```

## Can I use multiple authentication strategies?

Yes, you can use multiple authentication strategies. For example, you can enable email and password authentication and phone number authentication at the same time.

```tsx
providers: {
  emailPassword: true,
  phoneNumber: true,
  emailLink: false,
  oAuth: [GoogleAuthProvider],
},
```

With that said, the UI is not designed to support multiple authentication strategies. *You will need to customize the UI to support multiple authentication strategies**.


-----------------
FILE PATH: kits/next-fire/how-to/authentication/require-email-verification.mdoc

---
status: "published"

title: "How to require email verification"
label: Require Email Verification
order: 1
description: "Learn how to require email verification for your users using the Firebase kit."
---


By default, Firebase does not enforce email verification. However, we added support for this feature in the Firebase kit by using server-side checks.

### Can I require email verification for my users using the Firebase kit?

**Yes** - you can.

If you decide to enable this functionality, we will check server-side if the user has verified their email address before allowing them to view the protected pages.

To enable this functionality, you need to set the environment variable `NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION` to `true` in your `.env` file.

```tsx
NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION=true
```

Feel free to add this variable to the chosen environment file (`.env.development`, `.env.production`, etc) or to the `.env` file if you want to enable it for all environments.


-----------------
FILE PATH: kits/next-fire/how-to/contextual-data/organization.mdoc

---
status: "published"

title: "Fetching the selected Organization"
label:  "Fetching the selected Organization"
order: 1
description: "Learn how to fetch the selected organization from the backend and frontend."
---


There are several ways to fetch the current selected organization, depending on the use-case.

## Fetching the selected Organization from the backend

You will use this function fairly often for fetching the current organization from the backend.

```tsx
import { NextApiRequest,NextApiResponse } from "next";
import { getCurrentOrganization } from '~/lib/server/organizations/get-current-organization';
import { withPipe } from '~/core/middleware/with-pipe';
import { withAuthedUser } from '~/core/middleware/with-authed-user';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  const user = req.firebaseUser;
  const organization = await getCurrentOrganization(user.uid);
  // ...
}

export default withPipe(withAuthedUser, handler);
```

## Fetching the selected Organization from the frontend

The `withAppProps` middleware used in the gated app pages makes sure that the current organization is passed from the backend to the frontend.

### Getting the Organization with "useCurrentOrganization"

To retrieve the signed in user from the frontend, you can use the `useUserSession` hook:

```tsx
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const organization = useCurrentOrganization();
```

This is a React hook and can only be used inside a React component.

### Getting if the Organization subscription status is set to "active" on the frontend

You can use the `useIsSubscriptionActive` hook to get the subscription status of the current organization. You may want to use this function to gate access to certain parts of the app.

```tsx
const isActive = useIsSubscriptionActive();
```

This is a React hook and can only be used inside a React component.




-----------------
FILE PATH: kits/next-fire/how-to/contextual-data/user.mdoc

---
status: "published"

title: "Fetching the signed in User"
label:  "Fetching the signed in User"
order: 0
description: "Learn how to fetch the signed in user from the backend and frontend."
---


There are several ways to fetch the signed in user, depending on the use-case.

## Fetching the signed in user from the backend

To do so, use the function `getLoggedInUser` where you have access to the request object, such as in a Next.js API route, or `getServerSideProps` function.

```tsx
import { GetServerSidePropsContext } from "next";
import { getLoggedInUser } from '~/core/firebase/admin/auth/get-logged-in-user';

async function getServerSideProps(ctx: GetServerSidePropsContext) {
  const user = await getLoggedInUser(ctx);

  return {
    props: {
      user,
    },
  };
}
```

Alternatively, you can use the `withAuthedUser` middleware to make sure that the API handler is only called when the user is signed in:

```tsx
import { NextApiRequest,NextApiResponse } from "next";
import { withPipe } from '~/core/middleware/with-pipe';
import { withAuthedUser } from '~/core/middleware/with-authed-user';

async function handler(req: NextApiRequest, res: NextApiResponse) {
  const user = req.firebaseUser;
  // ...
}

export default withPipe(withAuthedUser, handler);
```

The `firebaseUser` property is added to the request object by the `withAuthedUser` middleware. If the user is not signed in, the middleware will return a 401 response.

## Fetching the signed in user from the frontend

The `withAppProps` middleware used in the gated app pages makes sure that the current signed in user is passed from the backend to the frontend.

### Getting the user Session with "useUserSession"

To retrieve the signed in user from the frontend, you can use the `useUserSession` hook:

```tsx
import { useUserSession } from '~/core/hooks/use-user-session';

const userSession = useUserSession();
```

The `userSession` objects contains the following properties:

```ts
interface UserSession {
  auth: AuthUser | undefined;
  data: UserData | undefined;
}
```

1. **Auth State**: The `auth` property contains the user's authentication data, such as the user ID, email, and so on.
2. **Firestore Data**:  The `data` property contains the user's Firestore record. By default, we do not add any data to the user's Firestore record - but it's assumed you will allow users to store additional data in their Firestore record.

This is a React hook and can only be used inside a React component.

### Getting the user ID with "useUserId"

To retrieve the signed in user ID from the frontend, you can use the `useUserId` hook:

```tsx
import { useUserId } from '~/core/hooks/use-user-id';

const userId = useUserId();
```

This is a React hook and can only be used inside a React component.

### Getting the user role within the organization with "useUserRole"

To retrieve the signed in user role of the current organization from the frontend, you can use the `useUserRole` hook:

```tsx
import { useUserRole } from '~/core/hooks/use-user-role';

const role = useUserRole();
```

This is a React hook and can only be used inside a React component.


