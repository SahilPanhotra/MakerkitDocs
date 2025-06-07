-----------------
FILE PATH: kits/next-supabase/organizations/subscription-permissions.mdoc

---
status: "published"
title: Subscription Permissions | Next.js Supabase SaaS Kit
label: Subscription Permissions
order: 3
description: "Learn how to gate access to certain features or pagess based
on the organization's subscription"
---

Makerkit provides some utilities to retrieve the current organization's
subscription: you can use these utilities to allow or gate access to
specific features, pages, or to perform certain actions.

1. On the client side, we can use the hook `useIsSubscriptionActive`
2. On the server-side, we can use the functions
`getOrganizationSubscriptionActive` and `getOrganizationSubscription`

## Client Side Gating

Let's assume we want to allow only users who belong to paying organizations to
perform a certain action:

 ```tsx {% title="src/lib/organizations/permissions.ts" %}
function isAdmin(
  role: MembershipRole
) {
  return role === MembershipRole.Admin;
}
```

Now, combine this function with other conditions:

 ```tsx {% title="src/lib/organizations/permissions.ts" %}
export function useCreateNewThing(
  userRole: MembershipRole,
) {
  const isPayingUser = useIsSubscriptionActive();

  return isPayingUser && isAdmin(userRole);
}
```

As you may know, I recommend adding all the permissions logic to the file
`permissions.ts` (or similar files), and never inline it: this makes it easy to
store logic in one single place rather than scattered across the
application. 

Inlining logic in our code will make it hard to track logic
down and debugging it.

Once defined out logic functions in `permissions.ts`, we can then use these
in our components to hide the sections that users do
not have access to:

```tsx
function Feature() {
  const userRole = useCurrentUserRole();
  const canCreateThing = useCreateNewThing(userRole);

  if (!canCreateThing) {
    return <div>Sorry, you do not have access to this feature. Subscribe?</div>
  }

  return <FeatureContainer />;
}
```

NB: You should also tighten up the security on the server-side using RLS to prevent users from accessing data they should not have access to.

## Server Side Gating

When gating access on the server side, we can use two utilities that allow
us to get the organization's subscription by its ID.

### Gating access to certain pages using Server Components

For example, let's assume we want to gate access to a page if the
`organization` is not on a paid plan: we can prevent the page from rendering by adding some logic to the Server Component rendering the page.

```tsx
import { use } from "react";
import { redirect } from "next/navigation";
import { Stripe } from "stripe";
import loadAppData from '~/lib/server/loaders/load-app-data';

interface Params {
  params: {
    organization: string;
  }
}

function GatedPage({ params }: Params) {
  const canAccessPage = use(canUserAccessPage(params.organization));

  if (!canAccessPage) {
    redirect('/dashboard?error=forbidden-subscription');
  }

  // ...
}

async function canUserAccessPage(
  organizationUid: string
) {
  const data = await loadAppData(organizationUid);
  const subscription = data.organization.subscription?.data;

  return subscription && isSubscriptionActive(subscription.status);
}

function isSubscriptionActive(status: Stripe.Subscription.Status) {
  return ['active', 'trialing'].includes(status);
}

export default GatedPage;
```

If you want to go more granular, you can also check the subscription's price ID to match a specific plan for example.

Of course, if you reuse the logic above very often, you can also extract it to a function and reuse it across your Server Components:

```tsx
import { redirect } from "next/navigation";

async function canUserAccessProFeature(
  organizationUid: string
) {
  const data = await loadAppData(organizationUid);
  const subscription = data.organization.subscription?.data;

  return subscription && isSubscriptionActive(subscription.status);
}

function isSubscriptionActive(status: Stripe.Subscription.Status) {
  return ['active', 'trialing'].includes(status);
}

export async function withSubscriptionGuard(organizationUid: string) {
  const canAccessPage = await canUserAccessProFeature(organizationUid);

  if (!canAccessPage) {
    redirect('/dashboard?error=forbidden-subscription');
  }
}
```

And reuse it in your Server Components:

```tsx
interface Params {
  params: {
    organization: string;
  }
}

async function GatedPage({ params }: Params) {
  await withSubscriptionGuard(params.organization);

  // ...
}

export default GatedPage;
```

-----------------
FILE PATH: kits/next-supabase/organizations.mdoc

---
status: "published"
title: "Organizations in Next.js Supabase (legacy)"
label: "Organizations"
order: 11
description: "Learn how to add organizations to your Next.js Supabase application"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/payments/lemon-squeezy.mdoc

---
status: "published"
title: Use Lemon Squeezy with MakerKit | Next.js Supabase
label: Migrate to Lemon Squeezy
order: 4
description: 'Learn how to migrate your Next.js Supabase MakerKit project to Lemon Squeezy'
---

If you prefer using Lemon Squeezy instead of Stripe with your Makerkit kit, we provide a different branch from the main repository that you can use.

The branch is called `main-ls`. If you want to use Lemon Squeezy, you can clone the repository and checkout the `main-ls` branch.

NB: the integration is in beta, and I am working on improving it.

## Setting up Lemon Squeezy

### Step 1: Create a Lemon Squeezy account

First, you need to create a Lemon Squeezy account. You can do so by visiting [https://lemonsqueezy.io](https://lemonsqueezy.com).

Get your store setup (start in Test mode, so you don't need to wait for activation).

### Step 2: Create a Product/Variant in Lemon Squeezy

You can see this guide for [setting up SaaS subscription plans in Lemon Squeezy](https://docs.lemonsqueezy.com/guides/tutorials/saas-subscription-plans).

If you want, set up a monthly/annual plan for your product, or just a single plan.

{% img src="/assets/images/posts/lemon-squeezy-plans.webp" width="1286" height="816" /%}

To replicate the basic Makerkit subscription plans, you can create a plan for each of the following: Basic, Pro and Premium plans.

To get started quickly, you can also set up a single plan for your product.

### Step 3: Create a Lemon Squeezy API key

Now, we need to setup an API Key. Go to your Lemon Squeezy settings page and click on the [API Keys tab](https://app.lemonsqueezy.com/products).

Click on the "Create API Key" button and give it a **very specific name**, so you won't confuse it with your production keys. Copy the API key and ensure to store it. We will be adding this to your Makerkit's local environment variables file.

{% img src="/assets/images/posts/lemon-squeezy-api-key-name.webp" width="946" height="614" /%}

Depending on which kit you're using, you will add this key to either `.env.local` (Next.js) or `.env` (Next.js). This key is not supposed to be committed to your repository, so we add them safely to our local files.

Additionally, we need to add the `LEMON_SQUEEZY_STORE_ID` to the local environment variables file, and a secret string `LEMON_SQUEEZY_SIGNING_SECRET` that we use to verify the webhook signature.

Update them in your local environment variables file:

```
LEMON_SQUEEZY_API_KEY=<YOUR_KEY>
LEMON_SQUEEZY_STORE_ID=<YOUR_STORE_ID>
LEMON_SQUEEZY_SIGNING_SECRET=<a random string>
```

#### Find your Lemon Squeezy Store ID

To find your Lemon Squeezy Store ID, visit your `Stores` page in the Lemon Squeezy dashboard.

You can do so by visiting the following URL: [https://app.lemonsqueezy.com/settings/stores](https://app.lemonsqueezy.com/settings/stores).

The ID is the number next to the store name.

Proceed by copying the Store ID into the `LEMON_SQUEEZY_STORE_ID` environment variable.

## Step 4: Configure Lemon Squeezy plans in the Makerkit configuration

Now, we need to configure the Lemon Squeezy plans in the Makerkit configuration, and enter the correct plan IDs.

To do so, open the `~/configuration.ts` file and edit the `stripe` object.

We can convert this to something more abstract, such as `subscriptions`, and add the relevant Lemon Squeezy product and variant IDs.

 ```ts {% title="src/configuration.ts" %}
{
  subscriptions: {
    products: [
      {
        name: 'Basic',
        description: 'Description of your Basic plan',
        badge: `Up to 20 users`,
        productId: 1, // <-- Lemon Squeezy product ID
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
            variantId: 1, // <-- Lemon Squeezy variant ID
          },
          {
            name: 'Yearly',
            price: '$90',
            variantId: 2, // <-- Lemon Squeezy variant ID
          },
        ],
      }
    ]
  }
}
```

**Note**: The `variantId` is the ID of the plan in Lemon Squeezy. The `productId` is the ID of the product.

##### Find your Lemon Squeezy product and variant IDs

You can find your Lemon Squeezy Product and Variant IDs using the Lemon Squeezy dashboard.

### Step 4: Creating the Webhook In Lemon Squeezy

We need to create a Webhook from the Lemon Squeezy Dashboard so that we can receive the events to our webhook handler.

You will be prompted for the following:

- URL: point it to the ngrok URL (see below). Ensure you point it to the `/api/ls/webhook` endpoint.
- Secret: the secret `LEMON_SQUEEZY_SIGNING_SECRET` that you have set in the `.env` file
- Events: select the events that you want to receive. In our case, we want to receive the `subscription_created` and `subscription_updated` events.

{% img src="/assets/images/posts/lemon-squeezy-webhook-setup.webp" width="1118" height="1666" /%}

#### Testing Locally

To test the webhook handler locally, we can use the [ngrok](https://ngrok.com/) tool.

We add the following command to the `package.json` file:

```
"ngrok": "npx ngrok http 3000"
```

And you can run it with:

```bash
npm run ngrok
```

You can also use other alternatives, such as Localtunnel or Cloudflare Tunnel.

### Step 5: Go to production

Once you're ready to go to production, you can switch your Lemon Squeezy store to production mode.

Additionally, remember to set up the webhook in production mode as well, pointing to your production URL.

-----------------
FILE PATH: kits/next-supabase/payments/stripe-configuration.mdoc

---
status: "published"
title: Set up Stripe Payments for Next.js and Supabase SaaS applications
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

The API endpoint needs to point to your application's Stripe webhook: the path is `/api/stripe/webhook`. For example, if your application is hosted at `https://myapp.com`, the webhook endpoint would be `https://myapp.com/api/stripe/webhook`.

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
          stripePriceId: 'price_1LfXGaI1i3VnbZTq7l3VgZNa',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$90',
          stripePriceId: 'basic-plan-yr',
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

If you prefer to redirect users to Stripe Checkout, you can set `stripe.embedded` to `false`. This is the default configuration in versions before 0.14.0 as embedded checkouts were not available (released in October 2023).

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

Restart the next.js server to apply the changes.

## Billing Portal

MakerKit uses the [Stripe Billing Portal](https://dashboard.stripe.com/test/settings/billing/portal) to let your users manage their invoices and subscriptions.

The portal becomes accessible once the organization subscribes to a plan, which creates a `customerId` property, making it possible for the organization to access the portal.

### Enabling the Billing Portal in test mode

Please visit the [Stripe Billing Page](https://dashboard.stripe.com/test/settings/billing/portal) to enable the Billing Portal.

Choose the settings that best apply to your product and save your changes.

The only required fields to get started are `Terms of Service` and
`Privacy` URLs. You can add any URL at this stage. **However, remember to update these pages when you go live**.

For a technical deep-dive, we recommend reading our blog post where we [build the integration between Stripe and Next.js from scratch](/blog/tutorials/guide-nextjs-stripe). Because the result of the blog post is a simplified version of the Makerkit's implementation, it can be beneficial for understanding its code.


-----------------
FILE PATH: kits/next-supabase/payments/stripe-one-time-payments.mdoc

---
status: "published"
title: Enabling One-Time Payments with Stripe | Next.js Supabase Kit
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
FILE PATH: kits/next-supabase/payments/stripe-webhooks.mdoc

---
status: "published"
title: How Makerkit handles Stripe Webhooks in your Next.js Supabase application
label: Stripe Webhooks
order: 1
description: 'Learn how MakerKit handles Stripe Webhooks in your Next.js Supabase application'
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
FILE PATH: kits/next-supabase/payments.mdoc

---
status: "published"
title: "Payments in Next.js Supabase"
label: "Payments"
order: 6
description: "Payments in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/plugins/ai-chatbot.mdoc

---
status: "published"

title: "AI Chatbot Plugin for the Next.js Supabase Saas Starter Kit"
label: "AI Chatbot"
order: 2
description: "This plugin adds an AI chatbot to your Next.js Supabase SaaS application."
---


This plugin adds an AI chatbot trained on your website content to your Next.js
application built using the OpenAI API. It's a simple component that will be
displayed on the bottom right of your website.

This documentation is for the Next.js Supabase version. Please make sure you're looking at the right documentation.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/chatbot git@github.com:makerkit/next-supabase-saas-kit-plugins.git chatbot --squash
```

After running this command, you will have the plugin in your repository at
`plugins/chatbot`. Once pulled, you can apply any customization you need.

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
    "plugins/chatbot"
  ]
}
```

Add it next to the other workspaces in your `package.json` file.

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:

1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/chatbot https://github.com/makerkit/next-supabase-saas-kit-plugins chatbot --squash
```

#### Add the paths alias to the TypeScript configuration

To make sure that the TypeScript compiler can find the plugin, you will need
to add the following paths alias to your `tsconfig.json` file, in addition
to the other paths aliases that you may have:

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

### Installing dependencies

To install the dependencies, you can run the following command:

```
npm i
```

NPM will install the dependencies in the `plugins/chatbot` folder as an NPM workspace.

### Configuring the database migration

This plugin needs to store the documents that the AI will be trained on as
vectors in the Supabase database. As such, we need to add a migration to the
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

You can import the `Chatbot` component from the plugin in your `app/layout.tsx`
file if you want it available on all pages:

```tsx
import dynamic from 'next/dynamic';

const ChatBot = dynamic(() => import('~/plugins/chatbot/components/ChatBot'));

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
pages, then you will import it in the `app/(site)/layout.tsx` layout.

#### Initial Prompts

You can pass an array of strings to the `defaultPrompts` prop of the
component:

```tsx
const DEFAULT_CHAT_PROMPTS = [
  `Can you tell me more about ${configuration.site.siteName}?`,
  `What is the price of ${configuration.site.siteName}?`,
  `How can I contact you?`,
  `I want to share some feedback`,
];

// in the component

<Chatbot defaultPrompts={DEFAULT_CHAT_PROMPTS} />
```

### Configuring the plugin

To configure the plugin, add the following environment variables to your
`.env.local` files:

```
OPENAI_API_KEY=
NEXT_PUBLIC_CHATBOT_FALLBACK_URL=
```

- `OPENAI_API_KEY` - Your OpenAI API key. You can get one from your OpenAI
dashboard.
- `NEXT_PUBLIC_CHATBOT_FALLBACK_URL` - The URL of the fallback chatbot. This
is the link the chatbot will display if it can't answer the question. You
can use an email such as `mailto:youremail@yourstartup.com` or a link to
the customer service page of your website.

When going to production, you will need to add these environment variables
from your provider dashboard.

### Setting the API Route Handler

#### CSRF exemption

Open the file `src/app/middleware.ts` and add the route `/api/chat` to the
CSRF exemption list:

```ts
export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|locales|assets|api/chat|api/stripe/webhook).*)',
  ],
};
```

#### Next.js Route

Create a route at `app/api/chat/route.ts` with the following content:

```ts
import handleChatBotRequest from '~/plugins/chatbot/lib/server/route-handler';

export const runtime = 'edge';

export const POST = handleChatBotRequest;
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
will need to provide the Supabase credentials in the `.env.production` file.

### Captcha Protection (optional) (WIP)

To protect your chatbot from bots (ha ha!), you can add a captcha challenge
using [Cloudflare's Turnstile](https://developers.cloudflare.com/turnstile/)
service.

To do so, you will need to add the following environment variables to your
environment files:

```
NEXT_PUBLIC_CHATBOT_TURNSTILE_SITE_KEY=
```

[You can get the site key from your Turnstile dashboard](https://developers.
cloudflare.com/turnstile/get-started/).

On the server side, you will need to add the following environment
variables: `CHATBOT_TURNSTILE_SECRET_KEY`.

You can also get the secret key from your Turnstile dashboard. Since this is
a secret key - avoid adding it to your repository. Please use the
environment variables of your provider to add it to your production environment.

### De-duplicating indexing

To avoid indexing the same content multiple times, the CLI generates a file
named `plugins/chatbot/indexed-files.json` that contains a list of all the
pages that have been indexed and the SHA256 hash of their content.

If the content of the file changes, the CLI will re-index the pages.

### Keeping the plugin up to date

To keep the plugin up to date, you can use git subtrees again:

```
git subtree pull --prefix plugins/chatbot git@github.com:makerkit/next-supabase-saas-kit-plugins.git chatbot --squash
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

