-----------------
FILE PATH: kits\next-supabase-turbo\billing\billing-webhooks.mdoc

---
status: "published"
label: "Handling Webhooks"
title: Handle billing webhooks with your custom code in Next.js Supabase Kit"
order: 9
description: "Learn how to handle billing webhooks with your custom code in Next.js Supabase Kit"
---

{% sequence title="Steps to handle billing webhooks" description="Learn how to handle billing webhooks with your custom code in Next.js Supabase Kit." %}

[Handling billing webhooks](#handling-billing-webhooks)

[Customizing the webhook handler](#customizing-the-webhook-handler)

{% /sequence %}

## Handling billing webhooks

Makerkit takes care of handling billing webhooks to update the Database based on the events received from Stripe.

Sometimes - you will need to set more webhooks, or do something custom with the webhooks.

In these cases, you can customize the billing webhook handler in Makerkit at `api/billing/webhooks/route.ts`.

By default, the webhook handler is set to `service.handleWebhookEvent(request)`:

```tsx
await service.handleWebhookEvent(request);
```

However, you can extend it using the callbacks provided by the `BillingService`:

```tsx
await service.handleWebhookEvent(request, {
  onPaymentFailed: async (sessionId) => {},
  onPaymentSucceeded: async (sessionId) => {},
  onCheckoutSessionCompleted: async (subscription, customerId) => {},
  onSubscriptionUpdated: async (subscription) => {},
  onSubscriptionDeleted: async (subscriptionId) => {},
});
```

You can provide one or more of the callbacks to handle the events you are interested in.

## Customizing the webhook handler

If the event is not in one of these methods, you can handle it in the `onEvent` method:

```tsx
await service.handleWebhookEvent(request, {
  async onEvent(data: unknown) {
    logger.info(
      `Received billing event`,
    );

    // Your custom code here
  }
});
```

However, you need to set the correct interface for the `data` parameter to handle the event correctly.

For example, to handle the `invoice.payment_succeeded` event, you can use the `onEvent` method:

```tsx
await service.handleWebhookEvent(request, {
  async onEvent(data: unknown) {
    if (data.type === 'invoice.payment_succeeded') {
      const invoice = data as Stripe.Invoice;
      // Your custom code here
    }
  }
});
```

You can find the list of events and their data in the [Stripe documentation](https://stripe.com/docs/api/events/types).


-----------------
FILE PATH: kits\next-supabase-turbo\billing\credit-based-billing.mdoc

---
status: "published"
label: 'Credits Based Billing'
title: 'How to add Credit Based billing to Makerkit | Next.js Supabase'
order: 7
description: 'Learn how to add credit based usage in your Next.js Supabase application'
---

{% sequence title="How to add credit based usage in your Next.js Supabase application" description="Learn how to add credit based usage in your Next.js Supabase application" %}

[Credit-based billing](#credit-based-billing)

[How to implement credit-based billing](#how-to-implement-credit-based-billing)

{% /sequence %}

## Credit-based billing

Credit-based billing is a billing model where you charge your users based on the number of credits they consume. This model is useful when you want to charge your users based on the number of actions they perform in your application.

As you may know, this is extremely popular for AI SaaS products, where users may have a limited amount of tokens (or credits) to perform actions - such as requesting data from an LLM.

Makerkit doesn't have a built-in credit-based billing system, but you can easily implement it using the existing billing system.

{% alert type="info" title="This example is for Stripe Subscriptions" %}
The example below is specific to Stripe Subscriptions.
{% /alert %}

## How to implement credit-based billing

To do so, we can introduce two new tables to our database: `plans` and `credits`. The `plans` table will store the pricing information for each plan, and the `credits` table will store the credits consumed by each user.

Here's how you can set it up:

### Step 1: Create the `plans` table

First we need to create a `plans` table to store the pricing information for each plan.

```sql
create table public.plans (
  id serial primary key,
  name text not null,
  variant_id text not null
);

alter table public.plans enable row level security;

-- allow authenticated users to read plans
create policy read_plans
  on public.plans
  for select
  to authenticated
  using (true);
```

We've created a `plans` table with two columns: `name` and `variant_id`. The `name` column stores the name of the plan, and the `variant_id` column stores the ID of the plan variant.

We also enabled row-level security on the `plans` table and created a policy that allows authenticated users to read the plans.

### Step 2: Create the `credits` table

Next, we need to create a `credits` table to store the credits consumed by each user.

```sql
create table public.credits (
  account_id uuid not null references public.accounts(id),
  tokens integer not null
);

alter table public.credits enable row level security;

-- allow authenticated users to read their credits
create policy read_credits
  on public.credits
  for select
  to authenticated
  using (
    account_id = (select auth.uid())
  );
```

In the above SQL, we create two tables: `plans` and `credits`. The `plans` table stores the pricing information for each plan, and the `credits` table stores the credits consumed by each user.

We also enabled row-level security on the `credits` table and created a policy that allows authenticated users to read their credits.

### Step 3: Functions to manage credits

Next, we need to create functions to manage the credits. We'll call these functions `has_credits` and `consume_credits`.

```sql
create or replace function public.has_credits(account_id uuid, tokens integer)
  returns boolean
  set search_path = ''
  as $$
  begin
    return (select tokens >= tokens from public.credits where account_id = account_id);
  end;
  $$ language plpgsql;

grant execute on function public.has_credits to authenticated, service_role;
```

The `has_credits` function checks if the user has enough credits to perform an action. It takes the `account_id` and the number of tokens required as arguments and returns `true` if the user has enough credits, `false` otherwise.

```sql
create or replace function public.consume_credits(account_id uuid, tokens integer)
  returns void
  set search_path = ''
  as $$
  begin
    update public.credits set tokens = tokens - tokens where account_id = account_id;
  end;
  $$ language plpgsql;

grant execute on function public.has_credits to service_role;
```

You can now use these functions in your RLS policies, or in your application code to manage credits.

NB: we only allow authenticated users to read their credits. To update the credits, we use the service role key, since we lock down the `credits` table to only allow authenticated users to read their credits.

### Step 4: Using credits in your application

Now that you have set up the `plans` and `credits` tables, you can use them in your application to manage credits.

For example, when a user performs an action that consumes credits, you can call the `consume_credits` function to deduct the credits from the user's account. You can use the Supabase client to call the function from your application code.

NB: the `callOpenAIApi` is a placeholder for your API call to OpenAI. You should replace it with your actual API call.

NB: we assume this is in a route handler, so we use the `getSupabaseRouteHandlerClient` client. If you're in a server action, you should use the `getSupabaseServerClient` client. [Learn more about the Supabase clients here](../data-fetching/supabase-clients).

```tsx
export function async consumeApi(accountId: string) {
  // Call the OpenAI API to get the usage
  const { usage, data } = await callOpenAIApi();

  const client = getSupabaseRouteHandlerClient({
    admin: true,
  });

  await client.rpc('consume_credits', {
    account_id: accountId,
    tokens: usage,
  });

  return data;
}
```

You can also use the function `has_credits` as part of your RLS policy to restrict access to certain resources based on the user's credits.

```sql
create policy tasks_write_policy
  on public.tasks
  for select
  using (
    (select auth.uid()) === account_id and
    public.has_credits((select auth.uid()), 1)
  );
```

### Step 5: Recharge credits

You can use webhooks from your billing provider to recharge credits when a user makes a payment. To do so, listen to events and update the `credits` table accordingly.

In the example below, we extend the webook handler to listen to the `onInvoicePaid` (subscriptions-only) event and update the `credits` table with the new credits.

{% alert type="warning" title="This example is for Stripe Subscriptions" %}
By default, Stripe does not create invoices for one-off payments. You need to manually enable this functionality in the Stripe dashboard, or you can listen to the `onPaymentSucceeded` event instead.
{% /alert %}

1. First, update the existing webhook handler to listen to the `onInvoicePaid` event.
2. Then, update the `credits` table with the new credits.

 ```tsx {% title="apps/web/app/api/billing/webhook/route.ts" %}
import { getBillingEventHandlerService } from '@kit/billing-gateway';
import { enhanceRouteHandler } from '@kit/next/routes';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

import billingConfig from '~/config/billing.config';
import { Database } from '~/lib/database.types';

/**
 * @description Handle the webhooks from Stripe related to checkouts
 */
export const POST = enhanceRouteHandler(
  async ({ request }) => {
    const provider = billingConfig.provider;
    const logger = await getLogger();

    const ctx = {
      name: 'billing.webhook',
      provider,
    };

    logger.info(ctx, `Received billing webhook. Processing...`);

    const supabaseClientProvider = () =>
      getSupabaseServerAdminClient();

    const service = await getBillingEventHandlerService(
      supabaseClientProvider,
      provider,
      billingConfig,
    );

    try {
      await service.handleWebhookEvent(request, {
        onInvoicePaid: async (data) => {
          const accountId = data.target_account_id;
          const lineItems = data.line_items;

          // we only expect one line item in the invoice
          // if you add more than one, you need to handle that here
          // by finding the correct line item to get the variant ID
          const variantId = lineItems[0]?.variant_id;

          if (!variantId) {
            logger.error(
              {
                accountId,
              },
              'Variant ID not found in line items',
            );

            throw new Error('Variant ID not found in invoice');
          }

          await updateMessagesCountQuota({
            variantId,
            accountId,
          });
        },
      });

      logger.info(ctx, `Successfully processed billing webhook`);

      return new Response('OK', { status: 200 });
    } catch (error) {
      logger.error({ ...ctx, error }, `Failed to process billing webhook`);

      return new Response('Failed to process billing webhook', {
        status: 500,
      });
    }
  },
  {
    auth: false,
  },
);

async function updateMessagesCountQuota(params: {
  variantId: string;
  accountId: string;
}) {
  const adminClient = getSupabaseServerAdminClient();

  // get the max messages for the price based on the price ID
  const plan = await adminClient
    .from('plans')
    .select('tokens_quota')
    .eq('variant_id', params.variantId)
    .single();

  if (plan.error) {
    throw plan.error;
  }

  const { tokens_quota } = plan.data;

  // upsert the message count for the account
  // and set the period start and end dates (from the subscription)
  const response = await adminClient
    .from('credits')
    .update({
      tokens_quota,
    })
    .eq('account_id', params.accountId);

  if (response.error) {
    throw response.error;
  }
}
```

We assume that we have a table named `plans` that stores the pricing information for each plan. We also assume that we have a table named `credits` that stores the credits consumed by each user.
The `credits` table has a column named `tokens_quota` that stores the number of credits available to the user.

In the `onInvoicePaid` event handler, we get the variant ID from the invoice line items and update the `credits` table with the new credits.

-----------------
FILE PATH: kits\next-supabase-turbo\billing\lemon-squeezy.mdoc

---
status: "published"
label: "Lemon Squeezy"
title: "Configuring Lemon Squeezy Billing in the Next.js Supabase SaaS Starter Kit"
order: 3
description: "Lemon Squeezy is a payment processor that allows you to charge your users for your SaaS product. It is a Merchant of Record, which means they handle all the billing and compliance for you."
---

For using Lemon Squeezy, we need to first set it as the default billing provider:

```bash
NEXT_PUBLIC_BILLING_PROVIDER=lemon-squeezy
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

However, with Lemon Squeezy, you can adjust various fields to fit your needs, albeit with very limited flexibility.

#### Metered Usage

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
FILE PATH: kits\next-supabase-turbo\billing\metered-usage.mdoc

---
status: "published"
label: "Metered Usage"
title: "Configuring Metered Usage Billing | Next.js Supabase SaaS Kit Turbo"
order: 5
description: "Metered usage billing is a billing model where you charge your customers based on their usage of your product. This model is common in APIs, where you charge based on the number of requests made to your API."
---

{% sequence title="How to configure metered usage billing in the Next.js Supabase SaaS Kit Turbo" description="Let's see how to configure metered usage billing in the Next.js Supabase SaaS Kit Turbo" %}

[Providers Differences in reporting usage](#providers-differences-in-reporting-usage)

[Defining a Metered Usage Plan](#defining-a-metered-usage-plan)

[Reporting Usage in Stripe](#reporting-usage-in-stripe)

[Reporting Usage in Lemon Squeezy](#reporting-usage-in-lemon-squeezy)

{% /sequence %}

Metered usage billing is a billing model where you charge your customers based on their usage of your product. This model is common in APIs, where you charge based on the number of requests made to your API.

As we have already seen in the schema definition section, we can define a metered usage plan in the billing schema. This plan will charge the customer based on the number of requests they make to your API.

### Providers Differences in reporting usage

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

NB: this is meta-code. These functions need to be implemented by you.

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

NB: You need to implement the `reportUsageForApiRequests` function in your code. The above is just an example.

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
FILE PATH: kits\next-supabase-turbo\billing\one-off-payments.mdoc

---
status: "published"
label: "One Off Payments"
title: "Configuring One Off Payments | Next.js Supabase Kit"
order: 8
description: "How to configure one-off payments in Makerkit for products that are not recurring in nature."
---

{% sequence title="One Off payments" description="Setting up one-off payments in Makerkit" %}

[Schema Definition](#schema-definition)

[Setting the billing mode](#setting-the-billing-mode)

{% /sequence %}

{% alert type="warning" title="One off payments and subscriptions cannot coexist" %}
At this time, one off payments and subscriptions cannot coexist. If you enable one off payments, you will not be able to use subscriptions.
{% /alert %}

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

When a products gets purchased, Makerkit will create an order in the `orders` table and an order item in the `orders_items` table. You can use this data to fulfill the order and grant access to the product.

## Setting the billing mode

If you would like to switch from `subscriptions` to `orders` as your primary billing mechanism, you can do so by setting the following environment variable:

```bash
BILLING_MODE=one-time
```

When this flags in set to `one-time`, Makerkit's plan's page will be looking for a valid order in the `orders` table instead of a subscription in the `subscriptions` table. This means that the billing section will display one-off payments instead of subscriptions.

As mentioned - this is a best-effort implementation. You may need to customize the billing pages to fit your specific use case.

-----------------
FILE PATH: kits\next-supabase-turbo\billing\overview.mdoc

---
status: "published"
label: "How Billing Works"
title: "How Billing works in the Next.js Supabase SaaS kit"
description: "A quick introduction to billing in Makerkit and how to set it up in the Next.js Supabase SaaS kit"
order: 0
---

{% sequence title="How Billing Works" description="A quick introduction to billing in Makerkit and how to set it up in the Next.js Supabase SaaS kit" %}

[Billing in Makerkit](#billing-in-makerkit)

[Subscriptions vs. One-off payments](#subscriptions-vs.-one-off-payments)

{% /sequence %}

The billing package is used to manage subscriptions, one-off payments, and more.

The billing package is abstracted from the billing gateway package, which is used to manage the payment gateway (e.g., Stripe, Lemon Squeezy, etc.).

To set up the billing package, you need to set the following environment variables:

```bash
NEXT_PUBLIC_BILLING_PROVIDER=stripe # or lemon-squeezy
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

Whatever provider you choose, you need to set the environment variable `NEXT_PUBLIC_BILLING_PROVIDER` to the provider you want to use. The Gateway Service will then route the requests to the correct provider. This means you can switch between providers without changing the code. The schema is the same for all providers - but the details of the API calls are different - so some details apply.

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

