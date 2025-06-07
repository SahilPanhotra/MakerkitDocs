-----------------
FILE PATH: kits/next-fire/admin/adding-admin-users.mdoc

---
status: "published"
title: "Adding Admin users | Next.js Firebase SaaS Kit"
label: Adding Admins
order: 2
description: "A quick guide to set the correct permissions to add new admin users"
---

To access the Super Admin, your users need to be updated with the correct Custom Claims. We use custom claims to verify they can access the super admin.

We do so by setting the `role` property to `super-admin`.

```json
{
  "role": "super-admin"
}
```

You can provide more roles for your super admin users - but we only use `super-admin` by default, which is equipped with all permissions.

If you're using the emulator, the user `test@makerkit.dev` is set up with super admin permissions by default.

## Updating Custom claims to set User as Super Admin

To set a user's custom claims against a production environment, we need to write a script that can update the user's custom claims
using the Firebase Admin package.

Below is a script that allows you to set a user's custom claims.

Before proceeding, make sure to:

1. Install "dotenv" using `npm i dotenv` if not installed yet
2. Add your environment variables to `.env.local`, where we can store secret keys without them being committed to the Git repository
3. Grab the ID of the user (from the Firebase console) you want to set as super-admin and place it in the variable `userId` at the bottom of the script.

 ```tsx {% title="assign-super-admin.ts" %}
import { config } from 'dotenv';

config({
  path: '.env.local'
});

import * as admin from 'firebase-admin';

const privateKey = process.env.SERVICE_ACCOUNT_PRIVATE_KEY.replace(/\\n/g, '\n');

export const app = admin.initializeApp({
  credential: admin.credential.cert({
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
    privateKey,
    clientEmail: process.env.SERVICE_ACCOUNT_CLIENT_EMAIL,
  }),
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
});

const auth = app.auth();

async function writeClaims(userId: string, claims: object, merge = true) {
  console.log(`Setting claims for user with ID ${userId}`);

  try {
    const user = await auth.getUser(userId);
    const currentClaims = user.customClaims || {};

    await auth.setCustomUserClaims(
      userId,
      merge
        ? {
            ...currentClaims,
            ...claims,
          }
        : claims,
    );

    console.log(`Claims set successfully`);
  } catch (e) {
    console.error(e);
  }

  process.exit();
}

const userId = '<user_id>';

const claims = {
  role: 'super-admin'
};

void writeClaims(userId, claims);
```

**NB: Update the "userId" variable with the ID of your user**.

To run the script, we can run the following command:

```bash
npx tsx ./assign-super-admin.ts
```

This command assumes you're running it from the root folder of your project.

---


After running the script, you user is a super admin. You may now need to sign out and then sign in again to see the changes if not working.

You can now navigate to the Admin at the path `/admin` of your application!

-----------------
FILE PATH: kits/next-fire/admin/admin-overview.mdoc

---
status: "published"
title: "Admin Overview | Next.js Firebase SaaS Kit"
label: Overview
order: 1
description: "Overview of the Super Admin panel in the Next.js Firebase kit"
---

The Super Admin was released in the version 0.12.0.

It allows your administrators to manage your users and organizations - such as:

1. Viewing the app's users and their details
2. Viewing the app's organizations
3. Viewing an organization's members
4. Banning or Disabling users
5. Impersonating users

## Accessing the Super Admin

To access the super admin, you need to navigate to the path `/admin`. If you're using the emulator, you can do so using the `test@makerkit.dev` account, which is set to having a super admin role.

To find out how to set admin permissions to your production users, please continue reading the next post.

## Viewing Users

The Users page allows you to manage your users. You can view all your users paginated and search for a specific user. You can also ban, reactivate and impersonate users.

{% img src="/assets/images/posts/admin-users-page.webp" width="2844" height="1938" alt="Super Admin Users Page" /%}

You can view the detail of a user by clicking on the user's ID, where you can see the user's email, name, organizations, and other information.

#### Disabling/Banning users

Admins can easily disable/ban some users from the users page or from the user's page:

{% img src="/assets/images/posts/admin-disable-user.webp" width="2862" height="1952" alt="Super Admin Disable User" /%}

#### Impersonating Users

An extremely useful feature for SaaS founders is the ability to impersonate users, usually for debugging ot helping users out.

{% img src="/assets/images/posts/admin-impersonate-user.webp" width="2862" height="1952" alt="Super Admin Impersonate User" /%}


### Organizations

The Organizations page allows you to manage your organizations. You can view all your organizations paginated and search for a specific organization. You can also view its members and the subscription status.

{% img src="/assets/images/posts/admin-organizations-page.webp" width="2868" height="1960" alt="Super Admin Organizations Page" /%}

#### View Members

The View Members page allows you to view all the members of an organization. You can also search for a specific member. From here, you can navigate to the detail of a member by clicking on the member's ID.

{% img src="/assets/images/posts/admin-organization-members-page.webp" width="2868" height="1960" alt="Super Admin Organization Members Page" /%}

-----------------
FILE PATH: kits/next-fire/admin.mdoc

---
status: "published"
title: "Super Admin"
label: "Super Admin"
order: 7
description: "The Super Admin helps you manage your MakerKit application."
collapsible: true
collapsed: true
---

The Admin section of Makerkit provides a powerful set of tools for managing your SaaS application. As your application grows, you'll need ways to manage users, monitor subscriptions, and keep things running smoothly. Let's explore what's available in the Admin dashboard.

### User Management

Browse through all accounts in your system, whether they're individual users or team accounts. You can:
- View account details
- Check subscription status
- Ban problematic users
- Delete accounts when needed

### Account Details

Clicking into an account gives you detailed information about:
- Account status
- Subscription details
- Team members
- Billing information

### Actions

As an admin, you can:
- Ban accounts to restrict access
- Delete accounts when necessary
- View detailed account information
- Monitor subscription status

-----------------
FILE PATH: kits/next-fire/api/reactfire.mdoc

---
status: "published"
title: Reactfire
label: Reactfire
description: 'Reactfire is the semi-official React library for Firebase. The API of Reactfire is fundamental to the Next.js Firebase SaaS Starter kit.'
---

The Next.js Firebase kit uses [ReactFire](https://github.com/firebaseextended/reactfire) extensively throughout the kit. ReactFire is a library that allows you to bind Firebase data to React components. This allows you to easily display data from Firebase in your React components.

For the majority of use cases, you will be using Reactfire's React hooks to interact with your Firebase data, as we've seen in the previous sections.

These include:
- `useFirestore`: this will allow you to interact with your Firestore database
- `useAuth`: this will allow you to interact with your Firebase authentication
- `useStorage`: this will allow you to interact with your Firebase storage (you are required to add the FirebaseStorageProvider above your component tree for this to work)

To use the Next.js Firebase effectively, it's extremely important to understand how Reactfire works, so please do familiarise yourself with the [ReactFire documentation](https://github.com/firebaseextended/reactfire).

-----------------
FILE PATH: kits/next-fire/api/useCurrentOrganization.mdoc

---
status: "published"
title: useCurrentOrganization
label: useCurrentOrganization
description: 'The "useCurrentOrganization" custom hook allows you to get the current organization from the context of the current user.'
---

This hook will allow you to get the current user organization.

The data for the current user organization is fetched server-side using the getServerSideProps function, which is executed on the server before the page is rendered. This ensures that the organization data is available to the client-side code as soon as the page loads.

Once the data is fetched, it's injected into the React component tree using the Context API. The `useCurrentOrganization` hook uses this context to retrieve the current user organization and return it to your code.

For example, let's see how we can use this hook to get the current user organization name:

```ts
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

export function useCurrentOrganizationName() {
  const organization = useCurrentOrganization();

  return organization?.name;
}
```

In the example provided, the `useCurrentOrganization` hook is used to retrieve the organization object, which contains various properties including the name of the organization. The `useCurrentOrganizationName` function then extracts the name property from the organization object and returns it.

Finally, it's worth noting that there are many other hooks available in the `~/lib/organizations/hooks` folder that allow you to retrieve other data related to the current user organization, such as the organization ID or the list of users belonging to the organization. These hooks can be used in a similar way to the `useCurrentOrganization` hook, by importing them and using them in your code.

-----------------
FILE PATH: kits/next-fire/api/useCurrentUserRole.mdoc

---
status: "published"
label: useCurrentUserRole
title: useCurrentUserRole
description: 'The "useCurrentUserRole" custom hook allows you to get the current user role within the selected organization'
---

In case you need to access the current user role **within the current organization**, you can use the `useCurrentUserRole` hook.

To use this hook, simply import it from the `~/lib/organizations/hooks/use-current-user-role` module, and then call it from your component.

Here's an example:

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

n this example, the useCurrentUserRole hook is used to retrieve the current user's role within their current organization.

Overall, the `useCurrentUserRole` hook is a convenient and easy-to-use way to retrieve the role of the current user within their current organization in your Next.js app.

-----------------
FILE PATH: kits/next-fire/api/useIsSubscriptionActive.mdoc

---
status: "published"
label: useIsSubscriptionActive
title: useIsSubscriptionActive
description: 'The "useIsSubscriptionActive" custom hook allows you to get a boolean to check if the organization subscription is active or not.'
---


The useIsSubscriptionActive hook is a custom hook that allows you to determine whether the current organization has an active paid subscription. This can be useful if you want to display a message to the user if they are on a paid subscription, or if you want to restrict access to certain pages.

A subscription is considered active if it's either `active` or `trialing`. The `useIsSubscriptionActive` hook will return `true` if the current organization is on any paid subscription, regardless of plan.

To use this hook, simply import it from the `~/lib/organizations/hooks/use-is-subscription-active` module, and then call it from your component.
Here's an example:

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

Then, we can use this hook in our component:

```tsx
import { useIsSubscriptionActive } from '~/lib/organizations/hooks/use-is-subscription-active';

function MyComponent() {
  const isSubscriptionActive = useIsSubscriptionActive();

  if (isSubscriptionActive) {
    return <p>You are on a paid subscription!</p>;
  } else {
    return <p>You are not on a paid subscription.</p>;
  }
}
```

In this example, the `useIsSubscriptionActive` hook is used to determine whether the current organization is on a paid subscription. The resulting `isSubscriptionActive` value is then used to render a message to the user.

It's worth noting that the `useIsSubscriptionActive` hook can be customized to fit your needs. For example, you could modify the `ACTIVE_STATUSES` constant to include additional subscription statuses, or you could use a different method to retrieve the subscription status.

-----------------
FILE PATH: kits/next-fire/api/useUserSession.mdoc

---
status: "published"
title: useUserSession
label: useUserSession
order: 3
description: 'The "useUserSession" custom hook allows you to get the user session data provided by the SSR props'
---

The `useUserSession` hook is a custom hook that allows you to access the current user session data. This data includes information from both Firebase Authentication and the user's Firestore document.

This hook will return the current user session data, which includes the following information:
- `auth`: the data that comes from Firebase Authentication
- `data`: the data that comes from the user's Firestore document

```tsx
export interface UserSession {
  auth: AuthUser | undefined;
  data: UserData | undefined;
}
```

To use this hook, simply import it from the `~/core/hooks/use-user-session` module, and then call it from your component.

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

In this example, the useUserSession hook is used to retrieve the current user session data. The resulting userSession object is then used to extract the user ID, which is displayed to the user.

-----------------
FILE PATH: kits/next-fire/api/withAppProps.mdoc

---
status: "published"
title: withAppProps
label: withAppProps
order: 5
description: 'The "withAppProps" is a function that is used to populate the props of the application pages. It is used in the "getServerSideProps" function of the pages you want to gate.'
---

Props functions are functions that are executed server-side and are used to populate the props of a page. They are executed before the page is rendered, and are used to fetch data that is required for the page to render.

The `withAppProps` function is used to populate the props of the application pages. It is used in the `getServerSideProps` function of the pages you want to gate.

If you take a look at the previous sections, we used this function to protect the `tasks` application so that only authenticated users can access it.

```tsx
import { withAppProps } from "~/lib/props/with-app-props";
import { GetServerSidePropsContext } from "next";

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  return await withAppProps(ctx);
}
```

### Checks performed by the function

This function will perform the following checks:
1. Check if the user is authenticated. If not, it will redirect the user to the login page.
2. Check if the user is onboarded. If not, it will redirect the user to the onboarding page.
3. (Optionally) Check if the user has the required subscription. If not, it will redirect the user to the home page.

### Props returned to the page components

If successful, it will return the props of the page. If not, it will redirect the user to the appropriate page.

The data returned to the page includes:
1. the user session (that can be accessed using the hook `useUserSession`)
2. the organization (that can be accessed using the hook `useCurrentOrganization`)

-----------------
FILE PATH: kits/next-fire/api/withAuthProps.mdoc

---
status: "published"

title: withAuthProps
label: withAuthProps
order: 7
description: 'The "withAuthProps" is a function that is used to populate the props of the auth pages. It is used in the "getServerSideProps" function of the pages you want to gate.'
---

The `withAuthProps` function is used to populate the props of the authentication pages. You may never need to use it unless you introduce new authentication pages.

This function is used in the `getServerSideProps` function of the authentication pages.

```tsx
import { withAuthProps } from "~/lib/props/with-auth-props";
import { GetServerSidePropsContext } from "next";

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  return await withAuthProps(ctx);
}
```

This function ensures **authenticated users won't be able to access the authentication pages**.


-----------------
FILE PATH: kits/next-fire/api/withAuthedUser.mdoc

---
status: "published"
title: withAuthedUser
label: withAuthedUser
order: 10
description: 'The "withAuthedUser" is a utility function that help you load the authenticated user from the request.'
---

The `withAuthedUser` function is used to require authentication to access an API endpoint. It will return a `401` error if the user is not authenticated.

```ts {12}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

-----------------
FILE PATH: kits/next-fire/api/withCsrf.mdoc

---
status: "published"

title: withCsrf
label: withCsrf
order: 12
description: 'The "withCsrf" is a utility function that help you protect API endpoints from CSRF attacks.'
---

The `withCsrf` function is used to protect API endpoints from CSRF attacks. It will return a `403` error if the CSRF token is invalid.

```ts {10}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';
import { withCsrf } from '~/core/middleware/with-csrf';

export default function owner(req: NextApiRequest, res: NextApiResponse) {
  const handler = withPipe(
    withCsrf(),
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

-----------------
FILE PATH: kits/next-fire/api/withExceptionFilter.mdoc

---
status: "published"

title: withExceptionFilter
label: withExceptionFilter
order: 11
description: 'The "withExceptionFilter" is a utility function that help you gracefully handle exceptions in your API endpoints.'
---

The `withExceptionFilter` function is used to handle exceptions in API endpoints. It will log errors and report to Sentry when errors are encountered.

```ts {18}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```


-----------------
FILE PATH: kits/next-fire/api/withMethodsGuard.mdoc

---
status: "published"

title: withMethodsGuard
label: withMethodsGuard
order: 9
description: 'The "withMethodsGuard" is a utility function that helps you to create a guard for your API methods based on the HTTP method'
---

The `withMethodsGuard` function is used to guard API endpoints by HTTP methods. For example, in the example below, we only allow `GET` and `POST` requests to the `/api/hello-world` endpoint.

```ts {13}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withMethodsGuard(['GET', 'POST']),
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

-----------------
FILE PATH: kits/next-fire/api/withPipe.mdoc

---
status: "published"

title: withPipe
label: withPipe
order: 8
description: 'The "withPipe" is a utility function that helps you pipe functions when handling API requests.'
---

API Routes have several utilities that help you build your API endpoints, and save you from writing boilerplate code.

#### 1. `withPipe`: piping functions when handling API requests

The `withPipe` function is used to pipe functions when handling API requests.

```ts {12}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withMethodsGuard(['GET']),
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

This helps you write the utility functions we will see next in a more readable way.

-----------------
FILE PATH: kits/next-fire/api/withTranslationsProps.mdoc

---
status: "published"

title: withTranslationProps
label: withTranslationProps
order: 6
description: 'The "withTranslationProps" is a function that is used to populate the props of pages that require translation.'
---

This function is used to populate the translations of the application. All the other props functions include this by default, so you don't need to use it directly unless it's on a page that does not use any of the other ones.

You will be using this for the majority of your marketing pages.

```tsx
import { withTranslationProps } from "~/lib/props/with-translation-props";
import { GetStaticPropsContext } from "next";

export async function getStaticProps(
  context: GetStaticPropsContext
) {
  const { props } = await withTranslationProps(context);

  return {
    props,
  };
}
```

-----------------
FILE PATH: kits/next-fire/api.mdoc

---
status: "published"
title: Next.js Firebase API
label: API
description: "Makerkit exposes some API you can use in your project."
order: 14
collapsible: true
collapsed: true
---

In this section, we will explain the APIs that are available to you when using Next.js Firebase.

Makerkit provides a set of APIs that you can use in your project. These APIs are documented below.

- [`reactfire`](api/reactfire)
- [`useCurrentUserRole`](api/useApiRequest)
- [`useCurrentOrganization`](api/useFetchOrganization)
- [`useUserSession`](api/useUserSession)
- [`useIsSubscriptionActive`](api/useIsSubscriptionActive)
- [`withAppProps`](api/withAppProps)
- [`withAuthProps`](api/withAuthProps)
- [`withExceptionFilter`](api/withExceptionFilter)
- [`withPipe`](api/withPipe)
- [`withTranslationProps`](api/withTranslationProps)
- [`withAuthedUser`](api/withAuthedUser)


-----------------
FILE PATH: kits/next-fire/architecture/application-structure.mdoc

---
status: "published"
title: Structure your Makerkit Application | Next.js Firebase SaaS Kit
label: Structure your Application
order: 7
description: 'Learn how to structure your application with additional entities and business logic in your Next.js Firebase SaaS Kit'
---

In the previous section, we learned the fundamentals of Makerkit's architecture and the application layers.

In this section, we learn how to structure your application in practical terms with an example. For example, your application has an entity "events": how do we add this entity to a Makerkit application?

{% alert type="info" title="Entities are added to core" %}
NB: entities rarely (or never) get added to "core". Business domain is added to "components" and "lib".
{% /alert %}

In short, this is how we add a new entity to the application:

1. First, we add a new folder to `lib`. If the entity is "event", we add `lib/events`.
2. Then, we add the components of the `event` domain to `components/events`
3. Finally, we add the pages of the `event` domain to `pages/events`

 ```txt {% title="Structure" %}
- src
  - components
    - events
      - EventsContainerComponent.tsx
      - ...

  - lib
    - events
      - types
        - event-model.ts
        - ...
      - hooks
        - use-fetch-events.ts
        - use-create-event.ts
        - ...
      - utils
        - create-event-model.ts

  - pages
    - events
      - page.tsx
      - [event].tsx
```

### 1) Adding the entity's business domain

We will add various business logic units in the `lib/events` folder, such as types, custom hooks, API calls, factories, functions, utilities, etc.

#### Types

First, we define a type `EventModel` at `lib/events/types/event-model.ts`:

 ```tsx {% title="lib/events/types/event-model.ts" %}
export interface EventModel {
  name: string;
  description: string;
}
```

#### Custom Hooks

For example, let's write a custom hook that retrieves a list of "events" from a Firestore collection.

We create a file at `lib/events/hooks/use-fetch-events.ts` with the following content:

 ```tsx {% title="src/lib/events/hooks/use-fetch-events.ts" %}
import EventModel from '~/lib/events/types/event-model'

export function useFetchEvents() {
  const firestore = useFirestore();
  const eventsCollection = 'events';

  const collectionRef = collection(
    firestore,
    eventsCollection,
  ) as CollectionReference<EventModel>;

  return useFirestoreCollectionData(collectionRef, {
    idField: 'id',
  });
}
```

Good! We have a way to fetch our events, but we have to use it somewhere: to do so, let's create a component `EventsListContainer`. 

{% alert type="warning" title="Update Firestore rules" %}
NB: remember to update the Firestore rules to be able to read the collection.
{% /alert %}

### 2) Components

As said before, we add React components that belong to the "events" domain to `components/events`. 

In the component below, we will fetch a list of events with `useFetchEvents`:

 ```tsx {% title="components/events/EventsListContainer.tsx" %}
import { useFetchEvents } from '~/lib/events/hooks/use-fetch-events';
import { Alert } from `~/core/ui/Alert`;

const EventsListContainer: React.FC = () => {
  const { data: events, status } = useFetchEvents();

  if (status === `loading`) {
    return <p>Loading Events...</p>
  }

  if (status === `error`) {
    return (
      <Alert type='error'>
        Ops, we encountered an error!
      </Alert>
    );
  }

  return (
    <div>
      {events.map(event => {
        return (
          <div key={event.name}>
            <p>{event.name}</p>
            <p>{event.description}</p>;
          </div>
        );
      })}
    </div>
  )
};

export default EventsListContainer;
```

### 3) Pages

Finally, we can add the events component `EventsListContainer` to a page. To do so, let's create a page component at `pages/events/index.ts`:

 ```tsx {% title="pages/events/index.ts" %}
import { GetServerSidePropsContext } from 'next';

import RouteShell from '~/components/RouteShell';
import EventsListContainer from '~/components/EventsListContainer';

import { withAppProps } from '~/lib/props/with-app-props';

const EventsPage: React.FC = () => {
  return (
    <>
      <Head>
        <title key="title">Events</title>
      </Head>
      
      <RouteShell title="Events">
        <EventsListContainer />
      </RouteShell>
    </>
  );
};

export default EventsPage;

// guard the page!
export function getServerSideProps(ctx: GetServerSidePropsContext) {
  return withAppProps(ctx);
}
```

🎉 That's it! We have now built a nicely structured "events" domain.

-----------------
FILE PATH: kits/next-fire/architecture/data-model.mdoc

---
status: "published"

title: The MakerKit Data Model for your SaaS project built with Next.js and Firebase
label: Data Model
order: 8
description: 'Learn how MakerKit
defines the model of your SaaS application with Next.js and Firebase'
tags:
- saas
---

MakerKit's data model is voluntarily as simple as possible, with a limited
set of assumptions.

MakerKit is not supposed to be a finite product but a solid foundation on which it is quick to get started and build your SaaS product.

To summarize, the Firestore model contains three main entities: `users`, `organizations`, and `invites`.

1. **Users**: a Firestore representation of the authentication database. In fact, to store data about users, we need to create a user record within our `/users` collection in Firestore
2. **Organizations**: Organizations are groups of users. Here, we store the list of members that belong to the organizations and everything that needs to be shared between the users: for example, templates, settings, integrations, and so on.
3. **Invites**: invites are a sub-collection of organizations used to store invitations that users can accept.

{% img src="/assets/images/docs/data-model.webp" width="1775" height="2436" /%}

## Users

Users are, of course, foundational to any application.

If a user signed in using Firebase Authentication, it merely means we have verified their credentials, not that they have an account.

We have to create an additional collection to store a user's data, such as:
- settings
- the memberships they belong to
- any other information which is not possible to update using their Authentication record (profile photo, name, email, and sign-in providers)

We only store the user's organizations as an object on the `users` collections by default.

The Firestore model looks like something similar:

```
- users
  - [userId]:
    - ... user data
```

We store the relationships between users and organizations within the
`Organizations` collection.

## Organizations

`Organizations` are groups of users.

If the name doesn't suit your domain, don't worry: you can always change the terminology using the localization files.

A user can belong to one or many organizations: each organization lists its members in an object, similar to what we do with `users`.

Below is what the `Organizations` schema looks like:

```
- organizations
  - [organizationId]
      - name
      - timezone
      - members
        - [memberId]
          - role
          - user // this is a Firestore reference
  }
```

Of course, we can assign a `name` to every organization and a `timezone` property (before you realize you're going to need it).

We define each membership by the user ID. Each membership contains the following information:

- the user role
- the user reference to `/users/[userId]`

Thus, we store the role in both collections to reduce the number of reads when we query the project members: we need to keep them in sync.

### User Roles

By default, MakerKit defines three roles: Owner (can be only one), Admin, and Member.

This enum is hierarchical and associated with a number:

```tsx
enum MembershipRole {
  Member,    // 0
  Admin,     // 1
  Owner      // 2
}
```

An Admin can do anything a member can do, and an Owner can do anything an Admin can.

### Invites

To create an invite to join an organization, we use a new collection
`invites` placed below an `organization` document: for example,
`/organizations/1/invites/1`.

The invite's interface is also straightforward:

```tsx
interface MembershipInvite {
  code: string;
  email: string;
  role: MembershipRole;
  expiresAt: number;

  organization: {
    id: string;
    name: string;
  };
}
```

- the `email` property represents the email of the user invited. The application supports users signing up with a different email: you have to change this if you wish to disallow it
- the `role` is the assigned role, which cannot be `Owner`
- the `code` property is the unique identifier from which the user can access
the invite through a link
- the `expiresAt` property is the Unix timestamp when the invite will
expire. By default, it's going to be one week
- the `organization` object duplicates some valuable data from its
parent (which saves us from rereading the document)


