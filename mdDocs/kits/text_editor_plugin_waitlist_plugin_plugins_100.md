-----------------
FILE PATH: kits\remix-supabase-turbo\plugins\text-editor-plugin.mdoc

---
status: "published"

title: 'Add an AI Text Editor to the Remix Supabase SaaS Starter kit'
label: 'Text Editor'
order: 3
description: 'Add an AI text editor to your Remix Supabase SaaS Starter kit to provide a rich text editing experience to your users.'
---


**NB:** this is currently a work in progress and not yet available.

A Text Editor built with Lexical.

### Installation

Pull the plugin from the main repository:

```
npx @makerkit/cli@latest plugins install text-editor
```

Now, install the plugin from your main app by adding the following to your `package.json` file:

 ```json {% title="apps/web/package.json" %}
{
  "dependencies": {
    "@kit/text-editor": "workspace:*"
  }
}
```

And then run `pnpm install` to install the plugin.

### AI Routes

Add the following AI Routes to your Remix API routes:

One route at `apps/web/app/routes/api.editor.edit._index/route.ts`:

```ts
import { ActionFunctionArgs } from '@remix-run/server-runtime';

import { createAIEditRouteHandler } from '@kit/text-editor/server';

export const action = ({ request }: ActionFunctionArgs) => {
  return createAIEditRouteHandler(request);
};
```

And another route at `apps/web/app/routes/api.editor.autocomplete._index/route.ts`:

```ts
import { ActionFunctionArgs } from '@remix-run/server-runtime';

import { createAIAutocompleteRouteHandler } from '@kit/text-editor/server';

export const action = ({ request }: ActionFunctionArgs) => {
  return createAIAutocompleteRouteHandler(request);
};
```

### Environment Variables

Make sure to add the variable `OPENAI_API_KEY` to your `.env.local` file to test locally, and make sure it's added to your production environment variables.

### Import the component

Now, you can import the component from the plugin:

```tsx
import { TextEditor } from '@kit/text-editor';
import '@kit/text-editor/style';
```

And use it in your app:

```tsx
<TextEditor />
```

You can assign the following props to the `TextEditor` component:

```tsx
{
  className?: string;
  content?: string;
  placeholder?: () => React.ReactElement;
  onChange?: (content: string) => void;
}
```


-----------------
FILE PATH: kits\remix-supabase-turbo\plugins\waitlist-plugin.mdoc

---
status: "published"

title: 'Add a Waitlist to the Remix Supabase SaaS Starter kit'
label: 'Waitlist'
order: 1
description: 'Add a waitlist to your Remix Supabase SaaS Starter kit to collect emails from interested users.'
---


In this guide, you will learn how to add a waitlist to your Remix Supabase SaaS Starter kit to collect emails from interested users. This feature is useful for building an audience before launching your product.

This plugin allows you to create a waitlist for your app. Users can sign up for the waitlist and receive an email when the app is ready.

### How it works

1. You disable sign up in your app from Supabase. This prevents any user from using the public API to sign up.
2. We create a new table in Supabase called `waitlist`. Users will sign up for the waitlist and their email will be stored in this table.
3. When you want to enable a sign up for a user, mark users as `approved` in the `waitlist` table.
4. The database trigger will create a new user in the `auth.users` table and send an email to the user with a link to set their password.
5. The user can now sign in to the app and update their password.

### Installation

Install the waitlist plugin from your main app:

```
pnpm add --filter web @kit/waitlist
```

#### Add migrations to your app

Make sure to place the waitlist migration in your app's migrations by moving the migration file from the plugin to the app's migration folder at `apps/web/supabase/migrations`.

Now, re-run the migrations:

```
pnpm run supabase:web:reset
pnpm run supabase:web:typegen
```

You can now use the waitlist plugin in your app.

#### Replace the sign up form

Replace your sign up form with the waitlist form:

```tsx
import { Link, MetaFunction, redirect, useLoaderData } from '@remix-run/react';
import { LoaderFunctionArgs } from '@remix-run/server-runtime';

import { SignUpMethodsContainer } from '@kit/auth/sign-up';
import { requireUser } from '@kit/supabase/require-user';
import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { Button } from '@kit/ui/button';
import { Heading } from '@kit/ui/heading';
import { If } from '@kit/ui/if';
import { Trans } from '@kit/ui/trans';
import { signUpToWaitlistAction } from '@kit/waitlist/action';
import { WaitlistSignupForm } from '@kit/waitlist/client';

import authConfig from '~/config/auth.config';
import pathsConfig from '~/config/paths.config';
import { createI18nServerInstance } from '~/lib/i18n/i18n.server';

export const loader = async ({ request }: LoaderFunctionArgs) => {
  const i18n = await createI18nServerInstance(request);

  const inviteToken =
    new URL(request.url).searchParams.get('invite_token') ?? '';

  const user = await requireUser(getSupabaseServerClient(request));

  if (user.data) {
    return redirect(pathsConfig.app.home);
  }

  return {
    title: i18n.t('auth:signUp'),
    inviteToken,
  };
};

export const meta: MetaFunction<typeof loader> = ({ data }) => {
  return [
    {
      title: data?.title,
    },
  ];
};

const paths = {
  callback: pathsConfig.auth.callback,
  appHome: pathsConfig.app.home,
};

const waitListEnabled = import.meta.env.VITE_WAITLIST_ENABLED === 'true';

export default function SignUpPage() {
  const { inviteToken } = useLoaderData<typeof loader>();

  const signInPath =
    pathsConfig.auth.signIn +
    (inviteToken ? `?invite_token=${inviteToken}` : '');

  return (
    <>
      <Heading level={4}>
        <Trans i18nKey={'auth:signUpHeading'} />
      </Heading>

      <If condition={!waitListEnabled} fallback={<WaitlistSignupForm />}>
        <SignUpMethodsContainer
          providers={authConfig.providers}
          inviteToken={inviteToken}
          paths={paths}
        />
      </If>

      <div className={'justify-centers flex'}>
        <Button asChild variant={'link'} size={'sm'}>
          <Link to={signInPath}>
            <Trans i18nKey={'auth:alreadyHaveAnAccount'} />
          </Link>
        </Button>
      </div>
    </>
  );
}

export const action = async ({ request }: LoaderFunctionArgs) => {
  const json = await request.json();

  if (!waitListEnabled) {
    return new Response(null, { status: 404 });
  }

  if (request.method !== 'POST') {
    return new Response(null, { status: 405 });
  }

  return signUpToWaitlistAction(json);
};
```

In the example above, we check if the waitlist is enabled. If it is, we show the waitlist form. If it's not, we show the sign up form.

If you want to do the same, make sure to add the following environment variable to your `.env` file:

```
VITE_WAITLIST_ENABLED=true
```

### Adding the Database Webhook to listen for new signups

Let's extend the DB handler at `apps/web/app/api/db/webhook/route.ts`:

```tsx
import { LoaderFunctionArgs } from '@remix-run/server-runtime';

import { getDatabaseWebhookHandlerService } from '@kit/database-webhooks';
import appConfig from '~/config/app.config';
import pathsConfig from '~/config/paths.config';

/**
 * @description POST handler for the webhook route that handles the webhook event
 */
export const action = async ({ request }: LoaderFunctionArgs) => {
  const service = getDatabaseWebhookHandlerService();

  try {
    // handle the webhook event
    await service.handleWebhook(request, {
      async handleEvent(payload) {
        if (payload.table === 'waitlist' && payload.record.approved) {
          const { handleApprovedUserChange } = await import('@kit/waitlist/server');
          const redirectTo = appConfig.url + pathsConfig.auth.passwordUpdate;

          await handleApprovedUserChange({
            email: payload.record.email,
            redirectTo,
          });
        }
      },
    });

    // return a successful response
    return new Response(null, { status: 200 });
  } catch (error) {
    // return an error response
    return new Response(null, { status: 500 });
  }
};
```

During dev, you can simply add the webhook to your seed file `apps/web/supabase/seed.sql`:

```sql
create trigger "waitlist_approved_update" after update
on "public"."waitlist"
when (new.approved = true)
for each row
execute function "supabase_functions"."http_request"(
  'http://host.docker.internal:5173/api/db/webhook',
  'POST',
  '{"Content-Type":"application/json", "X-Supabase-Event-Signature":"WEBHOOKSECRET"}',
  '{}',
  '5000'
);
```

The above creates a trigger that listens for updates to the `waitlist` table and sends a POST request to the webhook route.

**Note**: You need to add this trigger to your production database as well. You will replace your `WEBHOOKSECRET` with the secret you set in your `.env` file and the `host.docker.internal:3000` with your production URL.
Just like you did for the other existing triggers.

### Approving users

Simply update the `approved` column in the `waitlist` table to `true` to approve a user. You can do so from the Supabase dashboard or by running a query.

Alternatively, run an update based on the created_at timestamp:

```sql
update public.waitlist
set approved = true
where created_at < '2024-07-01';
```

### Email Templates and URL Configuration

Please make sure to [edit the email template](https://makerkit.dev/docs/remix-supabase-turbo/authentication-emails) in your Supabase account.
The default email in Supabase does not support PKCE and therefore does not work. By updating it - we replace the existing strategy with the token-based strategy - which the `confirm` route in Makerkit can support.

Additionally, [please add the following URL to your Supabase Redirect URLS allow list](https://supabase.com/docs/guides/auth/redirect-urls):

```
<your-url>/password-reset
```

This will allow Supabase to redirect users to your app to set their password after they click the email link.

If you don't do this - the email links will not work.

### Translations

Add the following translations to your `auth.json` translation:

```json
{
  "waitlist": {
    "heading": "Join the Waitlist for Early Access",
    "submitButton": "Join Waitlist",
    "error": "Ouch, we couldn't add you to the waitlist. Please try again.",
    "errorDescription": "We couldn't add you to the waitlist. If you have already signed up, you are already on the waitlist.",
    "success": "You're on the waitlist!",
    "successDescription": "We'll let you know when you can sign up."
  }
}
```

---


Easy peasy! Now you have a waitlist for your app.


-----------------
FILE PATH: kits\remix-supabase-turbo\plugins.mdoc

---
status: "published"
title: "Plugins in Remix Supabase Turbo"
label: "Plugins"
order: 14
description: "Plugins in Remix Supabase Turbo"
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
FILE PATH: kits\remix-supabase-turbo\translations\adding-translations.mdoc

---
status: "published"
title: "Adding new translations | Remix Supabase SaaS Kit"
label: "Adding new translations"
description: "Learn how to add new translations to your  Remix Supabase SaaS project."
order: 1
---

Translations are stored at application level and can be found at `apps/web/public/locales`. Translations for English (which is the only language provided by default) will be found at `apps/web/public/locales/en`.

If you wish to provide a new language, there are two steps.

### 1. Add the language files

Create a new folder using the correct language code at `apps/web/public/locales/[lng]` where `[lng]` can be any correct language code, such as `de`, `it`, `es`, `ja`, etc.

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
FILE PATH: kits\remix-supabase-turbo\translations\language-selector.mdoc

---
status: "published"
title: "Adding a language selector to your application | Remix Supabase SaaS Kit"
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
FILE PATH: kits\remix-supabase-turbo\translations\using-translations.mdoc

---
status: "published"

title: "Using translations in your Remix Supabase project"
label: "Using translations"
description: "Makerkit uses i18next to translate your project into multiple languages. This guide will show you how to use translations in your Remix Supabase project."
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

-----------------
FILE PATH: kits\remix-supabase-turbo\translations.mdoc

---
status: "published"
title: "Translations in Remix Supabase Turbo"
label: "Translations"
order: 10
description: "Translations in Remix Supabase Turbo"
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
FILE PATH: kits\remix-supabase-turbo\troubleshooting\troubleshooting-authentication.mdoc

---
status: "published"

title: 'Troubleshooting authentication issues in the Remix Supabase kit'
label: 'Authentication'
order: 2
description: 'Troubleshoot issues related to authentication in the Remix Supabase SaaS kit'
---


## Supabase redirects to localhost instead of my website URL

This is most likely an issue related to not having set up the Authentication URL in the Supabase settings as describe in [the going to production guide](going-to-production/supabase);

## Cannot Receive Emails from Supabase

This issue may arise if you have not setup an SMTP provider from within Supabase.

Supabase's own SMTP provider has low limits and low deliverability, therefore you must set up your own - and only use the default Supabase's one for testing purposes.

## Cannot sign in with oAuth provider

This is most likely a settings issues from within the Supabase dashboard or in the provider's settings.

Please [read Supabase's documentation](https://supabase.com/docs/guides/auth/social-login) on how to set up third-party providers.


-----------------
FILE PATH: kits\remix-supabase-turbo\troubleshooting\troubleshooting-billing.mdoc

---
status: "published"
title: 'Troubleshooting billing issues in the Remix Supabase kit'
label: 'Billing'
order: 3
description: 'Troubleshoot issues related to billing in the Remix Supabase SaaS kit'
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
FILE PATH: kits\remix-supabase-turbo\troubleshooting\troubleshooting-deployment.mdoc

---
status: "published"

title: 'Troubleshooting deployment issues in the Remix Supabase kit'
label: 'Deployment'
order: 4
description: 'Troubleshoot issues related to deploying the application in the Remix Supabase SaaS kit'
---


## The deployment build fails

This is most likely an issue related to the environment variables not being set correctly in the deployment environment. Please analyse the logs of the deployment provider to see what is the issue.

The kit is very defensive about incorrect environment variables, and will throw an error if any of the required environment variables are not set. In this way - the build will fail if the environment variables are not set correctly - instead of deploying a broken application.

If you are deploying to Vercel, [please follow this guide](going-to-production/vercel).

-----------------
FILE PATH: kits\remix-supabase-turbo\troubleshooting\troubleshooting-emails.mdoc

---
status: "published"
title: 'Troubleshooting emails issues in the Remix Supabase kit'
label: 'Emails'
order: 5
description: 'Troubleshoot issues related to emails in the Remix Supabase SaaS kit'
---

To troubleshoot emails issues in the Remix Supabase kit, please make sure to:

1. **Correct Credentials**: please make sure you have setup the correct credentials in your environment variables. Refer to the [emails guide](../emails/email-configuration) for more information.
2. **Your domain is verified**: make sure your domain is verified in your Email provider
3. **The Database Webhooks are configured**: make sure you have configured the Database Webhooks in your Supabase project. Refer to the [Supabase deployment guide](../going-to-production/supabase) for more information. Make sure to read Webhooks are pointing to the correct URL and use the correct shared secret.
4. **The Webhooks use a public URL**: make sure the webhooks use a public URL and not behind authentication (such as a Vercel Preview URL).
5. **Read the Logs**: please verify your server logs in your hosting provider to see if there are any errors related to emails.

If you have done all the above and still having issues, please open a Support Ticket in Discord.

-----------------
FILE PATH: kits\remix-supabase-turbo\troubleshooting\troubleshooting-installation.mdoc

---
status: "published"

title: 'Troubleshooting installation issues in the Remix Supabase SaaS kit'
label: 'Installation'
order: 1
description: 'Troubleshoot issues related to installing the Remix Supabase SaaS kit'
---


## Cannot clone the repository

Issues related to cloning the repository are usually related to a Git misconfiguration in your local machine. The commands displayed in this guide using SSH: these will work only if you have setup your SSH keys in Github.

If you run into issues, [please make sure you follow this guide to set up your SSH key in Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

If this also fails, please use HTTPS instead. You will be able to see the commands in the repository's Github page under the "Clone" dropdown.

Please also make sure that the account that accepted the invite to Makerkit, and the locally connected account are the same.

## The Remix dev server does not start

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
FILE PATH: kits\remix-supabase-turbo\troubleshooting\troubleshooting-module-not-found.mdoc

---
status: "published"
title: 'Troubleshooting module not found issues in the Remix Supabase kit'
label: 'Modules not found'
order: 6
description: 'Troubleshoot issues related to modules not found in the Remix Supabase SaaS kit'
---

Let's walk through common "Module not found" errors and how to fix them in your Makerkit project.

This issue is mostly related to either dependency installed in the wrong package or issues with the file system.

### 1. Dependency Installation Issues

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

### 2. Windows OneDrive Conflicts

OneDrive can cause file system issues with Node.js projects. If you're using Windows with OneDrive:

1. Move your project outside of OneDrive-synced folders
2. Or disable OneDrive sync for your development folder

-----------------
FILE PATH: kits\remix-supabase-turbo\troubleshooting.mdoc

---
status: "published"
title: "Troubleshooting in Remix Supabase Turbo"
label: "Troubleshooting"
order: 17
description: "Troubleshooting in Remix Supabase Turbo"
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
FILE PATH: kits\supamode\configuration\mock-data.mdoc

---
status: "published"
title: "Adding mock data to Supamode"
label: "Mock Data"
order: 2
description: "How to add mock data to Supamode for development purposes"
---

Seeding the data is optional, but it can help you get started with some demo data.

The repository comes with a predefined schema (based off of Makerkit's schema) and a seed file that you can use to populate your database with mock data.

The seed file uses [Snaplet](https://snaplet.dev/) to generate realistic mock data based on the schema. This allows you to have a more realistic development environment.

## Creating a Seed File

```bash
pnpm --filter app run supabase:seed:gen
```

This command creates a seed file at `apps/app/supabase/seeds/03-demo-seed.
sql`.

## Modifying the Seed File

The default seed file is the below:

```tsx
import { copycat } from '@snaplet/copycat';
import { createSeedClient } from '@snaplet/seed';

(async () => {
  const seed = await createSeedClient({
    dryRun: true,
  });

  const { users } = await seed.users((x) =>
    x(10, (user) => {
      return {
        id: copycat.uuid(user.seed),
        email: copycat.email(user.seed),
        password: `testingpassword`,
        confirmation_token: copycat.uuid(user.seed),
        recovery_token: copycat.uuid(user.seed),
        email_change_token_new: copycat.email(user.seed),
      };
    }),
  );

  for (const user of users) {
    await seed.public_accounts(
      (x) =>
        x(1, (account) => {
          return {
            is_personal_account: false,
            name: copycat.fullName(account.seed),
            picture_url: null,
            notifications: (x) =>
              x({ min: 4, max: 20 }, (notification) => ({
                link: '',
                title: copycat.sentence(notification.seed),
                body: copycat.paragraph(notification.seed),
              })),
            subscriptions: (x) =>
              x(1, () => ({
                billing_provider: 'stripe',
                subscription_items: () => x(1),
              })),
            invitations: (x) =>
              x(3, (invitation) => ({
                email: copycat.email(invitation.seed),
                role: 'member',
                status: 'pending',
              })),
          };
        }),
      {
        connect: {
          users,
        },
      },
    );

    await seed.blog_posts(
      (x) =>
        x({ min: 4, max: 20 }, (blogPost) => ({
          title: copycat.sentence(blogPost.seed),
          tags: [copycat.word(blogPost.seed), copycat.word(blogPost.seed)],
          content: [
            copycat.paragraph(blogPost.seed),
            copycat.paragraph(blogPost.seed * 2),
            copycat.paragraph(blogPost.seed * 3),
            copycat.paragraph(blogPost.seed * 4),
          ].join('\n\n'),
          blog_post_comments: (x) =>
            x({ min: 0, max: 50 }, (comment) => ({
              content: copycat.paragraph(comment.seed),
              author_email: copycat.email(comment.seed),
            })),
        })),
      {
        connect: {
          users,
        },
      },
    );
  }

  process.exit(0);
})();
```

-----------------
FILE PATH: kits\supamode\configuration\permissions.mdoc

---
status: "published"
title: "Understanding Supamode's Permission System 🔐"
label: "Permission System"
order: 0
description: "A comprehensive guide to Supamode's role-based access control system"
---

In this comprehensive guide, we'll explore Supamode's straightforward yet powerful role-based access control (RBAC) system, designed to give you granular control over who can access what in your Supabase application.

## Key Learning Objectives:

- Master the four core components of Supamode's permission architecture
- Implement secure, multi-layered access verification
- Design scalable permission structures for your organization
- Deploy production-ready access control using seed templates

---

# The Four Core Components of Supamode 🏗️

Supamode's access control system is built around four interconnected components that work together to create secure, manageable permissions.

1. **Accounts**: User accounts linked with Supabase's auth users
2. **Roles**: Roles that define authority levels. You can assign one role to each account.
3. **Permissions**: Fine-grained access rights that control what actions
users can perform. There are two types of permissions: system permissions and data permissions.
   - **System Permissions**: Control access to Supamode's administrative interface and system functions.
   - **Data Permissions**: Control access to your actual application data with incredible granularity.
4. **Permission Groups**: Optional logical bundles of related permissions for
easier role management. A role can have multiple permission groups assigned
to it.

Example:
1. **Account**: A user has an account in Supamode, which is linked to their
Supabase auth
user.
2. **Role**: The user is assigned a role, such as "Content Editor", which
defines their authority level. All roles have a priority, which determines their authority level.
3. **Permissions**: The "Content Editor" role has permissions like "Create Blog Posts" and "Edit Own Blog Posts", which define what actions the user can perform.
4. **Permission Groups**: The "Content Editor" role is part of a "Content
Management" permission group, which bundles related permissions together for easier management.

## 1. Accounts: Your Identity Foundation 👤

**What it is:** The bridge between Supabase's authentication system and Supamode's permission management.

```typescript
// Account Structure
{
  id: "uuid-account-id",
  auth_user_id: "uuid-from-supabase-auth", // Links to auth.users
  is_active: true
}
```

**Key Concept:** Every authenticated user gets exactly one Supamode account. The account is necessary for being assigned roles and permissions, and it allows you to store additional metadata about the user.

Accounts with status `is_active: false` are considered deactivated and
cannot perform any actions in Supamode. This is useful for managing user access without deleting accounts.

## 2. Roles: Authority Levels 👑

Roles define the authority levels within your application, allowing you to control what users can do based on their assigned role.

A user can have a single role, which is assigned to their account. Roles are defined by their name, description, priority, and optional metadata.

```typescript
// Role Structure
{
  name: "Content Manager",
  description: "Manages blog posts and user content",
  priority: 70, // Higher = more authority
  metadata: {
    department: "Marketing"
  }
}
```

**Visual Hierarchy:**

```
Super Admin (Priority: 100)    ← Ultimate system control
├── Admin (Priority: 90)       ← System administration
├── Manager (Priority: 70)     ← Content management
├── Editor (Priority: 60)      ← Content editing
└── Viewer (Priority: 50)      ← Read-only access
```

**Key Concept:** Roles have priorities that determine authority levels, but **do not inherit permissions**. Each role must be explicitly assigned its own permissions.

A role with a higher priority **can perform actions** on roles that have lower priority, such as assigning them a different role, editing their account, and so on. However, they also must have the required permissions to modify accounts.

Supamode does not define any roles by default, allowing you to create a custom hierarchy that fits your application's needs.

## 3. Permissions: Granular Access Rights 🎯

**What it is:** The atomic units of access control that define specific actions on specific resources.

Supamode uses **two distinct permission types**:

- **System Permissions**: Control access to Supamode's administrative interface and system functions.
- **Data Permissions**: Control access to your actual application data with incredible granularity.

### System Permissions: Administrative Control

Control access to Supamode's administrative interface and system functions.

```typescript
// System Permission Example
{
  permission_type: "system",
  name: "Manage User Accounts",
  system_resource: "account",    // What system resource
  action: "update",              // What action is allowed
  description: "Can modify user account details"
}
```

**Available System Resources:**
- `account` - Managing user accounts in Supamode
- `role` - Creating and editing roles
- `permission` - Managing permission definitions
- `auth_user` - Managing auth accounts in Supabase (e.g., creating users, banning accounts, etc.)
- `table` - Database table configuration
- `log` - System audit log access

Normally, **you would grant this sort of permissions to high-level roles** like Admin or Super Admin, who need to manage the entire system.

### Data Permissions: Application Content Control

Control access to your actual application data with incredible granularity.

```typescript
// Table-Level Data Permission
{
  permission_type: "data",
  name: "Edit Blog Posts",
  scope: "table",
  schema_name: "public",
  table_name: "blog_posts",
  action: "update"
}

// Schema-Wide Data Permission
{
  permission_type: "data",
  name: "Read All Public Data",
  scope: "table",
  schema_name: "public",
  table_name: "*",              // Wildcard for all tables
  action: "select"
}
```

**Granularity Levels:**
```
Schema Level:   public.* (entire schema)
├── Table Level:  public.blog_posts (specific table)
└── Column Level: public.users.email (specific column)
```

This is the permission type you'll use most often, which grants read/write access to specific tables or columns.

For example, you might have permissions like:
- **"Read All Blog Posts"** - View all blog content for Content Management role
- **"Create Blog Posts"** - Allow content creators to add new posts
- **"Manage all data"** - Full access to all data in the application, typically reserved for Super Admins
- **"Readonly"** - Read-only access to all data in the application, typically reserved for Viewer role

## 4. Permission Groups: Logical Organization 📦

**What it is:** Bundles of related permissions that make role management
intuitive and scalable. Permission groups are fully optional, but they can greatly simplify your permission management.

```typescript
// Content Management Group
{
  name: "Content Management",
  description: "All permissions needed for content creators",
  permissions: [
    "Read All Blog Posts",
    "Create Blog Posts",
    "Edit Own Blog Posts",
    "Upload Media Files",
    "Moderate Comments"
  ]
}
```

**Strategic Grouping Examples:**
- **"Content Management"** → Blog editing, media upload, comment moderation
- **"User Administration"** → Account creation, role assignment, access reviews
- **"Customer Support"** → Ticket access, user lookup, limited account editing

**Benefits:**
- **Simplified Role Creation** - Assign entire groups instead of individual permissions
- **Consistent Access Patterns** - Related permissions stay together
- **Easier Maintenance** - Update group once, affects all roles using it

### Why Use Permission Groups?

While users can only have a single role, a role can have multiple permission groups assigned to it. This allows you to create complex permission structures without overwhelming individual roles with too many permissions.

---

# How Components Work Together 🔗

Here's how these components interact to create a complete access control system:

## The Permission Assignment Flow

```
graph TB
    A[User Login] --> B[Account Lookup]
    B --> C[Role Assignment Check]
    C --> D[Permission Resolution]
    D --> E[Access Decision]

    subgraph "Permission Sources"
        F[Direct Role Permissions]
        G[Permission Group Permissions]
    end

    D --> F
    D --> G
```

---

# Multi-Layer Security Verification: Defense in Depth 🛡️

Supamode implements three security layers to ensure comprehensive protection at every access point.

## Understanding the Security Layers

### Layer 1: JWT Token Validation (Lightning Fast) ⚡

**Purpose:** High-speed filtering of obviously invalid requests
**Performance:** Microsecond response times, no database queries

```sql
select supamode.check_admin_access();
```

This functions checks the `app_metadata` field of the JWT token to ensure
the user is an admin. We do so by checking if the `app_metadata` field
contains the `admin_access` key set to `true`.

The is a preliminary, lightweight check that quickly filters out requests
from users who are not admins.

This check is done both:
- **On the API-side** - The Hono API middleware runs this check before every request
- **On the DB-side** - The Supabase function `supamode.check_admin_access()`
gets checked on every execution of a remote procedure call (RPC).

### Layer 2: Database Permission Resolution (Comprehensive) 🗄️

**Purpose:** Deep, context-aware permission verification

```sql
-- Deep permission verification with complete context
SELECT supamode.has_permission(
  $current_user_account_id,
  $permission_id
) AS permission_granted;
```

**The Resolution Process:**
1. **Account Lookup** - Match JWT user to Supamode account
2. **Role Collection** - Gather all assigned roles
3. **Permission Aggregation** - Collect permissions from:
- Direct role assignments
- Permission group memberships
4. **Override Application** - Apply account-specific grants/denials
5. **Final Decision** - Boolean result with full audit trail

### Layer 3: Multi-Factor Authentication (optional, but recommended) 🔐

Optionally, you can enforce MFA for all users, which is recommended for all
applications.

If enabled, users must complete an additional authentication step before
signing in and being able to access the Supamode interface/data.

---

# Permission Resolution in Detail 🔍

Understanding how Supamode resolves permissions is crucial for designing effective access control.

## The Permission Resolution Algorithm

```
flowchart TD
    A[User Makes Request] --> B[Check Account-Level Denials]
    B --> C{Explicit Denial?}
    C -->|Yes| D[Access Denied]
    C -->|No| E[Check Permission Sources]

    E --> F[Direct Account Permissions]
    E --> G[Role-Based Permissions]
    E --> H[Permission Group Permissions]

    F --> I{Any Grant Found?}
    G --> I
    H --> I

    I -->|Yes| J[Access Granted]
    I -->|No| K[Access Denied]
```

## Permission Override System

Supamode supports account-specific permission overrides for exceptional cases:

```sql
-- Grant temporary admin access to a user
INSERT INTO supamode.account_permissions (
    account_id,
    permission_id,
    is_grant,
    valid_until,
    metadata
) VALUES (
    'user-account-id',
    (SELECT id FROM supamode.permissions WHERE name = 'Manage All Data'),
    true,
    '2025-12-31 23:59:59',
    '{"reason": "Temporary migration assistance"}'::jsonb
);
```

**Override Use Cases:**
- **Temporary Access** - Consultant needs admin access for 2 weeks
- **Emergency Permissions** - Support agent needs elevated access for critical issue
- **Access Revocation** - Suspend specific permissions for user under review

---

In the next steps, we will learn about:

1. **Templates**: Using the existing templates to deploy Supamode
2. **Your own schema setup**: Customizing the permission system to fit your
needs
3. **Mock Data**: Adding mock data for development purposes

