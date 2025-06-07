-----------------
FILE PATH: kits/next-supabase-turbo/security.mdoc

---
status: "published"
title: "Security in Next.js Supabase Turbo"
label: "Security"
order: 14
description: "Security in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-turbo/translations/adding-translations.mdoc

---
status: "published"
title: "Adding new translations | Next.js Supabase SaaS Kit"
label: "Adding new translations"
description: "Learn how to add new translations to your Next.js Supabase SaaS project."
order: 1
---

{% sequence title="Steps to add new translations" description="Learn how to add new translations to your Next.js Supabase SaaS project." %}

[Create Language Files](#1.-create-language-files)

[Update Language Settings](#2.-update-language-settings)

[Add new namespaces](#3.-add-new-namespaces)

{% /sequence %}

Makerkit projects store translations at the application level, specifically in the `apps/web/public/locales` directory. By default, We provide the English translations at `apps/web/public/locales/en`.

If you're looking to expand your application's language support, follow these two steps:

## 1. Create Language Files

First, create a new folder in `apps/web/public/locales/[lng]`, where `[lng]` represents the appropriate language code. This could be `de` for German, `it` for Italian, `es` for Spanish, `ja` for Japanese, and so on.

### Regional Language Codes

If you use a regional language code, such as `es-ES` for Spanish in Spain, you must use a lowercase `es-es` instead. Thus, the folder name should be `apps/web/public/locales/es-es`.

## 2. Update Language Settings

Next, add the new language to the settings file located at `apps/web/lib/i18n/i18n.settings.ts`:

```tsx
/**
 * The list of supported languages.
 * By default, only the default language is supported.
 * Add more languages here if needed.
 */
export const languages: string[] = [defaultLanguage, 'es'];
```

In the example above, the language `es` (Spanish) is added to the `languages` array. Feel free to add as many languages as your application requires.

## 3. Add new namespaces

If you want to add a new namespace (for example, `chatbots.json`) where you will store translations for chatbots, you can do so by creating a new file in the `apps/web/public/locales/[lng]` directory. For example, `apps/web/public/locales/en/chatbots.json`.

Then, you register the new namespace in the `i18n.settings.ts` file:

```tsx {8} title="apps/web/lib/i18n/i18n.settings.ts"
export const defaultI18nNamespaces = [
  'common',
  'auth',
  'account',
  'teams',
  'billing',
  'marketing',
  'chatbots',      // <--- Add the new namespace here
];
```

In the example above, the `chatbots` namespace is added to the `defaultI18nNamespaces` array. And that's it! You can now start adding translations to your new namespace.

-----------------
FILE PATH: kits/next-supabase-turbo/translations/language-selector.mdoc

---
status: "published"
title: "Adding a language selector to your application | Next.js Supabase SaaS Kit"
label: "Language Selector"
description: "Learn how to use the Language selector in your application."
order: 2
---

You can import and use the `LanguageSelector` component and drop it anywhere in your application to allow users to change the language of the application.

```tsx
import { LanguageSelector } from '@kit/ui/language-selector';

<LanguageSelector />
```

This component is automatically populated with the languages that you have defined in the `i18n` configuration. When a user selects a language, the application will automatically change the language and update the UI.

It is already part of the Account settings when more than one language is defined in the `i18n` configuration. If you want, add it to any other part of your application.


-----------------
FILE PATH: kits/next-supabase-turbo/translations/using-translations.mdoc

---
status: "published"
title: "Using translations in your Next.js Supabase project"
label: "Using translations"
description: "Makerkit uses i18next to translate your project into multiple languages. This guide will show you how to use translations in your Next.js Supabase project."
order: 0
---

Makerkit uses the library `i18next` for translations, but it's abstracted behind the `@kit/i18n` package. This abstraction ensures that any future changes to the translation library won't impact your code.

{% sequence title="Steps to use translations" description="Learn how to use translations in your Next.js Supabase project." %}

[Using Translations in your Next.js Supabase Project](#using-translations-in-your-nextjs-supabase-project)

[Initializing i18n in Server Components](#initializing-i18n-in-server-components)

[Using Translations in Client Components](#using-translations-in-client-components)

{% /sequence %}

The translation system in Makerkit is versatile and can be used in:

1. Server components (RSC)
2. Client components
3. Server-side rendering

## Using Translations in your Next.js Supabase Project

The guide below will show you how to use translations in your Next.js Supabase project in all the scenarios mentioned above.

### Initializing i18n in Server Components

When creating a new page, ensure to wrap it with the `withI18n` function. This function initializes the translations before rendering the server component.

Given that server components are rendered in parallel, it's uncertain which one will render first. Therefore, it's crucial to initialize the translations before rendering the server component on each page/layout.

```tsx
import { withI18n } from '~/lib/i18n/with-i18n';

const Page = () => {
  return <div>My page</div>;
};

export default withI18n(Page);
```

### Implementing Translations in Server Components

After initializing the translations, you can use the `Trans` component to translate strings in your server components.

```tsx
import { Trans } from '@kit/ui';

const Page = () => {
  return (
    <div>
      <Trans i18nKey="auth:signIn" />
    </div>
  );
};

export default withI18n(Page);
```

Ensure to import the `Trans` component from `@kit/ui` and not directly from `react-i18next`.

## Using Translations in Client Components

Besides simply using the `Trans` component, you can also use the `useTranslation` hook to translate strings in your client components.

### Using the useTranslation Hook

The `useTranslation` hook is another way to translate strings in components that are not in the render path.

```tsx
import { useTranslation } from 'react-i18next';

const MyComponent = () => {
  const { t } = useTranslation();
  
  return <div>{t('auth:signIn')}</div>;
};
```

### Conclusion

By following these steps, you can easily add translations to your Next.js Supabase project. This approach ensures that your application is accessible to a global audience, providing a better user experience for all users.

-----------------
FILE PATH: kits/next-supabase-turbo/translations.mdoc

---
status: "published"
title: "Translations in Next.js Supabase Turbo"
label: "Translations"
order: 10
description: "Translations in Next.js Supabase Turbo"
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
FILE PATH: kits/next-supabase-turbo/troubleshooting/troubleshooting-authentication.mdoc

---
status: "published"
title: 'Troubleshooting authentication issues in the Next.js Supabase kit'
label: 'Authentication'
order: 2
description: 'Troubleshoot issues related to authentication in the Next.js Supabase SaaS kit'
---


## Supabase redirects to localhost instead of my website URL

This is most likely an issue related to not having set up the Authentication URL in the Supabase settings as describe in [the going to production guide](../going-to-production/supabase);

## Cannot Receive Emails from Supabase

This issue may arise if you have not setup an SMTP provider from within Supabase.

Supabase's own SMTP provider has low limits and low deliverability, therefore you must set up your own - and only use the default Supabase's one for testing purposes.

## Cannot sign in with oAuth provider

This is most likely a settings issues from within the Supabase dashboard or in the provider's settings.

Please [read Supabase's documentation](https://supabase.com/docs/guides/auth/social-login) on how to set up third-party providers.


-----------------
FILE PATH: kits/next-supabase-turbo/troubleshooting/troubleshooting-billing.mdoc

---
status: "published"
title: 'Troubleshooting billing issues in the Next.js Supabase kit'
label: 'Billing'
order: 3
description: 'Troubleshoot issues related to billing in the Next.js Supabase SaaS kit'
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
FILE PATH: kits/next-supabase-turbo/troubleshooting/troubleshooting-deployment.mdoc

---
status: "published"
title: 'Troubleshooting deployment issues in the Next.js Supabase kit'
label: 'Deployment'
order: 4
description: 'Troubleshoot issues related to deploying the application in the Next.js Supabase SaaS kit'
---

## The deployment build fails

This is most likely an issue related to the environment variables not being set correctly in the deployment environment. Please analyse the logs of the deployment provider to see what is the issue.

The kit is very defensive about incorrect environment variables, and will throw an error if any of the required environment variables are not set. In this way - the build will fail if the environment variables are not set correctly - instead of deploying a broken application.

If you are deploying to Vercel, [please follow this guide](../going-to-production/vercel).

-----------------
FILE PATH: kits/next-supabase-turbo/troubleshooting/troubleshooting-emails.mdoc

---
status: "published"
title: 'Troubleshooting emails issues in the Next.js Supabase kit'
label: 'Emails'
order: 5
description: 'Troubleshoot issues related to emails in the Next.js Supabase SaaS kit'
---

To troubleshoot emails issues in the Next.js Supabase kit, please make sure to:

1. **Correct Credentials**: please make sure you have setup the correct credentials in your environment variables. Refer to the [emails guide](../emails/email-configuration) for more information.
2. **Your domain is verified**: make sure your domain is verified in your Email provider
3. **The Database Webhooks are configured**: make sure you have configured the Database Webhooks in your Supabase project. Refer to the [Supabase deployment guide](../going-to-production/supabase) for more information. Make sure to read Webhooks are pointing to the correct URL and use the correct shared secret.
4. **The Webhooks use a public URL**: make sure the webhooks use a public URL and not behind authentication (such as a Vercel Preview URL).
5. **Read the Logs**: please verify your server logs in your hosting provider to see if there are any errors related to emails.

If you have done all the above and still having issues, please open a Support Ticket in Discord.

-----------------
FILE PATH: kits/next-supabase-turbo/troubleshooting/troubleshooting-installation.mdoc

---
status: "published"

title: 'Troubleshooting installation issues in the Next.js Supabase SaaS kit'
label: 'Installation'
order: 1
description: 'Troubleshoot issues related to installing the Next.js Supabase SaaS kit'
---


## Cannot clone the repository

Issues related to cloning the repository are usually related to a Git misconfiguration in your local machine. The commands displayed in this guide using SSH: these will work only if you have setup your SSH keys in Github.

If you run into issues, [please make sure you follow this guide to set up your SSH key in Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

If this also fails, please use HTTPS instead. You will be able to see the commands in the repository's Github page under the "Clone" dropdown.

Please also make sure that the account that accepted the invite to Makerkit, and the locally connected account are the same.

## The Next.js dev server does not start

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
FILE PATH: kits/next-supabase-turbo/troubleshooting/troubleshooting-module-not-found.mdoc

---
status: "published"
title: 'Troubleshooting module not found issues in the Next.js Supabase kit'
label: 'Modules not found'
order: 6
description: 'Troubleshoot issues related to modules not found in the Next.js Supabase SaaS kit'
---

Let's walk through common "Module not found" errors and how to fix them in your Makerkit project.

This issue is mostly related to either dependency installed in the wrong package or issues with the file system.

### 1. Windows Paths issues

Windows has a limitation for the maximum path length. If you're running into issues with paths, you can try the following:

1. Move the project closer to the root of your drive
2. Use a shorter name for the project's folder name
3. Please Please Please use WSL2 instead of Windows

### 2. Windows OneDrive Conflicts

OneDrive can cause file system issues with Node.js projects. If you're using Windows with OneDrive:

1. Move your project outside of OneDrive-synced folders
2. Or disable OneDrive sync for your development folder


### 3. Dependency Installation Issues

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

-----------------
FILE PATH: kits/next-supabase-turbo/troubleshooting.mdoc

---
status: "published"
title: "Troubleshooting in Next.js Supabase Turbo"
label: "Troubleshooting"
order: 17
description: "Troubleshooting in Next.js Supabase Turbo"
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
FILE PATH: kits/react-native-supabase/development/database.mdoc

---
status: "published"
title: "Introduction to the Database in the React Native Supabase Kit"
label: "Database"
description: "How to use the Supabase Database in the React Native Supabase Kit"
order: 0
---

The Database provided in the React Native Supabase Kit is a simple schema with one single table: `accounts`. It is the same provided by the [Next.js Supabase Lite Kit](https://github.com/makerkit/next-supabase-saas-kit-lite).

**NB**: If you are using a Pro version of the Turbo kits, please replace the schema with the one provided by the Pro kit. The code in this kit is compatible with the schema of the Pro kit, as it's designed to be used in conjunction with it. However, it can also be used as a standalone schema.

## Accounts

The `accounts` table is the main table of the schema. It contains the following fields:

```sql
create table if not exists
    public.accounts
(
    id          uuid unique  not null default extensions.uuid_generate_v4(),
    name        varchar(255) not null,
    email       varchar(320) unique,
    updated_at  timestamp with time zone,
    created_at  timestamp with time zone,
    created_by  uuid references auth.users,
    updated_by  uuid references auth.users,
    picture_url varchar(1000),
    public_data jsonb                 default '{}'::jsonb not null,
    primary key (id)
);
```

When a new user signs up, the `accounts` table is automatically populated
with the user data. The `id` of the account is the `uuid` generated by the
Supabase database for the user's account in the `auth.users` table.

From here on, you can add more tables to the schema, and link them to the
`accounts` table using the `account_id` field to reference the user's account.

-----------------
FILE PATH: kits/react-native-supabase/development/development.mdoc

---
status: "published"
title: "Data Fetching in the React Native Supabase Kit"
label: "Data Fetching"
order: 1
collapsible: true
collapsed: false
---



-----------------
FILE PATH: kits/react-native-supabase/development/react-query.mdoc

---
status: "published"
description: "React Query is the best React library for managing asynchronous data fetching in your applications. Learn how to use it with React Native Supabase."
title: "Using React Query with React Native Supabase Kit"
label: "React Query"
order: 5
---

Makerkit uses React Query for client-side data fetching. This allows you to fetch data from your Supabase database and display it on the client-side using an ergonomic API.

```tsx
import { useQuery } from '@tanstack/react-query';
import { useSupabase } from '@kit/supabase';
import { Text, View } from 'react-native';

function TasksList(accountId: string) {
  const client = useSupabase();

  const { data, isLoading, error } = useQuery({
    queryKey: ['tasks', accountId],
    queryFn: async () => {
      const { data, error } = await client
        .from('tasks')
        .select('*')
        .eq('account_id', accountId)
        .order('created_at', { ascending: false });

      if (error) {
        throw error;
      }

      return data;
    }
  });

  if (isLoading) {
    return <Text>Loading...</Text>;
  }

  if (error) {
    return <Text>Error loading tasks</Text>;
  }

  return (
    <View className={'flex gap-4'}>
      {data.map(task => (
        <View key={task.id}>
          <Text>{task.title}</Text>
        </View>
      ))}
    </View>
  );
}
```

By leveraging the `useSupabase` hook, you can access the Supabase DB data right from your React components.

Similarly, you can use React Query for mutations:

```tsx
import { View } from 'react-native';
import { useMutation } from '@tanstack/react-query';
import { useSupabase } from '@kit/supabase';
import { Text, Button, Input } from '@kit/ui';

function CreateTaskForm() {
  const client = useSupabase();

  const mutation = useMutation({
    mutationFn: async (data) => {
      const { data, error } = await client
        .from('tasks')
        .insert({
          title: data.title,
          description: data.description,
          account_id: data.accountId,
        })
        .select('*')
        .single();

      if (error) {
        throw error;
      }

      return data;
    },
  });

  const handleSubmit = async (data) => {
    const { data, error } = await mutation.mutateAsync(data);

    if (error) {
      console.error(error);
    }
  };

  return (
    <View className="flex flex-col gap-4">
      <Input type="text" name="title" />
      <Input type="text" name="description" />

      <Button onPress={handleSubmit}>
        <Text>Create Task</Text>
      </Button>
    </View>
  );
}
```

-----------------
FILE PATH: kits/react-native-supabase/development/supabase-client.mdoc

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
FILE PATH: kits/react-native-supabase/installation/clone-repository.mdoc

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
FILE PATH: kits/react-native-supabase/installation/common-commands.mdoc

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

