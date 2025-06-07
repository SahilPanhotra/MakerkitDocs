-----------------
FILE PATH: kits/react-router-supabase-turbo/translations.mdoc

---
status: "published"
title: "Translations in React Router Supabase Turbo"
label: "Translations"
order: 10
description: "Translations in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Make your application multilingual:

1. [**Adding New Translations**](translations/adding-translations)
   Add support for new languages.
2. [**Language Selector**](translations/language-selector)
   Implement a language selection interface.
3. [**Using Translations**](translations/using-translations)
   Implement translations throughout your application.



-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting/troubleshooting-authentication.mdoc

---
status: "published"

title: 'Troubleshooting authentication issues in the React Router Supabase kit'
label: 'Authentication'
order: 2
description: 'Troubleshoot issues related to authentication in the React Router Supabase SaaS kit'
---


## Supabase redirects to localhost instead of my website URL

This is most likely an issue related to not having set up the Authentication URL in the Supabase settings as describe in [the going to production guide](going-to-production/supabase);

## Cannot Receive Emails from Supabase

This issue may arise if you have not setup an SMTP provider from within Supabase.

Supabase's own SMTP provider has low limits and low deliverability, therefore you must set up your own - and only use the default Supabase's one for testing purposes.

## Cannot sign in with oAuth provider

This is most likely a settings issues from within the Supabase dashboard or in the provider's settings.

Please [read Supabase's documentation](https://supabase.com/docs/guides/auth/social-login) on how to set up third-party providers.


-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting/troubleshooting-billing.mdoc

---
status: "published"
title: 'Troubleshooting billing issues in the React Router Supabase kit'
label: 'Billing'
order: 3
description: 'Troubleshoot issues related to billing in the React Router Supabase SaaS kit'
---

## Cannot create a Checkout

This happen in the following cases:

1. **The environment variables are not set correctly**: Please make sure you have set the following environment variables in your `.env` file if locally - or in your hosting provider's dashboard if in production
2. **The plan IDs used are incorrect**: Make sure to use the exact plan IDs as they are in the payment provider's dashboard.

## The Database is not updated after subscribing to a plan

This may happen if the webhook is not set up correctly. Please make sure you have set up the webhook in the payment provider's dashboard and that the URL is correct.

Common issues include:
1. **The webhook is not set up correctly**: Please make sure you have set up the webhook in the payment provider's dashboard and that the URL is correct.
2. **The plan IDs used are incorrect**: Make sure to use the exact plan IDs as they are in the payment provider's dashboard.
3. **The environment variables are not set correctly**: Please make sure you have set the following environment variables in your `.env` file if locally - or in your hosting provider's dashboard if in production

### Locally

If working locally, make sure that:

1. **Stripe**: If using Stripe, that the Stripe CLI is up and running
2. **Lemon Squeezy**: If using Lemon Squeezy, that the webhook set in Lemon Squeezy is correct and that the server is running. Additionally, make sure the proxy is set up correctly if you are testing locally (see the Lemon Squeezy documentation for more information).

### Production

If working in production, make sure that:

- The Stripe/Lemon Squeezy webhooks are set up correctly and are being sent to the correct URL. The URL must be publicly accessible. If the URL is not publicly accessible, you will not be able to receive webhooks from Stripe.
- If the webhooks are being sent, please read the logs in your hosting provider to see if there are any errors related to webhooks.
- If after reading the logs, the webhooks are not being sent, please contact support by posting relevant information in the Support Ticket.

When opening a support ticket, please make sure to include:
1. The logs being sent from Stripe in the Stripe Dashboard
2. The logs in your hosting provider
3. Any error messages you may have received

-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting/troubleshooting-deployment.mdoc

---
status: "published"

title: 'Troubleshooting deployment issues in the React Router Supabase kit'
label: 'Deployment'
order: 4
description: 'Troubleshoot issues related to deploying the application in the React Router Supabase SaaS kit'
---


## The deployment build fails

This is most likely an issue related to the environment variables not being set correctly in the deployment environment. Please analyse the logs of the deployment provider to see what is the issue.

The kit is very defensive about incorrect environment variables, and will throw an error if any of the required environment variables are not set. In this way - the build will fail if the environment variables are not set correctly - instead of deploying a broken application.

If you are deploying to Vercel, [please follow this guide](going-to-production/vercel).

-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting/troubleshooting-emails.mdoc

---
status: "published"
title: 'Troubleshooting emails issues in the React Router Supabase kit'
label: 'Emails'
order: 5
description: 'Troubleshoot issues related to emails in the React Router Supabase SaaS kit'
---

To troubleshoot emails issues in the React Router Supabase kit, please make sure to:

1. **Correct Credentials**: please make sure you have setup the correct credentials in your environment variables. Refer to the [emails guide](../emails/email-configuration) for more information.
2. **Your domain is verified**: make sure your domain is verified in your Email provider
3. **The Database Webhooks are configured**: make sure you have configured the Database Webhooks in your Supabase project. Refer to the [Supabase deployment guide](../going-to-production/supabase) for more information. Make sure to read Webhooks are pointing to the correct URL and use the correct shared secret.
4. **The Webhooks use a public URL**: make sure the webhooks use a public URL and not behind authentication (such as a Vercel Preview URL).
5. **Read the Logs**: please verify your server logs in your hosting provider to see if there are any errors related to emails.

If you have done all the above and still having issues, please open a Support Ticket in Discord.

-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting/troubleshooting-installation.mdoc

---
status: "published"

title: 'Troubleshooting installation issues in the React Router Supabase SaaS kit'
label: 'Installation'
order: 1
description: 'Troubleshoot issues related to installing the React Router Supabase SaaS kit'
---


## Cannot clone the repository

Issues related to cloning the repository are usually related to a Git misconfiguration in your local machine. The commands displayed in this guide using SSH: these will work only if you have setup your SSH keys in Github.

If you run into issues, [please make sure you follow this guide to set up your SSH key in Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

If this also fails, please use HTTPS instead. You will be able to see the commands in the repository's Github page under the "Clone" dropdown.

Please also make sure that the account that accepted the invite to Makerkit, and the locally connected account are the same.

## The React Router dev server does not start

This may happen due to some issues in the packages. Try to clean the workspace and reinstall everything again:

```bash
pnpm run clean:workspace
pnpm run clean
pnpm i
```

You can now retry running the dev server.

## Supabase does not start

If you cannot run the Supabase local development environment, it's likely you have not started Docker locally. Supabase requires Docker to be installed and running.

Please make sure you have installed Docker (or compatible software such as Colima, Orbstack) and that is running on your local machine.


-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting/troubleshooting-module-not-found.mdoc

---
status: "published"
title: 'Troubleshooting module not found issues in the React Router Supabase kit'
label: 'Modules not found'
order: 6
description: 'Troubleshoot issues related to modules not found in the React Router Supabase SaaS kit'
---

Let's walk through common "Module not found" errors and how to fix them in your Makerkit project.

This issue is mostly related to either dependency installed in the wrong package or issues with the file system.

### 1. Dependency Installation Issues

The most common cause is incorrect dependency installation. Here's how to fix it:

```bash
# First, clean your workspace
pnpm run clean:workspaces
pnpm run clean

# Reinstall dependencies
pnpm install
```

If you're adding new dependencies, make sure to install them in the correct package:

```bash
# For main app dependencies
pnpm install my-package --filter web

# For a specific package
pnpm install my-package --filter @kit/ui
```

For example, fi you're using the dependency in the `@kit/ui` package, you should install it in the `@kit/ui` package:

```bash
pnpm add my-package --filter "@kit/ui"
```

If it's in the main app, you should install it in the main app:

```bash
pnpm add my-package --filter web
```

### 2. Windows OneDrive Conflicts

OneDrive can cause file system issues with Node.js projects. If you're using Windows with OneDrive:

1. Move your project outside of OneDrive-synced folders
2. Or disable OneDrive sync for your development folder

-----------------
FILE PATH: kits/react-router-supabase-turbo/troubleshooting.mdoc

---
status: "published"
title: "Troubleshooting in React Router Supabase Turbo"
label: "Troubleshooting"
order: 17
description: "Troubleshooting in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Solve common issues in your application:

1. [**Installation Issues**](troubleshooting/troubleshooting-installation)
   Resolve common installation problems.
2. [**Authentication Issues**](troubleshooting/troubleshooting-authentication)
   Fix authentication-related problems.
3. [**Billing Issues**](troubleshooting/troubleshooting-billing)
   Resolve payment and billing problems.
4. [**Deployment Issues**](troubleshooting/troubleshooting-deployment)
   Fix common deployment challenges.

Each section provides focused guidance for its specific domain.


-----------------
FILE PATH: kits/remix-fire/architecture/application-structure.mdoc

---
status: "published"
title: Structure your Makerkit Application | Remix.js Firebase SaaS Kit
label: Structure your Application
order: 7
description: 'Learn how to structure your application with additional entities and business logic in your Remix.js Firebase SaaS Kit'
---

In the previous section, we learned the fundamentals of Makerkit's architecture and the application layers.

In this section, we learn how to structure your application in practical terms with an example. For example, your application has an entity "events": how do we add this entity to a Makerkit application?

{% alert type="info" %}
NB: entities rarely (or never) get added to "core". Business domain is added to "components" and "lib".
{% /alert %}

In short, this is how we add a new entity to the application:

1. First, we add a new folder to `lib`. If the entity is "event", we add `lib/events`.
2. Then, we add the components of the `event` domain to `components/events`
3. Finally, we add the pages of the `event` domain to `pages/events`

 ```txt {% title="Structure" %}
- app
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

  - routes
    - __app
      - events
        - page.tsx
        - $event.tsx
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

 ```tsx {% title="lib/events/hooks/use-fetch-events.ts" %}
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

{% alert type="warning" %}
NB: remember to update the Firestore rules to be able to read the collection.
{% /alert %}

### 2) Components

As said before, we add React components that belong to the "events" domain to `components/events`. 

In the component below, we will fetch a list of events with `useFetchEvents`:

 ```tsx {% title="components/events/EventsListContainer.tsx" %}
import { useFetchEvents } from '~/lib/events/hooks/use-fetch-events';
import Alert from `~/core/ui/Alert`;

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

### 3) Routes

Finally, we can add the events component `EventsListContainer` to a page. To do so, let's create a page component at `routes/__app/events/page.tsx`:

 ```tsx {% title="routes/__app/events/page.tsx" %}
import EventsListContainer from '~/components/EventsListContainer';

const EventsPage: React.FC = () => {
  return (
     <EventsListContainer />
  );
};

export default EventsPage;
```

🎉 That's it! We have now built a nicely structured "events" domain.

-----------------
FILE PATH: kits/remix-fire/architecture/architecture.mdoc

---
status: "published"

title: Architecture and Folder Structure for your Remix SaaS application
label: Architecture and Folder Structure
order: 6
description: 'Learn how MakerKit allows you to build a Remix application
with a scalable and production-grade architecture and folder structure'
---


MakerKit's primary goals are:

- **Solid Foundations** - providing you with the necessary code to get up and
running
with a Saas with minimal configuration
- **Extendable and Adaptable** - the ability to change and extend the codebase as efficiently as possible

While it's relatively simple to provide a functioning codebase, it's not always easy to make it:
- **Simple** - a clean codebase quickly understandable by anyone
- **Easy to change** - a codebase that can be easily changed to a
particular domain

Once you start your project, you can begin unpicking and replacing the existing code to adapt it to your application's domain. At the same time, MakerKit may have pushed an update that fixed a bug or added a new cool feature.

The two projects will diverge and inevitably end up in a conflicting state.

Thankfully, Git makes it easy to merge conflicts. But it's still a hassle.

### Framework Architecture

To reduce the number of possible conflicts, MakerKit ships with a particular folder structure:

```
- app
  - routes
    - __site
    - __app
    - __auth
  - lib
  - components
```

These folders represent four separate layers. MakerKit is an Onion:

{% img width="2103" height="1221" src="/assets/images/docs/architecture.png" alt="MakerKit's Remix Architecture" /%}

### Core

Let's talk about the lower level: the `core`. This layer is a collection of building blocks, utilities, and APIs that make up MakerKit.

This part of the boilerplate is configurable and makes no assumptions about your domain.

You are still welcome to change this code to suit your needs, but you can expect most updates from upstream to change the code in this directory.

### Lib and Components

The mid-level is in two separate folders (to avoid excessive nesting):

- `lib`: here is where we write most domain-related business logic (hooks, contexts, queries, mutations, etc.)
- `components`: here is where we write the domain-related components (ex. Profile Page, Create Project form, etc.)

MakerKit comes with some business logic (it would be nearly impossible otherwise to build a genuinely functioning SaaS). You should customize the existing business logic for your application, and it is likely where you will add most of your code.

This layer can still be updated by the `upstream` repository, but you should not expect too many changes unless it's additions for new features or bug fixes.

### Routes

Finally, the `routes` layer is the outer layer of the application.

It's entirely dependent on your application, and you should not expect updates from upstream unless for fixing blatantly buggy code.

The kit splits the routes into three separate layouts:
- `__site`: the routes that are accessible to everyone, even if they are not logged in
- `__app`: the routes that are accessible only to logged-in users
- `__auth`: the routes that are accessible only to logged-in users, but only if they are not already logged in

## Import Flow

{% img width="1433" height="218" src="/assets/images/docs/nodejs-structure-imports-wrong.png" /%}

We use `eslint` to check that imports are flowing correctly through your application: if this feds you up, you can remove them from the `eslint` configuration at `/.eslintrc.json`;



-----------------
FILE PATH: kits/remix-fire/architecture/data-model.mdoc

---
status: "published"

title: The MakerKit Data Model for your SaaS project built with Remix and Firebase
label: Data Model
order: 8
description: 'Learn how MakerKit
defines the model of your SaaS application with Remix and Firebase'
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


-----------------
FILE PATH: kits/remix-fire/authentication/app-check.mdoc

---
status: "published"

title: Protect your Remix application from abuse using Firebase AppCheck
label: Prevent abuse with AppCheck
order: 9
description: 'Learn how to set up Firebase AppCheck in your MakerKit application to protect your application from abuse'
---


App Check is a Firebase service that helps you to protect your web app from
bots, spammy users, and general abuse detected by Google's Recaptcha.

The Makerkit SaaS starter is already configured to secure your application with Firebase AppCheck, but you will need to set it up in the Firebase Console first.

### Registering your website for Recaptcha v3

[To register your website for Recaptcha v3, you need to complete the linked form](https://www.google.com/recaptcha/admin/create). After entering the details, you'll be provided with a `Site Key`, which we use to initialize the Firebase AppCheck service.

{% img src="/assets/images/posts/create-recaptchha-key.webp" width="1494" height="878" /%}

NB: when registering the website, create a **Recaptcha v3** key, not v2.

After submitting the form, you will receive two keys:

1. a public key (on the client side)
2. a private key (to be used on the server side)

Save the public key and add it to your Remix `.env` file:

```
APPCHECK_KEY=<YOUR_KEY>
```

### Enabling Firebase App Check from the console

First, we need to enable Firebase App Check from the Firebase Console to register our application using the keys we have generated in the previous step.

{% img src="/assets/images/posts/register-app-check.webp" width="2850" height="902" /%}

Above, you should have added the secret key that we have generated. If it all went well, you have successfully enabled app-check for your application 🎉!

#### Enforcing Firestore to use App Check

To enforce Firestore to always validate requests using AppCheck, you must enable it from the Console. From where you are, click on the `API` tab, then click on `Firebase Firestore`. You will then be presented with the popup as in the image below:

{% img src="/assets/images/posts/enforce-appcheck-firestore.webp" width="1752" height="1340" /%}

Continue and enforce Firestore to use AppCheck.

### Testing AppCheck

AppCheck will not work in emulator mode, so you have two options to ensure it's working as intended:

- run a production build locally
- or simply deploy your applications


-----------------
FILE PATH: kits/remix-fire/authentication/auth-flow.mdoc

---
status: "published"

title: Firebase and Remix authentication flow
label: Auth Flow
order: 2
description: 'Learn how MakerKit handles the authentication flow with
Firebase Auth and Remix'
---


If you use SSR, the flow is slightly different from what you may be
used.

Typically, Firebase allows signing users in on the client-side. If you use SSR,
we need a way to authenticate the user on the server-side.

To do so, we use Firebase Auth's session cookies: we create the cookie when
the user signs in and then destroy it when the user signs out.

### Step 1: User signs in using the client SDK

The user can sign in (or up) using the Firebase Auth client SDK from the client side.

### Step 2: Creating a Session Cookie using an API endpoint

After being authenticated (signing in or up), the application requests to create and store a session cookie using an API endpoint.

The cookie is stored as an HTTP-only cookie and will be sent to the server,
so we can authenticate users right when we render the page.

When needed, we can perform the necessary redirects and security checks server-side.

### Step 3: User gets redirected to Application Home Page

The application renders a session-aware page if you have opted to use SSR on every page. Otherwise, the Firebase SDK should reflect the current session once the page has been rendered.

When the user is redirected to a protected page, the user's session authentication state is checked in the `loader` function of the `__app` layout.

### Step 4: User signs out

The client SDK starts a sign-out request when the user decides to sign out.

When Firestore Auth destroys the current user session, the event is intercepted by a listener and then calls the API endpoint to destroy the session cookie.

Here is an illustration of the flows:

{% img src="/assets/images/docs/auth-flow.png" width="2344" height="1166" /%}

## Keeping the client and server sessions in sync

Because the Firebase Session cookies expire after 14 days, we need to keep the client SDK session in sync when the session cookie expires.

To do so, when the user signs in, we also store a cookie with the expiration date. When the client SDK detects that the server-side session cookie expired, the user gets automatically signed out.


