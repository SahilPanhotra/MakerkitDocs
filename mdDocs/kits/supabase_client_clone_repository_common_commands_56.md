-----------------
FILE PATH: kits\react-native-supabase\development\supabase-client.mdoc

---
status: "published"
title: "How to use the Supabase Client in the React Native Supabase kit"
label: "Supabase Client"
description: "To use the Supabase Client in the React Native Supabase kit, use the useSupabase hook in React Components"
order: 0
---

Before diving into the various ways we can communicate with the server, we need to introduce how we communicate with Supabase, which is hosting the database and therefore the source of our data.

## Using the Supabase client

To import the Supabase client in a browser environment,  you can use the `useSupabase` hook:

```tsx
import { useSupabase } from '@kit/supabase';

export default function Home() {
  const supabase = useSupabase()

  return (
    <div>
      <h1>Supabase Browser Client</h1>
      <button onClick={() => supabase.auth.signOut()}>Sign Out</button>
    </div>
  )
}
```

-----------------
FILE PATH: kits\react-native-supabase\installation\clone-repository.mdoc

---
status: "published"
title: "Clone the React Native Supabase SaaS Kit Turbo Repository"
label: "Clone the Repository"
order: 4
description: "Clone the React Native Supabase SaaS Kit Turbo repository to your local machine."
---

## This is a preview of the React Native Supabase Kit Turbo repository

Hi, this is a preview of the React Native Supabase Kit Turbo repository. It
may contain bugs and errors. If you spot any, please open an issue on the
[GitHub repository](https://github.com/makerkit/react-native-expo-turbo-saas-kit).

## Prerequisites

To get started with Makerkit, ensure you have the following prerequisites installed and set up:

- **Node.js 18.x or later**
- **Docker**
- **Pnpm**
- **Supabase account** (optional for local development)
- **Email Service account** (optional for local development)

## Cloning the Repository

Clone the repository using the following command:

```bash
git clone git@github.com:makerkit/react-native-expo-turbo-saas-kit.git
```

**Note:** If your SSH key isn't set up, use HTTPS instead:

```bash
git clone https://github.com/makerkit/react-native-expo-turbo-saas-kit
```

#### Windows Users: place the repository near the root of your drive

Some Windows users hit errors loading certain modules due to very long paths. To avoid this, try placing the repository near the root of your drive.

Avoid using OneDrive, as it can cause issues with Node.js.

### Important: Use HTTPS for All Commands if Not Using SSH

If you are not using SSH, ensure you switch to HTTPS for all Git commands, not just the clone command.

## Configuring Git Remotes

After cloning the repository, remove the original `origin` remote:

```bash
git remote rm origin
```

Add the upstream remote pointing to the original repository to pull updates:

```bash
git remote add upstream git@github.com:makerkit/react-native-expo-turbo-saas-kit
```

Once you have your own repository set up, add your repository as the `origin`:

```bash
git remote add origin <your-repository-url>
```

### Keeping Your Repository Up to Date

To pull updates from the upstream repository, run the following command daily (preferably with your morning coffee ☕):

```bash
git pull upstream main
```

This ensures your repository stays up to date with the latest changes.

## 0.1. Install Pnpm

Install Pnpm globally with the following command:

```bash
npm i -g pnpm
```

## 1. Setup Dependencies

Install the necessary dependencies for the project:

```bash
pnpm i
```

With these steps completed, your development environment will be set up and ready to go! 🚀

## 2. Post-merge Hooks

It's very useful to run the following command after pulling updates from the upstream repository:

```bash
pnpm i
```

This ensures that any new dependencies are installed and the project is up to date. We can run this command automatically after pulling updates by setting up a post-merge hook.

Create a new file named `post-merge` in the `.git/hooks` directory:

```bash
touch .git/hooks/post-merge
```

Add the following content to the `post-merge` file:

```bash
#!/bin/bash

pnpm i
```

Make the `post-merge` file executable:

```bash
chmod +x .git/hooks/post-merge
```

Now, every time you pull updates from the upstream repository, the `pnpm i` command will run automatically to ensure your project is up to date. This ensures you're always working with the latest changes and dependencies and avoid errors that may arise from outdated dependencies.


-----------------
FILE PATH: kits\react-native-supabase\installation\common-commands.mdoc

---
status: "published"
title: 'Common commands you need to know for the React Native Supabase Turbo Starter Kit'
label: 'Common Commands'
order: 5
description: 'Learn about the common commands you need to know to work with the React Native Supabase Turbo Starter Kit.'
---

Here are some common commands you'll need to know when working with the React Native Supabase Turbo Starter Kit.

**Note:** You don't need these commands to kickstart your project, but it's useful to know they exist for when you need them.

## Installing Dependencies

To install the dependencies, run the following command:

```bash
pnpm i
```

This command will install all the necessary dependencies for the project.

## Starting the Development Server

Start the development server for the web application with:

```bash
pnpm run dev
```

## Running Supabase CLI Commands

Supabase is installed in the `apps/expo-app` folder. To run commands with the Supabase CLI, use:

```bash
pnpm run --filter app supabase <command>
```

For example, if Supabase documentation recommends a command like:

```bash
supabase db link
```

You would run:

```bash
pnpm run --filter app supabase db link
```

## Starting Supabase

To start Supabase, run:

```bash
pnpm run supabase:web:start
```

This command starts the Supabase web server.

## Starting Stripe

To test the billing system, start Stripe with:

```bash
pnpm run stripe:listen
```

This routes webhooks to your local machine.

## Resetting Supabase

To reset the Supabase database, which is necessary when you update the schema or need a fresh start, run:

```bash
pnpm run supabase:web:reset
```

## Generate Supabase Types

When you update the Supabase schema, generate the latest types for the client by running:

```bash
pnpm run supabase:web:typegen
```

This should be done every time the Supabase schema is updated.

## Running Tests

To run the tests for the project, use:

```bash
pnpm run test
```

## Cleaning the Project

To clean the project, run:

```bash
pnpm run clean:workspaces
pnpm run clean
```

Then, reinstall the dependencies:

```bash
pnpm i
```

## Type-Checking the Project

To type-check the project, use:

```bash
pnpm run typecheck
```

## Linting the Project

To lint the project using ESLint, run:

```bash
pnpm run lint:fix
```

## Formatting the Project

To format the project using Prettier, run:

```bash
pnpm run format:fix
```

These commands will help you manage and maintain your project efficiently.

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

```sql {% title="apps/web/supabase/migrations/20230815034829_super-admin.sql" %}
UPDATE auth.users SET raw_app_meta_data = raw_app_meta_data || '{"role": "super-admin"}' WHERE id='<user_id>';
```

Please replace `<user_id>` with the user ID you want to assign as a super admin.

### Enable MFA for the Super Admin user

The Super Admin user will be required to use Multi-Factor Authentication (MFA) to access the Super Admin panel.

Please navigate to the `/settings` page and enable MFA for the Super Admin user.

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

