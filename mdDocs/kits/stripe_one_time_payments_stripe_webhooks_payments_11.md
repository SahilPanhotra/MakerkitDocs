-----------------
FILE PATH: kits/next-fire/payments/stripe-one-time-payments.mdoc

---
status: "published"
title: Enabling One-Time Payments with Stripe | Next.js Firebase
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

{% alert type="warning" title="Make sure to remove the subscription_data parameter" %}
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
FILE PATH: kits/next-fire/payments/stripe-webhooks.mdoc

---
status: "published"
title: "How Makerkit handles Stripe Webhooks | Next.js Supabase SaaS Kit"
label: Stripe Webhooks
order: 1
description: 'Learn how MakerKit handles Stripe Webhooks in your Next.js Supabase application'
---

Makerkit handles five webhooks from Stripe and stores "just enough" data into the organization's entity to function correctly.

The Stripe's webhooks topics are defined here at `src/core/stripe/stripe-webhooks.enum.ts`:

```tsx
export enum StripeWebhooks {
  Completed = 'checkout.session.completed',
  SubscriptionDeleted = 'customer.subscription.deleted',
  SubscriptionUpdated = 'customer.subscription.updated',
}
```

To handle webhooks, we created an API route at `pages/api/stripe/webhook.ts`.

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

-----------------
FILE PATH: kits/next-fire/payments.mdoc

---
status: "published"
title: "Payments in Next.js Firebase"
label: Payments
order: 6
description: "Learn how to disable Stripe in your Makerkit project."
collapsible: true
collapsed: true
---

In this section, you will learn everything you need to know about payments in Next.js Firebase.

By default, Makerkit uses Stripe to handle payments. However, you can also switch to Lemon Squeezy instead.

- [Stripe Configuration](payments/stripe-configuration): this is the guide to set up Stripe in your project
- [Stripe Webhooks](payments/stripe-webhooks): this is the guide to set up Stripe webhooks
- [Disable Stripe](payments/disable-stripe): this is the guide to disable Stripe
- [Lemon Squeezy](payments/lemon-squeezy): this is the guide to set up Lemon Squeezy instead of Stripe

-----------------
FILE PATH: kits/next-fire/plugins/ai-chatbot.mdoc

---
status: "published"

title: "AI Chatbot Plugin for the Next.js Firebase Saas Starter Kit"
label: "AI Chatbot"
order: 2
description: "This plugin adds an AI chatbot to your Next.js Firebase SaaS application."
---


This plugin adds an AI Chatbot trained on your website content to your Next.js
application built using the OpenAI API. It's a simple component that will be
displayed on the bottom right of your website.

This documentation is for the Next.js Firebase version. Please make sure you're looking at the right documentation.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```bash
git subtree add --prefix plugins/chatbot git@github.com:makerkit/next-firebase-saas-kit-plugins.git chatbot --squash
```

After running this command, you will have the plugin in your repository at
`plugins/chatbot`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```bash
npx @makerkit/cli@latest plugins install
```

Follow the instructions to install the plugin.

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:

1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/chatbot https://github.com/makerkit/next-firebase-saas-kit-plugins chatbot --squash
```

#### Add the plugin as a workspace in your package.json

You can do so by adding the following to your `package.json` file:

```json
{
  "workspaces": [
    "plugins/chatbot"
  ]
}
```

Add it next to the other workspaces in your `package.json` file. Please
don't miss this step - otherwise, the plugin will install duplicate node
modules and they will conflict with each other.

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

### Update TailwindCSS configuration

To make sure that the plugin's styles are applied, you will need to update
your TailwindCSS configuration.

Add the plugins path to the `content` array of your `tailwind.config.js` file:

```js
module.exports = {
  // ...
  content: [
    // ...
    './plugins/**/*.tsx',
  ],
  // ...
}
```

This is only needed the first time you install any plugin.

### Installing dependencies

To install the dependencies, you can run the following command:

```bash
npm i
```

NPM will install the dependencies in the `plugins/chatbot` folder as an NPM workspace.

### Importing the Plugin

You can import the `Chatbot` component from where you want to display it.

```tsx
import dynamic from 'next/dynamic';

const ChatBot = dynamic(() => import('~/plugins/chatbot/components/ChatBot'), { ssr: false });

export default function App() {
  return (
    <>
      <ChatBot />
      {/* ... */}
    </>
  );
}
```

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

```bash
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

We will use the App Directory to create a route handler for the chatbot - so we can leverage streaming.

#### Next.js Route

Create a route at `src/app/api/chat/route.ts` with the following content:

```ts
import handleChatBotRequest from '~/plugins/chatbot/lib/server/route-handler';

export const POST = handleChatBotRequest;
```

NB: make sure you're using the new App Directory (located at `src/app`) and
not the Pages Directory (located at `src/pages`). This is because the Pages
Directory does not support streaming.

Feel free to change the route to whatever you want - but make sure to update
the route in the `ChatBot` component.

## Creating a Vector Store with LangChain

Given the variety of possible Databases - we don't provide a database vector
store by default. Instead, we use [LangChain](https://js.langchain.com/) -
which provides an exhaustive list of Vector Store integrations out of the box.

Therefore - you will need to choose a Vector Store based on the technology
you want to use (Pinecone, Supabase, Chroma, TypeSense, etc.) and follow the
[Vector Store integration instructions](https://js.langchain.com/docs/modules/data_connection/vectorstores/).

Once you've decided which Vector Store you want to use, export a default
vector store from `~/plugins/chatbot/lib/server/vector-store.ts`:

### Example: Using Typesense as a Vector Store

For example, if you want to use TypeSense, you can create the following
vector store using the `typesense` client and the Langchain integration:

```ts
import { Typesense, TypesenseConfig } from 'langchain/vectorstores/typesense';
import { OpenAIEmbeddings } from 'langchain/embeddings/openai';
import { Client } from 'typesense';

const TYPESENSE_URL = process.env.TYPESENSE_URL;
const TYPESENSE_HOST = process.env.TYPESENSE_HOST || 'localhost';
const TYPESENSE_PORT = process.env.TYPESENSE_PORT || 8108;
const TYPESENSE_API_KEY = process.env.TYPESENSE_API_KEY || 'xyz';

const TYPESENSE_PROTOCOL =
  TYPESENSE_HOST.includes('localhost') ? 'http' : 'https';

const nodes = TYPESENSE_URL
  ? [
    {
      url: TYPESENSE_URL,
    },
  ]
  : [
    {
      host: TYPESENSE_HOST,
      port: Number(TYPESENSE_PORT),
      protocol: TYPESENSE_PROTOCOL,
    },
  ];

const typeSense = new Client({
  nodes,
  apiKey: TYPESENSE_API_KEY,
  numRetries: 2,
  connectionTimeoutSeconds: 5,
});

const TYPESENSE_SCHEMA_NAME =
  process.env.TYPESENSE_SCHEMA_NAME || 'chatbot_documents';

const typesenseVectorStoreConfig: TypesenseConfig = {
  typesenseClient,
  schemaName: TYPESENSE_SCHEMA_NAME,
  onFailedAttempt: (error) => {
    console.error(error);
  },
  import: async (data, collectionName) => {
    await typesenseClient
      .collections(collectionName)
      .documents()
      .import(data, { action: 'emplace', dirty_values: 'drop' });
  },
} satisfies TypesenseConfig;

export default function getVectorStore() {
  return new Typesense(
    new OpenAIEmbeddings({
      openAIApiKey: process.env.OPENAI_API_KEY,
    }),
    typesenseVectorStoreConfig,
  );
}
```

Don't forget to install the required dependencies for your Vector Store.

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

```bash
npx tsx plugins/chatbot/cli.ts generate
```

Follow the instructions to generate the embeddings.

If you are generating the embeddings for your production environment, you
will need to provide the Vector DB credentials in the `.env.production` file.

### Captcha Protection (optional) (WIP)

To protect your chatbot from bots (ha ha!), you can add a captcha challenge
using [Cloudflare's Turnstile](https://developers.cloudflare.com/turnstile/)
service.

To do so, you will need to add the following environment variables to your
environment files:

```bash
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

```bash
git subtree pull --prefix plugins/chatbot git@github.com:makerkit/next-firebaase-saas-kit-plugins.git chatbot --squash
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

-----------------
FILE PATH: kits/next-fire/plugins/cookie-banner.mdoc

---
status: "published"

title: "Cookie Banner Plugin for the Next.js Firebase Saas Starter Kit"
label: "Cookie Banner"
order: 0
description: "This plugin adds a cookie banner to your Next.js application. It's a simple component that will store the user's consent in a cookie. This banner will stop appearing once the user has given their consent - or rejected it."
---


This plugin adds a cookie banner to your Next.js application. It's a simple
component that will store the user's consent in a cookie. This banner will
stop appearing once the user has given their consent - or rejected it.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```bash
git subtree add --prefix plugins/cookie-banner git@github.com:makerkit/next-firebase-saas-kit-plugins.git cookie-banner --squash
```

After running this command, you will have the plugin in your repository at
`plugins/cookie-banner`. Once pulled, you can apply any customization you need.

Alternatively, you can use the [Makerkit CLI](https://github.com/makerkit/cli):

```bash
npx @makerkit/cli@latest plugin add cookie-banner
```

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:
1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/cookie-banner https://github.com/makerkit/next-firebase-saas-kit-plugins cookie-banner --squash
```

### Update TailwindCSS configuration

To make sure that the plugin's styles are applied, you will need to update
your TailwindCSS configuration.

Add the plugins path to the `content` array of your `tailwind.config.js` file:

```js
module.exports = {
  // ...
  content: [
    // ...
    './plugins/**/*.tsx',
  ],
  // ...
}
```

This is only needed the first time you install any plugin.

### Translations

Add the translations to your `public/locales/en/common.json` file:

```json
"cookieBanner": {
  "title": "Hi, we use cookies.",
  "description": "This website uses cookies to ensure you get the best
experience on our website.",
  "reject": "Reject",
  "accept": "Accept"
}
```

Feel free to update the translations to your liking, this is just an example.

### Importing the Plugin

You can import the `CookieBanner` component from the plugin in your `_app.
tsx` file and add it to your application:

```tsx
import dynamic from 'next/dynamic';

const CookieBanner = dynamic(
    () => import('~/plugins/cookie-banner/CookieBanner'),
    {
        ssr: false,
    },
);

export default function App() {
  return (
    <>
      <CookieBanner />
      {/* ... */}
    </>
  );
}
```

## API

To retrieve the current consent status, you can use the `useCookieConsent`:

```tsx
import { useCookieConsent } from 'plugins/cookie-banner';

function MyComponent() {
    const consent = useCookieConsent();
    // ...
}
```

You can use this hook to decide whether to load third-party scripts or not.
For example, you can use it to load Google Analytics only if the user has consented:

```tsx
import { useCookieConsent } from 'plugins/cookie-banner';

function MyAnalyticsScript() {
  const consent = useCookieConsent();

  if (consent === 'accepted') {
    return (
      <script
        async
        src="https://www.googletagmanager.com/gtag/js?id=UA-XXXXXX-X"
      />
    );
  }

  return null;
}
```

#### Keeping the plugin up to date

To keep the plugin up to date, you can use git subtrees again:

```bash
git subtree pull --prefix plugins/cookie-banner git@github.com:makerkit/next-firebase-saas-kit-plugins.git cookie-banner --squash
```

-----------------
FILE PATH: kits/next-fire/plugins/feedback-popup.mdoc

---
status: "published"

title: "Feedback Popup Plugin for the Next.js Firebase Saas Starter Kit"
label: "Feedback Popup"
order: 1
description: "This plugin adds a feedback popup to your Next.js application. You can import this component anywhere in your application. Also - this plugin provides some admin components to manage and view the feedback submitted by your users."
---


This plugin adds a feedback popup to your Next.js application. You can
import this component anywhere in your application.

Also - this plugin provides some admin components to manage and view the
feedback submitted by your users.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```bash
git subtree add --prefix plugins/feedback-popup git@github.com:makerkit/next-firebase-saas-kit-plugins.git feedback-popup --squash
```

After running this command, you will have the plugin in your repository at
`plugins/feedback-popup`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```bash
npx @makerkit/cli@latest plugins install
```

Follow the instructions to install the plugin.

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:
1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/feedback-popup https://github.com/makerkit/next-firebase-saas-kit-plugins feedback-popup --squash
```

#### Add the plugin as a workspace in your package.json

You can do so by adding the following to your `package.json` file:

```json
{
  "workspaces": [
    "plugins/feedback-popup"
  ]
}
```

Add it next to the other workspaces in your `package.json` file.

#### Add the paths alias to the TypeScript configuration

To make sure that the TypeScript compiler can find the plugin, you will need
to add the following paths alias to your `tsconfig.json` file, in addition
to the other paths aliases that you may have.

If not yet present, add the following to your `tsconfig.json` file:

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

You only need to add this once, even if you have multiple plugins.

### Update TailwindCSS configuration

To make sure that the plugin's styles are applied, you will need to update
your TailwindCSS configuration.

Add the plugins path to the `content` array of your `tailwind.config.js` file:

```js
module.exports = {
  // ...
  content: [
    // ...
    './plugins/**/*.tsx',
  ],
  // ...
}
```

This is only needed the first time you install any plugin.

### Installing dependencies

To install the dependencies, you can run the following command:

```bash
npm i
```

NPM will install the dependencies in the `plugins/feedback-popup` folder as an
NPM workspace.

### Importing the Plugin

You can import the `Chatbot` component from the plugin in your `app/layout.tsx`
file if you want it available on all pages:

```tsx
import { FeedbackPopupContainer } from '~/plugins/feedback-popup/FeedbackPopup';

export default function Component() {
  return (
    <>
      <FeedbackPopupContainer>
        <Button variant='outline'>Feedback</Button>
      </FeedbackPopupContainer>
    </>
  );
}
```

You can wrap any component with the `FeedbackPopupContainer` component to
trigger the feedback popup. For example, an icon:

```tsx
import { ChatBubbleLeftIcon } from '@heroicons/react/24/outline';
import { FeedbackPopupContainer } from '~/plugins/feedback-popup/FeedbackPopup';

export default function Component() {
  return (
    <>
      <FeedbackPopupContainer>
        <Button size="icon" variant='ghost'>
          <ChatBubbleLeftIcon className={'h-6'} />
        </Button>
      </FeedbackPopupContainer>
    </>
  );
}
```

Check out this repository's `SiteHeader` component for an example of how it's implemented.

### Adding Admin Routes

We need to import the routes from the plugin to the admin application.

#### Copy the admin routes files

Some of the admin routes are provided by the plugin. Now we have to copy
them over to our own repository.

1. Locate the file at `src/pages/admin/feedback/page.tsx` and paste
the same content into your own repository.
2. Do the same for the file at `src/pages/admin/feedback/[id].tsx`

Finally, add the page to the sidebar by updating the `AdminSidebar` component:

```tsx
<SidebarItem
  path={'/admin/feedback'}
  Icon={() => <ChatBubbleLeftRightIcon className={'h-6'} />}
>
  Feedback
</SidebarItem>
```

### Copy the API routes

The plugin provides some API routes to manage the feedback. We need to copy
them over to our own repository.

First, create a route at `src/pages/api/feedback/page.tsx` and paste the
following content:

```tsx
import { addFeedbackSubmission } from '~/plugins/feedback-popup/lib/mutations';

export default addFeedbackSubmission;
```

Then, create a route at `src/pages/api/admin/feedback/[id].ts` and paste the content:

```tsx
import { deleteFeedbackSubmission } from '~/plugins/feedback-popup/lib/mutations';

export default deleteFeedbackSubmission;
```

---


That's it! You should now be able to access the admin pages at `/admin/feedback` and `/admin/feedback/[id]`.

### Emulator Limitations

The emulator does not support creating signed URLs for the images. This
means that we cannot display the uploaded images during development.

-----------------
FILE PATH: kits/next-fire/plugins.mdoc

---
status: "published"
title: "Plugins for Next.js Firebase"
label: "Plugins"
order: 13
description: "Makerkit provides various plugins you can use to extend your Next.js Firebase application"
collapsible: true
collapsed: true
---

Makerkit provides various plugins you can use to extend your Next.js Firebase application.

