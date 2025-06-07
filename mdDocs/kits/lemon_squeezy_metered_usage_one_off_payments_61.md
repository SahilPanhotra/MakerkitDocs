-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\lemon-squeezy.mdoc

---
status: "published"
label: "Lemon Squeezy"
title: "Configuring Lemon Squeezy Billing | React Router Supabase SaaS Kit Turbo"
order: 3
description: "Lemon Squeezy is a payment processor that allows you to charge your users for your SaaS product. It is a Merchant of Record, which means they handle all the billing and compliance for you."
---

For using Lemon Squeezy, we need to first set it as the default billing provider:

```bash
VITE_BILLING_PROVIDER=lemon-squeezy
```

Also, we need to switch the DB config to use Lemon Squeezy

```sql
update config set billing_provider = 'lemon-squeezy';
```

Then, you'll need to set the following environment variables:

```bash
LEMON_SQUEEZY_SECRET_KEY=
LEMON_SQUEEZY_SIGNING_SECRET=
LEMON_SQUEEZY_STORE_ID=
```

I am aware you know this, but never add these variables to the `.env` file. Instead, add them to the environment variables of your CI/CD system.

To test locally, you can add them to the `.env.local` file. This file is not committed to Git, therefore it is safe to store sensitive information in it.

### Schema Definition

Makerkit's billing configuration allows adding multiple line items into a single subscription: for example, you can mix flat fees, metered usage, and per-seat billing.

Lemon Squeezy **does not support multiple line items** like Stripe. The billing schema will fail validation if you do so, and you will need to adjust it.

Metered Usage can only be applied to the entire subscription, not to individual line items. For example, if you have a plan that charges $1 per 1000 requests, you can set the `tiers` property to charge $1 per 1000 requests.

However, you cannot have a plan that charges $1 per 1000 requests for one line item and $2 per 1000 requests for another line item. Or using a flat fee for one line item and metered usage for another.

#### Setup Fee + Metered Usage

Lemon Squeezy has support for the property `setupFee`: this property allows you to create a metered usage plan with a setup fee. That is, assuming you have a plan that charges a flat fee of $10 and then $1 per 1000 requests, you can set the `setupFee` to 10 and then set up the `tiers` to charge $1 per 1000 requests.

NB: the setup fee is only charged once, when the subscription is created.

### Testing locally

To receive webhooks from Lemon Squeezy, you need a proxy. You can create one for free with `ngrok` (or others). Once set up, create a webhook in Lemon Squeezy pointing to `{proxy-url}/api/billing/webhook` where `{proxy-url}` is a valid URL pointing to your local machine.

If your proxy is for example `https://myawesomeproxy.com` you will use `https://myawesomeproxy.com/api/billing/webhook` as the endpoint in Lemon Squeezy.

### Production

When going to production, you will set your exact application URL and scratch `{proxy-url}`. Don't forget, please.

Please set the following webhook events in Lemon Squeezy:

1. `order_created`
2. `subscription_created`
3. `subscription_updated`
4. `subscription_expired`


-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\metered-usage.mdoc

---
status: "published"
label: "Metered Usage"
title: "Configuring Metered Usage Billing | React Router Supabase SaaS Kit Turbo"
order: 5
description: "Metered usage billing is a billing model where you charge your customers based on their usage of your product. This model is common in APIs, where you charge based on the number of requests made to your API."
---

Metered usage billing is a billing model where you charge your customers based on their usage of your product. This model is common in APIs, where you charge based on the number of requests made to your API.

As we have already seen in the schema definition section, we can define a metered usage plan in the billing schema. This plan will charge the customer based on the number of requests they make to your API.

### Providers Differences

NB: different providers (Stripe, Lemon Squeezy, etc.) have different ways of handling metered usage billing: we keep the API consistent across all providers - but the details of how you report usage to the billing provider may differ. Please read the provider's documentation to understand how to report usage to the billing provider.

### Defining a Metered Usage Plan

Let's assume we have the following schema:

```tsx

export default createBillingSchema({
  provider,
  products: [
    {
      id: 'starter',
      name: 'Starter',
      description: 'The perfect plan to get started',
      currency: 'USD',
      badge: `Value`,
      plans: [
        {
          name: 'Starter Monthly',
          id: 'starter-monthly',
          trialDays: 7,
          paymentType: 'recurring',
          interval: 'month',
          lineItems: [
            {
              id: 'price_1NNwYHI1i3VnbZTqI2UzaHIe',
              name: 'Addon 2',
              cost: 0,
              type: 'metered',
              unit: 'GBs',
              tiers: [
                {
                    upTo: 10,
                    cost: 0.1,
                },
                {
                    upTo: 100,
                    cost: 0.05,
                },
                {
                    upTo: 'unlimited',
                    cost: 0.01,
                }
              ]
            },
          ],
        }
      ],
    }
  ]
});
```

What happens here is that we create a checkout in Stripe that charges the customer based on the number of GBs they use. The first 10 GBs are free, the next 90 GBs are charged at $0.05 per GB, and anything above 100 GBs is charged at $0.01 per GB.

When the checkout succeeds, we store two records:

1. A `subscriptions` record that represents the subscription the customer has to the plan. This is the overall subscription record - and contains details like the customer's ID, the plan ID, the status of the subscription, etc.
2. A `subscription_items` record that represents the line item the customer is subscribed to. This is needed for reporting charges to the billing provider.

The billing service in Makerkit has a unified interface for interacting with your billing provider - whichever it may be. This means that you can switch from Stripe to Lemon Squeezy or any other billing provider without changing your code. As such - this is valid for all billing providers supported by Makerkit.

When we report a charge, we need three things:
1. the subscription ID (and billing provider it is associated with) or the Customer ID (Stripe)
2. the line item ID (Lemon Squeezy only)
3. the next quantity of the line item

The billing service will then calculate the charge based on the quantity and the cost of the line item and charge the user accordingly.

On our side, we need to identify what the user is being charged for. This is why we store the `subscription_items` record. This record contains the `subscription_id`, the `product_id`, the `variant_id` (price ID in Stripe) and the `line_item_id` - which we can use to identify what the user is being charged for. You can query the items using these details to retrieve the ID of the line item the user is being charged for.

We assume an account uses a function `consumeApi` to consume the API. This (hypothetical) function will be called every time the user makes a request to the API. We can then use this function to report the usage to the billing provider.

```tsx
export async function consumeApi(accountId: string): number {}
```

When the user makes a request to the API, we can call this function to report the usage to the billing provider.

```tsx
async function apiHandler(accountId: string) {
  try {
    // assume consumeApi returns the number of requests made
    const quantity = await consumeApi(accountId);

    await reportUsage(accountId, quantity);
  } catch (error) {
    console.error(error);
  }
}
```

### Reporting Usage in Stripe

Makerkit uses the newest [usage reporting API in Stripe](https://docs.stripe.com/billing/subscriptions/usage-based/implementation-guide). This API allows you to report usage for a subscription item. You can report usage for a subscription item by calling the `reportUsage` method on the billing service.

Unlike the previous version of the API, you don't need a subscription item ID, but the customer ID and the metric name you're reporting.

We assume you have created a metric in Stripe named `api_requests` that you use to report the usage. Let's create a function that reports the usage for this metric caling it `reportUsageForApiRequests`:

```tsx
import { getBillingGatewayProvider } from '@kit/billing-gateway';
import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { createAccountsApi } from '@kit/accounts/api';

async function reportUsageForApiRequests(
  accountId: string,
  quantity: number
) {
  // use the correct client: in this case, the server action client
  const client = getSupabaseServerClient();
  const api = createAccountsApi(client);

  const subscription = await api.getSubscription(accountId);

  // if the subscription is not active, we don't report usage
  if (!subscription) {
    throw new Error('No active subscription found');
  }

  // get the billing provider
  const service = await getBillingGatewayProvider(this.client);
  const customerId = await api.getCustomerId(accountId);

  if (!customerId) {
    throw new Error(`No customer ID found for account ${accountId}`);
  }

  // now we can report the usage to the billing provider
  return service.reportUsage({
    id: customerId,
    eventName: 'api_requests',
    usage: {
      quantity,
    }
  });
}
```

As you can see, we use the following parameters to report usage:
1. `id`: the customer ID of the user/account in Stripe
2. `eventName`: the name of the metric you're reporting
3. `usage`: the quantity of the metric you're reporting

### Reporting Usage in Lemon Squeezy

In Lemon Squeezy, you need to report usage for a subscription item.

Given an account ID - we need to retrieve the subscription ID and the line item ID. We can then report the usage to the billing provider.

```tsx
import { getBillingGatewayProvider } from '@kit/billing-gateway';
import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { createAccountsApi } from '@kit/accounts/api';

async function reportUsageForApiRequests(
  accountId: string,
  quantity: number
) {
  // use the correct client: in this case, the server action client
  const client = getSupabaseServerClient();
  const api = createAccountsApi(client);

  const subscription = await api.getSubscription(accountId);

  // if the subscription is not active, we don't report usage
  if (!subscription) {
    console.error('No active subscription found');
    return;
  }

  // now, we need to find the line item the user is being charged for
  // let's use Supabase for this!
  // we use the product ID to identify the line item
  // in your case, you have more choices to identify the line item
  const {
    data: subscriptionItem,
    error
  } = await client.from('subscription_items')
    .select('id')
    .eq('subscription_id', subscription.id)
    .eq('product_id', 'starter-pro')
    .eq('type', 'metered')
    .single();

  // get the billing provider
  const service = await getBillingGatewayProvider(this.client);

  // now we can report the usage to the billing provider
  return service.reportUsage({
    id: subscriptionItem.id,
    usage: {
      quantity,
      action: 'increment'
    }
  });
}
```


-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\one-off-payments.mdoc

---
status: "published"
label: "One Off Payments"
title: "Configuring One Off Payments | React Router Supabase Kit"
order: 8
description: "How to configure one-off payments in Makerkit for products that are not recurring in nature."
---

While not exactly a common SaaS billing mode, Makerkit offers support for one-off payments.

You can use the tables `orders` and `orders_items` to store single orders for products not recurring in nature. You can use this in two ways:

1. **Lifetime Access**: for products that are not recurring but offer lifetime access.
2. **Multiple one-off payments**: for products that are not recurring but can be purchased multiple times. For example, you have multiple add-ons that can be purchased separately.

Some of this will require custom code, but generally speaking, Makerkit does its best to support this use case in a broad sense. The specifics of your implementation will depend on your product and business model and may require additional customization.

## Schema Definition

One-off payments are defined by the following fields:

```tsx
export default createBillingSchema({
  provider,
  products: [
    {
      id: 'starter',
      name: 'Starter',
      description: 'The perfect plan to get started',
      currency: 'USD',
      badge: `Value`,
      plans: [
        {
          name: 'Starter Monthly',
          id: 'starter-monthly',
          paymentType: 'one-time',
          lineItems: [
            {
              id: 'price_1NNwYHI1i3VnbZTqI2UzaHIe',
              name: 'Addon 2',
              cost: 9.99,
              type: 'flat',
            },
          ],
        }
      ],
    }
  ]
});
```

Let's break down the fields:
- **name**: The name of the plan
- **id**: The unique identifier for the line item. **This must match the price ID in the billing provider**. The schema will validate this, but please remember to set it correctly.
- **paymentType**: The payment type (e.g., `recurring`, `one-time`). In this case, it's `one-time`.
- **lineItems**: The line items for the plan
- **id**: The unique identifier for the line item. **This must match the price ID in the billing provider**. The schema will validate this, but please remember to set it correctly.
- **name**: The name of the line item
- **cost**: The cost of the line item
- **type**: The type of the line item (e.g., `flat`). It can only be `flat` for one-off payments.

When a products gets purchased, Makerkit will create an order in the `orders` table and an order item in the `orders_items` table. You can use this data to fulfill the order and grant access to the product.

If you would like to switch from `subscriptions` to `orders` as your primary billing mechanism, you can do so by setting the following environment variable:

```bash
BILLING_MODE=one-time
```

When this flags in set to `one-time`, Makerkit's plan's page will be looking for a valid order in the `orders` table instead of a subscription in the `subscriptions` table. This means that the billing section will display one-off payments instead of subscriptions.

As mentioned - this is a best-effort implementation. You may need to customize the billing pages to fit your specific use case.

-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\overview.mdoc

---
status: "published"
label: "How Billing Works"
title: "How Billing works in the React Router Supabase SaaS kit"
description: "A quick introduction to billing in Makerkit and how to set it up in the React Router Supabase SaaS kit"
order: 0
---

The billing package is used to manage subscriptions, one-off payments, and more.

The billing package is abstracted from the billing gateway package, which is used to manage the payment gateway (e.g., Stripe, Lemon Squeezy, etc.).

To set up the billing package, you need to set the following environment variables:

```bash
VITE_BILLING_PROVIDER=stripe # or lemon-squeezy
```

## Billing in Makerkit

Makerkit implements two packages for managing billing:

1. `core`: this package is responsible for exporting the billing service and the schema getFontDefinitionFromNetwork
2. `gateway`: the gateway is a router that handles the billing service and the billing provider (e.g., Stripe, Lemon Squeezy, etc.)

Then, we define a package for each provider:

1. `stripe`: this package is responsible for handling the Stripe API
2. `lemon-squeezy`: this package is responsible for handling the Lemon Squeezy API
3. `paddle`: this package is responsible for handling the Paddle API (Coming soon)

To summarize:

1. core defines the service and the schema
2. the provider's packages define the API calls based on the provider's API
3. the gateway package is responsible for routing the requests to the correct provider

Whatever provider you choose, you need to set the environment variable `VITE_BILLING_PROVIDER` to the provider you want to use. The Gateway Service will then route the requests to the correct provider. This means you can switch between providers without changing the code. The schema is the same for all providers - but the details of the API calls are different - so some details apply.

## Subscriptions vs. One-off payments

Makerkit supports both one-off payments and subscriptions. You have the choice to use one or both. What Makerkit cannot assume with certainty is the billing mode you want to use. By default, we assume you want to use subscriptions, as this is the most common billing mode for SaaS applications.

This means that - by default - Makerkit will be looking for a subscription plan when visiting the billing section of the personal or team account. This means we fetch data from the tables `subscriptions` and `subscription_items`.

If you want to use one-off payments, you need to set the billing mode to `one-time`:

```bash
BILLING_MODE=one-time
```

By doing so, Makerkit will be looking for one-off payments when visiting the billing section of the personal or team account. This means we fetch data from the tables `orders` and `order_items`.

### But - I want to use both

Perfect - you can, but you need to customize the pages to display the correct data.

---


Depending on the service you use, you will need to set the environment variables accordingly. By default - the billing package uses Stripe. Alternatively, you can use Lemon Squeezy. In the future, we will also add Paddle.

-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\paddle.mdoc

---
status: "published"
label: "Paddle"
title: "Configuring Paddle Billing | React Router Supabase SaaS Kit Turbo"
order: 4
description: "Paddle is a payment processor that allows you to charge your users for your SaaS product. It is a Merchant of Record, which means they handle all the billing and compliance for you."
---

{% alert type="info" title="Paddle is not yet available" %}
The integration will be developed after Paddle releases their Customer Portal.
{% /alert %}

Stay tuned!


-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\per-seat-billing.mdoc

---
status: "published"

label: "Per Seat Billing"
title: "How to configure per seat billing in Makerkit"
order: 6
description: "Learn how to setup per seat billing in Makerkit to charge customers for additional seats (users) in your application."
---


Per Seat billing is a common pricing model for SaaS applications where customers are charged based on the number of seats (users) they have in the application. In this guide, we'll show you how to configure per seat billing in Makerkit.

Makerkit will automatically:
1. understand if your pricing model depends on the number of seats (using the `per_seat` line item type in your pricing model)
2. calculate the number of seats based on the members count in a Team account
3. report the number of seats to your billing system when subscribing
4. report the number of seats to your billing system when adding or removing members

## Define a per-seat billing schema

Per-seat billing is defined by the following fields:

```tsx
export default createBillingSchema({
  provider,
  products: [
    {
      id: 'starter',
      name: 'Starter',
      description: 'The perfect plan to get started',
      currency: 'USD',
      badge: `Value`,
      plans: [
        {
          name: 'Starter Monthly',
          id: 'starter-monthly',
          trialDays: 7,
          paymentType: 'recurring',
          interval: 'month',
          lineItems: [
            {
              id: 'price_1NNwYHI1i3VnbZTqI2UzaHIe',
              name: 'Addon 2',
              cost: 0,
              type: 'per_seat',
              tiers: [
                {
                    upTo: 3,
                    cost: 0,
                },
                {
                    upTo: 5,
                    cost: 7.99,
                },
                {
                    upTo: 'unlimited',
                    cost: 5.99,
                }
              ]
            },
          ],
        }
      ],
    }
  ]
});
```

Let's break down the fields:
- **id**: The unique identifier for the line item. **This must match the price ID in the billing provider**. The schema will validate this, but please remember to set it correctly.
- **name**: The name of the line item
- **cost**: The cost of the line item. This can be set to `0` as the cost is calculated based on the tiers.
- **type**: The type of the line item (e.g., `flat`, `metered`, `per-seat`). In this case, it's `per-seat`.
- **tiers**: The tiers of the line item. Each tier is defined by the following fields:
- **upTo**: The upper limit of the tier. If the usage is below this limit, the cost is calculated based on this tier.
- **cost**: The cost of the tier. This is the cost per unit.

If you set the first tier to `0`, it basically means that the first `n` seats are free. This is a common practice in per-seat billing.

Please remember that the cost is set for UI purposes. **The billing provider will handle the actual billing** - therefore, **please make sure the cost is correctly set in the billing provider**.

## Report the number of seats

This is done automatically for you. Makerkit will report the number of seats to your billing provider when subscribing, and when adding or removing members.

-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\stripe.mdoc

---
status: "published"
label: "Stripe"
title: "Configuring Stripe Billing in the React Router Supabase SaaS Kit"
description: "Learn how to configure Stripe in the Makerkit React Router Supabase SaaS Kit"
order: 2
---

Stripe is the default billing provider in both the local config and the DB, so you don't need to set these values if you want to use Stripe.

For Stripe, you'll need to set the following environment variables:

```bash
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
VITE_STRIPE_PUBLISHABLE_KEY=
```

While the `VITE_STRIPE_PUBLISHABLE_KEY` is public and can be added anywhere - **please do not add the secret keys** to the `.env` file.

During development, you can place them in `.env.local` as it's not committed to the repository. In production, you can set them in the environment variables of your hosting provider.

## Stripe CLI

The Stripe CLI which allows you to listen to Stripe events straight to your own localhost. You can install and use the CLI using a variety of methods, but we recommend using Docker - since you already have it installed.

First - login to your Stripe account using the project you want to run:

```bash
docker run --rm -it --name=stripe -v ~/.config/stripe:/root/.config/stripe stripe/stripe-cli:latest login
```

Now, you can listen to Stripe events running the following command:

```bash
pnpm run stripe:listen
```

**If you have not logged in - the first time you set it up, you are required to sign in**. This is a one-time process. Once you sign in, you can use the CLI to listen to Stripe events.

**Please sign in and then re-run the command**. Now, you can listen to Stripe events.

Once you start listening to Stripe events, you can use the CLI to listen to Stripe events.

Now, please copy the webhook secret displayed in the terminal and set it as the `STRIPE_WEBHOOK_SECRET` environment variable in your `.env.local` file:

```bash
STRIPE_WEBHOOK_SECRET=*your_webhook_secret*
```

If you're not receiving events, please make sure that:

1. the webhook secret is correct
2. the account you signed in is the same as the one you're using in your app

## Configuring the Stripe Customer Portal

Stripe requires you to set up the Customer Portal so that users can manage their billing information, invoices and plan settings from there.

{% img src="/assets/images/docs/stripe-customer-portal.webp" width="2712" height="1870" /%}

1. Please make sure to enable the setting that lets users switch plans
2. Configure the behavior of the cancellation according to your needs

## Setting Production Webhooks in Stripe

When going to production, you will need to set the webhook URL and the events you want to listen to in Stripe.

The webhook path is `/api/billing/webhook`. If your app is hosted at `https://myapp.com` then you need to enter `https://myapp.com/api/billing/webhook`.

{% alert title="Use a public URL" type="warning" %}
Please make sure to use a public URL for your webhooks. If you use a private URL (such as Vercel preview URLs), you will not be able to receive webhooks from Stripe.
{% /alert %}

To ensure the URL is publicly accessible, please visit it using Incognito mode in your browser.

Makerkit needs the following events to work:

1. `checkout.session.completed`
2. `customer.subscription.updated`
3. `customer.subscription.deleted`

Only if you're using one-off payments, please add:

1. `checkout.session.async_payment_failed`
2. `checkout.session.async_payment_succeeded`

If your application needs more events, please add them, [but remember to handle them](billing-webhooks).

You can [handle additional events](billing-webhooks) by adding the required handlers in the `apps/web/app/routes/api/billing/webhook.ts` file.


-----------------
FILE PATH: kits\react-router-supabase-turbo\billing.mdoc

---
status: "published"
label: "Billing"
title: "How Billing works in the React Router Supabase SaaS kit"
description: "A quick introduction to billing in Makerkit and how to set it up in the React Router Supabase SaaS kit"
order: 7
collapsible: true
collapsed: true
---

The billing package is used to manage subscriptions, one-off payments, and more.

The billing package is abstracted from the billing gateway package, which is used to manage the payment gateway (e.g., Stripe, Lemon Squeezy, etc.).

To set up the billing package, you need to set the following environment variables:

```bash
VITE_BILLING_PROVIDER=stripe # or lemon-squeezy
```

## Billing in Makerkit

Makerkit implements two packages for managing billing:

1. `core`: this package is responsible for exporting the billing service and the schema getFontDefinitionFromNetwork
2. `gateway`: the gateway is a router that handles the billing service and the billing provider (e.g., Stripe, Lemon Squeezy, etc.)

Then, we define a package for each provider:

1. `stripe`: this package is responsible for handling the Stripe API
2. `lemon-squeezy`: this package is responsible for handling the Lemon Squeezy API
3. `paddle`: this package is responsible for handling the Paddle API (Coming soon)

To summarize:

1. core defines the service and the schema
2. the provider's packages define the API calls based on the provider's API
3. the gateway package is responsible for routing the requests to the correct provider

Whatever provider you choose, you need to set the environment variable `VITE_BILLING_PROVIDER` to the provider you want to use. The Gateway Service will then route the requests to the correct provider. This means you can switch between providers without changing the code. The schema is the same for all providers - but the details of the API calls are different - so some details apply.

## Subscriptions vs. One-off payments

Makerkit supports both one-off payments and subscriptions. You have the choice to use one or both. What Makerkit cannot assume with certainty is the billing mode you want to use. By default, we assume you want to use subscriptions, as this is the most common billing mode for SaaS applications.

This means that - by default - Makerkit will be looking for a subscription plan when visiting the billing section of the personal or team account. This means we fetch data from the tables `subscriptions` and `subscription_items`.

If you want to use one-off payments, you need to set the billing mode to `one-time`:

```bash
BILLING_MODE=one-time
```

By doing so, Makerkit will be looking for one-off payments when visiting the billing section of the personal or team account. This means we fetch data from the tables `orders` and `order_items`.

### But - I want to use both

Perfect - you can, but you need to customize the pages to display the correct data.

---


Depending on the service you use, you will need to set the environment variables accordingly. By default - the billing package uses Stripe. Alternatively, you can use Lemon Squeezy. In the future, we will also add Paddle.


