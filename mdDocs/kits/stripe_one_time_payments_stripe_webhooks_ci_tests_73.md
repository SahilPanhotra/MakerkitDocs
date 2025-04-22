-----------------
FILE PATH: kits\remix-fire\payments\stripe-one-time-payments.mdoc

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
FILE PATH: kits\remix-fire\payments\stripe-webhooks.mdoc

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

-----------------
FILE PATH: kits\remix-fire\testing\ci-tests.mdoc

---
status: "published"

title: CI Tests
label: CI Tests
order: 1
description: 'Learn how to test your Makerkit in your CI'
---


To run your tests in any CI, we recommend you simply run the following command:

```
npm test
```

This command takes care of running all the required services, importing the test data, executing the tests and then exiting.

### The Makerkit testing pipeline

As mentioned in the previous section, testing a Makerkit requires a few services to be up and running:

- the Remix server
- the Firebase emulators
- optionally, the Stripe Mock server

While executing these tests in your CI, this can get cumbersome. Therefore, we provided a single script to execute all your tests in one simple command: you can find the script at `scripts/test.sh`, which will contain the following content:

```
set -e

npm run dev:test & npm run stripe:mock-server &
npm run cypress:headless
sh scripts/kill-ports.sh
```

Let's explain it, so you can add your own modifications to it.

The first command will run (in parallel and in the background) the Remix server and the Stripe mock server:

```
npm run dev:test & npm run stripe:mock-server &
```

Afterward, we execute the Cypress tests in headless mode:

```
npm run cypress:headless
```

Finally, we clean up all the ports that may have remained open after executing the script:

```
sh scripts/kill-ports.sh
```

If you update your ports, remember to change them also in `scripts/kill-ports.sh`.

### Running the Tests in CI mode

Finally, the tests can be executed with the command `npm test`.

This command **will automatically run the Firebase emulators** and the `test.sh` script, and is defined as:

```
"test": "firebase emulators:exec --project demo-makerkit --import ./firebase-seed \"sh ./scripts/test.sh\"",
```

When finished, the emulators will exit.


-----------------
FILE PATH: kits\remix-fire\testing\running-tests.mdoc

---
status: "published"

title: Running Cypress Tests with Firebase and Remix
label: Running Tests
order: 0
description: 'Learn how to run Cypress E2E tests for your Makerkit application'
---


The Makerkit SaaS boilerplate comes with a set of predefined tests with the following goals:

1. Verify that the boilerplate application works well
2. Provide instructions for building your own tests

As your application grows, the tests will inevitably need to be updated and
maintained. We hope that our tests will be simple to understand and to
change when you will need them.

## Running the Remix development server

First, we have to run the following command to run Remix with a testing
environment (i.e. using `.env.test`):

```
npm run dev
```

This command will start a Remix dev server at http://localhost:3000, so
it's important to make sure you have no other Remix server running at the
same port.

Additionally, we have to start the **Firebase Emulator server**:

```
npm run firebase:emulators:start
```

## Running Cypress in development mode

While you build your tests, it can be handy to use a windowed version of the
Cypress testing suite, so that you can easily retry the tests without having
to re-run the while tests suite.

To do that, run the following command:

```
npm run cypress
```

## Running All Cypress tests

Whenever you want to run all your tests at once, you can use the following commands:

```
npm run cypress:headless
```

This will run all the tests and exit.

## Running Cypress in CI mode

Whenever you want to run all your tests in a CI
environment, you can use the following commands:

```
npm test
```

The command above will:

1. Start the Firebase Emulator
2. Start the Remix environment
3. Run the tests in headless mode
4. Exit and close all the processes

As you may have noticed, this is the command to run in your CI environment.

## Testing Stripe

Testing Stripe requires one more process to start, i.e. the official Stripe
emulator.

The command below **will require Docker installed**, which is why you can
choose to disable it by setting the environment variable `ENABLE_STRIPE_TESTING`
to `false` from the `.env.test` environment file:

 ```bash {% title=".env.test" %}
ENABLE_STRIPE_TESTING=false
```

If instead, you want to test Stripe Checkout as well, run the command below:

```
npm run stripe:mock-server
```

As we've mentioned, this will require Docker running. Of course, we think
it's worth it.


-----------------
FILE PATH: kits\remix-fire\testing\testing-limitations.mdoc

---
status: "published"

title: Testing Limitations
label: Limitations
order: 2
description: ''
---


Due to the hard-to-rollback nature of some tests (such as accepting invitations) - some Tests won't work well in development mode.

To have a clean testing environment at each run, we recommend running the following command:

```
npm test
```

Of course, some of the tests will work just fine, but not the ones related to accepting invites.

This would be solved if we could reset the Firebase Firestore data programmatically between test runs, but as far as we know this is not yet possible.

-----------------
FILE PATH: kits\remix-fire\tutorials\api-routes-validation.mdoc

---
status: "published"

title: "API Routes Validation"
label: "API Routes Validation"
order: 14
description: "The best practices to validate your API routes payloads using Zod in your Remix Firebase application."
---


Validating payloads is necessary to ensure your API endpoints receive the expected data. To validate the API, we use Zod.

[Zod](https://github.com/colinhacks/zod) is a Typescript library that helps us
secure our API endpoints by validating the payloads sent
from the client and also facilitating the typing of the payload with
Typescript.

Using Zod is the first line of defense to validate the data sent against our
API: as a result, it's something we recommend you keep doing. It ensures we
write safe, resilient, and valid code.

When we write an API endpoint, we first define the schema of the payload:

```tsx
function getBodySchema() {
  return z.object({
    displayName: z.string(),
    email: z.string().email(),
  });
}
```

This function represents the schema, which will validate the following
interface:

```tsx
interface Body {
  displayName: string;
  email: Email;
}
```

Now, let's write the body of the API handler that validates the body of the
function, which we expect to be equal to the `Body` interface.

```tsx
import { throwBadRequestException } from `~/core/http-exceptions`;

export const action: ActionFunction = async ({ request }) => {
  try {
     // we can safely use data with the interface Body
    const schema = getBodySchema();
    const body = await request.json();
    const { displayName, email } = schema.parse(body);

    return sendInvite({ displayName, email });
  } catch(e) {
    return throwBadRequestException();
  }
}
```

I encourage you to never skip the validation step when writing your API endpoints.

-----------------
FILE PATH: kits\remix-fire\tutorials\api-routes.mdoc

---
status: "published"

title: "API Routes"
label: "API Routes"
order: 13
description: "Learn how to create and manage API routes in your Remix Firebase
application."
---


Makerkit provides some utilities to reduce the boilerplate needed to write Remix API functions. This section will teach you everything you need to know to write your API functions.

### Calling API functions from the client

To make requests to the API functions in the `api` folder, we provide a
custom hook `useApiRequest`, a wrapper around `fetch`.

It has the following functionality:

1. Uses an internal `state` to keep track of the state of the request, like
`loading`, `error` and `success`. This helps you write fetch requests the
"hooks way"
2. Automatically adds a `Firebase AppCheck Token` to the request headers if you
enabled Firebase AppCheck
3. Automatically adds a `CSRF token` to the request headers

Similarly to making Firestore Requests, it's a convention to create a custom
hook for each request for readability/reusability reasons.

 ```tsx {% title="src/core/hooks/use-create-session.ts" %}
import useApiRequest from '~/core/hooks/use-api';

interface Body {
  idToken: string;
}

export function useCreateSession() {
  return useApiRequest<void, Body>('/api/session/sign-in');
}
```

The hook returns an array:

1. **Callback**: the first element is the **callback** to make the request
2. **State**: the second element is the **state object** of the request

```tsx
const [createSession, createSessionState] = useCreateSession();
```

The state object internally uses `useApiRequest`, and has the following
interface:

```tsx
{
  success: boolean;
  loading: boolean;
  error: string;
  data: T | undefined;
}
```

When `success` is `true`, the property `data` is inferred with its correct type.

You can use this hook in your components:

```tsx
import { useCreateSession } from '~/core/hooks/use-create-session';

function Component() {
  const [createSession, createSessionState] = useCreateSession();

  return (
    <>
      { createSessionState.loading ? `Loading...` : null }
      { createSessionState.error ? `Error :(` : null }
      { createSessionState.success ? `Yay, success!` : null }

      <SignInForm onSignIn={(idToken) => createSession({ idToken })} />
    </>
  );
}
```

Similarly, you can also use it to fetch data:

```tsx
import { useCreateSession } from '~/core/hooks/use-create-session';

function Component() {
  const [fetchMembers, { loading, error, data }]
    = useFetchMembers();

  // fetch data when the component mounts
  useEffect(() => {
    fetchMembers();
  }, [fetchMembers]);

  if (loading) {
    return <div>Fetching members...</div>;
  }

  if (error) {
    return <div>{error}</div>;
  }

  return (
    <div>
      {data.map(member => <MemberItem member={member} />)}
    </div>
  );
}
```

#### Writing your own Fetch Hook

You are free to use your own implementation for sending HTTP requests to your API or use a powerful third-party library.

With that said, you need to ensure that you're sending the required headers:
1. The AppCheck token (if you use Firebase AppCheck)
2. The CSRF Token (for POST requests)

##### Generating an App Check Token

To generate an App Check token, you can use the `useGetAppCheckToken` hook:

```tsx
const getAppCheckToken = useGetAppCheckToken();
const appCheckToken = await getAppCheckToken();

console.log(appCheckToken) // token
```

You will need to send a header `X-Firebase-AppCheck` with the resolved value returned by the promise `getAppCheckToken()`.

##### Sending the CSRF Token

To retrieve the page's CSRF token, you can use the `useGetCsrfToken` hook:

```tsx
const getCsrfToken = useGetCsrfToken();
const csrfToken = getCsrfToken();

console.log(csrfToken) // token
```

You will need to send a header `x-csrf-token` with the value returned by `getCsrfToken()`.

### CSRF Token check

We must pass a `CSRF token` when using POST requests to prevent malicious attacks. To do so, we use the pipe `withCsrfToken`.

1. The CSRF token is generated when the page is server-rendered
2. The CSRF token is stored in a `meta` tag
3. The CSRF token is sent to an HTTP POST request automatically when using the `useApiRequest` hook
4. This function will throw an error when the token is invalid

By using this function, **we ensure all the following functions will not be executed unless the token is valid**.

```tsx
export const action: ActionFunction = async ({ request }) => {
  await withCsrf(request);
}
```

### Firebase App Check

App Check is a Firebase service that helps you to protect your web app from
bots, spammy users, and general abuse detected by Google's Recaptcha.

[Check out the documentation to setup Firebase App Check](/docs/next-fire/app-check).

Using the `withAppCheck` pipe, **we ensure all the following functions will not be executed unless the App Check token is valid**.

```tsx
export const action: ActionFunction = async ({ request }) => {
  await withAppCheck(request);
}
```



### Catching and Handling Exceptions

To catch and gracefully handle API exceptions, we use the function `withExceptionFilter`.

As seen from its usage above, we wrap the API function within the utility. When errors are caught, this function will:

1. Log and debug the error to the console
2. Report the error to Sentry (if configured)
3. Return an object stripping the error's data to avoid leaking information

### API Logging

The boilerplate uses `Pino` for API logging, a lightweight logging utility for Node.js.

Logging is necessary to debug your applications and understand what happens when things don't behave as expected. You will find various instances of logging throughout the API, but we encourage you to log more if necessary.

We import the logging utility from `~/core/logger`. Typically, you can log every time you perform an action, both before and after. For example:

```tsx
async function myFunction(params: {
  organizationId: string;
  userId: string;
}) {
  logger.info(
    {
      organizationId: params.organizationId,
      userId: params.userId,
    },
    `Performing action...`
  );

  await performAction();

  logger.info(
    {
      organizationId: params.organizationId,
      userId: params.userId,
    },
    `Action successful`
  );
}
```

It's always important to add context to your logs: as you can see, we use the first parameter to add important information.


-----------------
FILE PATH: kits\remix-fire\tutorials\application-pages.mdoc

---
status: "published"

title: "Application Pages"
label: "Application Pages"
order: 12
description: "Learn how to create and manage application pages in your Makerkit app"
---


Now that we have some components to display, we need to add them to the actual Remix pages.

If you have not created the files in the `Routing` section above, it's time to do it.

#### Using the components AppHeader and AppContainer

These components are used to wrap the content of the pages. They are responsible for displaying the header and the container of the page.
```tsx
function DashboardPage() {
  return (
    <>
      <AppHeader>
        <span className={'flex space-x-2'}>
          <Squares2X2Icon className="w-6" />

          <span>
            <Trans i18nKey={'common:dashboardTabLabel'} />
          </span>
        </span>
      </AppHeader>

      <AppContainer>
        Your content goes here
      </AppContainer>
    </>
  );
}
```


-----------------
FILE PATH: kits\remix-fire\tutorials\authentication.mdoc

---
status: "published"

title: "Authentication"
label: Authentication
order: 7
description: "Learn about the authentication strategies supported by Makerkit and how to configure them."
---


The Remix/Firebase template uses Firebase Auth to manage authentication into the internal application.

The kit supports the following strategies:

1. Email/Password
2. oAuth Providers such as Google, Twitter, Facebook, Microsoft, Apple, etc.
3. Phone Number
4. Email Link

You can choose one, more, or all of them together, and you can easily tweak this using the global configuration.

By default, our SaaS Starter uses Email/Password and Google Auth.

### How does Authentication work?

First, let's just take a high-level overview of how Makerkit's authentication works.

MakerKit **uses SSR throughout the application**, except for the marketing pages. Using SSR, we can persist the user's authentication on every page and access the user's object on the server **before rendering the page**.

This can help you both in terms of UX and DX. In fact, persisting the user's session server-side can help you in various scenarios:

- Simplifying the business logic: for example, checking server-side if a user can access a specific page before rendering that page
- Rendering the user's data server-side by fetching some data from Firestore in the `getServerSideProps` function

#### How does SSR work?

To enable SSR with Firebase Auth, we use [Session Cookies](https://firebase.google.com/docs/auth/admin/manage-cookies).

1. First, we use the Firebase Auth Client SDK to sign users in, as described in every tutorial. Furthermore, when a user signs in or signs up, we retrieve the ID Token and make a request to our API to create a session cookie. The session cookie remains valid for 14 days, after which it will expire.
2. The cookie `sessionId` gets stored as an HTTP-only cookie: we can use this server-side for fetching the current user.
3. When the token expires, we sign users out of the application

So... how do they work?

1. The client SDK uses the Firebase Auth credentials stored in the IndexedDB Storage. This includes interacting with Firebase Storage and Firebase Firestore on the client side.
2. The server-side API uses the `sessionId` cookie to retrieve and authenticate the current user. We can then use the Firebase Admin API and interact with the various Firebase services.

### Authentication Strategies

To add or tweak the authentication strategies of our application, we can update the configuration:

 ```tsx {% title="src/configuration.ts" %}
auth: {
  // Enable MFA. You must upgrade to GCP Identity Platform to use it.
  // see: https://cloud.google.com/identity-platform/docs/product-comparison
  enableMultiFactorAuth: false,
  // NB: Enable the providers below in the Firebase Console
  // in your production project
  providers: {
    emailPassword: true,
    phoneNumber: false,
    emailLink: false,
    oAuth: [GoogleAuthProvider],
  },
},
```

Above, you can see the **default configuration**. It should look like the following:

{% img src="/assets/images/posts/tutorial-sign-in.webp" width="2834" height="1972" /%}

Ok, cool. But what if we wanted to swap `emailPassword` with `emailLink` and add `Twitter oAuth`?

We will do the following:

 ```tsx {% title="src/configuration.ts" %}
import { GoogleAuthProvider, TwitterAuthProvider } from 'firebase/auth';

auth: {
  // Enable MFA. You must upgrade to GCP Identity Platform to use it.
  // see: https://cloud.google.com/identity-platform/docs/product-comparison
  enableMultiFactorAuth: false,
  // NB: Enable the providers below in the Firebase Console
  // in your production project
  providers: {
    emailPassword: false, // set this to false
    phoneNumber: false,
    emailLink: true, // set this to true
    oAuth: [GoogleAuthProvider, TwitterAuthProvider],
  },
},
```

And the result will be similar to the image below:

{% img src="/assets/images/posts/tutorial-sign-in-email-link.webp" width="2834" height="1972" /%}

#### Custom oAuth providers

Additionally, we can add custom oAuth providers, such as `Microsoft` and `Apple`:

```tsx
class MicrosoftAuthProvider extends OAuthProvider {
  constructor() {
    super('microsoft.com');
  }
}

class AppleAuthProvider extends OAuthProvider {
  constructor() {
    super('apple.com');
  }
}
```

Then, you will add these classes to the `auth.providers.oAuth` object of the configuration.

{% alert type="warning" %}
Remember that you will always need to enable the authentication methods you want to use from the Firebase Console once you deploy your application to production
{% /alert %}

### Creating Accounts

Users can be redirected to the [Sign Up page](http://localhost:3000/auth/sign-up) to create an account.

### Requiring Email Verification

By default, Firebase does not require users to verify their email address before being able to access the application. It is a flag on the user account, but the user is not prevented from accessing the application.

We solve this server-side by checking if the user has verified their email address before rendering the page.

To enable this feature, you can set the `requireEmailVerification` flag to `true` using the following environment variable:

 ```bash {% title=".env" %}
NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION=true
```

-----------------
FILE PATH: kits\remix-fire\tutorials\deploying-production.mdoc

---
status: "published"

title: "Deploying to Production"
label: "Deploying to Production"
order: 16
description: "Learn how to deploy your Remix Firebase app to your hosting provider."
---


Before deploying to production or any remote server, you need to [follow all the steps outlined in the documentation](/docs/remix-fire/going-to-production-overview).

**Much of this work needs to be done in Firebase**, not in the Makerkit codebase.

Generally, you will need to:

1. **Firebase**: Create a new Firebase project (database, storage bucket, getting the private keys)
2. **Security Rules**: Update the remote security rules (both Firestore and Storage)
3. **Environment Variables**: Ensure that your environment variables are set correctly, and that you've added the relevant environment variables to your hosting provider
4. **Auth**: Enable the authentication providers you want to use from the Firebase Console
5. **Indexes**: Update all the Firestore indexes required by your application in the Firebase Console
6. **SMTP**: Set up an SMTP server to send emails by adding the required configuration to `src/configuration.ts`
7. **Payments**: Set up your Stripe or Lemon Squeezy accounts and add the relevant environment variables
8. **Deployment**: Finally, deploy your application to your hosting provider (Vercel, Firebase, Netlify, etc.): please follow the instructions provided by your hosting provider to deploy a Remix application.

