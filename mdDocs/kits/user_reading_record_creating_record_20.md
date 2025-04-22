-----------------
FILE PATH: kits\next-supabase\how-to\contextual-data\user.mdoc

---
status: "published"

title: "Fetching the signed in User"
label:  "Fetching the signed in User"
order: 0
description: "Learn how to fetch the signed in user from the backend and frontend."
---


There are several ways to fetch the signed in user, depending on the use-case.

## Fetching the signed in user from the backend

To do so, use the function `requireSession`, which also takes care of redirecting the user to the login page if they are not signed in, and checking the correct MFA access level if the user opted in.

You can use this function in:
- API Routes
- Server Actions
- Server Components

### Using "requireSession" in API Routes

When using API routes, you can retrieve the signed in user using the `requireSession` function.

```tsx
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';
import requireSession from '~/lib/user/require-session';

export async function POST() {
  const client = getSupabaseRouteHandlerClient();
  const session = await requireSession(client);
}
```

### Using "requireSession" in Server Actions

When using server actions, you can retrieve the signed in user using the `requireSession` function.

```tsx
'use server';

import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';
import requireSession from '~/lib/user/require-session';

export async function yourServerAction() {
  const client = getSupabaseRouteHandlerClient();
  const session = await requireSession(client);

  // use session here
}
```

### Using "requireSession" in Server Components

You can use the `requireSession` function in Server Components as well.

```tsx
async function PageComponent() {
  const client = getSupabaseServerComponentClient();
  const session = await requireSession(client);

  // use session here
}
```

## Fetching the signed in user from the frontend

When in the scope of the `app/dashboard/[organization]` layout, the app automatically injects the user session in the context layout.

This includes the user ID, email, and other authentication data, as well as the user's Supabase DB record.

### Getting the user Session with "useUserSession"

To retrieve the signed in user from the frontend, you can use the `useUserSession` hook:

```tsx
import useUserSession from '~/core/hooks/use-user-session';

const userSession = useUserSession();
```

This is a React hook and can only be used inside a React component.

### Getting the user ID with "useUserId"

To retrieve the signed in user ID from the frontend, you can use the `useUserId` hook:

```tsx
import useUserId from '~/core/hooks/use-user-id';

const userId = useUserId();
```

This is a React hook and can only be used inside a React component.

### Getting the user role within the organization with "useUserRole"

To retrieve the signed in user role of the current organization from the frontend, you can use the `useUserRole` hook:

```tsx
import useCurrentUserRole from '~/core/hooks/use-current-user-role';

const role = useCurrentUserRole();
```

This is a React hook and can only be used inside a React component.


-----------------
FILE PATH: kits\next-supabase\how-to\data-fetching\reading-record.mdoc

---
status: "published"
title: "Reading Records from Postgres | Next.js Supabase Kit"
label: "Reading Records from Postgres"
order: 0
description: "Learn how to read records from Supabase using Next.js Supabase application"
---

In Makerkit, we define all the queries in their own file (depending on their feature) named `queries.ts`.

Assuming we have a table named `tasks` with the following schema:

```sql
create table tasks (
  id serial primary key,
  name text,
  organization_id integer references organizations not null,
  due_date timestamp,
  description text,
  done boolean default false
);
```

We can define a query to read a single task from the table:

```tsx
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '~/database.types';

export function getTask(client: SupabaseClient<Database>, id: number) {
  return client
    .from('tasks')
    .select(
      `
      id,
      name,
      organizationId: organization_id,
      dueDate: due_date,
      description,
      done
    `,
    )
    .eq('id', id)
    .single();
}
```

The `single` modifier will return a single record from the database. If no record is found, it will throw an error. If you want to return `null` instead, you can use the `maybeSingle` modifier.

## Usage

You can now use the function above anywhere in the following layers:

1. API routes
2. React Server Components
3. React Client Components
4. Server Actions

In fact, we can import the Supabase client both on the browser and on the client.

### Backend

Assuming we want to read a task from a Server component, such as the Task page, we can do the following:

```tsx
interface Context {
  params: {
    task: string;
  };
}

const TaskPage = async ({ params }: Context) => {
  const data = await loadTaskData(params.task);
  const task = data.task;

  return (
    <>
      <h1>{task.name}</h1>
      <p>{task.description}</p>
    </>
  );
};

async function loadTaskData(taskId: string) {
  const client = getSupabaseServerComponentClient();
  const { data: task } = await getTask(client, Number(taskId));

  if (!task) {
    redirect('/dashboard');
  }

  return {
    task,
  };
}

export default TaskPage;
```

### Frontend

You can also choose to read the task from the frontend. This is useful if you want to use the data in a React component.

To do so, we use the **Supabase Web SDK client** using the hook `useSupabase` and combine it with SWR to fetch the data.

This is useful because you can fetch data directly from the client, which is extremely useful in scenarios such as loading data on-demand resulting from user interactions (click a table row, opening a modal, etc.)

```tsx
import useSWR from "swr";

function useFetchTask({id}: {id: number}) {
  const supabase = useSupabase();
  const key = ['tasks', id];

  return useSWR([key], async () => {
    return getTask(supabase, id);
  });
}
```

We can then use the hook in a React component:

```tsx
function TaskComponent({ id }: { id: number }) {
  const { data: task, isLoading, error } = useFetchTask({ id });

  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <>
      <h1>{task.name}</h1>
      <p>{task.description}</p>
    </>
  );
}
```

-----------------
FILE PATH: kits\next-supabase\how-to\data-mutations\creating-record.mdoc

---
status: "published"
title: "Creating a Record | Next.js Supabase SaaS Kit"
label: "Creating a Record"
order: 0
description: "In this tutorial we show how you can create a record in Postgres with Supabase and Next.js"
---

Creating a record with Supabase is easy. We can use the `insert` method to create a record in a table.

We combine the mutation with Next.js Server Actions so that we get data revalidation on-the-fly - pretty magical! 🧙‍♂️

The Makerkit convention is to store these in files called `mutations.ts` in the `lib` folder of your entity.

For example, here is a mutations file for a `task` entity.

```tsx
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '~/database.types';

type Task = {
  id: string;
  name: string;
  organizationId: string;
  description: string;
  dueDate: Date;
  done: boolean;
};

export function createTask(
  client: SupabaseClient<Database>,
  task: Omit<Task, 'id'>
) {
  return client.from('tasks').insert({
    name: task.name,
    organization_id: task.organizationId,
    description: task.description,
    due_date: task.dueDate,
    done: task.done,
  });
}
```

Ideally, the `Task` interface is defined externally, but I am showing it here for simplicity and clarity.

### Calling the mutation in a Server Action

We can now export a server action that uses this function to create a task.

```tsx
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

import { createTask } from '~/lib/tasks/mutations';
import type Task from '~/lib/tasks/types/task';
import { withSession } from '~/core/generic/actions-utils';
import getSupabaseServerActionClient from '~/core/supabase/action-client';
import { parseOrganizationIdCookie } from '~/lib/server/cookies/organization.cookie';
import requireSession from '~/lib/user/require-session';

type CreateTaskParams = {
  task: Omit<Task, 'id'>;
  csrfToken: string;
};

export const createTaskAction = withSession(
  async (params: CreateTaskParams) => {
    const client = getSupabaseServerActionClient();
    const session = await requireSession(client);
    const uid = await parseOrganizationIdCookie(session.user.id);
    const path = `/dashboard/${uid}/tasks`;

    await createTask(client, params.task);

    // revalidate the tasks page
    revalidatePath(path, 'page');

    // redirect to the tasks page
    redirect(path);
  },
);
```

The `withSession` function is a helper that ensures that the user is logged in. If not, it will redirect to the login page. Use it for all server actions that require a user to be logged in.

### Calling the mutation in a React component

Finally, it's a matter of calling the Next.js Server Action within the React component that handles the form submission.

Assuming you have a `form` element, you can do something like this:

```tsx
import { createTaskAction } from '~/lib/tasks/actions';
import useCsrfToken from '~/core/hooks/use-csrf-token';

function TaskForm() {
  const csrfToken = useCsrfToken();

  return (
    <form action={createTaskAction}>
      <input type="text" name="task.name" />
      <input type="text" name="task.description" />
      <input type="date" name="task.dueDate" />
      <input type="hidden" name="csrfToken" value={csrfToken} />
      <button type="submit">Create Task</button>
    </form>
  );
}
```

Adding the CSRF token is required in all POST requests. You can use the `useCsrfToken` hook to get the token and passing it as a hidden input.

You can also call the function imperatively if you don't use a traditional form component.

```tsx
import { useTransition } from "react";
import useCsrfToken from '~/core/hooks/use-csrf-token';

const [isPending, starTransition] = useTransition()

const onSubmit = (task: Task) => {
  startTransition(async () => {
    await createTaskAction({ task, csrfToken });
  })
};
```

-----------------
FILE PATH: kits\next-supabase\how-to\database\generate-database-types.mdoc

---
status: "published"
title: "How to generate your Supabase Database types | Next.js Supabase SaaS Kit"
label: "Generating Database Types"
order: 2
description: "Learn how to generate your Supabase Database types using the typegen command in the Next.js Supabase SaaS Kit"
---

When you change your Database schema, you need to also remember to update the Typescript types that the Supabase Client can use to strong-type your queries and mutations.

Following a change in the schema, you need to do two things:

1. **Reset the Database**: if you have changed the schema manually, e.g. updating the SQL code, you need to either reset or restart your Supabase instance. You can do so using the command `npm run supabase:db:reset`
2. **Generate the new type definitions**: now, you can run the command `npm run typegen` to generate the types. The generated file will be available at `src/database.types.ts`

That's it! Always remember to update your schema types after changing your database schema.

-----------------
FILE PATH: kits\next-supabase\how-to\database\resetting-local-database.mdoc

---
status: "published"
title: "How to reset your local Database | Next.js Supabase SaaS Kit"
label: "Resetting the local DB"
order: 1
description: "Learn how to reset the seed data in your Next.js Supabase Database"
---

Resetting the DB can be useful when updating the schema of your database using the SQL schema files instead of the Supabase Studio UI.

To reflect the new changes, and/or resetting the database to the local seed, run the following command:

```
npm run supabase:db:reset
```

The database is now in the same state as when you started your Supabase server.

**Important**: users created during the session will be reset, which means your application may be in a conflicting state. Log out or delete the cookies to restart from scratch.

-----------------
FILE PATH: kits\next-supabase\how-to\database\seed-data.mdoc

---
status: "published"

title: "How to seed data in your Supabase application"
label: "Seeding Local Data"
order: 0
description: "Learn how to seed data in your Supabase Database"
---


The Makerkit repository adds some seed data by default for running the E2E tests.

To update the seed data, you will update the SQL file `supabase/seed.sql`. This file contains the SQL code that seeds the database everytime you start the Supabase Server using `npm run supabase:start`.

### Adding data to the seed file

If you'd like to extend this data with your own, update the data in your local environment, either by using your application or using the Supabase Studio interface.

Then, you can run the `diff` command:

```
supabase diff
```

This command will output the SQL code with the difference between the seed data and the current local DB data. You can copy and paste the outputted data into your `seed.sql` file.

The next database restarts will use the new data inserted.


-----------------
FILE PATH: kits\next-supabase\how-to\introduction.mdoc

---
status: "published"
title: "Introduction to the How to section of the Makerkit Next.js Supabase documentation"
label: Introduction
order: 0
description: "Here is a brief introduction to the How to section of the Makerkit Next.js Supabase documentation."
---

The "How to" documentation of Makerkit Next.js Supabase is a collection of tutorials that will help you to build your own web application using Next.js and Supabase using small and practical examples.

## What you will learn


1. Setting up your Makerkit Next.js Supabase project
2. Authentication
3. Updating the marketing pages of your SaaS
4. Interacting with the Database
5. Updating the Marketing pages of your SaaS
6. Updating the Internal pages of your SaaS
7. Setting up Payments
8. Updating Translations
9. ... and a lot more.

This documentation is under development and will be updated regularly. If you have any questions, please contact me.

-----------------
FILE PATH: kits\next-supabase\how-to\payments\configuring-plans.mdoc

---
status: "published"

title: "Configuring your SaaS Stripe Plans in your Makerkit Next.js Supabase App"
label: Configuring Plans
order: 0
description: "Learn how to configure the Stripe plans in your SaaS application with Makerkit"
---


When taking payments from Stripe, you're required to configure your subscription's plans.

Your plans need to be configured in two places:

1. **Stripe** - first, you need to define your **products** and **plans** in Stripe itself. This requires you to have an account in Stripe.
2. **Configuration**: - then, you will copy your plans data in the Makerkit configuration, which is used to render the Pricing table and display the prices metadata

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
          stripePriceId: '<price_id>',
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

#### Update the Stripe Price IDs values!

Please replace the property `stripePriceId` with the real price ID from the
Stripe Dashboard.

<div>
  {% alert type="warning" title="Update your real Stripe Price IDs" %}

         The properties "stripePriceId" are placeholders. You will need to replace them with the IDs of your plans.

{% /alert %}
</div>

### Additional Configuration

You can also configure the following properties on each `product`:

- `recommended` - set to `true` if you want to highlight a plan as recommended
- `badge` - set to a string if you want to display a badge next to the plan name

You can also configure the following properties on each `plan`:

- `price` - any string that you want to display as the price (e.g. `$9.99` or `Free`)
- `label` - set to a custom string if you want to display a button label
- `href` - set to a custom URL if you want to link to a custom page (for example, a contact page)
- `trialPeriodDays` - set to the number of days you want to offer a free trial for


-----------------
FILE PATH: kits\next-supabase\how-to\payments\stripe-webhooks-locally.mdoc

---
status: "published"

title: "Running the Stripe Webhook locally"
label: Running the Stripe Webhook locally
order: 1
description: "Want to test and run the Stripe Webhook locally? This guide will show you how to do it."
---


When testing webhooks sent by Stripe, it's useful to redirect them to our local webhook endpoint so that we can test them locally.

Makerkit has a built-in NPM script that starts the Stripe CLI and redirects webhooks to our local endpoint.

## Prerequisites

The only prerequisite is to have Docker installed and running on your machine.

## Running the Stripe Webhook locally

To run the Stripe Webhook locally, run the following command:

```bash
npm run stripe:listen
```

The first time you run this command, it will ask you to login to your Stripe account. Follow the instructions on the screen to do so.

Once you're logged in, run the command again:

```bash
npm run stripe:listen
```

This time, the Stripe CLI will start and redirect webhooks to our local endpoint.

You can now test your webhooks locally when testing your Stripe integration with Makerkit 🎉


-----------------
FILE PATH: kits\next-supabase\how-to\payments\using-lemon-squeezy.mdoc

---
status: "published"
title: "Using Lemon Squeezy instead of Stripe in your Next.js Supabase application"
label: Using Lemon Squeezy instead of Stripe
order: 4
description: "How to use Lemon Squeezy instead of Stripe in your Next.js Supabase application"
---

To use Lemon Squeezy instead of Stripe, you should use the branch `main-ls` to develop your SaaS.

The Lemon Squeezy configuration is slightly different from the Stripe configuration as it accepts a variant ID instead of a price ID.

```tsx
subscriptions: {
  plans: [
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
          variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$90',
           variantID: '<your variant ID>',
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
           variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$200',
          variantID: '<your variant ID>',
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
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ]
}
```

Please refer to the [complete Lemon Squeezy documentation](/docs/next-supabase/lemon-squeezy) for more details.

-----------------
FILE PATH: kits\next-supabase\how-to\server-actions\server-actions-csrf.mdoc

---
status: "published"
title: "Sending CSRF Token to Server Actions | Next.js Supabase SaaS Kit"
label: "Sending CSRF Token to Actions"
order: 1
description: "Learn how to send CSRF tokens to server actions in Next.js Supabase SaaS Kit"
---

Makerkit's Middleware ensures that POST requests to your server actions are protected against CSRF attacks. This means that you need to send a CSRF token with every POST request to your server actions. The only instance where you need to manually protect your Actions is when you use forms that send data using `FormData`.

## Getting the CSRF Token

To retrieve the CSRF token to send along with your requests, use the `useCsrfToken` hook. This hook returns the CSRF token as a string.

```tsx
import useCsrfToken from '~/core/hooks/use-csrf-token';

// somehwere in your component
const token = useCsrfToken();
```

## Sending the CSRF Token

When you call a Server Action, simply add the token as `csrfToken` in the parameters object.

For example, assuming this is our Server Action:

```tsx
type CreateTaskParams = {
  task: Omit<Task, 'id'>;
  csrfToken: string;
};

export const createTaskAction = withSession(
  async (params: CreateTaskParams) => {
    const client = getSupabaseServerActionClient();
    const session = await requireSession(client);
    const uid = await parseOrganizationIdCookie(session.user.id);
    const path = `/dashboard/${uid}/tasks`;

    await createTask(client, params.task);

    // revalidate the tasks page
    revalidatePath(path, 'page');

    // redirect to the tasks page
    redirect(path);
  },
);
```

NB: we added `csrfToken` to the `CreateTaskParams` type.

Then, we can call the Server Action like this:

```tsx
import { useTransition } from "react";
import useCsrfToken from '~/core/hooks/use-csrf-token';

function TaskForm() {
  const csrfToken = useCsrfToken();
  const [isMutating, startTransition] = useTransition();

  const onSubmit = (task: Task) => {
    startTransition(async () => {
      await createTaskAction({ task, csrfToken });
    });
  };
}
```

As you can see, we're passing an object with two properties: `task` and `csrfToken`. The `csrfToken` property is the one that will be used to send the CSRF token to the server.

## Using "FormData"

When using plain forms, we can pass the CSRF token as an input field.

```tsx
function TaskForm() {
  const csrfToken = useCsrfToken();

  return (
    <form action={createTaskAction}>
      <input type={'hidden'} name={'csrfToken'} value={csrfToken} />
    </form>
  );
}
```

In your server action, you will manually need to extract the CSRF token from the request body and verify it using the `verifyCsrfToken` function.

```tsx
import verifyCsrfToken from '~/core/verify-csrf-token';

export const createTaskAction = async (data: FormData) => {
  const csrfToken = data.get('csrfToken');

  await verifyCsrfToken(csrfToken);

  // ...
};
```

-----------------
FILE PATH: kits\next-supabase\how-to\server-actions\server-actions-error-handling.mdoc

---
status: "published"
title: "How to handle errors in Server Actions | Next.js Supabase SaaS Kit"
label: "Server Actions Error Handling"
order: 2
description: "Learn how to handle errors in Server Actions in the Next.js Supabase SaaS Kit"
---

Generally speaking, you can handle Server Action in three ways:

1. You can use the `try/catch` block to catch errors and handle them in the `catch` block.
2. You can use a React `ErrorBoundary` component to catch errors
3. You can also use the special `error.tsx` file to handle errors at layout level.

### Handling errors from a Server Action invoked by a Form

Let's assume we have a Form using a server action that throws an error:

```tsx title='Form.tsx'
async function serverActionWithError() {
  'use server';

  throw new Error(`This is error is in the Server Action`);
}

function FormWithServerAction() {
  return (
    <form action={serverActionWithError}>
      <button>Submit Form</button>
    </form>
  );
}

export default FormWithServerAction;
```

We can wrap our form using the `ErrorBoundary` component we have at `~/core/ui/ErrorBoundary`, and handle the error from there:

```tsx title='FormWithErrorBoundary.tsx'
import ErrorBoundary from '~/core/ui/ErrorBoundary';
import FormWithServerAction from './Form';

function FormWithErrorBoundary() {
  return (
    <ErrorBoundary fallback={<p>Something went wrong</p>}>
      <FormWithServerAction />
    </ErrorBoundary>
  );
}
```

If you click on the button, you'll see the error message rendered.

### Handling errors from an imperatively invoked Server Action

If you are invoking a Server Action imperatively, we can wrap the `useTransition` hook in a `try/catch` block, and handle the error from there.

Let's assume we have a Server Action that throws an error:

```tsx title='actions.tsx'
'use server';

export async function serverActionWithError() {
  throw new Error(`This is error is in the Server Action`);
}
```

We can invoke it imperatively from a client component, and handle the error from there:

```tsx title='ImperativeServerAction.tsx'
'use client';

import { useTransition } from 'react';
import { serverActionWithError } from './actions';

function ImperativeServerAction() {
  const [pending, startTransition] = useTransition();

  return (
    <button
      disabled={pending}
      onClick={() => {
        startTransition(async () => {
          try {
            await serverActionWithError()
          } catch (e) {
            alert('error');
          }
        });
      }}
    >
      Click Button
    </button>
  );
}

export default ImperativeServerAction;
```

If you click on the button, you'll see the `alert` popup with the error message.

-----------------
FILE PATH: kits\next-supabase\how-to\server-actions\server-actions.mdoc

---
status: "published"

title: "Server Actions"
label: "Server Actions"
order: 0
description: "Learn how to write a server action in your Next.js Supabase SaaS application"
---


Makerkit makes extensive use of Next.js Server Actions to mutate data in your Supabase database.

Server Actions are a powerful feature of Next.js that allow you to write server-side code that we can call from the client-side. This is useful for a number of reasons:
1. We don't need to write any API routes to handle our server-side logic
2. We can tell Next.js to revalidate our data after a server action has been called, thus refreshing our data on the client-side
3. We can easily call them just like any other function in our code

{% alert%}
This page is important to explain how we can handle Supabase mutations in the next sections.
{% /alert %}

## Creating a Server Action

To create a server action, we need to create a separate file marked with the `use server` directive at the top of the file. This tells Next.js that this file contains a server action that we can call from the client-side.

```tsx
'use server';

export async function handleFormSubmit(
  formData: FormData
) {
  // Do something with the form data
}
```

## Calling a Server Action

Calling a server action can be done in several ways

1. Calling it from a form submit handler
2. Calling it from a button click handler
3. Calling it imperatively as a function

### Calling it from a form submit handler

Let's assume we defined our server actions in a file called `actions.ts`:

```tsx
'use server';

export async function createPostAction(
  formData: FormData
) {
  // Do something with the form data
}
```

We can call this server action from a form submit handler like so:

```tsx
<form action={createPostAction}>
  <div className='flex flex-col space-y-4'>
    <h2 className='text-lg font-semibold'>Create a new Post</h2>

    <Label className='flex flex-col space-y-1.5'>
      <span>Title</span>
      <Input name='title' placeholder='Ex. The best Next.js libraries' required />
    </Label>

    <Label className='flex flex-col space-y-1.5'>
      <span>Description</span>
      <Input />
    </Label>

    <SubmitButton />
  </div>
</form>
```

### Calling it from a button click handler

We can also call a server action from a button click handler:

```tsx
<form action={createPostAction}>
  <div className='flex flex-col space-y-4'>
    <h2 className='text-lg font-semibold'>Create a new Post</h2>

    <Label className='flex flex-col space-y-1.5'>
      <span>Title</span>
      <Input name='title' placeholder='Ex. The best Next.js libraries' required />
    </Label>

    <Label className='flex flex-col space-y-1.5'>
      <span>Description</span>
      <Input />
    </Label>

    <button formAction={createPostAction}>Click</button>
  </div>
</form>
```

### Calling it imperatively as a function

Finally, we can call a server action imperatively as a function using `startTransition`:

```tsx
import { useTransition } from "react";

function ClientComponent({ id }) {
  let [isPending, startTransition] = useTransition();

  return (
    <button onClick={() => startTransition(() => createPostAction(id))}>
      Save
    </button>
  );
}
```

## Send CSRF token with server actions

Remember: Makerkit protects by default every POST request with a CSRF token. This is a security measure to prevent CSRF attacks.

It's your job to send this along when you call a server action.

