-----------------
FILE PATH: kits\react-native-supabase\installation\installation.mdoc

---
status: "published"
title: "How to develop using the React Native Supabase Kit"
label: "Development"
order: 0
collapsible: true
collapsed: false
---

-----------------
FILE PATH: kits\react-native-supabase\installation\introduction.mdoc

---
status: "published"
title: 'Introduction to React Native Supabase SaaS Kit Turbo Repository'
label: 'Introduction'
order: 1
description: 'Introducing the React Native Supabase SaaS Kit Turbo repository, a starter kit for building mobile apps with Supabase.'
---

Welcome to the documentation of the React Native Supabase SaaS Kit!

This mobile starter is an **open source kit** to help you build companion
apps to the SaaS product built with [Makerkit](/). However, it can also be
used as a standalone starter for building mobile apps with Supabase.

NB: this kit is not included in any license, and therefore **no support is
provided**.

{% alert type="info" title="This documentation is still in progress" %}
This documentation is still in progress. Please check back soon for updates.
{% /alert %}

## What's Included

### Core Architecture

- 🏗️ [Expo](https://expo.dev) + [React Native](https://reactnative.dev)
- 🎨 [NativeWind](https://nativewind.dev) + [Tailwind CSS](https://tailwindcss.com)
[React Native Reusable Components](https://rnr-docs.vercel.app/getting-started/introduction/)
- 🔐 [Supabase](https://supabase.com) for database, authentication and storage
- ✨ Full TypeScript + ESLint + Prettier configuration

### Key Features

- 👤 User authentication flow
- ⚙️ User profile & settings
- 🔒 Protected routes

## Scope of This Documentation

### Focus on the Kit

While building a SaaS application involves many moving parts, this documentation focuses specifically on the React Native Supabase SaaS Kit. For in-depth information on the underlying technologies (React Native, Supabase, Turbo, etc.), please refer to their respective official documentation.

This documentation will guide you through configuring, running, and deploying the kit, and will provide links to the official documentation of the underlying technologies where necessary. To fully grasp the kit's capabilities, it's essential to understand these technologies, so be sure to explore their documentation as well.

For anything strictly related to the React Native Supabase SaaS Kit, this documentation has you covered!

### Refer to Official Documentation

For in-depth understanding of the underlying technologies, refer to their official documentation:

- **React Native:** [React Native Documentation](https://reactnative.dev)
- **Expo:** [Expo Documentation](https://docs.expo.dev)
- **Supabase:** [Supabase Documentation](https://supabase.com)
- **NativeWind:** [NativeWind Documentation](https://nativewind.dev)

Understanding these technologies is crucial for building your mobile app.

## Conclusion

This React Native Supabase SaaS Kit is designed to accelerate your SaaS development with a robust, modular, and extensible foundation. Dive into the documentation, explore the features, and start building your SaaS application today! 🚀


-----------------
FILE PATH: kits\react-native-supabase\installation\llms.mdoc

---
status: "published"
title: "LLM rules for Cursor and Windsurf | React Native Supabase"
label: "LLMs rules"
order: 11
description: "Use these rules for Cursor and Windsurf to generate the best possible context for your LLMs"
---

The following content is a guideline for [Cursor](https://www.cursor.com) and [Codeium Windsurf](https://codeium.com) to help you generate the best possible context for your LLMs. It should also work well for other coding tools, such as Windsurf.

1. **Cursor**: create a file `.cursorrules` file in your project root and paste the rules below.
2. **Windsurf**: create a file `.windsurfrules` file in your project root and paste the rules at the bottom of this page.

**Please Remember**: These tools are brilliant, but not perfect. Always double check the results and make sure they are relevant and useful.

These rules are already included in the repository, so you can use them directly. For your convenience, I have also included them below:

````md
# Makerkit Guidelines

You are an expert programming assistant focusing on:

- Expertise: Expo, React Native, Supabase, TypeScript, Tailwind CSS in a Turborepo project
- Focus: Code clarity, Readability, Best practices, Maintainability
- Style: Expert level, factual, solution-focused
- Libraries: TypeScript, React Hook Form, React Query, Zod, Lucide React Native

## Project Structure

The below is the Expo app structure:

```
- apps
-- expo-app
--- (app)
---- (main)      # protected routes
---- auth        # auth pages
--- components   # global components
--- supabase     # supabase root
```

- Features are located in the "packages/features" directory.
- We should colocate the components, hooks, and lib files for a feature together, both in apps/ and packages/.
- Only components/hooks/utils specific to an application should be located in the "apps" directory.

## Database

- Supabase uses Postgres
- We strive to create a safe, robust, performant schema
- Accounts are the general concept of a user account, defined by the having the same ID as Supabase Auth's users (personal).
- Generally speaking, other tables will be used to store data related to the account. For example, a table `notes` would have a foreign key `account_id` to link it to an account.
- Using RLS, we must ensure that only the account owner can access the data. Always write safe RLS policies and ensure that the policies are enforced.
- Unless specified, always enable RLS when creating a table. Propose the required RLS policies ensuring the safety of the data.
- Always consider any required constraints and triggers are in place for data consistency
- Always consider the compromises you need to make and explain them so I can make an educated decision. Follow up with the considerations make and explain them.
- Always consider the security of the data and explain the security implications of the data.
- Always use Postgres schemas explicitly (e.g., `public.accounts`)
- Data types should always be inferred using the `Database` types from `@kit/supabase/database`

```tsx
import type { Tables } from "@kit/supabase";
type Bookmark = Tables<"bookmarks">;
```

## UI Components

Reusable UI components are defined in the "packages/ui" package named `@kit/ui`. All components are exported from `@kit/ui`

### Code Standards

- Files
  - Always use kebab-case
- Naming
  - Functions/Vars: camelCase
  - Constants: UPPER_SNAKE_CASE
  - Types/Classes: PascalCase
- TypeScript
  - Prefer types over interfaces
  - Use type inference whenever possible
  - Avoid any, any[], unknown, or any other generic type
  - Use spaces between code blocks to improve readability

### Styling

- Styling is done using Tailwind CSS. We use the "cn" function from the "@kit/ui" package to generate class names.
- Avoid fixes classes such as "bg-gray-500". Instead, use Shadcn classes such as "bg-background", "text-secondary-foreground", "text-muted-foreground", etc.

### Data Fetching

- In a Client Component context, please use the `useQuery` hook from the "@tanstack/react-query" package to wrap any async data fetching.

#### Supabase Clients

- Use the `useSupabase` hook from the `@kit/supabase` package in React Components to get the Supabase client. You can use the Supabase Client in combination with the `useQuery` hook to fetch data.

#### React Query

When using `useQuery`, make sure to define the data fetching hook. Create two components: one that fetches the data and one that displays the data.

## Forms

- Use Zod for form validation.
- Use the `zodResolver` function to resolve the Zod schema to the form.
- Always use `@kit/ui/form` for writing the UI of the form
- Use `react-hook-form` for the form's state management
- Use React Query's mutation to submit the form mutation

## Error Handling

- Don't swallow errors, always handle them appropriately
- Handle promises and async/await gracefully
- Consider the unhappy path and handle errors appropriately
- Context without sensitive data
````


-----------------
FILE PATH: kits\react-native-supabase\installation\running-project.mdoc

---
status: "published"
title: "Running the React Native Supabase Turbo project"
label: "Running the Project"
order: 4
description: "Learn how to run the React Native Supabase Turbo project on your local machine."
---

To run the project, follow these steps to start the development server, Supabase, and Stripe (optional for billing system testing).

## 1. Start the Development Server

To start the web application development server, run:

```bash
# Start the development server
pnpm dev
```

This command launches the web application.

### Quick Start Credentials

Use the following credentials to get started right away:

- **Email**: `test@makerkit.dev`
- **Password**: `testingpassword`

To confirm email addresses, visit [Inbucket](http://localhost:54324/status). Supabase uses Inbucket to capture emails sent during the authentication process.

Bookmark the Inbucket URL, as you will need it quite often.

## 2. Start Supabase

Ensure Docker is running, then start Supabase with:

```bash
pnpm run supabase:web:start
```

This command initiates the local Supabase web server. This allows us to develop locally without having to deploy to Supabase.

-----------------
FILE PATH: kits\react-native-supabase\installation\updating-codebase.mdoc

---
status: "published"
title: "Updating your React Native Supabase Turbo Starter Kit"
label: "Updating the Codebase"
order: 6
description: "Learn how to update your React Native Supabase Turbo Starter Kit to the latest version."
---

🚀 Ready to keep your project at the cutting edge? 

This guide will walk you through the process of updating your codebase by pulling the latest changes from the GitHub repository and merging them into your project. This ensures you're always equipped with the latest features and bug fixes.

If you've been following along with our previous guides, you should already have a Git repository set up for your project, with an `upstream` remote pointing to the original repository.

Updating your project involves fetching the latest changes from the `upstream` remote and merging them into your project. Let's dive into the steps!

## 0. Stashing your changes

If you have uncommited changes, before updating your project, it's a good idea to stash your changes to avoid any conflicts during the update process. You can stash your changes by running:

```bash
git stash
```

This will save your changes temporarily, allowing you to update your project without any conflicts. Once you've updated your project, you can apply your changes back by running:

```bash
git stash pop
```

If you don't have any changes to stash, you can skip this step and proceed with the update process. 🛅

Alternatively, you can commit your changes.

## 1. Refresh the `upstream` remote

The first step is to fetch the latest changes from the `upstream` remote. You can do this by running the following command:

```bash
git pull upstream main
```

When prompted the first time, please opt for merging instead of rebasing.

Don't forget to run `pnpm i` in case there are any updates in the dependencies. 🔄

## 2. Resolve any conflicts

Encountered conflicts during the merge? No worries! You'll need to resolve them manually. Git will highlight the files with conflicts, and you can edit them to resolve the issues.

### 2.1 Conflicts in the lock file "pnpm-lock.yaml"

If you find conflicts in the `pnpm-lock.yaml` file, accept either of the two changes (avoid manual edits), then run:

```bash
pnpm i
```

Your lock file will now reflect both your changes and the updates from the `upstream` repository. 🎉

### 2.2 Conflicts in the DB types "database.types.ts"

Your types might differ from those in the `upstream` repository, so you'll need to rebuild them. To do this, reset the DB:

```bash
npm run supabase:web:reset
```

Next, regenerate the types with the following command:

```bash
npm run supabase:web:typegen
```

Your types will now reflect the changes from both the `upstream` repository and your project. 🚀

### Run a health check on your project after resolving conflicts

After resolving the conflicts, it's time to test your project to ensure everything is working as expected. Run your project locally and navigate through the various features to verify that everything is functioning correctly.

You can run the following commands for a quick health check:

```bash
pnpm run typecheck
```

And lint your code with:

```bash
pnpm run lint
```

If everything looks good, you're all set! Your project is now up to date with the latest changes from the `upstream` repository. 🎉

## Commit and push changes

Once everything is working fine, don't forget to commit your changes with git commit -m "COMMIT_MESSAGE" and push them to your remote repository with git push origin BRANCH_NAME.

Of course - please make sure to replace COMMIT_MESSAGE and BRANCH_NAME with your actual commit message and branch name. 🚀

-----------------
FILE PATH: kits\react-router-supabase-turbo\admin\adding-super-admin.mdoc

---
status: "published"
label: "Adding a Super Admin"
title: "Adding a Super Admin to your React Router Supabase application"
description: "In this post, you will learn how to set up a Super Admin in your React Router Supabase application"
order: 0
---

The Super Admin panel allows you to manage users and accounts.

To access the super admin panel at `/admin`, you will need to assign a user as a super admin.

## Testing the Super Admin locally

By default, we seed the `auth.users` table with a super admin user. To login as this user, you can use the following credentials:

```json
{
  "email": "super-admin@makerkit.dev",
  "password": "testingpassword"
}
```

Since you require MFA for the Super Admin user, please use the following steps to pass MFA:

1. **TOTP**: Use the following [TOTP generator](https://totp.danhersam.com/) to generate a TOTP code.
2. **Secret Key**: Use the test secret key `NHOHJVGPO3R3LKVPRMNIYLCDMBHUM2SE` to generate a TOTP code.
3. **Verify**: Use the TOTP code and the secret key to verify the MFA code.
Make sure the TOTP code is not expired when you verify the MFA code.

{% alert type="warning" title="These are test credentials" %}
The flow above is for testing purposes only. For production, you must use an authenticator app such as Google Authenticator, Authy, Bitwarden, Proton, or similar.
{% /alert %}

## Assigning a Super Admin role to a user

To add your own super admin user, you will need to:

1. **Role**: Add the `super-admin` role to the user in your database
2. **Enable MFA**: Enable MFA for the user (mandatory) from the profile
settings of the user

### Modify the `auth.users` record in your database

To assign a user as a super admin, run the following SQL query from your Supabase SQL Query editor:

```sql
UPDATE auth.users SET raw_app_meta_data = raw_app_meta_data || '{"role": "super-admin"}' WHERE id='<user_id>';
```

Please replace `<user_id>` with the user ID you want to assign as a super admin.

### Enable MFA for the Super Admin user

The Super Admin user will be required to use Multi-Factor Authentication (MFA) to access the Super Admin panel.

Please navigate to the `/home/settings` page and enable MFA for the Super Admin user.

-----------------
FILE PATH: kits\react-router-supabase-turbo\admin.mdoc

---
status: "published"
label: "Super Admin"
title: "Adding a Super Admin to your React Router Supabase application"
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
FILE PATH: kits\react-router-supabase-turbo\analytics\analytics-and-events.mdoc

---
status: "published"

title: 'Understanding Analytics and Events in the React Router Supabase Turbo kit'
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
FILE PATH: kits\react-router-supabase-turbo\analytics\analytics-api.mdoc

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

NB: The `trackPageView` method is automatically called when the route changes in a React Router application.

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

