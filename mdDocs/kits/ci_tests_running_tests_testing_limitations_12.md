-----------------
FILE PATH: kits/next-fire/testing/ci-tests.mdoc

---
status: "published"

title: CI Tests
label: CI Tests
order: 1
description: 'Learn how to test your Makerkit in your CI'
---


To run your tests in any CI, we recommend you simply run the following command:

```bash
npm test
```

This command takes care of running all the required services, importing the test data, executing the tests and then exiting.

### The Makerkit testing pipeline

As mentioned in the previous section, testing a Makerkit requires a few services to be up and running:

- the Next.js server
- the Firebase emulators
- optionally, the Stripe Mock server

While executing these tests in your CI, this can get cumbersome. Therefore, we provided a single script to execute all your tests in one simple command: you can find the script at `scripts/test.sh`, which will contain the following content:

```bash
set -e

npm run dev:test & npm run stripe:mock-server &
npm run cypress:headless
sh scripts/kill-ports.sh
```

Let's explain it, so you can add your own modifications to it.

The first command will run (in parallel and in the background) the Next.js server and the Stripe mock server:

```bash
npm run dev:test & npm run stripe:mock-server &
```

Afterward, we execute the Cypress tests in headless mode:

```bash
npm run cypress:headless
```

Finally, we clean up all the ports that may have remained open after executing the script:

```bash
sh scripts/kill-ports.sh
```

If you update your ports, remember to change them also in `scripts/kill-ports.sh`.

### Running the Tests in CI mode

Finally, the tests can be executed with the command `npm test`. 

This command **will automatically run the Firebase emulators** and the `test.sh` script, and is defined as:

```bash
"test": "firebase emulators:exec --project demo-makerkit --import ./firebase-seed \"sh ./scripts/test.sh\"",
```

When finished, the emulators will exit.

-----------------
FILE PATH: kits/next-fire/testing/running-tests.mdoc

---
status: "published"
title: Running Cypress Tests with Firebase and Next.js
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

## Running the Next.js testing environment

First, we have to run the following command to run Next.js with a testing
environment (i.e. using `.env.test`):

```bash
npm run dev:test
```

This command will start a Next.js dev server at http://localhost:3000, so
it's important to make sure you have no other Next.js server running at the
same port.

Additionally, we have to start the **Firebase Emulator server**:

```bash
npm run firebase:emulators:start
```

## Running Cypress in development mode

While you build your tests, it can be handy to use a windowed version of the
Cypress testing suite, so that you can easily retry the tests without having
to re-run the while tests suite.

To do that, run the following command:

```bash
npm run cypress
```

## Running All Cypress tests

Whenever you want to run all your tests at once, you can use the following commands:

```bash
npm run cypress:headless
```

This will run all the tests and exit.

## Running Cypress in CI mode

Whenever you want to run all your tests in a CI
environment, you can use the following commands:

```bash
npm test
```

The command above will:

1. Start the Firebase Emulator
2. Start the Next.js environment
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

```bash
npm run stripe:mock-server
```

As we've mentioned, this will require Docker running. Of course, we think
it's worth it.


-----------------
FILE PATH: kits/next-fire/testing/testing-limitations.mdoc

---
status: "published"

title: Testing Limitations
label: Limitations
order: 2
description: ''
---


Due to the hard-to-rollback nature of some tests (such as accepting invitations) - some Tests won't work well in development mode.

To have a clean testing environment at each run, we recommend running the following command:

```bash
npm test
```

Of course, some of the tests will work just fine, but not the ones related to accepting invites.

This would be solved if we could reset the Firebase Firestore data programmatically between test runs, but as far as we know this is not yet possible.

-----------------
FILE PATH: kits/next-fire/testing.mdoc

---
status: "published"
title: Tests in Makerkit Next.js Firebase
label: Tests
order: 11
description: ''
collapsible: true
collapsed: true
---

Makerkit provides E2E Tests using Cypress.

-----------------
FILE PATH: kits/next-fire/tutorials/adding-blog-posts.mdoc

---
status: "published"

title: "Adding Blog Posts"
label: "Adding Blog Posts"
order: 17
description: "Learn how to add blog posts to your site."
---

We can add your website's blog posts within the `_posts` folder.

#### Collections

Before writing a blog post, we have to define the post's collection. We use `collections` to organize our blog posts by their category.

For example, your blog could have collections such as changelog,
announcements, tutorials, etc.

MakerKit generates every collection with its page, listing all the articles
associated with it.

To change a collection page, please change the file `src/pages/blog/[collection].ts`.

##### Adding collections

Let's see how we can define a `collection` in Typescript:

```tsx
interface Collection {
  name: string;

  // image
  logo?: string;
  emoji?: string;
}
````

As you can see, the only required property to create a collection is a name. You can also attach an image or a simple emoji for each `collection`.

A collection can be simply the following file:

```json
{
  "name":"Changelog"
}
```

#### Writing a Blog Post
The interface of a blog post is the following:

```tsx
interface Post {
  title: string;
  excerpt: string;

  date: string;
  live: boolean;
  tags?: string[];

  coverImage?: string;

  ogImage?: {
    url: string;
  };

  author?: {
    name: string;
    picture: string;
    url: string;
  };

  canonical?: string;
}
```

These values can be defined in MDX files using the `frontmatter`, for example:

```yaml
---

title: An Awesome Post title
except: "Write here a short description for your blog post"
collection: changelog.json
date: '2022-01-05'
live: true
coverImage: '/assets/images/posts/announcement.webp'
tags:
  - changelog
---

```


-----------------
FILE PATH: kits/next-fire/tutorials/adding-documentation-pages.mdoc

---
status: "published"

title: "Adding Documentation pages"
label: "Adding Documentation pages"
order: 18
description: "Learn how to add documentation pages to your product's website"
---

We place the documentation pages within the `_docs` folder.

#### Topics

The pages are defined hierarchically so that you can define your
documentation in the following way:

```
- _docs
  - [topic]
    - [topic.json]
    - [page].mdx
```

The documentation you're reading, for example, is defined as follows.

The MakerKit kit has a folder called `_docs`: in this folder, we have a list of
sub-folders for each topic we are describing.

Each folder has a metadata file named `meta.json`:

```json
{
  "title": "Blog and Docs",
  "position": 2,
  "description": "Learn how to configure and write your product's Blog and Documentation"
}
```

The `position` property defines the order of the topics. In
the case above, the topic `Blog and Docs` will be the third topic in the list.

#### Topics Pages

Within each topic, we define the collection pages defined as MDX files.
They share the same components as the blog posts' files.

A page is defined as follows:

```yaml
---

title: Blog
order: 1
---

```

The `position` property defines the order in which the page is listed within
the topic.

-----------------
FILE PATH: kits/next-fire/tutorials/api-routes-validation.mdoc

---
status: "published"

title: "API Routes Validation"
label: "API Routes Validation"
order: 14
description: "The best practices to validate your API routes payloads using Zod in your Next.js Firebase application."
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

function inviteMemberHandler(
  req: NextApiRequest,
  res: NextApiResponse,
) {
  try {
     // we can safely use data with the interface Body
    const schema = getBodySchema();
    const { displayName, email } = schema.parse(req.body);

    return sendInvite({ displayName, email });
  } catch(e) {
    return throwBadRequestException(res);
  }
}

export default function apiHandler() {
  const handler = withPipe(
    withMethodsGuard(['POST']),
    withAuthedUser,
    inviteMemberHandler,
  );

  // manage exceptions
  return withExceptionFilter(req, res)(handler);
}
```

I encourage you to **never skip the validation step when writing your API endpoints**.


-----------------
FILE PATH: kits/next-fire/tutorials/api-routes.mdoc

---
status: "published"

title: "API Routes"
label: "API Routes"
order: 13
description: "Learn how to create and manage API routes in your Next.js application."
---

Makerkit provides some utilities to reduce the boilerplate needed to write Next.js API functions. This section will teach you everything you need to know to write your API functions.

As you may know, we write Next.js' API functions at `pages/api`. Therefore, every file listed in this folder becomes a callable HTTP function.

For example, we can write the file `pages/api/hello-world.ts`:

 ```tsx {% title="pages/api/hello-world.ts" %}
export default function helloWorldHandler(
  req: NextApiRequest,
  res: NextApiResponse,
) {
  res.send(`Hello World`);
}
```

This API endpoint will simply return `Hello World`.

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
import useSWRMutation from "swr/mutation";
import { useApiRequest } from '~/core/hooks/use-api';

interface Body {
  idToken: string;
}

export function useCreateSession() {
  const endpoint = '/api/session/sign-in';
  const fetcher = useApiRequest<void, Body>();

  return useSWRMutation(endpoint, (path, { arg }: { arg: Body }) => {
    return fetcher({
      path,
      body: arg,
    });
  });
}
```

As you can see above, we're using SWR, a popular library for data fetching. We

You can use this hook in your components:

```tsx
import { useCreateSession } from '~/core/hooks/use-create-session';

function Component() {
  const { trigger, error, isMutating, data } = useCreateSession();

  return (
    <>
      { isMutating ? `Loading...` : null }
      { error ? `Error :(` : null }
      { data ? `Yay, success!` : null }

      <SignInForm onSignIn={(idToken) => trigger({ idToken })} />
    </>
  );
}
```

Similarly, you can also use it to fetch data. Below, is an example for fetching data using SWR:

```tsx
import type { User } from 'firebase/auth';
import useSWR from 'swr';

import { useApiRequest } from '~/core/hooks/use-api';

/**
 * @name useFetchOrganizationMembersMetadata
 * @param organizationId
 */
export function useFetchOrganizationMembersMetadata(organizationId: string) {
  const endpoint = getFetchMembersPath(organizationId);
  const fetcher = useApiRequest<User[]>();

  return useSWR(endpoint, (path) => {
    return fetcher({ path, method: 'GET' });
  });
}

function getFetchMembersPath(organizationId: string) {
  return `/api/organizations/${organizationId}/members`;
}
```

And below, we're using the hook in a component:

```tsx
import { Trans } from "next-i18next";
import Alert from "~/core/ui/Alert";
import LoadingMembersSpinner from "~/core/ui/LoadingMembersSpinner";

function MembersListComponent() {
  const {
    data: members,
    isLoading: loading,
    error,
  } = useFetchOrganizationMembersMetadata(organizationId);

  if (loading) {
    return (
      <LoadingMembersSpinner>
        <Trans i18nKey={'organization:loadingMembers'} />
      </LoadingMembersSpinner>
    );
  }

  if (error) {
    return (
      <Alert type={'error'}>
        <Trans i18nKey={'organization:loadMembersError'} />
      </Alert>
    );
  }

  // display members using "members"
  return <>...</>;
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

### Piping utilities

Makerkit uses a dead-simple utility to pipe functions named `withPipe`. We can pass any `function` to this utility, which will iterate over its parameters and execute each.

Each function must be a Next.js handler that accepts the `req` and `res` objects. Let's take a look at the below:

```tsx
import {NextApiRequest, NextApiResponse } from "next";

import { withPipe } from '~/core/middleware/with-pipe';
import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';
import { withAppCheck } from '~/core/middleware/with-app-check';

export default function members(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withMethodsGuard(['POST']),
    withAuthedUser,
    withAppCheck,
    membersHandler
  );

  return withExceptionFilter(req, res)(handler);
}

function membersHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // API logic
}
```

What happens? We've passed 4 functions to the `withPipe` utility. The utility iterates over each of them. So it will execute `withMethodsGuard`, `withAuthedUser`, `withAppCheck`, and if everything went well during the checks, it will run the API logic we define in `membersHandler`.

1. `withMethodsGuard` checks if the method passed is supported
2. `withAuthedUser` checks if the user is authenticated
3. `withAppCheck` executes a Firebase App Check validation to prevent abuse
4. `membersHandler` is the actual logic of the API endpoint

### API Authentication

To validate that API functions are called by users authenticated with Firebase Auth, we use `withAuthedUser`.

This function will:

1. Initialize the Firebase Admin - this is always needed when using Firebase!
2. Get the user using the `session` cookie
3. Throw an error when not authenticated

By using this function, **we ensure all the following functions will not be executed unless the user is authenticated**.

```tsx
import {NextApiRequest, NextApiResponse } from "next";
import { withPipe } from "~/core/middleware/with-pipe";
import { withAuthedUser } from "~/core/middleware/with-authed-user";
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function members(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    membersHandler
  );

  return withExceptionFilter(req, res)(handler);
}

function membersHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // API logic
}
```

### CSRF Token check

We must pass a `CSRF token` when using POST requests to prevent malicious attacks. To do so, we use the pipe `withCsrfToken`.

1. The CSRF token is generated when the page is server-rendered
2. The CSRF token is stored in a `meta` tag
3. The CSRF token is sent to an HTTP POST request automatically when using the `useApiRequest` hook
4. This function will throw an error when the token is invalid

By using this function, **we ensure all the following functions will not be executed unless the token is valid**.

```tsx
import { NextApiRequest, NextApiResponse } from "next";
import { withPipe } from "~/core/middleware/with-pipe";
import { withAuthedUser } from "~/core/middleware/with-authed-user";
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';
import { withCsrf } from '~/core/middleware/with-csrf';

export default function members(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    withCsrf(),
    membersHandler
  );

  return withExceptionFilter(req, res)(handler);
}

function membersHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // API logic
}
```

### Firebase App Check

App Check is a Firebase service that helps you to protect your web app from
bots, spammy users, and general abuse detected by Google's Recaptcha.

[Check out the documentation to setup Firebase App Check](/docs/next-fire/app-check).

Using the `withAppCheck` pipe, **we ensure all the following functions will not be executed unless the App Check token is valid**.

```tsx
import {NextApiRequest, NextApiResponse } from "next";
import { withPipe } from "~/core/middleware/with-pipe";
import { withAuthedUser } from "~/core/middleware/with-authed-user";
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';
import { withCsrf } from '~/core/middleware/with-csrf';

export default function members(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    withCsrf(),
    membersHandler
  );

  return withExceptionFilter(req, res)(handler);
}

function membersHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // API logic
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

We import the logging utility from `~/core/logger`.

Typically, you can log every time you perform an action, both before and after.

For example:

```tsx
import logger from '~/core/logger';

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

async function performAction() {
  // do something complex here
}
```

It's always important to add context to your logs: as you can see, we use the first parameter to add important information.

-----------------
FILE PATH: kits/next-fire/tutorials/application-pages.mdoc

---
status: "published"

title: "Application Pages"
label: "Application Pages"
order: 12
description: "Learn how to create and manage application pages in your Makerkit app"
---

Now that we have some components to display, we need to add them to the actual Next.js pages.

If you have not created the files in the `Routing` section above, it's time to do it.

#### Using the RouteShell component

The RouteShell component is a wrapper around your internal application pages. This component will:

1. Automatically protect pages when the user is not authenticated
2. Initializes the Firestore provider on the page
3. Wraps the page with one of the two layouts available: `Sidebar` or `Top Header`

```tsx {9,15}
import { ArrowLeftIcon } from "@heroicons/react/24/outline";
import Heading from "~/core/ui/Heading";
import Button from "~/core/ui/Button";
import ErrorBoundary from "~/core/ui/ErrorBoundary";
import Alert from "~/core/ui/Alert";

const TaskPage: React.FC<{ taskId: string }> = ({ taskId }) => {
  return (
    <RouteShell title={<TaskPageHeading />}>
      <ErrorBoundary
        fallback={<Alert type={'error'}>Ops, an error occurred :</Alert>}
      >
        <TaskItemContainer taskId={taskId} />
      </ErrorBoundary>
    </RouteShell>
  );
};

function TaskPageHeading() {
  return (
    <div className={'flex items-center space-x-6'}>
      <Heading type={4}>
        <span>Task</span>
      </Heading>

      <Button size={'small'} color={'transparent'} href={'/tasks'}>
        <ArrowLeftIcon className={'mr-2 h-4'} />
        Back to Tasks
      </Button>
    </div>
  );
}

export default TaskPage;
```

#### Protecting Internal Pages with Route Guards

When we create an internal page that should be protected using authentication, we need to use a Makerkit function named `withAppProps`.

This function is a `server-side props function` that takes the context `ctx` as a parameter:

```tsx {20-24}
import { GetServerSidePropsContext } from "next";

import RouteShell from `~/components/RouteShell`;
import { withAppProps } from '~/lib/props/with-app-props';
import ErrorBoundary from "~/core/ui/ErrorBoundary";
import Alert from "~/core/ui/Alert";

const TaskPage: React.FC<{ taskId: string }> = ({ taskId }) => {
  return (
    <RouteShell title={<TaskPageHeading />}>
      <ErrorBoundary
        fallback={<Alert type={'error'}>Ops, an error occurred :</Alert>}
      >
        <TaskItemContainer taskId={taskId} />
      </ErrorBoundary>
    </RouteShell>
  );
};

export function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  return withAppProps(ctx);
}
```

The `withAppProps` function will:

1. return the current user data (both Auth and Firestore)
2. return the selected user's authentication
3. return the page's translations
4. return a CSRF token

```tsx
interface PageProps {
  session?: Maybe<AuthUser>;
  user?: Maybe<UserData>;
  organization?: Maybe<WithId<Organization>>;
  csrfToken?: string;
  ui?: UIState;
}
```

Additionally, it will validate the current session:

1. Is the user onboarded? Otherwise, redirect to the onboarding flow
2. Does the user exist? Otherwise, redirect the user back to the login page
3. Is the session cookie valid? Otherwise, redirect the user back to the login page

You can extend this function and add any additional validation you deem needed.

-----------------
FILE PATH: kits/next-fire/tutorials/authentication.mdoc

---
status: "published"

title: "Authentication"
label: Authentication
order: 7
description: "Learn about the authentication strategies supported by Makerkit and how to configure them."
---

The Next.js/Firebase template uses Firebase Auth to manage authentication into the internal application.

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
  enableMultiFactorAuth: true,
  // When enabled, users will be required to verify their email address
  // before being able to access the app
  requireEmailVerification:
    process.env.NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION === 'true',
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

{% alert type="warning" title="Enable the methods you want to use from the Firebase Console" %}
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

