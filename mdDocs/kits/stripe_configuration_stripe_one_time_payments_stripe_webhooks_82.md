-----------------
FILE PATH: kits\remix-supabase\payments\stripe-configuration.mdoc

---
status: "published"

title: Set up Stripe Payments for Remix and Supabase SaaS applications
label: Stripe Configuration
order: 0
description: 'Learn how to configure Stripe with your MakerKit application'
---


MakerKit offers support for subscriptions and payments using [Stripe](https://stripe.com).

To get started with [Stripe Checkout](https://stripe.com/docs/payments/checkout), you will need to create an account using the [Stripe website](https://stripe.com).

If you want, you can get your business details sorted right away. If you
want to continue with the set-up, you can skip it for now and get back to it later.

## Configuration

Visit the [Stripe Checkout Tutorial](https://stripe.com/docs/checkout/quickstart?client=next) where we can copy the
configuration for our development environment.

If you click on the file `.env` of the code editor on the right-hand side,
you will be able to see the variables needed:

- `STRIPE_SECRET_KEY`
- `STRIPE_WEBHOOK_SECRET`

Copy these values to your local `.env` file.

{% alert type="warning" %}
Make sure to keep your <code>.env</code> file private. It should not be committed to your repository. These environment variables should be added using your hosting provider's dashboard.
{% /alert %}

## Creating your SaaS plans in Stripe

Stripe allows us to create our plans using the UI in the Stripe Dashboard.

To get started, let's add a couple of testing plans that you can use while
developing your application.

Visit [the Stripe Dashboard](https://dashboard.stripe.com/test/products/prod_KpXDAnvbPkXkgp) and add a new product (for example, a one-time purchase or a SaaS subscription).

You can add multiple "prices": for example, if you're defining a
subscription, you could use two plans, "Basic" and "Pro".

## Creating a Webhook endpoint

Stripe allows us to create a webhook endpoint to receive events. To tell Stripe
where to send the events, we need to create a webhook endpoint that points to our application.

After creating a Stripe endpoint, we need to add the "Signing Secret" key in Stripe to the production environment variables. To do so, use the environment variables editor in your hosting provider's dashboard.

{% img src="/assets/images/docs/stripe-webhook-config.webp" width="1670" height="368" /%}

### Webhook API Endpoint

The API endpoint needs to point to your application's Stripe webhook: the path is `/resources/stripe/webhook`. For example, if your application is hosted at `https://myapp.com`, the webhook endpoint would be `https://myapp.com/resources/stripe/webhook`.

### Events

The events you want to receive from Stripe are:
- `checkout.session.completed`
- `customer.subscription.updated`
- `customer.subscription.deleted`

If you have any other events you want to receive, you need to add them here.

### Signing Secret

The signing secret is the value of the `STRIPE_WEBHOOK_SECRET` environment variable. 

{% alert type="warning" %}
If you do not follow these steps, you will not be able to receive events from Stripe. This will prevent you from being able to update the subscription status of your users. Please make sure to follow these steps.
{% /alert %}

Before you continue... **please make sure to follow these steps**:

1. Create a webhook endpoint in Stripe
2. Make sure to add the **events** you want to receive from Stripe
3. Copy the **signing secret** to your production environment variables as `STRIPE_WEBHOOK_SECRET`
4. Redeploy your application to apply the changes

### Pricing Table Configuration

Below is the default Pricing Table configuration of the kits. The `PricingTable` component will automatically generate the pricing table based on the Stripe plans you have created and added to the configuration (see below).

We have 3 products (Basic, Pro and Premium), each with 2 plans (monthly, yearly). By default, the Pro plan is set as the recommended plan.

Of course, the below is simply an example. You will need to customize this according to your application's plans.

```tsx
stripe: {
  embedded: true,
  displayMode: StripeCheckoutDisplayMode.Popup,
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
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$90',
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
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
          stripePriceId: 'pro-plan-mth',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$200',
          stripePriceId: 'pro-plan-yr',
          trialPeriodDays: 7,
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
          trialPeriodDays: 7,
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ],
}
```

{% alert type="warning" %}
The properties "stripePriceId" are placeholders. You will need to replace them with the IDs of your plans.
{% /alert %}

### Embedded Display Mode

You can choose between two layouts for displaying the embedded checkout:

1. **Popup [default]** - the default layout. The checkout will be displayed as a popup.
2. **Overlay** - the checkout will be displayed as a full page overlay.

### Hosted Stripe Checkout

If you prefer to redirect users to Stripe Checkout, you can set `embedded` to `false`. This is the default configuration in versions before 0.14.0 as embedded checkouts were not available (released in October 2023).


### Additional Configuration

You can also configure the following properties on each `product`:

- `recommended` - set to `true` if you want to highlight a plan as recommended
- `badge` - set to a string if you want to display a badge next to the plan name

You can also configure the following properties on each `plan`:

- `price` - any string that you want to display as the price (e.g. `$9.99` or `Free`)
- `label` - set to a custom string if you want to display a button label
- `href` - set to a custom URL if you want to link to a custom page (for example, a contact page)
- `trialPeriodDays` - set to the number of days you want to offer a free trial for

### Install The Stripe CLI

The Stripe CLI is required to help us test our integration.
Please [follow this guide to install the CLI on your system](https://stripe.com/docs/stripe-cli).

The default recommendation is to use Docker, which Makerkit uses by default.

### Connect CLI with your account

To connect the Stripe CLI with your account, run the following command:

```
npm run stripe:listen
```

This command requires Docker, but you can alternatively install Stripe on your OS and change the command to use `stripe` directly.

The CLI should prompt you to connect your account. If you signed in already, visit the link and follow the instructions.

If you followed the instructions, get back to the terminal, and then copy
the webhook secret to your `STRIPE_WEBHOOK_SECRET` environment variables in the
`.env` file.

Restart the Remix server to apply the changes.

## Billing Portal

MakerKit uses the [Stripe Billing Portal](https://dashboard.stripe.com/test/settings/billing/portal) to let your users manage their invoices and subscriptions.

The portal becomes accessible once the organization subscribes to a plan, which creates a `customerId` property, making it possible for the organization to access the portal.

### Enabling the Billing Portal in test mode

Please visit the [Stripe Billing Page](https://dashboard.stripe.com/test/settings/billing/portal) to enable the Billing Portal.

Choose the settings that best apply to your product and save your changes.

The only required fields to get started are `Terms of Service` and
`Privacy` URLs. You can add any URL at this stage. **However, remember to update these pages when you go live**.

-----------------
FILE PATH: kits\remix-supabase\payments\stripe-one-time-payments.mdoc

---
status: "published"
title: Enabling One-Time Payments with Stripe | Remix Supabase Kit
label: One-Time Payments
order: 3
description: 'Learn how to switch from Stripe Subscriptions to One-Time Payments in your Makerkit application'
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
FILE PATH: kits\remix-supabase\payments\stripe-webhooks.mdoc

---
status: "published"
title: How Makerkit handles Stripe Webhooks in your Remix Supabase application
label: Stripe Webhooks
order: 1
description: 'Learn how MakerKit handles Stripe Webhooks in your Remix Supabase application'
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

To handle webhooks, we created an API route at `routes/resources/stripe/webhook.ts`.

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
export interface OrganizationSubscription {
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

This interface is added to the `Organization` object.

### Subscription Updated

When a subscription is updated, we rebuild the subscription object and update it in the organization's Database row.

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

If you want to add additional data to the subscription object, you can do so by adding the data to the `OrganizationSubscription` interface.

For example, if you want to add the `quantity` property to the subscription object, you can do so by adding the following property to the interface:

```tsx
export interface OrganizationSubscription {
  // ...
  quantity: number | null;
}
```

Then you will update the `subscription` table in the PostgreSQL database to add the `quantity` column.

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
FILE PATH: kits\remix-supabase\payments.mdoc

---
status: "published"
title: "Payments in Remix Supabase"
label: "Payments"
order: 6
description: "Payments in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\remix-supabase\plugins\ai-chatbot.mdoc

---
status: "published"

title: "AI Chatbot Plugin for the Remix Supabase Saas Starter Kit"
label: "AI Chatbot"
order: 2
description: "This plugin adds an AI chatbot to your Remix Supabase SaaS application."
---


This plugin adds an AI chatbot trained on your website content to your Remix
application built using the OpenAI API. It's a simple component that will be
displayed on the bottom right of your website.

This documentation is for the Remix Supabase version. Please make sure you're looking at the right documentation.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/chatbot git@github.com:makerkit/remix-supabase-saas-kit-plugins.git chatbot --squash
```

After running this command, you will have the plugin in your repository at
`plugins/chatbot`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```
npx @makerkit/cli plugins install
```

Follow the instructions to install the plugin.

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:

1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/chatbot https://github.com/makerkit/remix-supabase-saas-kit-plugins chatbot --squash
```

#### Add the plugin as a workspace in your package.json

You can do so by adding the following to your `package.json` file:

```json
{
  "workspaces": [
    "plugins/chatbot"
  ]
}
```

Add it next to the other workspaces in your `package.json` file.

#### Add the paths alias to the TypeScript configuration

To make sure that the TypeScript compiler can find the plugin, you will need
to add the following paths alias to your `tsconfig.json` file, in addition
to the other paths aliases that you may have.

If not yet present, add the following to your `tsconfig.json` file:

```json
{
  "compilerOptions": {
    "paths": {
      "~/plugins/*": [
        "./plugins/*"
      ]
    }
  }
}
```

You only need to add this once, even if you have multiple plugins.

#### Add the plugin path to your Tailwind configuration

You can do so by adding the following to your `tailwind.config.js` file:

```js
content: ['./app/**/*.{ts,tsx,jsx,js}', './plugins/**/*.{ts,tsx,jsx,js}']
```

This is only needed the first time you install a plugin.

#### Add Langchain to the Remix config

To work with Remix, Langchain needs to be bundled with the server code.

You can do so by adding the following to your `remix.config.ts` file:

```ts
serverDependenciesToBundle: [
  /^(@)?contentlayer.*/,
  /@mdx-js.*/,
  /langchain.*/,
]
```

### Installing dependencies

To install the dependencies, you can run the following command:

```
npm i
```

NPM will install the dependencies in the `plugins/chatbot` folder as an
NPM workspace.

### Configuring the database migration

This plugin needs to store the documents that the AI will be trained on as
vectors in the Supabase database. As such, we need to add a migrration to the
database.

The migration SQL file is available at `plugins/chatbot/migration.sql`. You
can generate a new mutation using the following command:

```
supabase migration new chatbot
```

Make sure you're happy with the default table names before proceeding so
that they don't conflict with your existing tables.

Now paste the content of the `plugins/chatbot/migration.sql` file into the
new migration file that was created.

When going to production, you will need to run the migration using the
Supabase CLI:

```
supabase db push
```

### Importing the Plugin

You can import the `Chatbot` component from the plugin in your `root.tsx`
file if you want it available on all pages:

```tsx
import ChatBot from '~/plugins/chatbot/components/ChatBot';

export default function RootLayout() {
  return (
    <>
      <ChatBot />
      {/* ... */}
    </>
  );
}
```

Alternatively, you can import the `Chatbot` component in the layout of the page
where you want it to appear. If you only want it available in the website
pages, then you will import it in the site layout.

### Configuring the plugin

To configure the plugin, add the following environment variables to your
`.env.local` files:

```
OPENAI_API_KEY=
```

- `OPENAI_API_KEY` - Your OpenAI API key. You can get one from your OpenAI
dashboard.

When going to production, you will need to add these environment variables
from your provider dashboard.

---


Additionally, we need to pass the fallback URL to the chatbot:

- This is the link the chatbot will display if it can't answer the question. You
can use an email such as `mailto:youremail@yourstartup.com` or a link to
the customer service page of your website.

```tsx
<Chatbot fallbackUrl="mailto:yourapp.com" />
```

### Setting the API Route Handler

#### Remix Route

Create a route at `app/api.chat.ts` with the following content:

```ts
import type { ActionArgs } from '@remix-run/node';
import handleChatBotRequest from '~/plugins/chatbot/lib/server/route-action.server';

export const action = (args: ActionArgs) => {
  return handleChatBotRequest(args.request);
};
```

### Indexing the content's embeddings

By default, the AI will be trained on the content of your website available
within your documentation pages at `/docs`.

In addition, you can provide a list of questions and answers.

To do so, add MDX files at `plugins/chatbot/questions/<filename>.mdx` file
and add content with the following format:

```mdx
---

question: "<question>"
---


<answer>
```

For example, you can create a file at
`plugins/chatbot/questions/refund-policy.mdx`:

```mdx
---

question: "What is your refund policy?"
---


We offer a 30-day money-back guarantee. If you're not happy with our product,
we will refund you.
```

To generate the embeddings, you can run the following command:

```
npx tsx plugins/chatbot/cli.ts generate
```

Follow the instructions to generate the embeddings.

If you are generating the embeddings for your production environment, you
will need to provide the Supabase credentials in the `.env` file.

### Captcha Protection (optional) (WIP)

To protect your chatbot from bots (ha ha!), you can add a captcha challenge
using [Cloudflare's Turnstile](https://developers.cloudflare.com/turnstile/)
service.

To do so, you will need to pass the site key to the Chatbot component:

```tsx
<Chatbot siteKey="************8" />
```

[You can get the site key from your Turnstile dashboard](https://developers.
cloudflare.com/turnstile/get-started/).

On the server side, you will need to add the following environment
variables: `CHATBOT_TURNSTILE_SECRET_KEY`.

You can also get the secret key from your Turnstile dashboard. Since this is
a secret key - avoid adding it to your repository. **Please use the
environment variables of your provider** to add it to your
production environment.

### Deduplicating Indexing

To avoid indexing the same content multiple times, the CLI generates a file
named `plugins/chatbot/indexed-files.json` that contains a list of all the
pages that have been indexed and the SHA256 hash of their content.

If the content of the file changes, the CLI will re-index the pages.

### Keeping the plugin up to date

To keep the plugin up to date, you can use git subtrees again:

```
git subtree pull --prefix plugins/chatbot git@github.com:makerkit/remix-supabase-saas-kit-plugins.git chatbot --squash
```

## Best Practices

As you may know - the AI is only as good as the data it's trained on. As
such, make sure to provide as much content as possible to the AI. I don't
mean dozens of pages - but hundreds. Seriously, the more content you provide
- the more useful the AI will be. Without context, hallucinations are bound
to happen - and you don't want that.

### Provide a fallback email

If the AI can't answer the question, it will display a fallback URL. Make
sure to provide a fallback URL that will allow your users to contact you in
case the AI can't answer their question.

-----------------
FILE PATH: kits\remix-supabase\plugins\ai-text-editor.mdoc

---
status: "published"

title: "AI Text Editor Plugin for the Remix Supabase Saas Starter Kit"
label: "AI Text Editor"
order: 3
description: "This plugin adds an AI WYSIWYG Editor to your Remix Supabase SaaS application."
---


This plugin adds an AI Editor component Remix application. You can
import this component anywhere in your application.

It is built with Lexical, a text editor framework for React, by Meta.

**This plugin is currently experimental.**

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/text-editor git@github.com:makerkit/remix-supabase-saas-kit-plugins.git text-editor --squash
```

After running this command, you will have the plugin in your repository at
`plugins/text-editor`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```
npx @makerkit/cli@latest plugins install
```

Follow the instructions to install the plugin.

#### Add the plugin as a workspace in your package.json

You can do so by adding the following to your `package.json` file:

```json
{
  "workspaces": [
    "plugins/text-editor"
  ]
}
```

Add it next to the other workspaces in your `package.json` file.

#### Add the paths alias to the TypeScript configuration

To make sure that the TypeScript compiler can find the plugin, you will need
to add the following paths alias to your `tsconfig.json` file, in addition
to the other paths aliases that you may have.

If not yet present, add the following to your `tsconfig.json` file:

```json
{
  "compilerOptions": {
    "paths": {
      "~/plugins/*": [
        "./plugins/*"
      ]
    }
  }
}
```

You only need to add this once, even if you have multiple plugins.

### Installing dependencies

To install the dependencies, you can run the following command:

```
npm i
```

NPM will install the dependencies in the `plugins/text-editor` folder as an NPM
workspace.

### Add the required API handlers

You will need to add the following API handlers to your application:
- `app/routes/resources.ai.autocomplete.ts`
- `app/routes/resources.ai.edit.ts`

#### Edit Route

To add the edit route, create a file at `app/routes/resources.ai.edit.ts` with the
following content:

```ts
import type { ActionFunctionArgs } from '@remix-run/node';
import { createAIEditActionHandler } from '~/plugins/text-editor/lib/route-handler';

export const action = (params: ActionFunctionArgs) =>
  createAIEditActionHandler(params.request);
```

#### Autocomplete Route

To add the autocomplete route, create a file at `app/routes/resources.ai.autocomplete.ts` with the
following content:

```ts
import type { ActionFunctionArgs } from '@remix-run/node';
import { createAIAutocompleteActionHandler } from '~/plugins/text-editor/lib/route-handler';

export const action = (params: ActionFunctionArgs) =>
  createAIAutocompleteActionHandler(params.request);
```

### Style

Import the CSS file at `plugins/text-editor/editor.css` into your global CSS
file at `app/styles/index.css`:

```css
@import "../../plugins/text-editor/editor.css";
```

### Using the plugin

To use the plugin, you can import the component from the plugin folder.

NB: the example below uses `rxjs`, which is a dependency not included in the
kit. I do recommend using it, but you can use any other library to handle
debouncing and other logic that you may need if you want to use autosaving
for your application.

```tsx
import { useCallback, useEffect, useMemo, useRef } from 'react';
import { toast } from 'sonner';

import {
  debounceTime,
  distinctUntilChanged,
  Subject,
  switchMap,
  tap,
} from 'rxjs';

import Editor from '~/plugins/text-editor/components/Editor';

export default function EditorContainer() {
  const subject$ = useMemo(() => new Subject(), []);
  const currentToastId = useRef<string | number>();

  const onChange = useCallback(
    (content: string) => {
      subject$.next(content);
    },
    [subject$],
  );

  useEffect(() => {
    const subscription = subject$
      .pipe(
        debounceTime(2000),
        distinctUntilChanged(),
        switchMap(() => {
          if (currentToastId.current) {
            toast.dismiss(currentToastId.current);
          }

          currentToastId.current = toast.loading('Saving...', {
            id: currentToastId.current,
          });

          return new Promise((resolve) => {
            setTimeout(resolve, 2000);
          });
        }),
        tap(() => {
          toast.success('Saved!', {
            id: currentToastId.current,
          });
        }),
      )
      .subscribe(() => {
        currentToastId.current = undefined;
      });

    return () => {
      subscription.unsubscribe();
    };
  }, [subject$]);

  return (
    <Editor
      className={'h-[80vh] w-[100vh]'}
      content={getInitialContent()}
      onChange={onChange}
    />
  );
}

function getInitialContent() {
  return `
## Introducing Makerkit's AI Editor

This plugin is powered by OpenAI's GPT-3 and Lexical's Editor, and helps you add an AI-powered editor to your SaaS, in just a few lines of code.

Install it using the CLI:

\`\`\`
npx @makerkit/cli plugins install
\`\`\`

Available for *Pro* and *Teams* customers **for free**.
  `.trim();
}
```

Now, import the component `EditorContainer` anywhere in your application:

```tsx
import { Toaster } from 'sonner';
import { lazy } from 'react';

import DarkModeToggle from '~/components/DarkModeToggle';
import Logo from '~/core/ui/Logo';
import ClientOnly from '~/core/ui/ClientOnly';

const EditorContainer = lazy(() => {
  return import('~/components/EditorContainer');
});

function EditorPage() {
  return (
    <div
      className={
        'w-screen h-screen flex justify-center items-center bg-gray-50' +
        ' dark:bg-dark-900 flex flex-col space-y-4'
      }
    >
      <Toaster />

      <div className={'fixed top-4 right-4'}>
        <DarkModeToggle />
      </div>

      <Logo href={'/'} className={'w-32 h-32'} />

      <ClientOnly>
        <EditorContainer />
      </ClientOnly>
    </div>
  );
}

export default EditorPage;
```

