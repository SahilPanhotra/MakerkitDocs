-----------------
FILE PATH: kits/remix-supabase/plugins/cookie-banner.mdoc

---
status: "published"

title: "Cookie Banner Plugin for the Remix Supabase Saas Starter Kit"
label: "Cookie Banner"
order: 0
description: "This plugin adds a cookie banner to your Remix Supabase application. It's a simple component that will store the user's consent in a cookie. This banner will stop appearing once the user has given their consent - or rejected it."
---


This plugin adds a cookie banner to your Remix application. It's a simple
component that will store the user's consent in a cookie. This banner will
stop appearing once the user has given their consent - or rejected it.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/cookie-banner git@github.com:makerkit/remix-supabase-saas-kit-plugins.git cookie-banner --squash
```

After running this command, you will have the plugin in your repository at
`plugins/cookie-banner`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```
npx @makerkit/cli plugins install
```

Follow the instructions to install the plugin.

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:

1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/cookie-banner https://github.com/makerkit/remix-supabase-saas-kit-plugins cookie-banner --squash
```

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

#### Add the plugin path to your Tailwind configuration

You can do so by adding the following to your `tailwind.config.js` file:

```js
content: ['./app/**/*.{ts,tsx,jsx,js}', './plugins/**/*.{ts,tsx,jsx,js}']
```

This is only needed the first time you install a plugin.

### Translations

Add the translations to your `public/locales/en/common.json` file:

```json
{
  "cookieBanner": {
    "title": "Hi, we use cookies.",
    "description": "This website uses cookies to ensure you get the best
    experience on our website.",
    "reject": "Reject",
    "accept": "Accept"
  }
}
```

### Importing the Plugin

You can import the `CookieBanner` component from the plugin in your `_app.
tsx` file and add it to your application:

```tsx
import CookieBanner from '~/plugins/cookie-banner/CookieBanner';

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

```
git subtree pull --prefix plugins/cookie-banner git@github.com:makerkit/next-supabase-saas-kit-plugins.git cookie-banner --squash
```


-----------------
FILE PATH: kits/remix-supabase/plugins/feedback-popup.mdoc

---
status: "published"

title: "Feedback Popup Plugin for the Remix Supabase Saas Starter Kit"
label: "Feedback Popup"
order: 1
description: "This plugin adds a feedback popup to your Remix Supabase application. You can import this component anywhere in your application. Also - this plugin provides some admin components to manage and view the feedback submitted by your users."
---


This plugin adds a feedback popup to your Remix application. You can
import this component anywhere in your application.

Also - this plugin provides some admin components to manage and view the
feedback submitted by your users.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/feedback-popup git@github.com:makerkit/remix-supabase-saas-kit-plugins.git feedback-popup --squash
```

After running this command, you will have the plugin in your repository at
`plugins/feedback`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```
npx @makerkit/cli@latest plugins install
```

Follow the instructions to install the plugin.

#### If the installation fails

Some users are not able to install using the GitHub SSH URL. If you're having issues with that:

1. properly set up SSH access to GitHub with your SSH key
2. use the HTTPS URL instead of the SSH URL

To use the HTTPS URL, you can run the following command:

```bash
git subtree add --prefix plugins/feedback-popup https://github.com/makerkit/remix-supabase-saas-kit-plugins feedback-popup --squash
```

#### Add the plugin as a workspace in your package.json

You can do so by adding the following to your `package.json` file:

```json
{
  "workspaces": [
    "plugins/feedback"
  ]
}
```

Add it next to the other workspaces in your `package.json` file.

#### Add the plugin path to your Tailwind configuration

You can do so by adding the following to your `tailwind.config.js` file:

```js
  content: ['./app/**/*.{ts,tsx,jsx,js}', './plugins/**/*.{ts,tsx,jsx,js}'],
```

This is only needed the first time you install a plugin.

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

### Installing dependencies

To install the dependencies, you can run the following command:

```
npm i
```

NPM will install the dependencies in the `plugins/feedback` folder as an NPM
workspace.

### Configuring the database migration

This plugin needs to store the documents that the AI will be trained on as
vectors in the Supabase database. As such, we need to add a migration to the
database.

The migration SQL file is available at `plugins/feedback/migration.sql`. You
can generate a new mutation using the following command:

```
supabase migration new feedback
```

Make sure you're happy with the default table names before proceeding so
that they don't conflict with your existing tables.

Now paste the content of the `plugins/feedback/migration.sql` file into the
new migration file that was created.

#### Pushing the migration to production

**When going to production**, you will need to run the migration using the
Supabase CLI:

```
supabase db push
```

You don't need to run this command in development.

### Importing the Plugin

You can import the `Chatbot` component from the plugin in any of your pages:

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

Create the following file at `app/routes/admin.feedback._index.tsx`:

```tsx
import * as feedbackSubmissionsPage from '~/plugins/feedback-popup/admin/FeedbackSubmissionsPage';

export default feedbackSubmissionsPage.default;

export const loader = feedbackSubmissionsPage.loader;
export const meta = feedbackSubmissionsPage.meta;
export const action = feedbackSubmissionsPage.action;
```

And the following file at `app/routes/admin.feedback.$id._index.tsx`:

```tsx
import * as feedbackSubmissionPage from '~/plugins/feedback-popup/admin/FeedbackSubmissionPage';

export default feedbackSubmissionPage.default;

export const loader = feedbackSubmissionPage.loader;
export const meta = feedbackSubmissionPage.meta;
```

Create the route `app/routes/resources.feedback.tsx` and add the following
content:

```tsx
import type { ActionFunctionArgs } from '@remix-run/node';
import { handleFeedbackAction } from '~/plugins/feedback-popup/lib/handle-feedback-action';

export async function action(args: ActionFunctionArgs) {
  return handleFeedbackAction(args);
}
```

Finally, add the page to the sidebar by updating the `AdminSidebar` component:

```tsx
<SidebarItem
  path={'/admin/feedback'}
  Icon={() => <ChatBubbleLeftRightIcon className={'h-6'} />}
>
  Feedback
</SidebarItem>
```

That's it! You should now be able to access the admin pages at `/admin/feedback` and `/admin/feedback/[id]`.

-----------------
FILE PATH: kits/remix-supabase/plugins.mdoc

---
status: "published"
title: "Plugins in Remix Supabase"
label: "Plugins"
order: 14
description: "Plugins in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/remix-supabase/sdk/data-loader.mdoc

---
status: "published"
title: "Data Loader SDK - Fetching data from Remix Supabase directly from declarative React Components"
label: "Data Loader SDK"
order: 5
description: "Fetching data from Remix Supabase directly from declarative React Components using the Data Loader SDK - a lightweight wrapper around the Supabase JS Client."
---

The Data Loader SDK is a lightweight open-source wrapper around the Supabase JS Client. [The source code can be found here](https://github.com/makerkit/makerkit).

The Data Loader SDK provides a declarative way to fetch data from Supabase directly from React components - from both React Client and Server components (RSC). This means they play well both using RSC and SSR/SPA - so you can choose the approach that best fits your needs.

You can also use the utility anywhere you want - e.g. in a React Hook, in a React Component, in a Remix Loader/Action, etc.

### Why?

These components help us load data from Supabase in a declarative way - so we can focus on building our application instead of worrying about how to fetch data from Supabase.

It's type-safe thanks to the types exported by the Supabase JS Client - so you can be sure that the data you're fetching is the data you're expecting.

## The Data Loader SDK

### Installation

The Data Loader SDK is provided as an independent (and open-source) package - `@makerkit/data-loader-supabase-remix`. It is not included in the SaaS Kit by default - so (for the time being) you need to install it separately.

To install it, run the following command:

```bash
npm i @makerkit/data-loader-supabase-remix
```

To import the Data Loader SDK, you can use multiple approaches.

1. **Components** - i.e. to be used in SPA/SSR
2. **React Hook** - i.e. to be used in React Client Components
3. **Directly** using the `fetchDataFromSupabase` function. Useful for server-side code - e.g. in a Route Handler, or Server Action

### React Components

With **React Client Components** - i.e. to be used in SPA/SSR:

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';
```

These are ideal when you want to fetch the data in both the server and the client.

### useSupabaseQuery React Hook

Alternatively, you can use a **React Hook** - which can only be used in React Client Components:

```tsx
import { useSupabaseQuery } from '@makerkit/data-loader-supabase-remix';
import useSupabase from '~/core/supabase/use-supabase';

function OrganizationsTable() {
  const client = useSupabase();

  const { data, isLoading, error } = useSupabaseQuery({
    client,
    table: 'organizations',
    select: '*',
  });

  if (isLoading) {
    return <span>Loading...</span>;
  }

  if (error) {
    return <span>Error: {error.message}</span>;
  }

  return (
    <DataTable
      data={data}
      columns={[
        {
          header: 'ID',
          accessorKey: 'id',
        },
        {
          header: 'Name',
          accessorKey: 'name',
        }
      ]}
    />
  );
}
```

### Directly from the "useSupabaseQuery" function

Underneath the hood, the `ClientDataLoader` component uses the `fetchDataFromSupabase` function - which can be used directly to fetch data from Supabase.

The `fetchDataFromSupabase` function is exported from the `@makerkit/data-loader-supabase-core` package - which is a dependency of the `@makerkit/data-loader-supabase-remix` package.

```tsx
import { ActionFunctionArgs, json } from "@remix-run/node";
import { fetchDataFromSupabase } from '@makerkit/data-loader-supabase-core';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function action(args: ActionFunctionArgs) {
  const client = getSupabaseServerClient(args.request);

  const { data, count, pageSize, pageCount } = await fetchDataFromSupabase({
    client,
    table: 'organizations',
    select: '*',
    where: {
      name: {
        textSearch: `'makerkit'`,
      }
    }
  });

  return json({
    data,
    count,
    pageSize,
    pageCount,
  });
}
```

## Usage

The `ClientDataLoader` uses React Query under the hood and exposes the properties `data`, `loading` and `error` - since the loading state is something you need to worry about as it's handled by the client and the data is not available when the component is rendered.

### API

Both components accept the following properties:

Required:
- `client` - the Supabase Client to use to fetch data. This is required.
- `table` - the table to fetch data from. This is autocompleted thanks to the Typescript types exported by the Supabase JS Client. This is required.

Optional:
- `select` - the columns to fetch listed as an array. You can also use first-level joins - e.g. `['id', 'name', 'organization_id.name']`. The `*` wildcard is also supported - as in `'*'` (not an array). In this case, all the columns will be fetched.
- `where` - the where clause to use to filter the data. This will change based on the table you're fetching data from.
- `sort` - the order clause to use to order the data. This will change based on the table you're fetching data from.
- `page` - the page to use to paginate the data.
- `limit` - the limit clause to use to limit the data - i.e. the page size.
- `single` - whether to return a single object or an array of objects. This is useful when you're fetching a single object - e.g. a user by its id. By default, just like the Supabase JS Client, it returns an array of objects. Unlike the Supabase JS Client, it will not throw an error when 0 or many objects are returned.
- `count` - the count type to use to count the data. You can use either `exact` or `estimated`. By default, it uses `exact`.

### ClientDataLoader

Let's see how to use the `ClientDataLoader` component.

We will build a simple table that lists all the organizations the user can read. It works much the same way as the `ServerDataLoader` component - but it returns the following properties:

1. `result`:
  1. `data` - the data fetched from Supabase. In this case, an array of objects with all the columns of the `organizations` table.
  2. `count` - the total number of rows in the table. In this case, the total number of rows in the `organizations` table.
  3. `pageSize` - the page size used to paginate the data. In this case, the default page size.
  4. `pageCount` - the total number of pages used to paginate the data. In this case, the total number of pages used to paginate the `organizations` table.
2. `isLoading` - whether the data is loading or not.
3. `error` - the error, if any.
4. `onPageChange` - the function to call when the page changes. This is useful when you want to use the `page` property to paginate the data.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';
import useSupabase from '~/core/supabase/use-supabase';

const OrganizationsTable = (props: { page: string }) => {
  const client = useSupabase();
  const page = Number(props.page) || 1;

  return (
    <ClientDataLoader
      client={client}
      table="organizations"
      page={page} // the page to fetch
      select="*" // all the columns - can be omitted
      limit={10} // retrieve 10 organizations per page
    >
      {({ result, isLoading }) => {
        const { data, count, pageSize, pageCount } = result;

        if (isLoading) {
          return <span>Loading...</span>;
        }

        return (
          <DataTable
            data={data}
            count={count}
            pageSize={pageSize}
            pageCount={pageCount}
            columns={[
              {
                header: 'ID',
                accessorKey: 'id',
              },
              {
                header: 'Name',
                accessorKey: 'name',
              }
            ]}
          />
        );
      }}
    </ClientDataLoader>
  );
};
```

### Filters

Let's see how to use the `where` property. We can use nearly all the supported operators by the Supabase JS Client.

Below, we pass `where` to the `ClientDataLoader` component to filter the organizations by their name. We use the `textSearch` operator to search for the word `supabase` in the `name` column.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client} // the Supabase Client
  table="organizations"
  select="*"
  where={{
    name: {
      textSearch: `'supabase'`,
    }
  }}
/>
```

We can also use other operators - e.g. `in` to filter the organizations by their id - i.e. to fetch only the organizations with the ids `1`, `2` and `3`.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client} // the Supabase Client
  table="organizations"
  select="*"
  where={{
    id: {
      in: [1, 2, 3],
    }
  }}
/>
```

Alternatively, we can use the `eq` operator to filter the organizations by their id - i.e. to fetch only the organizations with the id `1`.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client}
  table="organizations"
  select="*"
  where={{
    id: {
      eq: 1,
    }
  }}
/>
```

### Select

Let's see how to use the `select` property.

Below, we pass  `select` to the `ClientDataLoader` component to select only the `id` and `name` columns.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client}
  table="organizations"
  select={['id', 'name']}
/>
```

Let's assume we want to fetch the organization name of the `tasks` table, which has a `organization_id` column that references the `organizations` table. We can use first-level joins to fetch the organization name.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client}
  table="tasks"
  select={['id', 'name', 'organization_id.name']}
/>
```

The data returned will be an array of objects with the following structure:

```json
[
  {
    "id": 1,
    "name": "Task 1",
    "organization_id": {
      "name": "Organization 1"
    }
  },
  {
    "id": 2,
    "name": "Task 2",
    "organization_id": {
      "name": "Organization 2"
    }
  }
]
```

The '*' wildcard is also supported - as in `'*'` (not an array). In this case, all the columns will be fetched.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client}
  table="organizations"
  select="*"
/>
```

You don't have to provide it - as it's the default value. The following is equivalent to the previous example.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  client={client}
  table="organizations"
/>
```

### Single Object

Let's see how to use the `single` property. We can use this property to unwrap the data returned by the `ClientDataLoader` component - since it returns an array of objects by default.

Below, we pass  `single` to the `ClientDataLoader` component to fetch a single organization by its id.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-remix';

<ClientDataLoader
  table="organizations"
  select={["id", "name"]}
  single
  where={{
    id: {
      eq: 1,
    }
  }}
/>
```

The data returned will be an object with the following structure:

```json
{
  "id": 1,
  "name": "Organization 1"
}
```

Bear in mind, a value can be `undefined` when not found - so you need to handle this case.

### Camel Case

Let's see how to use the `camelCase` property. We can use this property to convert the column names to camel case - since normally your Postgres column names are snake case.

Below, we pass `camelCase` to the `ClientDataLoader` component to convert the column names to camel case.

Assuming the `organizations` table has a `organization_name` column, the data returned will be an array of objects with the following structure:

```json
[
  {
    "id": 1,
    "organizationName": "Organization 1"
  },
  {
    "id": 2,
    "organizationName": "Organization 2"
  }
]
```

Check out the following example:

```tsx
import { ServerDataLoader } from '@makerkit/data-loader-supabase-remix';

function Component() {
  return (
    <ClientDataLoader
      client={client}
      table="organizations"
      select={["id", "organization_name"]}
      camelCase
    >
      {({ data }) => {
        return (
          <div>
            <span>{data.id}</span>
            <span>{data.organizationName}</span>
          </div>
        );
      }}
    </ClientDataLoader>
  );
}
```

The TypeScript types are also updated accordingly - so you can be sure that the data you're fetching is the data you're expecting.

-----------------
FILE PATH: kits/remix-supabase/sdk/organization-sdk.mdoc

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

## Fetch the current Organization ID

To fetch the currently selected user's organization ID, use the `getCurrentOrganizationId` method:

```tsx
import type { LoaderFunctionArgs } from "@remix-run/node";
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);

  const organizationUid = await sdk.organization.getCurrentOrganizationId();
  // use the organizationUid
}
```

The UID may be `null` - so do make sure to check for that. For example, you may want to redirect the user to the Organizations page if no organization is selected:

```tsx
import { LoaderFunctionArgs, redirect } from "@remix-run/node";

import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);

  const organizationUid = await sdk.organization.getCurrentOrganizationId();

  if (!organizationUid) {
    // No organization selected
    return redirect('/organizations')
  }
}
```

## Fetch the current Organization

To fetch the currently selected user's organization, use the `getCurrent` method:

```tsx
import { LoaderFunctionArgs } from "@remix-run/node";

import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);

  const organization = await sdk.organization.getCurrent();
  // use the organization
}
```

The `getCurrent` method returns the interface `GetCurrentOrganizationResponse` which is defined as:

```ts
interface GetCurrentOrganizationResponse {
  organization: {
    id: number;
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
FILE PATH: kits/remix-supabase/sdk/organization-subscription-sdk.mdoc

---
status: "published"
title: "Organization Subscription SDK | Remix Supabase Kit"
label: "Organization Subscription SDK"
order: 3
description: "Learn how to use the Organization Subscription SDK to fetch the current organization Subscription details"
---

The Subscription SDK helps with getting details about the current organization Subscription. The Subscription is available in the `organization` namespace of the SDK

## Get the current organization Subscription

To retrieve the current organization Subscription, use the `organization.getSubscription()` method to first get the current organization, and then get the Subscription from the organization.

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();

  // ...
}
```

## Check if the current organization has an active subscription

To check if the current organization has an active subscription, use the `subscription.isActive()` method.

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
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
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
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
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();
  const status = await sdk.organization.status; // Stripe.Subscription.Status

  // ...
}
```

## Check the status of the Subscription is a certain status

To check the status of the current organization Subscription, use the `subscription.isStatus` method. The method accepts a single argument, which is the status to check against. This is the `Stripe.Subscription.Status` enum.

```tsx
import { LoaderFunctionArgs } from '@remix-run/node';
import getSdk from '~/lib/sdk';
import getSupabaseServerClient from '~/core/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const client = getSupabaseServerClient(args.request);
  const sdk = getSdk(client);

  const subscription = await sdk.organization.getSubscription();
  const isActive = await sdk.organization.isStatus('active');

  // ...
}
```


