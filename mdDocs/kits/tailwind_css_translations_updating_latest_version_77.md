-----------------
FILE PATH: kits\remix-fire\tutorials\tailwind-css.mdoc

---
status: "published"
title: "Tailwind CSS and Styling | Remix.js Firebase SaaS Starter Kit"
label: Tailwind CSS and Styling
order: 6
description: "Learn how to configure Tailwind CSS and tweak the Firebase Remix SaaS Kit theme."
---

This SaaS template uses Tailwind CSS for styling the application.

You will likely want to tweak the brand color of your application: to do so, open the file `tailwind.config.js` and replace the `primary` color (which it's `indigo` by default):

 ```js {% title="tailwind.config.js" %}
extend: {
  colors: {
    primary: {
      ...colors.indigo,
      contrast: '#fff',
    },
    black: {
      50: '#525252',
      100: '#363636',
      200: '#282828',
      300: '#222',
      400: '#121212',
      500: '#0a0a0a',
      600: '#040404',
      700: '#000',
    },
  },
},
```

### Updating the Primary color of your app

To update the `primary` color, you can either:

1. choose another color from the Tailwind CSS Color palette (for example, try `colors.blue`)
2. define your own colors, from `50` to `900`

The `contrast` color will be helpful to define the text color of some components, so tweaking it may be required.

Once updated, the Makerkit's theme will automatically adapt to your new color palette! For example, below, we set the primary color to `colors.blue`:

{% img src="/assets/images/posts/blue-dark-theme.webp" width="2842" height="1966" /%}

### Dark Mode

Makerkit also ships with a dark theme.

The light theme is the default, but users can switch to the other, thanks to the component `DarkModeToggle`.

If you wish to only use one theme, you can tweak the configuration using the following flag in the configuration file `~/configuration.ts`:

 ```ts {% title="~/configuration.ts" %}
{
  ...
  features: {
    enableThemeSwitcher: false,
  },
  ...
}
```

## Caveats

1. Setting the dark theme by default is currently not supported without some minor tweaks. It is being developed.
2. The default Tailwind media switcher is not supported since Makerkit uses its own system that allows users to choose the System theme (light or dark) or the theme defined by the application.

-----------------
FILE PATH: kits\remix-fire\tutorials\translations.mdoc

---
status: "published"

title: "Translations"
label: "Translations"
order: 14
description: "Learn how to easily translate your Remix Firebase Makerkit SaaS into multiple languages with our guide. Optimize your app and reach a wider audience."
---


By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:

1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **Remix i18n config**: We need to also add a new language to the Remix
configuration at `app/i18n/i18next.config.ts`. Simply add your new
language's code to the `supportedLanguages` array.

The configuration will look like the below:

 ```js {% title="app/i18n/i18next.config.ts" %}
const i18Config = {
  fallbackLanguage: DEFAULT_LOCALE,
  supportedLanguages: [DEFAULT_LOCALE, 'es'],
  defaultNS: ['common', 'auth', 'organization', 'profile', 'subscription'],
  react: { useSuspense: false },
};
```

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```


-----------------
FILE PATH: kits\remix-fire\tutorials\updating-latest-version.mdoc

---
status: "published"

title: "Updating to the latest version"
label: "Updating to the latest version"
order: 17
description: "Learn how to update your Makerkit repository to the latest version."
---


If you have followed the steps to set up Git at the beginning of this tutorial, you've already set the original Makerkit repository as `upstream`.

As the repository is constantly updated, it's recommended to fetch updates regularly. You can do so by running the following Git command:

```
git pull upstream main --allow-unrelated-histories
```

Unfortunately, you may need to resolve eventual conflicts.

### Any questions?

If you have questions, please join our [Discord Community](https://discord.gg/BeJSyQJ6jP), even if you have not purchased any kit. We chat and help each other build products.


-----------------
FILE PATH: kits\remix-fire\utilities\hooks.mdoc

---
status: "published"

title: Reusable React Hooks
label: React Hooks
order: 1
description: Reusable React Hooks utilities for common tasks
---


Makerkit uses various React Hooks for very common tasks, such as a simple
reducer for asynchronous tasks, or for executing HTTP requests.

## useRequestState

You can use this hook as a simple reducer for asynchronous tasks, like HTTP
requests or asynchronous operations with Firestore.

To create the hook, you can use the following snippet:

```tsx
const { state, setError, setData, setLoading } =
    useRequestState<Data>();
```

This hooks returns the current state `state` and the methods to update it.
The `state` variable has the following interface:

```tsx
type State<Data, ErrorType> =
  | {
      data: Data;
      loading: false;
      success: true;
      error: Maybe<ErrorType>;
    }
  | {
      data: undefined;
      loading: true;
      success: false;
      error: Maybe<ErrorType>;
    }
  | {
      data: undefined;
      loading: false;
      success: false;
      error: Maybe<ErrorType>;
    };
```

## useApiRequest

You can use this hook to perform HTTP requests from your UI to your API and
uses `useRequestState` under the hood for managing the state.

The function has the following interface:

```tsx
function useApiRequest<Resp = unknown, Body = void>(
  path: string,
  method: HttpMethod = 'POST'
)
```

For example, if we wanted to make an API request to an endpoint, we could do
the following:

```tsx
function SignIn() {
  const [signIn, signInState] = useApiRequest(`/api/sign-in`);

  useEffect(() => {
    if (signInState.success) {
      window.location.href = '/dashboard';
    }
  }, [signInState]);

  if (signInState.loading) {
    return <span>Loading...</span>
  }

  return (
    <form onSubmit={() => signIn()}></form>
  );
}
```


-----------------
FILE PATH: kits\remix-supabase\admin\adding-admin-users.mdoc

---
status: "published"
title: "Adding Admin users | Remix Supabase Kit"
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

## Setting your user as Super Admin

To set a user's custom claims against a production environment, we need to run an SQL query from the Supabase dashboard and set the correct claims for your user.

1. **Navigate to the SQL Query runner**: From the Supabase dashboard, navigate to the SQL Query page - where we can run arbitrary commands against our Postgres Database.
2. **Grab the ID of your user**: Now, copy this code below in a text editor and replace `<user_id>` with your real user's ID.

```sql
UPDATE auth.users SET raw_app_meta_data = raw_app_meta_data || '{"role": "super-admin"}' WHERE id='<user_id>';
```

Once copied the full query with your correct user ID, run the query in the Supabase query runner. Your user is now a super-admin and can access the Super Admin panel in your app.

Your user may need to log out and log in again to see the changes.

**Please test this in a staging environment before running this in production.**

### Enforce MFA

For added security, you can enforce the admin to be signed out using MFA by toggling the constant `ENFORCE_MFA` to from `false` to `true`.

 ```tsx {% title="app/lib/admin/utils/assert-is-super-admin.ts" %}
const ENFORCE_MFA = true;
```

-----------------
FILE PATH: kits\remix-supabase\admin\admin-overview.mdoc

---
status: "published"
title: "Admin Overview | Remix Supabase SaaS Kit"
label: Overview
order: 1
description: "Overview of the Super Admin panel in the Remix Supabase kit"
---

The Super Admin was released in the version 0.6.0.

It allows your administrators to manage your users and organizations - such as:

1. Viewing the app's users and their details
2. Viewing the app's organizations
3. Viewing an organization's members
4. Banning or Disabling users
5. Impersonating users

## Accessing the Super Admin

To access the super admin, you need to navigate to the path `/admin`. If you're using the emulator, you can do so using the `test@makerkit.dev` account, which is set to having a super admin role.

You can also find the link in the profile dropdown of users with admin privileges.

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
FILE PATH: kits\remix-supabase\admin.mdoc

---
status: "published"
title: "Admin in Remix Supabase"
label: "Admin"
order: 7
description: "The Admin section of Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\remix-supabase\architecture\application-structure.mdoc

---
status: "published"
title: Structure your Makerkit Application | Remix Supabase SaaS Kit
label: Structure your Application
order: 7
description: 'Learn how to structure your application with additional entities and business logic in your Remix Supabase SaaS Kit'
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
    - _app.events._index.tsx
    - _app.events.$event.tsx
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

For example, let's write a custom hook that retrieves a list of "events" from a Postgres table.

We create a file at `lib/events/hooks/use-fetch-events.ts` with the following content:

 ```tsx {% title="src/lib/events/hooks/use-fetch-events.ts" %}
export function useFetchEvents(
  organizationId: number
) {
  const client = useSupabase();
  const key = [`events`, organizationId];

  return useQuery(key, async () => {
    const { data, error } = client
      .from('events')
      .select('*')
      .eq('organization_id', organizationId);

    if (error) {
      throw error;
    }

    return data;
  })
}
```

Good! We have a way to fetch our events, but we have to use it somewhere: to do so, let's create a component `EventsListContainer`. 

{% alert type="warning" %}
NB: remember to update add the required Row Level Security policies to protect your tables.
{% /alert %}

### 2) Components

As said before, we add React components that belong to the "events" domain to `components/events`. 

In the component below, we will fetch a list of events with `useFetchEvents`:

 ```tsx {% title="components/events/EventsListContainer.tsx" %}
import { useFetchEvents } from '~/lib/events/hooks/use-fetch-events';
import Alert from '~/core/ui/Alert';

const EventsListContainer: React.FC<{
  organizationId: number
}> = ({ organizationId }) => {
  const { data: events, isLoading, error } = useFetchEvents(organizationId);

  if (isLoading) {
    return <p>Loading Events...</p>
  }

  if (error) {
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

 ```tsx {% title="routes/_app.events._index.tsx" %}
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
FILE PATH: kits\remix-supabase\architecture\architecture.mdoc

---
status: "published"
title: Architecture and Folder Structure for your Remix SaaS application
label: Architecture and Folder Structure
order: 6
description: 'Learn how MakerKit allows you to build a Remix application
with a scalable and production-grade architecture and folder structure'
collapsible: true
collapsed: true
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
- `_site`: the routes of the marketing site
- `_app`: the routes of the internal app
- `auth`: the routes of the authentication pages



-----------------
FILE PATH: kits\remix-supabase\architecture\data-model.mdoc

---
status: "published"

title: The MakerKit Data Model for your SaaS project built with Remix and Supabase
label: Data Model
order: 8
description: 'Learn how MakerKit
defines the model of your SaaS application with Remix and Supabase'
---


MakerKit's data model is voluntarily as simple as possible, with a limited
set of assumptions.

MakerKit is not supposed to be a finite product but a solid foundation on which it is quick to get started and build your SaaS product.

To summarize, the Postgres Database model contains the following tables: `users`, `organizations`, `memberships`, `subscriptions` and `organizations_subscriptions`.

## Users

Users are, of course, foundational to any application.

If a user signed in using Supabase Authentication, it merely means we have
verified their credentials, not that they have an account.

We have to create an additional collection to store a user's data, such as:
- names (first name, last name)
- profile photo
- settings
- any other information which is not possible to update using their Authentication record (email, and sign-in providers)

Below is the `users` table:

```sql
create table users (
  id uuid references auth.users not null primary key,
  photo_url text,
  display_name text,
  onboarded bool not null
);
```

## Organizations

`Organizations` are groups of users.

If the name doesn't suit your domain, don't worry: you can always change the terminology using the localization files.

Below is what the `Organizations` schema looks like:

```sql
create table organizations (
  id bigint generated always as identity primary key,
  name text not null,
  logo_url text
);
```

## Memberships

To link an user to an organization we use another table named `memberships`.

A user is associated to this table when:
1. it is part of the organization
2. it is not yet part, but has been invited

Furthermore, in this table we store the `role` property which defines the role of the user for the organization.

Below is what the `Organizations` schema looks like:

```sql
create table memberships (
  id bigint generated always as identity primary key,
  user_id uuid references public.users,
  organization_id bigint not null references public.organizations,
  role int not null,
  invited_email text,
  code text,
  unique (user_id, organization_id)
);
```

### Invites

To create an invite to join an organization, we create a membership for the user with two properties `code` and `invite_email`: these two properties are going to become `null` once the invite gets accepted.

### User Roles

By default, MakerKit defines three roles: Owner (can be only one), Admin, and Member.

This enum is hierarchical and associated with a number:

```tsx
enum MembershipRole {
  Member = 0,
  Admin = 1,
  Owner = 2
}
```

An Admin can do anything a member can do, and an Owner can do anything an Admin can.

## Subscriptions

When users complete the Stripe checkout, the application will store some information from the subscription object sent by Stripe. 

Below is the schema of the information stored in the Database:

```sql
create table subscriptions (
  id text not null primary key,
  price_id text not null,
  status subscription_status not null,
  currency text,
  interval text,
  interval_count int,
  created_at timestamptz,
  period_starts_at timestamptz,
  period_ends_at timestamptz,
  trial_starts_at timestamptz,
  trial_ends_at timestamptz
);
```

To link a `customer_id` from Stripe with an organization, we use a join table `organizations_subscriptions`:

```sql
create table organizations_subscriptions (
  organization_id bigint not null references public.organizations (id) on delete cascade,
  subscription_id text unique references public.subscriptions (id) on delete set null,
  customer_id text not null unique,
  primary key (organization_id)
);
```

We store the `customer_id` property so that, when an organization is first linked to a Stripe customer, can access their Stripe data using the Billing Portal. 

When a subscription is active, the `subscription_id` field will be populated with the subscription ID. Otherwise, the field will be `null`.


## Storage

By default, the kit also creates two Storage buckets: `logos` and `avatars`:

```sql
insert into storage.buckets (id, name, PUBLIC)
  values ('logos', 'logos', true);

insert into storage.buckets (id, name, PUBLIC)
  values ('avatars', 'avatars', true);
```

-----------------
FILE PATH: kits\remix-supabase\architecture.mdoc

---
status: "published"
title: "Architecture in Remix Supabase"
label: "Architecture"
order: 8
description: "Architecture in Remix Supabase"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits\remix-supabase\authentication\auth-overview.mdoc

---
status: "published"
title: Supabase and Remix authentication with MakerKit | Remix Supabase SaaS Kit
label: Auth Overview
order: 0
description: 'MakerKit uses Supabase Authentication to allow access to your Remix application using oAuth providers and email/password.'
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
FILE PATH: kits\remix-supabase\authentication\auth-setup.mdoc

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

