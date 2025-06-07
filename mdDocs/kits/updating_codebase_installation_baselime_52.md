-----------------
FILE PATH: kits/next-supabase-turbo/installation/updating-codebase.mdoc

---
status: "published"
title: "Updating your Next.js Supabase Turbo Starter Kit"
label: "Updating the Codebase"
order: 6
description: "Learn how to update your Next.js Supabase Turbo Starter Kit to the latest version."
---

This guide will walk you through the process of updating your codebase by pulling the latest changes from the GitHub repository and merging them into your project. This ensures you're always equipped with the latest features and bug fixes.

If you've been following along with our previous guides, you should already have a Git repository set up for your project, with an `upstream` remote pointing to the original repository.

Updating your project involves fetching the latest changes from the `upstream` remote and merging them into your project. Let's dive into the steps!

{% sequence title="Steps to update your codebase" description="Learn how to update your Next.js Supabase Turbo Starter Kit to the latest version." %}
[Stashing your changes (if any)](#0.-stashing-your-changes-(if-any))

[Refresh the `upstream` remote](#1.-refresh-the-remote)

[Resolve any conflicts](#2.-resolve-any-conflicts)

[Run a health check on your project](#run-a-health-check-on-your-project-after-resolving-conflicts)

[Merge the changes](#3.-merge-the-changes)
{% /sequence %}

## 0. Stashing your changes (if any)

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

Create a new branch for your updates from the `main` branch:

```bash
git checkout -b update-codebase-<date>
```

In this way, you can keep track of your updates and visualize the changes in the branch before merging them into your main branch.

Now, fetch the latest changes from the `upstream` remote. You can do this by running the following command:

```bash
git pull upstream main
```

When prompted the first time, please opt for **merging instead of rebasing**.

Now, run `pnpm i` to update the dependencies:

```bash
pnpm i
```

## 2. Resolve any conflicts

Encountered conflicts during the merge? No worries! You'll need to resolve them manually. Git will highlight the files with conflicts, and you can edit them to resolve the issues.

### 2.1 Conflicts in the lock file "pnpm-lock.yaml"

If you find conflicts in the `pnpm-lock.yaml` file, accept either of the two changes (avoid manual edits), then run:

```bash
pnpm i
```

Your lock file will now reflect both your changes and the updates from the `upstream` repository. 🎉

### 2.2 Conflicts in the DB types "database.types.ts"

Your types might differ from those in the `upstream` repository, so you'll need to rebuild them and accept the latest state of the DB.

To do this, first you want to reset the DB to apply the latest changes from the `upstream` repository:

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

## 3. Merge the changes

If everything looks good, commit the changes and push them to your remote repository:

```bash
git commit -m "COMMIT_MESSAGE"
git push origin update-codebase-<date>
```

Once the changes are pushed, you can create a pull request to merge the changes into the `main` branch, assuming all is working fine.

Your project is now up to date with the latest changes from the `upstream` repository. 🎉

-----------------
FILE PATH: kits/next-supabase-turbo/installation.mdoc

---
status: "published"
title: "Installation in Next.js Supabase Turbo"
label: "Installation"
order: 1
description: "Installation in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Hi, welcome to the installation guide for the Next.js Supabase SaaS Kit Turbo. This guide will help you set up your development environment and get started with the kit.

In this section, we will cover:

1. [Introduction](introduction): the second version of Makerkit
2. [Technical Details](technical-details): the technical details of this kit
3. [Conventions](conventions): some of the conventions used in this kit
4. [Clone the Repository](clone-repository): cloning the repository
5. [Running the Project](running-the-project): running the project
6. [Common Commands](common-commands): some common commands to help you work with the project
7. [Updating the Project](updating-codebase): how to update the codebase



-----------------
FILE PATH: kits/next-supabase-turbo/monitoring/baselime.mdoc

---
status: "published"
title: 'Configuring Baselime in your Next.js Supabase SaaS kit'
label: 'Baselime'
order: 1
description: 'Learn how to configure Baselime in your Next.js Supabase SaaS kit'
---


[Baselime](https://baselime.io) (now part of Cloudflare) is an observability platform that helps you monitor your application's performance and errors. In this guide, you'll learn how to configure Baselime in your Next.js Supabase SaaS kit.

To use [Baselime](https://baselime.io) to capture exceptions and performance metrics of your app, please define the below variables:

```bash
NEXT_PUBLIC_BASELIME_KEY=your_key
NEXT_PUBLIC_MONITORING_PROVIDER=baselime
```

You can find your Baselime key in the Baselime dashboard.

If you also want to enable OpenTelemetry, you can set the following variables:

```bash
ENABLE_MONITORING_INSTRUMENTATION=true
INSTRUMENTATION_SERVICE_NAME=your_service_name
```


-----------------
FILE PATH: kits/next-supabase-turbo/monitoring/overview.mdoc

---
title: "Setting Monitoring in Makerkit | Next.js SaaS Boilerplate"
status: "published"
label: "How Monitoring works"
position: 0
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the Next.js Supabase SaaS kit"
---

{% sequence title="Steps to configure monitoring" description="Learn how to configure monitoring in the Next.js Supabase Starter Kit." %}

[Monitoring Providers](#monitoring-providers)

[Configuring Monitoring in Makerkit](#configuring-monitoring-in-makerkit)

[Performance Monitoring](#performance-monitoring)

{% /sequence %}

## Monitoring Providers
Makerkit provides first-class support for two monitoring providers:

1. [Baselime](https://baselime.io) (now part of Cloudflare)
2. [Sentry](https://sentry.io)

Our plugin system can allow you to add support for other providers in the future. For example, [here we added support for Telegram](/blog/tutorials/telegram-saas-error-monitoring).

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
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using Next.js

For the provider's specific settings, please check the relative documentation.

## Performance Monitoring

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using Next.js's instrumentation hook
2. **Client exceptions** - Automatically capturing uncaught exceptions on the client
3. **Server exceptions** - Automatically capturing server-side exceptions captured by the [Next.js instrumentation hook](https://nextjs.org/docs/app/api-reference/file-conventions/instrumentation#onrequesterror-optional)

Additionally, it provides you with the tools to manually capturing exceptions, should you want to.

-----------------
FILE PATH: kits/next-supabase-turbo/monitoring/sentry.mdoc

---
status: "published"
description: 'Learn how to configure Sentry in your Next.js Supabase SaaS kit'
title: 'Configuring Sentry in your Next.js Supabase SaaS kit'
label: 'Sentry'
order: 2
---

{% sequence title="Steps to configure Sentry" description="Learn how to configure Sentry in your Next.js Supabase SaaS kit." %}

[Configuring Sentry in Makerkit](#configuring-sentry-in-makerkit)

[Uploading source maps](#uploading-source-maps)

{% /sequence %}

## Configuring Sentry in Makerkit

[Sentry](https://sentry.com) is an observability platform that helps you monitor your application's performance and errors. In this guide, you'll learn how to configure Sentry in your Next.js Supabase SaaS kit.

To use [Sentry](https://sentry.com) to capture exceptions and performance metrics of your app, please define the below variables:

```bash
NEXT_PUBLIC_MONITORING_PROVIDER=sentry
NEXT_PUBLIC_SENTRY_DSN=your_dsn
```

Please install the package `@sentry/nextjs` in `apps/web/package.json` as a dependency.

```bash
pnpm i @sentry/nextjs --filter web
```

Finally, update the Next.js configuration in your `next.config.js` file:

 ```tsx {% title="next.config.mjs" %}
import { withSentryConfig } from '@sentry/nextjs';

// wrap your Next.js configuration with the Sentry configuration
withSentryConfig(nextConfig);
```

You can find your Sentry DSN in the Sentry dashboard.

## Uploading source maps

To upload source maps to Sentry, use the following options:

 ```tsx {% title="next.config.mjs" %}
import { withSentryConfig } from '@sentry/nextjs';

export default withSentryConfig(
  withBundleAnalyzer({
    enabled: process.env.ANALYZE === 'true',
  })(config),
  {
    org: 'your-sentry-org-name',
    project: 'your-sentry-project-name',

    // An auth token is required for uploading source maps.
    authToken: process.env.SENTRY_AUTH_TOKEN,

    silent: !IS_PRODUCTION, // Used to suppress logs
    autoInstrumentServerFunctions: false,
    widenClientFileUpload: true,
  },
);
```

And make sure to add the `SENTRY_AUTH_TOKEN` to your CI environment variables.

```bash
SENTRY_AUTH_TOKEN=your_auth_token
```


-----------------
FILE PATH: kits/next-supabase-turbo/monitoring.mdoc

---
status: "published"
title: "Setting Monitoring in Makerkit | Next.js SaaS Boilerplate"
label: "Monitoring"
order: 12
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the Next.js Supabase SaaS kit"
collapsible: true
collapsed: true
---

Makerkit provides first-class support for two monitoring providers:

1. Baselime (now part of Cloudflare)
2. Sentry

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using Next.js's instrumentation hook
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
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using Next.js

For the provider's specific settings, please check the relative documentation.

#### Performance Monitoring

Performance monitoring uses Next.js' experimental instrumentation for reporting performance metrics using OpenTelemetry.

NB: Performance monitoring is only enabled for Node.js runtimes.


-----------------
FILE PATH: kits/next-supabase-turbo/notifications/notifications-configuration.mdoc

---
status: "published"
title: "Configuring Notifications in the Next.js Supabase Starter Kit"
description: "Learn how to configure notifications in the Next.js Supabase Starter Kit."
label: "Notifications Configuration"
order: 0
---

Makerkit allows you to configure and send notifications using the `@kit/notifications` package. The package provides a simple API to send notifications.

{% sequence title="Steps to configure notifications" description="Learn how to configure notifications in the Next.js Supabase Starter Kit." %}

[Enabling notifications](#enabling-notifications)

[Enabling real-time notifications](#enabling-real-time-notifications)

{% /sequence %}

There are two variables that you can set in the `.env` file to configure notifications:

1. **NEXT_PUBLIC_ENABLE_NOTIFICATIONS**: Set this to `true` to enable notifications.
2. **NEXT_PUBLIC_REALTIME_NOTIFICATIONS**: Set this to `true` to enable real-time notifications.

```bash
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=true
NEXT_PUBLIC_REALTIME_NOTIFICATIONS=true
```

## Enabling notifications
If `NEXT_PUBLIC_ENABLE_NOTIFICATIONS` is set to `false`, the bell icon will not appear in the header. Otherwise, it will appear.

```bash
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=false # Disable notifications
```

## Enabling real-time notifications

If `NEXT_PUBLIC_REALTIME_NOTIFICATIONS` is set to `false`, the notifications will not be real-time. Otherwise, they will be real-time.

```bash
NEXT_PUBLIC_REALTIME_NOTIFICATIONS=false # Disable real-time notifications  
```

If you want to enable real-time notifications, you need to set the `NEXT_PUBLIC_REALTIME_NOTIFICATIONS` environment variable to `true`. This will enable real-time notifications in your application.

This may be your preference for a more cost-effective solution, as it will reduce the number of requests to the server.

### Why would you not want to enable notifications?

If you are not using notifications in your application, you can disable them to reduce the complexity of your application. This can be useful if you are building a simple application and do not need notifications.


-----------------
FILE PATH: kits/next-supabase-turbo/notifications/sending-notifications.mdoc

---
status: "published"
title: "Sending a notification in the Next.js Supabase Starter Kit"
description: "Learn how to send a notification in the Next.js Supabase Starter Kit."
label: "Sending Notifications"
order: 1
---

Makerkit allows you to send notifications using the `@kit/notifications` package. The package provides a simple API to send notifications.

{% sequence title="Notifications" description="Learn how to send notifications in the Next.js Supabase Starter Kit." %}

[Notifications API](#notifications-api)

[Notification channels](#notification-channels)

[Notification types](#notification-types)

[Notification links](#notification-links)

{% /sequence %}

## Notifications API

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

## Notification channels

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


NB: at the time of writing, this is not yet implemented in the Next.js Supabase Starter Kit. It is a feature that is planned for the future.

## Notification types

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

## Notification Links

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
FILE PATH: kits/next-supabase-turbo/notifications.mdoc

---
status: "published"
title: "Notifications in Next.js Supabase Turbo"
label: "Notifications"
order: 10
description: "Notifications in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Add notifications to your application:

1. [**Notification Configuration**](/docs/next-supabase-turbo/notifications/notifications-configuration)
   Set up and configure your notification system.
2. [**Sending Notifications**](/docs/next-supabase-turbo/notifications/sending-notifications)
   Learn how to send notifications to your users.

-----------------
FILE PATH: kits/next-supabase-turbo/plugins/chatbot-plugin.mdoc

---
status: "published"
title: 'Add an AI Chatbot to the Next.js Supabase SaaS Starter kit'
label: 'AI Chatbot'
order: 4
description: 'Add an AI chatbot to your Next.js Supabase SaaS Starter kit to provide instant support to your users.'
---

This plugin allows you to create a chatbot for your app. Users can interact with the chatbot to get information about
your app.

The chatbot will index your app's content and provide answers to user queries.

## Installation

Install the chatbot plugin from your main app:

```bash
npx @makerkit/cli plugins install chatbot
```

Now, install the plugin from your main app by adding the following to your `package.json` file:

 ```json {% title="apps/web/package.json" %}
{
  "dependencies": {
    "@kit/chatbot": "workspace:*"
  }
}
```

And then run `pnpm install` to install the plugin.

## Import the Chatbot component

You can now import the Chatbot component in your app:

```tsx
import { Chatbot } from '@kit/chatbot';

<Chatbot sitename={'Makerkit'} />
```

## Create the API route handler

You can add this anywhere since it's configurable, but my suggestion is to add it to the `api/chat` directory. Create a new file called `route.ts` at `apps/web/app/api/chat/route.ts`:

```ts
import { handleChatBotRequest } from '@kit/chatbot/server';

export const POST = handleChatBotRequest;
```

If you use a different path, please set the `NEXT_PUBLIC_CHATBOT_API_URL` environment variable:

```bash
NEXT_PUBLIC_CHATBOT_API_URL=/api/some-other-path
```

## Add the Migration file

Copy the migration file at `packages/plugins/chatbot/migrations` into your apps migration directory at `apps/web/supabase/migrations`.

Make sure to reset the database after copying the migration file and make sure to push this to your remote instance.

### Tailwind CSS Styles

If using TailwindCSS v3, please update the Tailwind CSS styles to include the new folder:

```js { % title="tooling/tailwind/index.ts" %}
export default {
  darkMode: ['class'],
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/chatbot/src/**/*.tsx'  // <-- add this line
    '!**/node_modules',
  ],
  // ...
};
```

The above is not required if you are using TailwindCSS v4.

If using TailwindCSS v4, please add the following style to `apps/wer/styles/globals.css`:

```css {% title="apps/web/styles/globals.css" %}
@import '@kit/chatbot/style';
```

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
<Chatbot
    settings={{
        title: 'Chat with us',
        order: 'bottom-right',
        branding: {
            primaryColor: '#000',
            accentColor: '#000',
            textColor: '#fff',
        },
    }}
/>
```

## Indexing Content

At this time - the Chatbot will crawl your app's content and index it - by reading the sitemap.xml file.

The crawler will infer the sitemap.xml from the website's robots.txt file. Otherwise, you can provide the sitemap.xml
URL using the `CHATBOT_WEBSITE_SITEMAP_URL` environment variable.

```bash
CHATBOT_WEBSITE_SITEMAP_URL=https://example.com/sitemap.xml
```

To index your website's content, the chatbot will read the `sitemap.xml` file. If the sitemap of your website is not located at `/sitemap.xml`, you can set the `CHATBOT_WEBSITE_SITEMAP_URL` environment variable to the correct URL.

```bash
CHATBOT_WEBSITE_SITEMAP_URL=https://example.com/my-sitemap.xml
```

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

### Chatbot Configuration

You can customize the chatbot model by providing the following environment variables at `apps/web/.env.local`:

```bash {% title="apps/web/.env.local" %}
LLM_MODEL_NAME=gpt-3.5-turbo
LLM_BASE_URL=
LLM_API_KEY=
```

The model needs to be compatible with the OpenAI API.

1. `LLM_MODEL_NAME` - The model name to use. For example, `gpt-3.5-turbo`.
2. `LLM_BASE_URL` - The base URL for the API. For example, `https://api.openai.com/v1`. If you use OpenAI, you can leave this empty.
3. `LLM_API_KEY` - The API key to use. You can get this from OpenAI.

If you use an OpenAI-compatible model, you can set the `LLM_BASE_URL` to the URL of the API. Otherwise, you can leave it empty.

### Chatbot Indexer

In addition, we need to provide the following environment variables for the chatbot indexer:

{% alert type="info" %}
The chatbot indexer is a separate service that indexes your app's content and provides answers to user queries. As such, the `.env.local` file is located in the `packages/plugins/chatbot` directory, not in `apps/web`.
{% /alert %}

``` bash {% title="packages/plugins/chatbot/.env.local" %}
SUPABASE_SERVICE_ROLE_KEY=
NEXT_PUBLIC_SUPABASE_URL=
LLM_MODEL_NAME=
LLM_BASE_URL=
LLM_API_KEY=
```

1. `SUPABASE_SERVICE_ROLE_KEY` - The service role key for the Supabase instance.
2. `NEXT_PUBLIC_SUPABASE_URL` - The URL of the Supabase instance.

By adjusting these variables, you can choose which Supabase instance to use for the chatbot indexer. Normally, you would use the same Supabase instance as your app.

You can also use your local Supabase instance for development purposes.

In that case, these values would be the same ones you're using for your app's Supabase instance in `apps/web/.env.development`.


-----------------
FILE PATH: kits/next-supabase-turbo/plugins/feedback-plugin.mdoc

---
status: "published"
title: 'Add a Feedback Widget plugin to your Next.js Supabase SaaS Starter kit'
label: 'Feedback Widget'
order: 5
description: 'Add a Feedback Widget plugin to your Next.js Supabase SaaS Starter kit'
---


This plugin is a lighter version of the Roadmap plugin. It is recommended to install the Roadmap plugin if you need more features.

The feedback plugin allows you to add a feedback widget to your app. Users can provide feedback on your app, and you can view and manage feedback submissions in the admin panel.

### Installation

Pull the plugin from the main repository:

```
npx @makerkit/cli@latest plugins install feedback
```

Now, install the plugin from your main app by adding the following to your `package.json` file:

 ```json {% title="apps/web/package.json" %}
{
  "dependencies": {
    "@kit/feedback": "workspace:*"
  }
}
```

And then run `pnpm install` to install the plugin.

### Add the translations

Add the following to your `apps/web/public/locales/en/feedback.json` file:

 ```json {% title="apps/web/locales/en.json" %}
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
  'common',
  'auth',
  'account',
  'teams',
  'billing',
  'marketing',

  // Add the feedback namespace
  'feedback',
];
```

### Add the migrations

Since the feedback relies on a new table in the database, you will need to run the migrations to create the table. Please create a new migration file:

```
pnpm --filter web supabase migration new feedback
```

And copy the content of the migration file from the plugin repository to your new migration file.

Run the reset command to apply the migration:

```
pnpm run supabase:web:reset
```

### Import the component

Now, you can import the component from the plugin:

```tsx
import { FeedbackPopup } from '@kit/feedback';
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

Add the following to your `apps/web/admin/feedback/page.tsx` file:

 ```tsx {% title="apps/web/admin/feedback/page.tsx" %}
import { FeedbackSubmissionsPage } from '@kit/feedback/admin';

export default FeedbackSubmissionsPage;
```

And the submission detail page at `apps/web/admin/feedback/[id]/route.tsx`:

 ```tsx {% title="apps/web/admin/feedback/[id]/page.tsx" %}
import { FeedbackSubmissionPage } from '@kit/feedback/admin';

export default FeedbackSubmissionPage;
```

Add the sidebar item to `apps/web/app/admin/_components/admin-sidebar.tsx`:

 ```tsx {% title="apps/web/app/admin/_components/admin-sidebar.tsx" %}
<SidebarItem
    path={'/admin/feedback'}
    Icon={<MessageCircle className={'h-4'} />}
    >
    Feedback
</SidebarItem>
```

### Tailwind CSS Styles

If using TailwindCSS v3, please update the Tailwind CSS styles to include the new folder:

```js { % title="tooling/tailwind/index.ts" %}
export default {
  darkMode: ['class'],
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/feedback/src/**/*.tsx'  // <-- add this line
    '!**/node_modules',
  ],
  // ...
};
```

The above is not required if you are using TailwindCSS v4.

