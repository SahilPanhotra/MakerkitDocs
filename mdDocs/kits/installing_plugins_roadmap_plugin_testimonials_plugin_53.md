-----------------
FILE PATH: kits/next-supabase-turbo/plugins/installing-plugins.mdoc

---
status: "published"
title: 'Installing Plugins in the Next.js Supabase SaaS Starter kit'
label: 'Installing Plugins'
order: 0
description: 'Learn how to install plugins in the Next.js Supabase SaaS Starter kit.'
---


Plugins are placed into a separate repository that mirrors the original repository structure. This allows us to build the plugins using the same files and structure as the main repository.0

You may wonder why we don't include the plugins in the main repository. The reason is that plugins are optional and may not be needed by all users. By keeping them separate, we can keep the main repository clean and focused on the core functionality. Instead you can install the plugins you need when you need them.

## Installing Plugins

To install a plugin, you can use the Makerkit CLI:

```bash
npx @makerkit/cli@latest plugins install
```

This command will prompt you to select a plugin to install. Once selected, the plugin will be installed in your project.

## How Makerkit installs plugins in your project

Plugins use `git subtree` to pull in the plugin repository into the `plugins` directory of your project. This allows you to keep the plugin up-to-date by pulling in changes from the plugin repository.

If you don't want to use git subtree, you can also manually clone a copy of the plugin repository, and then manually moving the folder from `packages/plugins` into your own repository.

-----------------
FILE PATH: kits/next-supabase-turbo/plugins/roadmap-plugin.mdoc

---
status: "published"
title: 'Roadmap Plugin in the Next.js Supabase SaaS Starter kit'
label: 'Roadmap Plugin'
order: 2
description: 'Learn how to install the Roadmap plugin in the Next.js Supabase SaaS Starter kit.'
---


This plugin allows you to create a roadmap for your project and display it on your website.

Your users can see what features are planned, in progress, and completed and suggest new features or comment on existing ones.

## Functionality

The plugin provides the following functionality:

1. Display the feature requests on the website.
2. Allow users to suggest new features.
3. Allow users to comment on existing features.
4. Display the Feature Requests in the Admin panel.
5. Allow Admins to manage the Feature Requests, update their status, and delete them.
6. Allow Admins to manage the comments on the Feature Requests.

## Installation

To install the plugin, run the following command:

```bash
npx @makerkit/cli plugins install
```

Since this plugin depends on the Kanban plugin, you need to install both. Please select the `kanban` plugin from the list of available plugins.

Then, please select the `roadmap` plugin from the list of available plugins.

### Add the translations

Add the following to your `apps/web/public/locales/en/roadmap.json` file:

 ```json {% title="apps/web/locales/en.json" %}
{
  "loadingComments": "Loading comments...",
  "noComments": "There are no comments on this feature request yet. Be the first to comment!",
  "comments": "Comments ({{count}})",
  "writtenOn": "Written on {{date}}",
  "loadMoreComments": "Load More Comments",
  "pleaseSignInComment": "Please sign in to leave a comment",
  "pleaseSignIn": "Please sign in to submit a post",
  "pleaseSignInDescription": "You must be signed in to submit a post to the community",
  "submitComment": "Submit Comment",
  "planned": "Planned",
  "inProgress": "In Progress",
  "completed": "Completed",
  "closed": "Closed",
  "underReview": "Under Review",
  "submitPost": "Submit your Post",
  "fillOutForm": "Fill out the form below to submit your post to the community",
  "type": "Type",
  "feature": "Feature",
  "bug": "Bug",
  "question": "Question",
  "task": "Task",
  "title": "Title",
  "titleDescription": "A concise title will help others understand your suggestion",
  "description": "Description",
  "descriptionDescription": "Provide as much detail as possible to help others understand your suggestion or issue",
  "submit": "Submit",
  "cancel": "Cancel",
  "votes": "{{votes}} votes",
  "improvement": "Improvement",
  "noIssues": "No feature requests found. Be the first to suggest a feature!",
  "noIssuesWithFilters": "No issues found with the current filters.",
  "clearFilters": "Clear Filters",
  "all": "All",
  "featureRequests": "Feature Requests",
  "bugReports": "Bug Reports",
  "recentlyCreated": "Recently Created",
  "mostVotes": "Most Votes",
  "allStatuses": "All Statuses",
  "issues": "Issues",
  "roadmap": "Roadmap",
  "heroPill": "Propose a feature, report a bug, or share your ideas",
  "subheading": "A place to share your ideas and collaborate on features."
}
```

Please adjust the translations to your needs.

Now, add the namespace to the loaded translations in your `apps/web/lib/i18n/i18n.settings.ts` file:

 ```tsx {% title="apps/web/lib/i18n/i18n.settings.ts" %}
export const defaultI18nNamespaces = [
  // Add the 'roadmap' namespace
  'roadmap',
];
```

### Migration

Create a new migration file by running the following command:

```bash
pnpm --filter web supabase migration new roadmap
```

And place the content you can see at `packages/plugins/roadmap/migrations/migration.sql` into the newly created file.

Then run the migration:

```bash
pnpm run supabase:web:reset
```

And update the types:

```bash
pnpm run supabase:web:typegen
```

Once the plugin is added to your repository, please install it as a dependency in your application in the package.json file of `apps/web`:

```json
{
    "dependencies": {
      "@kit/roadmap": "workspace:*"
    }
}
```

Now run the following command to install the plugin:

```bash
pnpm i
```

## Displaying the Roadmap and Feature Requests

To display the roadmap and feature requests on your website, add the following code to the `apps/web/app/(marketing)/roadmap/page.tsx` file:

```tsx
import { RoadmapPage } from "@kit/roadmap/server";

export default RoadmapPage;
```

Let's now add the comments GET route at `apps/web/app/(marketing)/roadmap/comments/route.ts`:

```tsx
import { createFetchCommentsRouteHandler } from '@kit/roadmap/route-handler';

export const GET = createFetchCommentsRouteHandler;
```

## Admin Pages

To add the Admin pages from where you can manage the roadmap and feature requests, add the following code to the `apps/web/app/admin/roadmap/page.tsx` file:

```tsx
import { AdminGuard } from '@kit/admin/components/admin-guard';
import { AdminIssuesPage } from '@kit/roadmap/admin';

export default AdminGuard(AdminIssuesPage);
```

And now, we add the feature request's detail page at `apps/web/app/admin/roadmap/[id]/page.tsx`:

```tsx
import { AdminGuard } from '@kit/admin/components/admin-guard';
import { AdminIssueDetailPage } from '@kit/roadmap/admin';

export default AdminGuard(AdminIssueDetailPage);
```

Add the sidebar item as well at `apps/web/app/admin/_components/admin-sidebar.tsx`:

```tsx
<SidebarItem
    path={'/admin/roadmap'}
    Icon={<FolderKanbanIcon className={'h-4'} />}
>
    Roadmap
</SidebarItem>
```

Import the `FolderKanbanIcon` icon from the existing `lucide-react` package.

You can now run the application and navigate to the Admin panel to manage the roadmap and feature requests.

### Tailwind CSS Styles

If using TailwindCSS v3, please update the Tailwind CSS styles to include the new folder:

```js { % title="tooling/tailwind/index.ts" %}
export default {
  darkMode: ['class'],
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/roadmap/src/**/*.tsx'  // <-- add this line
    '!**/node_modules',
  ],
  // ...
};
```

The above is not required if you are using TailwindCSS v4.

-----------------
FILE PATH: kits/next-supabase-turbo/plugins/testimonials-plugin.mdoc

---
status: "published"
title: 'Testimonials Plugin in the Next.js Supabase SaaS Starter kit'
label: 'Testimonials Plugin'
order: 3
description: 'Learn how to install the Testimonials plugin in the Next.js Supabase SaaS Starter kit.'
---

This plugin allows Makerkit users to easily collect and manage testimonials from their customers. It integrates seamlessly with the existing Makerkit structure and provides both backend and frontend components.

## Features

1. Testimonial submission form and manual entry
2. Admin panel for managing testimonials
3. API endpoints for CRUD operations
4. Widgets components for showing testimonials on the website

## Installation

To install the plugin, run the following command:

```bash
npx @makerkit/cli plugins install
```

Please select the `testimonial` plugin from the list of available plugins.

### Add the translations

Add the following to your `apps/web/public/locales/en/testimonials.json` file:

 ```json {% title="apps/web/locales/en.json" %}
{
  "welcomeMessage": "We'd love to hear your feedback!",
  "welcomeMessageDescription": "Your opinion helps us improve our service.",
  "textButtonText": "Write a review",
  "videoButtonText": "Record a video review",
  "backButtonText": "Choose a different review type",
  "customerName": "Your Name",
  "testimonial": "Testimonial",
  "rating": "Rating",
  "submitting": "Submitting...",
  "submitTestimonial": "Submit Testimonial",
  "errorTitle": "Sorry, something went wrong",
  "errorDescription": "Apologies, we were unable to submit your video review. Please try again later.",
  "customerNameDescription": "Your name will be displayed with your video review",
  "recording": "Recording... {{timer}}",
  "startRecording": "Start Recording",
  "stopRecording": "Stop Recording",
  "discardAndRetry": "Discard and retry",
  "successTitle": "Thank you!",
  "successDescription": "Your feedback helps us improve our services. We appreciate your time!",
  "wallOfLove": "Wall of Love",
  "videoTestimonialBy": "Video testimonial by {{customerName}}",
  "clickToPlay": "Click to play video testimonial",
  "videoTagNotSupported": "Your browser does not support the video tag.",
  "moreTestimonials": "{{count}} more testimonials"
}
```

Please adjust the translations to your needs.

Now, add the namespace to the loaded translations in your `apps/web/lib/i18n/i18n.settings.ts` file:

 ```tsx {% title="apps/web/lib/i18n/i18n.settings.ts" %}
export const defaultI18nNamespaces = [
  // Add the 'roadmap' namespace
  'testimonials',
];
```

### Migration

Create a new migration file by running the following command:

```bash
pnpm --filter web supabase migration new testimonials
```

And place the content you can see at `packages/plugins/testimonial/migrations/migration.sql` into the newly created file.

Then run the migration:

```bash
pnpm run supabase:web:reset
```

And update the types:

```bash
pnpm run supabase:web:typegen
```

Once the plugin is added to your repository, please install it as a dependency in your application in the package.json file of `apps/web`:

```json
{
    "dependencies": {
      "@kit/testimonial": "workspace:*"
    }
}
```

Now run the following command to install the plugin:

```bash
pnpm i
```

You can now add the Admin pages from where you can manage the testimonials. To do so, add the following code to the `apps/web/app/admin/testimonials/page.tsx` file:

```tsx
import { TestimonialsPage } from '@kit/testimonial/admin';

export default TestimonialsPage;
```

And now the testimonial's page at  `apps/web/app/admin/testimonials/[id]/page.tsx`:

```tsx
import { TestimonialPage } from '@kit/testimonial/admin';

export default TestimonialPage;
```

Add the sidebar item as well at `apps/web/app/admin/_components/admin-sidebar.tsx`:

```tsx
<SidebarItem
    path={'/admin/testimonials'}
    Icon={<StarIcon className={'h-4'} />}
>
    Testimonials
</SidebarItem>
```

You can now run the application and navigate to the Admin panel to manage the testimonials.

## Displaying the Testimonial Form

To display the testimonial form on your website, you can import the form component from the plugin and use it in your page.

Create a new component, and import the form:

```tsx
'use client';

import { useState } from 'react';

import {
  TestimonialContainer,
  TestimonialForm,
  TestimonialSuccessMessage,
  VideoTestimonialForm,
} from '@kit/testimonial/client';

export function Testimonial() {
  const [success, setSuccess] = useState(false);
  const onSuccess = () => setSuccess(true);

  if (success) {
    return <SuccessMessage />;
  }

  return (
    <TestimonialContainer
      className={
        'w-full max-w-md rounded-lg border bg-background p-8 shadow-xl'
      }
      welcomeMessage={<WelcomeMessage />}
      enableTextReview={true}
      enableVideoReview={true}
      textReviewComponent={<TestimonialForm onSuccess={onSuccess} />}
      videoReviewComponent={<VideoTestimonialForm onSuccess={onSuccess} />}
      textButtonText="Write your thoughts"
      videoButtonText="Share a video message"
      backButtonText="Switch review method"
    />
  );
}

function SuccessMessage() {
  return (
    <div
      className={
        'w-full max-w-md rounded-lg border bg-background p-8 shadow-xl'
      }
    >
      <div className="flex flex-col items-center space-y-4 text-center">
        <div className="space-y-1">
          <h1 className="text-2xl font-semibold">
            Thank you for your feedback!
          </h1>

          <p className="text-muted-foreground">
            Your review has been submitted successfully.
          </p>
        </div>

        <div>
          <TestimonialSuccessMessage />
        </div>
      </div>
    </div>
  );
}

function WelcomeMessage() {
  return (
    <div className="flex flex-col items-center space-y-1 text-center">
      <h1 className="text-2xl font-semibold">
        We&apos;d love to hear your feedback!
      </h1>

      <p className="text-muted-foreground">
        Your opinion helps us improve our service.
      </p>
    </div>
  );
}
```

Please customize the components as needed to fit your website's design.

### Tailwind CSS Styles

If using TailwindCSS v3, please update the Tailwind CSS styles to include the new folder:

```js { % title="tooling/tailwind/index.ts" %}
export default {
  darkMode: ['class'],
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/testimonials/src/**/*.tsx'  // <-- add this line
    '!**/node_modules',
  ],
  // ...
};
```

The above is not required if you are using TailwindCSS v4.

## API Endpoints

Please add the GET and POST endpoints to fetch the testimonials at `apps/web/app/api/testimonials/route.ts`:

```ts
import {
  createTestimonialsRouteHandler,
  createVideoTestimonialRouteHandler,
} from '@kit/testimonial/server';

export const GET = createTestimonialsRouteHandler;
export const POST = createVideoTestimonialRouteHandler;
```

## Widgets

To display the testimonials on your website, you can use the following widget:

```tsx
import { TestimonialWallWidget } from '@kit/testimonial/widgets';

export default function TestimonialWidgetPage() {
    return (
        <div className={'flex h-full w-screen flex-1 flex-col items-center py-16'}>
            <TestimonialWallWidget />
        </div>
    );
}
```

Done! You now have a fully functional Testimonial Collection plugin integrated with your Makerkit application.


-----------------
FILE PATH: kits/next-supabase-turbo/plugins/waitlist-plugin.mdoc

---
status: "published"
title: 'Add a Waitlist to the Next.js Supabase SaaS Starter kit'
label: 'Waitlist'
order: 1
description: 'Add a waitlist to your Next.js Supabase SaaS Starter kit to collect emails from interested users.'
---

In this guide, you will learn how to add a waitlist to your Next.js Supabase SaaS Starter kit to collect emails from interested users. This feature is useful for building an audience before launching your product.

This plugin allows you to create a waitlist for your app. Users can sign up for the waitlist and receive an email when the app is ready.

### How it works

1. You disable sign up in your app from Supabase. This prevents any user from using the public API to sign up.
2. We create a new table in Supabase called `waitlist`. Users will sign up for the waitlist and their email will be stored in this table.
3. When you want to enable a sign up for a user, mark users as `approved` in the `waitlist` table.
4. The database trigger will create a new user in the `auth.users` table and send an email to the user with a link to set their password.
5. The user can now sign in to the app and update their password.
6. User gets removed from the waitlist as soon as the email is sent.

### Installation

#### Get the plugin using the CLI

Please run the following command in your terminal:

```bash
npx @makerkit/cli@latest plugins install waitlist
```

After completed, the CLI will install the plugin at `packages/plugins/waitlist`.

#### Add the plugin to your main app

Now, install the plugin from your main app by adding the following to your `package.json` file:

 ```json {% title="apps/web/package.json" %}
{
  "dependencies": {
    "@kit/waitlist": "workspace:*"
  }
}
```

And then run `pnpm install` to install the plugin.

#### Add migrations to your app

From the folder `packages/plugins/waitlist/migrations`, copy the migrations to your app's migrations folder at `apps/web/supabase/migrations`.

This will add the `waitlist` table to your database.

Now, re-run the migrations:

```bash
pnpm run supabase:web:reset
pnpm run supabase:web:typegen
```

You can now use the waitlist plugin in your app.

#### Replace the sign up form

Replace your sign up form with the waitlist form in your app (retain the existing imports):

```tsx
import { WaitlistSignupForm } from '@kit/waitlist/client';

function SignUpPage({ searchParams }: Props) {
  const inviteToken = searchParams.invite_token;

  const signInPath =
    pathsConfig.auth.signIn +
    (inviteToken ? `?invite_token=${inviteToken}` : '');

  return (
    <>
      <Heading level={4}>
        <Trans i18nKey={'auth:signUpHeading'} />
      </Heading>

      <WaitlistSignupForm inviteToken={inviteToken} />

      <div className={'justify-centers flex'}>
        <Button asChild variant={'link'} size={'sm'}>
          <Link href={signInPath}>
            <Trans i18nKey={'auth:alreadyHaveAnAccount'} />
          </Link>
        </Button>
      </div>
    </>
  );
}

export default withI18n(SignUpPage);
```

#### Adding the Database Webhook to listen for new signups

Let's extend the DB handler at `apps/web/app/api/db/webhook/route.ts`. This handler will listen for new signups and send an email to the user:

```tsx
import { getDatabaseWebhookHandlerService } from '@kit/database-webhooks';
import { getServerMonitoringService } from '@kit/monitoring/server';
import { enhanceRouteHandler } from '@kit/next/routes';

import appConfig from '~/config/app.config';
import pathsConfig from '~/config/paths.config';

/**
 * @name POST
 * @description POST handler for the webhook route that handles the webhook event
 */
export const POST = enhanceRouteHandler(
  async ({ request }) => {
    const service = getDatabaseWebhookHandlerService();

    try {
      const signature = request.headers.get('X-Supabase-Event-Signature');

      if (!signature) {
        return new Response('Missing signature', { status: 400 });
      }

      const body = await request.clone().json();

      // handle the webhook event
      await service.handleWebhook({
        body,
        signature,
        async handleEvent(payload) {
          if (payload.table === 'waitlist' && payload.record.approved) {
            const { handleApprovedUserChange } = await import(
              '@kit/waitlist/server'
            );

            const inviteToken = payload.record.invite_token;
            
            const redirectToUrl = new URL(
              pathsConfig.auth.passwordUpdate,
              appConfig.url,
            );

            if (inviteToken) {
              const next = encodeURI(
                pathsConfig.app.joinTeam + '?invite_token=' + inviteToken,
              );

              redirectToUrl.searchParams.append('callback', next);
            }

            const redirectTo = redirectToUrl.toString();

            await handleApprovedUserChange({
              email: payload.record.email,
              redirectTo,
            });
          }
        },
      });

      // return a successful response
      return new Response(null, { status: 200 });
    } catch (error) {
      const service = await getServerMonitoringService();

      await service.ready();
      await service.captureException(error as Error);

      // return an error response
      return new Response(null, { status: 500 });
    }
  },
  {
    auth: false,
  },
);
```

#### Adding the Trigger to the Database

We need to add a trigger to the `waitlist` table to listen for updates and send a webhook to the app when a user is approved.

During development, you can simply add the webhook to your seed file `apps/web/supabase/seed.sql`:

```sql
create trigger "waitlist_approved_update" after update
on "public"."waitlist"
for each row
when (new.approved = true)
execute function "supabase_functions"."http_request"(
  'http://host.docker.internal:3000/api/db/webhook',
  'POST',
  '{"Content-Type":"application/json", "X-Supabase-Event-Signature":"WEBHOOKSECRET"}',
  '{}',
  '5000'
);
```

The above creates a trigger that listens for updates to the `waitlist` table and sends a POST request to the webhook route.

**Note**: You need to add this trigger to your production database as well. You will replace your `WEBHOOKSECRET` with the secret you set in your `.env` file and the `host.docker.internal:3000` with your production URL.
Just like you did for the other existing triggers.

#### Approving users

Simply update the `approved` column in the `waitlist` table to `true` to approve a user. You can do so from the Supabase dashboard or by running a query.

Alternatively, run an update based on the created_at timestamp:

```sql
update public.waitlist
set approved = true
where created_at < '2024-07-01';
```

#### Email Templates and URL Configuration

Please make sure to [edit the email template](https://makerkit.dev/docs/next-supabase-turbo/authentication-emails) in your Supabase account.
The default email in Supabase does not support PKCE and therefore does not work. By updating it - we replace the existing strategy with the token-based strategy - which the `confirm` route in Makerkit can support.

Additionally, [please add the following URL to your Supabase Redirect URLS allow list](https://supabase.com/docs/guides/auth/redirect-urls):

```
<your-url>/password-reset
```

This will allow Supabase to redirect users to your app to set their password after they click the email link.

If you don't do this - the email links will not work.

#### Translations

Please add the following translations to your `auth.json` translation:

```json
{
  "waitlist": {
    "heading": "Join the Waitlist for Early Access",
    "submitButton": "Join Waitlist",
    "error": "Ouch, we couldn't add you to the waitlist. Please try again.",
    "errorDescription": "We couldn't add you to the waitlist. If you have already signed up, you are already on the waitlist.",
    "success": "You're on the waitlist!",
    "successDescription": "We'll let you know when you can sign up."
  }
}
```

#### Disable oAuth

If you are using any oAuth providers, please disable them in the [Makerkit Auth configuration](../configuration/authentication-configuration#third-party-providers). Since sign-ups are disabled, users will hit errors.

### Tailwind CSS (v3) Styles

If using TailwindCSS v3, please update the Tailwind CSS styles to include the new folder:

```js { % title="tooling/tailwind/index.ts" %}
export default {
  darkMode: ['class'],
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/waitlist/src/**/*.tsx'  // <-- add this line
    '!**/node_modules',
  ],
  // ...
};
```

The above is not required if you are using TailwindCSS v4.

---

Easy peasy! Now you have a waitlist for your app.

-----------------
FILE PATH: kits/next-supabase-turbo/plugins.mdoc

---
status: "published"
title: "Plugins in Next.js Supabase Turbo"
label: "Plugins"
order: 16
description: "Plugins in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Extend your application's functionality with our plugin system:

1. [**Getting Started with Plugins**](/docs/next-supabase-turbo/installing-plugins)
   Learn how to install and manage plugins in your application.
2. Available Plugins
   - [Chatbot Plugin](/docs/next-supabase-turbo/plugins/chatbot-plugin)
   - [Feedback Plugin](/docs/next-supabase-turbo/plugins/feedback-plugin)
   - [Text Editor Plugin](/docs/next-supabase-turbo/plugins/text-editor-plugin)
   - [Waitlist Plugin](/docs/next-supabase-turbo/plugins/waitlist-plugin)
   - [Testimonials Plugin](/docs/next-supabase-turbo/plugins/testimonials-plugin)
   - [Roadmap Plugin](/docs/next-supabase-turbo/plugins/roadmap-plugin)


-----------------
FILE PATH: kits/next-supabase-turbo/recipes/drizzle-supabase.mdoc

---
status: "published"
title: "Using Drizzle as a client for interacting with Supabase"
label: "Drizzle"
order: 6
description: "Learn how to add Drizzle to your Next.js Supabase Turbo project and use it to interact with Supabase"
---

By default, the kit interacts with Supabase using the plain Supabase client. 

This guide will walk you through adding [Drizzle ORM](https://orm.drizzle.team/) support to your Makerkit project. Drizzle is a TypeScript ORM that provides type-safe database queries with great developer experience.

This guide is an adaptation of the Drizzle guide for Next.js applications. [The original guide can be found here](https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase).

Please refer to the [Drizzle documentation](https://orm.drizzle.team/docs/overview) for more advanced usage and configuration options.

## Prerequisites
- A working Makerkit project
- Basic understanding of TypeScript and SQL
- Supabase project set up

## Step 1: Install Dependencies

First, let's add the required dependencies to the `@kit/supabase` package:

```bash
pnpm --filter "@kit/supabase" add jwt-decode postgres
pnpm --filter "@kit/supabase" add -D drizzle-orm drizzle-kit
```

We added the required dependencies to the `@kit/supabase` package.

## Step 2: Create Drizzle Configuration

Create a new file `packages/supabase/drizzle.config.js`:

```javascript {% title="packages/supabase/drizzle.config.js" %}
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/schema.ts',
  out: './src/drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    // This will use local development DB by default
    url: process.env.DATABASE_URL ?? 'postgresql://postgres:postgres@127.0.0.1:54322/postgres'
  },
  schemaFilter: ['public'],
  verbose: true,
  strict: true,
});
```

We will use the Drizzle Kit CLI to generate a schema out of the Supabase database. This will in turn create the `schema.ts` file in the `out` directory.

The `schema.ts` file will contain the TypeScript types matching your database schema that Drizzle uses to infer the TS types for your database queries.

## Step 3: Add Scripts to package.json

Update `packages/supabase/package.json`:

```json {% title="packages/supabase/package.json" %}
{
  "scripts": {
    // ... existing scripts
    "drizzle": "drizzle-kit",
    "pull": "drizzle-kit pull --config drizzle.config.js"
  },
  "exports": {
    // Add these new exports
    "./drizzle-client": "./src/clients/drizzle-client.ts",
    "./drizzle-schema": "./src/drizzle/schema.ts"
  }
}
```

These commands will help us pull the schema from the Supabase database and generate the `schema.ts` file.

## Step 4: Create Drizzle Client

Create a new file `packages/supabase/src/clients/drizzle-client.ts`:

```typescript {% title="packages/supabase/src/clients/drizzle-client.ts" %}
import 'server-only';

import { DrizzleConfig, sql } from 'drizzle-orm';
import { drizzle } from 'drizzle-orm/postgres-js';
import { JwtPayload, jwtDecode } from 'jwt-decode';
import postgres from 'postgres';
import { z } from 'zod';

import * as schema from '../drizzle/schema';

// Config validation
const SUPABASE_DATABASE_URL = z
  .string({
    description: `The URL of the Supabase database. Please provide the variable SUPABASE_DATABASE_URL.`,
    required_error: 'The environment variable SUPABASE_DATABASE_URL is required',
  })
  .url()
  .parse(process.env.SUPABASE_DATABASE_URL!);

const config = {
  casing: 'snake_case',
  schema,
} satisfies DrizzleConfig<typeof schema>;

// Admin client bypasses RLS
const adminClient = drizzle({
  client: postgres(SUPABASE_DATABASE_URL, { prepare: false }),
  ...config,
});

// RLS protected client
const rlsClient = drizzle({
  client: postgres(SUPABASE_DATABASE_URL, { prepare: false }),
  ...config,
});

export function getDrizzleSupabaseAdminClient() {
  return adminClient;
}

export async function getDrizzleSupabaseClient() {
  const client = getSupabaseServerClient();
  const { data } = await client.auth.getSession();
  const accessToken = data.session?.access_token ?? '';
  const token = decode(accessToken);

  const runTransaction = ((transaction, config) => {
    return rlsClient.transaction(async (tx) => {
      try {
        // Set up Supabase auth context
        await tx.execute(sql`
          select set_config('request.jwt.claims', '${sql.raw(
            JSON.stringify(token),
          )}', TRUE);
          select set_config('request.jwt.claim.sub', '${sql.raw(
            token.sub ?? '',
          )}', TRUE);
          set local role ${sql.raw(token.role ?? 'anon')};
        `);

        return await transaction(tx);
      } finally {
        // Clean up
        await tx.execute(sql`
          select set_config('request.jwt.claims', NULL, TRUE);
          select set_config('request.jwt.claim.sub', NULL, TRUE);
          reset role;
        `);
      }
    }, config);
  }) as typeof rlsClient.transaction;

  return {
    runTransaction,
  };
}

function decode(accessToken: string) {
  try {
    return jwtDecode<JwtPayload & { role: string }>(accessToken);
  } catch {
    return { role: 'anon' } as JwtPayload & { role: string };
  }
}
```

## Step 5: Generate the Schema

Now we'll generate the Drizzle schema from your existing Supabase database:

```bash
pnpm --filter "@kit/supabase" drizzle pull
```

This will create a `schema.ts` file in your `out` directory with the TypeScript types matching your database schema.

You should see the following output:

```
Pulling from ['public'] list of schemas

Using 'postgres' driver for database querying
[✓] 14  tables fetched
[✓] 104 columns fetched
[✓] 9   enums fetched
[✓] 18  indexes fetched
[✓] 23  foreign keys fetched
[✓] 28  policies fetched
[✓] 3   check constraints fetched
[✓] 2   views fetched

[✓] Your SQL migration file ➜ src/drizzle/0000_easy_black_panther.sql 🚀
[✓] Your schema file is ready ➜ src/drizzle/schema.ts 🚀
[✓] Your relations file is ready ➜ src/drizzle/relations.ts 🚀
```

### Adding the auth schema to the schema

After generating the schema, please add the following code at the top of your `schema.ts` file:

1. Import the `pgSchema` function from Drizzle
2. Add the `auth` schema to the schema, which we need to reference the `auth.users` table in Supabase. This is required because we only pull the public schema by default, however some tables reference the `auth` schema.

```ts {% title="packages/supabase/src/drizzle/schema.ts" %}
const authSchema = pgSchema('auth');

const usersInAuth = authSchema.table('users', {
  id: uuid('id').primaryKey(),
});
```

### Disable linting on schema.ts

Since this is a generated file, we want to disable linting on it. To do so, add this line to the top of the file:

```ts {% title="packages/supabase/src/drizzle/schema.ts" %}
/* eslint-disable */
```

## Step 6: Using the Drizzle Client

Here's an example of how to use the Drizzle client in a Next.js page:

```typescript
import { use } from 'react';
import { getDrizzleSupabaseClient } from '@kit/supabase/drizzle-client';
import { accounts } from '@kit/supabase/drizzle-schema';

function MyComponent() {
  const client = use(getDrizzleSupabaseClient());

  // Example query using transactions
  const data = use(client.runTransaction((tx) => {
    return tx.select().from(accounts);
  }));

  // Rest of your component code
}
```

Alternatively, in a Server Action:

```typescript
'use server';

import { getDrizzleSupabaseClient } from '@kit/supabase/drizzle-client';
import { accounts } from '@kit/supabase/drizzle-schema';

export const getAccountsAction = enhanceAction(async (data) => {
  const client = await getDrizzleSupabaseClient();

  // Fetch accounts
  const data = await client.runTransaction((tx) => {
    return tx.select().from(accounts);
  });

  // Return the data
  return data;
},
{
  auth: true
});
```

### Using the Admin Client

If you need elevated permissions, you can use the `getDrizzleSupabaseAdminClient()` function:

```typescript
import { use } from 'react';
import { getDrizzleSupabaseAdminClient } from '@kit/supabase/drizzle-client';
import { accounts } from '@kit/supabase/drizzle-schema';

function MyComponent() {
  const client = getDrizzleSupabaseAdminClient();

  // Example query using transactions
  const data = await client.select().from(accounts);

  // Rest of your component code
}
```

## Important Notes

1. **RLS Support**: The Drizzle client respects Supabase Row Level Security (RLS) by passing the user's JWT token and role in transactions.
2. **Client Types**: There are two client types:
   - `getDrizzleSupabaseAdminClient()` - bypasses RLS (use with caution)
   - `getDrizzleSupabaseClient()` - respects RLS (use for regular user operations)
3. **Environment Variables**: Make sure to set `SUPABASE_DATABASE_URL` in your environment variables:
   ```bash
   SUPABASE_DATABASE_URL=postgres://user:pass@host:port/database
   ```

You can only use the Drizzle client in a Server environment - therefore in Server Components, Server Actions and API Routes. It will fail if you bundle it within a client-side component.

### Keep the SUPABASE_DATABASE_URL variable confidential

The variable `SUPABASE_DATABASE_URL` is required for the Drizzle client to work correctly. You will find this in the Supabase dashboard under the project settings.

{% alert type="warning" title="Do not commit the SUPABASE_DATABASE_URL variable to the repository" %}
Please keep the variable `SUPABASE_DATABASE_URL` confidential. It contains the DB credentials and should not be committed to the repository. Therefore, only add it to the `.env.development` file for development purposes containing the local DB credentials.
{% /alert %}

When in production, please only add the `SUPABASE_DATABASE_URL` variable using your hosting provider's environment variables.

## Next Steps

While the core kit will keep using the plain Supabase client, from here on you can use the Drizzle client in your application.

If you want to use the Drizzle utilities for migrations, you may need to place the `src/drizzle` folder in the root of your project and replace the path in the `drizzle.config.js` file pointing to the `apps/web` folder.data

However, please remember to keep up to date the `drizzle-schema` export in the `@kit/supabase` package, since you want to be able to use the Drizzle schema throughout the packages.

## Benefits of Using Drizzle

- Type-safe database queries
- Better developer experience with autocomplete
- SQL-like query builder
- Transaction support
- Automatic schema generation from your existing database

Please refer to the [Drizzle documentation](https://orm.drizzle.team/docs/overview) for more advanced usage and configuration options.


