-----------------
FILE PATH: kits/next-supabase-lite/payments/stripe-one-time-payments.mdoc

---
status: "published"
title: Enabling One-Time Payments with Stripe | Next.js Supabase Lite
label: One-Time Payments
order: 3
description: 'Learn how to switch from Stripe Subscriptions to One-Time Payments in your Makerkit application'
canonical: '/docs/next-supabase/stripe-one-time-payments'
---

While Makerkit uses subscriptions by default, you can offer your customers one-time payments.

Fortunately, Stripe Checkout makes it insanely easy. Let's see how to update  Makerkit's codebase to enable Stripe one-off payments for your product.

1) First, when choosing your Stripe Price's billing type, select `One Time.` [Learn more about creating Prices with Stripe](https://support.stripe.com/questions/how-to-create-products-and-prices).

2) Secondly, we need to change the payment mode in Makerkit's codebase. To do so, we will update the `createStripeCheckout` function defined at `~/lib/stripe/create-checkout.ts`.

If you open the file, you'll find the line below:

```tsx
const mode: Stripe.Checkout.SessionCreateParams.Mode = 'subscription';

// some code here...

return stripe.checkout.sessions.create({
  mode,
  // more code here...
});
```

To enable one-time payments, we will replace the `mode` constant from `subscription` to `payment`:

```tsx
const mode: Stripe.Checkout.SessionCreateParams.Mode = 'payment';

// some code here...

return stripe.checkout.sessions.create({
  mode,
  // more code here...
});
```

Now, **make sure to remove** the `subscription_data` parameter from the `stripe.checkout.sessions.create` function call. Otherwise, it will throw an error.

```tsx {7}
return stripe.checkout.sessions.create({
  mode,
  ui_mode: uiMode,
  customer,
  line_items: [lineItem],
  client_reference_id: clientReferenceId.toString(),
  subscription_data: subscriptionData,
  customer_email: params.customerEmail,
  ...urls,
});
```

And that's it! Stripe will not charge our users recurringly but will simply charge them once.

{% alert type="warning" %}
As the UI is set up for recurring subscriptions, you may want to tweak the Subscription Page UI to reflect that charges are not recurring and that the product has been successfully purchased.
{% /alert %}

## Handling the Payment Method dynamically

The above works great if you only offer one plan. However, let's assume you have multiple prices and want to dynamically update the payment mode based on the selected price.

In this case, let's do the following:
1. Extend the plan information in the application configuration
2. Select the appropriate method when creating the checkout by retrieving the plan's model

If you open your application's configuration, you will notice that plans have the following interface:

 ```tsx {% title="src/configuration.ts" %}
{
  name: 'Basic',
  description: 'Unlimited applications and 2-hour onboarding session',
  price: '$249/year',
  stripePriceId: 'price_***********',
}
```

Now, we can extend it with the payment mode, which would be `subscription` or `payment`:

```tsx
{
  name: 'Basic',
  description: 'Unlimited applications and 2-hour onboarding session',
  price: '$249 one off!',
  stripePriceId: 'price_***********',
  mode: 'payment'
}
```

Now, let's reopen the file `create-checkout.ts` file and find the payment mode of the selected price ID:

```tsx
const price = configuration.plans.find(item => {
  return item.stripePriceId === params.priceId;
});

if (!price) {
  throw new Error(`Price with ID ${params.priceId} not found in config`);
}

const mode: Stripe.Checkout.SessionCreateParams.Mode = price.mode;

// some code here...

return stripe.checkout.sessions.create({
  mode,
  // more code here...
});
```

🎉 Now it looks better and works for multiple plans based on their payment mode!


-----------------
FILE PATH: kits/next-supabase-lite/payments/stripe-webhooks.mdoc

---
status: "published"
title: How Makerkit handles Stripe Webhooks | Next.js Supabase Lite SaaS Kit
label: Stripe Webhooks
order: 1
description: 'Learn how MakerKit handles Stripe Webhooks in your Next.js Supabase Lite application'
canonical: '/docs/next-supabase/stripe-webhooks'
---


Makerkit handles five webhooks from Stripe and stores "just enough" data into the organization's entity to function correctly.

The Stripe's webhooks topics are defined here at `app/core/stripe/stripe-webhooks.enum.ts`:

```tsx
export enum StripeWebhooks {
  Completed = 'checkout.session.completed',
  SubscriptionDeleted = 'customer.subscription.deleted',
  SubscriptionUpdated = 'customer.subscription.updated',
}
```

To handle webhooks, we created an API route at `app/api/stripe/webhook.ts`.

This route is responsible for:

1. Validating that the webhook is coming from Stripe
2. Routing the webhook to its handler

## Validating Stripe's Webhooks

To validate the webhook, we do the following:
1. get the header 'stripe-signature'
2. create a Stripe instance
3. get the raw body
4. call `constructEvent` to validate and build an event object sent by Stripe. When this fails, it will throw an error

```tsx
const signature = req.headers['stripe-signature'];

// verify signature header is not missing
if (!signature) {
  return throwBadRequestException(res);
}

const rawBody = await getRawBody(req);
const stripe = await getStripeInstance();

const event = stripe.webhooks.constructEvent(
  rawBody,
  signature,
  webhookSecretKey
);
```

## Handling Stripe's Webhooks

After validating the webhook, we can now handle the webhook type relative to its topic:

```tsx
switch (event.type) {
  case StripeWebhooks.Completed: {
    // handle completed
  }

  case StripeWebhooks.AsyncPaymentSuccess: {
     // handle async payment success
  }
}
```

As described above, we handle 5 events:

1. `Completed`: the checkout session was completed (but payment may not have arrived yet if it's asynchronous)
2. `SubscriptionDeleted`: the subscription was deleted by the customer
3. `SubscriptionUpdated`: the subscription was updated by the customer

### Checkout Completed

When a user completes the checkout session, we create the `subscription` object on the currently connected user's organization.

The subscription's object has the following interface:

```tsx
export interface Subscription {
  id: string;
  priceId: string;

  status: Stripe.Subscription.Status;
  currency: string | null;
  cancelAtPeriodEnd: boolean;

  interval: string | null;
  intervalCount: number | null;

  createdAt: UnixTimestamp;
  periodStartsAt: UnixTimestamp;
  periodEndsAt: UnixTimestamp;
  trialStartsAt: UnixTimestamp | null;
  trialEndsAt: UnixTimestamp | null;
}
```

### Subscription Updated

When a subscription is updated, we rebuild the subscription object and update it in the user's Database row.

### Subscription Deleted

When a subscription is deleted, we simply remove it from the organization entity.

## Adding additional webhooks

If you want to add additional webhooks, you can do so by adding a new case in the switch statement.

For example, if you want to handle the `customer.subscription.trial_will_end` event, you can do so by adding the following case:

```tsx
case 'customer.subscription.trial_will_end': {
  // handle trial will end
}
```

## Adding additional data to the subscription object

If you want to add additional data to the subscription object, you can do so by adding the data to the `Subscription` interface.

For example, if you want to add the `quantity` property to the subscription object, you can do so by adding the following property to the interface:

```tsx
export interface OrganizationSubscription {
  // ...
  quantity: number | null;
}
```

Then you will update the `subscription` table in the Postgres database to add the `quantity` column.

Finally, you can add the data to the subscription object using the `subscriptionMapper` function:

```tsx
function subscriptionMapper(
  subscription: Stripe.Subscription
): SubscriptionRow {
  const lineItem = subscription.items.data[0];
  const price = lineItem.price;
  const priceId = price.id;
  const interval = price?.recurring?.interval ?? null;
  const intervalCount = price?.recurring?.interval_count ?? null;

  const row: Partial<SubscriptionRow> = {
    // custom props
    quantity: lineItem.quantity,

    // default props
    price_id: priceId,
    currency: subscription.currency,
    status: subscription.status ?? 'incomplete',
    interval,
    interval_count: intervalCount,
    cancel_at_period_end: subscription.cancel_at_period_end ?? false,
    created_at: subscription.created ? toISO(subscription.created) : undefined,
    period_starts_at: subscription.current_period_start
      ? toISO(subscription.current_period_start)
      : undefined,
    period_ends_at: subscription.current_period_end
      ? toISO(subscription.current_period_end)
      : undefined,
  };

  if (subscription.trial_start) {
    row.trial_starts_at = toISO(subscription.trial_start);
  }

  if (subscription.trial_end) {
    row.trial_ends_at = toISO(subscription.trial_end);
  }

  return row as SubscriptionRow;
}
```

-----------------
FILE PATH: kits/next-supabase-lite/payments.mdoc

---
status: "published"
title: "Payments in Next.js Lite Supabase"
label: "Payments"
order: 6
description: "Payments in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/plugins.mdoc

---
status: "published"
title: "Plugins in Next.js Lite Supabase"
label: "Plugins"
order: 14
description: "Plugins in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/sdk.mdoc

---
status: "published"
title: "SDK in Next.js Lite Supabase"
label: "SDK"
order: 9
description: "SDK in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/testing/ci-tests.mdoc

---
status: "published"

title: CI Tests
label: CI Tests
order: 1
description: 'Learn how to test your Makerkit in your CI'
canonical: '/docs/next-supabase/ci-tests'
---


To run your tests in any CI, we recommend you simply run the following command:

```
npm test:e2e
```

This command takes care of running all the required services, importing the test data, executing the tests and then exiting.

### The Makerkit testing pipeline

As mentioned in the previous section, testing a Makerkit requires a few services to be up and running:

- the Next.js server
- the Supabase environment

While executing these tests in your CI, this can get cumbersome. Therefore, we provided a single script to execute all your tests in one simple command: you can find the script at `scripts/test.sh`, which will contain the following content:

```
set -e

npm run supabase:start -- -x studio,migra,deno-relay,pgadmin-schema-diff,imgproxy &
docker run --add-host=host.docker.internal:host-gateway --rm -it --name=stripe -d stripe/stripe-cli:latest listen --forward-to http://host.docker.internal:3000/api/stripe/webhook --skip-verify --api-key "$STRIPE_SECRET_KEY" --log-level debug
npm run dev:test &
npm run cypress:headless
npm run supabase:stop -- --no-backup
exit 0
```

Let's explain it, so you can add your own modifications to it.

### Running the Tests in CI mode

If you were to run the tests in CI mode, you would need to run the following command:

```
npm run supabase:start
npm run dev:test &
npm run cypress
```

Then, you would need to stop the Supabase environment:

```
npm run supabase:stop
```

Since the DB gets updated, reset it to the initial state:

```
npm run supabase:reset
```

-----------------
FILE PATH: kits/next-supabase-lite/testing/running-tests.mdoc

---
status: "published"
title: Running Cypress Tests with Supabase Lite and Next.js
label: Running Tests
order: 0
description: 'Learn how to run Cypress E2E tests for your Next.js Supabase Lite application'
canonical: '/docs/next-supabase/running-tests'
---

The Makerkit SaaS boilerplate comes with a set of predefined tests with the following goals:

1. Verify that the boilerplate application works well
2. Provide instructions for building your own tests

As your application grows, the tests will inevitably need to be updated and
maintained. We hope that our tests will be simple to understand and to
change when you will need them.

## Running the Next.js development server

First, we have to run the following command to run Next.js with a testing
environment (i.e. using `.env.test`):

```
npm run dev:test
```

This command will start a Next.js dev server at http://localhost:3000, so
it's important to make sure you have no other Next.js server running at the
same port.

Additionally, we have to start the **Supabase local instance**:

```
npm run supabase:start
```

## Running Cypress in development mode

While you build your tests, it can be handy to use a windowed version of the
Cypress testing suite, so that you can easily retry the tests without having
to re-run the while tests suite.

To do that, run the following command:

```
npm run cypress
```

## Running All Cypress tests

Whenever you want to run all your tests at once, you can use the following commands:

```
npm run cypress:headless
```

This will run all the tests and exit.

## Running Cypress in CI mode

Whenever you want to run all your tests in a CI
environment, you can use the following commands:

```
npm test
```

The command above will:

1. Start the Supabase local instance
2. Start the Next.js environment
3. Run the tests in headless mode
4. Exit and close all the processes

As you may have noticed, this is the command to run in your CI environment.

## Testing Stripe

Testing Stripe requires one more process to start, i.e. the official Stripe
emulator.

The command below **will require Docker installed**, which is why you can
choose to disable it by setting the environment variable `ENABLE_STRIPE_TESTING`
to `false` from the `.env.test` environment file:

 ```bash {% title=".env.test" %}
ENABLE_STRIPE_TESTING=false
```

If instead, you want to test Stripe Checkout as well, run the command below:

```
npm run stripe:mock-server
```

As we've mentioned, this will require Docker running. Of course, we think
it's worth it.


-----------------
FILE PATH: kits/next-supabase-lite/testing.mdoc

---
status: "published"
title: "Testing in Next.js Lite Supabase"
label: "Testing"
order: 11
description: "Testing in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/tutorials.mdoc

---
status: "published"
title: "Next.js Lite Supabase (legacy) Tutorials"
label: "Tutorials"
order: 15
description: "Tutorials for the Next.js Lite Supabase (legacy) kit"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-lite/ui/shadcn.mdoc

---
status: "published"
title: Shadcn UI | Next.js Supabase Lite
label: Shadcn UI
order: 1
description: 'Learn how to add new components from Shadcn UI to your project.'
---


ShadCN UI is a template library based on Radix UI and Tailwind CSS. It provides a set of high-quality React components out of the box that are fully accessible, customizable, and themable.

Makerkit was an early adopter of ShadCN UI, and we use it to build our own UI components. With that, we used very different styling conventions early on, which means copy/pasting components from our docs to your project might not work as expected.

ShadCN UI is **configured** in all Makerkit projects as of the latest versions, while versions released before 28 September 2023 will not work seamlessly.

Makerkit only ships the components it uses in the core kit: to use other components, you will need to either use the CLI or import them manually.

## Theming

### Updating the Colors in CSS

To update the theme's colors, please update the `global.css` file in your project and update the CSS variables.

For examples, check out the [ShadCN UI Themes Page](https://ui.shadcn.com/themes).

### Updating the Colors in Tailwind

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
FILE PATH: kits/next-supabase-lite/ui/tailwindcss.mdoc

---
status: "published"
title: "Tailwind CSS | Next.js Supabase Lite SaaS Kit"
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

Additionally, please update the CSS variables of your global CSS file `globals.css` to match the new colors by updating the `--primary` color to match using the `hsl` value of the `indigo` color.

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
FILE PATH: kits/next-supabase-lite/ui/themes.mdoc

---
status: "published"
title: Themes | Next.js Supabase SaaS Kit (Lite)
label: Themes
order: 2
description: 'Learn how to configure the default theme of your Next.js Supabase SaaS Kit application.'
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
FILE PATH: kits/next-supabase-lite/ui.mdoc

---
status: "published"
title: "UI in Next.js Lite Supabase"
label: "UI"
order: 9
description: "UI in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase-turbo/admin/adding-super-admin.mdoc

---
status: "published"
label: "Adding a Super Admin"
title: "Adding a Super Admin to your Next.js Supabase application"
description: "In this post, you will learn how to set up a Super Admin in your Next.js Supabase application"
order: 0
---

The Super Admin panel allows you to manage users and accounts.

{% sequence title="Steps to add a Super Admin" description="Learn how to add a Super Admin to your Next.js Supabase application." %}

[How to access the Super Admin panel](#how-to-access-the-super-admin-panel)

[Testing the Super Admin locally](#testing-the-super-admin-locally)

[Assigning a Super Admin role to a user](#assigning-a-super-admin-role-to-a-user)

{% /sequence %}

## How to access the Super Admin panel

To access the super admin panel at `/admin`, you will need to assign a user as a super admin. In addition, you will need to enable MFA for the user from the normal user profile settings.

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

Starting from version `2.5.0`, the Super Admin user will be required to use Multi-Factor Authentication (MFA) to access the Super Admin panel.

Please navigate to the `/home/settings` page and enable MFA for the Super Admin user.

-----------------
FILE PATH: kits/next-supabase-turbo/admin.mdoc

---
status: "published"
label: "Super Admin"
title: "Adding a Super Admin to your Next.js Supabase application"
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

Browse through all accounts in your system, whether they're individual users or team accounts. 

You can:
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
FILE PATH: kits/next-supabase-turbo/analytics/analytics-and-events.mdoc

---
status: "published"
title: 'Understanding Analytics and Events in Makerkit'
label: 'Analytics and Events'
description: 'Learn how to use the Analytics and App Events systems in Makerkit to track user behavior and app-wide occurrences.'
order: 0
---

{% sequence title="How to use the Analytics and App Events systems in Makerkit" description="Learn how to use the Analytics and App Events systems in Makerkit to track user behavior and app-wide occurrences." %}

[Analytics and App Events](#analytics-and-app-events)

[Analytics Providers](#analytics-providers)

[Understanding the Relationship Between Analytics and App Events](#understanding-the-relationship-between-analytics-and-app-events)

[Recommended Approach: Centralized Analytics](#recommended-approach:-centralized-analytics)

[Using App Events with Analytics](#using-app-events-with-analytics)

[Extending with Custom Events](#extending-with-custom-events)

[Default Event Types](#default-event-types)

[When to use Custom Events](#when-to-use-custom-events)

{% /sequence %}


## Analytics and App Events
Makerkit provides two powerful, interconnected systems for tracking user behavior and app-wide occurrences: **Analytics** and **App Events**.

While these systems are separate, they are designed to work seamlessly together, providing a flexible and maintainable approach to event tracking and analysis in your SaaS application.

One doesn't need the other to function, but they are designed to work together. Here's a brief overview of each system.

## Analytics Providers

The Analytics system is implemented in the `@kit/analytics` package, and is abstracted to allow for easy integration with various analytics services, and not lock you into a specific provider.

The implementations are made possible using Makerkit Plugins, which means you can install them using the normal plugins system. 

By default, Makerkit provides three analytics providers: 
1. Google Analytics
2. Umami
3. PostHog

These are provided as plugins.

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

