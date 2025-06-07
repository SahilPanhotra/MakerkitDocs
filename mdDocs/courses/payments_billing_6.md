-----------------
FILE PATH: courses/nextjs-turbo/payments-billing.mdoc

---
title: "Payments and Billing in Makerkit Next.js Supabase Turbo"
description: "Learn how to set up payments and billing in your Makerkit Next.js Supabase Turbo project."
label: "Payments and Billing"
order: 7
---

Hi there! 👋 Welcome to this module on **setting up payments and billing** in your Makerkit Next.js Supabase Turbo project.

## Introduction to Billing in Makerkit

Makerkit provides a flexible billing system that allows you to define your pricing structure using a billing schema. This schema is designed to work with multiple payment providers, not just Stripe.

### Overview of Makerkit's billing capabilities

Here's a high-level overview of Makerkit's billing capabilities:

1. **Billing Schema**: Makerkit uses a billing schema to define products, plans, and pricing. This schema is provider-agnostic, meaning it can work with different payment processors.
2. **Multiple Payment Providers**: While Stripe is supported, Makerkit also supports other providers like Lemon Squeezy, with plans to support more in the future.
3. **Flexible Pricing Models**: The schema supports various pricing models including flat-rate, per-seat, and metered billing.
4. **Product and Plan Definition**: You can define multiple products, each with its own set of plans and pricing details.

### Billing Gateway

Instead of direct integration with a single provider, Makerkit uses a Billing Gateway approach:

1. **Provider Abstraction**: The Billing Gateway abstracts the implementation details of different payment providers.
2. **Easy Switching**: This design allows you to switch between payment providers with minimal code changes.
3. **Unified API**: Regardless of the provider you choose, you interact with billing functionality through a consistent API.

Here's a basic example of how you might define a billing schema in Makerkit:

```typescript
import { createBillingSchema } from '@kit/billing';

export default createBillingSchema({
  provider: process.env.NEXT_PUBLIC_BILLING_PROVIDER,
  products: [
    {
      id: 'your-product',
      name: 'Your Product Name',
      description: 'Description of your product',
      plans: [
        {
          id: 'basic-plan',
          name: 'Basic Plan',
          lineItems: [
            {
              id: 'price_id_from_provider',
              type: 'flat',
              cost: 9.99,
            }
          ],
        },
        // More plans...
      ],
    },
  ],
});
```

This schema defines the structure of your pricing, which Makerkit then uses to interact with your chosen payment provider.

In the following sections, we'll dive deeper into how to construct this schema for your specific needs and how to leverage Makerkit's billing capabilities in your application.

Certainly! Let's dive into understanding billing schemas in Makerkit.

## Understanding the Makerkit Billing Schema

The Makerkit Billing schema is a unified interface for defining your pricing structure. Whether you're using Stripe, Lemon Squeezy, or another payment provider, the billing schema provides a consistent way to define your products, plans, and pricing models.

The schema is validated with Zod to ensure that it's correctly defined and adheres to the expected structure. This helps prevent errors and ensures that your billing configuration is accurate.

{% img src="/assets/courses/next-turbo/billing-schema.webp" alt="Billing Schema" width="916" height="595" /%}

### How Makerkit uses the billing schema

What we define in the billing schema defines what we display in the UI. This schema is used to generate the pricing table and the plan picker (in the internal part of the application).

However, it's worth mentioning that **the actual logic of how the billing works is handled in your provider's configuration** (e.g., Stripe dashboard).

For example, if you define a metered billing item in your schema, Makerkit will display this as a metered item in the UI. But the actual metering logic (e.g., tracking usage) will be handled by your payment provider. Makerkit exposes useful APIs to help you interact with the billing schema and manage subscriptions, but the core billing logic is handled by the provider.

#### Exceptions to the rule - Per Seat Billing

The billing schema will replicate the structure of your pricing in the UI, but the actual billing logic will be handled by the payment provider.

One exception is the per-seat billing items: when setting an item as `per_seat` it will automatically update the quantity of the item based on the number of users invited to a team account.

### What is a billing schema?

A billing schema in Makerkit is a configuration object that describes your pricing structure. It's designed to be:

1. **Provider-agnostic**: Works with multiple payment providers (like Stripe or Lemon Squeezy)
2. **Flexible**: Supports various pricing models
3. **Declarative**: Clearly defines your product and plan structure

This allows us to abstract the billing logic from the payment provider, making it easier to switch providers or adjust pricing without changing the core logic of your application.

Imagine one day you decide to switch from Stripe to Lemon Squeezy. With a well-defined billing schema, **you can make this change with minimal effort, as the schema remains the same regardless of the provider** (or minimal changes).

### Components of a billing schema in Makerkit

A billing schema typically consists of the following components:

1. **Provider**: Specifies which payment provider you're using.
2. **Products**: An array of products you're offering (Basic, Pro, Free, etc.)
3. **Plans**: Define the different pricing tiers for each product (Monthly, Yearly)
4. **Line Items**: The specific charges within a plan (Base price, Per-seat cost, etc.)

Here's a basic structure of a billing schema:

```typescript
import { createBillingSchema } from '@kit/billing';

export default createBillingSchema({
  provider: 'stripe', // or 'lemon-squeezy', etc.
  products: [
    {
      id: 'product-id',
      name: 'Product Name',
      description: 'Product Description',
      currency: 'USD',
      features: ['Feature 1', 'Feature 2'],
      plans: [
        {
          id: 'plan-id',
          name: 'Plan Name',
          paymentType: 'recurring',
          lineItems: [
            {
              id: 'line-item-id',
              name: 'Line Item Name',
              type: 'flat', // or 'per_seat', 'metered', etc.
              cost: 9.99,
            },
          ],
        },
      ],
    },
  ],
});
```

Let's break down these components:

- **Provider**: This tells Makerkit which payment provider to use. It's typically set from an environment variable.
- **Products**: These are the high-level offerings in your SaaS. For example, you might have a "Basic Support" product and a "Premium Support" product.
- **Plans**: Each product can have multiple plans. For example: "Monthly", "Yearly"
- **Line Items**: These are the individual charges within a plan. They can be flat fees, per-seat charges, or usage-based (metered) charges.

The beauty of this schema is that it allows you to define complex pricing structures in a clear, readable way. Whether you're doing simple flat-rate pricing or complex tiered pricing with multiple charges, you can express it all in this schema.

In the next sections, we'll dive deeper into the different types of billing models you can implement with this schema, and how to create a schema for our Support Tickets SaaS example.

## Creating Our Support Tickets SaaS Billing Schema

Now that we understand what a billing schema is, let's create one for our Support Tickets SaaS.

We'll implement a two-tier pricing model with per-seat billing and ticket limits.

### A. Defining our pricing strategy

For our Support Tickets SaaS, we'll create two plans:

1. **Free Plan**:
    - Up to 50 tickets
    - No base price
    - $0/month for each additional seat
2. **Starter Plan**:
    - Up to 1000 tickets
    - $49/month base price
    - $10/month for each additional seat
3. **Pro Plan**:
    - Unlimited tickets
    - $199/month base price
    - $10/month for each additional seat

### B. Implementing the schema in code

Let's translate this pricing strategy into a Makerkit billing schema:

```typescript {% title="apps/web/config/billing.sample.config.ts" %}
import { BillingProviderSchema, createBillingSchema } from '@kit/billing';

// The billing provider to use. This should be set in the environment variables
// and should match the provider in the database. We also add it here so we can validate
// your configuration against the selected provider at build time.
const provider = BillingProviderSchema.parse(
  process.env.NEXT_PUBLIC_BILLING_PROVIDER,
);

export default createBillingSchema({
  // also update config.billing_provider in the DB to match the selected
  provider,
  // products configuration
  products: [
    {
      id: 'support-tickets',
      name: 'Support Tickets',
      badge: 'Free',
      description: 'Manage your customer support tickets efficiently',
      currency: 'USD',
      features: ['Up to 50 tickets per month', '$0 per agent', 'Email support'],
      plans: [
        {
          id: 'free-plan',
          name: 'Free Plan',
          lineItems: [],
          custom: true,
          label: 'Free',
          buttonLabel: 'Get started with the free plan',
          paymentType: 'recurring',
          interval: 'month',
        },
        {
          id: 'free-plan-yearly',
          name: 'Free Plan',
          lineItems: [],
          custom: true,
          label: 'Free',
          buttonLabel: 'Get started with the free plan',
          paymentType: 'recurring',
          interval: 'year',
        },
      ],
    },
    {
      id: 'starter-plan',
      name: 'Starter Plan',
      badge: 'Popular',
      highlighted: true,
      description: 'The best plan for small teams',
      currency: 'USD',
      features: [
        'Up to 1000 tickets per month',
        'Email support',
        'Up to 5 agents',
      ],
      plans: [
        {
          id: 'starter-plan-monthly',
          name: 'Starter Plan Monthly',
          paymentType: 'recurring',
          interval: 'month',
          lineItems: [
            {
              id: 'starter-base-price',
              name: 'Base Price',
              type: 'flat',
              cost: 49,
            },
            {
              id: 'starter-per-seat',
              name: 'Per Seat',
              type: 'per_seat',
              cost: 10,
            },
          ],
        },
        {
          id: 'starter-plan-yearly',
          name: 'Starter Plan Yearly',
          paymentType: 'recurring',
          interval: 'year',
          lineItems: [
            {
              id: 'starter-base-yearly',
              name: 'Base Price',
              type: 'flat',
              cost: 490,
            },
            {
              id: 'starter-per-seat-yearly',
              name: 'Per Seat',
              type: 'per_seat',
              cost: 100,
            },
          ],
        },
      ],
    },
    {
      id: 'pro-plan',
      name: 'Pro Plan',
      description: 'The best plan for growing teams',
      currency: 'USD',
      features: [
        'Unlimited tickets per month',
        'Priority support',
        'Email and phone support',
      ],
      plans: [
        {
          id: 'pro-plan-monthly',
          name: 'Pro Plan Monthly',
          paymentType: 'recurring',
          interval: 'month',
          lineItems: [
            {
              id: 'pro-base-price-monthly',
              name: 'Base Price',
              type: 'flat',
              cost: 199,
            },
            {
              id: 'pro-per-seat-monthly',
              name: 'Per Seat',
              type: 'per_seat',
              cost: 10,
            },
          ],
        },
        {
          id: 'pro-plan-yearly',
          name: 'Pro Plan Yearly',
          paymentType: 'recurring',
          interval: 'year',
          lineItems: [
            {
              id: 'pro-base-price-yearly',
              name: 'Base Price',
              type: 'flat',
              cost: 1990,
            },
            {
              id: 'pro-per-seat-yearly',
              name: 'Per Seat',
              type: 'per_seat',
              cost: 100,
            },
          ],
        },
      ],
    },
  ],
});
```

#### Use real Line Item IDs!

NB: we added "fake" Line Item IDs for the plans. **These IDs should correspond to the actual price IDs in your payment provider's dashboard**.

For example, if you stripe, these are associated with the Price ID in your Stripe dashboard. These would have the format `price_12345` in Stripe.

Below we will see how you can use environment variables to set the actual IDs in your application.

### C. Result of the schema

Let's take a look at how the UI will look like based on this schema.

We have:
1. the **Pricing table**, displayed externally (e.g., on the marketing site)
2. the **Plan Picker**, displayed internally (e.g., in the application)

The Pricing table shows the different products and plans available, along with their features and pricing details. The Plan Picker allows users to select a plan and enter their payment information.

{% img src="/assets/courses/next-turbo/pricing-table.webp" alt="Pricing Table" width="2592" height="1502" /%}

And below we have the Plan Picker, where customers can pick a plan from within the application:

{% img src="/assets/courses/next-turbo/plan-picker.webp" alt="Plan Picker" width="2138" height="1232" /%}

As you can see custom plans aren't displayed in the plan picker since these are supposed to be created by the user or simply used as a placeholder for the free plan.

#### Replacing Variant IDs with Environment Variables

A good practice I found useful for managing the different variant IDs is to use environment variables.

This way, you can easily switch between different environments (e.g., development, staging, production) without changing the code.

```typescript
const starterPlanMonthlyVariantId = process.env.NEXT_PUBLIC_STARTER_PLAN_MONTHLY_VARIANT_ID!;

const starterPlanYearlyVariantId = process.env.NEXT_PUBLIC_STARTER_PLAN_YEARLY_VARIANT_ID!;

const proPlanMonthlyVariantId = process.env.NEXT_PUBLIC_PRO_PLAN_MONTHLY_VARIANT_ID!;

const proPlanYearlyVariantId = process.env.NEXT_PUBLIC_PRO_PLAN_YEARLY_VARIANT_ID!;
```

Or, even better, can also use Zod to validate these environment variables and ensure they're set correctly.

```typescript
import { z } from 'zod';

const VariantsSchema = z.object({
  NEXT_PUBLIC_STARTER_PLAN_MONTHLY_VARIANT_ID: z.string().min(1),
  NEXT_PUBLIC_STARTER_PLAN_YEARLY_VARIANT_ID: z.string().min(1),
  NEXT_PUBLIC_PRO_PLAN_MONTHLY_VARIANT_ID: z.string().min(1),
  NEXT_PUBLIC_PRO_PLAN_YEARLY_VARIANT_ID: z.string().min(1),
  NEXT_PUBLIC_PER_SEAT_PLAN_MONTHLY_VARIANT_ID: z.string().min(1),
});

const variants = VariantsSchema.parse(
  process.env,
);
```

This way, you can easily manage your variant IDs across different environments and ensure they're set correctly.

You can now replace the hardcoded variant IDs in your billing schema with these environment variables.

```tsx
lineItems: [
  {
    id: variants.NEXT_PUBLIC_STARTER_PLAN_MONTHLY_VARIANT_ID,
    name: 'Base Price',
    type: 'flat',
    cost: 49,
  },
  {
    id: variants.NEXT_PUBLIC_PER_SEAT_PLAN_MONTHLY_VARIANT_ID,
    name: 'Per Seat',
    type: 'per_seat',
    cost: 10,
  },
]
```

This makes it easier to manage your billing schema and ensures that your variant IDs are set correctly across different environments.

#### Line Item IDs must match to the actual price IDs in your payment provider's dashboard

I will reiterate that **these IDs should correspond to the actual price IDs in your payment provider's dashboard**. For example, if you're using Stripe, these would be the Price IDs in your Stripe dashboard.

#### Differences between Billing Providers

It's important to note that different billing providers have different capabilities and limitations. For example, Stripe supports multiple line items per plan, while other providers may not. When defining your billing schema, you'll need to consider the capabilities of your chosen provider.

NB: **Only Stripe supports multiple line items per plan**. If you're using a different provider, you may need to adjust the schema to fit the provider's capabilities.

### Walkthrough of the schema structure

Let's break down the key parts of this schema:

1. **Provider**: We're using an environment variable to set the billing provider, making it easy to switch between providers if needed.
2. **Product**: We have one product, "Support Tickets", which encompasses our entire SaaS offering.
3. **Plans**: We've defined two plans, "Starter Plan" and "Pro Plan".
4. **Line Items**: Each plan has two line items:
- A 'flat' type for the base price
- A 'per_seat' type for the additional seat cost

This schema gives us a flexible structure that can easily be adjusted as our pricing evolves. For example, if we decide to change prices or add a new plan, we just need to update this schema.

Remember, this schema is largely **provider-agnostic**. Whether we're using Stripe, Lemon Squeezy, or another provider, this schema will work the same way. The actual IDs for products and prices will need to be set up in your chosen payment provider's dashboard.

In the next section, we'll dive deeper into how per-seat billing works and how to implement it in your application logic.

### Free Plan

The Free Plan gets added as a custom plan with no line items. This plan is ideal for small teams and comes with a $0/month base price. This plan allows customers to get started with the product without any upfront costs and without going through the payment flow.

However, there are situations where you'd want to collect payment information upfront, even for free plans, for example, when you need monthly webhooks to be triggered.

If you were to require payment information upfront, you could add a line item for $0 to the Free Plan. This would allow you to collect payment information and set up the customer in your billing system, even though they wouldn't be charged. Of course, this needs be an existing plan in your payment provider's dashboard.

## Getting Started with Stripe

If you don't yet have a Stripe account, please create one [here](https://stripe.com/).

### Create Stripe Products

To test our products, you need to create the products in your Stripe account. These products are what your users will subscribe to.

I will show you how to create one of the products, and you can create the rest of the products in the same way.

#### Flat-rate Subscription

If you remember, we created a flat-rate subscription in our project. This subscription is a fixed price subscription that your users can subscribe to.

Let's add the Starter Products to your Stripe account. To do this, follow these steps:

1. Navigate to the `Product Catalog` section in your Stripe account.
2. Click on the `Add product` button.
3. Fill in the `Name` with `Starter Plan`.
4. Set it as a `Recurring` product at a `Monthly` interval with a price of `49.00` USD. You can also add a Yearly interval if you want.

Once you have created your `Product` (Starter Plan) and your `Prices` (Monthly and Yearly), you can retrieve the Price IDs which we will define in our environment variables as we've seen in the previous section.

```bash {% title="apps/web/.env.production" %}
NEXT_PUBLIC_STARTER_PLAN_MONTHLY_VARIANT_ID=<your-monthly-price-id>
NEXT_PUBLIC_STARTER_PLAN_YEARLY_VARIANT_ID=<your-yearly-price-id>
```

#### Per Seat Subscription

Our application charges $10 per user per month. To set this up, you need to create a `Product` in your Stripe account.

To create a Per Seat Subscription, we can create a `Product` in our Stripe account.

To do this, follow these steps:

1. Navigate to the `Product Catalog` section in your Stripe account.
2. Click on the `Add product` button.
3. Fill in the `Name` with `Per Seat Plan`.
4. Set it as a `Recurring` product at a `Monthly` interval with a price of `10.00` USD.

Makerkit will handle the quantity of the subscription. This means that the quantity of the subscription will be increased or decreased based on the number of users in the team.

- **When each user gets added to a Team Account**, Makerkit will increase the quantity of the subscription by 1. This way, you can charge $10 per user per month.
- **When a user gets removed from a plan**, Makerkit will decrease the quantity of the subscription by 1. When a team subscribes to a Pro Plan (say with 5 users), the quantity will be set to 5 and they will be charged $50 per month.

Once you have created your `Product` (Per Seat Plan) and your `Prices` (Monthly and Yearly), you can retrieve the Price IDs which we will define in our environment variables as we've seen in the previous section.

```bash {% title="apps/web/.env.production" %}
NEXT_PUBLIC_PER_SEAT_PLAN_MONTHLY_VARIANT_ID=<your-monthly-price-id>
NEXT_PUBLIC_PER_SEAT_PLAN_YEARLY_VARIANT_ID=<your-yearly-price-id>
```

You can set these environment variables in your `.env` files depending on which environment you are deploying to.

### Starting the Stripe CLI

The Stripe CLI allows us to develop and test our Stripe integration locally. Makerkit provides a command to start the Stripe CLI using Docker:

```bash
pnpm run stripe:listen
```

The first time you run this command - Stripe will ask you to authenticate. Follow the instructions in the terminal to authenticate your Stripe account.

Now, run the command again to start the Stripe CLI. This will start the Stripe CLI in listen mode, which will listen for events from your Stripe account.

```bash
pnpm run stripe:listen
```

Once started, the CLI will output a webhook secret that you can use to set up webhooks in your Stripe account.

Please add this to your `.env.development` or `.env.local` file:

```bash {% title=".env.development" %}
STRIPE_WEBHOOK_SECRET=<your-webhook-secret>
```

When we deploy to production in the next step - we will replace this with the production webhook secret.

Now that the Stripe CLI is up and running, we can receive webhooks to our local server at `http://localhost:3000`: This will allow us to test our Stripe integration locally.

## Enforcing Feature Limits based on Plan Selection

In our Support Tickets SaaS, we have different plans with varying feature limits. For example, the Free Plan allows up to 50 tickets per month, while the Starter Plan allows up to 1000 tickets per month. We need to enforce these feature limits based on the plan selected by the user.

One useful pattern for storing feature limits is to **define a table in your database that maps each feature to its limit for each plan**. This table can then be queried when checking if a user has exceeded their feature limits.

### A. Defining the feature limits table

Let's define a feature limits table in our database that maps each feature to its limit for each plan. We will add this table to the existing migrations in our project.

```sql {% title="migrations/<timestamp>_support-schema.sql" %}
-- public.plans: table for subscription plans
create table if not exists public.plans (
  variant_id varchar(255) primary key,
  name varchar(255) not null,
  max_tickets int not null
);

-- revoke all permissions from the plans table
revoke all on public.plans from public, service_role;

-- grant permissions to the authenticated role
grant select on public.plans to authenticated, service_role;

-- RLS
alter table public.plans enable row level security;

-- SELECT(public.plans)
create policy select_plans
  on public.plans
  for select
  to authenticated
  using (true);
```

For unlimited tickets, you can set the `max_tickets` to `-1`. This way, you can easily check if a plan has unlimited tickets by comparing the `max_tickets` to `-1`.

You can copy/paste this SQL code into your Supabase Studio SQL runner so that's applied to your database without needing a rest.

Now, let's also re-generate the types for our database schema:

```bash
pnpm run supabase:web:typegen
```

This will generate the types for the `plans` table in our database.

#### Adding Plans to the Database

Now that we have the `plans` table in our database, we can add the plans for our Support Tickets SaaS. We'll add the Free Plan and the Starter Plan with their respective feature limits.

```sql {% title="apps/web/supabase/seed.sql" %}
-- insert plans
insert into public.plans (variant_id, name, max_tickets)
values
  ('starter-plan', 'Starter Plan', 1000),
  ('pro-plan', 'Pro Plan', -1);
```

You can copy/paste this SQL code into your Supabase Studio SQL runner to add the plans to your database.

NB: the values 'starter-plan' and 'pro-plan' should correspond to the actual price IDs in your payment provider's dashboard. In development, these correspond to the ones you use in the Test mode.

Because we only need this for development, we add it to the `seed.sql` file. This seed file is only run in development and is used to populate the database with initial data.

When you go to production, you can add the plans directly from Supabase Studio using the real Price IDs from your payment provider's dashboard.

#### Table Structure

Let's explain the structure of this table:

- **variant_id**: The unique identifier for the plan variant. This needs to correspond to the actual price ID in your payment provider's dashboard.
- **name**: The name of the plan variant - so we can easily identify it.
- **max_tickets**: The maximum number of tickets allowed for this plan variant.

#### Permissions and RLS Policy

We also revoked all permissions on this table from the public and service_role roles, and granted select permissions to the authenticated role. This ensures that only authenticated users can access this table.

The RLS policy `select_plans` allows authenticated users to select data from this table, so they can check the feature limits for the plan they're on.

### B. How do we enforce feature limits?

To enforce limits we will need to:

1. **Check the feature limits for the plan the user is on**.
2. **Compare the feature limits to the user's usage**.
3. **Prevent the user from exceeding their limits**.

We can do this by checking the feature limits for the plan the user is on and comparing it to the number of tickets they've created. If the user has exceeded their limits, we can prevent them from creating more tickets until they upgrade their plan.

### C. Enforcing feature limits in the application

We decided that we will disallow replies past the limit, so we need to enforce this in our application logic. We can do this by checking the feature limits for the plan the user is on and comparing it to the number of tickets they've replied to.

The limit kicks in when a customer receives over the number of tickets allowed by their plan. For example, **if the customer is on the free plan, they can only keep replying to tickets until they reach the 50 tickets limit**. After that, they will not be able to reply to any more tickets until they upgrade their plan.

1. **Free Plan**: On a free plan, we can check the number of tickets created in the last 30 days and compare it to the limit of 50 tickets.
2. **Paid Plan**: On a paid plan, we can check the number of tickets during the billing period and compare it to the limit of the plan.

#### Per-Seat Billing

As for the number of agents added to a team, Makerkit will automatically handling the process of adding/removing a seat from the customer's line items. Stripe will then charge the customer accordingly.

### D. Enforcing feature limits at the DB level

We want to enforce feature limits at the DB level to ensure that users cannot bypass the application logic and exceed their limits. To do this, we create a database trigger that checks the feature limits when a new ticket is created or a reply is added to an existing ticket.

#### Getting Subscription Details

We can create a function that retrieves the subscription details for the user, including the plan variant, period start, and period end. This function can be used to check the feature limits for the user's plan.

More specifically, we want:

1. The variant ID of the plan - so we can look up the feature limits in the `plans` table
2. The period start and end dates of the subscription - so we can check the number of tickets created during the billing period

And we filter by the following conditions:

1. The account ID of the user
2. The type of the item (e.g., 'flat' for tickets), as the other plan item types are not relevant for this check

```sql {% title="migrations/<timestamp>_support-schema.sql" %}
create
or replace function public.get_subscription_details (target_account_id uuid) returns table (
  variant_id varchar,
  period_starts_at timestamptz,
  period_ends_at timestamptz
)
set search_path = ''
as $$
begin
  -- select the subscription details for the target account
  return query select
        item.variant_id,
        subscription.period_starts_at,
        subscription.period_ends_at
  from
        public.subscription_items as item
  join
        public.subscriptions as subscription
  on
        subscription.id = item.subscription_id
  where
        subscription.account_id = target_account_id
  and   subscription.active = true
  and
        item.type = 'flat';
end;
$$ language plpgsql;

grant execute on function public.get_subscription_details(uuid) to authenticated, service_role;
```

Thanks to this function, we can now retrieve the subscription details for the user, including the plan variant, period start, and period end. This information is crucial for checking the feature limits for the user's plan.

If the function doesn't return any results, it means the user is on a free plan - or they have an inactive subscription - such as an unpaid invoice.

In this case, we can check the number of tickets created in the last 30 days to enforce the feature limits.

#### Checking the Feature Limits on Ticket Creation

We can create a trigger that checks the feature limits when a new ticket is created. This trigger will fire before inserting a new ticket into the `tickets` table and raise an exception if the user has exceeded their limits.

This is to ensure that users are held to the limits of their plan and cannot bypass the application logic to exceed their limits.

```sql {% title="migrations/<timestamp>_support-schema.sql" %}
create
or replace function public.check_ticket_limit ()
returns trigger
set search_path = ''
as $$
declare
  subscription record;
  ticket_count int;
  max_tickets int;
begin
  -- get the subscription details for the account
  select *
    into subscription
    from public.get_subscription_details(NEW.account_id);

  -- is the user on a free plan?
  if subscription is null then
    select count(*)
      into ticket_count
      from public.tickets
      where account_id = NEW.account_id and
      created_at >= now() - interval '30 days';

    -- check if the user has exceeded the limit
    if ticket_count >= 50 then
      raise exception 'You have reached the maximum number of tickets allowed for your plan';
    end if;

    -- allow the user to create the ticket
    return NEW;
  end if;

  -- get the max tickets allowed for the plan
  select max_tickets
    into max_tickets
    from public.plans
    where variant_id = subscription.variant_id;

  -- Unlimited tickets for the plan, so allow the user to create the ticket
  if max_tickets = -1 then
    return NEW;
  end if;

  -- check the number of tickets created during the billing period
  select count(*)
    into ticket_count
    from public.tickets
    where account_id = NEW.account_id and
    created_at >= subscription.period_starts_at and
    created_at <= subscription.period_ends_at;

  if ticket_count >= max_tickets then
    raise exception 'You have reached the maximum number of tickets allowed for your plan';
  end if;

  return NEW;
end;
$$ language plpgsql;

create or replace trigger check_ticket_limit
before insert on public.tickets
for each row
execute function public.check_ticket_limit ();
```

This function `check_ticket_limit` checks the feature limits for the plan the user is on and raises an exception if the user has exceeded their limits. This trigger is fired before inserting a new ticket into the `tickets` table.

Let's break down the key parts of our function:

1. **Get Subscription Details**: This function `get_subscription_details` retrieves the subscription details for the user, including the plan variant, period start, and period end.
2. **Check Ticket Limit**: This function `check_ticket_limit` checks the feature limits for the plan the user is on and raises an exception if the user has exceeded their limits.
3. **Enforce Limits**: We check the number of tickets created in the last 30 days for free plans and during the billing period for paid plans. If the user has exceeded their limits, we raise an exception.
4. **Trigger**: This trigger `check_ticket_limit` is fired before inserting a new ticket into the `tickets` table.

By enforcing feature limits at the DB level, we ensure that users cannot bypass the application logic and exceed their limits. This provides an additional layer of security and ensures that users are held to the limits of their plan.

For completeness, **we should also enforce feature limits in the application logic to provide a better user experience**. This way, users are informed in real-time if they've exceeded their limits and can take action to upgrade their plan.

```sql
create or replace function public.get_remaining_tickets(account_id uuid)
returns int
set search_path = ''
as $$
declare
  subscription record;
  ticket_count int;
  max_tickets int;
begin
  select *
    into subscription
    from public.get_subscription_details(account_id);

  if subscription is null then
    select count(*)
      into ticket_count
      from public.tickets
      where account_id = account_id and
      created_at >= now() - interval '30 days';

    return 50 - ticket_count;
  end if;

  select max_tickets
    into max_tickets
    from public.plans
    where variant_id = subscription.variant_id;

  -- Unlimited tickets
  if max_tickets = -1 then
    return -1;
  end if;

  select count(*)
    into ticket_count
    from public.tickets
    where account_id = account_id and
    created_at >= subscription.period_starts_at and
    created_at <= subscription.period_ends_at;

  return max_tickets - ticket_count;
end;
$$ language plpgsql;

grant execute on function public.get_remaining_tickets(uuid) to authenticated, service_role;
```

If a plan has unlimited tickets, we can return `-1` to indicate that there is no limit. This function can be used to display the remaining tickets to the user in the UI.

To add these functions to your database, you can copy/paste the SQL code into your Supabase Studio SQL runner - or reset your database to apply the changes using the following command:

```bash
pnpm run supabase:db:reset
```

and running the types generation command:

```bash
pnpm run supabase:web:typegen
```

This will ensure that the functions are added to your database and the types are generated for the new functions.

## UX Considerations - Displaying Remaining Tickets

For UX purposes -** we want support agents to know when their ticket limit is about to be reached**. We can display the remaining tickets in the UI so that agents can see how many tickets they have left before they reach their limit.

Of course - this is gonna be a very common pattern in SaaS businesses, where you want customers to know when they're about to reach their limits, so they can subscribe to a higher plan, purchase more credits, or take any other action.

### Restricting Access When Limit is Reached

When it comes to restricting access when a customer has exceeded their plan, **you need to think how strict you want to be**.

For example, you might want to allow a customer to reply to a ticket even if they've exceeded their limit, but not create a new ticket. This way, they can still provide support to their customers even if they're over their limit - and send emails to customers to let them know they're over their limit.

In our case - **we will be strict and disallow tickets to be created when the limit is reached** 😈.

This is a business decision that you need to make based on your specific use case - but bear in mind you need to handle this in the application logic.

### Displaying the remaining tickets in the UI

Letting customers know how many tickets they have left before they reach their limit is a great way to keep them informed and encourage them to upgrade their plan. We can display the remaining tickets in the UI so that customers can see how many tickets they have left.

To do so, **we will use the Postgres function we created earlier to get the remaining tickets for the user**. We can then display this information in the UI so that customers can see how many tickets they have left before they reach their limit.

We can display this information on the whole `[account]` layout, so that the user can see how many tickets they have left at any time in the account's workspace.

```typescript {% title="apps/web/app/home/[account]/layout.tsx" %} {37,77-85,102-110}
import { use } from 'react';

import { cookies } from 'next/headers';
import Link from 'next/link';

import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { TeamAccountWorkspaceContextProvider } from '@kit/team-accounts/components';
import { If } from '@kit/ui/if';
import {
  Page,
  PageLayoutStyle,
  PageMobileNavigation,
  PageNavigation,
} from '@kit/ui/page';

import { AppLogo } from '~/components/app-logo';
import { getTeamAccountSidebarConfig } from '~/config/team-account-navigation.config';
import { withI18n } from '~/lib/i18n/with-i18n';

// local imports
import { TeamAccountLayoutMobileNavigation } from './_components/team-account-layout-mobile-navigation';
import { TeamAccountLayoutSidebar } from './_components/team-account-layout-sidebar';
import { TeamAccountNavigationMenu } from './_components/team-account-navigation-menu';
import { loadTeamWorkspace } from './_lib/server/team-account-workspace.loader';

type TeamWorkspaceLayoutProps = React.PropsWithChildren<{
  params: Promise<{ account: string }>;
}>;

function TeamWorkspaceLayout({
  children,
  params,
}: React.PropsWithChildren<TeamWorkspaceLayoutProps>) {
  const account = use(params).account;
  const data = use(loadTeamWorkspace(account));
  const remainingTickets = use(getRemainingTicketsForAccount(data.account.id));
  const style = use(getLayoutStyle(account));

  const accounts = data.accounts.map(({ name, slug, picture_url }) => ({
    label: name,
    value: slug,
    image: picture_url,
  }));

  return (
    <Page style={style}>
      <PageNavigation>
        <If condition={style === 'sidebar'}>
          <TeamAccountLayoutSidebar
            account={account}
            accountId={data.account.id}
            accounts={accounts}
            user={data.user}
          />
        </If>

        <If condition={style === 'header'}>
          <TeamAccountNavigationMenu workspace={data} />
        </If>
      </PageNavigation>

      <PageMobileNavigation className={'flex items-center justify-between'}>
        <AppLogo />

        <div className={'flex space-x-4'}>
          <TeamAccountLayoutMobileNavigation
            userId={data.user.id}
            accounts={accounts}
            account={account}
          />
        </div>
      </PageMobileNavigation>

      <TeamAccountWorkspaceContextProvider value={data}>
        <If condition={remainingTickets >= 0 && remainingTickets < 10}>
          <div
            className={
              'bg-red-500 py-1 text-center text-xs font-medium text-white'
            }
          >
            You have {remainingTickets} tickets remaining.{' '}
            <Link className={'underline'} href={`billing`}>
              Please upgrade your plan to continue receiving tickets
            </Link>
            .
          </div>
        </If>

        {children}
      </TeamAccountWorkspaceContextProvider>
    </Page>
  );
}

async function getLayoutStyle(account: string) {
  const cookieStore = await cookies();

  return (
    (cookieStore.get('layout-style')?.value as PageLayoutStyle) ??
    getTeamAccountSidebarConfig(account).style
  );
}

export default withI18n(TeamWorkspaceLayout);

async function getRemainingTicketsForAccount(accountId: string) {
  const client = getSupabaseServerClient();

  const { data } = await client.rpc('get_remaining_tickets', {
    target_account_id: accountId,
  });

  return data ?? 0;
}
```

And this is how it looks like in the UI:

{% img src="/assets/courses/next-turbo/remaining-tickets-banner.webp" width="2932" height="750" /%}

### Further considerations for billing and feature limits

When it comes to billing and enforcing feature limits, there are a few additional considerations to keep in mind, whether you're using Makerkit or not:

1. **Grace Periods**: You might want to provide a grace period for users who exceed their limits. This allows them to continue using the service for a short period before they're locked out.
2. **Handle Downgrades**: If a user downgrades their plan, you need to handle the transition gracefully. For example, you might need to adjust their feature limits or prorate their charges. You may also need to handle the case where the current user resources are exceeding the new plan limits. In this case, you have to think about how to handle this situation.
3. **Notifications**: Inform users when they're approaching their limits or have exceeded them. This helps them stay informed and take action to avoid disruptions in service. You can use Postgres crons to periodically check customers who have exceeded their limits and send them an email.

By considering these factors, you can create a seamless billing experience for your users and ensure that they're aware of their plan limits and usage.

## Conclusion

This module marks the end of our journey into setting up payments and billing in your Makerkit Next.js Supabase Turbo project. We've covered a lot of ground, from understanding billing schemas to enforcing feature limits based on plan selection.

Once your billing is set up and ready to go, you can start accepting payments from your customers and manage their subscriptions seamlessly. With Makerkit's flexible billing system, you can define your pricing structure and adapt it as your business grows.

I hope this module has given you a solid foundation for setting up payments and billing in your project. If you have any questions or need further assistance, feel free to reach out to the Makerkit community for support.

In the next lesson, we will deploy the application to production, including setting up Vercel, Supabase, and Stripe. Let's get ready to launch your project to the world! 🚀🌍


