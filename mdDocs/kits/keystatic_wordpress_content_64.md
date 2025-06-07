-----------------
FILE PATH: kits/react-router-supabase-turbo/content/keystatic.mdoc

---
status: "published"
title: "Using KeyStatic with the React Router Supabase SaaS Starter Kit"
label: "Keystatic"
description: "KeyStatic is a CMS that stores data in a JSON file or on your Github repository. It's the default CMS implementation in Makerkit."
order: 1
---

This implementation is used when the host app's environment variable is set as:

```bash
CMS_CLIENT=keystatic
```

Additionally, the following environment variables may be required:

```bash
VITE_KEYSTATIC_STORAGE_KIND=local # local, cloud, github
VITE_KEYSTATIC_CONTENT_PATH=content # apps/web/content
KEYSTATIC_PATH_PREFIX=apps/web
```

You can also use Keystatic Cloud or GitHub as the storage kind as remote storage.

If `VITE_KEYSTATIC_STORAGE_KIND` is set to `cloud`, the following environment variables are required:

```bash
KEYSTATIC_STORAGE_KIND=cloud
KEYSTATIC_STORAGE_PROJECT=project-id
```

If `KEYSTATIC_STORAGE_KIND` is set to `github`, the following environment variables are required:

```bash
VITE_KEYSTATIC_STORAGE_KIND=github
VITE_KEYSTATIC_STORAGE_REPO=makerkit/react-router-supabase-saas-kit-turbo-demo
KEYSTATIC_GITHUB_TOKEN=github_*****************************************************
KEYSTATIC_PATH_PREFIX=apps/web
VITE_KEYSTATIC_CONTENT_PATH=./content
```

Of course, you need to replace the `VITE_KEYSTATIC_STORAGE_KIND` and `KEYSTATIC_GITHUB_TOKEN` with your own values.

GitHub mode requires the installation of a GitHub app for displaying the admin.

Please refer to the [Keystatic documentation](https://keystatic.com/docs/github-model) for more information.

If your content folder is not at `content`, you can set the `VITE_KEYSTATIC_CONTENT_PATH` environment variable to the correct path. For example, if your content folder is at `data/content`, you can set the `VITE_KEYSTATIC_CONTENT_PATH` environment variable as:

```bash
VITE_KEYSTATIC_CONTENT_PATH=data/content
```

**Note**: your Github token must have permissions to read to the repository (for read-only).


-----------------
FILE PATH: kits/react-router-supabase-turbo/content/wordpress.mdoc

---
status: "published"
title: "Using Wordpress in Makerkit | React Router Supabase Kit"
label: "Wordpress"
description: "Wordpress is a popular CMS that you can use with Makerkit. Learn how to set it up and use it in your project."
order: 2
---

The Wordpress implementation is used when the host app's environment variable is set as:

```bash
CMS_CLIENT=wordpress
```

Additionally, please set the following environment variables:

```bash
WORDPRESS_API_URL=http://localhost:8080
```

For development purposes, we ship a Docker container that runs a Wordpress instance. To start the container, run:

```bash
docker-compose up
```

or

```bash
pnpm run start
```

from this package's root directory.

The credentials for the Wordpress instance are:

```bash
WORDPRESS_DB_HOST=db
WORDPRESS_DB_USER=wordpress
WORDPRESS_DB_PASSWORD=wordpress
WORDPRESS_DB_NAME=wordpress
```

You will be asked to set up the Wordpress instance when you visit `http://localhost:8080` for the first time.

## Note for Wordpress REST API

To make the REST API in your Wordpress instance work, please change the permalink structure to `/%post%/` from the Wordpress admin panel.

## Blog

To include Blog Posts from Wordpress - please create a **post** with category named `blog` and add posts to it.

## Documentation

To include Documentation from Wordpress - please create a **page** with category named `documentation` and add pages to it.

This **involves enabling categories for pages. To do this, add the following code to your theme's `functions.php` file:

```php
function add_categories_to_pages() {
    register_taxonomy_for_object_type('category', 'page');
}
add_action('init', 'add_categories_to_pages');
```

Please refer to `wp-content/themes/twentytwentyfour/functions.php` for an example of a theme that includes this code.

## Language Filtering

To make the language filtering work, please add a tag with the language name to the post. For example, if you have a post in English, add the tag `en` to it, and if you have a post in German, add the tag `de` to it.

-----------------
FILE PATH: kits/react-router-supabase-turbo/content.mdoc

---
status: "published"
title: "Content in React Router Supabase Turbo"
label: "Content"
order: 8
description: "Content in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Content in React Router Supabase Turbo


-----------------
FILE PATH: kits/react-router-supabase-turbo/customization/adding-shadcn-ui-components.mdoc

---
status: "published"

label: "Adding Shadcn UI components"
title: "How to add new Shadcn UI components to your React Router Supabase application"
order: 2
description: "Update your React Router Supabase application with new Shadcn UI components"
---


To install a Shadcn UI component, you can use the following command:

```bash
npx shadcn@latest add <component> -c ./packages/ui
```

For example, to install the `Button` component, you can use the following command:

```bash
npx shadcn@latest add button -c ./packages/ui
```

We pass the `--path` flag to specify the path where the component should be installed. You may need to adjust the path based on your project structure.

### Update the imports

**NB**: you may need to update the imports to the `cn` utility function to use the relative imports because it somehow breaks. Please do that.

### Export the component

To achieve optimal tree-shaking, we export each component individually using the `exports` field in the `package.json` file. This allows you to import the component directly from the package.

Once the component has been created, you can export by adding a new export to the `package.json` file.

We assume that the component is located at `src/shadcn/avatar.tsx`. To export the component, you can append a new export field to the `exports` map inside the `package.json` file:

```json
{
  "exports": {
    "./avatar": "./src/shadcn/avatar.tsx"
  }
}
```

Now you can import it directly from the package:

```tsx
import { Avatar } from '@kit/ui/avatar';
```

**NB**: this is an example, you need to adjust the component name based on the component you are exporting.

-----------------
FILE PATH: kits/react-router-supabase-turbo/customization/fonts.mdoc

---
status: "published"
label: "Updating Fonts"
title: "Update the default fonts of your SaaS"
order: 3
description: "Learn how to update the fonts of your Makerkit application"
---

By default, Makerkit uses two fonts:

1. `font-sans`: using Apple's default font on Apple devices, or `Inter` on others
2. `font-heading`: uses `Urbanist` from Google Fonts

The fonts are defined at `apps/web/styles/shadcn-ui.css`:

```css
@import url('//fonts.googleapis.com/css2?family=Inter:wght@300;400;500;700;800&family=Urbanist:wght@500;700&display=swap&display=swap');

--font-sans: -apple-system, Inter, Helvetica, Arial, sans-serif;
--font-heading: Urbanist, var(--font-sans);
```

Please change them to your preferred fonts.

-----------------
FILE PATH: kits/react-router-supabase-turbo/customization/layout-style.mdoc

---
status: "published"
label: "Layout Style"
title: "Update the default layout style of your SaaS | React Router Supabase SaaS Starter Kit"
order: 4
description: "Learn how to update the default layout style of your Makerkit application"
---

By default, Makerkit uses the `sidebar` layout style for both the user and the team workspaces.

You can change the layout style by setting the `VITE_USER_NAVIGATION_STYLE` and `VITE_TEAM_NAVIGATION_STYLE` environment variables. The default style is `sidebar`.

To set the layout style to `header`, update the environment variables:

```bash
VITE_TEAM_NAVIGATION_STYLE=header
VITE_USER_NAVIGATION_STYLE=header
```

The default layout style is `sidebar`:

{% img src="/assets/images/docs/turbo-sidebar-layout.webp" width="2522" height="1910" /%}

And here is `header` layout:

{% img src="/assets/images/docs/turbo-header-layout.webp" width="3282" height="1918" /%}


-----------------
FILE PATH: kits/react-router-supabase-turbo/customization/logo.mdoc

---
status: "published"

label: "Updating the Logo"
title: "Updating the default application logo in your Makerkir application"
order: 1
description: "Updating the default application logo in your Makerkit application"
---


Of course - you'd like to have your own logo in your application. To do so, please update the component at `apps/web/components/app-logo.tsx`.

You can either use an SVG component, or drop an image. As long as it's rendered within this component, it doesn't matter how you do it.

The logo image will be used across the application, including the auth page, the site header, the site footer, and the app sidebar (when team accounts are disabled).

-----------------
FILE PATH: kits/react-router-supabase-turbo/customization/theme.mdoc

---
status: "published"
label: "Updating the theme"
title: "Updating the Shadcn theme in your Makerkit Application | React Router Supabase SaaS Kit"
order: 0
description: "How to update the theme in your  React Router Supabase Makerkit application"
---

Makerkit uses [Shadcn UI](https://ui.shadcn.com) and it defines the theme according to its guidelines.

You can find the default theme inside the application at `apps/web/styles/global.css`.

If you want to override the default style, either define your own colors or pick a theme from the [Shadcn Themes page](https://ui.shadcn.com/themes), and copy/paste the theme into this file.

-----------------
FILE PATH: kits/react-router-supabase-turbo/customization.mdoc

---
status: "published"
title: "Customization in React Router Supabase Turbo"
label: "Customization"
order: 3
description: "Customization in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Welcome to the customization guide!

One of Makerkit's greatest strengths is its flexibility - you can customize nearly every aspect of your application to match your brand and requirements. Let's explore the key areas you can personalize.

## Core Customization Areas

### Environment Variables
The foundation of customization in Makerkit starts with environment variables. These control everything from your app's name and description to feature flags and authentication methods. Most customizations can be made without touching the source code - just update your environment variables and you're good to go!

### Branding and Design
Make your app uniquely yours with:
- Custom colors and themes using Shadcn UI
- Your own logo and favicons
- Personalized fonts (using any Google Font or system font)
- Light and dark mode customization

### Layout and Navigation
Choose how your app looks and feels:
- Switch between sidebar and header navigation layouts
- Customize navigation items for both user and team workspaces
- Add or remove menu entries based on your needs

### Feature Configuration
Enable or disable various features to match your business needs:
- Authentication methods (password, magic link, OAuth)
- Team accounts and permissions
- Billing capabilities
- Notifications system

### Content Management
Manage your marketing content and documentation using either:
- Keystatic (default) - perfect for Git-based content management
- WordPress - ideal for traditional CMS workflows

## Where to Start?

1. Begin with the [Environment Variables](customization/environment-variables) to set up your basic configuration
2. Move on to [Theme Customization](customization/theme) to match your brand colors
3. Update your [Logo](customization/logo) and [Fonts](/docs/next-supabase-turbo/fonts)
4. Configure your [Layout Style](customization/layout-style)

## A Note on Customization

While Makerkit is highly customizable, we've worked hard to make these customizations manageable and maintainable. Most changes can be made through configuration rather than code changes, making it easier to update to newer versions of Makerkit in the future.

Remember - you don't need to customize everything at once! Start with the essentials (branding, layout) and progressively enhance your application as needed.


-----------------
FILE PATH: kits/react-router-supabase-turbo/data-fetching/actions.mdoc

---
title: "Using Actions in React Router Supabase Turbo"
label: "Actions"
description: "Learn how to use the Actions in React Router Supabase Turbo"
position: 2
---

Actions in Makerkit Supabase Turbo follow React Router's pattern, letting you perform server-side operations directly from your components. They provide a clean way to handle form submissions, data mutations, and other server interactions.

## What Are Actions?

Actions are server-side functions that can be triggered from the client. They handle operations like:
- Form submissions
- Data mutations (create, update, delete)
- Authentication flows
- Server-side business logic

## Basic Structure of an Action

Actions are exported from route files and receive a parameter object with:
- `request`: The incoming request object
- Other context information from React Router

```tsx
export const action = async (args: Route.ActionArgs) => {
  // Parse incoming data
  const json = SomeSchema.parse(await args.request.json());

  // Get Supabase client
  const client = getSupabaseServerClient(args.request);

  // Perform different operations based on the "intent"
  switch (json.intent) {
    case 'some-action':
      return doSomething({ client, data: json.payload });

    default:
      return new Response('Invalid action', { status: 400 });
  }
};
```

## Using Actions with Forms

From your components, you can trigger actions using React Router's `useFetcher`:

```tsx
'use client';

import { useFetcher } from 'react-router';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

function MyForm() {
  const form = useForm({
    resolver: zodResolver(MyFormSchema),
    defaultValues: {
      // Your default values
    },
  });

  const fetcher = useFetcher();
  const pending = fetcher.state === 'submitting';

  return (
    <form
      onSubmit={form.handleSubmit((data) => {
        return fetcher.submit(
          {
            intent: 'my-action-intent',
            payload: data,
          },
          {
            encType: 'application/json',
            method: 'POST',
          },
        );
      })}
    >
      {/* Form fields */}

      <Button type="submit" disabled={pending}>
        {pending ? 'Processing...' : 'Submit'}
      </Button>
    </form>
  );
}
```

## Action Patterns in Makerkit

### The Intent Pattern

Makerkit uses an "intent" pattern to handle multiple actions from a single route:

```tsx
// In your route file
export const action = async (args: Route.ActionArgs) => {
  const json = ActionsSchema.parse(await args.request.json());
  const client = getSupabaseServerClient(args.request);

  switch (json.intent) {
    case 'delete-account':
      return deletePersonalAccountAction({ client, otp: json.payload.otp });

    case 'update-profile':
      return updateProfileAction({ client, data: json.payload });

    default:
      return new Response('Invalid action', { status: 400 });
  }
};
```

### Creating Reusable Actions

For complex operations, create separate action functions:

```tsx
// In a separate file
export const deletePersonalAccountAction = async ({
  client,
  otp,
}: {
  client: SupabaseClient<Database>;
  otp: string;
}) => {
  // Implementation here
};

// In your route file
import { deletePersonalAccountAction } from './actions';

export const action = async (args: Route.ActionArgs) => {
  // ...
  return deletePersonalAccountAction({ client, otp });
};
```

## Data Validation with Zod

Always validate incoming data with Zod:

```tsx
const DeleteAccountFormSchema = z.object({
  otp: z.string().min(1),
});

// In your action
const data = DeleteAccountFormSchema.parse(json.payload);
```

## Error Handling

Handle errors gracefully and return appropriate responses:

```tsx
try {
  // Action implementation
  return redirectDocument('/success');
} catch (error) {
  console.error('Action failed:', error);
  return json(
    { error: 'Something went wrong' },
    { status: 500 }
  );
}
```

## Authentication in Actions

Check authentication status before performing sensitive operations:

```tsx
const auth = await requireUser(client);

if (!auth.data) {
  return redirectDocument(auth.redirectTo);
}

const user = auth.data;
// Continue with authorized action
```

## Example: Complete Account Deletion Flow

Here's a complete example of the account deletion flow:

1. **Route file exports the action:**

```tsx
export const action = async (args: Route.ActionArgs) => {
  const json = ActionsSchema.parse(await args.request.json());
  const client = getSupabaseServerClient(args.request);

  switch (json.intent) {
    case 'delete-account':
      return deletePersonalAccountAction({ client, otp: json.payload.otp });

    default:
      return new Response('Invalid action', { status: 400 });
  }
};
```

2. **Component uses the action:**

```tsx
function DeleteAccountForm(props: { email: string }) {
  const form = useForm({
    resolver: zodResolver(DeleteAccountFormSchema),
    defaultValues: {
      otp: '',
    },
  });

  const fetcher = useFetcher();
  const pending = fetcher.state === 'submitting';

  return (
    <Form {...form}>
      <form
        onSubmit={form.handleSubmit((data) => {
          return fetcher.submit(
            {
              intent: 'delete-account',
              payload: data,
            },
            {
              encType: 'application/json',
              method: 'POST',
            },
          );
        })}
      >
        {/* Form fields */}

        <Button
          type="submit"
          disabled={pending}
          variant="destructive"
        >
          {pending ? 'Deleting Account...' : 'Delete Account'}
        </Button>
      </form>
    </Form>
  );
}
```

3. **Implementation of the action:**

```tsx
export const deletePersonalAccountAction = async ({
  client,
  otp,
}: {
  client: SupabaseClient<Database>;
  otp: string;
}) => {
  const auth = await requireUser(client);

  if (!auth.data) {
    return redirectDocument(auth.redirectTo);
  }

  const user = auth.data;

  // Verify OTP
  const otpApi = createOtpApi(client);
  const result = await otpApi.verifyToken({
    purpose: 'delete-personal-account',
    userId: user.id,
    token: otp,
  });

  if (!result.valid) {
    throw new Error('Invalid OTP');
  }

  // Delete account
  await deleteAccount(user.id);

  // Sign out and redirect
  await client.auth.signOut();
  return redirectDocument('/');
};
```

## Tips for Effective Actions

1. **Keep them focused** - Each action should do one thing well
2. **Validate input** - Always validate input data with Zod
3. **Check permissions** - Verify the user has permission to perform the action
4. **Handle errors** - Return appropriate error responses
5. **Use services** - Move complex logic to service functions for better organization
6. **Provide feedback** - Return status information the UI can use to show feedback

By following these patterns, you'll be able to create robust, type-safe server actions in your Makerkit application.

-----------------
FILE PATH: kits/react-router-supabase-turbo/data-fetching/loaders.mdoc

---
title: "Using Loaders in React Router Supabase Turbo"
label: "Loaders"
description: "Learn how to use the Loaders in React Router Supabase Turbo"
position: 1
---

Loaders in Makerkit Supabase Turbo are a powerful React Router feature that let you fetch data before a route component renders. They provide a clean way to handle data fetching and make your application more responsive.

## What Are Loaders?

Loaders are server-side functions that:
- Fetch data before rendering a component
- Run on every navigation to their route
- Pass their data to components via `loaderData` props
- Enable type-safe data handling with TypeScript

## Basic Structure of a Loader

A basic loader exports a function that receives context from React Router:

```tsx
export const loader = async (args: Route.LoaderArgs) => {
  // Get Supabase client
  const client = getSupabaseServerClient(args.request);

  // Retrieve data
  const data = await fetchSomeData(client);

  // Return data to the component
  return {
    title: "My Page",
    data
  };
};
```

## Using Loader Data in Components

Components automatically receive loader data through props:

```tsx
export default function MyPage(props: Route.ComponentProps) {
  const { title, data } = props.loaderData;

  return (
    <div>
      <h1>{title}</h1>
      <DataDisplay data={data} />
    </div>
  );
}
```

## Type Safety with Route Types

Makerkit provides route types to ensure type safety throughout your loader chain:

```tsx
import type { Route } from '~/types/app/routes/home/account/+types';

export const loader = async (args: Route.LoaderArgs) => {
  // Typed arguments
  return {
    title: "My Page"
  };
};

export default function MyPage(props: Route.ComponentProps) {
  // props.loaderData is typed based on loader return type
  const { title } = props.loaderData;
  return <h1>{title}</h1>;
}
```

## Authentication in Loaders

Require authentication before processing a loader:

```tsx
export const loader = async (args: Route.LoaderArgs) => {
  // Require authenticated user
  await requireUserLoader(args.request);

  // Continue with loader logic
  return {
    title: "Protected Page"
  };
};
```

## Handling Errors and Redirects

Loaders can throw responses that React Router will catch:

```tsx
export const loader = async (args: Route.LoaderArgs) => {
  try {
    const data = await fetchData();
    return { data };
  } catch (error) {
    // Redirect to error page
    throw redirect('/error');
  }
};
```

## Common Loader Patterns in Makerkit

### Loading Team Account Workspace

```tsx
export const loader = async (args: Route.LoaderArgs) => {
  const accountSlug = args.params.account as string;
  const client = getSupabaseServerClient(args.request);

  const workspace = await loadTeamWorkspace({
    accountSlug,
    client,
  });

  return {
    workspace,
    accountSlug,
  };
};
```

### Shared Loaders for Reuse

Create reusable loader functions:

```tsx
// In a shared loader file
export async function loadMembersPageData(
  client: SupabaseClient<Database>,
  slug: string,
) {
  return Promise.all([
    loadAccountMembers(client, slug),
    loadInvitations(client, slug),
    loadUser(client),
    canAddMember(),
  ]);
}

// In your route file
export const loader = async (args: Route.LoaderArgs) => {
  const client = getSupabaseServerClient(args.request);
  const slug = args.params.account as string;

  const [members, invitations, user, canAdd] =
    await loadMembersPageData(client, slug);

  return {
    members,
    invitations,
    user,
    canAdd
  };
};
```

### Conditional Data Loading

Load different data based on configuration:

```tsx
const BILLING_MODE = z
  .enum(['subscription', 'one-time'])
  .default('subscription')
  .parse(process.env.BILLING_MODE);

export const loader = async (args: Route.LoaderArgs) => {
  const client = getSupabaseServerClient(args.request);
  const accountId = args.params.id;

  const api = createAccountsApi(client);

  const data = BILLING_MODE === 'subscription'
    ? await api.getSubscription(accountId)
    : await api.getOrder(accountId);

  return { data };
};
```

## Using Internationalization in Loaders

For internationalized apps, add translations to your loaders:

```tsx
export const loader = async (args: Route.LoaderArgs) => {
  const i18n = await createI18nServerInstance(args.request);

  return {
    title: i18n.t('account:settingsTab')
  };
};
```

## Type Inference in React Router

React Router 7 infers types from your loader to your components. Here's how it works:

1. **Route Types Definition**

Makerkit defines route types for each route in your application:

```tsx
// In ~/types/app/routes/home/account/+types.ts
export namespace Route {
  export type LoaderArgs = {
    request: Request;
    params: {
      account: string;
    };
  };

  export type LoaderData = Awaited<ReturnType<typeof loader>>;

  export type ComponentProps = {
    loaderData: LoaderData;
  };

  export type MetaArgs = {
    data: LoaderData | undefined;
  };
}
```

2. **Loader Type Inference**

The loader return type is automatically inferred:

```tsx
export const loader = async (args: Route.LoaderArgs) => {
  return {
    title: "My Page",
    data: [1, 2, 3]
  };
};
```

3. **Component Props Typing**

The component props are typed based on the loader return:

```tsx
export default function MyPage(props: Route.ComponentProps) {
  // TypeScript knows props.loaderData has:
  // - title: string
  // - data: number[]
  const { title, data } = props.loaderData;

  return (
    <div>
      <h1>{title}</h1>
      <ul>
        {data.map(item => <li key={item}>{item}</li>)}
      </ul>
    </div>
  );
}
```

## Best Practices for Loaders

1. **Keep loaders focused**
- Load only what the component needs
- Use separate loaders for different data concerns

2. **Handle errors gracefully**
- Use try/catch to handle errors
- Provide fallback UIs or redirects for error states

3. **Consider performance**
- Use parallel data fetching with `Promise.all` when appropriate
- Consider caching strategies for expensive data

4. **Type everything**
- Use TypeScript to ensure type safety
- Take advantage of Makerkit's type system

5. **Extract reusable loaders**
- Move common loading patterns to shared functions
- Keep route files clean and focused

## Example: Complete Page with Loader

Here's a complete example of a page with a loader:

```tsx
import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { PageBody } from '@kit/ui/page';
import { Trans } from '@kit/ui/trans';

import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { requireUserLoader } from '~/lib/require-user-loader';
import type { Route } from '~/types/app/routes/home/account/+types';

import { TeamAccountLayoutPageHeader } from './_components/team-account-layout-page-header';
import { loadTeamData } from './_lib/team-data-loader.server';

export const loader = async (args: Route.LoaderArgs) => {
  const i18n = await createI18nServerInstance(args.request);

  // Require user authentication
  await requireUserLoader(args.request);

  const client = getSupabaseServerClient(args.request);
  const account = args.params.account as string;

  // Load team data
  const teamData = await loadTeamData(client, account);

  return {
    title: i18n.t('teams:home.pageTitle'),
    account,
    teamData
  };
};

export const meta = ({ data }: Route.MetaArgs) => {
  return [
    {
      title: data?.title,
    },
  ];
};

export default function TeamHomePage(props: Route.ComponentProps) {
  const { account, teamData } = props.loaderData;

  return (
    <>
      <TeamAccountLayoutPageHeader
        account={account}
        title={<Trans i18nKey={'common:teamDashboard'} />}
      />

      <PageBody>
        <TeamDataDisplay data={teamData} />
      </PageBody>
    </>
  );
}
```

By following these patterns, you'll be able to build robust, type-safe data fetching in your Makerkit application with React Router's powerful loader system.

-----------------
FILE PATH: kits/react-router-supabase-turbo/data-fetching/supabase-clients.mdoc

---
title: "Using the Supabase Clients in React Router Supabase Turbo"
label: "Supabase Clients"
description: "Learn how to use the Supabase Clients in React Router Supabase Turbo"
position: 0
---

Before diving into the various ways we can communicate with the server, we need to introduce how we communicate with Supabase, which is hosting the database and therefore the source of our data.

Depending on where your code runs, you may need to use different clients to interact with Supabase. This is due to how cookies are set differently in various parts of the React Router application.

You can use 3 different clients to interact with Supabase:

1. **Browser** - This runs in the browser and is used to interact with Supabase from the client
2. **Server** - This runs in the server and is used to interact with Supabase from the server
3. **Server Admin** - This runs in the server and is used to interact with Supabase from the server, but with admin privileges

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

## Server Client

To use the server actions client, use the `useSupabase` hook:

```tsx
import { LoaderFunctionArgs } from 'react-router';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function loader(args: LoaderFunctionArgs) {
  const supabase = getSupabaseServerClient()

  const { data, error } = await supabase.from('tasks').select('*');

  return {
    data,
  };
}
```

## Server Admin Client

To use the Server Admin client, use the `getSupabaseServerAdminClient` function:

```tsx
import { LoaderFunctionArgs } from 'react-router';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

export async function loader(args: LoaderFunctionArgs) {
  const supabase = getSupabaseServerAdminClient()

  const { data, error } = await supabase.from('tasks').select('*');

  return {
    data,
  };
}
```

NB: only use this client when you need to perform admin operations, such as bypassing row-level security. Do not use this client for regular operations.

-----------------
FILE PATH: kits/react-router-supabase-turbo/data-fetching.mdoc

---
status: "published"
title: "Data Fetching in React Router Supabase Turbo"
label: "Data Fetching"
order: 5
description: "Data Fetching in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Here's a draft of the Data Fetching pillar page for Makerkit:

# Data Fetching in Makerkit

Makerkit provides a robust approach to fetching and managing data in your React Router Supabase application. Whether you're building server components, handling real-time updates, or managing client-side state, Makerkit offers patterns and utilities to make data fetching clean and efficient.

## Data Loading Patterns

Common patterns for loading data include:

- Server Component data fetching for initial page loads
- React Query for client-side data management
- Server Actions for mutations
- CSRF and Captcha protection for API Routes


-----------------
FILE PATH: kits/react-router-supabase-turbo/development/adding-pages.mdoc

---
status: "published"
label: "Adding Pages"
title: "Adding Pages in the React Router Supabase Turbo Starter Kit"
description: "Learn how to create new pages in the React Router Supabase Turbo Starter Kit."
order: 8
---

Adding new pages to your React Router Supabase Turbo Starter Kit application involves understanding the routing structure and following some conventions. This guide will walk you through creating new pages in both the marketing section and authenticated application areas.

## Understanding the Router Structure

The starter kit uses React Router and organizes routes in a file-based structure. Looking at the `apps/web/app/routes.ts` file, you can see how routes are grouped:

- `rootRoutes`: Static utility routes like robots.txt, sitemap, etc.
- `apiRoutes`: Backend API endpoints
- `marketingLayout`: Public-facing pages (homepage, blog, etc.)
- `authLayout`: Authentication-related pages
- `adminLayout`: Admin portal pages
- `userAccountLayout`: User personal account pages
- `teamAccountLayout`: Team account pages

## Route Configuration Format

Routes are defined using these helper functions:

```typescript
route('path', 'component-path')
layout('layout-component', [child-routes])
index('index-component')
```

## Adding a New Page to the Marketing Section

Let's create a new "Features" page in the marketing section:

1. **Create a new component file**:

```tsx
// apps/web/app/routes/marketing/features.tsx
import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { SitePageHeader } from '~/routes/marketing/_components/site-page-header';
import type { Route } from '~/types/app/routes/marketing/+types/features';

export const meta = ({ data }: Route.MetaArgs) => {
  return [
    {
      title: data?.title,
    },
  ];
};

export const loader = async function ({ request }: Route.LoaderArgs) {
  const { t } = await createI18nServerInstance(request);

  return {
    title: t('marketing:features'),
    subtitle: t('marketing:featuresDescription'),
  };
};

export default function FeaturesPage(props: Route.ComponentProps) {
  const data = props.loaderData;

  return (
    <div>
      <SitePageHeader title={data.title} subtitle={data.subtitle} />

      <div className={'container mx-auto py-8'}>
        <div className="grid grid-cols-1 gap-8 md:grid-cols-2 lg:grid-cols-3">
          {/* Feature cards */}
          <FeatureCard
            title="Multi-tenant Support"
            description="Support for multiple organizations and team accounts"
          />
          <FeatureCard
            title="Authentication"
            description="Various authentication methods including email, password, and OAuth"
          />
          <FeatureCard
            title="Billing Integration"
            description="Seamless integration with popular payment providers"
          />
        </div>
      </div>
    </div>
  );
}

function FeatureCard({ title, description }: { title: string; description: string }) {
  return (
    <div className="rounded-lg border p-6">
      <h3 className="text-xl font-semibold">{title}</h3>
      <p className="text-muted-foreground mt-2">{description}</p>
    </div>
  );
}
```

2. **Update the routes configuration** in `apps/web/app/routes.ts`:

```typescript
const marketingLayout = layout('routes/marketing/layout.tsx', [
  index('routes/marketing/index.tsx'),
  route('terms-of-service', 'routes/marketing/terms-of-service.tsx'),
  route('privacy-policy', 'routes/marketing/privacy-policy.tsx'),
  route('pricing', 'routes/marketing/pricing.tsx'),
  route('contact', 'routes/marketing/contact/index.tsx'),
  route('features', 'routes/marketing/features.tsx'), // Add your new route here
  route('faq', 'routes/marketing/faq.tsx'),
  // ...other routes
]);
```

3. **Update navigation** in `apps/web/app/routes/marketing/_components/site-navigation.tsx`:

```typescript
const links = {
  Blog: {
    label: 'marketing:blog',
    path: '/blog',
  },
  Features: {    // Add this new item
    label: 'marketing:features',
    path: '/features',
  },
  Docs: {
    label: 'marketing:documentation',
    path: '/docs',
  },
  // ...other links
};
```

4. **Add translations** (optional) by updating your i18n files with the new labels.

## Adding a Page to the Authenticated App Area

Let's add a "Reports" page for authenticated users:

1. **Create the component file**:

```tsx
// apps/web/app/routes/home/user/reports.tsx
import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';
import { PageBody } from '@kit/ui/page';
import { Trans } from '@kit/ui/trans';

import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { requireUserLoader } from '~/lib/require-user-loader';
import type { Route } from '~/types/app/routes/home/user/+types/reports';

import { HomeLayoutPageHeader } from './_components/home-page-header';

export const meta = ({ data }: Route.MetaArgs) => {
  return [
    {
      title: data?.title,
    },
  ];
};

export const loader = async (args: Route.LoaderArgs) => {
  // require user
  await requireUserLoader(args.request);

  const i18n = await createI18nServerInstance(args.request);
  const title = i18n.t('account:reportsTab');

  return {
    title,
  };
};

export default function UserReportsPage(props: Route.ComponentProps) {
  return (
    <>
      <HomeLayoutPageHeader
        title={<Trans i18nKey={'account:reportsTabLabel'} />}
        description={<AppBreadcrumbs />}
      />

      <PageBody>
        <div className="flex w-full flex-col space-y-6">
          <h1 className="text-2xl font-semibold">Reports</h1>
          <p className="text-muted-foreground">
            View usage statistics and reports for your account.
          </p>

          {/* Add your reports components here */}
          <div className="grid grid-cols-1 gap-4 lg:grid-cols-2">
            <ReportCard title="Usage Statistics" />
            <ReportCard title="Activity Log" />
          </div>
        </div>
      </PageBody>
    </>
  );
}

function ReportCard({ title }: { title: string }) {
  return (
    <div className="rounded-md border p-4">
      <h2 className="text-lg font-medium">{title}</h2>
      <div className="h-32 w-full bg-gray-100 dark:bg-gray-800 mt-2 flex items-center justify-center">
        <span className="text-muted-foreground">Report data will appear here</span>
      </div>
    </div>
  );
}
```

2. **Update the routes configuration** in `apps/web/app/routes.ts`:

```typescript
const userAccountLayout = layout('routes/home/user/layout.tsx', [
  route('home', 'routes/home/user/index.tsx'),
  route('home/settings', 'routes/home/user/settings.tsx'),
  route('home/billing', 'routes/home/user/billing.tsx'),
  route('home/billing/return', 'routes/home/user/billing-return.tsx'),
  route('home/reports', 'routes/home/user/reports.tsx'), // Add your new route here
]);
```

3. **Update navigation configuration** in `~/config/personal-account-navigation.config.ts`:

```typescript
// Add this to the routes array
{
  label: 'account:reportsTabLabel',
  Icon: <BarChart className={'h-4'} />,
  path: '/home/reports',
  end: true,
},
```

## Creating Nested Routes

For more complex scenarios, you might need nested routes with their own layout:

1. **Create a layout component**:

```tsx
// apps/web/app/routes/home/user/reports/layout.tsx
import { Outlet } from 'react-router';
import { PageHeader } from '@kit/ui/page';
import { Tabs, TabsList, TabsTrigger } from '@kit/ui/tabs';

export default function ReportsLayout() {
  return (
    <div className="flex flex-col space-y-4">
      <PageHeader>
        <Tabs defaultValue="overview">
          <TabsList>
            <TabsTrigger value="overview" href="/home/reports">Overview</TabsTrigger>
            <TabsTrigger value="activity" href="/home/reports/activity">Activity</TabsTrigger>
            <TabsTrigger value="usage" href="/home/reports/usage">Usage</TabsTrigger>
          </TabsList>
        </Tabs>
      </PageHeader>

      <Outlet />
    </div>
  );
}
```

2. **Create child route components**:

```tsx
// apps/web/app/routes/home/user/reports/activity.tsx
export default function ActivityReport() {
  return <div>Activity report content here</div>;
}
```

3. **Update the routes configuration** with a nested structure:

```typescript
const reportsLayout = layout('routes/home/user/reports/layout.tsx', [
  index('routes/home/user/reports/index.tsx'),
  route('activity', 'routes/home/user/reports/activity.tsx'),
  route('usage', 'routes/home/user/reports/usage.tsx'),
]);

const userAccountLayout = layout('routes/home/user/layout.tsx', [
  // ... existing routes
  reportsLayout, // Add the nested layout
]);
```

## Best Practices

1. **Route Organization**:
- Group related routes under shared layouts
- Use descriptive naming for route paths and components
2. **UI Consistency**:
- Reuse existing components like `PageHeader`, `PageBody`, etc.
- Follow the established UI patterns for each section
3. **Authentication**:
- Use `requireUserLoader` for authenticated routes
- Consider permissions for team/admin routes
4. **i18n Support**:
- Always use translation keys for user-visible text
- Structure translation keys logically (e.g., `marketing:features`)
5. **Type Safety**:
- Create proper type definitions for loaders and components
- Use the existing Route types pattern

By following these conventions, you can seamlessly add new pages to the starter kit while maintaining consistent structure and behavior.

