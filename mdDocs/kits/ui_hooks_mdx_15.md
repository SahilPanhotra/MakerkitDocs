-----------------
FILE PATH: kits/next-fire/ui.mdoc

---
status: "published"
title: "UI in the Next.js Firebase kit"
label: UI
order: 3
description: 'Learn more about the UI in the Next.js Firebase kit'
collapsible: true
collapsed: true
---

In this section, you will learn everything you need to know about the UI in the Next.js Firebase kit.

Makerkit uses Shadcn UI and Tailwind CSS to build the UI of your SaaS - however we do also provide more custom components that you can use in your project.

- [Shadcn UI](ui/shadcn-ui): this is the guide to use the Shadcn UI components\
- [TailwindCSS](ui/tailwindcss): this is the guide to use the Tailwind CSS components
- [Themes](ui/themes): this is the guide to set up themes




-----------------
FILE PATH: kits/next-fire/utilities/hooks.mdoc

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
FILE PATH: kits/next-fire/utilities/mdx.mdoc

---
status: "published"

title: Compile MDX Documents with Makerkit
label: MDX
order: 0
description: Compile MDX Documents with Makerkit
---

MDX is one of the most common Markdown flavours for React-based websites. It
allows us to include our own components into our documents and take out
blog's interactivity to the next level.

The Makerkit starter uses MDX for writing documents in the blog and in the
documentation; it may be likely that you would want to add your own document
types and use MDX for other purposes; for example, you could replace some of
your HTML with MDX documents if you felt like for making it easier to change
by non-technical teammates.

In this section, we will explain how to compile MDX content with the
Makerkit's utilities.

## 1) Compiling MDX to JS

The first step is to compile MDX to Javascript.

To do so, compile your MDX files with the following utility:

```tsx
import { compileMdx } from '~/core/generic';

// MDX string
const content = '';

// JS representation of your MDX content
const compiled = compileMdx(content);
```

## 2) Pass the compiled MDX file into the MDXRenderer component

Finally, we import the `MDXRenderer` component that takes the string
compiled with the `compileMDX` utility and renders it as `HTML`.

```tsx
import MDXRenderer from '~/components/blog/MDXRenderer';

function ArticleBody(props: React.PropsWithChildren<{ content: string }>) {
  return (
    <div>
      <MDXRenderer code={content} />;
    </div>
  );
}
```

Under the hood, the component `MDXRenderer` will inject a couple of
custom components we use in the documents: these can be viewed and
edited at `~/components/blog/MDXComponents`.

## Custom MDX Components

### NextImage

The `NextImage` component replaces the default `img` HTML tag and adds the
awesome capabilities of the default `Image` component baked in Next.js.

You have 2 ways to use this:
1. Simply use an image tag as you would normally
2. Use `NextImage` as a component, which is better if you have to pass
custom attributes such as `width` and `height`

## ExternalLink

All links are automatically patched with the following attributes:
`target="_blank"` and `rel="noopener noreferrer"` for security and UX reasons.

## Video

To embed videos in your blog and docs, simply use the `Video` component:

```tsx
<Video src={"/path/to/video.mp4"} />
```

### Other UI Components

Additionally, we also imported some components from `~/core/ui` by default
that you can use in any MDX document.


-----------------
FILE PATH: kits/next-fire/utilities.mdoc

---
status: "published"
title: Utilities in Next.js Firebase
label: Utilities
order: 10
description: Utilities in Next.js Firebase
collapsible: true
collapsed: true
---

In this section,

-----------------
FILE PATH: kits/next-supabase/admin/adding-admin-users.mdoc

---
status: "published"
title: "Adding Admin users | Next.js Supabase Kit"
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

 ```tsx {% title="src/app/admin/utils/is-user-super-admin.ts" %}
const ENFORCE_MFA = true;
```

-----------------
FILE PATH: kits/next-supabase/admin/admin-overview.mdoc

---
status: "published"
title: "Admin Overview | Next.js Supabase SaaS Kit"
label: Overview
order: 1
description: "Overview of the Super Admin panel in the Next.js Firebase kit"
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
FILE PATH: kits/next-supabase/admin.mdoc

---
status: "published"
title: "Admin in Next.js Supabase"
label: "Admin"
order: 7
description: "The Admin section of Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/architecture/application-structure.mdoc

---
status: "published"
title: Structure your Makerkit Application | Next.js Supabase Lite SaaS Kit
label: Structure your Application
order: 7
description: 'Learn how to structure your application with additional entities and business logic in your Next.js Supabase Lite SaaS Kit'
---

In the previous section, we learned the fundamentals of Makerkit's architecture and the application layers.

In this section, we learn how to structure your application in practical terms with an example. For example, your application has an entity "events": how do we add this entity to a Makerkit application?

{% alert type="info" %}
NB: entities rarely (or never) get added to "core". Business domain is added to "components" and "lib".
{% /alert %}

In short, this is how we add a new entity to the application:

1. First, we add a new folder to `lib`. If the entity is "event", we add `lib/events`.
2. Then, we add the components of the `event` domain to `app/dashboard/[organization]/events/components`
3. Finally, we add the pages of the `event` domain to `app/dashboard/[organization]/events`

 ```txt {% title="Structure" %}
- app
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

  - app
    - dashboard
      - [organization]
        - events
          components
            - events
              - EventsContainerComponent.tsx
              - ...
          - page.tsx
          - $event.tsx
```

NB: In the Next.js App Directory, you can also add the `lib` folders within the `app` folder. For example, you can add `app/dashboard/[organization]/events/lib` instead of `lib/events`.

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

Adding interfaces is optional, but it's a good practice to define the shape of your data.

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
    const { data, error } client
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
import Alert from `~/core/ui/Alert`;

const EventsListContainer: React.FC = () => {
  const { data: events, isLoading, error } = useFetchEvents();

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

Finally, we can add the events component `EventsListContainer` to a page. To
do so, let's create a page component at `app/dashboard/[organization]/events/page.tsx`:

 ```tsx {% title="app/dashboard/[organization]/events/page.tsx" %}
import EventsListContainer from '~/app/dashboard/[organization]/events/components/EventsListContainer';

const EventsPage: React.FC = () => {
  return (
     <EventsListContainer />
  );
};

export default EventsPage;
```

🎉 That's it! We have now built a nicely structured "events" domain.

-----------------
FILE PATH: kits/next-supabase/architecture/architecture.mdoc

---
status: "published"

title: Architecture and Folder Structure for your Next.js SaaS application
label: Architecture and Folder Structure
order: 6
description: 'Learn how MakerKit allows you to build a Next.js application
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
  - (site)
  - (app)
    - dashboard
      - [organization]
  - auth
  - invite
- lib
- components
```

These folders represent four separate layers. MakerKit is an Onion:

{% img width="2103" height="1221" src="/assets/images/docs/architecture.png" alt="MakerKit's Next.js Architecture" /%}

### Core
Let's talk about the lower level: the `core`. This layer is a collection of building blocks, utilities, and APIs that make up MakerKit.

This part of the boilerplate is configurable and makes no assumptions about your domain.

You are still welcome to change this code to suit your needs, but you can expect most updates from upstream to change the code in this directory.

### Lib and Shared Components

The mid-level is in two separate folders (to avoid excessive nesting):

- `lib`: here is where we write most domain-related business logic (hooks, contexts, queries, mutations, etc.)
- `components`: here is where we write components shared across the application

This layer can still be updated by the `upstream` repository, but you should not expect too many changes unless it's additions for new features or bug fixes.

### Routes

Finally, the `app` layer is the outer layer of the application.

It's entirely dependent on your application, and you should not expect updates from upstream unless for fixing blatantly buggy code.

The kit splits the routes into three separate layouts:
- `(site)`: the routes that are accessible to everyone, even if they are not logged in
- `(app)/dashboard`: the routes that are only accessible to authenticated users
- `(app)/dashboard/[organization]`: routes that are organization-specific and accessible only to authenticated users that are part of that organization
- `auth`: they auth routes, only accessible to unauthenticated users

Assuming you wanted to add user-specific routes (and not org-specific), my recommendation would be to place them under `dashboard/`.

#### Websites routes - (site)

If you are building a SaaS, you will probably have a marketing website. This is the place where you can put your landing page, pricing, and other marketing-related pages.

#### Auth routes - auth

The auth routes are the routes that are accessible to unauthenticated users. This is where your users can sign up, log in, and reset their password.

If a user is already logged in, they will be redirected to the dashboard.

#### Dashboard routes

The dashboard is the place where your users will spend most of their time. It's the place where they can manage their account, invite new users, and perform other actions.

The `dashboard` index page is a page where a user can select an organization if they have not selected one yet.

All the other routes under `dashboard/[organization]` organization-specific. This means that they are only accessible to users that are part of that organization.

#### Domain Components

Domain components are co-located together with their routes. This means that you can easily find the components that are used in a particular route.

For example, you find the authentication components at `/app/auth/components`.

-----------------
FILE PATH: kits/next-supabase/architecture/data-model.mdoc

---
status: "published"

title: The MakerKit Data Model for your SaaS project built with Next.js and Supabase
label: Data Model
order: 8
description: 'Learn how MakerKit
defines the model of your SaaS application with Next.js and Supabase'
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
  uuid uuid not null default uuid_generate_v4(),
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
FILE PATH: kits/next-supabase/architecture.mdoc

---
status: "published"
title: "Architecture in Next.js Supabase"
label: "Architecture"
order: 8
description: "Architecture in Next.js Supabase"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits/next-supabase/authentication/auth-overview.mdoc

---
status: "published"
title: Setting up Supabase and Next.js authentication with MakerKit | Next.js Supabase SaaS Kit
label: Auth Overview
order: 0
description: 'MakerKit uses Supabase Authentication to allow access to your Next.js application using oAuth providers and email/password.'
---

MakerKit uses Supabase to manage authentication within your application.

By default, every kit comes with the following built-in authentication methods:
- **Email/Password** - we added, by default, the traditional way of signing in
- **Third Party Providers** - we also added by default Google Auth sign-in
- **Email Links**
- **Phone Number**
- **Email OTP**

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

{% alert type="warning" title="Enable and Configure Authentication Methods in Supabase" %}


  Remember that you will always need to enable and configure the authentication
  methods you want to use in your Supabase project. Failure to do so means that your users will not be able to sign in.
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

