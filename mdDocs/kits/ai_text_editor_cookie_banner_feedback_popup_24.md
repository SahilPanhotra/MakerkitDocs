-----------------
FILE PATH: kits/next-supabase/plugins/ai-text-editor.mdoc

---
status: "published"

title: "AI Text Editor Plugin for the Next.js Supabase Saas Starter Kit"
label: "AI Text Editor"
order: 3
description: "This plugin adds an AI WYSIWYG Editor to your Next.js Supabase SaaS application."
---


This plugin adds an AI Editor component Next.js application. You can
import this component anywhere in your application.

It is built with Lexical, a text editor framework for React, by Meta.

**This plugin is currently experimental and not yet officially released.**

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/text-editor git@github.com:makerkit/next-supabase-saas-kit-plugins.git text-editor --squash
```

After running this command, you will have the plugin in your repository at
`plugins/text-editor`. Once pulled, you can apply any customization you need.

#### Using the CLI

If you're using the CLI, you can run the following command to install the plugin:

```
npx @makerkit/cli@latest plugins install
```

Follow the instructions to install the plugin.

#### Add the plugin as a workspace in your package.json

You can do so by adding the following to your `package.json` file:

```json
{
  "workspaces": [
    "plugins/text-editor"
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

### Installing dependencies

To install the dependencies, you can run the following command:

```
npm i
```

NPM will install the dependencies in the `plugins/text-editor` folder as an NPM
workspace.

### Add the required API handlers

You will need to add the following API handlers to your application:
- `api/ai/autocomplete/route.ts`
- `api/ai/edit/route.ts`

#### Edit Route

To add the edit route, create a file at `src/app/api/ai/edit/route.ts` with the
following content:

```ts
import { createAIEditRouteHandler } from '~/plugins/text-editor/lib/route-handler';

export const runtime = `edge`;

export const POST = createAIEditRouteHandler;
```

#### Autocomplete Route

To add the autocomplete route, create a file at `src/app/api/ai/autocomplete/route.ts` with the
following content:

```ts
import { createAIAutocompleteRouteHandler } from '~/plugins/text-editor/lib/route-handler';

export const runtime = `edge`;

export const POST = createAIAutocompleteRouteHandler;
```

### Style

Import the CSS file at `plugins/text-editor/editor.css` into your global CSS
file at `src/app/styles/globals.css`:

```css
@import "../../plugins/text-editor/editor.css";
```

### Using the plugin

To use the plugin, you can import the component from the plugin folder.

NB: the example below uses `rxjs`, which is a dependency not included in the
kit. I do recommend using it, but you can use any other library to handle
debouncing and other logic that you may need if you want to use autosaving
for your application.

```tsx
'use client';

import { useCallback, useEffect, useMemo, useRef } from 'react';
import { toast } from 'sonner';

import {
  debounceTime,
  distinctUntilChanged,
  Subject,
  switchMap,
  tap,
} from 'rxjs';

import Editor from '~/plugins/text-editor/components/Editor';

export default function EditorContainer() {
  const subject$ = useMemo(() => new Subject(), []);
  const currentToastId = useRef<string | number>();

  const onChange = useCallback(
    (content: string) => {
      subject$.next(content);
    },
    [subject$],
  );

  useEffect(() => {
    const subscription = subject$
      .pipe(
        debounceTime(2000),
        distinctUntilChanged(),
        switchMap(() => {
          if (currentToastId.current) {
            toast.dismiss(currentToastId.current);
          }

          currentToastId.current = toast.loading('Saving...', {
            id: currentToastId.current,
          });

          return new Promise((resolve) => {
            setTimeout(resolve, 2000);
          });
        }),
        tap(() => {
          toast.success('Saved!', {
            id: currentToastId.current,
          });
        }),
      )
      .subscribe(() => {
        currentToastId.current = undefined;
      });

    return () => {
      subscription.unsubscribe();
    };
  }, [subject$]);

  return (
    <>
      <Editor
        className={'h-[80vh] w-[100vh]'}
        content={getInitialContent()}
        onChange={onChange}
      />
    </>
  );
}

function getInitialContent() {
  return `
## Introducing Makerkit's AI Editor

This plugin is powered by OpenAI's GPT-3 and Lexical's Editor, and helps you add an AI-powered editor to your SaaS, in just a few lines of code.

Install it using the CLI:

\`\`\`
npx @makerkit/cli plugins install
\`\`\`

Available for *Pro* and *Teams* customers **for free**.
  `.trim();
}
```

Now, import the component `EditorContainer` anywhere in your application:

```tsx
import { Toaster } from 'sonner';
import dynamic from 'next/dynamic';

import PageLoadingIndicator from '~/core/ui/PageLoadingIndicator';
import { withI18n } from '~/i18n/with-i18n';
import I18nProvider from '~/i18n/I18nProvider';
import getLanguageCookie from '~/i18n/get-language-cookie';

const EditorContainer = dynamic(
  () => {
    return import('~/app/editor/EditorContainer');
  },
  {
    ssr: false,
    loading: () => (
      <PageLoadingIndicator fullPage={false}>Loading...</PageLoadingIndicator>
    ),
  },
);

function EditorPage() {
  return (
    <I18nProvider lang={getLanguageCookie()}>
      <div
        className={
          'w-screen h-screen flex justify-center items-center bg-gray-50' +
          ' dark:bg-dark-900 flex flex-col space-y-4'
        }
      >
        <Toaster />

        <EditorContainer />
      </div>
    </I18nProvider>
  );
}

export default withI18n(EditorPage);
```

-----------------
FILE PATH: kits/next-supabase/plugins/cookie-banner.mdoc

---
status: "published"

title: "Cookie Banner Plugin for the Next.js Supabase Saas Starter Kit"
label: "Cookie Banner"
order: 0
description: "This plugin adds a cookie banner to your Next.js Supabase application. It's a simple component that will store the user's consent in a cookie. This banner will stop appearing once the user has given their consent - or rejected it."
---


This plugin adds a cookie banner to your Next.js application. It's a simple
component that will store the user's consent in a cookie. This banner will
stop appearing once the user has given their consent - or rejected it.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/cookie-banner git@github.com:makerkit/next-supabase-saas-kit-plugins.git cookie-banner --squash
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
git subtree add --prefix plugins/cookie-banner https://github.com/makerkit/next-supabase-saas-kit-plugins cookie-banner --squash
```

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

```
git subtree pull --prefix plugins/cookie-banner git@github.com:makerkit/next-supabase-saas-kit-plugins.git cookie-banner --squash
```


-----------------
FILE PATH: kits/next-supabase/plugins/feedback-popup.mdoc

---
status: "published"

title: "Feedback Popup Plugin for the Next.js Supabase Saas Starter Kit"
label: "Feedback Popup"
order: 1
description: "This plugin adds a feedback popup to your Next.js Supabase application. You can import this component anywhere in your application. Also - this plugin provides some admin components to manage and view the feedback submitted by your users."
---


This plugin adds a feedback popup to your Next.js application. You can
import this component anywhere in your application.

Also - this plugin provides some admin components to manage and view the
feedback submitted by your users.

## Using the Plugin

### Installation

To install the plugin, you can use git subtrees from your original repository:

```
git subtree add --prefix plugins/feedback-popup git@github.com:makerkit/next-supabase-saas-kit-plugins.git feedback-popup --squash
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
git subtree add --prefix plugins/feedback-popup https://github.com/makerkit/next-supabase-saas-kit-plugins feedback-popup --squash
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

### Update the Next.js configuration

To support the package `@xenova/transformers.js`, we need to add an
additional property to the Next.js configuration.

Extend the Next.js configuration at `next.config.js` with the following object:

```js
experimental: {
    serverComponentsExternalPackages: ['sharp', 'onnxruntime-node'],
}
```

If the server is running, it may need to be restarted for the changes to take effect.

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

Create the following file at `app/admin/feedback/page.tsx`:

```tsx
import FeedbackSubmissionsPage, {
    metadata,
} from '~/plugins/feedback-popup/admin/FeedbackSubmissionsPage';

export default FeedbackSubmissionsPage;

export { metadata };
```

And the following file at `app/admin/feedback/[id]/page.tsx`:

```tsx
import FeedbackSubmissionPage, {
  metadata,
} from '~/plugins/feedback-popup/admin/FeedbackSubmissionPage';

export default FeedbackSubmissionPage;

export { metadata };
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
FILE PATH: kits/next-supabase/plugins.mdoc

---
status: "published"
title: "Plugins in Next.js Supabase"
label: "Plugins"
order: 14
description: "Plugins in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/sdk/data-loader.mdoc

---
status: "published"
title: "Data Loader SDK - Fetching data from Next.js Supabase directly from declarative React Components"
label: "Data Loader SDK"
order: 5
description: "Fetching data from Next.js Supabase directly from declarative React Components using the Data Loader SDK - a lightweight wrapper around the Supabase JS Client."
---

The Data Loader SDK is a lightweight open-source wrapper around the Supabase JS Client. [The source code can be found here](https://github.com/makerkit/makerkit).

The Data Loader SDK provides a declarative way to fetch data from Supabase directly from React components - from both React Client and Server components (RSC). This means they play well both using RSC and SSR/SPA - so you can choose the approach that best fits your needs.

You can also use the utility anywhere you want - e.g. in a React Hook, in a React Component, in a Next.js API Route, a Server Action, etc.

### Why?

These components help us load data from Supabase in a declarative way - so we can focus on building our application instead of worrying about how to fetch data from Supabase.

It's type-safe thanks to the types exported by the Supabase JS Client - so you can be sure that the data you're fetching is the data you're expecting.

## The Data Loader SDK

### Installation

The Data Loader SDK is provided as an independent (and open-source) package - `@makerkit/data-loader-supabase-nextjs`. It is not included in the SaaS Kit by default - so (for the time being) you need to install it separately.

To install it, run the following command:

```bash
npm i @makerkit/data-loader-supabase-nextjs
```

To import the Data Loader SDK, you can use multiple approaches.

1. **Server Components** - i.e. to be used in RSCs
2. **Client Components** - i.e. to be used in SPA/SSR
3. **React Hook** - i.e. to be used in React Client Components
4. **Directly** using the `fetchDataFromSupabase` function. Useful for server-side code - e.g. in a Route Handler, or Server Action

### Server Components

With **React Server Components** - i.e. to be used in RSCs:

```tsx
import { ServerDataLoader } from '@makerkit/data-loader-supabase-nextjs';
```

These stream compiled JSX directly to the browser and are ideal when you want to fetch the data only on the server and never on the client.

### Client Components

With **React Client Components** - i.e. to be used in SPA/SSR:

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';
```

These are ideal when you want to fetch the data in both the server and the client.

### useSupabaseQuery React Hook

Alternatively, you can use a **React Hook** - which can only be used in React Client Components:

```tsx
import { useSupabaseQuery } from '@makerkit/data-loader-supabase-nextjs';
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

Underneath the hood, the `ServerDataLoader` and `ClientDataLoader` components use the `fetchDataFromSupabase` function - which can be used directly to fetch data from Supabase.

The `fetchDataFromSupabase` function is exported from the `@makerkit/data-loader-supabase-core` package - which is a dependency of the `@makerkit/data-loader-supabase-nextjs` package.

You can use this anywhere you want - e.g. in a React Hook, in a React Component, in a Next.js API Route, etc. This is why you will need to pass the appropriate Supabase Client to the function.

```tsx
import { NextResponse } from "next/server";
import { fetchDataFromSupabase } from '@makerkit/data-loader-supabase-core';
import getSupabaseRouteHandlerClient from '~/core/supabase/route-handler-client';

export async function GET() {
  const client = getSupabaseRouteHandlerClient();

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

  return NextResponse.json({
    data,
    count,
    pageSize,
    pageCount,
  });
}
```

If you were to use it in a Server Action, you would need to pass the appropriate Supabase Client to the function.

```tsx
'use server';

import { fetchDataFromSupabase } from '@makerkit/data-loader-supabase-core';
import getSupabaseServerActionClient from '~/core/supabase/server-action-client';

export async function serverAction() {
  const client = getSupabaseServerActionClient();

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

  return {
    data,
    count,
    pageSize,
    pageCount,
  };
}
```

## Usage

The Data Loader SDK provides two components:
- `ServerDataLoader` - for React Server Components
- `ClientDataLoader` - for React Client Components

The two works exactly the same way - but you need to choose the one that best fits your needs. The only difference is in what they return.

1. `ServerDataLoader` returns directly the data fetched from Supabase - since the loading state is not something you need to worry about as it's handled by the server and the data is already available when the component is rendered.
2. `ClientDataLoader` uses SWR under the hood and exposes the properties `data`, `loading` and `error` - since the loading state is something you need to worry about as it's handled by the client and the data is not available when the component is rendered.

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

### ServerDataLoader

Let's see how to use the `ServerDataLoader` component.

We will build a simple table that lists all the organizations the user can read.

```tsx
import { ServerDataLoader } from '@makerkit/data-loader-supabase-nextjs';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';
import DataTable from '~/core/ui/DataTable';

const OrganizationsTable = () => {
  const client = getSupabaseServerComponentClient();

  return (
    <ServerDataLoader
      client={client}
      table="organizations"
      select={["id", "name"]}
    >
      {({ data, count, pageSize, pageCount }) => {
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
    </ServerDataLoader>
  );
};
```

We used the following properties:
1. `table` - the table to fetch data from. In this case, `organizations`.
2. `select` - the columns to fetch. In this case, `["id", "name"]` - i.e. the `id` and `name` columns of the `organizations` table.

The `ServerDataLoader` component returns the following properties:

1. `data` - the data fetched from Supabase. In this case, an array of objects with all the columns of the `organizations` table.
2. `count` - the total number of rows in the table. In this case, the total number of rows in the `organizations` table.
3. `pageSize` - the page size used to paginate the data. In this case, the default page size.
4. `pageCount` - the total number of pages used to paginate the data. In this case, the total number of pages used to paginate the `organizations` table.

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
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';
import useSupabase from '~/core/supabase/use-supabase';

interface SearchParams {
  page: string;
}

const OrganizationsTable = ({ searchParams }: { searchParams: SearchParams }) => {
  const client = useSupabase();
  const page = Number(searchParams.page) || 1;

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
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

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
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

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
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

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
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

<ClientDataLoader
  client={client}
  table="organizations"
  select={['id', 'name']}
/>
```

Let's assume we want to fetch the organization name of the `tasks` table, which has a `organization_id` column that references the `organizations` table. We can use first-level joins to fetch the organization name.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

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
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

<ClientDataLoader
  client={client}
  table="organizations"
  select="*"
/>
```

You don't have to provide it - as it's the default value. The following is equivalent to the previous example.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

<ClientDataLoader
  client={client}
  table="organizations"
/>
```

### Single Object

Let's see how to use the `single` property. We can use this property to unwrap the data returned by the `ServerDataLoader` component - since it returns an array of objects by default.

Below, we pass  `single` to the `ClientDataLoader` component to fetch a single organization by its id.

```tsx
import { ClientDataLoader } from '@makerkit/data-loader-supabase-nextjs';

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

Below, we pass `camelCase` to the `ServerDataLoader` component to convert the column names to camel case.

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
import { ServerDataLoader } from '@makerkit/data-loader-supabase-nextjs';

function Component() {
  return (
    <ServerDataLoader
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
    </ServerDataLoader>
  );
}
```

The TypeScript types are also updated accordingly - so you can be sure that the data you're fetching is the data you're expecting.

