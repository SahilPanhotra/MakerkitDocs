-----------------
FILE PATH: kits/next-supabase/sdk/organization-sdk.mdoc

---
status: "published"

title: "Organization SDK - Fetch Organization"
label: "Organization SDK"
order: 2
description: "Learn how to use the Organization SDK to interact with the current user organization"
---


The Organization SDK allows you to interact with the currently selected Organization. Here we list the methods available in the Organization SDK.

The example below work within a Server Components: by using the required client, you can easily adapt the code examples below to both Server Actions and Route Handlers.

The Organization's SDK is namespaced under `sdk.organization` - so all the methods below are available under `sdk.organization`.

## Fetch the current Organization UID

To fetch the currently selected user's organization UID, use the `getCurrentOrganizationUid` method:

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const organizationUid = await sdk.organization.getCurrentOrganizationUid();
}
```

The UID may be `null` - so do make sure to check for that. For example, you may want to redirect the user to the Organizations page if no organization is selected:

```tsx
import { redirect } from "next/navigation";

import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const organizationUid = await sdk.organization.getCurrentOrganizationUid();

  if (!organizationUid) {
    // No organization selected
    redirect('/organizations')
  }
}
```

## Fetch the current Organization

To fetch the currently selected user's organization, use the `getCurrent` method:

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const organization = await sdk.organization.getCurrent();
}
```

The `getCurrent` method returns the interface `GetCurrentOrganizationResponse` which is defined as:

```ts
interface GetCurrentOrganizationResponse {
  organization: {
    id: number;
    uuid: string;
    name: string;
    logoURL?: string | null;

    // your custom fields...

    subscription?: {
      customerId: string | undefined;

      // your custom fields...

      data: {
        id: string;

        priceId: string;

        status: Stripe.Subscription.Status;
        cancelAtPeriodEnd: boolean;
        currency: string | null;

        interval: string | null;
        intervalCount: number | null;

        createdAt: string;
        periodStartsAt: string;
        periodEndsAt: string;
        trialStartsAt: string | null;
        trialEndsAt: string | null;

        // your custom fields ...
      }
    }
  }
}
```

{% alert type="warning" title="This can change if you have a custom Organization model." %}


  It's very likely that you will be updating the Organization model to include more fields. If that's the case, this
  method will reflect those changes.
{% /alert %}

## Fetch the current Organization's Subscription

The `organization` namespace also includes a `subscription` namespace. This namespace allows you to interact with the current Organization's subscription.

We will show it in the next section.

-----------------
FILE PATH: kits/next-supabase/sdk/organization-subscription-sdk.mdoc

---
status: "published"

title: "Organization Subscription SDK - Fetch the Organization Subscription"
label: "Organization Subscription SDK"
order: 3
description: "Learn how to use the Organization Subscription SDK to fetch the current organization Subscription details"
---


The Subscription SDK helps with getting details about the current organization Subscription. The Subscription is available in the `organization` namespace of the SDK

## Get the current organization Subscription

To retrieve the current organization Subscription, use the `organization.getSubscription()` method to first get the current organization, and then get the Subscription from the organization.

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();

  // ...
}
```

## Check if the current organization has an active subscription

To check if the current organization has an active subscription, use the `subscription.isActive()` method.

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();
  const isActive = await sdk.organization.isActive(); // false|true

  // ...
}
```

This method will return `true` in two cases:
- the status of the subscription is `active`
- the status of the subscription is `trialing`

## Check if it's a Trial Subscription

To check if the current organization is a trial, use the `subscription.isTrial()` method.

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();
  const isTrial = await sdk.organization.isTrial(); // false|true

  // ...
}
```

This method will return `true` when the status of the subscription is `trialing`.

## Check the status of the Subscription

To check the status of the current organization Subscription, use the `subscription.status` computed property.

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();
  const status = await sdk.organization.status; // Stripe.Subscription.Status

  // ...
}
```

## Check the status of the Subscription is a certain status

To check the status of the current organization Subscription, use the `subscription.isStatus` method. The method accepts a single argument, which is the status to check against. This is the `Stripe.Subscription.Status` enum.

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();
  const isActive = await sdk.organization.isStatus('active');

  // ...
}
```


-----------------
FILE PATH: kits/next-supabase/sdk/sdk.mdoc

---
status: "published"

title: "The Makerkit SDK - an easy API for your Makerkit projects"
label: The Makerkit SDK
order: 0
description: "The Makerkit SDK is a simple API for your Makerkit projects. It allows you to easily interact with the Makerkit core entities"
---


The Makerkit SDK is a simple API for your Makerkit projects. It allows you to easily interact with the Makerkit core entities, such as: Users, Organizations, Subscriptions and Memberships.

As you build your application, you will need to interact with the core entities. The SDK provides a simple way to do that.


## Usage

Initializing the SDK is simple. You will need to provide any of the Supabase Server clients - depending on where you are using the SDK (route, server component or server action).

### Server Components

In the example below, we initialize the SDK in a Server Component

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);

  // ...
}
```

### Server Actions

In the example below, we initialize the SDK in a Server Action

```tsx
'use server';

import getSdk from '~/lib/sdk';
import getSupabaseServerActionClient from '~/core/supabase/server-action-client';

export async function myServerAction() {
  const client = getSupabaseServerActionClient();
  const sdk = getSdk(client);

  // ...
}
```

### Route Handlers

In the example below, we initialize the SDK in a Route Handler

```tsx
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';
import getSdk from '~/lib/sdk';

export async function GET() {
  const client = getSupabaseRouteHandlerClient();
  const sdk = getSdk(client);

  // ...
}
```

### Client Side

You may also initialize the SDK on the client side - but **bear in mind some API are not going to work in non-server environments**. This is useful if you need to interact with the core entities on the client side.

```tsx
import useSupabase from '~/core/supabase/use-supabase';

export function useSdk() {
  const client = useSupabase();
  return getSdk(client);
}
```

-----------------
FILE PATH: kits/next-supabase/sdk/user-sdk.mdoc

---
status: "published"
title: "User SDK - Fetch the current User | Next.js Supabase SaaS Starter Kit"
label: "User SDK"
order: 1
description: "Learn how to use the User SDK to interact with the current user"
---


The User SDK allows you to interact with the current user. Here we list the methods available in the User SDK.

The example below work within a Server Components: by using the required client, you can easily adapt the code examples below to both Server Actions and Route Handlers.

The User SDK is namespaced under `sdk.user` - so all the methods below are available under `sdk.user`.

## Get the current user

To get the current user, use the `sdk.user.getCurrent()` method:

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);
  const user = await sdk.user.getCurrent();

  // ...
}
```

This method wraps the Supabase `auth.getUser()` function.

## Get the current user's session

To get the current session, use the `sdk.user.getCurrentSession()` method:

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);
  const session = await sdk.user.getCurrentSession();

  // ...
}
```

This method wraps the Supabase `auth.getSession()` function.

## Get the current user's ID

To get the current user's ID, use the `sdk.user.getCurrentId()` method:

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);
  const userId = await sdk.user.getCurrentId();

  // ...
}
```

## Get the current user's Database profile

We store the user's record data in the database table `users`. To get the current user's profile data, use the `sdk.user.getData()` method:

```tsx
import getSdk from '~/lib/sdk';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';

async function PageServerComponent() {
  const client = getSupabaseServerComponentClient();
  const sdk = getSdk(client);
  const data = await sdk.user.getData();

  // ...
}
```

The `data` object will contain the user's profile data.

```tsx
interface UserData {
  displayName: string | null;
  photoUrl: string | null;

  // your custom fields...
}
```

It's very likely that you'll be adding more fields, so this interface may change.

-----------------
FILE PATH: kits/next-supabase/sdk.mdoc

---
status: "published"
title: "SDK in Next.js Supabase"
label: "SDK"
order: 9
description: "SDK in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/testing/ci-tests.mdoc

---
status: "published"

title: CI Tests
label: CI Tests
order: 1
description: 'Learn how to test your Makerkit in your CI'
canonical: '/docs/next-supabase/ci-tests'
---


To run your tests in any CI, we recommend you simply run the following command:

```
npm test:e2e
```

This command takes care of running all the required services, importing the test data, executing the tests and then exiting.

### The Makerkit testing pipeline

As mentioned in the previous section, testing a Makerkit requires a few services to be up and running:

- the Next.js server
- the Supabase environment

While executing these tests in your CI, this can get cumbersome. Therefore, we provided a single script to execute all your tests in one simple command: you can find the script at `scripts/test.sh`, which will contain the following content:

```
set -e

npm run supabase:start -- -x studio,migra,deno-relay,pgadmin-schema-diff,imgproxy &
docker run --add-host=host.docker.internal:host-gateway --rm -it --name=stripe -d stripe/stripe-cli:latest listen --forward-to http://host.docker.internal:3000/api/stripe/webhook --skip-verify --api-key "$STRIPE_SECRET_KEY" --log-level debug
npm run dev:test &
npm run cypress:headless
npm run supabase:stop -- --no-backup
exit 0
```

Let's explain it, so you can add your own modifications to it.

### Running the Tests in CI mode

If you were to run the tests in CI mode, you would need to run the following command:

```
npm run supabase:start
npm run dev:test &
npm run cypress
```

Then, you would need to stop the Supabase environment:

```
npm run supabase:stop
```

Since the DB gets updated, reset it to the initial state:

```
npm run supabase:reset
```

-----------------
FILE PATH: kits/next-supabase/testing/running-tests.mdoc

---
status: "published"

title: Running Cypress Tests with Supabase and Next.js
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

## Running the Next.js development server

First, we have to run the following command to run Next.js with a testing
environment (i.e. using `.env.test`):

```
npm run dev:test
```

This command will start a Next.js dev server at http://localhost:3000, so
it's important to make sure you have no other Next.js server running at the
same port.

Additionally, we have to start the **Supabase local instance**:

```
npm run supabase:start
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

1. Start the Supabase local instance
2. Start the Next.js environment
3. Run the tests in headless mode
4. Exit and close all the processes

As you may have noticed, this is the command to run in your CI environment.


-----------------
FILE PATH: kits/next-supabase/testing.mdoc

---
status: "published"
title: "Testing in Next.js Supabase"
label: "Testing"
order: 11
description: "Testing in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/tutorials/api-routes-validation.mdoc

---
status: "published"

title: "API Routes Validation"
label: "API Routes Validation"
order: 14
description: "The best practices to validate your API routes payloads using Zod in your Next.js Supabase application."
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

export async function PUT(req: Request) {
  const result = await getBodySchema().safeParseAsync(await req.json());

  // validate the form data
  if (!result.success) {
    throw throwBadRequestException();
  }
}

function getBodySchema() {
  return z.object({
    membershipId: z.coerce.number(),
  });
}
```

I encourage you to never skip the validation step when writing your API endpoints.

-----------------
FILE PATH: kits/next-supabase/tutorials/api-routes.mdoc

---
status: "published"

title: "API Routes"
label: "API Routes"
order: 13
description: "Learn how to create and manage API routes in your Next.js
application."
---


Makerkit provides some utilities to reduce the boilerplate needed to write Next.js API functions.
This section will teach you everything you need to know to write your API functions.

### Calling API functions from the client

In Next.js App Router, the API functions are defined using the special file conventions named `route.ts`. You can add these anywhere in your app directory.

When the API functions are "standalone" and not tied to a specific page, it makes sense to add them to `app/api`.

First, we define a function using the utility `useApiRequest` and SWR's `useSWRMutation` hook.

```tsx
import useSWRMutation from 'swr/mutation';

import configuration from '~/configuration';
import useApiRequest from '~/core/hooks/use-api';
import useRefresh from '~/core/hooks/use-refresh';

interface Params {
  membershipId: number;
}

const path = configuration.paths.api.organizations.transferOwnership;

function useTransferOrganizationOwnership() {
  const fetcher = useApiRequest<void, Params>();
  const refresh = useRefresh();
  const key = ['organizations', 'transfer-ownership'];

  return useSWRMutation(
    key,
    (_, { arg }: { arg: Params }) => {
      return fetcher({
        path,
        method: `PUT`,
        body: {
          membershipId: arg.membershipId,
        },
      });
    },
    {
      onSuccess: refresh,
    }
  );
}

export default useTransferOrganizationOwnership;
```

Then, we can define a route function in the file `app/api/organizations/owner/route.ts`.

```tsx
export async function PUT(req: Request) {
  // logic to transfer ownership
}
```

#### Sending the CSRF Token

When you use the hook `useApiRequest`, you don't need to worry about sending the CSRF token. The hook will automatically send the token.

Instead, if you use `fetch` directly, you need to send the CSRF token.
To retrieve the page's CSRF token, you can use the `useCsrfToken` hook:

```tsx
const csrfToken = useCsrfToken();

console.log(csrfToken) // token
```

You will need to send a header `x-csrf-token` with the value returned by `useCsrfToken()`.

### CSRF Token check

The middleware will automatically check that the CSRF Token is present and is valid in the request header for requests that require it.

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
FILE PATH: kits/next-supabase/tutorials/application-pages.mdoc

---
status: "published"

title: "Application Pages"
label: "Application Pages"
order: 13
description: "Learn how to create and manage application pages in your Makerkit app"
---


Now that we have some components to display, we need to add them to the
actual Next.js pages.

If you have not created the files in the `Routing` section above, it's time to do it.

#### Using the components AppHeader and AppContainer

These components are used to wrap the content of the pages.

They are responsible for displaying the header and the container of the page.

```tsx
import { Squares2X2Icon } from '@heroicons/react/24/outline';

import Trans from '~/core/ui/Trans';
import AppHeader from '~/app/dashboard/[organization]/components/AppHeader';
import AppContainer from '~/app/dashboard/[organization]/components/AppContainer';

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

## Guarding Application Pages for Authenticated Users

One of the ways we can use to protect application pages, we can use the `requireSession` function. This function also verifies that the user is authenticated using MFA.

```tsx
import { redirect } from "next/navigation";

async function MyPage() {
  const session = await requireSession();

  if (!session) {
    redirect('/');
  }
}
```

If you don't want to redirect users away, you can simply display an error message instead.

```tsx
import { redirect } from "next/navigation";

async function MyPage() {
  const session = await requireSession();

  if (!session) {
    return <div>You are not authenticated</div>;
  }

  return <div>Your content goes here</div>;
}
```

## Using Dynamic Parameters

Every page and layout can access dynamic parameters from the URL.

For example, if you have a page with the following URL:

```
https://myapp.com/dashboard/[organization]/[project]
```

You can access the `organization` and `project` parameters from the page.

```tsx
interface PageData {
  params: {
    organization: string;
    project: string;
  }
}

function Page(data: PageData) {
  // data.params.organization
}
```

## Using Search Parameters

Similarly to dynamic parameters, you can also access search parameters from the URL.

For example, if you have a page with the following URL:

```
https://myapp.com/dashboard/[organization]/[project]?page=1&limit=10
```

You can access the `page` and `limit` parameters from the page.

```tsx
interface PageData {
  params: {
    organization: string;
    project: string;
  }

  searchParams: {
    page: string;
    limit: string;
  }
}

function Page(data: PageData) {
  // data.searchParams.page
}
```

## Internationalization

To enable i18n for a layout or a route, simply wrap them using the `withI18n` HOC.

This component makes sure that the i18n is initialized before the page is rendered.

```tsx
import { withI18n } from '~/i18n/with-i18n';


function TaskPage() {
  // ...
}

export default withI18n(TaskPage);
```

-----------------
FILE PATH: kits/next-supabase/tutorials/authentication.mdoc

---
status: "published"

title: "Authentication"
label: Authentication
order: 7
description: "Learn about the authentication strategies supported by Makerkit and how to configure them."
---


The Next.js/Supabase template uses Supabase Auth to manage authentication into
the internal application.

The kit supports the following strategies:

1. Email/Password
2. All the oAuth Providers supported by Supabase Auth (Google, GitHub, etc.)
3. Phone Number
4. Magic Link

You can choose one, more than one, or all of them together, and you can easily tweak this using the global configuration.

By default, the Makerkit SaaS Starter uses Email/Password and Google Auth. If you plan on using more than one together, it would be wise to adjust the UI so they play well together.

### How does Authentication work?

First, let's just take a high-level overview of how Makerkit's authentication works.

MakerKit **uses SSR throughout the application**, except for the marketing pages. Using SSR, we can persist the user's authentication on every page and access the user's object on the server **before rendering the page**.

This can help you both in terms of UX and DX. In fact, persisting the user's session server-side can help you in various scenarios:

- Simplifying the business logic: for example, checking server-side if a user can access a specific page before rendering that page
- Rendering the user's data server-side by fetching some data from Supabase

### Authentication Strategies

To add or tweak the authentication strategies of our application, we can update the configuration:

 ```tsx {% title="src/configuration.ts" %}
auth: {
  // NB: Enable the providers below in the Supabase Console
  // in your production project
  providers: {
    emailPassword: true,
    phoneNumber: false,
    emailLink: false,
    oAuth: ['google'] as Provider[],
  },
},
```

Above, you can see the **default configuration**. It should look like the following:

{% img src="/assets/images/posts/tutorial-sign-in.webp" width="2834" height="1972" /%}

Ok, cool. But what if we wanted to swap `emailPassword` with `emailLink` and add `Twitter oAuth`?

We will do the following:

 ```tsx {% title="src/configuration.ts" %}
auth: {
  // NB: Enable the providers below in the Supabase Console
  // in your production project
  providers: {
    emailPassword: false, // set this to false
    phoneNumber: false,
    emailLink: true, // set this to true
    oAuth: ['google', 'twitter'] as Provider[],
  },
},
```

And the result will be similar to the image below:

{% img src="/assets/images/posts/tutorial-sign-in-email-link.webp" width="2834" height="1972" /%}

### Creating Accounts

Users can be redirected to the [Sign Up page](http://localhost:3000/auth/sign-up) to create an account.

### Requiring Email Verification

By default, Supabase requires users to verify their email address before being able to access the application.

To disable this behavior, besides **doing it in your Supabase console**, you also need to set the `requireEmailVerification` flag to `false` using the following environment variable:

 ```bash {% title=".env" %}
REQUIRE_EMAIL_VERIFICATION=false
```

This will let the application know that you don't want to require email verification.

NB: this may not work with the latest Supabase versions - so please double check with the Supabase support team if you are having issues.

## Emails sent by Supabase

Supabase spins up an [InBucket](http://localhost:54324/) instance where all the emails are sent: this is where you can find emails related to password reset, sign-in links, and email verifications.

To access the InBucket instance, you can go to the following URL: [http://localhost:54324/](http://localhost:54324/). Save this URL, you will use it very often.


-----------------
FILE PATH: kits/next-supabase/tutorials/database-schema.mdoc

---
status: "published"
title: "Database Schema | Next.js Supabase SaaS Kit"
label: "Database Schema"
order: 8
description: "Learn how to get started developing new features from the database schema in your Makerkit Next.js Supabase project."
---


Before writing the React Components, we want to tackle the task of fetching
data from Supabase.

This involves a couple of steps:

1. We need to define, on paper, what the data model of the Supabase tables
looks like
2. We need to update the SQL schema of the database to match the data model
and write the required RLS policies to secure the data
3. We need to write the hooks to fetch data from Supabase

### Learning about RLS

Learning RLS is fundamental to understanding how to secure data in Supabase. The best resource (other than checking Makerkit's code) is the [Supabase RLS docs](https://supabase.com/docs/guides/auth/row-level-security).

Please check it out - and if you have any questions, please ask in the Makerkit Discord.

##### How is the data structured?

We can place tasks as our own Postgres table `tasks`. Let's create an SQL
table that defines the following columns:

```sql
create table tasks (
  id bigint generated always as identity primary key,
  organization_id bigint not null references public.organizations,
  name text not null,
  description text,
  done bool not null,
  due_date timestamptz not null
);
```

Because tasks belong to an
organization, we need to ensure that only users of that organization can read
and write those tasks by adding a foreign key to each `task` named
`organization_id`.

##### Replicating the tasks table as a Type

We can define a `Task` model at `src/lib/tasks/types/task.ts`:

 ```tsx {% title="lib/tasks/types/task.ts" %}
export interface Task {
  id: number;
  name: string;
  organizationId: string;
  dueDate: string;
  done: boolean;
  description: string | null;
}
```

NB: this may not be necessary if you generate the types from the SQL schema.
Up to you.

##### How do we protect the data with RLS?

We can add an RLS policy to the `tasks` table to ensure that only users of the same organization can read and write tasks.

We know that a user belongs to an organization using the `memberships` table.

The `memberships` table has two foreign keys: `user_id` and
`organization_id`: we can use these foreign keys to ensure that only users
of the same organization can read and write tasks.

First, we enable RLS on the `tasks` table:

```sql
alter table tasks enable row level security;
```

Then, we can add a policy to the `tasks` table:

```sql
create policy "Tasks can be read by users of the organizations to which it belongs" on tasks
  for all
    using (
      exists (
        select
          1
        from
          memberships
        where
          user_id = auth.uid () and
          tasks.organization_id = memberships.organization_id
      )
    );
```

#### Generating the types from the SQL schema

We can generate the types from the SQL schema using the Supabase CLI.

To run this command, you need to ensure Supabase is running locally using
`npm run supabase:start`.

You can generate the types by running the following command:

```bash
npm run typegen
```

The types will be generated at `src/database-types.ts`.

We will import the types whenever we use the Supabase client to ensure that it is typed correctly, which helps us write type-safe queries with the Supabase client.

For example:

```tsx
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '../../database.types';

type Client = SupabaseClient<Database>;
```


