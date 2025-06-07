-----------------
FILE PATH: kits/next-supabase-lite/how-to/payments/stripe-webhooks-locally.mdoc

---
status: "published"

title: "How to run the Stripe CLI to test the Stripe Webhooks locally in localhost"
label: "Running the Stripe Webhooks locally"
order: 1
description: "Want to test and run the Stripe Webhook locally? This guide will show you how to do it."
---


When testing webhooks sent by Stripe, it's useful to redirect them to our local webhook endpoint so that we can test them locally.

Makerkit has a built-in NPM script that starts the Stripe CLI and redirects webhooks to our local endpoint.

## Prerequisites

The only prerequisite is to have Docker installed and running on your machine.

## Running the Stripe Webhook locally

To run the Stripe Webhook locally, run the following command:

```bash
npm run stripe:listen
```

The first time you run this command, it will ask you to login to your Stripe account. Follow the instructions on the screen to do so.

Once you're logged in, run the command again:

```bash
npm run stripe:listen
```

This time, the Stripe CLI will start and redirect webhooks to our local endpoint.

You can now test your webhooks locally when testing your Stripe integration with Makerkit 🎉


-----------------
FILE PATH: kits/next-supabase-lite/how-to/payments/using-lemon-squeezy.mdoc

---
status: "published"
title: "Using Lemon Squeezy instead of Stripe in your Next.js Supabase Lite application"
label: Using Lemon Squeezy instead of Stripe
order: 4
description: "How to use Lemon Squeezy instead of Stripe in your Next.js Supabase Lite application"
---

To use Lemon Squeezy instead of Stripe, you should use the branch `main-ls` to develop your SaaS.

The Lemon Squeezy configuration is slightly different from the Stripe configuration as it accepts a variant ID instead of a price ID.

```tsx
subscriptions: {
  plans: [
    {
      name: 'Basic',
      description: 'Description of your Basic plan',
      badge: `Up to 20 users`,
      features: [
        'Basic Reporting',
        'Up to 20 users',
        '1GB for each user',
        'Chat Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$9',
          variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$90',
           variantID: '<your variant ID>',
        },
      ],
    },
    {
      name: 'Pro',
      badge: `Most Popular`,
      recommended: true,
      description: 'Description of your Pro plan',
      features: [
        'Advanced Reporting',
        'Up to 50 users',
        '5GB for each user',
        'Chat and Phone Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$29',
           variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$200',
          variantID: '<your variant ID>',
        },
      ],
    },
    {
      name: 'Premium',
      description: 'Description of your Premium plan',
      badge: ``,
      features: [
        'Advanced Reporting',
        'Unlimited users',
        '50GB for each user',
        'Account Manager',
      ],
      plans: [
        {
          name: '',
          price: 'Contact us',
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ]
}
```

Please refer to the [complete Lemon Squeezy documentation](/docs/next-supabase/lemon-squeezy) for more details.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/server-actions/server-actions-csrf.mdoc

---
status: "published"
title: "Sending CSRF Token to Server Actions | Next.js Supabase SaaS Kit (Lite)"
label: "Sending CSRF Token to Actions"
order: 1
description: "Learn how to send CSRF tokens to server actions in the Next.js Supabase SaaS kit (Lite)"
---

Makerkit's Middleware ensures that POST requests to your server actions are protected against CSRF attacks. This means that you need to send a CSRF token with every POST request to your server actions. The only instance where you need to manually protect your Actions is when you use forms that send data using `FormData`.

## Getting the CSRF Token

To retrieve the CSRF token to send along with your requests, use the `useCsrfToken` hook. This hook returns the CSRF token as a string.

```tsx
import useCsrfToken from '~/core/hooks/use-csrf-token';

// somehwere in your component
const token = useCsrfToken();
```

## Sending the CSRF Token

When you call a Server Action, simply add the token as `csrfToken` in the parameters object.

For example, assuming this is our Server Action:

```tsx
type CreateTaskParams = {
  task: Omit<Task, 'id'>;
  csrfToken: string;
};

export const createTaskAction = withSession(
  async (params: CreateTaskParams) => {
    const client = getSupabaseServerActionClient();
    const path = `/tasks`;

    await createTask(client, params.task);

    // revalidate the tasks page
    revalidatePath(path, 'page');

    // redirect to the tasks page
    redirect(path);
  },
);
```

NB: we added `csrfToken` to the `CreateTaskParams` type.

Then, we can call the Server Action like this:

```tsx
import { useTransition } from "react";
import useCsrfToken from '~/core/hooks/use-csrf-token';

function TaskForm() {
  const csrfToken = useCsrfToken();
  const [isMutating, startTransition] = useTransition();

  const onSubmit = (task: Task) => {
    startTransition(async () => {
      await createTaskAction({ task, csrfToken });
    });
  };
}
```

As you can see, we're passing an object with two properties: `task` and `csrfToken`. The `csrfToken` property is the one that will be used to send the CSRF token to the server.

## Using "FormData"

When using plain forms, we can pass the CSRF token as an input field.

```tsx
function TaskForm() {
  const csrfToken = useCsrfToken();

  return (
    <form action={createTaskAction}>
      <input type={'hidden'} name={'csrfToken'} value={csrfToken} />
    </form>
  );
}
```

In your server action, you will manually need to extract the CSRF token from the request body and verify it using the `verifyCsrfToken` function.

```tsx
import verifyCsrfToken from '~/core/verify-csrf-token';

export const createTaskAction = async (data: FormData) => {
  const csrfToken = data.get('csrfToken');

  await verifyCsrfToken(csrfToken);

  // ...
};
```

-----------------
FILE PATH: kits/next-supabase-lite/how-to/server-actions/server-actions-error-handling.mdoc

---
status: "published"
title: "How to handle errors in Server Actions | Next.js Supabase SaaS Kit (Lite)"
label: "Server Actions Error Handling"
order: 2
description: "Learn how to handle errors in Server Actions in the Next.js Supabase SaaS Kit (Lite)"
---

Generally speaking, you can handle Server Action in three ways:

1. You can use the `try/catch` block to catch errors and handle them in the `catch` block.
2. You can use a React `ErrorBoundary` component to catch errors
3. You can also use the special `error.tsx` file to handle errors at layout level.

### Handling errors from a Server Action invoked by a Form

Let's assume we have a Form using a server action that throws an error:

```tsx title='Form.tsx'
async function serverActionWithError() {
  'use server';

  throw new Error(`This is error is in the Server Action`);
}

function FormWithServerAction() {
  return (
    <form action={serverActionWithError}>
      <button>Submit Form</button>
    </form>
  );
}

export default FormWithServerAction;
```

We can wrap our form using the `ErrorBoundary` component we have at `~/core/ui/ErrorBoundary`, and handle the error from there:

```tsx title='FormWithErrorBoundary.tsx'
import ErrorBoundary from '~/core/ui/ErrorBoundary';
import FormWithServerAction from './Form';

function FormWithErrorBoundary() {
  return (
    <ErrorBoundary fallback={<p>Something went wrong</p>}>
      <FormWithServerAction />
    </ErrorBoundary>
  );
}
```

If you click on the button, you'll see the error message rendered.

### Handling errors from an imperatively invoked Server Action

If you are invoking a Server Action imperatively, we can wrap the `useTransition` hook in a `try/catch` block, and handle the error from there.

Let's assume we have a Server Action that throws an error:

```tsx title='actions.tsx'
'use server';

export async function serverActionWithError() {
  throw new Error(`This is error is in the Server Action`);
}
```

We can invoke it imperatively from a client component, and handle the error from there:

```tsx title='ImperativeServerAction.tsx'
'use client';

import { useTransition } from 'react';
import { serverActionWithError } from './actions';

function ImperativeServerAction() {
  const [pending, startTransition] = useTransition();

  return (
    <button
      disabled={pending}
      onClick={() => {
        startTransition(async () => {
          try {
            await serverActionWithError()
          } catch (e) {
            alert('error');
          }
        });
      }}
    >
      Click Button
    </button>
  );
}

export default ImperativeServerAction;
```

If you click on the button, you'll see the `alert` popup with the error message.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/server-actions/server-actions.mdoc

---
status: "published"

title: "Server Actions"
label: "Server Actions"
order: 0
description: "Learn how to write a server action in your Next.js Supabase SaaS application"
---


Makerkit makes extensive use of Next.js Server Actions to mutate data in your Supabase database.

Server Actions are a powerful feature of Next.js that allow you to write server-side code that we can call from the client-side. This is useful for a number of reasons:
1. We don't need to write any API routes to handle our server-side logic
2. We can tell Next.js to revalidate our data after a server action has been called, thus refreshing our data on the client-side
3. We can easily call them just like any other function in our code

{% alert%}
This page is important to explain how we can handle Supabase mutations in the next sections.
{% /alert %}

## Creating a Server Action

To create a server action, we need to create a separate file marked with the `use server` directive at the top of the file. This tells Next.js that this file contains a server action that we can call from the client-side.

```tsx
'use server';

export async function handleFormSubmit(
  formData: FormData
) {
  // Do something with the form data
}
```

## Calling a Server Action

Calling a server action can be done in several ways

1. Calling it from a form submit handler
2. Calling it from a button click handler
3. Calling it imperatively as a function

### Calling it from a form submit handler

Let's assume we defined our server actions in a file called `actions.ts`:

```tsx
'use server';

export async function createPostAction(
  formData: FormData
) {
  // Do something with the form data
}
```

We can call this server action from a form submit handler like so:

```tsx
<form action={createPostAction}>
  <div className='flex flex-col space-y-4'>
    <h2 className='text-lg font-semibold'>Create a new Post</h2>

    <Label className='flex flex-col space-y-1.5'>
      <span>Title</span>
      <Input name='title' placeholder='Ex. The best Next.js libraries' required />
    </Label>

    <Label className='flex flex-col space-y-1.5'>
      <span>Description</span>
      <Input />
    </Label>

    <SubmitButton />
  </div>
</form>
```

### Calling it from a button click handler

We can also call a server action from a button click handler:

```tsx
<form action={createPostAction}>
  <div className='flex flex-col space-y-4'>
    <h2 className='text-lg font-semibold'>Create a new Post</h2>

    <Label className='flex flex-col space-y-1.5'>
      <span>Title</span>
      <Input name='title' placeholder='Ex. The best Next.js libraries' required />
    </Label>

    <Label className='flex flex-col space-y-1.5'>
      <span>Description</span>
      <Input />
    </Label>

    <button formAction={createPostAction}>Click</button>
  </div>
</form>
```

### Calling it imperatively as a function

Finally, we can call a server action imperatively as a function using `startTransition`:

```tsx
import { useTransition } from "react";

function ClientComponent({ id }) {
  let [isPending, startTransition] = useTransition();

  return (
    <button onClick={() => startTransition(() => createPostAction(id))}>
      Save
    </button>
  );
}
```

## Send CSRF token with server actions

Remember: Makerkit protects by default every POST request with a CSRF token. This is a security measure to prevent CSRF attacks.

It's your job to send this along when you call a server action.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/branding.mdoc

---
status: "published"

title: "Update your Makerkit application Branding"
label: "Branding"
order: 1
description: "A unique branding is essential to any SaaS. Learn how to update your Makerkit application branding."
---


Most of the branding details of your SaaS will be stored in the main configuration file of your application. This file is located at `src/configurations.ts`.

The `site` object contains most of your branding details. You can update the following properties:

```tsx
site: {
  name: 'Awesomely - Your SaaS Title',
  description: 'Your SaaS Description',
  themeColor: '#ffffff',
  themeColorDark: '#0a0a0a',
  siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
  siteName: 'Awesomely',
  twitterHandle: '',
  githubHandle: '',
  language: 'en',
  convertKitFormId: '',
  locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
},
```

1. The `name` and `description` properties will be used in the meta tags of your application
2. The `themeColor` and `themeColorDark` properties will be used to update the color of the browser bar on mobile devices.
3. The `siteName` property is a short version of your SaaS title, without the subtitle.
4. The `twitterHandle` and `githubHandle` properties will be used to link to your social media accounts (if you have any).
5. The `locale` property is the default language of your application
6. The `language` property is deprecated and will be removed in a future version of Makerkit. Please use the `locale` property instead.
7. The `convertKitFormId` property is used to link your application to your ConvertKit account if you use the Newsletter form.

## Design Branding

Please refer to the [Theming](theming) section to learn how to update the design of your application.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/environment-variables-setup.mdoc

---
status: "published"
title: "Understanding and setting the environment variables in your Next.js Supabase Lite project"
label: "Environment variables"
order: 5
description: "Learn how to set environment variables in your Next.js project and how to use them in your code."
---

Makerkit ships with some environment variables pre-configured for you.

If you don't have experience with environment variables, you can think of them as a way to store configuration values for your project.

Here are the types of environment variables you can use in your Next.js project:
1. **.env**: This file is used to store environment variables that are common to all environments (development, staging, production). **Only use it for public and shared variables**.
2. **.env.local**: This file is used to store environment variables that are specific to your local development environment. **Do not commit this file to your repository** (this is already ignored by the kit's defaults).
3. **.env.development**: This file is used to store environment variables that are specific to your development environment. As long as you don't use it for sensitive data, you can commit this file to your repository.
4. **.env.production**: This file is used to store environment variables that are specific to your production environment. Do not add sensitive data to this file. Instead, use your CI provider.

## If I set mt variables in Vercel/another hosting provider, do I still need to set them in my project?

No. If you set your environment variables in your hosting provider, you don't need to set them in your project.

## Can I add secrets to my environment variables, like my OpenAI API key?

**No.** You should never add secrets to your environment variables. Instead, you should use your CI provider to set them.

## Does Next.js automatically load my environment variables?

Yes, Next.js automatically loads your environment variables. You can access them in your code using `process.env.VARIABLE_NAME`.

## I added a variable to my .env file, but I can't access it in my code. What's wrong?

Next.js only loads environment variables that start with `NEXT_PUBLIC_` if you are trying to access them from your client-side. If you want to access a variable in your client code, you need to prefix it with `NEXT_PUBLIC_`. **Note**: never add secrets to your client-side code, therefore do not prefix them with `NEXT_PUBLIC_` if it's private data.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/project-configuration.mdoc

---
status: "published"

title: "How to edit your Makerkit project configuration"
label: Project Configuration
order: 0
description: "Learn how and where to edit your project configuration in your Makerkit SaaS application."
---


Makerkit stores your project configuration in a Typescript file named `configuration.ts`, in which we store **non-secret** data needed in the client-side bundles of your project.

## What is stored in the configuration file?

In this configuration file, we store data that is safe to be exposed in the client-side bundles of your project.

The configuration file contains the following data:

```tsx
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

const production = process.env.NODE_ENV === 'production';

enum Themes {
  Light = 'light',
  Dark = 'dark',
}

const configuration = {
  site: {
    name: 'Awesomely - Your SaaS Title',
    description: 'Your SaaS Description',
    themeColor: '#ffffff',
    themeColorDark: '#0a0a0a',
    siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
  },
  auth: {
    // ensure this is the same as your Supabase project. By default - it's true
    requireEmailConfirmation:
      process.env.NEXT_PUBLIC_REQUIRE_EMAIL_CONFIRMATION === 'true',
    // NB: Enable the providers below in the Supabase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: ['google'] as Provider[],
    },
  },
  production,
  environment: process.env.NEXT_PUBLIC_ENVIRONMENT,
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    signInMfa: '/auth/verify',
    onboarding: `/onboarding`,
    appPrefix: '/dashboard',
    appHome: '/dashboard',
    authCallback: '/auth/callback',
    settings: {
      profile: 'settings/profile',
      authentication: 'settings/profile/authentication',
      email: 'settings/profile/email',
      password: 'settings/profile/password',
    },
  },
  sentry: {
    dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  },
  stripe: {
    products: [
      {
        name: 'Basic',
        description: 'Description of your Basic plan',
        badge: `Up to 20 users`,
        features: [
          'Basic Reporting',
          'Up to 20 users',
          '1GB for each user',
          'Chat Support',
        ],
        plans: [
          {
            name: 'Monthly',
            price: '$9',
            stripePriceId: '<price_id>',
          },
          {
            name: 'Yearly',
            price: '$90',
            stripePriceId: '<price_id>',
          },
        ],
      },
      {
        name: 'Pro',
        badge: `Most Popular`,
        recommended: true,
        description: 'Description of your Pro plan',
        features: [
          'Advanced Reporting',
          'Up to 50 users',
          '5GB for each user',
          'Chat and Phone Support',
        ],
        plans: [
          {
            name: 'Monthly',
            price: '$29',
            stripePriceId: '<price_id>',
          },
          {
            name: 'Yearly',
            price: '$200',
            stripePriceId: '<price_id>',
          },
        ],
      },
      {
        name: 'Premium',
        description: 'Description of your Premium plan',
        badge: ``,
        features: [
          'Advanced Reporting',
          'Unlimited users',
          '50GB for each user',
          'Account Manager',
        ],
        plans: [
          {
            name: '',
            price: 'Contact us',
            stripePriceId: '',
            label: `Contact us`,
            href: `/contact`,
          },
        ],
      },
    ],
  },
};

export default configuration;
```

## How should I store my secrets?

You should store your secrets in your **non public** environment variables, and ideally set up from your CI or Hosting Provider (such as Vercel).

Never add secrets to the configuration file, as it is exposed in the client-side bundles of your project.

Please refer to the Next.js official documentation on how to best store your secrets.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/set-up-emails.mdoc

---
status: "published"
title: "How to setup emails in your Makerkit SaaS application | Next.js Supabase Lite SaaS Kit"
label: Setting up Emails
order: 6
description: "How to setup emails in your Makerkit SaaS application"
---

Sending emails is a critical part of your SaaS application. Makerkit uses emails to send invites to your users only, but it's likely you'll want to send other emails to your users.

To set up emails in your Makerkit application, you should add the following environment variables to your project, using a secure environment:

```bash
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

Makerkit will use these values to send emails on your behalf using the node library `nodemailer`.

NB: this does not refer to emails sent by Supabase. These have to be setup
from within the Supabase Dashboard. You can reuse the same SMTP settings.

### Where do I get these values?

These values are normally provided by your service provider.

### Which services are supported?

Any service that supports SMTP should work. We recommend using [Resend](https://resend.com/).

### How do I test emails locally?

Makerkit uses [InBucket](https://inbucket.org) - a platform to testing emails.

InBucket saves time during development since we can test
our emails without setting up a real SMTP service - and works locally and
offline.

InBucket will run automatically when starting the local Supabase environment.

InBucket will now be running at [localhost:54324](http://localhost:54324). When we send an email using the
`sendEmail` function in the kit, we can visualize it using the InBucket UI.

InBucket is used by default during development. Instead, for production
usage, you will need to set up a real SMTP service.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/updating-favicon.mdoc

---
status: "published"
title: "Updating the favicons of your Makerkit site | Next.js Supabase Lite"
label: Updating the Favicons
order: 4
description: "Learn how to update the favicons of your Makerkit Next.js Supabase Lite landing pages"
---

If you have a logo for your company, you can easily update the favicons of your Makerkit applications. Favicons are the small icons that appear in the browser tab of your website.

### Where to find the favicons

Favicons are stored at `public/assets/images/favicon`. You can replace the existing files with your own images. If you keep the same names and sizes, then you don't need to update the code.

### How to generate favicons

If you do, I suggest to generate them using any o the online favicon generator, and then dropping the resulting images in the `public/assets/images/favicon` folder.

If you have different sizes and names, then you need to update the code to reflect these changes.

## How to update the favicons in Makerkit

This code is stored in the root layout:

 ```tsx {% title="app/layout.tsx" %}
icons: {
  icon: '/assets/images/favicon/favicon.ico',
  shortcut: '/shortcut-icon.png',
  apple: '/assets/images/favicon/apple-touch-icon.png',
  other: {
    rel: 'apple-touch-icon-precomposed',
    url: '/apple-touch-icon-precomposed.png'
  },
}
```

Update the paths to match your new favicons.


-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/updating-fonts.mdoc

---
status: "published"

title: "Updating the fonts of your Makerkit site"
label: Updating the Fonts
order: 3
description: "Learn how to update the fonts of your Makerkit landing pages"
---


By default, Makerkit defines the site's fonts using the package `next/font/google`.

The fonts are defined at `src/components/Fonts.tsx` and you can update them to your liking.

### What fonts are used by default?

By default, **Makerkit uses the system Apple font on Apple devices**, the `Inter` font as fallback.

### Removing Apple font as default

If you want to remove the `Apple` font as default, simply update the `FontsFamily` component at `src/components/Fonts.tsx` to remove the `Apple` font.

 ```tsx {% title="src/components/Fonts.tsx" %} {26-27}
'use client';

import { Inter as SansFont } from 'next/font/google';
import { useServerInsertedHTML } from 'next/navigation';

const sans = SansFont({
  subsets: ['latin'],
  variable: '--font-family-sans',
  fallback: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
  preload: true,
  weight: ['300', '400', '500', '600', '700', '800'],
});

// replace with your heading font
// by default, it will use the sans font
const heading = sans;

function Fonts() {
  useServerInsertedHTML(() => {
    return (
      <style
        key={'fonts'}
        dangerouslySetInnerHTML={{
          __html: `
          :root {
            --font-family-sans: ${sans.style.fontFamily}, 'Segoe UI', 'Roboto', 'Ubuntu', 'sans-serif';
            --font-family-heading: ${heading.style.fontFamily};
          }
        `,
        }}
      />
    );
  });

  return null;
}

export default Fonts;
```

If you compare the above with the version of the file you have in your project, you'll notice that we removed the `--apple-system` fonts.

### Using a different font

If you want to use a different font, you can import it from the `next/font/google` package.

For example, below, we import the font `Manrope` from Google Fonts. It replaces `Inter` as the default font.

```tsx
import { Manrope as SansFont } from 'next/font/google';
```

You can use two different fonts for the `--font-family-sans` and `--font-family-heading` variables. One is used an the main body font, the other as the heading font.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/setup/updating-logo.mdoc

---
status: "published"

title: "Updating the logo of your Makerkit site"
label: Updating the Logo
order: 2
description: "Learn how to update the logos of your Makerkit site"
---


Makerkit by default provides an SVG logo with the default Makerkit logo. In addition, we also provide a `mini` version of the logo that is used when the sidebar is collapsed.

You should change both the `LogoImage` and `LogoImageMini` to match your brand.

### Where are the logos defined?

The logos are defined in two files:

1. `src/core/ui/Logo/LogoImage.tsx`
2. `src/core/ui/Logo/LogoImageMini.tsx`

Feel free to replace the default logo with your own logo.

### What sizes should my logo be?

The optimal sizes for the logs are:

1. The logos should be 20px tall to fit well in the header.
2. The logo mini should be 20px tall and 20px wide.

### I like Makerkit's logo! How can I reuse it for my own logo?

[Download the Makerkit Figma file from here](/assets/makerkit.fig) and use the logo from there.

-----------------
FILE PATH: kits/next-supabase-lite/how-to/site/adding-marketing-pages.mdoc

---
status: "published"
title: "Adding pages to the marketing site of your Makerkit Next.js Lite Supabase project"
label: Adding Pages
order: 3
description: "Learn how to add new pages to the marketing site of your Makerkit Next.js Supabase Lite project."
---

To add a new page to the marketing site of your Next.js project, you can create a new page in the `src/app/(site)` directory.

By "marketing site", we mean the public pages of your project, such as the homepage, the about page, the contact page, etc.

{% alert type="warning" %}
      Use the below code only for public pages accessible by everyone, such as the homepage, the about page, the contact
      page, etc.

{% /alert %}

### Adding a new page to the marketing site

As you know already, we can add pages to our Next.js App Router application by creating files within the `src/app` directory named as `page.tsx`.

For example, if you want to add a page at `/about`, you can create a new file at `src/app/(site)/about/page.tsx` with the following content:

 ```tsx {% title="src/app/(site)/about/page.tsx" %}
function AboutPage() {
  return <div>About page</div>;
}

export default AboutPage;
```

This page will **automatically inherit the site layout** from `src/app/(site)/layout.tsx` - which means it will already display the header and footer.

#### I don't want to use the same layout though?

If you don't want to use the site layout, create a route group inside `src/app`, and add your page there.

### Adding a new page to the marketing site with translations

If you have translations on your marketing site - to ensure the page is translated correctly server-side, we use the `withI18n` HOC:

 ```tsx {% title="src/app/(site)/about/page.tsx" %}
import { withI18n } from '~/i18n/with-i18n';

function AboutPage() {
  return <div>About page</div>;
}

export default withI18n(AboutPage);
```


