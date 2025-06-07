-----------------
FILE PATH: kits/remix-fire/going-to-production/publish-firebase-security-rules.mdoc

---
status: "published"

title: "Publish the Firebase Security Rules variables before going to production"
label: Security Rules
order: 2
---


When running the Firebase Emulator, you can update your Firebase Firestore and Storage security rules by changing the relative files `firestore.rules` and `storage.rules`.

Before deploying your application to your users, you need to make sure that
you're also publishing the security rules from your Firebase Console.

### Storage Security Rules

To publish your Firebase Storage security rules, navigate to the Firebase
Console.

From the main sidebar, click on "Storage". Afterward, click on the "Rules" tab:

{% img width="955" height="719" src="/assets/images/docs/storage-rules-tab.webp" /%}

Afterward, paste the content from `storage.rules` to the text editor, and
click "Publish".

{% img width="758" height="628" src="/assets/images/docs/storage-rules-editor.webp" /%}

### Firestore Security Rules

To update your Firestore Security rules, follow the same steps as for the
Storage rules explained above.

Of course, the only difference is navigating to the Firestore tab, and
choosing the content of `firestore.rules`.


-----------------
FILE PATH: kits/remix-fire/how-to/introduction.mdoc

---
status: "published"
title: "Introduction to the How to section of the Makerkit Remix documentation"
label: Introduction
order: 0
description: "Here is a brief introduction to the How to section of the Makerkit Remix Lite documentation."
---


-----------------
FILE PATH: kits/remix-fire/organizations/extending-organizations.mdoc

---
status: "published"
title: Extending Organizations in Remix.js Firebase SaaS Starter Kit
description: Learn how to extend the organization's data model in your Remix.js Firebase SaaS Starter Kit application
label: Extending Organizations
order: 1
---

There are many ways you can extend the organization's data model in your applications: you will use this entity for the objects that are shared among the group's users.

For example, here is a popular scenario: we assume users can install integrations for their organizations. Integrations can have the following interface:

 ```tsx {% title="integration.ts" %}
enum IntegrationType {
  Zapier,
  Notion,
  // etc
}

interface Integration<IntegrationData = unknown> {
  type: IntegrationType;
  data: IntegrationData;
}
```

Then, you can extend the organization's interface and add an `integrations` property to `Organization`, and initialize it as an empty array `[]` when you create a new integration:

 ```tsx {% title="organization.ts" %}
interface Organization {
  // ...
  integrations: Integration[];
}
```

-----------------
FILE PATH: kits/remix-fire/organizations/organization-hooks.mdoc

---
status: "published"
title: Organization React Hooks | Remix.js Firebase SaaS Kit
label: Custom React Hooks
order: 10
description: 'Reference for the organizations React hooks in your Remix.js Firebase SaaS Kit'
---

## useCurrentOrganization

This hook returns the current organization. It's a wrapper around the `OrganizationContext` context. The returned value is an `Organization` object.

```tsx
import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';

const organzation = useCurrentOrganization();
```

## useIsSubscriptionActive

This hook returns a boolean indicating whether the current organization has
an active subscription. It's a shortcut around the `useCurrentOrganization`
hook that checks if the `status` property is set to `active` or `trialing`.

```tsx
import useIsSubscriptionActive from '~/lib/organizations/hooks/use-is-subscription-active';

const isSubscriptionActive = useIsSubscriptionActive();
```

## Other Organization Hooks

The kit has many more hooks that are used internally: they are not meant to be used directly unless you want to update the logic or fix a bug.

All the hooks are available in the `~/lib/organizations/hooks` directory.

-----------------
FILE PATH: kits/remix-fire/organizations/organizations-overview.mdoc

---
status: "published"

title: MakerKit's Organizations
label: Overview
order: 0
---


The Makerkit boilerplate's data model is modeled around *Organizations*. 

Organizations are groups of users, which can also be named Projects, Teams, and so on: it really depends on your application's domain.

The data model of an organization looks like the following:

 ```tsx {% title="organization.ts" %}
type UserId = string;

interface Organization {
  name: string;
  timezone?: string;
  logoURL?: string | null;
  subscription?: OrganizationSubscription;
  customerId?: string;

  members: Record<UserId, {
    role: number;
    user: Reference;
  }>;
}
```

### Renaming "Organizations" to another entity

Makerkit uses "organizations" to refer to a group of users. However, the name "organizations" may not make sense to your application's domain, even with a similar technical data model. 

For example, you may want to call them "Teams", or "Workspaces". In this case, we suggest you keep using "organization" as a technical concept (otherwise, you'd have to rename every instance) within the codebase while changing the labels in the UI using the i18n files at `public/locales`. 

To do so, simply rename all "Organization" instances according to your preferred choice.

-----------------
FILE PATH: kits/remix-fire/organizations/permissions.mdoc

---
status: "published"
title: Managing User Permissions in Remix Firebase
label: User Permissions
description: "Learn how to write a simple permissions system based on the
users' role in your Makerkit applications using Remix Firebase"
order: 2
---

Most permissions are written in a single file at `src/lib/organizations/permissions.ts`.

Here, you can find some of the examples used in the boilerplate so that you can start writing your own.

Why are permissions written in a single file? Because it's easy to write inline logic and lose track of it. Therefore, we will write all the business logic within the same file and encapsulated as simple functions.

Let's take a look at a simple permission function in the boilerplate:

 ```tsx {% title="src/lib/organizations/permissions.ts" %}
/**
 *
 * @param currentUserRole The current logged-in user
 * @param targetUser The role of the target of the action
 * @description Checks if a user can perform actions (such as update a role) of another user
 * @name canUpdateUser
 */
export function canUpdateUser(
  currentUserRole: MembershipRole,
  targetUser: MembershipRole
) {
  return currentUserRole > targetUser;
}
```

The function takes two parameters: the current user's role and the target user's role, and checks if the current user can update the target user's role (or anything).

Now, we can use the function above with the `IfHasPermissions` component to display or hide some parts of the application. This component automatically injects the current user's role, such as below:

```tsx
<IfHasPermissions
  condition={(currentUserRole) =>
    canInviteUser(currentUserRole, targetUserRole)
  }
>
  <InviteUserComponent />
</IfHasPermissions>
```

The `InviteUserComponent` component will be displayed if the condition is truthy.

Otherwise, you can use these functions throughout the application on both the client and the server.


-----------------
FILE PATH: kits/remix-fire/organizations/subscription-permissions.mdoc

---
status: "published"
title: Subscription Permissions | Remix Firebase SaaS Kit
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
`isOrganizationSubscriptionActive` and `getOrganizationSubscription`

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

### Firestore Rules

Another essential thing to take into account is securing our Firestore
rules using the organization's subscription.

For example, let's assume your application requires organizations to be
paying customers to write to a certain collection:

```js
function getOrganizationSubscription() {
  let organization = getOrganization(organizationId);

  return organization.subscription;
}

function isPayingOrganization(subscription) {
  return subscription != null && (subscription.status == 'paid' || subscription.status == 'trialing');
}

function isProPlan(subscription) {
  let organization = getOrganization(organizationId);

  return subscription.stripePriceId == 'pro-plan-id';
}

function canWriteToCollection(organizationId) {
  let subscription = getOrganizationSubscription();

  return isPayingOrganization(subscription) && isProPlan(subscription);
}

match /organizations/{organizationId} {
  match /tasks/{task} {
    allow create: canWriteToCollection(organizationId);
    allow list: if userIsMemberByOrganizationId(organizationId);
  }
}
```

## Server Side Gating

When gating access on the server side, we can use two utilities that allow
us to get the organization's subscription by its ID.

### Using Firestore utilities

When you need to check the user's subscription in your API functions, you can
use the functions `isOrganizationSubscriptionActive` and
`getOrganizationSubscription` exported from
`src/lib/server/organizations/memberships.ts`.

For example, let's assume we have an API handler that performs an action:

```tsx
export default async function action({ request }) {
  const organization = await parseOrganizationCookie(request);

  const isSubscriptionActive =
    await isOrganizationSubscriptionActive(organization);

  if (!isSubscriptionActive) {
    return throwForbiddenError();
  }

  // all good! perform action
}
```


-----------------
FILE PATH: kits/remix-fire/payments/lemon-squeezy.mdoc

---
status: "published"
title: "Use Lemon Squeezy with MakerKit | Remix.js Firebase"
label: Migrate to Lemon Squeezy
order: 4
description: 'Lear how to migrate your Remix.js Firebase MakerKit project to Lemon Squeezy'
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

Depending on which kit you're using, you will add this key to either `.env.local` (Next.js) or `.env` (Remix). This key is not supposed to be committed to your repository, so we add them safely to our local files.

Additionally, we need to add the `LEMON_SQUEEZY_STORE_ID` to the local environment variables file, and a secret string `LEMON_SQUEEZY_SIGNING_SECRET` that we use to verify the webhook signature.

Update them in your local environment variables file:

```
LEMON_SQUEEZY_API_KEY=<YOUR_KEY>
LEMON_SQUEEZY_STORE_ID=<YOUR_STORE_ID>
LEMONS_SQUEEZY_SIGNATURE_SECRET=<a random string>
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

#### Find your Lemon Squeezy product and variant IDs

You can find your Lemon Squeezy Product and Variant IDs using the Lemon Squeezy dashboard.

### Step 4: Creating the Webhook In Lemon Squeezy

We need to create a Webhook from the Lemon Squeezy Dashboard so that we can receive the events to our webhook handler.

You will be prompted for the following:

- URL: point it to the ngrok URL (see below). Ensure you point it to the `/resources/ls/webhook` endpoint.
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
FILE PATH: kits/remix-fire/payments/stripe-configuration.mdoc

---
status: "published"

title: Set up Stripe Payments for Remix and Firebase SaaS applications
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
}
```

{% alert type="warning" %}
The properties "stripePriceId" are placeholders. You will need to replace them with the IDs of your plans.
{% /alert %}

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
FILE PATH: kits/remix-fire/payments/stripe-one-time-payments.mdoc

---
status: "published"
title: Enabling One-Time Payments with Stripe | Remix Firebase
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
FILE PATH: kits/remix-fire/payments/stripe-webhooks.mdoc

---
status: "published"
title: How Makerkit handles Stripe Webhooks in your Remix Firebase application
label: Stripe Webhooks
order: 1
description: 'Learn how MakerKit handles Stripe Webhooks in your Remix Firebase application'
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

When a subscription is updated, we rebuild the subscription object and update it in the organization's Firestore entity.

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

Then, you can add the data to the subscription object using the `buildOrganizationSubscription` function:

 ```tsx {% title="lib/stripe/build-organization-subscription.ts" %}
import type { Stripe } from 'stripe';
import type { OrganizationSubscription } from '~/lib/organizations/types/organization-subscription';

export function buildOrganizationSubscription(
  subscription: Stripe.Subscription,
): OrganizationSubscription {
  const lineItem = subscription.items.data[0];
  const price = lineItem.price;

  return {
    // your props
    quantity: lineItem.quantity,

    // default props
    id: subscription.id,
    priceId: price?.id,
    status: subscription.status,
    cancelAtPeriodEnd: subscription.cancel_at_period_end,
    currency: lineItem.price.currency ?? null,
    interval: price?.recurring?.interval ?? null,
    intervalCount: price?.recurring?.interval_count ?? null,
    createdAt: subscription.created,
    periodStartsAt: subscription.current_period_start,
    periodEndsAt: subscription.current_period_end,
    trialStartsAt: subscription.trial_start ?? null,
    trialEndsAt: subscription.trial_end ?? null,
  };
}
```

