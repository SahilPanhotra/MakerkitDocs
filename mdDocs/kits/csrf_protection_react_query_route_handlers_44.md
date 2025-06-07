-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching\csrf-protection.mdoc

---
status: "published"
title: "CSRF Protection for your API Routes"
description: "Learn how to protect your API routes against CSRF attacks."
label: "CSRF Protection"
order: 6
---

{% sequence title="How to protect your API routes against CSRF attacks" description="Learn how to protect your API routes against CSRF attacks." %}

[CSRF Protection](#csrf-protection)

[Passing the CSRF Token to API Routes](#passing-the-csrf-token-to-api-routes)

{% /sequence %}

## CSRF Protection

By default, all POST, PUT, PATCH and DELETE requests to your API routes are protected against CSRF attacks. This means that you need to send a CSRF token with your requests, otherwise they will be rejected.

There are two exceptions to this rule:

1. **Server Actions**: When using Server Actions, protection is built-in and you don't need to worry about it.
2. **API Routes under /api**: When the route is under the `api` path. In this case, the CSRF protection is disabled, since we use this prefix for webhooks or external services that need to access your API.

## Passing the CSRF Token to API Routes

When using requests against API Route Handlers, you need to pass the CSRF token with your requests, otherwise the middleware will reject them.

To retrieve the CSRF token, you can use the `useGetCsrfToken` function from `@kit/shared/hooks`:

```tsx
'use client';

function MyComponent() {
  const csrfToken = useGetCsrfToken();

  const handleSubmit = async (e) => {
    e.preventDefault();

    const response = await fetch('/my-endpoint', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken,
      },
      body: JSON.stringify({ message: 'Hello, world!' }),
    });
  };

  // your component code
}
```

-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching\react-query.mdoc

---
status: "published"
description: "React Query is the best React library for managing asynchronous data fetching in your applications. Learn how to use it with Next.js Supabase."
title: "Using React Query with Next.js Supabase"
label: "React Query"
order: 5
---

{% sequence title="How to use React Query with Next.js Supabase" description="React Query is the best React library for managing asynchronous data fetching in your applications. Learn how to use it with Next.js Supabase." %}

[Using React Query](#using-react-query)

[Use the `use client` directive](#use-the-use-client-directive)

[Fetching data from Supabase using React Query](#fetching-data-from-supabase-using-react-query)

{% /sequence %}

## Using React Query

Makerkit uses React Query for client-side data fetching. This allows you to fetch data from your Supabase database and display it on the client-side using an ergonomic API.

### Use the `use client` directive

Since we use React Hooks, you need to use the `use client` directive at the top of your component. If you use this code inside a page, please write a separate component (with `use client`) and import it in your page.

### Fetching data from Supabase using React Query

In the example below, we fetch the tasks from the Supabase database and display them in a list.

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { useSupabase } from '@kit/supabase/hooks/use-supabase';

function TasksList(accountId: string) {
  const client = useSupabase();

  const { data, isLoading, error } = useQuery({
    queryKey: ['tasks', accountId],
    queryFn: async () => {
      const { data, error } = await client
        .from('tasks')
        .select('*')
        .eq('account_id', accountId)
        .order('created_at', { ascending: false });

      if (error) {
        throw error;
      }

      return data;
    }
  });

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error loading tasks</div>;
  }

  return (
    <ul>
      {data.map(task => (
        <li key={task.id}>{task.title}</li>
      ))}
    </ul>
  );
}
```

By leveraging the `useSupabase` hook, you can access the Supabase DB data right from your React components.

Similarly, you can use React Query for mutations:

```tsx
import { useMutation } from '@tanstack/react-query';
import { useSupabase } from '@kit/supabase/hooks/use-supabase';

function CreateTaskForm() {
  const client = useSupabase();

  const mutation = useMutation({
    mutationFn: async (data) => {
      const { data, error } = await client
        .from('tasks')
        .insert({
          title: data.title,
          description: data.description,
          account_id: data.accountId,
        })
        .select('*')
        .single();

      if (error) {
        throw error;
      }

      return data;
    },
  });

  const handleSubmit = async (data) => {
    const { data, error } = await mutation.mutateAsync(data);

    if (error) {
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="title" />
      <input type="text" name="description" />
      <button type="submit">Create Task</button>
    </form>
  );
}
```

-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching\route-handlers.mdoc

---
status: "published"
title: "Using API Route Handlers in the Next.js Supabase SaaS Kit"
label: "Route Handlers"
description: "Learn how to write Route Handlers in the Next.js Supabase SaaS Kit to fetch and write data"
order: 2
---

{% sequence title="How to write API Route Handlers in the Next.js Supabase SaaS Kit" description="Learn how to write Route Handlers in the Next.js Supabase SaaS Kit to fetch and write data" %}

[What are API Route Handlers?](#what-are-api-route-handlers)

[Using the `enhanceRouteHandler` utility](#using-the-enhance-route-handler-utility)

[Using a Captcha token protection](#using-a-captcha-token-protection)

[Passing the CSRF Token to API Routes](#passing-the-csrf-token-to-api-routes)

{% /sequence %}

## What are API Route Handlers?

API Route handlers are added using the convention `route.ts` and exporting one or many HTTP methods (e.g., `GET`, `POST`, `PUT`, `DELETE`).

In the older Pages Router, we used to write API routes using the `api` folder. However, in the new App Router, simply use the `route.ts` file convention to define your API routes.

## Using the enhanceRouteHandler utility

While you won't be writing too many API route handlers (prefer Server Actions for mutations) - you can use the `enhanceRouteHandler` utility to help you with the following:

1. **User state**: checks the user state (if the user is authenticated)
2. **Validation**: given a Zod schema, it validates the request body
3. **Captcha**: given a captcha site key, it validates the captcha token

Fantastic, let's see how we can use it

```tsx
import { z } from 'zod';

import { enhanceRouteHandler } from '@kit/next/routes';
import { NextResponse } from 'next/server';

const ZodSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export const POST = enhanceRouteHandler(
  async function({ body, user, request }) {
    // 1. "body" has been validated against the Zod schema, and it's safe to use
    // 2. "user" is the authenticated user
    // 3. "request" is the request object that contains the headers, query, etc.

    // ... your code here
    return NextResponse.json({
      success: true,
    });
  },
  {
    schema: ZodSchema,
  },
);
```

### Using a Captcha token protection

If you want to protect your API route handlers with a captcha token, you can do so by passing the captcha site token to the `enhanceRouteHandler` function and setting the `captcha` flag to `true`.

```tsx
import { enhanceRouteHandler } from '@kit/next/routes';

export const POST = enhanceRouteHandler(
  async function({ body, user, request }) {
    // ... your code here
    return NextResponse.json({
      success: true,
    });
  },
  {
    captcha: true,
    schema: ZodSchema,
  },
);
```

When calling the API route handler, we must supply the captcha token in the request body.

The captcha token can be retrieved from the `useCaptchaToken` hook in the package `@kit/auth/captcha/client`.

```tsx
import { useCaptchaToken } from '@kit/auth/captcha/client';

function Component() {
  const captchaToken = useCaptchaToken();
  
  // ... your code here
}
```

## Passing the CSRF Token to API Routes

When calling the API route handler, we can pass the captcha and the CSRF token. This is required if:

1. It's a mutation
2. The route is not inside the `/api` folder

NB: The CSRF token **must be added for all API routes** making mutations in routes that are outside `/api/*`. Routes inside `/api/*` are not protected by default as they're meant to be used externally.

```tsx
import { useCaptchaToken } from '@kit/auth/captcha/client';
import { useCsrfToken } from '@kit/shared/hooks';

function Component() {
  const captchaToken = useCaptchaToken();
  const csrfToken = useCsrfToken();
  
  const onSubmit = async (params: {
    email: string;
    password: string;
  }) => {
    const response = await fetch('/my-api-route', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-csrf-token': csrfToken,
        'x-captcha-token': captchaToken,
      },
      body: JSON.stringify(params),
    });
    
    // ... your code here
  };
  
  // ... your code here
}
```

You can improve the above using React Query:

```tsx
import { useMutation } from '@tanstack/react-query';
import { useCaptchaToken } from '@kit/auth/captcha/client';

function Component() {
  const captchaToken = useCaptchaToken();
  const mutation = useMutateData();

  const onSubmit = async (params: {
    email: string;
    password: string;
  }) => {
    await mutation.mutateAsync(params);
  };
  
  // ... your code here
}

function useMutateData() {
  return useMutation({
    mutationKey: 'my-mutation',
    mutationFn: async (params: { email: string; password: string }) => {
      const response = await fetch('/my-api-route', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-captcha-token': captchaToken,
        },
        body: JSON.stringify(params),
      });
      
      if (!response.ok) {
        throw new Error('An error occurred');
      }

      return response.json();
    },
  });
}
```

NB: to use Captcha protection, you need to set the captcha token in the environment variables.

```bash
CAPTCHA_SECRET_TOKEN=
NEXT_PUBLIC_CAPTCHA_SITE_KEY=
```

As a secret environment variable, please do not add it to the `.env` file. Instead, add it to the environment variables of your CI/CD system.

The only captcha provider supported is Cloudflare Turnstile.

-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching\server-actions.mdoc

---
status: "published"
title: "Using Server Actions in the Next.js Supabase SaaS Kit"
label: "Server Actions"
description: "Learn how to write Server Actions to mutate and revalidate your data"
order: 1
---

{% sequence title="How to write Server Actions in the Next.js Supabase SaaS Kit" description="Learn how to write Server Actions to mutate and revalidate your data" %}

[What are Server Actions?](#what-are-server-actions)

[Using the enhanceAction utility](#using-the-enhance-action-utility)

[Using a Captcha token protection](#using-a-captcha-token-protection)

[Passing the Captcha Token to a Server Action](#passing-the-captcha-token-to-a-server-action)

{% /sequence %}

## What are Server Actions?

Server Actions help us communicate with the server just by creating normal Javascript functions that Next.js converts to server POST endpoints. They are particularly useful to mutate data and revalidate the data that we fetched from Server Components.

Generally speaking, there is nothing special in Makerkit compared to any other Next.js application in how you will use Server Actions. However, I want to introduce you to a nifty utility to make your life easier while writing Server Actions: `enhanceAction`.

Let's first introduce Server Actions.

In the large majority of cases - you will be writing React Server Actions to update data in your DB. Server Actions are used to perform mutations on the server-side - while being called like normal functions.

To create a server action, it's enough to add `use server` at the top of the file and export the function.

```tsx
'use server';

// I am now a server action!
export const myServerAction = async function () {
  // ... your code here
  return {
    success: true,
  };
};
```

The above is a plain POST request that basically does nothing. Let's see how we can make it more useful.

Makerkit ships with a utility to help you write these actions. The utility is called `enhanceAction` and we import it from `@kit/next/actions`.

```tsx
import { enhanceAction } from '@kit/next/actions';
```

This utility helps us with three main things:
1. checks the user state (if the user is authenticated)
2. given a Zod schema, it validates the request body
3. given a captcha site key, it validates the captcha token
4. if you configured a monitoring provider, it sends the caught exception to the monitoring provider

Fantastic, let's see how we can use it.

```tsx
'use server';

import { z } from 'zod';
import { enhanceAction } from '@kit/next/actions';

const ZodSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export const myServerAction = enhanceAction(
  async function (data, user) {
    // 1. "data" has been validated against the Zod schema, and it's safe to use
    // 2. "user" is the authenticated user
    
    // ... your code here
    return {
      success: true,
    };
  },
  {
    schema: ZodSchema,
  },
);
```

### Using a Captcha token protection

If you want to protect your server actions with a captcha token, you can do so by passing the captcha site token to the `enhanceAction` function and setting the `captcha` flag to `true`.

```tsx
'use server';

import { enhanceAction } from '@kit/next/actions';

export const myServerAction = enhanceAction(
  async function (data, user) {
    // ... your code here
    return {
      success: true,
    };
  },
  {
    captcha: true,
    schema: ZodSchema,
  },
);
```

When calling the server action, we must supply the captcha token in the request body.

The captcha token can be retrieved from the `useCaptchaToken` hook in the package `@kit/auth/captcha/client`.

```tsx
import { useCaptchaToken } from '@kit/auth/captcha/client';

function Component() {
  const captchaToken = useCaptchaToken();
  
  // ... your code here
}
```

Now, when calling the server action, we can pass the captcha

```tsx
import { useCaptchaToken } from '@kit/auth/captcha/client';

function Component() {
  const captchaToken = useCaptchaToken();
  
  const onSubmit = async (params: {
    email: string;
    password: string;
  }) => {
    const response = await myServerAction({
      ...params,
      captchaToken,
    });
    
    // ... your code here
  };
}
```

NB: to use Captcha protection, you need to set the captcha token in the environment variables.

```bash
CAPTCHA_SECRET_TOKEN=
NEXT_PUBLIC_CAPTCHA_SITE_KEY=
```

As a secret environment variable, please do not add it to the `.env` file. Instead, add it to the environment variables of your CI/CD system.

The only captcha provider supported is Cloudflare Turnstile.

#### Passing the Captcha Token to a Server Action

NB: you must pass the captcha token in the request body when calling the server action. The function's type checker will ensure that you pass the captcha token as it will error out if the token is not defined in the Zod schema as `captchaToken`.


-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching\server-components.mdoc

---
status: "published"
title: "Fetching data from Server Components"
label: "Server Components"
description: "Learn how to fetch data from Server Components in the Next.js Supabase SaaS Kit"
order: 3
---

{% sequence title="How to fetch data from Server Components in the Next.js Supabase SaaS Kit" description="Learn how to fetch data from Server Components in the Next.js Supabase SaaS Kit" %}

[What are Server Components?](#what-are-server-components)

[Fetching data from Supabase in a Server Component](#fetching-data-from-supabase-in-a-server-component)

{% /sequence %}

## What are Server Components?

Server Components are the primary way we fetch and render data in the Next.js Supabase SaaS kit. 

When you create a new page and want to fetch some data, Server Components are the perfect place where to fetch it: it is done when you render your page (which means one less round-trip) and the data is streamed to the client (so it's very fast).

In Next.js, every component is a Server Component, unless you specify `use client`, which converts it to a client component. 

1. Client components are both server-rendered and client-rendered.
2. Server Components are only rendered on the server - so we can use data-fetching methods (using Supabase, in our case) and fetch all the required data for a particular layout or page. In general, you can use any server-side code inside a Server Component.

### Fetching data from Supabase in a Server Component

For example, let's assume we have a page that displayes a list of tasks from a `tasks` table. This is a Next.js page, and therefore a Server Component. It runs **on the server and on the server only**, and we can use to fetch data and render it streamed to the client. 

The client will never run the React code in this component: it will only render and bundle the code/data that is sent to it.

```tsx
export default async function TasksPage() {}
```

Let's now fetch some data from Supabase. To do so, we use the server component client.

```tsx
const supabase = getSupabaseServerClient();

const { data, error } = await supabase.from('tasks').select('*');
```

Now, let's put it all together:

```tsx
export default async function TasksPage() {
  const supabase = getSupabaseServerClient();

  const { data, error } = await supabase.from('tasks').select('*');

  if (error) {
    return <p>Error :(</p>;
  }

  return <TasksList data={data}>
}
```

As you can see, we are fetching data and rendering it in `TasksList`. All on the server.

-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching\supabase-clients.mdoc

---
status: "published"
title: "Supabase Clients in Next.js Supabase Turbo"
label: "Supabase Clients"
description: "Depending on whether you are running your code in the browser or on the server, you will need to use different clients to interact with Supabase."
order: 0
---

Before diving into the various ways we can communicate with the server, we need to introduce how we communicate with Supabase, which is hosting the database and therefore the source of our data.

{% sequence title="How to use the Supabase client" description="Depending on whether you are running your code in the browser or on the server, you will need to use different clients to interact with Supabase." %}

[Using the Supabase client in the browser](#using-the-supabase-client-in-the-browser)

[Using the Supabase client in a Server environment](#using-the-supabase-client-in-a-server-environment)

[Important Note about the Server Admin Client](#important-note-about-the-server-admin-client)

{% /sequence %}

## Using the Supabase client

Depending on whether you are running your code in the browser or on the server, you will need to use different clients to interact with Supabase.

### Using the Supabase client in the browser

To import the Supabase client in a browser environment,  you can use the `useSupabase` hook:

```tsx
import { useSupabase } from '@kit/supabase/hooks/use-supabase';

export default function Home() {
  const supabase = useSupabase()

  return (
    <div>
      <h1>Supabase Browser Client</h1>
      <button onClick={() => supabase.auth.signOut()}>Sign Out</button>
    </div>
  )
}
```

### Using the Supabase client in a Server environment

To import the Supabase client in a server environment, you can use the `getSupabaseServerClient` function, and you can do the same across all server environments like Server Actions, Route Handlers, and Server Components:

```tsx
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function myServerAction() {
  const supabase = getSupabaseServerClient();

  const { data, error } = await supabase.from('users').select('*')

  return {
    success: true,
  };
}
```

To use the Server Admin, e.g. an admin client with elevated privileges, you can use the `getSupabaseServerAdminClient` function:

```tsx
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

export async function myServerAction() {
  const supabase = getSupabaseServerAdminClient();

  const { data, error } = await supabase.from('users').select('*')

  return {
    success: true,
  };
}
```

### Important Note about the Server Admin Client

The `getSupabaseServerAdminClient` function should only be used in server environments, as it requires the `SUPABASE_SERVICE_ROLE_KEY` environment variable to be set. Additionally, it should only be used in very exceptional cases, as it has elevated privileges. In most cases, please use the `getSupabaseServerClient` function.

The admin client uses the service role key, which has elevated privileges and should be used with extreme caution. For example, it can be used to bypass Row Level Security (RLS) policies. If you use the admin client, always make sure the server-side code makes the necessary checks to ensure the data is not exposed to unauthorized users.

### DEPRECATED: Older Versions of the Kit

NB: the below applies to older versions of the kit.

In older versions of the kit, you may see different ways of importing the Supabase client. The code below will work, but please note that it is deprecated and will be removed in future versions of the kit.

Depending on where your code runs, you may need to use different clients to interact with Supabase. This is due to how cookies are set differently in various parts of the Next.js application.

You can use 4 different clients to interact with Supabase:

1. **Browser** - This runs in the browser and is used to interact with Supabase from the client
2. **Server Actions** - This runs in Server Actions and is used to interact with Supabase from the server
3. **Route Handlers** - This runs in Route Handlers and is used to interact with Supabase from the server
4. **Server Components** - This runs in Server Components and is used to interact with Supabase from the server

## Browser

To use the browser client, use the `useSupabase` hook:

```tsx
import { useSupabase } from '@kit/supabase/hooks/use-supabase';

export default function Home() {
  const supabase = useSupabase()

  return (
    <div>
      <h1>Supabase Browser Client</h1>
      <button onClick={() => supabase.auth.signOut()}>Sign Out</button>
    </div>
  )
}
```

## Server Actions

To use the server actions client, use the `getSupabaseServerActionClient` hook:

```tsx
'use server';

import { getSupabaseServerActionClient } from '@kit/supabase/server-actions-client';

export async function myServerAction() {
  const supabase = getSupabaseServerActionClient();

  const { data, error } = await supabase.from('users').select('*')

  return {
    success: true,
  };
}
```

## Route Handlers

To use the route handlers client, use the `getSupabaseRouteHandlerClient` function:

```tsx
import { NextRequest, NextResponse } from 'next/server';
import { getSupabaseRouteHandlerClient } from '@kit/supabase/route-handlers-client';

export async function POST(req: NextRequest) {
  const supabase = getSupabaseRouteHandlerClient();

  const { data, error } = await supabase.from('users').select('*')

  return NextResponse.json({ data });
}
```

## Server Components

To use the server components client, use the `getSupabaseServerComponentClient` function:

```tsx
import { getSupabaseServerComponentClient } from '@kit/supabase/server-component-client';

export default async function TasksPage() {
  const supabase = getSupabaseServerComponentClient();

  const { data, error } = await supabase.from('users').select('*');

  return <TasksList tasks={data} />
}
```

-----------------
FILE PATH: kits\next-supabase-turbo\data-fetching.mdoc

---
status: "published"
title: "Data Fetching in Next.js Supabase Turbo"
label: "Data Fetching"
order: 5
description: "Data Fetching in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Here's a draft of the Data Fetching pillar page for Makerkit:

# Data Fetching in Makerkit

Makerkit provides a robust approach to fetching and managing data in your Next.js Supabase application. Whether you're building server components, handling real-time updates, or managing client-side state, Makerkit offers patterns and utilities to make data fetching clean and efficient.

## Data Loading Patterns

Common patterns for loading data include:

- Server Component data fetching for initial page loads
- React Query for client-side data management
- Server Actions for mutations
- CSRF and Captcha protection for API Routes

## Navigation

In the section below, you'll find detailed information on each of these patterns:

- [Supabase Clients](data-fetching/supabase-clients)
- [Server Components](data-fetching/server-components)
- [Server Actions](data-fetching/server-actions)
- [Route Handlers](data-fetching/route-handlers)
- [CSRF Protection](data-fetching/csrf-protection)
- [Captcha Protection](data-fetching/captcha-protection)

-----------------
FILE PATH: kits\next-supabase-turbo\dev-tools\environment-variables.mdoc

---
status: "published"
description: "The Next.js Supabase Turbo Dev Tools allows you to debug environment variables using contextual information and error messages."
title: "Debugging Environment Variables using the Next.js Supabase Turbo Dev Tools"
label: "Environment Variables"
order: 0
---

The Next.js Supabase Turbo Dev Tools allows you to debug environment variables using contextual information and error messages.

{% sequence title="How to debug environment variables using the Next.js Supabase Turbo Dev Tools" description="The Next.js Supabase Turbo Dev Tools allows you to debug environment variables using contextual information and error messages." %}

[Getting Started with the Dev Tool](#getting-started-with-the-dev-tool)

[Development Mode](#development-mode)

[Production Mode](#production-mode)

[Contextual Validation](#contextual-validation)

[Using the Dev Tool to Debug Environment Variables](#using-the-dev-tool-to-debug-environment-variables)

[Debugging Production Environment Variables](#debugging-production-environment-variables)

[Adding your own Environment Variables](#adding-your-own-environment-variables)

{% /sequence %}

{% img src="/assets/images/dev-tools-env-variables.webp" width="1000"
height="600" /%}

## Getting Started with the Dev Tool

If you run the `pnpm run dev` command, you will see the Dev Tools at `http://localhost:3010/variables`.

You can choose two different modes:

1. **Development Mode**: This mode is used when you run the `pnpm run dev` command
2. **Production Mode**: This mode is used when you run the `pnpm run build` command

### Development Mode

In the Development Mode, the Dev Tools will show you the environment variables used during development. This is useful when you are developing your application and want to see the environment variables that are used in your application.

This mode will use the variables from the following files:

1. `.env`
2. `.env.development`
3. `.env.local`

### Production Mode

In the Production Mode, the Dev Tools will show you the environment variables used during production. This is useful when you are deploying your application and want to see the environment variables that are used in your application.

This mode will use the variables from the following files:

1. `.env`
2. `.env.production`
3. `.env.local`
4. `.env.production.local`

### Generating Environment Variables for Production

The right-hand side of the Dev Tool shows the effective environment variables that will be used in production. That is, the value that will ultimately be used in your application based on where they're defined.

The "Copy" button will copy the environment variables to your clipboard in a format that is ready to be pasted into your hosting provider's environment variables.

The Copy button will merge the variables from each environment file using the effective order of resolution.

The recommendation is to create a new file at `apps/web/.env.production.local` and paste the copied variables into it, so that they will override the variables in the other files. Afterwards, copy the result using the "Copy" button again and paste it into your hosting provider's environment variables.

### Contextual Validation

Thanks to contextual validation, we can validate environment variables based
on the value of other environment variables. For example, if you have a variable
`NEXT_PUBLIC_BILLING_PROVIDER` with the value `stripe`, we can validate that
all the variables required for Stripe are set.

## Using the Dev Tool to Debug Environment Variables

The Dev tool shows at a glance the current state of your environment variables. It also shows you the errors that might occur when using the environment variables.

1. **Valid**: This shows you the valid values for the environment variable. Bear in mind, the fact a value is valid does not mean it is correct. It only means that the data-type is correct for the variable.
2. **Invalid**: This shows you the errors that might occur when using the environment variable. For example, if you try to use an invalid data-type, the Dev Tool will show you an error message. It will also warn when a variable is required but not set.
3. **Overridden**: This shows you if the environment variable is overridden. If the variable is overridden, the Dev Tool will show you the value that is being used.

Use the filters to narrow down the variables you want to debug.

### Debugging Production Environment Variables

Of course, most of your production environment variables will be set in your hosting provider for security reasons. To temporarily debug your production environment variables, you can use the following steps:

1. Copy the variables from your hosting provider
2. Create a file at `apps/web/.env.production.local`. This file will be ignored by Git.
3. Paste the copied variables into the `apps/web/.env.production.local` file.
4. Use `Production` as the mode in the Dev Tool.
5. Analyze the data in the Dev Tool.

NB: Delete the `apps/web/.env.production.local` file when you're done or store it securely.

## Adding your own Environment Variables

During your development workflow, you may need to add new environment
variables. So that you can debug your application, you can add your own
environment variables to the Dev Tool.

Let's assume you want to add the following environment variables:

```bash
NEXT_PUBLIC_ANALYTICS_ENABLED=true
NEXT_PUBLIC_ANALYTICS_API_KEY=value
```

To add these variables, you need to create a new file in the `apps/dev-tool/app/variables/lib/env-variables-model.ts` file.

The file should look like this:

```tsx {% title="apps/dev-tool/app/variables/lib/env-variables-model.ts" %}
[
  {
    name: 'NEXT_PUBLIC_ANALYTICS_ENABLED',
    description: 'Enables analytics',
    category: 'Analytics',
    validate: ({ value }) => {
      return z
        .coerce
        .boolean()
        .optional()
        .safeParse(value);
    },
  },
  {
  name: 'NEXT_PUBLIC_ANALYTICS_API_KEY',
  description: 'API Key for the analytics service',
  category: 'Analytics',
  contextualValidation: {
    dependencies: [{
      variable: 'NEXT_PUBLIC_ANALYTICS_ENABLED',
      condition: (value) => {
        return value === 'true';
      },
      message:
        'NEXT_PUBLIC_ANALYTICS_API_KEY is required when NEXT_PUBLIC_ANALYTICS_ENABLED is set to "true"',
    }],
    validate: ({ value }) => {
     return z
        .string()
        .min(1, 'An API key is required when analytics is enabled')
        .safeParse(value);
    }
  }
}]
```

In the above, we added two new environment variables: `NEXT_PUBLIC_ANALYTICS_ENABLED` and `NEXT_PUBLIC_ANALYTICS_API_KEY`.

We also added a validation function for the `NEXT_PUBLIC_ANALYTICS_API_KEY` variable. This function checks if the `NEXT_PUBLIC_ANALYTICS_ENABLED` variable is set to `true`. If it is, the `NEXT_PUBLIC_ANALYTICS_API_KEY` variable becomes required. If it is not, the `NEXT_PUBLIC_ANALYTICS_API_KEY` variable is optional.

In this way, you can make sure that your environment variables are valid and meet the requirements of your application.

