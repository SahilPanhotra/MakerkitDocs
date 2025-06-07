-----------------
FILE PATH: kits\next-supabase-turbo\recipes\team-accounts-only.mdoc

---
status: "published"
title: 'Disabling Personal Accounts in the Next.js Supabase Turbo SaaS Kit and Only Allowing Team Accounts'
label: 'Disabling Personal Accounts'
order: 1
description: 'Learn how to disable personal accounts in the Next.js Supabase Turbo SaaS kit and only allow team accounts'
---

The Next.js Supabase Turbo SaaS kit is designed to allow personal accounts by default. However, you can disable the personal account view, and only allow user to access team accounts.

Let's walk through the steps to disable personal accounts in the Next.js Supabase Turbo SaaS kit:

1. **Store team slug in cookies**: When a user logs in, store the team slug in a cookie. If none is provided, redirect the user to the team selection page.
2. **Set up Redirect**: Redirect customers to the latest selected team account
3. **Create a Team Selection Page**: Create a page where users can select the team they want to log in to.
4. **Duplicate the user settings page**: Duplicate the user settings page so they can access their own settings from within the team workspace

## Storing the User Cookie and Redirecting to the Team Selection Page

To make sure that users are always redirected to the team selection page, you need to store the team slug in a cookie. If the team slug is not found, redirect the user to the team selection page. We will do all of this in the middleware.

First, let's create these functions in the `apps/web/middleware.ts` file:

 ```tsx {% title="apps/web/middleware.ts" %}
const createTeamCookie = (userId: string) => `${userId}-selected-team-slug`;

function handleTeamAccountsOnly(request: NextRequest, userId: string) {
  // always allow access to the teams page
  if (request.nextUrl.pathname === '/home/teams') {
    return NextResponse.next();
  }

  if (request.nextUrl.pathname === '/home') {
    return redirectToTeam(request, userId);
  }

  if (isTeamAccountRoute(request) && !isUserRoute(request)) {
    return storeTeamSlug(request, userId);
  }

  if (isUserRoute(request)) {
    return redirectToTeam(request, userId);
  }

  return NextResponse.next();
}

function isUserRoute(request: NextRequest) {
  const pathName = request.nextUrl.pathname;
  return ['settings', 'billing', 'members'].includes(pathName.split('/')[2]!);
}

function isTeamAccountRoute(request: NextRequest) {
  const pathName = request.nextUrl.pathname;

  return pathName.startsWith('/home/');
}

function storeTeamSlug(request: NextRequest, userId: string): NextResponse {
  const accountSlug = request.nextUrl.pathname.split('/')[2];
  const response = NextResponse.next();

  if (accountSlug) {
    const cookieName = createTeamCookie(userId);

    response.cookies.set({
      name: cookieName,
      value: accountSlug,
      path: '/',
    });
  }

  return response;
}

function redirectToTeam(request: NextRequest, userId: string): NextResponse {
  const cookieName = createTeamCookie(userId);
  const lastTeamSlug = request.cookies.get(cookieName);

  if (lastTeamSlug) {
    return NextResponse.redirect(
      new URL(`/home/${lastTeamSlug.value}`, request.url),
    );
  }

  return NextResponse.redirect(new URL('/home/teams', request.url));
}
```

We will now add the `handleTeamAccountsOnly` function to the middleware chain in the `apps/web/middleware.ts` file. This function will check if the user is on a team account route and store the team slug in a cookie. If the user is on the home route, it will redirect them to the team selection page.

 ```tsx {% title="apps/web/middleware.ts" %} {31}
 {
  pattern: new URLPattern({ pathname: '/home/*?' }),
  handler: async (req: NextRequest, res: NextResponse) => {
    const {
      data: { user },
    } = await getUser(req, res);

    const origin = req.nextUrl.origin;
    const next = req.nextUrl.pathname;

    // If user is not logged in, redirect to sign in page.
    if (!user) {
      const signIn = pathsConfig.auth.signIn;
      const redirectPath = `${signIn}?next=${next}`;

      return NextResponse.redirect(new URL(redirectPath, origin).href);
    }

    const supabase = createMiddlewareClient(req, res);

    const requiresMultiFactorAuthentication =
      await checkRequiresMultiFactorAuthentication(supabase);

    // If user requires multi-factor authentication, redirect to MFA page.
    if (requiresMultiFactorAuthentication) {
      return NextResponse.redirect(
        new URL(pathsConfig.auth.verifyMfa, origin).href,
      );
    }

    return handleTeamAccountsOnly(req, user.id);
  },
}
```

In the above code snippet, we have added the `handleTeamAccountsOnly` function to the middleware chain.

## Creating the Team Selection Page

Next, we need to create a team selection page where users can select the team they want to log in to. We will create a new page at `apps/web/app/home/teams/page.tsx`:

 ```tsx {% title="apps/web/app/home/teams/page.tsx" %}
import { PageBody, PageHeader } from '@kit/ui/page';
import { Trans } from '@kit/ui/trans';

import { HomeAccountsList } from '~/home/(user)/_components/home-accounts-list';
import { createI18nServerInstance } from '~/lib/i18n/i18n.server';
import { withI18n } from '~/lib/i18n/with-i18n';

export const generateMetadata = async () => {
  const i18n = await createI18nServerInstance();
  const title = i18n.t('account:homePage');

  return {
    title,
  };
};

function TeamsPage() {
  return (
    <div className={'container flex flex-col flex-1 h-screen'}>
      <PageHeader
        title={<Trans i18nKey={'common:routes.home'} />}
        description={<Trans i18nKey={'common:homeTabDescription'} />}
      />

      <PageBody>
        <HomeAccountsList />
      </PageBody>
    </div>
  );
}

export default withI18n(TeamsPage);
```

The page is extremely minimal, it just displays a list of teams that the user can select from. You can customize this page to fit your application's design.

## Duplicating the User Settings Page

Finally, we need to duplicate the user settings page so that users can access their settings from within the team workspace.

We will create a new page called `user-settings.tsx` in the `apps/web/app/home/[account]` directory.

 ```tsx {% title="apps/web/app/home/[account]/user-settings/page.tsx" %}
import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';
import { PageHeader } from '@kit/ui/page';

import UserSettingsPage, { generateMetadata } from '../../(user)/settings/page';

export { generateMetadata };

export default function Page() {
  return (
    <>
      <PageHeader title={'User Settings'} description={<AppBreadcrumbs />} />

      <UserSettingsPage />
    </>
  );
}
```

Feel free to customize the path or the content of the user settings page.

### Adding the page to the Navigation Menu

Finally, you can add the `Teams` page to the navigation menu.

You can do this by updating the `apps/web/config/team-account-navigation.config.tsx` file:

 ```tsx {% title="apps/web/config/team-account-navigation.config.tsx" %} {26-30}
import { CreditCard, LayoutDashboard, Settings, User, Users } from 'lucide-react';

import { NavigationConfigSchema } from '@kit/ui/navigation-schema';

import featureFlagsConfig from '~/config/feature-flags.config';
import pathsConfig from '~/config/paths.config';

const iconClasses = 'w-4';

const getRoutes = (account: string) => [
  {
    label: 'common:routes.application',
    collapsible: false,
    children: [
      {
        label: 'common:routes.dashboard',
        path: pathsConfig.app.accountHome.replace('[account]', account),
        Icon: <LayoutDashboard className={iconClasses} />,
        end: true,
      }
    ],
  },
  {
    label: 'common:routes.settings',
    collapsible: false,
    children: [
      {
        label: 'common:routes.settings',
        path: createPath(pathsConfig.app.accountSettings, account),
        Icon: <Settings className={iconClasses} />,
      },
      {
        label: 'common:routes.account',
        path: createPath('/home/[account]/user-settings', account),
        Icon: <User className={iconClasses} />,
      },
      {
        label: 'common:routes.members',
        path: createPath(pathsConfig.app.accountMembers, account),
        Icon: <Users className={iconClasses} />,
      },
      featureFlagsConfig.enableTeamAccountBilling
        ? {
            label: 'common:routes.billing',
            path: createPath(pathsConfig.app.accountBilling, account),
            Icon: <CreditCard className={iconClasses} />,
          }
        : undefined,
    ].filter(Boolean),
  },
];

export function getTeamAccountSidebarConfig(account: string) {
  return NavigationConfigSchema.parse({
    routes: getRoutes(account),
    style: process.env.NEXT_PUBLIC_TEAM_NAVIGATION_STYLE,
  });
}

function createPath(path: string, account: string) {
  return path.replace('[account]', account);
}
```

In the above code snippet, we have added the `User Settings` page to the navigation menu.

## Removing Personal Account menu item

To remove the personal account menu item, you can remove the personal account menu item:

 ```tsx {% title="packages/features/accounts/src/components/account-selector.tsx" %}
<CommandGroup>
  <CommandItem
    onSelect={() => onAccountChange(undefined)}
    value={PERSONAL_ACCOUNT_SLUG}
  >
    <PersonalAccountAvatar />

    <span className={'ml-2'}>
      <Trans i18nKey={'teams:personalAccount'} />
    </span>

    <Icon item={PERSONAL_ACCOUNT_SLUG} />
  </CommandItem>
</CommandGroup>

<CommandSeparator />
```

Once you remove the personal account menu item, users will only see the team accounts in the navigation menu.

## Change Redirect in Layout

We now need to change the redirect (in case of errors) from `/home` to `/home/teams`. This is to avoid infinite redirects in case of errors.

 ```tsx {% title="apps/web/app/home/[account]/_lib/server/team-account-workspace.loader.ts" %} {13}
async function workspaceLoader(accountSlug: string) {
  const client = getSupabaseServerClient();
  const api = createTeamAccountsApi(client);

  const [workspace, user] = await Promise.all([
    api.getAccountWorkspace(accountSlug),
    requireUserInServerComponent(),
  ]);

  // we cannot find any record for the selected account
  // so we redirect the user to the home page
  if (!workspace.data?.account) {
    return redirect('/home/teams');
  }

  return {
    ...workspace.data,
    user,
  };
}
```

## Conclusion

By following these steps, you can disable personal accounts in the Next.js Supabase Turbo SaaS kit and only allow team accounts.

This can help you create a more focused and collaborative environment for your users. Feel free to customize the team selection page and user settings page to fit your application's design and requirements.


-----------------
FILE PATH: kits\next-supabase-turbo\recipes.mdoc

---
status: "published"
title: "Recipes in Next.js Supabase Turbo"
label: "Recipes"
order: 16
description: "Recipes in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Recipes are common patterns and solutions for your SaaS application:

1. [**Subscription Management**](recipes/subscription-entitlements)
   Handle subscription entitlements and access control.
2. [**Checkout Flows**](recipes/onboarding-checkout)
   Implement smooth onboarding and checkout processes.
3. [**Team Management**](recipes/team-accounts-only)
   Set up team-only functionality.
4. [**Data Modeling**](recipes/projects-data-model)
   Learn about common data modeling patterns.
5. [**Stripe Integration**](recipes/stripe-billing-checkout-addons)
   Implement advanced Stripe billing features.


-----------------
FILE PATH: kits\next-supabase-turbo\security\csp.mdoc

---
label: "Content Security Policy (CSP)"
title: "Content Security Policy (CSP) in the Next.js Supabase Turbo kit"
description: "Learn how to secure your Next.js application with Content Security Policy (CSP) using the Next.js Supabase Turbo kit."
order: 4
---

The Next.js Supabase Turbo kit provides a secure way to include [CSP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) in your application. This functionality is available starting from version 2.8.0 and uses [Nosecone](https://docs.arcjet.com/nosecone/reference#headers) to generate the CSP headers.

{% sequence title="Steps to configure CSP" description="Learn how to configure CSP in the Next.js Supabase Turbo kit." %}

[How to enable a strict CSP](#how-to-enable-a-strict-csp)

[Making your application compliant with CSP](#making-your-application-compliant-with-csp)

[Adding a nonce to your application](#adding-a-nonce-to-your-application)

[Modifying the default CSP configuration](#modifying-the-default-csp-configuration)

{% /sequence %}

## How to enable a strict CSP

To enable a strict CSP, you need to set the following environment variable:

```bash
ENABLE_STRICT_CSP=true
```

By default, this setting is disabled due to the overhead required for development. While this is a bit advanced, it is recommended to enable it before going to production - or at some point in your development process.

Once enabled, the CSP headers will be automatically added to the response headers using the Next.js middleware. 

## Making your application compliant with CSP

Enabling a strict CSP has a few consequences on your application - you will need to make sure your application is compliant with the CSP rules.

1. You will need to add the `nonce` to any third-party script tags or stylesheets you have in your application.
2. If you make external HTTP requests, you will need to add the domains to the default list of allowed domains (by default, only requests to your Supabase project are allowed).

## Adding a nonce to your application

From a Server Component, you can retrieve a nonce from the `headers()` object.

```tsx
import { headers } from "next/headers";

const headersStore = await headers();
const nonce = headersStore.get("x-nonce");
```

If you want to pass the nonce to a Script tag, you can do so by adding the `nonce` attribute to the script tag.

```tsx
<script nonce={nonce} src="https://example.com/script.js"></script>
```

## Modifying the default CSP configuration

In the application middleware, modify the `apps/web/lib/create-csp-response.ts` file to modify the CSP configuration. You may need to do this to allow safe external requests that your application makes.

Please refer to the [Nosecone documentation](https://docs.arcjet.com/nosecone/reference#headers) for more information on the available options.

```ts {% filename="apps/web/lib/create-csp-response.ts" %}
const config: NoseconeOptions = {
...noseconeConfig,
contentSecurityPolicy: {
    directives: {
    ...noseconeConfig.contentSecurityPolicy.directives,
    connectSrc: [
        ...noseconeConfig.contentSecurityPolicy.directives.connectSrc,
        ...ALLOWED_ORIGINS,
    ],
    imgSrc: [
        ...noseconeConfig.contentSecurityPolicy.directives.imgSrc,
        ...IMG_SRC_ORIGINS,
    ],
    upgradeInsecureRequests: UPGRADE_INSECURE_REQUESTS,
    },
},
crossOriginEmbedderPolicy: CROSS_ORIGIN_EMBEDDER_POLICY,
};
```

-----------------
FILE PATH: kits\next-supabase-turbo\security\data-validation.mdoc

---
label: "Data Validation"
title: "Data Validation in the Next.js Supabase Turbo kit"
description: "Learn how to validate data in the Next.js Supabase Turbo kit."
order: 3
---

Data Validation is a crucial aspect of building secure applications. In this section, we will look at how to validate data in the Next.js Supabase Turbo kit.

{% sequence title="Steps to validate data" description="Learn how to validate data in the Next.js Supabase Turbo kit." %}

[What are we validating?](#what-are-we-validating)

[Using Zod to validate data](#using-zod-to-validate-data)

[Validating payload data to Server Side API](#validating-payload-data-to-server-side-api)

[Validating Cookies](#validating-cookies)

[Validating URL parameters](#validating-url-parameters)

{% /sequence %}

## What are we validating?

A general rule, is that all client-side provided data should be validated/sanitized. This includes:

- URL parameters
- Search params
- Form data
- Cookies
- Any data provided by the user

## Using Zod to validate data

The Next.js Supabase Turbo kit uses [Zod](https://zod.dev/) to validate data. You can use the `z` object to validate data in your application.

```ts
import { z } from "zod";

const userSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

// Validate the data
const validatedData = userSchema.parse(data);
```

## Validating payload data to Server Side API

We generally use two ways for sending data from a client to a server:

1. Server Actions
2. API Route Handlers

Let's look at how we can validate data for both of these cases.

### Server Actions: Using the "enhanceAction" utility

The `enhanceAction` hook is a utility hook that enhances the `action` function with Zod validation.

```ts
'use server';

import { enhanceAction } from "@kit/next/actions";
import { z } from "zod";

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

export const createUserAction = enhanceAction(async (parsedInput, user) => {
  // parsedInput is now validated against the UserSchema

  // do something with the validated data
}, {
    schema: UserSchema,
    auth: true,
});
```

The `enhanceAction` hook is a utility hook that enhances Next.js Server Actions with Zod validation and authentication.

When you define an action using the `enhanceAction` hook, the `parsedInput` argument is now validated against the `schema` option automatically, without you having to do anything else. In addition, the `user` argument is now available to you, which is the authenticated user.

### API Route Handlers: Using the "enhanceRouteHandler" utility

The `enhanceRouteHandler` hook is a utility hook that enhances Next.js API Route Handlers with Zod validation.

```ts
import { enhanceRouteHandler } from "@kit/next/api-routes";
import { z } from "zod";

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

export const POST = enhanceRouteHandler(async ({ parsedInput, user, request }) => {
  // parsedInput is now validated against the UserSchema
  // user is the authenticated user
  // request is the incoming request

  // do something with the validated data
}, {
    schema: UserSchema,
    auth: true,
});
```

Very similar to the `enhanceAction` hook, the `enhanceRouteHandler` hook enhances the `handler` function with Zod validation and authentication.

When you define an API route using the `enhanceRouteHandler` hook, the `parsedInput` argument is now validated against the `schema` option automatically, without you having to do anything else. In addition, the `user` argument is now available to you, which is the authenticated user. Instead, the `request` argument is the incoming request object that you would have received in a standard API route handler.

## Validating Cookies

Whenever you use a cookie in your application, you should validate the cookie data. 

Let's assume you receive a cookie with the name `my-cookie`, and you expect it to be a number. You can validate the cookie data as follows:

```ts
import { z } from "zod";
import { cookies } from "next/headers";

const cookieStore = await cookies();

const cookie = z.coerce.number()
    .safeParse(cookieStore.get("my-cookie")?.value);

if (!cookie.success) {
  // handle the error or provide a default value
}
```

## Validating URL parameters

Whenever you receive a URL parameter, you should validate the parameter data. Let's assume you receive a URL parameter with the name `my-param`, and you expect it to be a number. You can validate the parameter data as follows:

```ts
interface PageProps {
    searchParams: Promise<{ myParam: string }>;
}

async function Page({ searchParams }: PageProps) {
    const params = await searchParams;

    const param = z.coerce.number()
        .safeParse(params.myParam);

    if (!param.success) {
        // handle the error or provide a default value
    }

    // render the page with the validated data
}
```

Going forward, remember to validate all data that you receive from the client, and never trust anything the client provides you with. Always have a default value ready to handle invalid data, which can prevent potential security issues or bugs in your application.

-----------------
FILE PATH: kits\next-supabase-turbo\security\nextjs-best-practices.mdoc

---
label: "Next.js Best Practices"
title: "Next.js Security Best Practices"
description: "Learn best practices for Next.js in the Next.js Supabase Turbo kit."
order: 1
---

This section contains a list of best practices for Next.js in general, applicable to both the Next.js Supabase Turbo kit or any other Next.js application.

{% sequence title="Best practices for Next.js in the Next.js Supabase Turbo kit" description="Learn best practices for Next.js in the Next.js Supabase Turbo kit." %}

[Do not pass sensitive data to client components](#do-not-pass-sensitive-data-to-client-components)

[Do not mix up client and server imports](#do-not-mix-up-client-and-server-imports)

[Environment variables](#environment-variables)

[Proper use of .env files](#proper-use-of-.env-files)

{% /sequence %}

## What goes in client components will be passed to the client

The first and most important rule you must remember is that what goes in client components will be passed to the client.

From this rule, you can derive the following best practices:

- Do not pass sensitive data to client components. If you're using an API Key server-side, do not pass it to the client.
- Avoid exporting server and client code from the same file. Instead, define separate entry points for server and client code.
- Use the `import 'server-only'` package to raise errors when server code unexpectedly ends up in the client bundle.

## Do not pass sensitive data to client components

One common mistake made by Next.js developers is passing sensitive data to client components. This is easier than it sounds, and it can happen without you even noticing it.

Let's look at the following example: we are using an API call in a server component, and we end up passing the API Key to the client.

```tsx
async function ServerComponent() {
    const config = {
        apiKey: process.env.API_KEY,
        storeId: process.env.STORE_ID,
    };

    const data = await fetchData(config);

    return <ClientComponent data={data} config={config} />;
}
```

In this example, the `config` object contains sensitive data, such as the API Key and the Store ID (which is a public identifier for the store, so safe to pass to the client). This data will be passed to the client, and can be accessed by the client.

```tsx
'use client';

function ClientComponent({ data, config }: { data: any, config: any }) {
    // ...
}
```

This is a problem, because the `config` object contains sensitive data, such as the API Key and the Store ID. The fact we pass it down to a `'use client'` component means that the data will be passed to the client and this means leaking sensitive data to the client, which is a security risk.

This can happen in many other ways, and it's a good idea to be aware of this.

A better approach is to define a service using `import 'server-only'` and use it in the server component.

```ts
import 'server-only';

const config = {
    apiKey: process.env.API_KEY,
    storeId: process.env.STORE_ID,
};

export async function fetchData() {
    const data = await fecthDataFromExternalApi(config);
    const storeId = config.storeId;

    return { data, storeId };
}
```

Now, we can use this service in the server component.

```tsx
import { fetchData } from './fetch-data';

async function ServerComponent() {
    const data = await fetchData();

    return <ClientComponent data={data} />;
}
```

While this doesn't fully solve the problem (you can still pass the config object to the client, but it's a lot harder), it's a good start and will help you separate concerns.

The [Taint API](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching#preventing-sensitive-data-from-being-exposed-to-the-client) will eventually help us solve this issue even better, but it's still experimental.

## Do not mix up client and server imports

Sometimes, you have a package that exports both client and server code. This is a problem, because it will end up in the client bundle.

For example, we assume we have a barrel file `index.ts` from which we export a `fetchData` function and a client component `ClientComponent`.

```ts
export * from './fetch-data';
export * from './client-component';
```

This is a problem, because it's possible that some server-side code ends up in the client bundle. 

Adding `import 'server-only'` to the barrel file will solve this issue and raise an error, however, as a best practice, you should avoid this in the first place and **use different barrel files for server and client code**; then use the `exports` field in `package.json` to re-export the server and client code from the barrel file.

```json
{
    "exports": {
        "./server": "./server.ts",
        "./client": "./client.tsx"
    }
}
```

This way, you can import the server and client code separately and you won't end up with a mix of server and client code in the client bundle.

## Environment variables

Environment variables are essential for configuring your application across different environments. However, they require careful management to prevent security vulnerabilities.

### Use NEXT_PUBLIC prefix for environment variables that are available on the client

Next.js handles environment variables uniquely, distinguishing between server-only and client-available variables:

- Variables without the `NEXT_PUBLIC_` prefix are only available on the server
- Variables with the `NEXT_PUBLIC_` prefix are available on both server and client

Client components can only access environment variables prefixed with `NEXT_PUBLIC_`:

```tsx
// This is available in client components
console.log(process.env.NEXT_PUBLIC_API_URL)

// This is undefined in client components
console.log(process.env.SECRET_API_KEY)
```

### Never use NEXT_PUBLIC_ variables for sensitive data

Since `NEXT_PUBLIC_` variables are embedded in the JavaScript bundle sent to browsers, they should never contain sensitive information:

```
# .env
# UNSAFE: This will be exposed to the client
NEXT_PUBLIC_API_KEY=sk_live_1234567890  # NEVER DO THIS!
```

#### SAFE: This is only available server-side
```
API_KEY=sk_live_1234567890  # This is correct
```

Remember:

1. API keys, secrets, tokens, and passwords should never use the `NEXT_PUBLIC_` prefix
2. Use `NEXT_PUBLIC_` only for genuinely public information like public API URLs, feature flags, or public identifiers

### Proper use of .env files

Next.js supports various .env files for different environments:

```
.env                # Loaded in all environments
.env.local          # Loaded in all environments, ignored by git
.env.development    # Only loaded in development environment
.env.production     # Only loaded in production environment
.env.production.local # Only loaded in production environment, ignored by git
.env.test           # Only loaded in test environment
```

As a general rule, **never add sensitive data to the `.env` file or any other committed file**. Instead, add it to the `.env.local` file, which by default is ignored by git (though you must check this if you're not using Makerkit).

### Best practices:

1. Store sensitive values in `.env.local` which should not be committed to your repository
2. Use environment-specific files for values that differ between environments
3. Use environment variables for sensitive data, not hardcoded values
4. Use `NEXT_PUBLIC_` prefix for environment variables that are available on the client
5. Never use `NEXT_PUBLIC_` variables for sensitive data
6. Use `import 'server-only'` for server-only code
7. Separate server and client code in different files and never mix them up in barrel files
8. Use the Taint API to prevent sensitive data from being exposed to the client (experimental). Makerkit will at some point adopt this API.


-----------------
FILE PATH: kits\next-supabase-turbo\security\row-level-security.mdoc

---
label: "Row Level Security"
title: "Row Level Security in the Next.js Supabase Turbo kit"
description: "Learn how to secure your Next.js application with Row Level Security (RLS) using the Next.js Supabase Turbo kit."
order: 2
---

If you've opted to using the Data API in your application, you can use Row Level Security (RLS) to secure your data (which Makerkit does by default).

{% sequence title="Steps to secure your data with RLS" description="Learn how to secure your data with RLS using the Next.js Supabase Turbo kit." %}

[Enable RLS for your tables](#enable-rls-for-your-tables)

[Add the RLS policy](#add-the-rls-policy)

[Using Makerkit's Functions to write RLS policies](#using-makerkits-functions-to-write-rls-policies)

[Testing RLS policies](#testing-rls-policies)

{% /sequence %}

The general rule of thumb is that you must always ensure RLS is enabled for your tables. **Failure to do so will result in a security vulnerability** because the table will be exposed to the public - and everyone will be able to read and write to it. You don't want that.

Supabase has a great [guide on how to use RLS](https://supabase.com/docs/guides/database/postgres/row-level-security) - don't skip it!

## Enable RLS for your tables

When you write your table migrations, you must ensure that RLS is enabled for the table.

First, create a table with the following command:

```sql
create table if not exists public.documents (
    id uuid primary key default uuid_generate_v4(),
    user_id uuid not null references auth.users(id),
    title text not null,
    content text not null,
    created_at timestamp with time zone default now(),
    updated_at timestamp with time zone default now()
);
```

### Enable RLS for the table

Now, you need to enable RLS for the table.

```sql
alter table public.documents enable row level security;
```

PS: you can also enable RLS for the table using the Supabase Studio - however I am not sure I'd recommend this approach.

## Add the RLS policy for restricted access

Now that we have a table with RLS enabled, **and no policies added**, you will notice that you won't be able to read or write to the table. This is good, because it means that the table is secure. However, we want to be able to read and write to the table **for the users that are authorized to do so**.

To do this, we need to add a policy to the table.

```sql
create policy "Users can view their own documents" 
    on public.documents 
    to authenticated
    for select
    using ((select auth.uid()) = user_id);
```

This policy will allow any **authenticated** user to read **their own documents** - e.g. the documents whose `user_id` matches the authenticated user's ID.

If you wanted to allow all actions at once, you could use the following policy:

```sql
create policy "Users can view their own documents" 
    on public.documents 
    to authenticated
    for all 
    using ((select auth.uid()) = user_id);
```

## Using Makerkit's Functions to write RLS policies

Makerkit comes with a set of functions that make it easier to write RLS policies. Below are some of the most common use cases:

### Verifying a user is part of a Team Account

If you want to verify that a user is part of a Team Account, you can use the `public.has_role_on_account` function.

```sql
create policy "Users can view their account's documents" 
    on public.documents 
    to authenticated
    for select 
    using (public.has_role_on_account(account_id));
```

If you want to check for a specific role, you can do so by passing the role name to the function.

```sql
create policy "Users can view their account's documents" 
    on public.documents 
    to authenticated
    for select 
    using (public.has_role_on_account(account_id, 'admin'));
```

NB: we're expecting the `public.documents` table to have an `account_id` column that references the `public.accounts` table.

### Verifying a user has the required permissions

If you want to verify that a user has the required permissions, you can use the `public.has_permission` function.

```sql
create policy "Users can view their account's documents" 
    on public.documents 
    to authenticated
    for select 
    using (public.has_permission(auth.uid(), account_id, 'documents.read'));
```

### Verifying a user is the owner of a Team Account

If you want to verify that a user is the owner of a Team Account, you can use the `public.is_owner` function.

```sql
create policy "Users can view their account's documents" 
    on public.documents 
    to authenticated
    for select 
    using (public.is_account_owner(account_id));
```

These are just some of the most common use cases and will likely cover the vast majority of your RLS policies.

## Testing RLS policies

We have two different ways to test RLS policies:

1. **Manually**: Using the Supabase Studio impersonation feature
2. **Automatically**: Using [pgTap](https://supabase.com/docs/reference/cli/supabase-test-db) tests

### Manually testing RLS policies

To test RLS policies manually, you can use the Supabase Studio impersonation feature.

1. Go to the Supabase Studio
2. Navigate to the Table you want to test
3. Impersonate a user
4. Try to read, write or delete data using the UI or the SQL editor
5. Verify that the data is restricted based on the RLS policy

### Automatically testing RLS policies

To test RLS policies automatically, you can use the `supabase test db` command. This command uses pgTap to test the RLS policies - which is an invaluable tool for testing RLS policies in an automated and repeatable way.

We have a set of tests in the `supabase/tests/database` folder that are designed to test the RLS policies. You can copy the same structure and add your own tests.

Here's an example of a test:

```sql
-- authenticate with a user
select tests.create_supabase_user('testuser', 'testuser@test.com');
select tests.create_supabase_user('testuser2', 'testuser2@test.com');

-- authenticate as the first user
select tests.authenticate_as('testuser');

-- create a document
insert into public.documents (user_id, title, content)
values (tests.get_supabase_uid('testuser'), 'Test Document', 'This is a test document');

-- test the user can read their own document
select row_eq(
    $$ SELECT * from public.documents $$,
    row(tests.get_supabase_uid('testuser'), 'Test Document', 'This is a test document'),
    'User should be able to read their own document'
);

-- alternatively, check the list is not empty
select not_empty(
    $$ SELECT * from public.documents $$,
    'User should be able to read their own document'
);

-- authenticate with another user
select tests.authenticate_as('testuser2');

-- test that the document is not visible to the other user
select is_empty(
    $$ SELECT * from public.documents $$,
    'No documents should be visible to unauthenticated users'
);
```

This is a simple example, but you can see how you can test the RLS policies for different scenarios.



