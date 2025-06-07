-----------------
FILE PATH: kits/remix-supabase/tutorials/translations.mdoc

---
status: "published"

title: "Translations"
label: "Translations"
order: 18
description: "Learn how to easily translate your Remix Firebase Makerkit SaaS into multiple languages with our guide. Optimize your app and reach a wider audience."
---


Most strings in the Makerkit template's application use `remix-i18next`, a
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
FILE PATH: kits/remix-supabase/tutorials/updating-latest-version.mdoc

---
status: "published"

title: "Updating to the latest version"
label: "Updating to the latest version"
order: 19
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
FILE PATH: kits/remix-supabase/tutorials.mdoc

---
status: "published"
title: "Remix Supabase (legacy) Tutorials"
label: "Tutorials"
order: 15
description: "Tutorials for the Remix Supabase (legacy) kit"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/ui/shadcn.mdoc

---
status: "published"
title: Shadcn UI | Remix Supabase
label: Shadcn UI
order: 1
description: 'Learn how to add new components from Shadcn UI to your project.'
---


ShadCN UI is a template library based on Radix UI and Tailwind CSS. It provides a set of high-quality React components out of the box that are fully accessible, customizable, and themable.

Makerkit was an early adopter of ShadCN UI, and we use it to build our own UI components. With that, we used very different styling conventions early on, which means copy/pasting components from our docs to your project might not work as expected.

ShadCN UI is **configured** in all Makerkit projects as of the latest versions, while versions released before 28 September 2023 will not work seamlessly.

Makerkit only ships the components it uses in the core kit: to use other components, you will need to either use the CLI or import them manually.

## Theming

### Updating the colors in the CSS

To update the theme's colors, please update the `index.css` file in your project and update the CSS variables.

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
FILE PATH: kits/remix-supabase/ui/tailwindcss.mdoc

---
status: "published"
title: Tailwind CSS | Remix Supabase SaaS Kit
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

Additionally, please update the CSS variables of your global CSS file `index.css` to match the new colors by updating the `--primary` color to match using the `hsl` value of the `indigo` color.

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
FILE PATH: kits/remix-supabase/ui/themes.mdoc

---
status: "published"
title: Themes | Remix.js Supabase SaaS Starter Kit (Legacy)
label: Themes
order: 2
description: 'Learn how to configure the default theme of your Remix.js Supabase application.'
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
FILE PATH: kits/remix-supabase/ui.mdoc

---
status: "published"
title: "UI in Remix Supabase"
label: "UI"
order: 9
description: "UI in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase-turbo/admin/adding-super-admin.mdoc

---
status: "published"

label: "Adding a Super Admin"
title: "Adding a Super Admin to your Remix Supabase application"
description: "In this post, you will learn how to set up a Super Admin in your Remix Supabase application"
order: 0
---


The Super Admin panel allows you to manage users and accounts.

To access the super admin panel at `/admin`, you will need to assign a user as a super admin.

To do so, pick the user ID of the user you want to assign as a super admin and run the following SQL query:

```sql
UPDATE auth.users SET raw_app_meta_data = raw_app_meta_data || '{"role": "super-admin"}' WHERE id='<user_id>';
```

Please replace `<user_id>` with the user ID you want to assign as a super admin.

-----------------
FILE PATH: kits/remix-supabase-turbo/admin.mdoc

---
status: "published"
label: "Super Admin"
title: "Adding a Super Admin to your Remix Supabase application"
description: "The Super Admin panel allows you to manage users and accounts."
order: 13
collapsible: true
collapsed: true
---

The Admin section of Makerkit provides a powerful set of tools for managing your SaaS application. As your application grows, you'll need ways to manage users, monitor subscriptions, and keep things running smoothly. Let's explore what's available in the Admin dashboard.

## What Can You Do in the Admin Panel?

The Admin dashboard lets you:
- View and manage all user accounts
- Monitor subscriptions and billing
- Track team accounts
- Manage user access and permissions

## Accessing the Admin Panel

To use the Admin features, you'll need to set up a Super Admin user. Head over to your Supabase SQL editor and run:

```sql
UPDATE auth.users
SET raw_app_meta_data = raw_app_meta_data || '{"role": "super-admin"}'
WHERE id='<user_id>';
```

Replace `<user_id>` with the ID of the user you want to make a Super Admin. Once done, you can access the Admin panel at `/admin`.

## Key Features

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
- Team members (for team accounts)
- Billing information

### Actions

As an admin, you can:
- Ban accounts to restrict access
- Delete accounts when necessary
- View detailed account information
- Monitor subscription status

-----------------
FILE PATH: kits/remix-supabase-turbo/analytics/analytics-and-events.mdoc

---
status: "published"

title: 'Understanding Analytics and Events in the Remix Supabase Turbo kit'
label: 'Analytics and Events in Makerkit'
description: 'Learn how to use the Analytics and App Events systems in Makerkit to track user behavior and app-wide occurrences.'
order: 0
---


Makerkit provides two powerful, interconnected systems for tracking user behavior and app-wide occurrences: **Analytics** and **App Events**.

While these systems are separate, they are designed to work seamlessly together, providing a flexible and maintainable approach to event tracking and analysis in your SaaS application.

One doesn't need the other to function, but they are designed to work together. Here's a brief overview of each system.

## Analytics Providers

The Analytics system is implemented in the `@kit/analytics` package, and is abstracted to allow for easy integration with various analytics services, and not lock you into a specific provider.

The implementations are made possible using Makerkit Plugins, which means you can install them using the normal plugins system. By default, Makerkit provides three analytics providers: Google Analytics, Umami, and PostHog.

However, should you prefer different providers than the ones provided by default, you can easily create your own custom analytics provider.

[Read more about creating a custom analytics provider](custom-analytics-provider).

## Understanding the Relationship Between Analytics and App Events

While separate, the Analytics and App Events systems in Makerkit are designed to work together to provide a centralized approach to event tracking and analysis in your SaaS application.

- **App Events**: A client-side event system for emitting and listening to important app-wide occurrences. Use this to bubble up important events in your app that you can handle centrally.
- **Analytics**: A centralized system for tracking and analyzing user behavior and app usage. Use this to track important events and user interactions in your app.

While these systems can be used independently, they shine when used together, creating a powerful, centralized approach to event handling and analytics.

{% img src="/assets/images/docs/analytics-events-overview.webp" alt="Analytics and Events in Makerkit" width="1062" height="342" /%}

## Recommended Approach: Centralized Analytics

We strongly recommend using a centralized approach to analytics by leveraging the App Events system. Instead of scattering `analytics.trackEvent()` calls throughout your codebase, use App Events to emit important occurrences, then handle these events centrally to track analytics.

Benefits of this approach:

1. **Cleaner Code**: Keeps your components focused on their primary responsibilities.
2. **Easier Maintenance**: Centralizes all analytics logic in one place.
3. **Flexibility**: Easily change or extend analytics tracking without modifying component code.
4. **Consistency**: Ensures a standardized approach to event tracking across your app.
5. **Visibility**: Provides a clear picture of all events emitted in your app in one place.

Of course - this is just a recommendation. Nothing prevents you from using the `analytics` object directly in your components. However, we believe that the centralized approach provides a cleaner and more maintainable solution for most SaaS applications.

## Using App Events with Analytics

Here's how to use the pre-configured App Events system for analytics in your Makerkit project:

1. Emit events in your components:

```typescript
import { useAppEvents } from '@kit/shared/events';

function SomeComponent() {
  const { emit } = useAppEvents();

  const handleSignUp = (userId: string) => {
    emit({ type: 'user.signedUp', payload: { userId } });
  };

  // ...
}
```

That's it! Makerkit automatically handles these events and tracks them in your configured analytics service.

## Extending with Custom Events

You can easily extend the system with your own custom events:

### Define your custom events

Create a new file to define your custom events:

```typescript
import { ConsumerProvidedEventTypes } from '@kit/shared/events';

export interface MyAppEvents extends ConsumerProvidedEventTypes {
  'feature.used': { featureName: string };
  'subscription.changed': { newPlan: string };
}
```

### Use your custom events

Once you've defined your custom events, you can use them in your components:

```typescript
import { useAppEvents } from '@kit/shared/events';
import { MyAppEvents } from './myAppEvents';

function SomeComponent() {
  const { emit } = useAppEvents<MyAppEvents>();

  const handleFeatureUse = () => {
    emit({ type: 'feature.used', payload: { featureName: 'coolFeature' } });
  };

  // ...
}
```

### Emit and handle your custom events

A common pattern is to emit custom events in your components and handle them in a centralized location. You can easily extend the App Events system to handle your custom events.

Here's an example of how you might handle custom events in your analytics provider:

 ```typescript {% title="apps/web/components/analytics-provider.tsx" %} {18-21}
const analyticsMapping: AnalyticsMapping = {
  'user.signedIn': (event) => {
    const userId = event.payload.userId;

    if (userId) {
      analytics.identify(userId);
    }
  },
  'user.signedUp': (event) => {
    analytics.trackEvent(event.type, event.payload);
  },
  'checkout.started': (event) => {
    analytics.trackEvent(event.type, event.payload);
  },
  'user.updated': (event) => {
    analytics.trackEvent(event.type, event.payload);
  },
  'feature.used': (event) => {
    analytics.trackEvent(event.type, event.payload);
  },
};
```

By following this approach, you can easily extend the App Events system with your own custom events, providing a flexible and maintainable approach to event tracking in your SaaS application.

## Default Event Types

Makerkit provides a set of default event types that are automatically tracked in your analytics service. These events are emitted by the Makerkit components and can be used to track common user interactions and app-wide occurrences.

Here are some of the default event types provided by Makerkit:
1. `user.signedIn`: Emitted when a user signs in. We use this event to identify the user in the analytics service. This makes sure that all subsequent events are associated with the user.
2. `user.signedUp`: Emitted when a user signs up. This event is used to track user signups in the analytics service. NB: this does not work automatically for social signups.
3. `checkout.started`: Emitted when a user starts the checkout process. This event is used to track the start of the checkout process in the analytics service.
4. `user.updated`: Emitted when a user updates their authentication details. This event is used to track user updates in the analytics service.

In addition to this, Makerkit tracks page views automatically. This means that you don't need to manually track page views in your application. However, you can still use the `trackPageView` method to manually track page views if needed.

### When to use Custom Events

As Makerkit becomes more and more like a framework, there is a need to expose more customization options to developers, ideally without the need to customize the core codebase. This is where Custom Events come in - which allow you to listen to and emit custom events in your application.

Custom Events are useful when you need to track specific user interactions or app-wide occurrences that are not covered by the default event types. For example, you might want to track when a user interacts with a specific feature, or for tracking specific user actions in your app.

By using Custom Events, you can extend the App Events system to track any event that is important to your application, providing a flexible and maintainable approach to event tracking in your SaaS application.

Other use cases may include:
1. Propagating events from Makerkit's deep components to the top-level application (please do let us know if you need any)
2. Centralizing event tracking for third-party integrations (e.g., tracking events in Segment, Amplitude, etc.)
3. Tracking user interactions with specific features in your application

## Conclusion

By using the Analytics and App Events systems in Makerkit, you can easily track (or react to) user behavior and app-wide occurrences in your SaaS application. The centralized approach to event tracking provides a clean and maintainable solution for tracking analytics, while the flexibility of Custom Events allows you to extend the system to track any event that is important to your application.

-----------------
FILE PATH: kits/remix-supabase-turbo/analytics/analytics-api.mdoc

---
status: "published"

title: 'Using the Analytics API in your Makerkit project'
label: 'Analytics API'
order: 1
description: 'Learn how to use the Analytics API in your Makerkit project to track user behavior and app usage.'
---


Makerkit provides a powerful and flexible Analytics API that integrates seamlessly with the App Events system. This API allows you to track user behavior and app usage easily and consistently across your SaaS application.

## Core Concepts of the Analytics API

The Analytics API is built around three main concepts:

1. **Identify**: Associate user data with a unique user ID.
2. **Track Events**: Record specific events or actions taken by users.
3. **Track Page Views**: Record when users view specific pages in your application.

### Using the Analytics API

The Analytics API is available through the `analytics` object imported from `@kit/analytics`.

#### Identifying Users

Use the `identify` method to associate a user with their actions:

```typescript
import { analytics } from '@kit/analytics';

void analytics.identify(userId, {
  email: user.email,
  plan: user.subscriptionPlan,
  // ... other user properties
});
```

#### Tracking Events

Use the `trackEvent` method to record specific actions or events:

```typescript
void analytics.trackEvent('Button Clicked', {
  buttonName: 'Submit',
  page: 'Sign Up',
});
```

#### Tracking Page Views

**Makerkit automatically tracks page views for you.** However, you can also manually track page views if needed.

Use the `trackPageView` method to record when users view specific pages:

```typescript
void analytics.trackPageView('Sign Up');
```

NB: The `trackPageView` method is automatically called when the route changes in a Remix application.

### Integration with App Events

While you can use these methods directly, we recommend leveraging the App Events system for a more centralized approach. Makerkit automatically maps common app events to analytics tracking:

```typescript
import { useAppEvents } from '@kit/shared/events';

function SomeComponent() {
  const { emit } = useAppEvents();

  const handleSignUp = (userId: string) => {
    emit({ type: 'user.signedUp', payload: { userId } });
    // This automatically calls analytics.identify and analytics.trackEvent
  };

  // ...
}
```

### Extending the Analytics API

You can extend the analytics functionality by creating custom event mappings:

```typescript
import { analytics } from '@kit/analytics';
import { useAppEvents } from '@kit/shared/events';

interface MyAppEvents {
  'feature.used': { featureName: string };
}

export function useMyAnalytics() {
  const { emit } = useAppEvents<MyAppEvents>();

  return {
    trackFeatureUse: (featureName: string) => {
      emit({ type: 'feature.used', payload: { featureName } });
      // If you need additional tracking logic:
      void analytics.trackEvent('Feature Used', { featureName });
    },
  };
}
```

## Best Practices

When implementing analytics in your Makerkit project, consider the following best practices:

1. **Use App Events**: Whenever possible, use the App Events system instead of calling analytics methods directly. This keeps your analytics logic centralized and easier to maintain.
2. **Consistent Naming**: Use consistent naming conventions for your events and properties across your application.
3. **Relevant Data**: Only track data that's relevant and useful for your business goals. Avoid tracking sensitive or personal information.
4. **Testing**: Always test your analytics implementation to ensure events are being tracked correctly.
5. **Documentation**: Keep a record of all the events and properties you're tracking. This will be invaluable as your application grows.

By leveraging Makerkit's Analytics API in conjunction with the App Events system, you can create a robust, maintainable analytics setup that grows with your SaaS application. This approach provides the flexibility to track the data you need while keeping your codebase clean and organized.

-----------------
FILE PATH: kits/remix-supabase-turbo/analytics/custom-analytics-provider.mdoc

---
status: "published"

title: 'Creating a Custom Analytics Provider in Makerkit'
label: 'Custom Analytics Provider'
description: 'Learn how to create a custom analytics provider in Makerkit to integrate with your preferred analytics service.'
order: 5
---


The Analytics package in Makerkit is meant to be flexible and extensible. You can easily create custom analytics providers to integrate with your preferred analytics service.

To create a custom analytics provider, you need to implement the `AnalyticsService` interface and then register it with the `AnalyticsManager`.

You can define one or more custom analytics providers, and the `AnalyticsManager` will handle the initialization and tracking of events for each provider.

NB: the methods expect to be Promise-based, so you can use async/await or return a Promise, or use the keyword `void` to ignore the return value and use it in non-async functions.

## Create your custom analytics service

Let's create a custom analytics service that implements the `AnalyticsService` interface:

 ```typescript {% title="packages/analytics/src/my-custom-analytics-service.ts" %}
import { AnalyticsService } from './path-to-types';

class MyCustomAnalyticsService implements AnalyticsService {
  async initialize() {
    // Initialize your analytics service
  }

  async identify(userId: string, traits?: Record<string, string>) {
    // Implement user identification
  }

  async trackPageView(url: string) {
    // Implement page view tracking
  }

  async trackEvent(eventName: string, eventProperties?: Record<string, string | string[]>) {
    // Implement event tracking
  }
}
```

### Register your custom provider

Update your analytics configuration file to include your custom provider:

 ```typescript {% title="packages/analytics/src/index.ts" %} {6, 8}
import { createAnalyticsManager } from './analytics-manager';
import { MyCustomAnalyticsService } from './my-custom-analytics-service';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
  providers: {
    myCustom: (config) => new MyCustomAnalyticsService(config),
    null: () => NullAnalyticsService,
  },
});
```

## Using the Custom Analytics Provider

Once you've created and registered your custom analytics provider, you can use it in your application as you would with any other analytics provider:

All the registered providers will dispatch the same events, so you can use them interchangeably:

```typescript
const analytics = createAnalyticsManager({
  providers: {
    googleAnalytics: (config) => new GoogleAnalyticsService(config),
    mixpanel: (config) => new MixpanelService(config),
    myCustom: (config) => new MyCustomAnalyticsService(config),
    null: () => NullAnalyticsService,
  },
});
```

That's it! You've successfully created a custom analytics provider in Makerkit. You can now integrate with any analytics service of your choice.

## Using the Custom Analytics Provider

Once you've created and registered your custom analytics provider, you can use it in your application as you would with any other analytics provider:

```typescript
import { analytics } from '@kit/analytics';

void analytics.identify('user123', { name: 'John Doe' });
void analytics.trackEvent('Button Clicked', { buttonName: 'Submit' });
```

That's it! You've successfully created a custom analytics provider in Makerkit. You can now integrate with any analytics service of your choice.

