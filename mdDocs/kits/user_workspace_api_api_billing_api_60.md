-----------------
FILE PATH: kits\react-router-supabase-turbo\api\user-workspace-api.mdoc

---
status: "published"
label: "User Workspace API"
order: 3
title: "User Workspace API | React Router Supabase Turbo"
description: "The user workspace API allows you to retrieve all the data related to the current user."
---

When within the layout `/home/(user)` - you have access to data fetched from the user workspace API.

The data in this layout has most of the information you need around the currently selected account and the user.

To access the data, you can use the `useRouterLoaderData` hook.

```tsx
import { useRouteLoaderData } from 'react-router';
import type { Route as UserWorkspaceRoute } from '~/types/app/routes/home/user/+types/layout';

export default function Layout() {
  const { workspace } = useRouteLoaderData(
    'routes/home/user/layout',
  ) as UserWorkspaceRoute.ComponentProps['loaderData'];

  // use workspace
}
```

This data is only available in the `/home/(user)` layout.


-----------------
FILE PATH: kits\react-router-supabase-turbo\api.mdoc

---
status: "published"
title: "The API services in React Router Supabse Tubo"
label: "API"
order: 5
description: "Services API allows you to interact with the entities in the Makerkit application."
collapsible: true
collapsed: true
---

The API services in React Router Supabase Tubo allow you to interact with the entities in the Makerkit application.

1. [**Account API**](account-api): Use this service to interact with the accounts in the application.
2. [**Team Account API**](team-account-api): Use this service to interact with the team accounts in the application.
3. [**Authentication API**](authentication-api): Use this service to interact with the authentication methods in the application.
4. [**User Workspace API**](user-workspace-api): Use this service to interact with the user workspace in the application.
5. [**Team Account Workspace API**](team-account-workspace-api): Use this service to interact with the team account workspace in the application.


-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\billing-api.mdoc

---
status: "published"

label: "Billing API"
title: "The Billing service API allows you to communicate with the billing service in Makerkit"
order: 10
description: "Learn how to use the Billing service API in Makerkit to manage billing for your preferred payment gateway"
---


The Billing service API allows you to communicate with the billing service in Makerkit. You can use the API to manage billing for your preferred payment gateway.

To instantiate the billing service, you need to import the `createBillingGatewayService` function from the `@kit/billing-gateway` package. You can then use this service to manage subscriptions, one-off payments, and more.

```tsx
import { createBillingGatewayService } from '@kit/billing-gateway';

const service = createBillingGatewayService('stripe');
```

Of course - it's best to get the billing provider from the `subscriptions` record. This way, you can switch between providers without changing the code.

The `createBillingGatewayService` instantiates a class that provides a set of methods for managing billing operations. The provider is determined by the `BillingProviderSchema` passed to the constructor. The `BillingGatewayService` class is a TypeScript class that provides a set of methods for managing billing operations. It uses the Strategy design pattern to delegate the actual implementation of these operations to a provider-specific strategy. The provider is determined by the BillingProviderSchema passed to the constructor.

The class provides methods for creating and retrieving checkout sessions, creating a billing portal session, cancelling a subscription, reporting and querying usage, and updating a subscription. Each of these methods accepts a parameter object that is validated and parsed using the corresponding Zod schema.

## Methods

### Creating a checkout session

The `createCheckoutSession` method creates a checkout session for billing. It takes an object of type CreateBillingCheckoutSchema as a parameter, which contains the necessary information for creating a checkout session.

Below is the Zod schema accepted by the `createCheckoutSession` method:

```tsx
returnUrl: z.string().url(),
accountId: z.string().uuid(),
plan: PlanSchema,
customerId: z.string().optional(),
customerEmail: z.string().email().optional(),
enableDiscountField: z.boolean().optional(),
variantQuantities: z.array(
  z.object({
    variantId: z.string().min(1),
    quantity: z.number(),
  }),
)
```

### Retrieving a checkout session

The `retrieveCheckoutSession` method retrieves the checkout session from the specified provider. It takes an object of type RetrieveCheckoutSessionSchema as a parameter, which contains the necessary information to retrieve the checkout session.

The method accepts the following Zod schema:

```tsx
sessionId: z.string(),
```

### Creating a billing portal session

The `createBillingPortalSession` method creates a billing portal session. It takes an object of type CreateBillingPortalSessionSchema as a parameter, which contains the necessary information to create a billing portal session.

```tsx
returnUrl: z.string().url(),
customerId: z.string().min(1),
```

### Cancelling a subscription

The `cancelSubscription` method cancels a subscription. It takes an object of type CancelSubscriptionParamsSchema as a parameter, which contains the necessary information to cancel a subscription.

The method accepts the Zod schema:

```tsx
subscriptionId: z.string(),
invoiceNow: z.boolean().optional(),
```

### Reporting metered usage

The `reportUsage` method reports the usage of the billing to the provider. It takes an object of type ReportBillingUsageSchema as a parameter, which contains the necessary information to report the usage.

The method accepts the following Zod schema:

```tsx
id: z.string({
  description:
    'The id of the usage record. For Stripe a customer ID, for LS a subscription item ID.',
}),
eventName: z
  .string({
    description: 'The name of the event that triggered the usage',
  })
  .optional(),
usage: z.object({
  quantity: z.number(),
  action: z.enum(['increment', 'set']).optional(),
}),
```

### Querying metered usage

The `queryUsage` method queries the usage of the metered billing. It takes an object of type QueryBillingUsageSchema as a parameter, which contains the necessary information to query the usage.

To query usage, please provide the following Zod schema:

```tsx

const TimeFilter = z.object(
  {
    startTime: z.number(),
    endTime: z.number(),
  },
  {
    description: `The time range to filter the usage records. Used for Stripe`,
  },
);

const PageFilter = z.object(
  {
    page: z.number(),
    size: z.number(),
  },
  {
    description: `The page and size to filter the usage records. Used for LS`,
  },
);

export const QueryBillingUsageSchema = z.object({
  id: z.string({
    description:
      'The id of the usage record. For Stripe a meter ID, for LS a subscription item ID.',
  }),
  customerId: z.string({
    description: 'The id of the customer in the billing system',
  }),
  filter: z.union([TimeFilter, PageFilter]),
});

```

### Updating a subscription

The `updateSubscriptionItem` method updates a subscription with the specified parameters. It takes an object of type UpdateSubscriptionParamsSchema as a parameter, which contains the necessary information to update a subscription.

The method acceps the following Zod schema:

```tsx
subscriptionId: z.string().min(1),
subscriptionItemId: z.string().min(1),
quantity: z.number().min(1),
```

-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\billing-schema.mdoc

---
status: "published"
label: "Billing Schema"
title: "Writing the billing schema in the React Router Supabase Kit"
order: 1
description: "Learn how to configure the plans in your Makerkit application"
---

The billing schema replicates your billing provider's schema, so that:

1. we can display the data in the UI (pricing table, billing section, etc.)
2. create the correct checkout session
3. make some features work correctly - such as per-seat billing

The billing schema is common to all billing providers. Some billing providers have some differences in what you can or cannot do. In these cases, the schema will try to validate and enforce the rules - but it's up to you to make sure the data is correct.

The schema is based on three main entities:

1. **Products**: The main product you are selling (e.g., "Pro Plan", "Starter Plan", etc.)
2. **Plans**: The pricing plan for the product (e.g., "Monthly", "Yearly", etc.)
3. **Line Items**: The line items for the plan (e.g., "flat subscription", "metered usage", "per seat", etc.)

#### Setting the Billing Provider

The billing provider is already set as `process.env.VITE_BILLING_PROVIDER` and defaults to `stripe`.

For clarity - this is set in the `apps/web/config/billing.config.ts` file:

```tsx
export default createBillingSchema({
  // also update config.billing_provider in the DB to match the selected
  provider,
  // products configuration
  products: []
});
```

We will now add the products to the configuration.

#### Products

Products are the main product you are selling. They are defined by the following fields:

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
      plans: [],
    }
  ]
});
```

Let's break down the fields:

1. **id**: The unique identifier for the product. **This is chosen by you, it doesn't need to be the same one as the one in the provider**.
2. **name**: The name of the product
3. **description**: The description of the product
4. **currency**: The currency of the product
5. **badge**: A badge to display on the product (e.g., "Value", "Popular", etc.)
6. **enableDiscountField**: Whether to enable discount fields in the UI

The majority of these fields are going to populate the pricing table in the UI.

#### Plans

Plans are the pricing plans for the product. They are defined by the following fields:

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
          lineItems: [],
        }
      ],
    }
  ]
});
```

Let's break down the fields:
- **name**: The name of the plan
- **id**: The unique identifier for the plan. **This is chosen by you, it doesn't need to be the same one as the one in the provider**.
- **trialDays**: The number of days for the trial period
- **paymentType**: The payment type (e.g., `recurring`, `one-time`)
- **interval**: The interval of the payment (e.g., `month`, `year`)
- **lineItems**: The line items for the plan

Now, we will be looking at the line items. The line items are the items that make up the plan, and can be of different types:
1. **Flat Subscription**: A flat subscription (e.g., $10/month) - specified as `flat`
2. **Metered Billing**: Metered billing (e.g., $0.10 per 1,000 requests) - specified as `metered`
3. **Per-Seat Billing**: Per-seat billing (e.g., $10 per seat) - specified as `per-seat`

You can add one or more line items to the plan when using Stripe. When using Lemon Squeezy, you can only add one line item - but you can decorate it with the necessary metadata to achieve a similar result.

#### Flat Subscriptions

Flat subscriptions are defined by the following fields:

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
- **id**: The unique identifier for the line item. **This must match the price ID in the billing provider**. The schema will validate this, but please remember to set it correctly.
- **name**: The name of the line item
- **cost**: The cost of the line item
- **type**: The type of the line item (e.g., `flat`, `metered`, `per-seat`). In this case, it's `flat`.

The cost is set for UI purposes. **The billing provider will handle the actual billing** - therefore, **please make sure the cost is correctly set in the billing provider**.

#### Metered Billing

Metered billing is defined by the following fields:

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

Let's break down the fields:
- **id**: The unique identifier for the line item. **This must match the price ID in the billing provider**. The schema will validate this, but please remember to set it correctly.
- **name**: The name of the line item
- **cost**: The cost of the line item. This can be set to `0` as the cost is calculated based on the tiers.
- **type**: The type of the line item (e.g., `flat`, `metered`, `per-seat`). In this case, it's `metered`.
- **unit**: The unit of the line item (e.g., `GBs`, `requests`, etc.). You can use a translation key here.
- **tiers**: The tiers of the line item. Each tier is defined by the following fields:
- **upTo**: The upper limit of the tier. If the usage is below this limit, the cost is calculated based on this tier.
- **cost**: The cost of the tier. This is the cost per unit.

The tiers data is used exclusively for UI purposes. **The billing provider will handle the actual billing** - therefore, **please make sure the tiers are correctly set in the billing provider**.

#### Per-Seat Billing

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

#### One-Off Payments

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

### Adding more Products, Plans, and Line Items

Simply add more products, plans, and line items to the arrays. The UI **should** be able to handle it in most traditional cases. If you have a more complex billing schema, you may need to adjust the UI accordingly.

## Custom Plans

Sometimes - you want to display a plan in the pricing table - but not actually have it in the billing provider. This is common for custom plans, free plans that don't require the billing provider subscription, or plans that are not yet available.

To do so, let's add the `custom` flag to the plan:

```tsx
{
  name: 'Enterprise Monthly',
  id: 'enterprise-monthly',
  paymentType: 'recurring',
  label: 'common:contactUs',
  href: '/contact',
  custom: true,
  interval: 'month',
  lineItems: [],
}
```

Here's the full example:

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
          name: 'Enterprise',
          id: 'enterprise',
          paymentType: 'recurring',
          label: 'common:contactUs',
          href: '/contact',
          custom: true,
          interval: 'month',
          lineItems: [],
        }
      ],
    }
  ]
});
```

As you can see, the plan is now a custom plan. The UI will display the plan in the pricing table, but it won't be available for purchase.

We do this by adding the following fields:

- **custom**: A flag to indicate that the plan is custom. This will prevent the plan from being available for purchase.
- **label**: The translation key for the label. This is used to display the label in the pricing table.
- **href**: The link to the page where the user can contact you. This is used in the pricing table.
- **lineItems**: The line items for the plan. This is empty as there are no line items for the plan. It must be an empty array.

### Custom Button Label
You can also provide a custom button label for the plan. This is done by adding the `buttonLabel` field:

```tsx
{
  name: 'Enterprise',
  id: 'enterprise',
  paymentType: 'recurring',
  label: 'common:contactUs',
  href: '/contact',
  custom: true,
  interval: 'month',
  lineItems: [],
  buttonLabel: 'common:contactUs',
}
```

As usual, strings can either be a translation key or a string. If it's a translation key, it will be translated using the `i18n` library.

-----------------
FILE PATH: kits\react-router-supabase-turbo\billing\billing-webhooks.mdoc

---
status: "published"
label: "Handling Webhooks"
title: "Learn how to handle billing webhooks with your custom code in React Router Supabase kit"
order: 9
description: "Learn how to handle billing webhooks with your custom code in React Router Supabase kit"
---

Makerkit takes care of handling billing webhooks to update the Database based on the events received from Stripe.

Sometimes - you will need to set more webhooks, or do something custom with the webhooks.

In these cases, you can customize the billing webhook handler in Makerkit at `apps/web/app/routes/api/billing/webhook.ts`.

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
FILE PATH: kits\react-router-supabase-turbo\billing\credit-based-billing.mdoc

---
status: "published"
label: "Credits Based Billing"
title: "How to add credit seat billing to Makerkit | React Router Supabase"
order: 7
description: "Learn how to configure credit based usage in your React Router Supabase application"
---

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

NB: we assume this is in a route handler, so we use the `getSupabaseRouteHandlerClient` client. If you're in a server action, you should use the `getSupabaseServerActionClient` client. [Learn more about the Supabase clients here](supabase-clients).

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

In the example below, we extend the webook handler to listen to the `onInvoicePaid` event and update the `credits` table with the new credits.

1. First, update the existing webhook handler to listen to the `onInvoicePaid` event.
2. Then, update the `credits` table with the new credits.

 ```tsx {% title="apps/web/app/routes/api/billing/webhook.ts" %}
import { ActionFunctionArgs } from 'react-router';
import { getBillingEventHandlerService } from '@kit/billing-gateway';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

import billingConfig from '~/config/billing.config';
import { Database } from '~/lib/database.types';

/**
 * @description Handle the webhooks from Stripe related to checkouts
 */
export const action = async ({ request }: ActionFunctionArgs) => {
    const provider = billingConfig.provider;
    const logger = await getLogger();

    const ctx = {
      name: 'billing.webhook',
      provider,
    };

    logger.info(ctx, `Received billing webhook. Processing...`);

    const supabaseClientProvider = () =>
      getSupabaseServerAdminClient<Database>();

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
};

async function updateMessagesCountQuota(params: {
  variantId: string;
  accountId: string;
}) {
  const client = getSupabaseServerAdminClient();

  // get the max messages for the price based on the price ID
  const plan = await client
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
  const response = await client
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

