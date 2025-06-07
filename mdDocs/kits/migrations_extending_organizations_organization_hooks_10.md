-----------------
FILE PATH: kits/next-fire/migrations.mdoc

---
status: "published"
title: "Migrations"
label: "Migrations"
order: 11
collapsible: true
collapsed: true
---

The 0.11.0 release of the kit introduces a number of breaking changes. This guide will help you migrate your code to the new version.

NB: If you have cloned the repository after the 0.11.0 release, you can skip this guide.

---


The changes impact the following areas:

1. Tailwind CSS classes
2. Installed Packages

## Tailwind CSS classes

We have updates the Tailwind CSS classes using a new convention named `dark`, which is used for dark mode. This is a breaking change, as it will impact the way you use the Tailwind CSS classes.

Before, we used to have a `black` color - but it is no longer available. Instead, we have a `dark` color, which is used for dark mode. This means that you will need to update your code to use the new `dark` color.

Additionally, the new color uses a different level of opacity, which is more suitable to the standard Tailwind CSS convention, and is more consistent with the other colors.

By default, the `dark` color is an alias for `colors.gray` from the Tailwind CSS palette. You can change it by updating the `colors.dark` property in the `tailwind.config.js` file.

For example, you can provide your own palette, or switch to another dark color such as `colors.slate`.

 ```js {% title="tailwind.config.js" %}
const colors = require('tailwindcss/colors');

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./app/**/*.{ts,tsx,jsx,js}'],
  darkMode: 'class',
  theme: {
    extend: {
      fontFamily: {
        serif: ['serif'],
        heading: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
        sans: [
          'system-ui',
          'BlinkMacSystemFont',
          'Inter',
          'Segoe UI',
          'Roboto',
          'Ubuntu',
        ],
        monospace: [`SF Mono`, `ui-monospace`, `Monaco`, 'Monospace'],
      },
      colors: {
        primary: {
          ...colors.indigo,
          contrast: '#fff',
        },
        dark: colors.slate,
      },
    },
  },
};
```

### Automating the migration

To automate the migration, you can use the following script. First, install the package (either globally or locally):

```bash
npm i replace-in-file
```

Then, copy this script into your project named "migrate-0.11.js":

```js title="migrate-0.11.js
const replace = require('replace-in-file');
const files = ['src/**/*.tsx', 'src/**/*.css'];

const promise = () =>
  new Promise((resolve, reject) => {
    replace({
      ...getConfigs(),
      files,
    })
      .then((results) => {
        results.forEach((result) => {
          if (result.hasChanged) {
            console.log('File changed:', result.file);
          }
        });

        resolve();
      })
      .catch((error) => {
        console.error('Error occurred:', error);

        reject(error);
      });
  });

(async () => {
  console.log(`Replacing classes in "${files}" ...`);
  await promise();
  console.log('Done!');
})();

function getConfigs() {
  return {
    from: [
      /black-600/g,
      /black-500/g,
      /black-400/g,
      /black-300/g,
      /black-200/g,
      /black-100/g,
      /black-50/g,
      "'classnames'",
    ],
    to: [
      'dark-900',
      'dark-900',
      'dark-800',
      'dark-700',
      'dark-600',
      'dark-500',
      'dark-400',
      "'clsx'",
    ],
  };
}
```

Then, run the script:

```bash
node migrate-0.11.js
```

This will replace all the classes in your project.

## Packages

### Replaced "classnames" with "clsx"

Additionally, we replaced the package `classNames` with a newer, smaller, faster and maintained package named `clsx`. This is a breaking change, as it will impact the way you import the package.

The command above will also replace the import statement for you. However, if you want to do it manually, rename every instance of `'classnames'` with `'clsx'` in your project.

### Removed "Headless UI"

We have removed the package `headlessui` from the kit. It's slowly maintained and we don't use it anymore.

This is a breaking change, as it will impact the way you import the package.

If you have components using `headlessui`, please keep it in your project. Otherwise, you can remove it.

## Conclusion

This guide should help you migrate your project to the new version of the kit. If you have any questions, please get in touch with me.

-----------------
FILE PATH: kits/next-fire/organizations/extending-organizations.mdoc

---
status: "published"
title: Extending Organizations in Next.js Firebase SaaS Starter Kit
description: Learn how to extend the organization's data model in your Next.js Firebase SaaS Starter Kit application
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
FILE PATH: kits/next-fire/organizations/organization-hooks.mdoc

---
status: "published"
title: Organization React Hooks | Next.js Firebase SaaS kit
label: Custom React Hooks
order: 10
description: 'Reference for the organizations React hooks in your Next.js Firebase SaaS kit'
---

## useCurrentOrganization

This hook returns the current organization. It's a wrapper around the `OrganizationContext` context. The returned value is an `Organization` object.

```tsx
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const organzation = useCurrentOrganization();
```

## useIsSubscriptionActive

This hook returns a boolean indicating whether the current organization has
an active subscription. It's a shortcut around the `useCurrentOrganization`
hook that checks if the `status` property is set to `active` or `trialing`.

```tsx
import { useIsSubscriptionActive } from '~/lib/organizations/hooks/use-is-subscription-active';

const isSubscriptionActive = useIsSubscriptionActive();
```

## Other Organization Hooks

The kit has many more hooks that are used internally: they are not meant to be used directly unless you want to update the logic or fix a bug.

All the hooks are available in the `~/lib/organizations/hooks` directory.

-----------------
FILE PATH: kits/next-fire/organizations/permissions.mdoc

---
status: "published"
title: Managing User Permissions in Next.js Firebase
label: User Permissions
description: "Learn how to write a simple permissions system based on the
users' role in your Makerkit applications using Next.js Firebase"
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
FILE PATH: kits/next-fire/organizations/subscription-permissions.mdoc

---
status: "published"
title: Subscription Permissions | Next.js Firebase SaaS Kit
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
export default function async apiHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const organization = req.cookies.organization;

  const isSubscriptionActive =
    await isOrganizationSubscriptionActive(organization);

  if (!isSubscriptionActive) {
    return throwForbiddenError();
  }

  // all good! perform action
}
```

### Gating access to pages server-side

For example, let's assume we want to gate access to a page if the
`organization` is not on a paid plan. To do so, we can add some logic to
`getServerSideProps`:

```tsx
import { withAppProps } from './with-app-props';
import { Stripe } from "stripe";

function GatedPage() {
  return <div>...</div>;
}

export default GatedPage;

export async function getServerSideProps(ctx) {
  const appProps = await withAppProps(ctx);

  // something wrong happened, user gets redirected either way
  if ('redirect' in appProps) {
    return appProps;
  }

  // get organization from subscription
  const subscription = appProps.props.organization?.subscription;

  // check the subscription exists and status is "active" or "trialing"
  if (!subscription || !isSubscriptionActive(subscription.status)) {
    return {
      redirect: {
        destination: '/dashboard?error=forbidden-subscription',
      },
    };
  }

  // all good, we can simply return the props
  return appProps;
}

function isSubscriptionActive(status: Stripe.Subscription.Status) {
  return ['active', 'trialing'].includes(status);
}
```

Of course, in this case, you could extend `with-app-props` and make the
above reusable for all the routes that require organizations to have
additional permissions.


-----------------
FILE PATH: kits/next-fire/organizations.mdoc

---
status: "published"
title: Organizations
label: Organizations
order: 7
collapsible: true
collapsed: true
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
FILE PATH: kits/next-fire/payments/disable-stripe.mdoc

---
status: "published"

title: Ho to disable Stripe in MakerKit
label: Disable Stripe
order: 5
description: "Learn how to disable Stripe in your Makerkit project."
---

Sometimes - you don't want to enable payments yet. That's fine. You can disable Stripe in your MakerKit project.

## Disable Stripe

First, set `configuration.stripe.embedded` to `false` in the global configuration file.

 ```tsx {% title="src/configuration.ts" %}
{
  ...
  stripe: {
    embedded: false,
  },
  ...
}
```

You also want to delete or hide the page where users can subscribe to your product.

You can do so by removing the 'Subscriptions' page from the settings navigation at `src/components/settings/SettingsPageContainer.tsx`.

 ```tsx {% title="src/components/settings/SettingsPageContainer.tsx" %}
const links = [
  {
    path: '/settings/profile',
    i18n: 'common:profileSettingsTabLabel',
  },
  {
    path: '/settings/organization',
    i18n: 'common:organizationSettingsTabLabel',
  },
  /*
  {
    path: '/settings/subscription',
    i18n: 'common:subscriptionSettingsTabLabel',
  },
   */
];
```

You can re-add the page later when you are ready to enable payments.

-----------------
FILE PATH: kits/next-fire/payments/lemon-squeezy.mdoc

---
status: "published"

title: Use Lemon Squeezy with MakerKit
label: Migrate to Lemon Squeezy
order: 4
description: 'Learn how to migrate your MakerKit project to Lemon Squeezy'
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

```bash
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

```bash
"ngrok": "npx ngrok http 3000"
```

And you can run it with:

```bash
npm run ngrok
```

You can also use other alternatives, such as Localtunnel or Cloudflare Tunnel.

### Step 5: Go to production

Once you're ready to go to production, you can **switch your Lemon Squeezy store to production mode**.

Additionally, remember to **set up the webhook in production mode as well**, pointing to your production URL.

Finally, **update all the environment variables** to the production values.

-----------------
FILE PATH: kits/next-fire/payments/stripe-configuration.mdoc

---
status: "published"

title: Set up Stripe Payments for Next.js and Firebase SaaS applications
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

If you are using the new embedded checkout (introduced in version 0.14.0), you will also need to add the public Stripe key using the `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` variable:

- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`

Copy these values to your local `.env.local` file.

{% alert type="warning" title="Keep your .env.local file private" %}
Make sure to keep your <code>.env.local</code> file private. It should not be committed to your repository. These environment variables should be added using your hosting provider's dashboard.
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

Before you continue... **please make sure to follow these steps**:

1. Create a webhook endpoint in Stripe
2. Make sure to add the **events** you want to receive from Stripe
3. Copy the **signing secret** to your production environment variables as `STRIPE_WEBHOOK_SECRET`
4. Redeploy your application to apply the changes

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

{% alert type="warning" title="Please make sure to follow these steps" %}
If you do not follow these steps, you will not be able to receive events from Stripe. This will prevent you from being able to update the subscription status of your users. Please make sure to follow these steps.
{% /alert %}

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
        },
        {
          name: 'Yearly',
          price: '$90',
          stripePriceId: '<price_id>',
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
        },
        {
          name: 'Yearly',
          price: '$200',
          stripePriceId: 'pro-plan-yr'
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
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ],
}
```

{% alert type="warning" title="The properties "stripePriceId" are placeholders" %}
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

```bash
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


