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

## 1. Start Supabase

{% alert type="warning" title="Have Docker running" %}
We need to have Docker running so that we can start the Supabase Docker container. If you don't have Docker running, please start it now.
{% /alert %}

[Docker Desktop](https://www.docker.com/products/docker-desktop/) is the best way to get started with Docker.

Ensure Docker is installed and running, then start Supabase with:

```bash
pnpm run supabase:web:start
```

This command initiates the local Supabase web server. This allows us to develop locally without having to deploy to Supabase.

When you're ready to deploy the project to production, follow the [checklist](../going-to-production/checklist) to ensure everything is properly configured.

Please read the [Supabase documentation](https://supabase.com/docs/guides/local-development/cli/getting-started#running-supabase-locally) for more information about starting Supabase locally.

## 2. Start the Development Server

To start the web application development server, run:

```bash
# Start the development server
pnpm dev
```

This command launches the web application.

For more details about the web application, please refer to `apps/web/README.md`.

### Quick Start Credentials

Makerkit seeds the DB with a few users by default. To login as a test user, you can use the following credentials:

- **Email**: `test@makerkit.dev`
- **Password**: `testingpassword`

To confirm email addresses, visit [Inbucket](http://localhost:54324). Supabase uses Inbucket/Mailpit to capture emails sent during the authentication process.

Bookmark the Inbucket/Mailpit URL, as you will need it quite often.

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

Since you require MFA for the Super Admin user, please use the following steps to pass MFA:

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

To receive emails during local development, Supabase uses [InBucket/Mailpit](http://localhost:54324). It's from here that you will be receving emails from both Supabase Auth and transactional emails during local development.

You can bookmark the InBucket/Mailpit URL, as you will need it quite often.

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

-----------------
FILE PATH: kits\next-supabase-turbo\installation\updating-codebase.mdoc

---
status: "published"

title: "Updating your Next.js Supabase Turbo Starter Kit"
label: "Updating the Codebase"
order: 6
description: "Learn how to update your Next.js Supabase Turbo Starter Kit to the latest version."
---


🚀 Ready to keep your project at the cutting edge? 

This guide will walk you through the process of updating your codebase by pulling the latest changes from the GitHub repository and merging them into your project. This ensures you're always equipped with the latest features and bug fixes.

If you've been following along with our previous guides, you should already have a Git repository set up for your project, with an `upstream` remote pointing to the original repository.

Updating your project involves fetching the latest changes from the `upstream` remote and merging them into your project. Let's dive into the steps!

## 0. Stashing your changes

If you have uncommited changes, before updating your project, it's a good idea to stash your changes to avoid any conflicts during the update process. You can stash your changes by running:

```bash
git stash
```

This will save your changes temporarily, allowing you to update your project without any conflicts. Once you've updated your project, you can apply your changes back by running:

```bash
git stash pop
```

If you don't have any changes to stash, you can skip this step and proceed with the update process. 🛅

Alternatively, you can commit your changes.

## 1. Refresh the `upstream` remote

The first step is to fetch the latest changes from the `upstream` remote. You can do this by running the following command:

```bash
git pull upstream main
```

When prompted the first time, please opt for merging instead of rebasing.

Don't forget to run `pnpm i` in case there are any updates in the dependencies. 🔄

## 2. Resolve any conflicts

Encountered conflicts during the merge? No worries! You'll need to resolve them manually. Git will highlight the files with conflicts, and you can edit them to resolve the issues.

### 2.1 Conflicts in the lock file "pnpm-lock.yaml"

If you find conflicts in the `pnpm-lock.yaml` file, accept either of the two changes (avoid manual edits), then run:

```bash
pnpm i
```

Your lock file will now reflect both your changes and the updates from the `upstream` repository. 🎉

### 2.2 Conflicts in the DB types "database.types.ts"

Your types might differ from those in the `upstream` repository, so you'll need to rebuild them. To do this, reset the DB:

```bash
npm run supabase:web:reset
```

Next, regenerate the types with the following command:

```bash
npm run supabase:web:typegen
```

Your types will now reflect the changes from both the `upstream` repository and your project. 🚀

### Run a health check on your project after resolving conflicts

After resolving the conflicts, it's time to test your project to ensure everything is working as expected. Run your project locally and navigate through the various features to verify that everything is functioning correctly.

You can run the following commands for a quick health check:

```bash
pnpm run typecheck
```

And lint your code with:

```bash
pnpm run lint
```

If everything looks good, you're all set! Your project is now up to date with the latest changes from the `upstream` repository. 🎉

## Commit and push changes

Once everything is working fine, don't forget to commit your changes with git commit -m "COMMIT_MESSAGE" and push them to your remote repository with git push origin BRANCH_NAME.

Of course - please make sure to replace COMMIT_MESSAGE and BRANCH_NAME with your actual commit message and branch name. 🚀

-----------------
FILE PATH: kits\next-supabase-turbo\installation.mdoc

---
status: "published"
title: "Installation in Next.js Supabase Turbo"
label: "Installation"
order: 1
description: "Installation in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Hi, welcome to the installation guide for the Next.js Supabase SaaS Kit Turbo. This guide will help you set up your development environment and get started with the kit.

In this section, we will cover:

1. [Introduction](introduction): the second version of Makerkit
2. [Technical Details](technical-details): the technical details of this kit
3. [Conventions](conventions): some of the conventions used in this kit
4. [Clone the Repository](clone-repository): cloning the repository
5. [Running the Project](running-the-project): running the project
6. [Common Commands](common-commands): some common commands to help you work with the project
7. [Updating the Project](updating-codebase): how to update the codebase



-----------------
FILE PATH: kits\next-supabase-turbo\monitoring\baselime.mdoc

---
status: "published"
title: 'Configuring Baselime in your Next.js Supabase SaaS kit'
label: 'Baselime'
order: 1
description: 'Learn how to configure Baselime in your Next.js Supabase SaaS kit'
---


[Baselime](https://baselime.io) (now part of Cloudflare) is an observability platform that helps you monitor your application's performance and errors. In this guide, you'll learn how to configure Baselime in your Next.js Supabase SaaS kit.

To use [Baselime](https://baselime.io) to capture exceptions and performance metrics of your app, please define the below variables:

```bash
NEXT_PUBLIC_BASELIME_KEY=your_key
NEXT_PUBLIC_MONITORING_PROVIDER=baselime
```

You can find your Baselime key in the Baselime dashboard.

If you also want to enable OpenTelemetry, you can set the following variables:

```bash
ENABLE_MONITORING_INSTRUMENTATION=true
INSTRUMENTATION_SERVICE_NAME=your_service_name
```


-----------------
FILE PATH: kits\next-supabase-turbo\monitoring\overview.mdoc

---
title: "Setting Monitoring in Makerkit | Next.js SaaS Boilerplate"
status: "published"
label: "How Monitoring works"
position: 0
description: "Introducing how Makerkit handles monitoring of performance metrics and exceptions in the Next.js Supabase SaaS kit"
---

Makerkit provides first-class support for two monitoring providers:

1. Baselime (now part of Cloudflare)
2. Sentry

Makerkit will set up a few things for you out of the box:

1. **Performance Metrics** - Instrumentation using Next.js's instrumentation hook
2. **Client exceptions** - Automatically capturing uncaught exceptions on the client
3. **Server exceptions** - Automatically capturing server-side exceptions captured by the [Next.js instrumentation hook](https://nextjs.org/docs/app/api-reference/file-conventions/instrumentation#onrequesterror-optional)

Additionally, it provides you with the tools to manually capturing exceptions, should you want to.

### Configuring Monitoring in Makerkit

To set up monitoring in your application, you need to define the two variables below:

```bash
# sentry or baselime
NEXT_PUBLIC_MONITORING_PROVIDER=

# performance monitoring (only required for Baselime)
ENABLE_MONITORING_INSTRUMENTATION=true
INSTRUMENTATION_SERVICE_NAME=your_service_name
```

1. **Monitoring Provider** - the monitoring provider to use. Based on this variable, Makerkit will provide the relative implementation.
2. **Enable Instrumentation** - if enabled, we report performance metrics to the provider using Next.js

For the provider's specific settings, please check the relative documentation.

#### Performance Monitoring

Performance monitoring uses Next.js' experimental instrumentation for reporting performance metrics using OpenTelemetry.

NB: Performance monitoring is only enabled for Node.js runtimes.

