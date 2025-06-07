-----------------
FILE PATH: kits/next-supabase/tutorials/translations.mdoc

---
status: "published"

title: "Translations"
label: "Translations"
order: 14
description: "Learn how to easily translate your Next.js Supabase Makerkit SaaS into multiple languages with our guide. Optimize your app and reach a wider audience."
---


Most strings in the Makerkit template's application use `i18next`, a
library to translate your application into multiple languages. Thanks to
this library, we can store all the application's text in `json` files for each supported language.

For example, if you are serving your application in English and Spanish, you'd have the following files:

- **English** translation files at `/public/locales/en/{file}.json`
- **Spanish** translation files at `/public/locales/es/{file}.json`

There are valid reasons to use translation files, even if you are not planning on translating your application, such as:

1. it's easier to search and replace text
2. tidier render functions
3. easy update path if you do decide to support a new language

### Adding new languages

By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:

1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **Next.js i18n config**: We need to also add a new language to the Next.js i18n
configuration at `src/i18n/i18next.settings.ts`.

Simply add your new language's code to the `locales` array.

The configuration will look like the below:

 ```js {% title="app/i18n/i18next.settings.ts" %}
const fallbackLng = process.env.DEFAULT_LOCALE ?? 'en';
const languages: string[] = [fallbackLng, 'es', 'fr', 'it'];
```

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```

### Translating Server Components strings

To translate the server components strings, you need to initialize i18n in your Server Component using the following function:


```tsx
const i18n = await initializeServerI18n(getLanguageCookie());
```

You can find examples of how we do it in every `layout` file.

### Translating Client Components strings

When translating client components, you need to ensure you:

1) Import the `i18nProvider` component and set the language provided from the server component
2) Use the `Trans` component from `~/core/ui/Trans`. An eslint rule will ensure you do not import it from `react-i18next`

For example, here is the `SiteLayout` layout:

```tsx
function SiteLayout(props: React.PropsWithChildren) {
  const data = use(loadUserData());

  return (
    <I18nProvider lang={data.language}>
      <AuthChangeListener accessToken={data.accessToken}>
        <SiteHeaderSessionProvider data={data} />

        {props.children}

        <Footer />
      </AuthChangeListener>
    </I18nProvider>
  );
}
```


-----------------
FILE PATH: kits/next-supabase/tutorials/updating-latest-version.mdoc

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
FILE PATH: kits/next-supabase/tutorials.mdoc

---
status: "published"
title: "Next.js Supabase (legacy) Tutorials"
label: "Tutorials"
order: 15
description: "Tutorials for the Next.js Supabase (legacy) kit"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/ui/shadcn.mdoc

---
status: "published"
title: Shadcn UI | Next.js Supabase
label: Shadcn UI
order: 1
description: 'Learn how to add new components from Shadcn UI to your project.'
---

ShadCN UI is a template library based on Radix UI and Tailwind CSS. It provides a set of high-quality React components out of the box that are fully accessible, customizable, and themable.

Makerkit was an early adopter of ShadCN UI, and we use it to build our own UI components. With that, we used very different styling conventions early on, which means copy/pasting components from our docs to your project might not work as expected.

ShadCN UI is **configured** in all Makerkit projects as of the latest versions, while versions released before 28 September 2023 will not work seamlessly.

Makerkit only ships the components it uses in the core kit: to use other components, you will need to either use the CLI or import them manually.

## Theming

### Updating the Colors in CSS

To update the theme's colors, please update the `global.css` file in your project and update the CSS variables.

For examples, check out the [ShadCN UI Themes Page](https://ui.shadcn.com/themes).

### Updating the colors in the Tailwind configuration

Additionally, we need to update the Tailwind configuration: since Makerkit uses a color palette based on Tailwind's palette system, we need to update the `tailwind.config.js` file in your project and update the `colors` object.

For example, since the default color for the `primary` color is `violette.500`, we need to update the `colors` object to:

```js
primary: {
  DEFAULT: 'hsl(var(--primary))',
  foreground: 'hsl(var(--primary-foreground))',
  ...colors.violet,
}
```

The code above spreads the `violet` color palette into the `primary` object, which means that the `primary` color will be the `violet.500` color.

The variable `--primary` is the hsl value of the `violet.500` color.

Additionally, we can access other colors of the `primary` palette using `primary-500`, `primary-600`, etc. This helps us to access the color palette in a more semantic way.

### Dark Mode

We need to do the same for the dark mode. Makerkit has historically used the `dark` color palette convention to define the dark mode colors.

By default, Makerkit uses `slate` as the dark mode color palette, so we need to update the `colors` object to:

```js
dark: {
  ...colors.slate,
  DEFAULT: colors.slate[950],
  foreground: colors.slate[100],
}
```

As you can see, we spread the `slate` color palette into the `dark` object, which means that the `dark` color will be the `slate.950` color.

If you were to use another color such as `zinc`, you would need to update the `colors` object to:

```js
dark: {
  ...colors.zinc,
  DEFAULT: colors.zinc[950],
  foreground: colors.zinc[100],
}
```

## Icons

While ShadCN UI uses `lucide` as their icons library, Makerkit has historically used `heroicons`. We have configured ShadCN UI to use `heroicons` instead of `lucide` in all Makerkit projects, but for new components, you will need to use `lucide` instead if you don't want to make changes. As such, please install `lucide` as a dependency in your project.


-----------------
FILE PATH: kits/next-supabase/ui/tailwindcss.mdoc

---
status: "published"
title: Tailwind CSS | Next.js Supabase SaaS Kit
label: Tailwind CSS
order: 0
description: 'Learn how to customize Tailwind CSS in your Makerkit project.'
---


All the Makerkit kits use Tailwind CSS for styling. Tailwind is a utility-first CSS framework that allows you to build custom designs without leaving your HTML. It's a great choice for Makerkit because it allows you to customize the look and feel of your project without having to write any CSS.

The Tailwind CSS configuration file is located at `tailwind.config.js` in the root of your project.

By default, it looks like this:

 ```js {% title="tailwind.config.js" %}
const colors = require('tailwindcss/colors');

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./app/**/*.{ts,tsx,jsx,js}'],
  darkMode: 'class',
  theme: {
    extend: {
      fontFamily: {
        serif: ['serif'],
        heading: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
        sans: [
          'system-ui',
          'BlinkMacSystemFont',
          'Inter',
          'Segoe UI',
          'Roboto',
          'Ubuntu',
        ],
        monospace: [`SF Mono`, `ui-monospace`, `Monaco`, 'Monospace'],
      },
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        dark: {
          ...colors.slate,
          DEFAULT: colors.slate[950],
          foreground: colors.slate[100],
        },
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
          ...colors.violet,
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: { height: 0 },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: 0 },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

The Tailwind configuration gets extended with two additional colors:
`primary` and `dark`.

1. `primary` is the main color of your project. It's used for buttons, links, and other interactive elements. It's also used for some backgrounds of the dark mode.
2. `dark` is the color palette for the dark mode. It's used for the background of the dark mode.

By updating these colors, you can customize the look and feel of your project.

For example:
1. If you want to change the primary color, you can update the `primary` color in the Tailwind configuration. Try changing it to `blue` and see what happens.
2. If you want to change the dark mode background color, you can update the `dark` color in the Tailwind configuration. Try changing it to `slate` and see what happens.

Of course - you can also pass your own palette as the `primary` and `dark` colors.

## Usage in practice

Let's say you want to change the primary color of your project to `indigo`. and the dark mode background color to `zinc`.

 ```js {% title="tailwind.config.js" %}
const colors = require('tailwindcss/colors');

extend: {
  colors: {
    dark: {
      ...colors.zinc,
      DEFAULT: colors.zinc[950],
      foreground: colors.zinc[100],
    },
    primary: {
      DEFAULT: 'hsl(var(--primary))',
      foreground: 'hsl(var(--primary-foreground))',
      ...colors.indigo,
    }
  },
},
```

Additionally, please update the CSS variables of your global CSS file `globals.css` to match the new colors by updating the `--primary` color to match using the `hsl` value of the `indigo` color.

To have a reference of the Tailwind palette as `hsl` values, you can use [this reference from the ShadcnUI repository](https://github.com/shadcn-ui/ui/issues/669).

When you want to use the primary color in your project, you can use the `primary` color class.

 ```tsx {% title="src/components/PrimaryButton.tsx" %}
import React from 'react';

export const PrimaryButton = () => {
  return (
    <button className="bg-primary text-white dark:bg-primary/10 dark:text-primary px-4 py-2 rounded">
      Click me
    </button>
  );
};
```

-----------------
FILE PATH: kits/next-supabase/ui/themes.mdoc

---
status: "published"
title: Themes | Next.js Supabase SaaS Kit (Legacy)
label: Themes
order: 2
description: 'Learn how to configure the default theme of your application.'
---

By default, the Makerkit kits come with a light and a dark theme, and we provide a toggle to let users switch between them.

## Changing the default theme

To change the default theme, you need to update the theme in your `src/configuration.ts` file.

 ```ts {% title="src/configuration.ts" %} {2}
const configuration = {
  theme: Themes.Dark,
  ...
}
```

## Disabling the theme toggle

To disable the theme toggle, you need to update the `src/configuration.ts` file.

 ```tsx {% title="src/configuration.ts" %} {3}
const configuration = {
  theme: Themes.Dark,
  features: {
    enableThemeSwitcher: false,
  },
  ...
}
```

This will prevent the theme toggle from appearing in the application.

-----------------
FILE PATH: kits/next-supabase/ui.mdoc

---
status: "published"
title: "UI in Next.js Supabase"
label: "UI"
order: 9
description: "UI in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/admin/adding-admin-users.mdoc

---
status: "published"
title: "Adding Admin users | Next.js Supabase Lite Kit"
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
FILE PATH: kits/next-supabase-lite/admin/admin-overview.mdoc

---
status: "published"
title: "Admin Overview | Next.js Supabase Lite SaaS Kit"
label: Overview
order: 1
description: "Overview of the Super Admin panel in the Next.js Firebase kit"
---

The Super Admin was released in the version 0.2.0.

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
FILE PATH: kits/next-supabase-lite/admin.mdoc

---
status: "published"
title: "Admin in Next.js Lite Supabase"
label: "Admin"
order: 7
description: "The Admin section of Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/architecture/application-structure.mdoc

---
status: "published"
title: Structure your Makerkit Application | Next.js Supabase Lite
label: Structure your Application
order: 7
description: 'Learn how to structure your application with additional entities and business logic in your Next.js Supabase Lite kit'
canonical: '/docs/next-supabase/application-structure'
---

In the previous section, we learned the fundamentals of Makerkit's architecture and the application layers.

In this section, we learn how to structure your application in practical terms with an example. For example, your application has an entity "events": how do we add this entity to a Makerkit application?

{% alert type="info" title="Entities are added to core" %}
NB: entities rarely (or never) get added to "core". Business domain is added to "components" and "lib".
{% /alert %}

In short, this is how we add a new entity to the application:

1. First, we add a new folder to `lib`. If the entity is "event", we add `lib/events`.
2. Then, we add the components of the `event` domain to `(app)/events/components`
3. Finally, we add the pages of the `event` domain to `(app)/events`

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
    - events
      components
        - events
          - EventsContainerComponent.tsx
          - ...
      - page.tsx
      - [event].tsx
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
FILE PATH: kits/next-supabase-lite/architecture/architecture.mdoc

---
status: "published"

title: Architecture and Folder Structure for your Next.js SaaS application
label: Architecture and Folder Structure
order: 6
description: 'Learn how MakerKit allows you to build a Next.js application
with a scalable and production-grade architecture and folder structure'
canonical: '/docs/next-supabase/architecture'
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
- `(app)`: internal routes that are only accessible to authenticated users
- `auth`: they auth routes, only accessible to unauthenticated users

#### Websites routes - (site)

If you are building a SaaS, you will probably have a marketing website. This is the place where you can put your landing page, pricing, and other marketing-related pages.

#### Auth routes - auth

The auth routes are the routes that are accessible to unauthenticated users. This is where your users can sign up, log in, and reset their password.

If a user is already logged in, they will be redirected to the dashboard.

#### App routes - (app)

The `(app)` routes are the routes that are only accessible to authenticated users. This is where you can put your dashboard, settings, and other internal pages.

#### Domain Components

Domain components are co-located together with their routes. This means that you can easily find the components that are used in a particular route.

For example, you find the authentication components at `/app/auth/components`.

## Import Flow

{% img width="1433" height="218" src="/assets/images/docs/nodejs-structure-imports-wrong.png" /%}

We use `eslint` to check that imports are flowing correctly through your application: if this feds you up, you can remove them from the `eslint` configuration at `/.eslintrc.json`;



-----------------
FILE PATH: kits/next-supabase-lite/architecture/data-model.mdoc

---
status: "published"
title: The MakerKit Data Model for your SaaS project built with Next.js and Supabase Lite Kit
label: Data Model
order: 8
description: 'Learn how MakerKit defines the model of your SaaS application with Next.js and Supabase'
---

MakerKit's data model is voluntarily as simple as possible, with a limited set of assumptions.

MakerKit is not supposed to be a finite product but a solid foundation on which it is quick to get started and build your SaaS product.

To summarize, the Postgres Database model contains the following tables: `users`, `subscriptions` and `users_subscriptions`.

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
  display_name text
);
```

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
create table users_subscriptions (
  user_id uuid not null references public.users (id) on delete cascade,
  subscription_id text unique references public.subscriptions (id) on delete set null,
  customer_id text not null unique,
  primary key (user_id)
);
```

We store the `customer_id` property so that, when a user is first linked to a Stripe customer, can access their Stripe data using the Billing Portal.

When a subscription is active, the `subscription_id` field will be populated with the subscription ID. Otherwise, the field will be `null`.

## Storage

By default, the kit also creates one Storage bucket: the user's `avatars`:

```sql
insert into storage.buckets (id, name, PUBLIC)
  values ('avatars', 'avatars', true);
```

