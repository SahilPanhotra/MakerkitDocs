-----------------
FILE PATH: kits/next-supabase-turbo/api/team-account-api.mdoc

---
status: "published"
label: "Team Account API"
order: 1
title: "Team Account API | Next.js Supabase SaaS Kit"
description: "A quick introduction to the Team Account API in the Next.js Supabase SaaS Kit"
---

You can use the Team Account API for retrieving information about the team account.

{% sequence title="How to use the Team Account API" description="Learn how to use the Team Account API in Makerkit" %}

[Using the Team Account API](#using-the-team-account-api)

[Get the team account by ID](#get-the-team-account-by-id)

[Get the team account subscription](#get-the-team-account-subscription)

[Get the team account order](#get-the-team-account-order)

[Get the account workspace data](#get-the-account-workspace-data)

[Checking a user's permission within an account](#checking-a-user-s-permission-within-an-account)

[Getting the members count in the account](#getting-the-members-count-in-the-account)

[Get the billing customer ID](#get-the-billing-customer-id)

[Retrieve an invitation](#retrieve-an-invitation)

{% /sequence %}

## Using the Team Account API

To use the Team Account API, you need to import the `createTeamAccountsApi` function from `@kit/team-account/api`.

We need to pass a valid `SupabaseClient` to the function - so we can interact with the database from the server.

```tsx
import { createTeamAccountsApi } from '@kit/team-accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const api = createTeamAccountsApi(client);

  // use api
}
```

If you're in a Server Action context, you'd use:

```tsx
'use server';

import { createTeamAccountsApi } from '@kit/team-accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function myServerAction() {
  const client = getSupabaseServerClient();
  const api = createTeamAccountsApi(client);

  // use api
}
```

## Methods

The Account API provides the following methods:

### Get the team account by ID

Retrieves the team account by ID using the `getTeamAccountById` method.

You can also use this method to check if the user is already in the account.

```tsx
const api = createTeamAccountsApi(client);
const account = await api.getTeamAccountById('account-id');
```

### Get the team account subscription

Get the subscription data for the account using the `getSubscription` method.

```tsx
const api = createTeamAccountsApi(client);
const subscription = await api.getSubscription('account-id');
```

### Get the team account order

Get the orders data for the given account using the `getOrder` method.

```tsx
const api = createTeamAccountsApi(client);
const order = await api.getOrder('account-id');
```

### Get the account workspace data

Get the account workspace data.

```tsx
const api = createTeamAccountsApi(client);
const workspace = await api.getAccountWorkspace('account-slug');
```

This method is already called in the account layout and is unlikely to be used in other contexts. This is used to hydrate the workspace data in the context.

Since it's already loaded, you can use the data from the context.

### Checking a user's permission within an account

Check if the user has permission to perform a specific action within the account using the `hasPermission` method.

```tsx
const api = createTeamAccountsApi(client);

const hasPermission = await api.hasPermission({
  accountId: 'account-id',
  userId: 'user-id',
  permission: 'billing.manage',
});
```

### Getting the members count in the account

Get the number of members in the account using the `getMembersCount` method.

```tsx
const api = createTeamAccountsApi(client);
const membersCount = await api.getMembersCount('account-id');
```

### Get the billing customer ID

Get the billing customer ID for the given account using the `getCustomerId` method.

```tsx
const api = createTeamAccountsApi(client);
const customerId = await api.getCustomerId('account-id');
```

### Retrieve an invitation

Get the invitation data from the invite token.

```tsx
const api = createTeamAccountsApi(client);
const invitation = await api.getInvitation(adminClient, 'invite-token');
```

This method is used to get the invitation data from the invite token. It's used when the user is not yet part of the account and needs to be invited. The `adminClient` is used to read the pending membership. The method returns the invitation data if it exists, otherwise `null`.


-----------------
FILE PATH: kits/next-supabase-turbo/api/user-workspace-api.mdoc

---
status: "published"
label: "User Workspace API"
order: 3
title: "User Workspace API"
description: "The user workspace API allows you to retrieve all the data related to the current user."
---

When within the layout `/home/(user)` - you have access to data fetched from the user workspace API. The data in this layout has most of the information you need around the currently selected account and the user.

{% sequence title="How to use the User Workspace API" description="The user workspace API allows you to retrieve all the data related to the current user." %}

[Accessing the Workspace Data in Server Components](#accessing-the-user-workspace-data-in-server-components)

[Accessing the Workspace Data in Client Components](#accessing-the-user-workspace-data-in-client-components)

{% /sequence %}

## Accessing the User Workspace Data in Server Components

To access the data, you can use the `loadUserWorkspace` loader function. This function is cached per-request, so you can call it multiple times without worrying about performance.

### Data is cached per-request

While multiple calls to this function are deduped within a single request, bear in mind that this request will be called when navigating to the page. If you only require a small subset of data, it would be best to make more granular requests.

### Example

```tsx
import { loadUserWorkspace } from '~/home/(user)/_lib/server/load-user-workspace';

export default async function SomeUserPage() {
  const data = await loadUserWorkspace();

  // use data
}
```

The data returned from the `loadUserWorkspace` function is an object with the following properties:

- `user`: The user object coming from Supabase Auth
- `workspace`: The account object of the user
- `accounts`: An array of all accounts the user is a member of

Here is an example of the data structure:

```tsx
import type { User } from '@supabase/supabase-js';

{
  workspace: {
    id: string | null;
    name: string | null;
    picture_url: string | null;
    public_data: Json | null;
    subscription_status: string | null;
  };

  user: User;

  accounts: Array<{
   id: string | null;
    name: string | null;
    picture_url: string | null;
    role: string | null;
    slug: string | null;
  }>;
}
```

The `workspace` object contains the following properties:
- `id`: The account ID
- `name`: The account name
- `picture_url`: The account picture URL
- `public_data`: The account public data
- `subscription_status`: The subscription status of the account. This can be 'active' | 'trialing' | 'past_due' | 'canceled' | 'unpaid' | 'incomplete' | 'incomplete_expired' | 'paused'.


## Accessing the User Workspace Data in Client Components

The data fetched from the user workspace API is available in the context. You can access this data using the `useUserWorkspace` hook.

```tsx
'use client';

import { useUserWorkspace } from '@kit/accounts/hooks/use-user-workspace';

export default function SomeComponent() {
  const { workspace, user, accounts } = useUserWorkspace();

  // use account, user, and accounts
}
```

The `useUserWorkspace` hook returns the same data structure as the `loadUserWorkspace` function.

NB: the hooks **is not to be used** is Server Components, only in Client Components. Additionally, this is only available in the pages under `/home/(user)` layout.


-----------------
FILE PATH: kits/next-supabase-turbo/api.mdoc

---
status: "published"
title: "The API services in Next.js Supabse Tubo"
label: "API"
order: 5
description: "Services API allows you to interact with the entities in the Makerkit application."
collapsible: true
collapsed: true
---

The API services in Next.js Supabase Tubo allow you to interact with the entities in the Makerkit application.

1. [**Account API**](account-api): Use this service to interact with the accounts in the application.
2. [**Team Account API**](team-account-api): Use this service to interact with the team accounts in the application.
3. [**Authentication API**](authentication-api): Use this service to interact with the authentication methods in the application.
4. [**User Workspace API**](user-workspace-api): Use this service to interact with the user workspace in the application.
5. [**Team Account Workspace API**](team-account-workspace-api): Use this service to interact with the team account workspace in the application.


-----------------
FILE PATH: kits/next-supabase-turbo/billing/billing-api.mdoc

---
status: "published"
label: "Billing API"
title: "The Billing service API allows you to communicate with the billing service in Makerkit"
order: 10
description: "Learn how to use the Billing service API in Makerkit to manage billing for your preferred payment gateway"
---

The Billing service API allows you to communicate with the billing service in Makerkit. You can use the API to manage billing for your preferred payment gateway.

{% sequence title="Steps to use the Billing service API" description="Learn how to use the Billing service API in Makerkit to manage billing for your preferred payment gateway." %}

[Instantiating the billing service](#instantiating-the-billing-service)

[Creating a checkout session](#creating-a-checkout-session)

[Retrieving a checkout session](#retrieving-a-checkout-session)

[Creating a billing portal session](#creating-a-billing-portal-session)

[Cancelling a subscription](#cancelling-a-subscription)

[Reporting metered usage](#reporting-metered-usage)

[Querying metered usage](#querying-metered-usage)

[Updating a subscription](#updating-a-subscription)


{% /sequence %}

## Instantiating the billing service

To instantiate the billing service, you need to import the `createBillingGatewayService` function from the `@kit/billing-gateway` package. You can then use this service to manage subscriptions, one-off payments, and more.

```tsx
import { createBillingGatewayService } from '@kit/billing-gateway';

const service = createBillingGatewayService('stripe');
```

Of course - it's best to get the billing provider from the `subscriptions` record. This way, you can switch between providers without changing the code.

The `createBillingGatewayService` instantiates a class that provides a set of methods for managing billing operations. The provider is determined by the `BillingProviderSchema` passed to the constructor. The `BillingGatewayService` class is a TypeScript class that provides a set of methods for managing billing operations. 

It uses the Strategy design pattern to delegate the actual implementation of these operations to a provider-specific strategy. The provider is determined by the BillingProviderSchema passed to the constructor.

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
FILE PATH: kits/next-supabase-turbo/billing/billing-schema.mdoc

---
status: "published"
label: "Billing Schema"
title: "Writing the billing schema in the Next.js Supabase Kit"
order: 1
description: "Learn how to configure the plans in your Makerkit application"
---

{% sequence title="Steps to write the billing schema" description="Learn how to configure the plans in your Makerkit application" %}

[Understanding the billing schema](#understanding-the-billing-schema)

[Getting the schema right is important](#getting-the-schema-right-is-important)

[Setting the billing provider](#setting-the-billing-provider)

[Products](#products)

[Plans](#plans)

[Line items](#line-items)

[Custom plans](#custom-plans)

[Metered billing](#metered-billing)

[Per-seat billing](#per-seat-billing)

[One-off payments](#one-off-payments)

[Custom Plans](#custom-plans)

{% /sequence %}

## Understanding the billing schema

The billing schema replicates your billing provider's schema, so that:

1. we can display the data in the UI (pricing table, billing section, etc.)
2. create the correct checkout session
3. make some features work correctly - such as per-seat billing

The billing schema is common to all billing providers. Some billing providers have some differences in what you can or cannot do. In these cases, the schema will try to validate and enforce the rules - but it's up to you to make sure the data is correct.

The schema is based on three main entities:

1. **Products**: The main product you are selling (e.g., "Pro Plan", "Starter Plan", etc.)
2. **Plans**: The pricing plan for the product (e.g., "Monthly", "Yearly", etc.)
3. **Line Items**: The line items for the plan (e.g., "flat subscription", "metered usage", "per seat", etc.)

#### Getting the schema right is important

Getting the IDs of your plans is extremely important - as these are used to:
1. create the correct checkout
2. populate the data in the DB

Please take it easy while you configure this, do one step at a time, and test it thoroughly.

#### Setting the Billing Provider

The billing provider is already set as `process.env.NEXT_PUBLIC_BILLING_PROVIDER` and defaults to `stripe`.

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

