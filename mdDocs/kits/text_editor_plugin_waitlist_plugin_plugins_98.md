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

Let's extend the DB handler at `apps/web/api/db/webhook/route.ts`:

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


