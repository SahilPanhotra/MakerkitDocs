-----------------
FILE PATH: kits\next-supabase-turbo\analytics\custom-analytics-provider.mdoc

---
status: "published"

title: 'Creating a Custom Analytics Provider in Makerkit'
label: 'Custom Analytics Provider'
description: 'Learn how to create a custom analytics provider in Makerkit to integrate with your preferred analytics service.'
order: 5
---


The Analytics package in Makerkit is meant to be flexible and extensible. You can easily create custom analytics providers to integrate with your preferred analytics service.

To create a custom analytics provider, you need to implement the `AnalyticsService` interface and then register it with the `AnalyticsManager`.

You can define one or more custom analytics providers, and the `AnalyticsManager` will handle the initialization and tracking of events for each provider.

NB: the methods expect to be Promise-based, so you can use async/await or return a Promise, or use the keyword `void` to ignore the return value and use it in non-async functions.

## Create your custom analytics service

Let's create a custom analytics service that implements the `AnalyticsService` interface:

 ```typescript {% title="packages/analytics/src/my-custom-analytics-service.ts" %}
import { AnalyticsService } from './path-to-types';

class MyCustomAnalyticsService implements AnalyticsService {
  async initialize() {
    // Initialize your analytics service
  }

  async identify(userId: string, traits?: Record<string, string>) {
    // Implement user identification
  }

  async trackPageView(url: string) {
    // Implement page view tracking
  }

  async trackEvent(eventName: string, eventProperties?: Record<string, string | string[]>) {
    // Implement event tracking
  }
}
```

### Register your custom provider

Update your analytics configuration file to include your custom provider:

 ```typescript {% title="packages/analytics/src/index.ts" %} {6, 8}
import { createAnalyticsManager } from './analytics-manager';
import { MyCustomAnalyticsService } from './my-custom-analytics-service';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
  providers: {
    myCustom: (config) => new MyCustomAnalyticsService(config),
    null: () => NullAnalyticsService,
  },
});
```

## Using the Custom Analytics Provider

Once you've created and registered your custom analytics provider, you can use it in your application as you would with any other analytics provider:

All the registered providers will dispatch the same events, so you can use them interchangeably:

```typescript
const analytics = createAnalyticsManager({
  providers: {
    googleAnalytics: (config) => new GoogleAnalyticsService(config),
    mixpanel: (config) => new MixpanelService(config),
    myCustom: (config) => new MyCustomAnalyticsService(config),
    null: () => NullAnalyticsService,
  },
});
```

That's it! You've successfully created a custom analytics provider in Makerkit. You can now integrate with any analytics service of your choice.

## Using the Custom Analytics Provider

Once you've created and registered your custom analytics provider, you can use it in your application as you would with any other analytics provider:

```typescript
import { analytics } from '@kit/analytics';

void analytics.identify('user123', { name: 'John Doe' });
void analytics.trackEvent('Button Clicked', { buttonName: 'Submit' });
```

That's it! You've successfully created a custom analytics provider in Makerkit. You can now integrate with any analytics service of your choice.

-----------------
FILE PATH: kits\next-supabase-turbo\analytics\google-analytics-provider.mdoc

---
status: "published"

title: 'Using the Google Analytics Provider in Next.js Supabase Turbo'
label: 'Google Analytics'
description: 'Learn how to use the Google Analytics provider in Next.js Supabase Turbo'
order: 2
---


The [Google Analytics](https://marketingplatform.google.com/about/analytics/) provider in Next.js Supabase Turbo is a simple way to integrate Google Analytics into your Next.js application using the Makerkit's Analytics package.

## Installation

First, you need to pull the `@kit/analytics` package into your project using the CLI

```bash
npx @makerkit/cli@latest plugins install
```

When prompted, select the `Google Analytics` package from the list of available packages. Once the command completes, you should see the `packages/plugins/google-analytics` directory in your project.

You can now import this package into your project:

```bash
pnpm add "@kit/google-analytics@workspace:*" --filter "@kit/analytics" -D
```

You can now use the Google Analytics plugin in the Analytics package. Update the `packages/analytics/src/index.ts` file as follows:

 ```tsx {% title="packages/analytics/src/index.ts" %}
import { createGoogleAnalyticsService } from '@kit/google-analytics';

import { createAnalyticsManager } from './analytics-manager';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
    providers: {
        'google-analytics': createGoogleAnalyticsService,
    },
});
```

## Configuration

Please add the following environment variables to your `.env` file:

```bash
NEXT_PUBLIC_GA_MEASUREMENT_ID=your-measurement-id
```

This is the Measurement ID of your Google Analytics property. You can find it in the Google Analytics dashboard.

Additionally, you can add the following environment variable to your `.env` file:

```bash
NEXT_PUBLIC_GA_DISABLE_PAGE_VIEWS_TRACKING=true
NEXT_PUBLIC_GA_DISABLE_LOCALHOST_TRACKING=true
```


-----------------
FILE PATH: kits\next-supabase-turbo\analytics\posthog-analytics-provider.mdoc

---
status: "published"
title: 'Using the PostHog Analytics Provider in Next.js Supabase Turbo'
label: 'PostHog'
description: 'Learn how to use the PostHog Analytics provider in Next.js Supabase Turbo'
order: 3
---

The [Posthog](https://posthog.com) provider in Next.js Supabase Turbo is a simple way to integrate PostHog Analytics into your Next.js application using the Makerkit's Analytics package.

## Installation

First, you need to pull the `@kit/analytics` package into your project using the CLI

```bash
npx @makerkit/cli@latest plugins install
```

When prompted, select the `PostHog` package from the list of available packages. Once the command completes, you should see the `packages/plugins/posthog` directory in your project.

You can now import this package into your project:

```bash
pnpm add "@kit/posthog@workspace:*" --filter "@kit/analytics" -D
```

You can now use the PostHog plugin in the Analytics package. Update the `packages/analytics/src/index.ts` file as follows:

 ```tsx {% title="packages/analytics/src/index.ts" %}
import { createPostHogAnalyticsService } from '@kit/posthog';

import { createAnalyticsManager } from './analytics-manager';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
  providers: {
    posthog: createPostHogAnalyticsService,
  },
});
```

## Configuration

Please add the following environment variables to your `.env` file:

```bash
NEXT_PUBLIC_POSTHOG_KEY=phc_your_key_here
NEXT_PUBLIC_POSTHOG_HOST=https://eu.posthog.com
NEXT_PUBLIC_POSTHOG_INGESTION_URL=http://localhost:3000/ingest
```

### Ingestion Rewrites

In your apps/web/next.config.mjs file, add the following config:

 ```js {% title="apps/web/next.config.mjs" %}
/** @type {import('next').NextConfig} */
const config = {
  // ...other config
  // This is required to support PostHog trailing slash API requests
  skipTrailingSlashRedirect: true,
  async rewrites() {
    // NOTE: change `eu` to `us` if applicable
    return [
      {
        source: '/ingest/static/:path*',
        destination: 'https://eu-assets.i.posthog.com/static/:path*'
      },
      {
        source: '/ingest/:path*',
        destination: 'https://eu.i.posthog.com/:path*'
      }
    ];
  }
}
```

### CSRF Exclusion

In your `apps/web/middleware.ts` file, exclude the PostHog ingestion URL from CSRF protection:

 ```ts {% title="apps/web/middleware.ts" %}
export const config = {
  matcher: ['/((?!_next/static|_next/image|images|locales|assets|ingest/*|api/*).*)'],
};
```


-----------------
FILE PATH: kits\next-supabase-turbo\analytics\umami-analytics-provider.mdoc

---
status: "published"

title: 'Using the Umami Analytics Provider in Next.js Supabase Turbo'
label: 'Umami'
description: 'Learn how to use the Umami Analytics provider in Next.js Supabase Turbo'
order: 4
---


The [Umami](https://umami.is/) analytics provider in Next.js Supabase Turbo is a simple way to integrate Umami Analytics into your Next.js application using the Makerkit's Analytics package.

## Installation

First, you need to pull the `@kit/analytics` package into your project using the CLI

```bash
npx @makerkit/cli@latest plugins install
```

When prompted, select the `Umami` package from the list of available packages. Once the command completes, you should see the `packages/plugins/umami` directory in your project.

You can now import this package into your project:

```bash
pnpm add "@kit/umami@workspace:*" --filter "@kit/analytics" -D
```

You can now use the Google Analytics plugin in the Analytics package. Update the `packages/analytics/src/index.ts` file as follows:

 ```tsx {% title="packages/analytics/src/index.ts" %}
import { createUmamiAnalyticsService } from '@kit/umami';

import { createAnalyticsManager } from './analytics-manager';
import type { AnalyticsManager } from './types';

export const analytics: AnalyticsManager = createAnalyticsManager({
    providers: {
        umami: createUmamiAnalyticsService,
    },
});
```

## Configuration

Please add the following environment variables to your `.env` file:

```bash
NEXT_PUBLIC_UMAMI_HOST=your-umami-host
NEXT_PUBLIC_UMAMI_WEBSITE_ID=your-umami-website-id
```

The `NEXT_PUBLIC_UMAMI_HOST` is the URL of your Umami instance's script. Since Umami can be self-hosted, this can be any valid URL - or can be Umami's cloud service. For example, `https://umami.is/umami.js`. Please replace with the correct path to your Umami instance's JS script.

The `NEXT_PUBLIC_UMAMI_WEBSITE_ID` is the ID of your website in your Umami instance. This is a required field to track events in your website.

NB: by default, Umami doesn't track events on localhost. You can use the `NEXT_PUBLIC_UMAMI_DISABLE_LOCALHOST_TRACKING` environment variable to enable tracking on localhost.

```
NEXT_PUBLIC_UMAMI_DISABLE_LOCALHOST_TRACKING=false
```

This is useful for testing your analytics setup locally.


-----------------
FILE PATH: kits\next-supabase-turbo\analytics.mdoc

---
status: "published"
title: 'Using the Analytics API in your Makerkit project'
label: 'Analytics API'
order: 13
description: 'Learn how to use the Analytics API in your Makerkit project to track user behavior and app usage.'
collapsible: true
collapsed: true
---

Makerkit provides a powerful and flexible Analytics API that integrates seamlessly with the App Events system. This API allows you to track user behavior and app usage easily and consistently across your SaaS application.

## Core Concepts of the Analytics API

The Analytics API is built around three main concepts:

1. **Identify**: Associate user data with a unique user ID.
2. **Track Events**: Record specific events or actions taken by users.
3. **Track Page Views**: Record when users view specific pages in your application.

### Using the Analytics API

The Analytics API is available through the `analytics` object imported from `@kit/analytics`.

#### Identifying Users

Use the `identify` method to associate a user with their actions:

```typescript
import { analytics } from '@kit/analytics';

void analytics.identify(userId, {
  email: user.email,
  plan: user.subscriptionPlan,
  // ... other user properties
});
```

#### Tracking Events

Use the `trackEvent` method to record specific actions or events:

```typescript
void analytics.trackEvent('Button Clicked', {
  buttonName: 'Submit',
  page: 'Sign Up',
});
```

#### Tracking Page Views

**Makerkit automatically tracks page views for you.** However, you can also manually track page views if needed.

Use the `trackPageView` method to record when users view specific pages:

```typescript
void analytics.trackPageView('Sign Up');
```

NB: The `trackPageView` method is automatically called when the route changes in a Next.js application.

### Integration with App Events

While you can use these methods directly, we recommend leveraging the App Events system for a more centralized approach. Makerkit automatically maps common app events to analytics tracking:

```typescript
import { useAppEvents } from '@kit/shared/events';

function SomeComponent() {
  const { emit } = useAppEvents();

  const handleSignUp = (userId: string) => {
    emit({ type: 'user.signedUp', payload: { userId } });
    // This automatically calls analytics.identify and analytics.trackEvent
  };

  // ...
}
```

### Extending the Analytics API

You can extend the analytics functionality by creating custom event mappings:

```typescript
import { analytics } from '@kit/analytics';
import { useAppEvents } from '@kit/shared/events';

interface MyAppEvents {
  'feature.used': { featureName: string };
}

export function useMyAnalytics() {
  const { emit } = useAppEvents<MyAppEvents>();

  return {
    trackFeatureUse: (featureName: string) => {
      emit({ type: 'feature.used', payload: { featureName } });
      // If you need additional tracking logic:
      void analytics.trackEvent('Feature Used', { featureName });
    },
  };
}
```

## Best Practices

When implementing analytics in your Makerkit project, consider the following best practices:

1. **Use App Events**: Whenever possible, use the App Events system instead of calling analytics methods directly. This keeps your analytics logic centralized and easier to maintain.
2. **Consistent Naming**: Use consistent naming conventions for your events and properties across your application.
3. **Relevant Data**: Only track data that's relevant and useful for your business goals. Avoid tracking sensitive or personal information.
4. **Testing**: Always test your analytics implementation to ensure events are being tracked correctly.
5. **Documentation**: Keep a record of all the events and properties you're tracking. This will be invaluable as your application grows.

By leveraging Makerkit's Analytics API in conjunction with the App Events system, you can create a robust, maintainable analytics setup that grows with your SaaS application. This approach provides the flexibility to track the data you need while keeping your codebase clean and organized.


-----------------
FILE PATH: kits\next-supabase-turbo\api\account-api.mdoc

---
status: "published"
label: "Account API"
order: 0
title: "Account API | Next.js Supabase SaaS Kit"
description: "A quick introduction to the Account API in Makerkit"
---

You can use the Account API for retrieving information about the personal user account.

## Using the Account API

To use the Account API, you need to import the `createAccountsApi` function from `@kit/account/api`. We need to pass a valid `SupabaseClient` to the function - so we can interact with the database from the server.

```tsx
import { createAccountsApi } from '@kit/accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const api = createAccountsApi(client);

  // use api
}
```

If you're in a Server Action context, you'd use:

```tsx
'use server';

import { createAccountsApi } from '@kit/accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function myServerAction() {
  const client = getSupabaseServerClient();
  const api = createAccountsApi(client);
  
  // use api
}
```

## Methods

The Account API provides the following methods:

### Get the account workspace data

Get the account workspace data using the `getAccountWorkspace` method. This method returns the workspace data for the user account.

```tsx
const api = createAccountsApi(client);
const workspace = await api.getAccountWorkspace();
```

This is already called in the user account layout, so it's very unlikely you'll need to call this method.

### Load the user accounts

Load the user accounts using the `loadUserAccounts` method.

This method returns an array of user accounts. Each account has a `label`, `value`, and `image` property.

```tsx
const api = createAccountsApi(client);
const accounts = await api.loadUserAccounts();
```

### Get the subscription data

Get the subscription data for the given user using the `getSubscription` method.

This method returns the subscription data for the given user account.

```tsx
const api = createAccountsApi(client);
const subscription = await api.getSubscription(accountId);
```

Returns the table `subscriptions` and `subscription_items`.

### Get the billing customer ID

Get the billing customer ID for the given user using the `getCustomerId` method.

This method returns the billing customer ID for the given user account.

```tsx
const api = createAccountsApi(client);
const customerId = await api.getCustomerId(accountId);
```

-----------------
FILE PATH: kits\next-supabase-turbo\api\account-workspace-api.mdoc

---
status: "published"

label: "Account Workspace API"
order: 4
title: "Account Workspace API"
description: "The account workspace API allows you to retrieve all the data related to the current account."
---


When within the layout `/home/[account]` - you have access to data fetched from the account workspace API.

The data in this layout has most of the information you need around the currently selected account and the user.

## Accessing the Team Account Workspace Data in Server Components

To access the data, you can use the `loadTeamWorkspace` loader function. This function is cached per-request, so you can call it multiple times without worrying about performance.

While multiple calls to this function are deduped within a single request, bear in mind that this request will be called when navigating to the page. If you only require a small subset of data, it would be best to make more granular requests.

```tsx
import { loadTeamWorkspace } from '~/home/[account]/_lib/server/team-account-workspace.loader';

export default async function SomeAccountPage() {
  const data = await loadTeamWorkspace();

  // use data
}
```

The data returned from the `loadTeamWorkspace` function is an object with the following properties:

- `account`: The account object
- `user`: The user object coming from Supabase Auth
- `accounts`: An array of all accounts the user is a member of

Here is an example of the data structure:

```tsx
import type { User } from '@supabase/supabase-js';

{
  account: {
    id: string;
    name: string;
    picture_url: string;
    slug: string;
    role: string;
    role_hierarchy_level: number;
    primary_owner_user_id: string;
    subscription_status: string;
    permissions: string[];
  };

  user: User;

  accounts: Array<{
   id: string | null;
    name: string | null;
    picture_url: string | null;
    role: string | null;
    slug: string | null;
  }>;
}
```

The `account` object contains the following properties:
1. `id`: The account ID
2. `name`: The account name
3. `picture_url`: The account picture URL
4. `slug`: The account slug
5. `role`: The user's role in the account
6. `role_hierarchy_level`: The user's role hierarchy level
7. `primary_owner_user_id`: The primary owner user ID
8. `subscription_status`: The subscription status of the account. This can be 'active' | 'trialing' | 'past_due' | 'canceled' | 'unpaid' | 'incomplete' | 'incomplete_expired' | 'paused'.
9. `permissions`: An array of permissions the user has in the account

## Accessing the Account Workspace Data in Client Components

The data fetched from the account workspace API is available in the context. You can access this data using the `useAccountWorkspace` hook.

```tsx
'use client';

import { useTeamAccountWorkspace } from '@kit/team-accounts/hooks/use-team-account-workspace';

export default function SomeComponent() {
  const { account, user, accounts } = useTeamAccountWorkspace();

  // use account, user, and accounts
}
```

The `useTeamAccountWorkspace` hook returns the same data structure as the `loadTeamWorkspace` function.

NB: the hooks **is not to be used** is Server Components, only in Client Components. Additionally, this is only available in the pages under `/home/[account]` layout.

-----------------
FILE PATH: kits\next-supabase-turbo\api\authentication-api.mdoc

---
status: "published"
label: "Authentication API"
order: 2
title: "Authentication API | Next.js Supabase SaaS Kit"
description: "A quick introduction to the Authentication API in Makerkit"
---

To check if users are authed, or to retrieve information about the currently signed-in user, use the `requireUser` function:

```tsx
import { redirect } from 'next/navigation';
import { requireUser } from '@kit/supabase/require-user';

import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const auth = await requireUser(client);

  // check if the user needs redirect
  if (auth.error) {
    redirect(auth.redirectTo);
  }

  // user is authed!
  const user = auth.data;
}
```

NB: use the correct Supabase client based on the context. In this case, we use the server component client.

```tsx
'use server';

import { redirect } from 'next/navigation';
import { requireUser } from '@kit/supabase/require-user';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function myServerAction() {
  const client = getSupabaseServerClient();
  const auth = await requireUser(client);

  // check if the user needs redirect
  if (auth.error) {
    redirect(auth.redirectTo);
  }

  // user is authed!
  const user = auth.data;
}
```

If the user needs MFA and is not yet verified, the `redirect` function will redirect the user to the MFA verification page. This is why it is important to check the `redirectTo` property in the response.


-----------------
FILE PATH: kits\next-supabase-turbo\api\otp-api.mdoc

---
status: "published"
label: "OTP API"
order: 5
title: "OTP API | Next.js Supabase SaaS Kit"
description: "The OTP API is a simple interface for working with one-time passwords (OTPs) in your application."
---

This package provides a complete solution for generating, sending, and verifying one-time passwords or tokens in your application. It integrates with Supabase for secure token storage and verification.

It is used for various destructive actions in the SaaS Kit, such as deleting
accounts, deleting teams, and deleting users. However, you can use it for a
variety of other purposes as well, such as:

- Your custom destructive actions
- oAuth account connections
- etc.

## Overview

The OTP package offers:

- **Secure Token Generation**: Create time-limited tokens with configurable expiration
- **Email Delivery**: Send OTP codes via email with customizable templates
- **Verification UI**: Ready-to-use verification form component
- **Token Management**: Revoke, verify, and check token status

## Installation

If you're using Makerkit, this package is already included. For manual installation:

```bash
pnpm add @kit/otp
```

## Basic Usage

### Creating and Sending an OTP

To create and send an OTP, you can use the `createToken` method:

```typescript
import { createOtpApi } from '@kit/otp/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

// Create the API instance
const client = getSupabaseServerClient();
const api = createOtpApi(client);

// Generate and send an OTP email
await api.createToken({
  userId: user.id,
  purpose: 'email-verification',
  expiresInSeconds: 3600, // 1 hour
  metadata: { redirectTo: '/verify-email' }
});

// Send the email with the OTP
await api.sendOtpEmail({
  email: userEmail,
  otp: token.token
});
```

### Verifying an OTP

To verify an OTP, you can use the `verifyToken` method:

```typescript
// Verify the token
const result = await api.verifyToken({
  token: submittedToken,
  purpose: 'email-verification'
});

if (result.valid) {
  // Token is valid, proceed with the operation
  const { userId, metadata } = result;
  // Handle successful verification
} else {
  // Token is invalid or expired
  // Handle verification failure
}
```

## Server Actions

The package includes a ready-to-use server action for sending OTP emails:

```typescript
import { sendOtpEmailAction } from '@kit/otp/server/server-actions';

// In a form submission handler
const result = await sendOtpEmailAction({
  email: userEmail,
  purpose: 'password-reset',
  expiresInSeconds: 1800 // 30 minutes
});

if (result.success) {
  // OTP was sent successfully
} else {
  // Handle error
}
```

NB: the `email` parameter is only used as verification mechanism, the actual
email address being used is the one associated with the user.

## Verification UI Component

The package includes a ready-to-use OTP verification form:

```tsx
import { VerifyOtpForm } from '@kit/otp/components';

function MyVerificationPage() {
  return (
    <VerifyOtpForm
      purpose="password-reset"
      email={userEmail}
      onSuccess={(otp) => {
        // Handle successful verification
        // Use the OTP for verification on the server
      }}
      CancelButton={
        <Button variant="outline" onClick={handleCancel}>
          Cancel
        </Button>
      }
    />
  );
}
```

## API Reference

### `createOtpApi(client)`

Creates an instance of the OTP API.

**Parameters**:
- `client`: A Supabase client instance
- **Returns**: OTP API instance with the following methods:

### `api.createToken(params)`

Creates a new one-time token.

**Parameters**:
- `params.userId` (optional): User ID to associate with the token
- `params.purpose`: Purpose of the token (e.g., 'password-reset')
- `params.expiresInSeconds` (optional): Token expiration time in seconds (default: 3600)
- `params.metadata` (optional): Additional data to store with the token
- `params.description` (optional): Description of the token
- `params.tags` (optional): Array of string tags
- `params.scopes` (optional): Array of permission scopes
- `params.revokePrevious` (optional): Whether to revoke previous tokens with the same purpose (default: true)

**Returns**:
```typescript
{
  id: string;           // Database ID of the token
  token: string;        // The actual token to send to the user
  expiresAt: string;    // Expiration timestamp
  revokedPreviousCount: number; // Number of previously revoked tokens
}
  ```

### `api.verifyToken(params)`

Verifies a one-time token.

**Parameters**:
- `params.token`: The token to verify
- `params.purpose`: Purpose of the token (must match the purpose used when creating)
- `params.userId` (optional): User ID for additional verification
- `params.requiredScopes` (optional): Array of required permission scopes
- `params.maxVerificationAttempts` (optional): Maximum allowed verification attempts

**Returns**:
```typescript
{
  valid: boolean;       // Whether the token is valid
  userId?: string;      // User ID associated with the token (if valid)
  metadata?: object;    // Metadata associated with the token (if valid)
  message?: string;     // Error message (if invalid)
  scopes?: string[];    // Permission scopes (if valid)
  purpose?: string;     // Token purpose (if valid)
}
  ```

### `api.revokeToken(params)`

Revokes a token to prevent its future use.

**Parameters**:
- `params.id`: ID of the token to revoke
- `params.reason` (optional): Reason for revocation

**Returns**:
```typescript
{
  success: boolean;     // Whether the token was successfully revoked
}
  ```

### `api.getTokenStatus(params)`

Gets the status of a token.

**Parameters**:
- `params.id`: ID of the token

**Returns**:
```typescript
{
  exists: boolean;      // Whether the token exists
  purpose?: string;     // Token purpose
  userId?: string;      // User ID associated with the token
  createdAt?: string;   // Creation timestamp
  expiresAt?: string;   // Expiration timestamp
  usedAt?: string;      // When the token was used (if used)
  revoked?: boolean;    // Whether the token is revoked
  revokedReason?: string; // Reason for revocation (if revoked)
  verificationAttempts?: number; // Number of verification attempts
  lastVerificationAt?: string;  // Last verification attempt timestamp
  lastVerificationIp?: string;  // IP address of last verification attempt
  isValid?: boolean;    // Whether the token is still valid
}
  ```

### `api.sendOtpEmail(params)`

Sends an email containing the OTP code.

**Parameters**:
- `params.email`: Email address to send to
- `params.otp`: OTP code to include in the email

**Returns**: Promise that resolves when the email is sent

## Database Schema

The package uses a `nonces` table in your Supabase database with the following structure:

- `id`: UUID primary key
- `client_token`: Hashed token sent to client
- `nonce`: Securely stored token hash
- `user_id`: Optional reference to auth.users
- `purpose`: Purpose identifier (e.g., 'password-reset')
- Status fields: `expires_at`, `created_at`, `used_at`, etc.
- Audit fields: `verification_attempts`, `last_verification_at`, etc.
- Extensibility fields: `metadata`, `scopes`

## Best Practices

1. **Use Specific Purposes**: Always use descriptive, specific purpose identifiers for your tokens.
2. **Short Expiration Times**: Set token expiration times to the minimum necessary for your use case.
3. **Handle Verification Failures**: Provide clear error messages when verification fails.
4. **Secure Your Tokens**: Never log or expose tokens in client-side code or URLs.

## Example Use Cases

- Email verification
- Two-factor authentication
- Account deletion confirmation
- Important action verification

Each use case should use a distinct purpose identifier. The purpose will
always need to match the one used when creating the token.

When you need to assign a specific data to a token, you can modify the
purpose with a unique identifier, such as `email-verification-12345`.

-----------------
FILE PATH: kits\next-supabase-turbo\api\team-account-api.mdoc

---
status: "published"
label: "Team Account API"
order: 1
title: "Team Account API | Next.js Supabase SaaS Kit"
description: "A quick introduction to the Team Account API in the Next.js Supabase SaaS Kit"
---

You can use the Team Account API for retrieving information about the team account.

## Using the Team Account API

To use the Team Account API, you need to import the `createTeamAccountsApi` function from `@kit/team-account/api`.

We need to pass a valid `SupabaseClient` to the function - so we can interact with the database from the server.

```tsx
import { createTeamAccountsApi } from '@kit/team-accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

async function ServerComponent() {
  const client = getSupabaseServerClient();
  const api = createTeamAccountsApi(client);

  // use api
}
```

If you're in a Server Action context, you'd use:

```tsx
'use server';

import { createTeamAccountsApi } from '@kit/team-accounts/api';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function myServerAction() {
  const client = getSupabaseServerClient();
  const api = createTeamAccountsApi(client);

  // use api
}
```

## Methods

The Account API provides the following methods:

### Get the team account by ID

Retrieves the team account by ID using the `getTeamAccountById` method.

You can also use this method to check if the user is already in the account.

```tsx
const api = createTeamAccountsApi(client);
const account = await api.getTeamAccountById('account-id');
```

### Get the team account subscription

Get the subscription data for the account using the `getSubscription` method.

```tsx
const api = createTeamAccountsApi(client);
const subscription = await api.getSubscription('account-id');
```

### Get the team account order

Get the orders data for the given account using the `getOrder` method.

```tsx
const api = createTeamAccountsApi(client);
const order = await api.getOrder('account-id');
```

### Get the account workspace data

Get the account workspace data.

```tsx
const api = createTeamAccountsApi(client);
const workspace = await api.getAccountWorkspace('account-slug');
```

This method is already called in the account layout and is unlikely to be used in other contexts. This is used to hydrate the workspace data in the context.

Since it's already loaded, you can use the data from the context.

### Checking a user's permission within an account

Check if the user has permission to perform a specific action within the account using the `hasPermission` method.

```tsx
const api = createTeamAccountsApi(client);

const hasPermission = await api.hasPermission({
  accountId: 'account-id',
  userId: 'user-id',
  permission: 'billing.manage',
});
```

### Getting the members count in the account

Get the number of members in the account using the `getMembersCount` method.

```tsx
const api = createTeamAccountsApi(client);
const membersCount = await api.getMembersCount('account-id');
```

### Get the billing customer ID

Get the billing customer ID for the given account using the `getCustomerId` method.

```tsx
const api = createTeamAccountsApi(client);
const customerId = await api.getCustomerId('account-id');
```

### Retrieve an invitation

Get the invitation data from the invite token.

```tsx
const api = createTeamAccountsApi(client);
const invitation = await api.getInvitation(adminClient, 'invite-token');
```

This method is used to get the invitation data from the invite token. It's used when the user is not yet part of the account and needs to be invited. The `adminClient` is used to read the pending membership. The method returns the invitation data if it exists, otherwise `null`.


