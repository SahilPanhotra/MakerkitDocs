-----------------
FILE PATH: kits/react-router-supabase-turbo/installation/navigating-codebase.mdoc

---
status: "published"
title: "Navigating the your React Router Supabase Turbo Starter Kit codebase"
label: "Navigating the Codebase"
order: 7
description: "Learn how to navigate the codebase of the React Router Supabase Turbo Starter Kit. Understand the project structure and how to update the codebase."
---

As mentioned before, this SaaS Starter Kit is modular, thanks to Turborepo - which makes it easy to manage multiple packages in a single repository.

## Project Structure

The main directories in the project are:
1. `apps` - the location of the React Router app
2. `packages` - the location of the shared code and the API

### `apps` Directory

This is where the app (or the apps, should you add more) lives. The main app is a React Router app, which is a React framework that allows you to build static and server-rendered web applications.

The React Router app defines:

1. the configuration of the project
2. the routes of the application

### `packages` Directory

This is where the shared code and the API live. The shared code is a collection of functions that are used by both the React Router app and the API. The API is a serverless function that is used to interact with the Supabase database.

The shared code defines:

1. shared libraries (Supabase, Mailers, CMS, Billing, etc.)
2. shared features (admin, accounts, account, auth, etc.)
3. UI components (buttons, forms, modals, etc.)

All apps can use and reuse the API exported from the `packages` directory. This makes it easy to have one, or many apps in the same codebase, sharing the same code.

-----------------
FILE PATH: kits/react-router-supabase-turbo/installation/running-project.mdoc

---
status: "published"
title: "Running the React Router Supabase Turbo project"
label: "Running the Project"
order: 4
description: "Learn how to run the React Router Supabase Turbo project on your local machine."
---

To run the project - there are some commands you need to run.

1. Start the development server (web application)
2. Start Supabase (ensure Docker is running)
3. Start Stripe (optional, if you want to test the billing system)

## 0. Environment variables

Please create a copy of the `.env.template` file and rename it `.env`.

## 1. Start the development server

```bash
# Start the development server
pnpm dev
```

This command will run the web application.

Please refer to `apps/web/README.md` for more information about the web application.

To get started right away, use the credentials below:

- **Email**: `test@makerkit.dev`
- **Password**: `testingpassword`

To confirm email addresses, please visit [Inbucket](http://localhost:54324/status): Supabase uses Inbucket to capture emails sent during the authentication process.

### Testing the Super Admin locally

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

## 2. Start Supabase

{% alert type="warning" title="Have Docker running" %}
We need to have Docker running so that we can start the Supabase Docker container. If you don't have Docker running, please start it now.
{% /alert %}

Ensure Docker is running, the run the following command to start Supabase (or use your IDE to run the command in the package.json).
[Docker Desktop](https://www.docker.com/products/docker-desktop/) is the best way to get started with Docker.

```bash
pnpm run supabase:web:start
```

This command will start the Supabase web server.

Please read the [Supabase documentation](https://supabase.com/docs/guides/local-development/cli/getting-started#running-supabase-locally) for more information about starting Supabase locally.

{% alert type="warning" title="Give Docker enough resources" %}
If the command above fails, you may need to give Docker more resources. Please refer to your Docker software's documentation for instructions on how to do this.
{% /alert %}

## 3. Start Stripe

If you want to test the billing system, you can start Stripe by running the following command:

```bash
pnpm run stripe:listen
```

This will route webhooks to your local machine.


-----------------
FILE PATH: kits/react-router-supabase-turbo/installation/technical-details.mdoc

---
status: "published"
title: "Technical Details of the React Router Supabase SaaS Boilerplate"
label: "Technical Details"
order: 2
description: "A detailed look at the technical details of the React Router Supabase SaaS Kit."
---

The kit is built as a [Turborepo](https://turbo.build) monorepo, and is broadly using the following technologies:

1. React Router 7
2. Supabase for the database, authentication and storage
3. Shadcn UI for the UI components
4. React Query, Zod, Lucide React and other libraries
5. **Nodemailer** for emails, or **Resend** via HTTP (for edge runtimes)

Makerkit's modular architecture allows you to easily add or remove features, and customize the application to your needs.

Additionally, the kit is designed to be easily deployable to Vercel, Cloudflare, or any other hosting provider. It also supports edge rendering so you can deploy the application to Cloudflare Workers and go FAST.

### Monorepo Structure

Below are the reusable packages that can be shared across multiple applications (or packages):

- **@kit/ui**: Shared UI components and styles (using Shadcn UI and some custom components)
- **@kit/shared**: Shared code and utilities
- **@kit/supabase**: Supabase package that defines the schema and logic for managing Supabase
- **@kit/i18n**: Internationalization package that defines utilities for managing translations
- **@kit/billing**: Billing package that defines the schema and logic for managing subscriptions
- **@kit/billing-gateway**: Billing gateway package that defines the schema and logic for managing payment gateways
- **@kit/email-templates**: Here we define the email templates using the react.email package.
- **@kit/mailers**: Mailer package that abstracts the email service provider (e.g., Resend, Cloudflare, SendGrid, Mailgun, etc.)
- **@kit/monitoring**: A unified monitoring package that defines the schema and logic for monitoring the application with third party services (e.g., Sentry, Baselime, etc.)
- **@kit/database-webhooks**: Database webhooks package that defines the actions following database changes (e.g., sending an email, updating a record, etc.)
- **@kit/cms**: CMS package that defines the schema and logic for managing content

And features that can be added to the application:

- **@kit/auth**: Authentication package (using Supabase)
- **@kit/accounts**: Package that defines components and logic for managing personal accounts
- **@kit/team-accounts**: Package that defines components and logic for managing team
- **@kit/admin**: Admin package that defines the schema and logic for managing users, subscriptions, and more.
- **@kit/notifications**: Notifications package that defines the schema and logic for managing notifications

And billing packages that can be added to the application:

- **@kit/stripe**: Stripe package that defines the schema and logic for managing Stripe. This is used by the @kit/billing-gateway package and abstracts the Stripe API.
- **@kit/lemon-squeezy**: Lemon Squeezy package that defines the schema and logic for managing Lemon Squeezy. This is used by the @kit/billing-gateway package and abstracts the Lemon Squeezy API.

The CMSs that can be added to the application:

- **@kit/keystatic**: [Keystatic](https://keystatic.com/) package that defines the schema and logic for managing Keystatic. This is used by the @kit/cms package and abstracts the Keystatic API.
- **@kit/wordpress**:  WordPress package that defines the schema and logic for managing WordPress. This is used by the @kit/cms package and abstracts the WordPress API.

Also planned (post-release):

- **@kit/plugins**: Move the existing plugins to a separate package here
- **@kit/analytics**: A unified analytics package to track user behavior

### Application Configuration

The configuration is defined in the `apps/web/config` folder. Here you can find the following configuration files:

- **app.config.ts**: Application configuration (e.g., name, description, etc.)
- **auth.config.ts**: Authentication configuration
- **billing.config.ts**: Billing configuration
- **feature-flags.config.ts**: Feature flags configuration
- **paths.config.ts**: Paths configuration (e.g., routes, API paths, etc.)
- **personal-account-sidebar.config.ts**: Personal account sidebar configuration (e.g., links, icons, etc.)
- **team-account-sidebar.config.ts**: Team account sidebar configuration (e.g., links, icons, etc.)


-----------------
FILE PATH: kits/react-router-supabase-turbo/installation/updating-codebase.mdoc

---
status: "published"

title: "Updating your React Router Supabase Turbo Starter Kit"
label: "Updating the Codebase"
order: 6
description: "Learn how to update your React Router Supabase Turbo Starter Kit to the latest version."
---


To update the codebase, you will pull the latest changes from the GitHub repository and merge them into your project. This will ensure that you have the latest features and bug fixes.

If you have followed the instructions in the previous sections, you should have a Git repository set up for your project, with an `upstream` remote pointing to the original repository.

To update your project, you will first fetch the latest changes from the `upstream` remote, and then merge them into your project.

Here are the steps to update your project.

### Update the `upstream` remote

First, you need to fetch the latest changes from the `upstream` remote. To do this, run the following command:

```bash
git pull upstream main
```

Please run `pnpm i` in case there are any changes in the dependencies.

### Resolve any conflicts

If there are any conflicts during the merge, you will need to resolve them manually. Git will show you the files with conflicts, and you can edit them to resolve the conflicts.

#### Conflicts in the lock file "pnpm-lock.yaml"

If you have conflicts in the `pnpm-lock.yaml` file, accept any of the two changes (don't do it manually), then run:

```
pnpm i
```

The lock file will now reflect both your changes and the changes from the `upstream` repository.

#### Conflicts in the DB types "database.types.ts"

Since your types will differ from the ones in the `upstream` repository, you need to rebuild them.

To do so, reset the DB:

```bash
npm run supabase:web:reset
```

Then, run the following command to regenerate the types:

```bash
npm run supabase:web:typegen
```

Now the types will reflect the changes from the `upstream` repository and your project.



-----------------
FILE PATH: kits/react-router-supabase-turbo/installation.mdoc

---
status: "published"
title: "Installation in React Router Supabase Turbo"
label: "Installation"
order: 1
description: "Installation in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Hi, welcome to the installation guide for the React Router Supabase SaaS Kit Turbo. This guide will help you set up your development environment and get started with the kit.

In this section, we will cover:

1. [Introduction](introduction): the second version of Makerkit
2. [Technical Details](technical-details): the technical details of this kit
3. [Conventions](conventions): some of the conventions used in this kit
4. [Clone the Repository](clone-repository): cloning the repository
5. [Running the Project](running-the-project): running the project
6. [Common Commands](common-commands): some common commands to help you work with the project
7. [Updating the Project](updating-codebase): how to update the codebase



-----------------
FILE PATH: kits/react-router-supabase-turbo/monitoring/baselime.mdoc

---
status: "published"

title: "Configuring Baselime in your React Router Supabase SaaS kit"
label: "Baselime"
order: 1
---


[Baselime](https://baselime.io) (now part of Cloudflare) is an observability platform that helps you monitor your application's performance and errors. In this guide, you'll learn how to configure Baselime in your React Router Supabase SaaS kit.

To use [Baselime](https://baselime.io) to capture exceptions and performance metrics of your app, please define the below variables:

```bash
VITE_BASELIME_KEY=your_key
VITE_MONITORING_PROVIDER=baselime
```

-----------------
FILE PATH: kits/react-router-supabase-turbo/monitoring/overview.mdoc

---
title: "Setting Monitoring in Makerkit"
label: "How Monitoring works"
position: 0
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the React Router Supabase SaaS kit"
---

Makerkit provides first-class support for two monitoring providers:

1. Baselime (now part of Cloudflare)
2. Sentry

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using React Router's instrumentation hook
2. **Client exceptions** - Automatically capturing uncaught exceptions on the client
3. **Server exceptions** - Automatically capturing server-side exceptions when using the functions `enhanceAction` and `enhanceRouteHandler`

Additionally, it provides you with the tools to manually capturing exceptions, should you want to.

To set up monitoring in your application, you need to define the two variables below:

```bash
# sentry or baselime
VITE_MONITORING_PROVIDER=

# performance monitoring
MONITORING_INSTRUMENTATION_ENABLED=true
```

1. **Monitoring Provider** - the monitoring provider to use. Based on this variable, Makerkit will provide the relative implementation.
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using React Router

For the provider's specific settings, please check the relative documentation.

#### Performance Monitoring

Performance monitoring uses React Router's experimental instrumentation for reporting performance metrics using OpenTelemetry.

NB: Performance monitoring is only enabled for Node.js runtimes. This means you won't see performance metrics in the edge runtime or Cloudflare.

-----------------
FILE PATH: kits/react-router-supabase-turbo/monitoring.mdoc

---
status: "published"
title: "Setting up Monitoring in Makerkit | React Router SaaS Boilerplate"
label: "Monitoring"
order: 12
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the React Router Supabase SaaS kit"
collapsible: true
collapsed: true
---

Makerkit provides first-class support for two monitoring providers:

1. Baselime (now part of Cloudflare)
2. Sentry

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using React Router's instrumentation hook
2. **Client exceptions** - Automatically capturing uncaught exceptions on the client
3. **Server exceptions** - Automatically capturing server-side exceptions when using the functions `enhanceAction` and `enhanceRouteHandler`

Additionally, it provides you with the tools to manually capturing exceptions, should you want to.

### Configuring Monitoring in Makerkit

To set up monitoring in your application, you need to define the two variables below:

```bash
# sentry or baselime
VITE_MONITORING_PROVIDER=

# performance monitoring (only required for Baselime)
ENABLE_MONITORING_INSTRUMENTATION=true
INSTRUMENTATION_SERVICE_NAME=your_service_name
```

1. **Monitoring Provider** - the monitoring provider to use. Based on this variable, Makerkit will provide the relative implementation.
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using React Router

For the provider's specific settings, please check the relative documentation.

#### Performance Monitoring

Performance monitoring uses React Router' experimental instrumentation for reporting performance metrics using OpenTelemetry.

NB: Performance monitoring is only enabled for Node.js runtimes.


-----------------
FILE PATH: kits/react-router-supabase-turbo/notifications/notifications-configuration.mdoc

---
status: "published"
title: "Configuring Notifications in the React Router Supabase Starter Kit"
description: "Learn how to configure notifications in the React Router Supabase Starter Kit."
label: "Notifications Configuration"
order: 0
---

Makerkit allows you to configure and send notifications using the `@kit/notifications` package. The package provides a simple API to send notifications.

There are two variables that you can set in the `.env` file to configure notifications:

1. **VITE_ENABLE_NOTIFICATIONS**: Set this to `true` to enable notifications.
2. **VITE_REALTIME_NOTIFICATIONS**: Set this to `true` to enable real-time notifications.

```bash
VITE_ENABLE_NOTIFICATIONS=true
VITE_REALTIME_NOTIFICATIONS=true
```

1. If `VITE_ENABLE_NOTIFICATIONS` is set to `false`, the bell icon will not appear in the header. Otherwise, it will appear.
2. If `VITE_REALTIME_NOTIFICATIONS` is set to `false`, the notifications will not be real-time. Otherwise, they will be real-time.

### Why would you not want to enable notifications?

If you are not using notifications in your application, you can disable them to reduce the complexity of your application. This can be useful if you are building a simple application and do not need notifications.

### Real-time notifications

If you want to enable real-time notifications, you need to set the `VITE_REALTIME_NOTIFICATIONS` environment variable to `true`. This will enable real-time notifications in your application.

This may be your preference for a more cost-effective solution, as it will reduce the number of requests to the server.

-----------------
FILE PATH: kits/react-router-supabase-turbo/notifications/sending-notifications.mdoc

---
status: "published"

title: "Sending a notification in the React Router Supabase Starter Kit"
description: "Learn how to send a notification in the React Router Supabase Starter Kit."
label: "Sending Notifications"
order: 1
---


Makerkit allows you to send notifications using the `@kit/notifications` package. The package provides a simple API to send notifications.

The `createNotificationsApi` provides an easy to use API to send notifications. You can send notifications to users after they sign up, create a task, or perform any other action.

The service is a server-side service, so you can use it in your server actions or route handlers. This service **cannot be used from the client-side** since the only role who can insert notifications is the `service_role` role.

Here is a super simple example of how you can send a notification after a user signs up.

```tsx
import { createNotificationsApi } from '@kit/notifications/api';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

async function sendNotificationAfterSignup(
  accountId: string,
) {
  const client = getSupabaseServerAdminClient();
  const api = createNotificationsApi(client);

  await api.createNotification({
    account_id: accountId,
    body: 'You have successfully signed up!',
  });
}
```

In this example, we are sending a notification to the user after they sign up. The `account_id` is the ID of the user who signed up, and the `body` is the message that will be displayed in the notification.

Users will see the notifications if:
1. they're sent to their own account
2. they're sent to a team they're a member of

### Notification channels

You can send notifications to different channels using the `channel` field. For example, you can send a notification to the `email` or `in_app`. In-app notifications will be displayed in the app, while email notifications will be sent to the user's email. By default, notifications are sent to the `in_app` channel.

The `createNotificationsApi` function takes the `client` parameter - eg. the Supabase client.

```tsx
import { createNotificationsApi } from '@kit/notifications/api';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

async function sendNotificationAfterSignup(
  accountId: string,
) {
  const client = getSupabaseServerAdminClient();
  const api = createNotificationsApi(client);

  await api.createNotification({
    account_id: accountId,
    body: 'You have successfully signed up!',
    channel: 'email',
  });
}
```

The example above sends an email notification to the user after they sign up.

NB: this require a Database trigger to send the email notification! Please dont't forget to add the trigger in your database.

---


NB: at the time of writing, this is not yet implemented in the React Router Supabase Starter Kit. It is a feature that is planned for the future.

### Notification types

You can send different types of notifications using the `type` field. For example, you can send a `info`, `warning`, or `error` notification.

By default, notifications are sent as `info`.

```tsx
import { createNotificationsApi } from '@kit/notifications/api';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

async function sendNotificationAfterSignup(
  accountId: string,
) {
  const client = getSupabaseServerAdminClient();
  const api = createNotificationsApi(client);

  await api.createNotification({
    account_id: accountId,
    body: 'You have successfully signed up!',
    type: 'info', // this is the default type, no need to specify it
  });
}
```

#### Warning notifications

To send a warning notification, you can use the `warning` type.

```tsx
api.createNotification({
  account_id: accountId,
  body: 'Your credit card is about to expire!',
  type: 'warning',
});
```

#### Error notifications

To send an error notification, you can use the `error` type.

```tsx
api.createNotification({
  account_id: accountId,
  body: 'There was an error processing your payment.',
  type: 'error',
});
```

### Notification Links

You can also include a link in the notification using the `link` field. This will create a clickable link in the notification.

```tsx
api.createNotification({
  account_id: accountId,
  body: 'You have created a task!',
  link: '/tasks/123',
});
```

The example above sends a notification to the user after they create a task. The notification will include a link to the task.


-----------------
FILE PATH: kits/react-router-supabase-turbo/notifications.mdoc

---
status: "published"
title: "Notifications in React Router Supabase Turbo"
label: "Notifications"
order: 10
description: "Notifications in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Add notifications to your application:

1. [**Notification Configuration**](/docs/next-supabase-turbo/notifications-configuration)
   Set up and configure your notification system.
2. [**Sending Notifications**](/docs/next-supabase-turbo/sending-notifications)
   Learn how to send notifications to your users.

-----------------
FILE PATH: kits/react-router-supabase-turbo/plugins.mdoc

---
status: "published"
title: "Plugins in React Router Supabase Turbo"
label: "Plugins"
order: 14
description: "Plugins in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Extend your application's functionality with our plugin system:

1. [**Getting Started with Plugins**](/docs/next-supabase-turbo/installing-plugins)
   Learn how to install and manage plugins in your application.
2. [**Available Plugins**](/docs/next-supabase-turbo/plugins)
   - [Chatbot Plugin](/docs/next-supabase-turbo/chatbot-plugin)
   - [Feedback Plugin](/docs/next-supabase-turbo/feedback-plugin)
   - [Text Editor Plugin](/docs/next-supabase-turbo/text-editor-plugin)
   - [Waitlist Plugin](/docs/next-supabase-turbo/waitlist-plugin)
   - [Testimonials Plugin](/docs/next-supabase-turbo/testimonials-plugin)
   - [Roadmap Plugin](/docs/next-supabase-turbo/roadmap-plugin)


-----------------
FILE PATH: kits/react-router-supabase-turbo/translations/adding-translations.mdoc

---
status: "published"
title: "Adding new translations | React Router Supabase SaaS Kit"
label: "Adding new translations"
description: "Learn how to add new translations to your  React Router Supabase SaaS project."
order: 1
---

Translations are stored at application level and can be found at `apps/web/public/locales`. Translations for English (which is the only language provided by default) will be found at `apps/web/public/locales/en`.

If you wish to provide a new language, there are two steps.

### 1. Add the language files

Create a new folder using the correct language code at `apps/web/public/locales/[lng]` where `[lng]` can be any correct language code, such as `de`, `it`, `es`, `ja`, etc.

### Regional Language Codes

If you use a regional language code, such as `es-ES` for Spanish in Spain, you must use a lowercase `es-es` instead. Thus, the folder name should be `apps/web/public/locales/es-es`.

### 2. Update settings

Add the language to the settings file at `apps/web/lib/i18n/i18n.settings.ts`:

```tsx
/**
 * The list of supported languages.
 * By default,  only the default language is supported.
 * Add more languages here if needed.
 */
export const languages: string[] = [defaultLanguage, 'es'];
```

In the above, I appended the language `es` to the `languages` array. You can add as many languages as you need.


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
  'chatbots',
];
```

In the example above, the `chatbots` namespace is added to the `defaultI18nNamespaces` array. And that's it! You can now start adding translations to your new namespace.

-----------------
FILE PATH: kits/react-router-supabase-turbo/translations/language-selector.mdoc

---
status: "published"
title: "Adding a language selector to your application | React Router Supabase SaaS Kit"
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
FILE PATH: kits/react-router-supabase-turbo/translations/using-translations.mdoc

---
status: "published"

title: "Using translations in your React Router Supabase project"
label: "Using translations"
description: "Makerkit uses i18next to translate your project into multiple languages. This guide will show you how to use translations in your React Router Supabase project."
order: 0
---


Makerkit uses `i18next` for translations, although we abstract if behind the package `@kit/i18n` - so that if we ever need to change the translation library, we can do so without affecting your code.

Makerkit's translations system works everywhere:

1. Server components
2. Client components
3. Server-side rendering

## How to use translations

### Bootstrapping i18n in Server Components

When you create a new page, make sure to wrap it around the `withI18n` function to bootstrap the translations before rendering the server component.

Because server components are rendered in parallel, there is no telling which one will be rendered first. This is why we need to bootstrap the translations before rendering the server component in each page/layout.

```tsx
import { withI18n } from '~/lib/i18n/with-i18n';

const Page = () => {
  return <div>My page</div>;
};

export default withI18n(Page);
```

### Using translations in Server Components

Once you have bootstrapped the translations, you can use the `Trans` component to translate strings in your server components.

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

It is very important you load the `Trans` component from `@kit/ui` and not from `react-i18next` directly.

### Using the useTranslation hook

You can also use the `useTranslation` hook to translate strings in components that are not in the render path.

```tsx
import { useTranslation } from 'react-i18next';

const MyComponent = () => {
  const { t } = useTranslation();

  return <div>{t('auth:signIn')}</div>;
};
```

