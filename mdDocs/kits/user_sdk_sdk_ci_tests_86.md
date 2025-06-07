-----------------
FILE PATH: kits/remix-supabase/sdk/user-sdk.mdoc

---
status: "published"
title: "User SDK - Fetch the current User | Remix Supabase SaaS Kit"
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
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);
  const user = await sdk.user.getCurrent();

  // ...
}
```

This method wraps the Supabase `auth.getUser()` function.

## Get the current user's session

To get the current session, use the `sdk.user.getCurrentSession()` method:

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);
  const session = await sdk.user.getCurrentSession();

  // ...
}
```

This method wraps the Supabase `auth.getSession()` function.

## Get the current user's ID

To get the current user's ID, use the `sdk.user.getCurrentId()` method:

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);
  const userId = await sdk.user.getCurrentId();

  // ...
}
```

## Get the current user's Database profile

We store the user's record data in the database table `users`. To get the current user's profile data, use the `sdk.user.getData()` method:

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
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
FILE PATH: kits/remix-supabase/sdk.mdoc

---
status: "published"
title: "SDK in Remix Supabase"
label: "SDK"
order: 9
description: "SDK in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/testing/ci-tests.mdoc

---
status: "published"

title: CI Tests
label: CI Tests
order: 1
description: 'Learn how to test your Makerkit in your CI'
---


To run your tests in any CI, we recommend you simply run the following command:

```
npm test:e2e
```

This command takes care of running all the required services, importing the test data, executing the tests and then exiting.

### The Makerkit testing pipeline

As mentioned in the previous section, testing a Makerkit requires a few services to be up and running:

- the Remix server
- the Supabase dev environment

While executing these tests in your CI, this can get cumbersome. Therefore, we provided a single script to execute all your tests in one simple command: you can find the script at `scripts/test.sh`, which will contain the following content:

```
set -e

npm run supabase:start -x studio,migra,deno-relay,pgadmin-schema-diff,imgproxy &
docker run --add-host=host.docker.internal:host-gateway --rm -it -d stripe/stripe-cli:latest listen --forward-to http://host.docker.internal:3000/resources/stripe/webhook --skip-verify --api-key "$STRIPE_SECRET_KEY" --log-level debug
dotenv -e .env.test npm run build
dotenv -e .env.test npm run start &
dotenv -e .env.test npm run cypress:headless
npm run supabase:stop -- --no-backup
exit 0
```


### Running the Tests in CI mode

If you were to run the tests in CI mode, you would need to run the following command:

```
dotenv -e .env.test npm run test:remix &
dotenv -e .env.test npm run cypress:headless
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
FILE PATH: kits/remix-supabase/testing/running-tests.mdoc

---
status: "published"

title: Running Cypress Tests with Supabase and Remix
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
npm run dev:test
```

This command will start a Remix dev server at http://localhost:3000, so
it's important to make sure you have no other Remix server running at the
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
FILE PATH: kits/remix-supabase/testing.mdoc

---
status: "published"
title: "Testing in Remix Supabase"
label: "Testing"
order: 11
description: "Testing in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/tutorials/api-routes-validation.mdoc

---
status: "published"

title: "API Routes Validation"
label: "API Routes Validation"
order: 15
description: "The best practices to validate your API routes payloads using Zod in your Remix Supabase application."
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
FILE PATH: kits/remix-supabase/tutorials/api-routes.mdoc

---
status: "published"

title: "API Routes"
label: "API Routes"
order: 14
description: "The best practices to write API routes in your Remix Supabase application"
---


Makerkit provides some utilities to reduce the boilerplate needed to write Remix API functions.
This section will teach you everything you need to know to write your API functions.

There are many ways to call an API route in Remix:
1. Using the Remix `useSubmit` or `useFetcher` hooks
2. Using the `fetch` API wrapper in Makerkit `useFetcher`
3. Using the `fetch` API directly

As a rule of thumb - it's best to use the `useSubmit` or `useFetcher` hooks whenever possible. It's the Remix way ad closer to the Remix philosophy.

With that said, there are times where the `useFetch` hook can be useful - where `useFetcher` and `useSubmit` may not be as convenient - for example - when you need to await the request using a Promise.

### Calling API functions from the client

API functions in Remix are defined using the special functions `loader` and
`action`.

These functions are defined in the routes.

These functions can be called in
various ways, for example using Remix's `useFetcher`, `useSubmit`, `<Form>`,
or can be called using simple `fetch` calls.

#### Defining global API functions

When the API functions are "standalone" and not tied to a specific page, it
makes sense to prefix them with the `resources` path.

For example, the following code calls the `action` function `owner` from the
file `resources.organizations.members.transfer-ownership.ts`.

##### Using Fetch + React Query
First, we define a function using the utility `useFetch` and React Query's `useMutation` hook.

```tsx
import useFetch from '~/core/hooks/use-fetch';
import { useMutation } from '@tanstack/react-query';

function useTransferOrganizationOwnership() {
  const transferOrganizationFetch = useFetch<{ membershipId: number }>(
    `/resources/organizations/members/transfer-ownership`,
    'PUT'
  );

  return useMutation((membershipId: number) => {
    return transferOrganizationFetch({ membershipId });
  });
}

export default useTransferOrganizationOwnership;
```

Then, we can define an `action` function in the file `resources.organizations.ts`.

```tsx
export async function action(args: ActionArgs) {
  const req = args.request;

  // logic to transfer ownership
}
```

#### Using useFetcher and useSubmit

Alternatively, we can use the Remix functions `useSubmit` or `useFetcher` to call the same endpoint.

This is more conventional to Remix, so you may prefer this latter approach.

```tsx {2,5}
function useTransferOrganizationOwnership() {
  const submit = useSubmit();

  return useCallback((membershipId: number) => {
    return submit({
      data: { membershipId: membershipId.toString() },
      action: `/resources/organizations/members/transfer-ownership`,
      method: `put`
    });
  });
}

export default useTransferOrganizationOwnership;
```

### Submitting Actions using JSON

It's important to notice that in this case the payload is sent as `FormData`, not as `JSON` - but you can specify the `encType` parameter to change this behavior.

```tsx
const fetcher = useFetcher();

<button onClick={() => {
  fetcher.submit(data, {
    method: 'POST',
    encType: 'application/json'
  })
}}>Submit</button>
```

```tsx
export async function action(args: ActionArgs) {
  const req = args.request;
  const body = await req.json();

  // ...
}
```

#### Sending the CSRF Token

When you use the hook `useFetch`, you don't need to worry about sending the CSRF token. The hook will automatically send the token.

Instead, if you use `fetch` directly, you need to send the CSRF token.
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
3. The CSRF token is sent to an HTTP POST request automatically when using
the `useFetch` hook
4. This function will throw an error when the token is invalid

By using this function, **we ensure all the following functions will not be executed unless the token is valid**.

```tsx
export const action: ActionFunction = async ({ request }) => {
  await withCsrf(request);
}
```

### Sending CSRF Token using useFetcher and useSubmit

When using `useSubmit` and `useFetcher` - you must pass the CSRF token manually:

```tsx
const submit = useSubmit();
const csrfToken = useCsrfToken();

submit({ csrfToken }, {
  method: 'POST',
  encType: 'application/json'
})
```

We can then validate it using the `withCsrf` function:

```tsx
export const action: ActionFunction = async ({ request }) => {
  const body = await request.json();
  await withCsrf(request, () => body.csrfToken);
}
```

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
FILE PATH: kits/remix-supabase/tutorials/application-pages.mdoc

---
status: "published"

title: "Application Pages"
label: "Application Pages"
order: 15
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

## Guarding Application Pages for Authenticated Users

One of the ways we can use to protect application pages, we can use the `requireSession` function. This function also verifies that the user is authenticated using MFA.

```tsx
import { redirect } from "@remix-run/node";

export async function loader() {
  const session = await requireSession();

  if (!session) {
    return redirect("/auth/sign-in");
  }

  return {
    session,
  };
}
```

## Using Dynamic Parameters

Every page and layout can access dynamic parameters from the URL.

For example, if you have a page with the following URL:

```
https://myapp.com/dashboard/[organization]/[project]
```

You can access the `organization` and `project` parameters from the page's `loader` function.

```tsx
import type { LoaderArgs } from '@remix-run/node';

export async function loader(args: LoaderArgs) {
  const organization = args.params.organization;
  const project = args.params.project;

  // ...
}
```

## Using Search Parameters

Similarly to dynamic parameters, you can also access search parameters from the URL.

For example, if you have a page with the following URL:

```
https://myapp.com/dashboard/[organization]/[project]?page=1&limit=10
```

You can access the `page` and `limit` parameters from the page's `loader` function.

```tsx
import type { LoaderArgs } from '@remix-run/node';

export async function loader(args: LoaderArgs) {
  const params = new URL(args.request.url).searchParams;
  const page = params.get('page');
  const limit = params.get('limit');

  // ...
}
```


-----------------
FILE PATH: kits/remix-supabase/tutorials/authentication.mdoc

---
status: "published"

title: "Authentication"
label: Authentication
order: 7
description: "Learn about the authentication strategies supported by Makerkit and how to configure them."
---


The Remix/Supabase template uses Supabase Auth to manage authentication into
the internal application.

The kit supports the following strategies:

1. Email/Password
2. All the oAuth Providers supported by Supabase Auth (Google, GitHub, etc.)
3. Phone Number

You can choose one, more, or all of them together, and you can easily tweak this using the global configuration.

By default, the Makerkit SaaS Starter uses Email/Password and Google Auth.

### How does Authentication work?

First, let's just take a high-level overview of how Makerkit's authentication works.

MakerKit **uses SSR throughout the application**, except for the marketing pages. Using SSR, we can persist the user's authentication on every page and access the user's object on the server **before rendering the page**.

This can help you both in terms of UX and DX. In fact, persisting the user's session server-side can help you in various scenarios:

- Simplifying the business logic: for example, checking server-side if a user can access a specific page before rendering that page
- Rendering the user's data server-side by fetching some data from Supabase in
the `loader` function of your Remix routes

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
    oAuth: ['google'],
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
    oAuth: ['google', 'twitter'],
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
FILE PATH: kits/remix-supabase/tutorials/database-schema.mdoc

---
status: "published"
title: "Database Schema | Remix Supabase SaaS Kit"
label: "Database Schema"
order: 8
description: "Learn how to get started developing new features from the database schema in your Makerkit Remix Supabase project."
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
  description?: string;
}
```

NB: this may not be necessary if you generate the types from the SQL schema.
Up to you.

##### How do we protect the data with RLS?

We can add an RLS policy to the `tasks` table to ensure that only users of the same organization can read and write tasks.

We know that a user belongs to an organization using the `memberships`
table.

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

The types will be generated at `app/database-types.ts`.

We will import the types whenever we use the Supabase client to ensure that it is typed correctly, which helps us write type-safe queries with the Supabase client.

For example:

```tsx
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '../../database.types';

type Client = SupabaseClient<Database>;
```


-----------------
FILE PATH: kits/remix-supabase/tutorials/deploying-production.mdoc

---
status: "published"

title: "Deploying to Production"
label: "Deploying to Production"
order: 20
description: "Learn how to deploy your Remix Supabase app to your hosting provider."
---


Before deploying to production or any remote server, you need to [follow all the steps outlined in the documentation](/docs/remix-supabase/going-to-production-overview).

**Much of this work needs to be done externally**, not in the Makerkit codebase.

1. **Supabase**: Create a new Supabase project, and add the relevant environment variables to your project
2. **Link project with the CLI**: Link the Supabase project with your local CLI
3. **Remote Database**: Deploy the database schema and migrations to your Supabase project
4. **Environment Variables**: Ensure that your environment variables are set correctly
5. **Auth**: Enable the authentication providers you want to use from the Supabase Console
6. **SMTP**: Set up an SMTP server to send emails for team invites - and also do the same in Supabase Console to avoid their very low limits and better deliverability
7. **Payments**: Set up your Stripe or Lemon Squeezy accounts and add the relevant environment variables
8. **Deployment**: Finally, deploy your application to your hosting provider (Vercel, Firebase, Netlify, etc.): please follow the instructions provided by your hosting provider to deploy a Remix application.


-----------------
FILE PATH: kits/remix-supabase/tutorials/environment-variables.mdoc

---
status: "published"

title: "Environment Variables"
label: Environment Variables
order: 5
description: "Learn how to use environment variables in your Remix project."
---


The starter project comes with two different environment variables files:
1. **.env**: the main environment file
2. **.env.test**: this environment file is loaded when running the Cypress
E2E tests. You would rarely need to use this.

NB: the `.env` file is never committed to the repository. This is because it
contains sensitive information, such as API keys. Instead, we use a `.env.
template` file to show what the `.env` file should look like.

The environment variables for the Remix Supabase kit look like the below:

```bash
DEFAULT_LOCALE=en
SITE_URL=http://localhost:3000

# set the below to "production" in your production environment
ENVIRONMENT=development

# SUPABASE
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# STRIPE
STRIPE_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# EMAIL
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

{% alert title="Define your environment variables in your hosting provider" %}

  Remix does not use .env files for bundling your app in production. Therefore, you will need to define your environment
  variables in your hosting provider (such as Vercel).
{% /alert %}

### Define your environment variables in your hosting provider

**NB**: Remix will not bundle environment variables when building your app, but only during development mode. This needs to be repeated, as it is often confused with how Next.js works.

Instead, you will need to define them in your hosting provider (such as your Vercel project settings).

