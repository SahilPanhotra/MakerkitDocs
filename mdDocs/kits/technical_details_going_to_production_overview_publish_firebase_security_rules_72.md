-----------------
FILE PATH: kits\remix-fire\getting-started\technical-details.mdoc

---
status: "published"
title: The technologies that make up a Remix.js Firebase application
label: Technical Details
order: 1
description: 'MakerKit uses Remix, Firebase, Firestore, Tailwind CSS, Zod,
Pino, Radix UI, Typescript and Cypress to build wonderful SaaS applications'
---

MakerKit is built primarily with Remix, Firebase, and Tailwind CSS.

Under the hood, there are many more libraries and architectural
decisions you should know.

### 1) A Scalable Remix Application
The codebase is entirely built with [Remix](https://remix.run).

The front end is fully built with React, leveraging React Hooks
and the best practices available today.

### 2) Built on top of Firebase

[Firebase](https://firebase.com) is one of the most popular *PaaS* out there,
with a very generous free tier and fairly affordable pricing for growing
applications.

MakerKit uses Firebase for Authentication, Firestore for storing your data,
Storage for storing files, and AppCheck to protect your endpoints against
possible attacks.

### 3) Theme built with Tailwind 3

The bulk of the application's theme, built with [Tailwind 3](https://tailwindcss.com/), is
stored in three CSS files rather than HTML.

It may sound like a bad practice, but we approached this way to help you find and edit
the theme's CSS in one single place.

We designed the theme in light and dark, so you can choose to use both according to your user's OS settings or manage it using a local cookie.

### 4) Radix UI Components

All the kits use [Radix UI](https://radix-ui.com), arguably the best headless UI library for React.

The components are styled using Tailwind CSS. Many components are inspired by the template made by [Shad CN](https://ui.shadcn.com/). We recommend taking a look if you want to build your own components.

Some components are still using Headless UI, but will be phased out in the future.

### 5) Fully and Strictly Typed with Typescript

We wrote every single line of code with *strictly-typed
Typescript* both on the client and server side.

### 6) Payload validation with Zod

The API endpoints are **validated and protected with [Zod](https://github.com/colinhacks/zod)**, which also helps with strong-typing.

### 7) E2E testing with Cypress

We ship MakerKit with [Cypress](https://www.cypress.io/), likely the most
popular E2E testing framework.

We made lots of examples and utilities to change the tests as you develop
your application.

### 8) Linted with EsLint and formatted with Prettier

We lint the codebase with **a very strict EsLint configuration**, but
we distribute it with less strict params so that it won't be in your way
as you're experimenting with the starter.

We will show you how to add stricter parameters if that's your thing, but we do not recommend jumping head-in with it. It's much better to activate the stricter configuration once you have already shipped the product and are looking to stabilize it.

Furthermore, the codebase is **formatted with Prettier** automatically.

### 9) Multi-language by default with remix-i18n

We configured the application to be translated by default thanks to the
`remix-i18n` package.

All the application's strings are translated by default, so you can easily
extend it to other languages if necessary.

Using translation files can help you if you want to **translate your apps in multiple
languages**, but it also allows you to find the text strings you
want to replace.

In this way, you do not need to change strings in the codebase, but they're
**neatly organized in JSON files**.

### 10) Logging with Pino

MakerKit uses [Pino](https://github.com/pinojs/pino), a lightweight logging library.

API errors are logged with just enough information to help you debug
your API routes.


-----------------
FILE PATH: kits\remix-fire\going-to-production\going-to-production-overview.mdoc

---
status: "published"
title: Checklist to ship a Remix.js Firebase SaaS application to Production
description: This checklist will help you ship your Remix.js Firebase SaaS application to production
label: Production Checklist
order: 0
---

Before going to production, it's essential to follow certain precautions to avoid failures.

## 1) Environment Variables

First, we must safely inject the correct environment variables into their relative environment. MakerKit has predefined templates that allow you to understand where each variable should be added.

Before publishing your SaaS, or if you are making a production release after
a change and this included an environment variable; ensure your CI provider is
up to date with the correct variable in the right environments.

Since the `.env` file will not be committed to your repository, you will
need to add the environment variables beforehand.

## 2) Firebase Security Rules (Firestore and Storage)

Whenever your Security rules need updating, you have to make sure to
propagate these changes and publish them from your Firebase Console.

## 3) Firestore Indexes

As you may know, the Firestore emulator does not fail when a new index is
needed, like its production version. This can be dangerous!

If you are setting up Makerkit for the first time, remember to set up the
an index for querying an invitation without knowing its organization.

### Create Index Exemption by clicking on the link below

You can do so using the following link (replace
"\[\[YOUR_PROJECT_ID\]\]" with your real project ID):

https://console.firebase.google.com/u/0/project/\[\[YOUR_PROJECT_ID\]\]/firestore/indexes/single-field?create_exemption=ClBwcm9qZWN0cy9zdG9yeWpveS00NmVlMy9kYXRhYmFzZXMvKGRlZ
mF1bHQpL2NvbGxlY3Rpb25Hcm91cHMvaW52aXRlcy9maWVsZHMvY29kZRACGggKBGNvZGUQAQ

If you have added other complex queries that need a Firestore index
exemption, **ensure you are testing thoroughly with an actual Firestore database
to avoid similar issues**.

## 4) Enable Authentication providers

If you have added authentication providers other than Google, make sure you
have enabled them from the Console and added the needed configuration so
that they work correctly.

For many of them, you will have to provide an app
ID or a key, so configure and test them in a production environment.

## 5) Enable your Stripe account Billing Status

Remember to complete your Stripe account's information and enable billing and to not forget it in `Testing Mode`.

Additionally, you will need to add your Stripe API keys to the environment variables to process payments. Ensure you have added the correct keys to the correct environment and that have selected all the required events to your webhook.

Please follow the [Stripe configuration guide](/docs/remix-fire/stripe-configuration) to configure your Stripe account.

## 6) Add your SMPT service credentials

You will need to add your SMTP service credentials to the environment variables to send emails. This is required for sending emails to users when they are invited to an organization.

To set the credentials, use the global configuration file `configuration.ts`.

## 7) Enable Google Analytics (Optional)

Understand how visitors find your website early on.

## 8) Enable Sentry (Optional)

With 4000 events for free per month, we recommend that you use [Sentry.io](https://sentry.io) for
ensuring you can catch, analyze and fix any runtime errors in your application that your users will encounter.

## 9) Set up a Logging Service (Optional, highly recommended)

We recommend that you use a logging service to log any events that occur in your application. This will allow you to debug any issues that may occur in your application.

While this is optional, we highly recommend that you use a logging service. Logging is how you can debug issues in your application, and it is essential to have a logging service in place.

Some logging services that we recommend are:

- [Baselime](https://baselime.io)
- [Logflare](https://logflare.app)
- [Axiom](https://axiom.co)

The above can be installed with a one-click install in the Vercel dashboard (if you are using Vercel).

-----------------
FILE PATH: kits\remix-fire\going-to-production\publish-firebase-security-rules.mdoc

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
FILE PATH: kits\remix-fire\how-to\introduction.mdoc

---
status: "published"
title: "Introduction to the How to section of the Makerkit Remix documentation"
label: Introduction
order: 0
description: "Here is a brief introduction to the How to section of the Makerkit Remix Lite documentation."
---


-----------------
FILE PATH: kits\remix-fire\organizations\extending-organizations.mdoc

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
FILE PATH: kits\remix-fire\organizations\organization-hooks.mdoc

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
FILE PATH: kits\remix-fire\organizations\organizations-overview.mdoc

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
FILE PATH: kits\remix-fire\organizations\permissions.mdoc

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
FILE PATH: kits\remix-fire\organizations\subscription-permissions.mdoc

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
FILE PATH: kits\remix-fire\payments\lemon-squeezy.mdoc

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
FILE PATH: kits\remix-fire\payments\stripe-configuration.mdoc

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


