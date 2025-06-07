-----------------
FILE PATH: kits/remix-supabase-turbo/analytics/google-analytics-provider.mdoc

---
status: "published"

title: 'Using the Google Analytics Provider in Remix Supabase Turbo'
label: 'Google Analytics'
description: 'Learn how to use the Google Analytics provider in Remix Supabase Turbo'
order: 2
---


The [Google Analytics](https://marketingplatform.google.com/about/analytics/) provider in Remix Supabase Turbo is a simple way to integrate Google Analytics into your Remix application using the Makerkit's Analytics package.

## Installation

First, you need to pull the `@kit/analytics` package into your project using the CLI

```bash
npx @makerkit/cli@latest plugins install
```

When prompted, select the `Google Analytics` package from the list of available packages. Once the command completes, you should see the `packages/plugins/google-analytics` directory in your project.

You can now import this package into your project:

```bash
pnpm add "@kit/google-analytics@workspace:*" --filter "@kit/analytics" -D
```

You can now use the Google Analytics plugin in the Analytics package. Update the `packages/analytics/src/index.ts` file as follows:

 ```tsx {% title="packages/analytics/src/index.ts" %}
import { createGoogleAnalyticsService } from '@kit/google-analytics';

import { createAnalyticsManager } from './analytics-manager';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
    providers: {
        'google-analytics': createGoogleAnalyticsService,
    },
});
```

## Configuration

Please add the following environment variables to your `.env` file:

```bash
VITE_GA_MEASUREMENT_ID=your-measurement-id
```

This is the Measurement ID of your Google Analytics property. You can find it in the Google Analytics dashboard.

Additionally, you can add the following environment variable to your `.env` file:

```bash
VITE_GA_DISABLE_PAGE_VIEWS_TRACKING=true
VITE_GA_DISABLE_LOCALHOST_TRACKING=true
```


-----------------
FILE PATH: kits/remix-supabase-turbo/analytics/posthog-analytics-provider.mdoc

---
status: "published"
title: 'Using the PostHog Analytics Provider in Remix Supabase Turbo'
label: 'PostHog'
description: 'Learn how to use the PostHog Analytics provider in Remix Supabase Turbo'
order: 3
---

The [Posthog](https://posthog.com) provider in Remix Supabase Turbo is a simple way to integrate PostHog Analytics into your Next.js application using the Makerkit's Analytics package.

## Installation

First, you need to pull the `@kit/analytics` package into your project using the CLI

```bash
npx @makerkit/cli@latest plugins install
```

When prompted, select the `PostHog` package from the list of available packages. Once the command completes, you should see the `packages/plugins/posthog` directory in your project.

You can now import this package into your project:

```bash
pnpm add "@kit/posthog@workspace:*" --filter "@kit/analytics" -D
```

You can now use the PostHog plugin in the Analytics package. Update the `packages/analytics/src/index.ts` file as follows:

 ```tsx {% title="packages/analytics/src/index.ts" %}
import { createPostHogAnalyticsService } from '@kit/posthog';

import { createAnalyticsManager } from './analytics-manager';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
  providers: {
    posthog: createPostHogAnalyticsService,
  },
});
```

## Configuration

Please add the following environment variables to your `.env` file:

```bash
VITE_POSTHOG_KEY=your-project-key
VITE_POSTHOG_HOST=your-host
```

If you have set up a Proxy to avoid Ad Blockers, please add the following environment variable to your `.env` file:

```bash
VITE_POSTHOG_INGESTION_URL=your-ingestion-url
```

Please make sure to proxy the `/ingest` endpoint to the PostHog server. Remix doesn't have a built-in proxying mechanism, so you will need to add it.

-----------------
FILE PATH: kits/remix-supabase-turbo/analytics/umami-analytics-provider.mdoc

---
status: "published"

title: 'Using the Umami Analytics Provider in Remix Supabase Turbo'
label: 'Umami'
description: 'Learn how to use the Umami Analytics provider in Remix Supabase Turbo'
order: 4
---


The [Umami](https://umami.is/) analytics provider in Remix Supabase Turbo is a simple way to integrate Umami Analytics into your Remix application using the Makerkit's Analytics package.

## Installation

First, you need to pull the `@kit/analytics` package into your project using the CLI

```bash
npx @makerkit/cli@latest plugins install
```

When prompted, select the `Umami` package from the list of available packages. Once the command completes, you should see the `packages/plugins/umami` directory in your project.

You can now import this package into your project:

```bash
pnpm add "@kit/umami@workspace:*" --filter "@kit/analytics" -D
```

You can now use the Google Analytics plugin in the Analytics package. Update the `packages/analytics/src/index.ts` file as follows:

 ```tsx {% title="packages/analytics/src/index.ts" %}
import { createUmamiAnalyticsService } from '@kit/umami';

import { createAnalyticsManager } from './analytics-manager';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
    providers: {
        umami: createUmamiAnalyticsService,
    },
});
```

## Configuration

Please add the following environment variables to your `.env` file:

```bash
VITE_UMAMI_HOST=your-umami-host
VITE_UMAMI_WEBSITE_ID=your-umami-website-id
```

The `VITE_UMAMI_HOST` is the URL of your Umami instance. Since Umami can be self-hosted, this can be any valid URL - or can be Umami's cloud service.

The `VITE_UMAMI_WEBSITE_ID` is the ID of your website in your Umami instance. This is a required field to track events in your website.

-----------------
FILE PATH: kits/remix-supabase-turbo/analytics.mdoc

---
status: "published"
title: 'Using the Analytics API in your Makerkit project'
label: 'Analytics API'
order: 13
description: 'Learn how to use the Analytics API in your Makerkit project to track user behavior and app usage.'
collapsible: true
collapsed: true
---

Makerkit provides a powerful and flexible Analytics API that integrates seamlessly with the App Events system. This API allows you to track user behavior and app usage easily and consistently across your SaaS application.

## Core Concepts of the Analytics API

The Analytics API is built around three main concepts:

1. **Identify**: Associate user data with a unique user ID.
2. **Track Events**: Record specific events or actions taken by users.
3. **Track Page Views**: Record when users view specific pages in your application.

### Using the Analytics API

The Analytics API is available through the `analytics` object imported from `@kit/analytics`.

#### Identifying Users

Use the `identify` method to associate a user with their actions:

```typescript
import { analytics } from '@kit/analytics';

void analytics.identify(userId, {
  email: user.email,
  plan: user.subscriptionPlan,
  // ... other user properties
});
```

#### Tracking Events

Use the `trackEvent` method to record specific actions or events:

```typescript
void analytics.trackEvent('Button Clicked', {
  buttonName: 'Submit',
  page: 'Sign Up',
});
```

#### Tracking Page Views

**Makerkit automatically tracks page views for you.** However, you can also manually track page views if needed.

Use the `trackPageView` method to record when users view specific pages:

```typescript
void analytics.trackPageView('Sign Up');
```

NB: The `trackPageView` method is automatically called when the route changes in a Remix application.

### Integration with App Events

While you can use these methods directly, we recommend leveraging the App Events system for a more centralized approach. Makerkit automatically maps common app events to analytics tracking:

```typescript
import { useAppEvents } from '@kit/shared/events';

function SomeComponent() {
  const { emit } = useAppEvents();

  const handleSignUp = (userId: string) => {
    emit({ type: 'user.signedUp', payload: { userId } });
    // This automatically calls analytics.identify and analytics.trackEvent
  };

  // ...
}
```

### Extending the Analytics API

You can extend the analytics functionality by creating custom event mappings:

```typescript
import { analytics } from '@kit/analytics';
import { useAppEvents } from '@kit/shared/events';

interface MyAppEvents {
  'feature.used': { featureName: string };
}

export function useMyAnalytics() {
  const { emit } = useAppEvents<MyAppEvents>();

  return {
    trackFeatureUse: (featureName: string) => {
      emit({ type: 'feature.used', payload: { featureName } });
      // If you need additional tracking logic:
      void analytics.trackEvent('Feature Used', { featureName });
    },
  };
}
```

## Best Practices

When implementing analytics in your Makerkit project, consider the following best practices:

1. **Use App Events**: Whenever possible, use the App Events system instead of calling analytics methods directly. This keeps your analytics logic centralized and easier to maintain.
2. **Consistent Naming**: Use consistent naming conventions for your events and properties across your application.
3. **Relevant Data**: Only track data that's relevant and useful for your business goals. Avoid tracking sensitive or personal information.
4. **Testing**: Always test your analytics implementation to ensure events are being tracked correctly.
5. **Documentation**: Keep a record of all the events and properties you're tracking. This will be invaluable as your application grows.

By leveraging Makerkit's Analytics API in conjunction with the App Events system, you can create a robust, maintainable analytics setup that grows with your SaaS application. This approach provides the flexibility to track the data you need while keeping your codebase clean and organized.


-----------------
FILE PATH: kits/remix-supabase-turbo/api/account-api.mdoc

---
status: "published"
label: "Account API"
order: 0
title: "Account API | Remix Supabase SaaS Kit"
description: "A quick introduction to the Account API in Makerkit"
---

You can use the Account API for retrieving information about the personal user account.

## Usage

To use the Account API, you need to import the `createAccountsApi` function from `@kit/account/api`. We need to pass a valid `SupabaseClient` to the function - so we can interact with the database from the server.

```tsx
import { createAccountsApi } from '@kit/accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const api = createAccountsApi(client);

  // use api
}
```

If you're in a Remix Action context, you'd use:

```tsx
import { createAccountsApi } from '@kit/accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function action(args: ActionFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const api = createAccountsApi(client);
  
  // use api
}
```

## Methods

The Account API provides the following methods:

### Get the account workspace data

Get the account workspace data using the `getAccountWorkspace` method. This method returns the workspace data for the user account.

```tsx
const api = createAccountsApi(client);
const workspace = await api.getAccountWorkspace();
```

This is already called in the user account layout, so it's very unlikely you'll need to call this method.

### Load the user accounts

Load the user accounts using the `loadUserAccounts` method.

This method returns an array of user accounts. Each account has a `label`, `value`, and `image` property.

```tsx
const api = createAccountsApi(client);
const accounts = await api.loadUserAccounts();
```

### Get the subscription data

Get the subscription data for the given user using the `getSubscription` method.

This method returns the subscription data for the given user account.

```tsx
const api = createAccountsApi(client);
const subscription = await api.getSubscription(accountId);
```

Returns the table `subscriptions` and `subscription_items`.

### Get the billing customer ID

Get the billing customer ID for the given user using the `getCustomerId` method.

This method returns the billing customer ID for the given user account.

```tsx
const api = createAccountsApi(client);
const customerId = await api.getCustomerId(accountId);
```

-----------------
FILE PATH: kits/remix-supabase-turbo/api/authentication-api.mdoc

---
status: "published"
label: "Authentication API"
order: 2
title: "Authentication API | Remix Supabase SaaS Kit"
description: "A quick introduction to the Authentication API in Makerkit"
---

The Authentication API is a set of functions that help you authenticate users in your application. It is built on top of the Supabase authentication system and provides a simple way to authenticate users in your application.

```tsx
import { redirect } from '@remix-run/react';
import { LoaderFunctionArgs } from '@remix-run/node';
import { requireUser } from '@kit/supabase/require-user';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const auth = await requireUser(client);

  // check if the user needs redirect
  if (auth.error) {
    return redirect(auth.redirectTo);
  }

  // user is authed!
  const user = auth.data;

  // return your data
  return {
    data: {} // your data here,
  };
}
```

If the user needs MFA and is not yet verified, the `redirect` function will redirect the user to the MFA verification page. This is why it is important to check the `redirectTo` property in the response.

-----------------
FILE PATH: kits/remix-supabase-turbo/api/team-account-api.mdoc

---
status: "published"
label: "Team Account API"
order: 1
title: "Team Account API | Remix Supabase SaaS Kit Turbo"
description: "A quick introduction to the Team Account API in the Remix Supabase SaaS Kit"
---

You can use the Team Account API for retrieving information about the team account.

## Usage

To use the Team Account API, you need to import the `createTeamAccountsApi` function from `@kit/team-account/api`. We need to pass a valid `SupabaseClient` to the function - so we can interact with the database from the server.

```tsx
import { createTeamAccountsApi } from '@kit/team-accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function loader() {
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
FILE PATH: kits/remix-supabase-turbo/api.mdoc

---
status: "published"
title: "The API services in Remix Supabse Tubo"
label: "API"
order: 5
description: "Services API allows you to interact with the entities in the Makerkit application."
collapsible: true
collapsed: true
---

The API services in Remix Supabase Tubo allow you to interact with the entities in the Makerkit application.

1. [**Account API**](account-api): Use this service to interact with the accounts in the application.
2. [**Team Account API**](team-account-api): Use this service to interact with the team accounts in the application.
3. [**Authentication API**](authentication-api): Use this service to interact with the authentication methods in the application.
4. [**User Workspace API**](user-workspace-api): Use this service to interact with the user workspace in the application.
5. [**Team Account Workspace API**](team-account-workspace-api): Use this service to interact with the team account workspace in the application.


-----------------
FILE PATH: kits/remix-supabase-turbo/billing/billing-api.mdoc

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
FILE PATH: kits/remix-supabase-turbo/billing/billing-schema.mdoc

---
status: "published"
label: "Billing Schema"
title: "Writing the billing schema in the Remix Supabase Kit"
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

