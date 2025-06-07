-----------------
FILE PATH: kits\next-supabase-turbo\installation\migration-from-makerkit-v1.mdoc

---
status: "published"
title: "Guidelines for migrating from Makerkit v1 | Next.js Supabase SaaS Starter Kit"
label: "Migrating from v1"
order: 9
description: "Guidelines for migrating from Makerkit v1 to Next.js SaaS Boilerplate."
---

🚀 Welcome to your journey from Makerkit v1 to the Next.js SaaS Boilerplate! 

This guide is designed to help you understand the changes between the two versions and navigate your project migration to the new v2. Whether you're a seasoned developer or just starting out, we've got you covered!

Here's a roadmap of the steps you'll take:

1. **Bootstrap a new project**: Kickstart a new project using the Next.js SaaS Boilerplate v2 🎉
2. **Update Schema**: Tweak the Postgres schema foreign key references, and integrate your existing tables 🧩
3. **Move files from older app**: Transport your files to the new app structure 📦
4. **Update imports to existing files**: Refresh imports to align with the new file structure 🔄
5. **Update configuration**: Modify the configuration files to match the new schemas ⚙️

## 1. Bootstrap a new project

The Next.js SaaS Boilerplate is a fresh take on Makerkit v1. You'll need to create a new project using this innovative boilerplate. Follow the [installation guide](clone-repository) to get your new project up and running in no time!

## 2. Update Schema

The schema in the Next.js SaaS Boilerplate has evolved significantly from Makerkit v1. You'll need to update your Postgres schema to match the new one.

Previously, you'd have a foreign key set to the organization ID:

```sql
organization_id bigint not null references organizations(id) on delete cascade,
```

Now, you'll have a foreign key set to the account ID as a UUID:

```sql
account_id uuid not null references public.accounts(id) on delete cascade,
```

In v2, an account can be both a user or an organization. So, instead of referencing the organization ID as in v1, you'll now reference the account ID. 

## 3. Move files from older app

In v2, you have the flexibility to add these files to either the "user scope" (`/home`) or the "account scope" (`/home/[account]`). Choose the one that best suits your project needs.

## 4. Update imports to existing files

You'll need to update the imports to your existing files to match the new file structure. This applies to all the components and utilities that you imported from Makerkit. For instance, a button previously imported from `~/core/ui/Button`, will now be imported from `@kit/ui/button`.

## 5. Update configuration

Lastly, you'll need to update the configuration files to match the new schemas. The configuration is now split across various files at `apps/web/config`. Pay special attention to the billing schema, which is now more versatile (and a bit more complex).

Ready to get started? Let's dive in! 🏊‍♀️

-----------------
FILE PATH: kits\next-supabase-turbo\installation\navigating-codebase.mdoc

---
status: "published"

title: "Navigating the your Next.js Supabase Turbo Starter Kit codebase"
label: "Navigating the Codebase"
order: 7
description: "Learn how to navigate the codebase of the Next.js Supabase Turbo Starter Kit. Understand the project structure and how to update the codebase."
---


As mentioned before, this SaaS Starter Kit is modular, thanks to Turborepo - which makes it easy to manage multiple packages in a single repository.

## Project Structure

The main directories in the project are:
1. `apps` - the location of the Next.js app
2. `packages` - the location of the shared code and the API

### `apps` Directory

This is where the app (or the apps, should you add more) lives. The main app is a Next.js app, which is a React framework that allows you to build static and server-rendered web applications.

The Next.js app defines:

1. the configuration of the project
2. the routes of the application

### `packages` Directory

This is where the shared code and the API live. The shared code is a collection of functions that are used by both the Next.js app and the API. The API is a serverless function that is used to interact with the Supabase database.

The shared code defines:

1. shared libraries (Supabase, Mailers, CMS, Billing, etc.)
2. shared features (admin, accounts, account, auth, etc.)
3. UI components (buttons, forms, modals, etc.)

All apps can use and reuse the API exported from the `packages` directory. This makes it easy to have one, or many apps in the same codebase, sharing the same code.

## Diving into the Codebase

The codebase is structured in a way that makes it easy to navigate and update. Here are some of the key files and directories you should be aware of:

```
- apps
---
 web
---
-- app
---
-- components
---
-- config
---
-- lib
---
-- content
---
-- styles
---
-- supabase
```


### Diving into `apps/web`

The `apps/web` directory is where the Next.js app lives. Here are some of the key directories and files you should be aware of:

1. `app` - this is where the main app lives. This is where you define the routes of the application.
2. `components` - this is where you define the shared components of the application.
3. `config` - this is where you define the configuration of the application.
4. `lib` - this is where you define the shared libraries of the application.
5. `content` - this is where you define the content of the application (by default, uses Keystatic Markdoc files)
6. `styles` - this is where you define the styles of the application.
7. `supabase` - this is where you define the Supabase configuration, migrations and tests

### Diving into `app`

The `app` directory is where the routing of the application is defined. Here are some of the key files you should be aware of:

```
- app
---
 home
---
 (marketing)
---
 auth
---
 join
---
 admin
---
 update-password
---
 server-sitemap.xml
```

Let's break down the directories:

1. `home` - this is where the internal home of the application lives. This is where you define the routes when the user is logged in.
2. `(marketing)` - this is where the marketing pages of the application live. This is where you define the routes when the user is not logged in. It's a pathless route, so you don't see it in the URL.
3. `auth` - this is where the authentication pages of the application live. This is where you define the routes for the login, signup, and forgot password pages.
4. `join` - this is where the join pages of the application live. This route is the route where the user can join a team account following an invitation.
5. `admin` - this is where the admin pages of the application live. This is where you define the routes for the super admin pages.
6. `update-password` - this is where the update password pages of the application live. This is the route the user is redirected to after resetting their password.
7. `server-sitemap.xml` - this is where the sitemap of the application is defined.

### Diving into `home`

The `home` directory is where the internal dashboard of the application lives. Here are some of the key files you should be aware of:

```
---
 home
---
---
 (user)
---
---
 [account]
```

Let's break down the directories:

1. `(user)` - this is where the user pages of the application live. This is where you define the routes for the personal account pages.
2. `[account]` - this is where the account pages of the application live. This is where you define the routes for the team account pages.

The `home` path allows us to separate the marketing pages from the internal dashboard pages.

So the user home page would be `/home/user` and the account home page would be `/home/[account]`.






-----------------
FILE PATH: kits\next-supabase-turbo\installation\running-project.mdoc

---
status: "published"
title: "Running the Next.js Supabase Turbo project"
label: "Running the Project"
order: 4
description: "Learn how to run the Next.js Supabase Turbo project on your local machine."
---

To run the project, follow these steps to start the development server, Supabase, and Stripe (optional for billing system testing).

{% sequence title="Steps to run the project" description="Learn how to run the Next.js Supabase Turbo project on your local machine." %}

[Ensure Docker is up and running](#0.-ensure-docker-is-running)

[Start Supabase](#1.-start-supabase)

[Start the Next.js development server](#2.-start-the-development-server)

[Start Stripe (optional for billing system testing)](#3.-start-stripe-(optional))
{% /sequence %}

## 1. Start Supabase

### Ensure Docker is running

Before starting the Supabase container, ensure Docker is running. This is because the Supabase container is a Docker container, and we can't start it without Docker running.

{% alert type="warning" title="Have Docker running" %}
If you don't have Docker running, please start it now, otherwise the Supabase container will not start.
{% /alert %}

[Docker Desktop](https://www.docker.com/products/docker-desktop/) is the best way to get started with Docker. Alternatively, on MacOS, you can use [OrbStack](https://orbstack.dev/) as an alternative to Docker Desktop.

### Starting the Supabase Container
After ensuring Docker is running, start the Supabase container using the following command:

```bash
pnpm run supabase:web:start
```

This command initiates the local Supabase web server. This allows us to develop locally without having to deploy to Supabase.

When you're ready to deploy the project to production, follow the [checklist](../going-to-production/checklist) to ensure everything is properly configured.

Please read the [Supabase documentation](https://supabase.com/docs/guides/local-development/cli/getting-started#running-supabase-locally) for more information about starting Supabase locally.

{% alert type="warning" title="Give Docker enough resources" %}
If the command above fails, you may need to give Docker more resources. Please refer to your Docker software's documentation for instructions on how to do this.
{% /alert %}

## Start the Development Server

To start the web application development server, run:

```bash
# Start the development server
pnpm dev
```

This command launches:

1. **Web Application**: The web application is built using Next.js and serves as the main user interface for the application at port 3000
2. **Dev Tools**: The dev tools are built using Next.js and serve as a debugging tool for the web application at port 3010. For more information, please refer to the [Dev Tools documentation](/docs/next-supabase-turbo/dev-tools/environment-variables).

You will mostly be using the web application at port 3000.

### Quick Start Credentials

Makerkit seeds the DB with a few users by default. To login as a test user, you can use the following credentials:

- **Email**: `test@makerkit.dev`
- **Password**: `testingpassword`

To confirm email addresses, visit [Inbucket](http://localhost:54324). Supabase uses Inbucket/Mailpit to capture emails sent during the authentication process.

Bookmark the Inbucket/Mailpit URL, as you will need it quite often.

**Ensure it's working:** Please try to use the test user credentials to login. If these credentials are not working, please try re-seeding the DB with the following command:

```bash
pnpm run supabase:web:reset
```

#### Troubleshooting

If you're hitting issues with loading modules (such as `Module not found:
Can't resolve 'react-dom/client'`), you're likely running into Windows-related
issues with Node.js.

Please see the [the Troubleshooting section](../troubleshooting/troubleshooting-module-not-found) for more information.

#### Super Admin Credentials

To login as a super admin, you can use the following credentials:

```json
{
  "email": "super-admin@makerkit.dev",
  "password": "testingpassword"
}
```

Makerkit forces MFA (multi-factor authentication) for the Super Admin user. Therefore, to sign in with the Super Admin user, please use the following steps to pass MFA:

1. **TOTP**: Use the following [TOTP generator](https://totp.danhersam.com/) to generate a TOTP code.
2. **Secret Key**: Use the test secret key `NHOHJVGPO3R3LKVPRMNIYLCDMBHUM2SE` to generate a TOTP code.
3. **Verify**: Use the TOTP code and the secret key to verify the MFA code.
Make sure the TOTP code is not expired when you verify the MFA code.

## 3. Start Stripe (Optional)

To test the billing system, start Stripe by running:

```bash
pnpm run stripe:listen
```

This command routes webhooks to your local machine.

You now have the Next.js Supabase Turbo project running on your local machine. 🚀

## 4. Email Server for local development

To receive emails during local development, Supabase uses [Mailpit](http://localhost:54324). It's from here that you will be receving emails from both Supabase Auth and transactional emails during local development.

You can bookmark the Mailpit URL, as you will need it quite often.

## Deploying to Production

When you're ready to deploy the project to production, follow the [checklist](../going-to-production/checklist) to ensure everything is set up correctly.

**Note:** Using Supabase's hosted instance is similar to deploying to production. Therefore, you still need to follow the checklist to ensure everything is properly configured.


-----------------
FILE PATH: kits\next-supabase-turbo\installation\technical-details.mdoc

---
status: "published"
title: "Technical Details of the Next.js Supabase SaaS Boilerplate"
label: "Technical Details"
order: 2
description: "Makerkit uses Next.js App Router, Supabase, Shadcn UI, React Query, Zod, Lucide, Nodemailer, and more."
---

The kit is built as a [Turborepo](https://turbo.build) monorepo, and is broadly using the following technologies:

1. **Next.js App Router** (using Turbopack for faster builds)
2. **Supabase** for the database, authentication and storage
3. **Shadcn UI** for the UI components
4. **React Query** for client-side data fetching
5. **Zod** for data validation
6. **Lucide** for icons
7. **Nodemailer** for emails, or **Resend** via HTTP (for edge runtimes)

Makerkit's modular architecture allows you to easily add or remove features, and customize the application to your needs.

Additionally, the kit is designed to be easily deployable to Vercel, Cloudflare, or any other hosting provider. It also supports edge rendering so you can deploy the application to Cloudflare Workers and go FAST.

### Monorepo Structure

Below are the reusable packages that can be shared across multiple applications (or packages):

- **@kit/ui**: Shared UI components and styles (using Shadcn UI and some custom components)
- **@kit/shared**: Shared code and utilities
- **@kit/supabase**: Supabase package that defines the schema and logic for managing Supabase
- **@kit/i18n**: Internationalization package that defines utilities for managing translations
- **@kit/billing**: Billing package that defines the schema and logic for managing subscriptions
- **@kit/billing-gateway**: Billing gateway package that defines the schema and logic for managing payment gateways
- **@kit/email-templates**: Here we define the email templates using the react.email package.
- **@kit/mailers**: Mailer package that abstracts the email service provider (e.g., Resend, Cloudflare, SendGrid, Mailgun, etc.)
- **@kit/monitoring**: A unified monitoring package that defines the schema and logic for monitoring the application with third party services (e.g., Sentry, Baselime, etc.)
- **@kit/database-webhooks**: Database webhooks package that defines the actions following database changes (e.g., sending an email, updating a record, etc.)
- **@kit/cms**: CMS package that defines the schema and logic for managing content
- **@kit/next**: Next.js specific utilities

And features that can be added to the application:

- **@kit/auth**: Authentication package (using Supabase)
- **@kit/accounts**: Package that defines components and logic for managing personal accounts
- **@kit/team-accounts**: Package that defines components and logic for managing team
- **@kit/admin**: Admin package that defines the schema and logic for managing users, subscriptions, and more.
- **@kit/notifications**: Notifications package that defines the schema and logic for managing notifications

And billing packages that can be added to the application:

- **@kit/stripe**: Stripe package that defines the schema and logic for managing Stripe. This is used by the @kit/billing-gateway package and abstracts the Stripe API.
- **@kit/lemon-squeezy**: Lemon Squeezy package that defines the schema and logic for managing Lemon Squeezy. This is used by the @kit/billing-gateway package and abstracts the Lemon Squeezy API.

The CMSs that can be added to the application:

- **@kit/keystatic**: [Keystatic](https://keystatic.com/) package that defines the schema and logic for managing Keystatic. This is used by the @kit/cms package and abstracts the Keystatic API
- **@kit/wordpress**:  WordPress package that defines the schema and logic for managing WordPress. This is used by the @kit/cms package and abstracts the WordPress API.

Also planned (post-release):

- **@kit/plugins**: Move the existing plugins to a separate package here
- **@kit/analytics**: A unified analytics package to track user behavior

### Application Configuration

The configuration is defined in the `apps/web/config` folder. Here you can find the following configuration files:

- **app.config.ts**: Application configuration (e.g., name, description, etc.)
- **auth.config.ts**: Authentication configuration
- **billing.config.ts**: Billing configuration
- **feature-flags.config.ts**: Feature flags configuration
- **paths.config.ts**: Paths configuration (e.g., routes, API paths, etc.)
- **personal-account-sidebar.config.ts**: Personal account sidebar configuration (e.g., links, icons, etc.)
- **team-account-sidebar.config.ts**: Team account sidebar configuration (e.g., links, icons, etc.)


-----------------
FILE PATH: kits\next-supabase-turbo\installation\update-nextjs15.mdoc

---
status: "published"

title: "Guidelines for updating Makerkit to Next.js 15"
label: "Updating to Next.js 15"
order: 9
description: "A guide to updating Makerkit to Next.js 15. All you need to know to migrate your codebase to the latest Next.js version."
---


Makerkit was originally built on top of Next.js 14. However, with Next.js 15 being released, we've updated the boilerplate to align with the latest version.

This means there will be a few changes to your codebase that you may need to apply or merge from the latest version of the boilerplate.

Fortunately, the API hasn't had large break changes, so the migration should be relatively straightforward. However, there are some changes that you may need to make, and you'll need to do them manually.

This documentation will only guide through what we need in Makerkit. However, for a more comprehensive guide, you can refer to the [Next.js documentation](https://nextjs.org/docs/canary/app/building-your-application/upgrading/version-15) and [Makerkit's own guide to updating to Next.js 15](/blog/changelog/upgrade-nextjs-15).

### The changes in Next.js 15

Here's a broad overview of the changes:

1. **Update Next.js version**: Update the Next.js version in the various packages
2. **Update Next.js configuration**: If you've modified the Next.js configuration, you'll need to update it to align with the latest version
3. **Dynamic Params are now Promise-based**: Probably the most significant change is the introduction of dynamic params. This means that you'll need to update your code to use promises instead of syncronous functions when accessing cookies, headers, and query params and segments.

The kit also added some other general improvements that regard linting and formatting, therefore some merge conflicts may arise. However, these are less important and you can simply resolve them manually by accepting your own changes.

The migration make take anything between 5 minutes to an hour depending on your codebase. Believe me, it's still a lot less than the changes and research I did on my side! 😅

## Update Makerkit to the latest version

To update Makerkit to the latest version, you can run the following command:

```bash
git pull upstream main
```

This will update all the packages in your project to the latest version. You may need to merge conflicts manually.

Then, install the packages again:

```bash
pnpm i
```

## Updated Promise-based dynamic APIs

The large majority of the changes in the Makerkit's Next.js 15 update are related to the change of dynamic APIs into Promise-based APIs.

This means that you'll need to update your code to use promises instead of syncronous functions when accessing cookies, headers, and query params and segments.

For example, where you before had the following code:

```tsx
const cookie = cookies().get('session');
```

You will now need to use the following code:

```tsx
const cookieStore = await cookies();
const cookie = cookieStore.get('session');
```

The same applies to headers, query params, and segments.

### Using the Next.js 15 Codemod

Next.js ships a codemod that will automatically update your codebase to use promises instead of syncronous functions. You can use it by running the following command:

```bash
npx @next/codemod@canary upgrade latest
```

If you want to run this command, please do it within the apps/web directory, because this command does not support monorepos.

Please note this command may not be perfect - so please double check the changes it makes. Please refer to Next.js Github issues if you encounter any problems caused by this command, not Makerkit's support.

### React Compiler

The React Compiler is set to be **off** by default. It is still experimental, and it is not recommended to enable it in production.

However, you can enable it by setting the `experimental.reactCompiler` flag to `true` in your `next.config.mjs` file.

You can do so by enabling the environment variable `ENABLE_REACT_COMPILER` in your `.env` file.

```tsx
ENABLE_REACT_COMPILER=true
```

### Speeding up Local Development with mock modules

Since we don't need to compile all the packages during development, we make sure to alias them to the mock modules, therefore speeding up the development compilation (eg., page transitions, hot reload, etc.).

We use Turbopack's alias feature to achieve this, in the `next.config.mjs` file.

There's not too much to it, but if you're curious, this is why we have added the code loading the mock modules.

### Dependencies Updates

All the dependencies have been updated to the latest version. This includes notable changes such as Radix UI, Stripe

## Granular TailwindCSS Paths

Tailwind's generation has slowed down considerably (for some reason), and the fix was to transform the path ` '../../packages/**/src/**/*.tsx'` into granular paths.

 ```tsx {% title="tooling/tailwind/index.ts" %}
'../../packages/ui/src/**/*.tsx',
   '../../packages/billing/gateway/src/**/*.tsx',
   '../../packages/features/auth/src/**/*.tsx',
   '../../packages/features/notifications/src/**/*.tsx',
   '../../packages/features/admin/src/**/*.tsx',
   '../../packages/features/accounts/src/**/*.tsx',
   '../../packages/features/team-accounts/src/**/*.tsx',
   '!**/node_modules',
 ],
```

If you have created your own library, please make sure to add it to the list.

For example, you've created a feature named `projects` and you want to use Tailwind in it. You can add the following path to the list:

 ```tsx {% title="tooling/tailwind/index.ts" %}
'../../packages/features/projects/src/**/*.tsx',
```

## Conclusion

Updating Makerkit to Next.js 15 is a relatively straightforward process. However, you may encounter merge conflicts, and you'll need to resolve them manually.

In addition, you will need to update your code to use promises instead of synchronous functions when accessing cookies, headers, and query params and segments.

Please head to our Discord server if you have any questions or need help with the update process.


-----------------
FILE PATH: kits\next-supabase-turbo\installation\update-tailwindcss-v4.mdoc

---
status: "published"
title: "Guidelines for updating Makerkit to Tailwind CSS v4"
label: "Updating to Tailwind CSS v4"
order: 9
description: "A guide to updating Makerkit to Tailwind CSS v4. All you need to know to migrate your codebase to the latest Tailwind CSS version."
---

Makerkit was originally built with Tailwind CSS v3, and has since been updated to Tailwind CSS v4. This guide will walk you through the changes and migration steps required to update your codebase to the latest version should you choose to update manually.

If you don't want to update manually, please pull the latest changes from Makerkit. You can use the below as a reference for the changes you need to make for the code you've written so far.

## Major Changes Overview

1. **Tailwind CSS Version Update**: Upgraded from v3.4.17 to v4.0.0
2. **CSS Architecture**: Moved to a component-based CSS architecture with separate stylesheets
3. **File Structure**: Reorganized CSS files into modular components
4. **Styling Changes**: Updated various UI components with new styling patterns

## File Structure Changes

The CSS structure has been reorganized into multiple files:

```txt {% title="apps/web/styles/*.css" %}
styles/
├── globals.css
├── theme.css
├── theme.utilities.css
├── shadcn-ui.css
├── markdoc.css
└── makerkit.css
```

1. `globals.css`: Global styles for the entire application
2. `theme.css`: Theme variables and colors
3. `theme.utilities.css`: Utility classes for the theme
4. `shadcn-ui.css`: ShadcN UI specific styles
5. `markdoc.css`: Markdown/documentation styles
6. `makerkit.css`: Makerkit specific components

### Retaining your existing styles

If you wish to kee your existing theme colors, please update the `shadcn-ui.css` file and keep the same variables for the theme.

NB: you have to use the `hsl` function to update the theme colors.

```css {% title="apps/web/styles/shadcn-ui.css" %}
@layer base {
  :root {
    --background: hsl(0 0% 100%);
    --foreground: hsl(224 71.4% 4.1%);
    --card: hsl(0 0% 100%);
    --card-foreground: hsl(224 71.4% 4.1%);
    --popover: hsl(0 0% 100%);
    --popover-foreground: hsl(224 71.4% 4.1%);
    --primary: hsl(220.9 39.3% 11%);
    --primary-foreground: hsl(210 20% 98%);
    --secondary: hsl(220 14.3% 95.9%);
    --secondary-foreground: hsl(220.9 39.3% 11%);
    --muted: hsl(220 14.3% 95.9%);
    --muted-foreground: hsl(220 8.9% 46.1%);
    --accent: hsl(220 14.3% 95.9%);
    --accent-foreground: hsl(220.9 39.3% 11%);
    --destructive: hsl(0 84.2% 60.2%);
    --destructive-foreground: hsl(210 20% 98%);
    --border: hsl(214.3 31.8% 94.4%);
    --input: hsl(214.3 31.8% 91.4%);
    --ring: hsl(224 71.4% 4.1%);
    --radius: 0.5rem;

    --chart-1: hsl(12 76% 61%);
    --chart-2: hsl(173 58% 39%);
    --chart-3: hsl(197 37% 24%);
    --chart-4: hsl(43 74% 66%);
    --chart-5: hsl(27 87% 67%);

    --sidebar-background: hsl(0 0% 98%);
    --sidebar-foreground: hsl(240 5.3% 26.1%);
    --sidebar-primary: hsl(240 5.9% 10%);
    --sidebar-primary-foreground: hsl(0 0% 98%);
    --sidebar-accent: hsl(240 4.8% 95.9%);
    --sidebar-accent-foreground: hsl(240 5.9% 10%);
    --sidebar-border: hsl(220 13% 91%);
    --sidebar-ring: hsl(217.2 91.2% 59.8%);
  }

  .dark {
    --background: hsl(224 71.4% 4.1%);
    --foreground: hsl(210 20% 98%);
    --card: hsl(224 71.4% 4.1%);
    --card-foreground: hsl(210 20% 98%);
    --popover: hsl(224 71.4% 4.1%);
    --popover-foreground: hsl(210 20% 98%);
    --primary: hsl(210 20% 98%);
    --primary-foreground: hsl(220.9 39.3% 11%);
    --secondary: hsl(215 27.9% 13%);
    --secondary-foreground: hsl(210 20% 98%);
    --muted: hsl(215 27.9% 13%);
    --muted-foreground: hsl(217.9 10.6% 64.9%);
    --accent: hsl(215 27.9% 13%);
    --accent-foreground: hsl(210 20% 98%);
    --destructive: hsl(0 62.8% 30.6%);
    --destructive-foreground: hsl(210 20% 98%);
    --border: hsl(215 27.9% 13%);
    --input: hsl(215 27.9% 13%);
    --ring: hsl(216 12.2% 83.9%);

    --chart-1: hsl(220 70% 50%);
    --chart-2: hsl(160 60% 45%);
    --chart-3: hsl(30 80% 55%);
    --chart-4: hsl(280 65% 60%);
    --chart-5: hsl(340 75% 55%);

    --sidebar-background: hsl(224 71.4% 4.1%);
    --sidebar-foreground: hsl(240 4.8% 95.9%);
    --sidebar-primary: hsl(224.3 76.3% 48%);
    --sidebar-primary-foreground: hsl(0 0% 100%);
    --sidebar-accent: hsl(215 27.9% 13%);
    --sidebar-accent-foreground: hsl(240 4.8% 95.9%);
    --sidebar-border: hsl(240 3.7% 15.9%);
    --sidebar-ring: hsl(217.2 91.2% 59.8%);
  }
}
```

## Breaking Changes

### 1. Class Name Updates

- Replace `space-x-` and `space-y-` with `gap-x-` and `gap-y-`
- Update shadow utilities:
- `shadow` → `shadow-xs`
- `shadow-sm` → `shadow-xs`
- `shadow-lg` → `shadow-xl`

NB: the spacing has changed from v3 using dynamic spacing, so you may need to update your custom spacing values.

### 2. Border Radius Changes

- Update rounded utilities:
- `rounded-sm` → `rounded-xs`
- Keep other rounded values the same

### 3. Color System Changes

- The theme has been updated with a better looking color system, especially on dark mode

### 4. Layout Updates

- Flex gap spacing:
- Replace `space-x-` with `gap-x-`
- Replace `space-y-` with `gap-y-`

### 5. Container Changes

```css
/* Old */
.container {
  @apply max-sm:px-4;
}

/* New */
@utility container {
  margin-inline: auto;
  @apply xl:max-w-[80rem] px-8;
}
```

1. **Outline Utilities**
```css
/* Old */
focus:outline-none

/* New */
focus:outline-hidden
```

2. **Shadow Utilities**
```css
/* Old */
shadow-sm

/* New */
shadow-xs
```

## Migration Steps

1. **Update Dependencies**
```json
{
  "dependencies": {
    "tailwindcss": "4.0.0",
    "@tailwindcss/postcss": "^4.0.0"
  }
}
```

The `tailwindcss-animate` dependency is now part of `apps/web/package.json` and should be removed from `package.json` in the Tailwind CSS v4 upgrade.

2. **Update PostCSS Config**
```js
module.exports = {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

3. **CSS Files**
- Move existing global styles to appropriate new CSS files
- Update import statements to include new CSS files
- Remove old tailwind.config.ts and replace with new CSS structure

4. **Component Updates**
- Review all components using spacing utilities
- Update shadow utilities
- Review and update color usage
- Update flex and grid gap utilities

## Testing Checklist

- [ ] Verify all components render correctly
- [ ] Check responsive layouts
- [ ] Test dark mode functionality
- [ ] Verify shadow and elevation styles
- [ ] Test container layouts
- [ ] Verify color system implementation
- [ ] Check form component styling
- [ ] Test navigation components
- [ ] Verify modal and overlay styling
- [ ] Check typography scaling

## Additional Resources

- [Tailwind CSS v4 Documentation](https://tailwindcss.com/)

