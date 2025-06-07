-----------------
FILE PATH: kits/remix-supabase-turbo/installation/technical-details.mdoc

---
status: "published"
title: "Technical Details of the Remix Supabase SaaS Boilerplate"
label: "Technical Details"
order: 2
description: "A detailed look at the technical details of the Remix Supabase SaaS Kit."
---

The kit is built as a [Turborepo](https://turbo.build) monorepo, and is broadly using the following technologies:

1. Remix
2. Supabase for the database, authentication and storage
3. Shadcn UI for the UI components
4. React Query, Zod, Lucide React and other libraries
5. Configurable mailers (SMTP with Nodemailer, Resend)

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
FILE PATH: kits/remix-supabase-turbo/installation/updating-codebase.mdoc

---
status: "published"

title: "Updating your Remix Supabase Turbo Starter Kit"
label: "Updating the Codebase"
order: 6
description: "Learn how to update your Remix Supabase Turbo Starter Kit to the latest version."
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
FILE PATH: kits/remix-supabase-turbo/installation.mdoc

---
status: "published"
title: "Installation in Remix Supabase Turbo"
label: "Installation"
order: 1
description: "Installation in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Hi, welcome to the installation guide for the Remix Supabase SaaS Kit Turbo. This guide will help you set up your development environment and get started with the kit.

In this section, we will cover:

1. [Introduction](introduction): the second version of Makerkit
2. [Technical Details](technical-details): the technical details of this kit
3. [Conventions](conventions): some of the conventions used in this kit
4. [Clone the Repository](clone-repository): cloning the repository
5. [Running the Project](running-the-project): running the project
6. [Common Commands](common-commands): some common commands to help you work with the project
7. [Updating the Project](updating-codebase): how to update the codebase



-----------------
FILE PATH: kits/remix-supabase-turbo/monitoring/baselime.mdoc

---
status: "published"

title: "Configuring Baselime in your Remix Supabase SaaS kit"
label: "Baselime"
order: 1
---


[Baselime](https://baselime.io) (now part of Cloudflare) is an observability platform that helps you monitor your application's performance and errors. In this guide, you'll learn how to configure Baselime in your Remix Supabase SaaS kit.

To use [Baselime](https://baselime.io) to capture exceptions and performance metrics of your app, please define the below variables:

```bash
VITE_BASELIME_KEY=your_key
VITE_MONITORING_PROVIDER=baselime
```

-----------------
FILE PATH: kits/remix-supabase-turbo/monitoring/overview.mdoc

---
title: "Setting Monitoring in Makerkit"
label: "How Monitoring works"
position: 0
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the Remix Supabase SaaS kit"
---

Makerkit provides first-class support for two monitoring providers:

1. Baselime (now part of Cloudflare)
2. Sentry

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using Remix's instrumentation hook
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
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using Remix

For the provider's specific settings, please check the relative documentation.

#### Performance Monitoring

Performance monitoring uses Remix's experimental instrumentation for reporting performance metrics using OpenTelemetry.

NB: Performance monitoring is only enabled for Node.js runtimes. This means you won't see performance metrics in the edge runtime or Cloudflare.

-----------------
FILE PATH: kits/remix-supabase-turbo/monitoring/sentry.mdoc

---
status: "published"

title: 'Configuring Sentry in your Remix Supabase SaaS kit'
label: 'Sentry'
order: 2
---


Please set the following environment variable:

```bash
VITE_MONITORING_PROVIDER=sentry
VITE_SENTRY_DSN=your_dsn
```

Update your root app at `apps/web/app/root.tsx` to include the following:

1. Install the Sentry SDK for Remix in your application project:

```bash
cd apps/web
pnpm i @sentry/remix
```

2. Update your root app to include the Sentry SDK:

```tsx
// import the Sentry SDK
import { withSentry } from '@sentry/remix';

// export your root app wrapped with Sentry
export default withSentry(App);
```


-----------------
FILE PATH: kits/remix-supabase-turbo/monitoring.mdoc

---
status: "published"
title: "Setting up Monitoring in Makerkit | Remix SaaS Boilerplate"
label: "Monitoring"
order: 12
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the Remix Supabase SaaS kit"
collapsible: true
collapsed: true
---

Makerkit provides first-class support for two monitoring providers:

1. Baselime (now part of Cloudflare)
2. Sentry

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using Remix's instrumentation hook
2. **Client exceptions** - Automatically capturing uncaught exceptions on the client
3. **Server exceptions** - Automatically capturing server-side exceptions when using the functions `enhanceAction` and `enhanceRouteHandler`

Additionally, it provides you with the tools to manually capturing exceptions, should you want to.

### Configuring Monitoring in Makerkit

To set up monitoring in your application, you need to define the two variables below:

```bash
# sentry or baselime
NEXT_PUBLIC_MONITORING_PROVIDER=

# performance monitoring (only required for Baselime)
ENABLE_MONITORING_INSTRUMENTATION=true
INSTRUMENTATION_SERVICE_NAME=your_service_name
```

1. **Monitoring Provider** - the monitoring provider to use. Based on this variable, Makerkit will provide the relative implementation.
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using Remix

For the provider's specific settings, please check the relative documentation.

#### Performance Monitoring

Performance monitoring uses Remix' experimental instrumentation for reporting performance metrics using OpenTelemetry.

NB: Performance monitoring is only enabled for Node.js runtimes.


-----------------
FILE PATH: kits/remix-supabase-turbo/notifications/notifications-configuration.mdoc

---
status: "published"

title: "Configuring Notifications in the Remix Supabase Starter Kit"
description: "Learn how to configure notifications in the Remix Supabase Starter Kit."
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
FILE PATH: kits/remix-supabase-turbo/notifications/sending-notifications.mdoc

---
status: "published"

title: "Sending a notification in the Remix Supabase Starter Kit"
description: "Learn how to send a notification in the Remix Supabase Starter Kit."
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


NB: at the time of writing, this is not yet implemented in the Remix Supabase Starter Kit. It is a feature that is planned for the future.

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
FILE PATH: kits/remix-supabase-turbo/notifications.mdoc

---
status: "published"
title: "Notifications in Remix Supabase Turbo"
label: "Notifications"
order: 10
description: "Notifications in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Add notifications to your application:

1. [**Notification Configuration**](/docs/next-supabase-turbo/notifications-configuration)
   Set up and configure your notification system.
2. [**Sending Notifications**](/docs/next-supabase-turbo/sending-notifications)
   Learn how to send notifications to your users.

-----------------
FILE PATH: kits/remix-supabase-turbo/plugins/chatbot-plugin.mdoc

---
status: "published"

title: 'Add an AI Chatbot to the Remix Supabase SaaS Starter kit'
label: 'AI Chatbot'
order: 2
description: 'Add an AI chatbot to your Remix Supabase SaaS Starter kit to provide instant support to your users.'
---


**NB:** this is currently a work in progress and not yet available.

This plugin allows you to create a chatbot for your app. Users can interact with the chatbot to get information about
your app.

The chatbot will index your app's content and provide answers to user queries.

## Installation

Install the chatbot plugin from your main app:

```bash
npx @makerkit/cli plugins install chatbot
```

Now, install the plugin package in your application:

```bash
pnpm add --filter web @kit/chatbot
```

```bash

## Import the Chatbot component

You can now import the Chatbot component in your app:

```tsx
import { Chatbot } from '@kit/chatbot';

<Chatbot sitename={'Makerkit'} />
```

## Create the API route handler

You can add this anywhere since it's configurable, but my suggestion is to add it to the `api.chat._index` directory. Create a new file called `route.ts` at `api.chat._index/route.ts`:

```ts
import { ActionFunctionArgs } from '@remix-run/server-runtime';

import { handleChatBotRequest } from '@kit/chatbot/server';

export const action = async ({ request }: ActionFunctionArgs) => {
  return handleChatBotRequest(request);
};

```

If you use a different path, please set the `VITE_CHATBOT_API_URL` environment variable:

```bash
VITE_CHATBOT_API_URL=/api/some-other-path
```

## Add the Migration file

Copy the migration file at `packages/plugins/chatbot/migrations` into your apps migration directory at `apps/web/supabase/migrations`.

Make sure to reset the database after copying the migration file and make sure to push this to your remote instance.

## Add your OpenAI API Key

In your hosting provider, add the `OPENAI_API_KEY` environment variable with your OpenAI API key.

```bash
OPENAI_API_KEY=sk-1234567890
```

Locally, you can set it in your `.env` file, which is never committed to the repository.

## Configuration

The Chatbot component accepts the following props:

```tsx
interface ChatBotProps {
    siteName: string;

    conversationId?: string;
    defaultPrompts?: string[];
    isOpen?: boolean;
    isDisabled?: boolean;
    settings?: ChatbotSettings;
    storageKey?: string;

    onClear?: () => void;
    onMessage?: (message: string) => void;
}
```

## Customizing the Chatbot

You can customize the chatbot by providing a `settings` object:

```tsx
type Position = `bottom-left` | `bottom-right`;

interface Branding {
    primaryColor: string;
    accentColor: string;
    textColor: string;
}

export interface ChatbotSettings {
    title: string;
    order: Position;
    branding: Branding;
}
```

For example:

```tsx
<Chatbot settings={{
    title: 'Chat with us',
    order: 'bottom-right',
    branding: {
        primaryColor: '#000',
        accentColor: '#000',
        textColor: '#fff',
    },
}} />
```

## Indexing Content

At this time - the Chatbot will crawl your app's content and index it - by reading the sitemap.xml file.

The crawler will infer the sitemap.xml from the website's robots.txt file. Otherwise, you can provide the sitemap.xml
URL using the `CHATBOT_WEBSITE_SITEMAP_URL` environment variable.

```bash
CHATBOT_WEBSITE_SITEMAP_URL=https://example.com/sitemap.xml
```

Then, you will need to provide your Supabase remote instance URL and key using the `.env` at `.env`.

```bash {% title="apps/web/.env" %}
SUPABASE_SERVICE_ROLE_KEY=*****
VITE_SUPABASE_URL=https://*****.supabase.co
LLM_MODEL_NAME=
LLM_BASE_URL=
LLM_API_KEY=
```

Please make sure to never commit these keys to your repository. By default, the

To index your website's content, the chatbot will read the sitemap.xml file.

To launch the chatbot crawler, run the command:

```bash
pnpm --filter chatbot indexer -- --url <url> --include <include-paths> --exclude <excluded-paths>
```

For example:

```bash
pnpm --filter chatbot indexer -- --url https://example.com --include /docs,/blog --exclude /blog/secret
```

The `--url` flag is required and should point to the website's root URL. The `--include` and `--exclude` flags are optional and should be comma-separated paths.

It's highly recommended to include the `--exclude` flag to prevent the chatbot from indexing way too many pages or stuff you don't want to be indexed.

## Environment Variables

You can customize the chatbot model by providing the following environment variables:

```
LLM_MODEL_NAME=gpt-3.5-turbo
LLM_BASE_URL=
LLM_API_KEY=
```

The model needs to be compatible with the OpenAI API.

-----------------
FILE PATH: kits/remix-supabase-turbo/plugins/feedback-plugin.mdoc

---
status: "published"

title: 'Add a Feedback Widget plugin to your Remix Supabase SaaS Starter kit'
label: 'Feedback Widget'
order: 4
description: 'Add a Feedback Widget plugin to your Remix Supabase SaaS Starter kit'
---


The feedback plugin allows you to add a feedback widget to your app. Users can provide feedback on your app, and you can view and manage feedback submissions in the admin panel.

### Installation

Pull the plugin from the main repository:

```
npx @makerkit/cli@latest plugins install feedback
```

Now, install the plugin from your main app:

```
pnpm add --filter web @kit/feedback
```

### Add the translations

Add the following to your `apps/web/lib/i18n/locales/en/feedback.json` file:

 ```json {% title="apps/web/lib/i18n/locales/en/feedback.json" %}
{
  "successTitle": "Thank you for your feedback!",
  "close": "Close",
  "errorTitle": "Sorry, Something went wrong!",
  "errorDescription": "Please try again later or contact us directly.",
  "contactUs": "Contact us about...",
  "email": "Email",
  "feedback": "Feedback",
  "question": "Question",
  "bug": "Bug",
  "send": "Send",
  "sending": "Sending...",
  "attachFileOrScreenshot": "Attach file or screenshot",
  "feedbackPlaceholder": "What do you like or dislike? What can we do better?",
  "questionPlaceholder": "Ask us anything",
  "bugPlaceholder": "What happened? What were you expecting to happen?",
  "uploadImage": "Upload Image"
}
```

Now, add the namespace to the loaded translations in your `apps/web/lib/i18n/i18n.settings.ts` file:

 ```tsx {% title="apps/web/lib/i18n/i18n.settings.ts" %}
export const defaultI18nNamespaces = [
  'feedback',
];
```

### Create an API route

Create an API route at `routes/api.feedback._index/route.ts` and call the action:

```tsx
import { ActionFunctionArgs } from '@remix-run/server-runtime';

import { submitFeedbackAction } from '@kit/feedback/server';

export const action = ({ request }: ActionFunctionArgs) => {
  if (request.method !== 'POST') {
    return new Response(null, { status: 405 });
  }

  return submitFeedbackAction(request);
};
```

### Import the component

Now, you can import the component from the plugin:

```tsx
import {FeedbackPopup} from '@kit/feedback';
```

And use it in your app:

```tsx
<FeedbackPopup>
    <Button>Gimme feedback</Button>
</FeedbackPopup>
```

You can also import the form alone - so you can customize its appearance:

```tsx
import {FeedbackForm} from '@kit/feedback';
```

And use it in your app:

```tsx
<FeedbackForm/>
```

## Adding the Admin Panel pages

Add the following to your `routes/admin.feedback._index/route.ts` file:

```tsx
import { LoaderFunctionArgs } from '@remix-run/server-runtime';

import { FeedbackSubmissionsPage } from '@kit/feedback/admin';
import { feedbackSubmissionsLoader } from '@kit/feedback/admin/server';
import { deleteFeedbackSubmissionAction } from '@kit/feedback/server';

export const loader = ({ request }: LoaderFunctionArgs) => {
  return feedbackSubmissionsLoader(request);
};

export default FeedbackSubmissionsPage;

export const action = async ({ request }: LoaderFunctionArgs) => {
  return deleteFeedbackSubmissionAction(request);
}
```

Add the following to your `routes/admin.feedback.$id._index/route.ts` file:

```tsx
import {
  ActionFunctionArgs,
  LoaderFunctionArgs,
} from '@remix-run/server-runtime';

import { FeedbackSubmissionPage } from '@kit/feedback/admin';
import { feedbackSubmissionLoader } from '@kit/feedback/admin/server';
import { deleteFeedbackSubmissionAction } from '@kit/feedback/server';

export const loader = ({ params, request }: LoaderFunctionArgs) => {
  return feedbackSubmissionLoader(request, params.id as string);
};

export default FeedbackSubmissionPage;

export const action = ({ request }: ActionFunctionArgs) => {
  return deleteFeedbackSubmissionAction(request);
};
```

Add the sidebar item to `apps/web/app/routes/admin/_components/admin-sidebar.tsx`:

```tsx

<SidebarItem
    path={'/admin/feedback'}
    Icon={<MessageCircle className={'h-4'} />}
>
    Feedback
</SidebarItem>
```


-----------------
FILE PATH: kits/remix-supabase-turbo/plugins/installing-plugins.mdoc

---
status: "published"

title: 'Installing Plugins in the Remix Supabase SaaS Starter kit'
label: 'Installing Plugins'
order: 0
description: 'Learn how to install plugins in the Remix Supabase SaaS Starter kit.'
---


Plugins are placed into a separate repository that mirrors the original repository structure. This allows us to build the plugins using the same files and structure as the main repository.0

You may wonder why we don't include the plugins in the main repository. The reason is that plugins are optional and may not be needed by all users. By keeping them separate, we can keep the main repository clean and focused on the core functionality. Instead you can install the plugins you need when you need them.

## Installing Plugins

To install a plugin, you can use the Makerkit CLI:

```bash
npx @makerkit/cli@latest plugins install
```

This command will prompt you to select a plugin to install. Once selected, the plugin will be installed in your project.

## How Makerkit installs plugins in your project

Plugins use `git subtree` to pull in the plugin repository into the `plugins` directory of your project. This allows you to keep the plugin up-to-date by pulling in changes from the plugin repository.

If you don't want to use git subtree, you can also manually clone a copy of the plugin repository, and then manually moving the folder from `packages/plugins` into your own repository.

-----------------
FILE PATH: kits/remix-supabase-turbo/plugins/testimonials-plugin.mdoc

---
status: "published"

title: 'Testimonials Plugin in the Remix Supabase SaaS Starter kit'
label: 'Testimonials Plugin'
order: 3
description: 'Learn how to install the Testimonials plugin in the Remix Supabase SaaS Starter kit.'
---


This plugin allows Makerkit users to easily collect and manage testimonials from their customers. It integrates seamlessly with the existing Makerkit structure and provides both backend and frontend components.

## Features

1. Testimonial submission form and manual entry
2. Admin panel for managing testimonials
3. API endpoints for CRUD operations
4. Widgets components for showing testimonials on the website

## Installation

To install the plugin, run the following command:

```bash
npx @makerkit/cli plugins install
```

Please select the `testimonial` plugin from the list of available plugins.

### Add the translations

Add the following to your `apps/web/lib/i18n/locales/en/testimonials.json` file:

 ```json {% title="apps/web/lib/i18n/locales/en/testimonials.json" %}
{
  "welcomeMessage": "We'd love to hear your feedback!",
  "welcomeMessageDescription": "Your opinion helps us improve our service.",
  "textButtonText": "Write a review",
  "videoButtonText": "Record a video review",
  "backButtonText": "Choose a different review type",
  "customerName": "Your Name",
  "testimonial": "Testimonial",
  "rating": "Rating",
  "submitting": "Submitting...",
  "submitTestimonial": "Submit Testimonial",
  "errorTitle": "Sorry, something went wrong",
  "errorDescription": "Apologies, we were unable to submit your video review. Please try again later.",
  "customerNameDescription": "Your name will be displayed with your video review",
  "recording": "Recording... {{timer}}",
  "startRecording": "Start Recording",
  "stopRecording": "Stop Recording",
  "discardAndRetry": "Discard and retry",
  "successTitle": "Thank you!",
  "successDescription": "Your feedback helps us improve our services. We appreciate your time!",
  "wallOfLove": "Wall of Love",
  "videoTestimonialBy": "Video testimonial by {{customerName}}",
  "clickToPlay": "Click to play video testimonial",
  "videoTagNotSupported": "Your browser does not support the video tag.",
  "moreTestimonials": "{{count}} more testimonials"
}
```

Now, add the namespace to the loaded translations in your `apps/web/lib/i18n/i18n.settings.ts` file:

 ```tsx {% title="apps/web/lib/i18n/i18n.settings.ts" %}
export const defaultI18nNamespaces = [
  // ... your existing namespaces
  'testimonials',
];
```

### Migration

Create a new migration file by running the following command:

```bash
pnpm --filter web supabase migration new testimonials
```

And place the content you can see at `packages/plugins/testimonial/migrations/migration.sql` into the newly created file.

Then run the migration:

```bash
pnpm run supabase:web:reset
```

And update the types:

```bash
pnpm run supabase:web:typegen
```

Once the plugin is added to your repository, please install it as a dependency in your application in the package.json file of `apps/web`:

```json
{
    "dependencies": {
      "@kit/testimonial": "workspace:*"
    }
}
```

Now run the following command to install the plugin:

```bash
pnpm i
```

You can now add the Admin pages from where you can manage the testimonials. To do so, add the following code to the `apps/web/app/routes/admin.testimonials._index/route.tsx` file:

```tsx
import { LoaderFunctionArgs } from '@remix-run/server-runtime';

import {
  testimonialsAction,
  testimonialsLoader,
} from '@kit/testimonial/admin/api';
import { TestimonialsPage } from '@kit/testimonial/admin/pages';

export const loader = (args: LoaderFunctionArgs) => {
  return testimonialsLoader(args);
};

export const action = (args: LoaderFunctionArgs) => {
  return testimonialsAction(args);
};

export default TestimonialsPage;
```

And now the testimonial's page at  `apps/web/app/routes/admin.testimonials.$id._index/route.ts`:

```tsx
import { LoaderFunctionArgs } from '@remix-run/server-runtime';

import {
  TestimonialPage,
} from '@kit/testimonial/admin/pages';

import {
  testimonialLoader,
  testimonialAction,
} from '@kit/testimonial/admin/api';

export const loader = (args: LoaderFunctionArgs) => {
  return testimonialLoader(args);
};

export const action = (args: LoaderFunctionArgs) => {
  return testimonialAction(args);
};

export default TestimonialPage;
```

Add the sidebar item as well at `apps/web/app/routes/admin/_components/admin-sidebar.tsx`:

```tsx
<SidebarItem
    path={'/admin/testimonials'}
    Icon={<StarIcon className={'h-4'} />}
>
    Testimonials
</SidebarItem>
```

You can now run the application and navigate to the Admin panel to manage the testimonials.

## Displaying the Testimonial Form

To display the testimonial form on your website, you can import the form component from the plugin and use it in your page.

Create a new component, and import the form:

```tsx
import { useState } from 'react';

import {
  TestimonialContainer,
  TestimonialForm,
  TestimonialSuccessMessage,
  VideoTestimonialForm,
} from '@kit/testimonial/client';

export function Testimonial() {
  const [success, setSuccess] = useState(false);
  const onSuccess = () => setSuccess(true);

  if (success) {
    return <SuccessMessage />;
  }

  return (
    <TestimonialContainer
      className={
        'w-full max-w-md rounded-lg border bg-background p-8 shadow-xl'
      }
      welcomeMessage={<WelcomeMessage />}
      enableTextReview={true}
      enableVideoReview={true}
      textReviewComponent={<TestimonialForm onSuccess={onSuccess} />}
      videoReviewComponent={<VideoTestimonialForm onSuccess={onSuccess} />}
      textButtonText="Write your thoughts"
      videoButtonText="Share a video message"
      backButtonText="Switch review method"
    />
  );
}

function SuccessMessage() {
  return (
    <div
      className={
        'w-full max-w-md rounded-lg border bg-background p-8 shadow-xl'
      }
    >
      <div className="flex flex-col items-center space-y-4 text-center">
        <div className="space-y-1">
          <h1 className="text-2xl font-semibold">
            Thank you for your feedback!
          </h1>

          <p className="text-muted-foreground">
            Your review has been submitted successfully.
          </p>
        </div>

        <div>
          <TestimonialSuccessMessage />
        </div>
      </div>
    </div>
  );
}

function WelcomeMessage() {
  return (
    <div className="flex flex-col items-center space-y-1 text-center">
      <h1 className="text-2xl font-semibold">
        We&apos;d love to hear your feedback!
      </h1>

      <p className="text-muted-foreground">
        Your opinion helps us improve our service.
      </p>
    </div>
  );
}
```

Please customize the components as needed to fit your website's design.

## API Endpoints

Please add the GET endpoint to fetch the testimonials at `apps/web/app/routes/api.testimonials._index/route.ts`:

```ts
import { ActionFunctionArgs } from '@remix-run/server-runtime';

import { createTestimonialsLoader, createAddTestimonialAction } from '@kit/testimonial/server';

export const action = ({ request }: ActionFunctionArgs) => {
  return createAddTestimonialAction(request);
};

export const loader = ({ request }: ActionFunctionArgs) => {
  return createTestimonialsLoader(request);
};
```

## Widgets

To display the testimonials on your website, you can use the following widget:

```tsx
import { TestimonialWallWidget } from '@kit/testimonial/widgets';

export default function TestimonialWidgetPage() {
    return (
        <div className={'flex h-full w-screen flex-1 flex-col items-center py-16'}>
            <TestimonialWallWidget />
        </div>
    );
}
```

Done! You now have a fully functional Testimonial Collection plugin integrated with your Makerkit application.


