-----------------
FILE PATH: kits\next-supabase-turbo\installation\code-health.mdoc

---
status: "published"
label: "Code Health and Testing"
title: "Code Health and Testing in Makerkit. Set up Github Actions, pre-commit hooks | Next.js Supabase SaaS Kit"
order: 10
description: "Learn how to set up Github Actions, pre-commit hooks, and more in Makerkit to ensure code health and quality."
---

Makerkit provides a set of tools to ensure code health and quality in your project. This includes setting up Github Actions, pre-commit hooks, and more.

## Github Actions

By default, Makerkit sets up Github Actions to run tests on every push to the repository. You can find the Github Actions configuration in the `.github/workflows` directory.

The workflow has two jobs:

1. `typescript` - runs the Typescript compiler to check for type errors.
2. `test` - runs the Playwright tests in the `tests` directory.

### Enable E2E Tests

Since it needs some setup - these are not enabled by default. To enable E2E tests, you need to set the following environment variables in the Github Actions workflow:

```bash
ENABLE_E2E_JOB=true
```

### Configuring the E2E environment for Github Actions
To set up Github Actions for your project, please add the following secrets to your repository Github Actions settings:

1. `SUPABASE_SERVICE_ROLE_KEY` - the service role key for Supabase.
2. `SUPABASE_DB_WEBHOOK_SECRET` - the webhook secret for Supabase.
3. `STRIPE_SECRET_KEY` - the secret key for Stripe.
4. `STRIPE_WEBHOOK_SECRET` - the webhook secret for Stripe.

These are the same as the ones you have running the project locally. Don't use real secrets here - use the development app secrets that you use locally. **This is a testing environment**, and you don't want to expose your production secrets.

Additionally, please set the following environment variables in the Github Actions workflow:

1. `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` - this is the publishable key for Stripe.
2. `SUPABASE_DB_WEBHOOK_SECRET`: Use the value `WEBHOOKSECRET` for the webhook secret. Again, this is a testing environment, so we add testing values.

### Enable Stripe Tests

Since it needs some setup - these are not enabled by default. To enable Stripe tests, you need to set the following environment variables in the Github Actions workflow:

```bash
ENABLE_BILLING_TESTS=true
```

Of course - make sure you have the Stripe secrets set up in the Github Actions secrets.

## Pre-Commit Hook

It's recommended to set up a pre-commit hook to ensure that your code is linted correctly and passes the typechecker before committing.

### Setting up the Pre-Commit Hook

To do so, create a `pre-commit` file in the `./.git/hooks` directory with the following content:

```bash
#!/bin/sh

pnpm run typecheck
pnpm run lint
```

Turborepo will execute the commands for all the affected packages - while skipping the ones that are not affected.

### Make the Pre-Commit Hook Executable

Now, make the file executable:

```bash
chmod +x ./.git/hooks/pre-commit
```

To test the pre-commit hook, try to commit a file with linting errors or type errors. The commit should fail, and you should see the error messages in the console.

### What about the e2e tests?

You could also add tests - but it'll slow down your commits. It's better to run tests in Github Actions.


-----------------
FILE PATH: kits\next-supabase-turbo\installation\common-commands.mdoc

---
status: "published"

title: 'Common commands you need to know for the Next.js Supabase Turbo Starter Kit'
label: 'Common Commands'
order: 5
description: 'Learn about the common commands you need to know to work with the Next.js Supabase Turbo Starter Kit.'
---


Here are some common commands you'll need to know when working with the Next.js Supabase Turbo Starter Kit.

**Note:** You don't need these commands to kickstart your project, but it's useful to know they exist for when you need them.

## Installing Dependencies

To install the dependencies, run the following command:

```bash
pnpm i
```

This command will install all the necessary dependencies for the project.

## Starting the Development Server

Start the development server for the web application with:

```bash
pnpm run dev
```

## Running Supabase CLI Commands

Supabase is installed in the `apps/web` folder. To run commands with the Supabase CLI, use:

```bash
pnpm run --filter web supabase <command>
```

For example, if Supabase documentation recommends a command like:

```bash
supabase db link
```

You would run:

```bash
pnpm run --filter web supabase db link
```

## Starting Supabase

To start Supabase, run:

```bash
pnpm run supabase:web:start
```

This command starts the Supabase web server.

## Starting Stripe

To test the billing system, start Stripe with:

```bash
pnpm run stripe:listen
```

This routes webhooks to your local machine.

## Resetting Supabase

To reset the Supabase database, which is necessary when you update the schema or need a fresh start, run:

```bash
pnpm run supabase:web:reset
```

## Generate Supabase Types

When you update the Supabase schema, generate the latest types for the client by running:

```bash
pnpm run supabase:web:typegen
```

This should be done every time the Supabase schema is updated.

## Running Tests

To run the tests for the project, use:

```bash
pnpm run test
```

## Cleaning the Project

To clean the project, run:

```bash
pnpm run clean:workspaces
pnpm run clean
```

Then, reinstall the dependencies:

```bash
pnpm i
```

## Type-Checking the Project

To type-check the project, use:

```bash
pnpm run typecheck
```

## Linting the Project

To lint the project using ESLint, run:

```bash
pnpm run lint:fix
```

## Formatting the Project

To format the project using Prettier, run:

```bash
pnpm run format:fix
```

These commands will help you manage and maintain your project efficiently.

-----------------
FILE PATH: kits\next-supabase-turbo\installation\conventions.mdoc

---
status: "published"
description: "Makerkit uses conventions to ensure consistency and readability in the codebase."
title: "Conventions in the Next.js Supabase Turbo Starter Kit"
label: "Conventions"
order: 3
---

Below are some of the conventions used in the Next.js Supabase Turbo Starter Kit.

You're not required to follow these conventions: they're simply a standard set of practices used in the core kit. If you like them - I encourage you to keep these during your usage of the kit - so to have consistent code style that you and your teammates understand.

### Turborepo Packages

In this project, we use Turborepo packages to define reusable code that can be shared across multiple applications.

- **Apps** are used to define the main application, including routing, layout, and global styles.
- Apps pass down configuration to the packages, and the packages provide the corresponding logic and components.

### Creating Packages

**Should you create a package for your app code?**

- **Recommendation:** Do not create a package for your app code unless you plan to reuse it across multiple applications or are experienced in writing library code.
- If your application is not intended for reuse, keep all code in the app folder. This approach saves time and reduces complexity, both of which are beneficial for fast shipping.
- **Experienced developers:** If you have the experience, feel free to create packages as needed.

### Imports and Paths

When importing modules from packages or apps, use the following conventions:

- **From a package:** Use `@kit/package-name` (e.g., `@kit/ui`, `@kit/shared`, etc.).
- **From an app:** Use `~/` (e.g., `~/lib/components`, `~/config`, etc.).

### Non-Route Folders

Non-route folders within the `app` directory are prefixed with an underscore (e.g., `_components`, `_lib`, etc.).

- This prefix indicates that these folders are not routes but are used for shared components, utilities, etc.

### Server Code

Files located in `server` folders are intended for server-side code only. They should not be used in client-side code.

- This convention helps clarify where the code is meant to run, which is particularly important in Next.js where the distinction can be blurry.
- For example, server-related code for a part of the app can be found in the `_lib/server` folder.
- Include the `server-only` package at the top of the file to ensure it is not accidentally imported in client-side code.

-----------------
FILE PATH: kits\next-supabase-turbo\installation\faq.mdoc

---
status: "published"
title: "FAQ - Questions about the Next.js SaaS Boilerplate"
label: "FAQ"
order: 12
description: "Frequently asked questions about the Next.js SaaS Boilerplate."
---

The below is a technical FAQ about this kit. For general questions about Makerkit, please see the [Makerkit FAQ](/faq).

## Technical FAQ

### Do I need to know Supabase to use the Next.js SaaS Boilerplate?

Yes, you should have a basic understanding of Supabase to use the Next.js SaaS Boilerplate. You'll need to know how to:

- Create a Supabase project
- Set up the database
- Understand PostgreSQL
- Use the Supabase client in your Next.js app

While you can use the kit to learn, it does not teach you how to use Supabase. For that, please refer to the [Supabase documentation](https://supabase.com/docs).

### Do I need to know Next.js to use the Next.js SaaS Boilerplate?

Yes, you should have a basic understanding of Next.js to use the Next.js SaaS Boilerplate.

### I don't know Supabase! Should I buy the Next.js SaaS Boilerplate?

You should be prepared for a learning curve or consider learning Supabase first. The Next.js SaaS Boilerplate is built on top of Supabase, so knowing how to use Supabase is essential.

### I don't know Turborepo! Should I buy the Next.js SaaS Boilerplate?

Yes, you can still use the Next.js SaaS Boilerplate without prior knowledge of Turborepo. Turborepo is used to manage the monorepo structure of the boilerplate. Your main focus will be on building your SaaS product within the `apps/web` directory, not on the tools used to build the boilerplate. Even without experience using Turborepo, you won't need to interact with it directly unless you plan to customize the core code in the `packages` directory.

### Will you add more packages in the future?

Very likely! This kit is designed to be modular, allowing for new features and packages to be added without interfering with your existing code. There are many ideas for new packages and features that may be added in the future.

### Can I use this kit for a non-SaaS project?

This kit is primarily intended for SaaS projects. If you're building a non-SaaS project, the Next.js SaaS Boilerplate might be overkill. You can still use it, but you might need to remove some features specific to SaaS projects.

### Can I use personal accounts only?

Yes, you can set a feature flag to disable team accounts and use personal accounts only.

### Can I use the React package X with this kit?

Yes, you can use any React package with this kit. The kit is a simple Next.js application, so you are generally only constrained by the underlying technologies (Next.js, Stripe, Supabase, etc.) and not by the kit itself. Since you own and can edit all the code, you can adapt the kit to your needs. However, if there are limitations with the underlying technology, you might need to work around them.

### Does Makerkit set up the production instance for me?

No, Makerkit does not set up the production instance for you. This includes setting up Supabase, Stripe, and any other services you need. Makerkit does not have access to your Stripe or Supabase accounts, so setup on your end is required. Makerkit provides the codebase and documentation to help you set up your SaaS project.

### How do I get support if I encounter issues?

For support, you can:

1. Visit our Discord
2. Contact us via support email

### Are there any example projects or demos?

Yes - you get access to the OpenAI demo.

### How do I deploy my application?

Please check the [production checklist](../going-to-production/checklist) for more information.

### How do I contribute to the Next.js SaaS Boilerplate?

We welcome contributions! Please ping me if you'd like to contribute (licensees only).

### How do I update my project when a new version of the boilerplate is released?

Please [read the documentation for updating your Makerkit project](updating-codebase).


-----------------
FILE PATH: kits\next-supabase-turbo\installation\functional-walkthrough.mdoc

---
status: "published"
title: 'Functional Walkthrough - Next.js Supabase Turbo Starter Kit'
label: 'Walkthrough'
order: 8
description: 'A functional walkthrough of the Next.js Supabase Turbo Starter Kit. Understand the features and how to use the kit.'
---

This is a functional walkthrough of the Next.js Supabase Turbo Starter Kit. In this guide, you'll learn about the functional aspects of the kit.

## Overview of the Next.js Supabase Turbo Starter Kit

We can break down the Next.js Supabase Turbo Starter Kit into the following functional sections:

1. **Marketing / External Section** - the public-facing part of the application. This also includes the blog and documentation.
2. **Authentication** - the authentication system of the application.
3. **Personal Account Features** - the features available to personal accounts.
4. **Team Account Features** - the features available to team accounts.
5. **Invitations** - the invitation system of the application.
6. **Super Admin** - the super admin features of the application.

## Marketing / External Section

The marketing section is the public-facing part of the application. It is where users can learn about the product, the pricing and the legal information.

### Home Page

The home page is the landing page of the application. It showcases the features of the product and encourages users to sign up.

{% img src="/assets/images/docs/walkthrough/home-page.webp" width="3420" height="2142" /%}

### Pricing

The pricing page is where users can learn about the different pricing plans of the product.

{% img src="/assets/images/docs/walkthrough/pricing.webp" width="3420" height="2142" /%}

This section is also added to the home page.

### FAQ

The FAQ page is where users can find answers to common questions about the product.

{% img src="/assets/images/docs/walkthrough/faq.webp" width="3420" height="2142" /%}

### Contact

The contact page is where users can get in touch with the company. It includes a contact form that allows users to send messages to the company directly.

{% img src="/assets/images/docs/walkthrough/contact.webp" width="3420" height="2142" /%}

### Content Pages

Content pages can be displayed using the CMS that you have setup. By default, the kit implements a Blog and a Documentation systems using either Keystatic or Wordpress. You can choose which one you prefer.

#### Blog

The blog is where the company can publish articles about the product, the industry, and other topics.

Below is the page where all the latest blog posts are listed:

{% img src="/assets/images/docs/walkthrough/blog.webp" width="3420" height="2142" /%}

And here is an example of a blog post:

{% img src="/assets/images/docs/walkthrough/blog-post.webp" width="3420" height="2142" /%}

#### Documentation

The documentation is where users can learn how to use the product. It includes guides, tutorials, and reference material.

{% img src="/assets/images/docs/walkthrough/walkthrough-documentation.webp" width="3420" height="2142" /%}

### Legal Pages

The legal pages are, of course, empty and need to be filled in with the appropriate legal information.

Don't use ChatGPT to fill them up. It's a bad idea.

## Authentication

The authentication system is where users can sign up, sign in, reset their password. It also includes multi-factor authentication.

### Sign Up

The sign-up page is where users can create an account. They need to provide their email address and password.

{% img src="/assets/images/docs/walkthrough/sign-up.webp" width="3420" height="2142" /%}

Once successful, users are asked to confirm their email address. This is enabled by default - and due to security reasons, it's not possible to disable it.

{% img src="/assets/images/docs/walkthrough/sign-up-success.webp" width="3420" height="2142" /%}

### Sign In

The sign-in page is where users can log in to their account. They need to provide their email address and password.

{% img src="/assets/images/docs/walkthrough/sign-in.webp" width="3420" height="2142" /%}

### Password Reset

The password reset page is where users can reset their password. They need to provide their email address.

{% img src="/assets/images/docs/walkthrough/password-reset.webp" width="3420" height="2142" /%}

### Multi-Factor Authentication

Multi-Factor Authentication (MFA) is an additional layer of security that requires users to provide two or more verification factors to sign in to their account.

First, users need to enable MFA and add a factor:

{% img src="/assets/images/docs/walkthrough/setup-mfa.webp" width="3420" height="2142" /%}

Then, after signing in, users need to provide the verification code:

{% img src="/assets/images/docs/walkthrough/verify-mfa.webp" width="3420" height="2142" /%}

## Internal / Behind authentication pages

After signing in - users are redirected to the internal pages of the application. These pages are only accessible to authenticated users.

The internal part of the application is divided into two workspaces:

1. The user workspace
2. The team workspace (optional)

This is how this works:

1. **Personal Account** - all users have a personal account. This is where they can: manage their settings, choose a team account - and **optionally** subscribe to a plan, or access the features you provide.
2. **Team Account (optional)** - users can create a team account - and invite other users to join. The team account has its own workspace - where users can manage the team settings, members, and billing.

Generally speaking, **it's up to you** to decide what features are available to personal accounts and team accounts. You can choose to disable billing for personal accounts - and only enable it for team accounts - or vice versa.

One simple rule of a thumb is that personal accounts are for individuals - and team accounts are for organizations. Personal accounts cannot be disabled, as that would disallow users from accessing the application should they not be part of a team - which doesn't make sense.

## Personal Account Features

The personal account workspace is where users can access the features available to their personal account.

This is the home page after logging in - and it's the user workspace:

1. Home Page - empty by default (but you can optionally provide the list of teams the user is part of)
2. Account Settings
3. Billing (if enabled)

### Home Page of the user workspace

By default - the user home is an empty page - as it's likely you will want to place some content that is unique to your SaaS.

However, we provide a component that allows you to lists the team an account is part of: this is useful for B2B SaaS rather than B2C.

The internal pages have two layouts:

1. A sidebar layout (default)
2. A top navigation layout

You can choose any of the two - and also choose either one for the user layout or the account layout.

Below is the user home page with the sidebar layout:

{% img src="/assets/images/docs/walkthrough/user-home-sidebar.webp" width="3420" height="2142" /%}

And below is the user home page with the top navigation layout:

{% img src="/assets/images/docs/walkthrough/user-home-header.webp" width="3420" height="2142" /%}

You can choose the one that fits your needs.

### Account Settings of the user workspace

From the navigation - users can access their account settings. Here they can update their profile information, change their password, language, multi-factor authentication, and more.

We've used light mode so far - how about dark mode? Let's switch to dark mode:

{% img src="/assets/images/docs/walkthrough/user-account-settings.webp" width="3420" height="2142" /%}

### Billing of the user workspace

Users can check out and subscribe to a plan - or visit the billing portal - from the billing page.

**This is only visible if billing is enabled**: you can choose to disable billing for a personal account - and only enable it for team accounts - or vice versa.

{% img src="/assets/images/docs/walkthrough/user-billing.webp" width="3420" height="2142" /%}

Once choosing a plan - we load the embedded checkout form from Stripe (or Lemon Squeezy).

After subscribing, the billing page will show the subscription details.

{% img src="/assets/images/docs/walkthrough/user-billing-plan.webp" width="3420" height="2142" /%}

## Team Account Features

From the profile dropdown, users can choose:

1. Switch to a team account
2. Create a team account

{% img src="/assets/images/docs/walkthrough/user-profile-dropdown.webp" width="876" height="796" /%}

In a team account workspace - users can access the following features:

1. A team account home page: by default - we display a mock dashboard, just as an example.
2. Account settings: users can update the team account settings.
3. Members: users can view the members of the team account.
4. Billing: users can view the billing of the team account.

### Home Page of the team workspace

By default - the team home is a mock dashboard - just as an example. You can replace this with your own dashboard - or any other content.

{% img src="/assets/images/docs/walkthrough/team-account-dashboard.webp" width="3420" height="2142" /%}

### Account Settings of the team workspace

From the navigation - users can access the team account settings. Here they can update the team account information, or delete the team account (if owner), or leave the team account (if member).

{% img src="/assets/images/docs/walkthrough/team-account-settings.webp" width="3420" height="2142" /%}

### Members page of the team workspace

From the navigation - users can access the members of the team account.

Here they can:

1. view the members
2. invite new members
3. remove or update an existing member
4. transfer ownership to another member
5. remove or update an invitation

{% img src="/assets/images/docs/walkthrough/team-account-members.webp" width="3420" height="2142" /%}

### Billing of the team workspace

If enabled - users can view the billing of the team account - and subscribe to a plan or visit the billing portal.

{% img src="/assets/images/docs/walkthrough/team-account-billing.webp" width="3420" height="2142" /%}

## Joining a team account

When a user is invited to join a team account - they receive an email with an invitation link. After signing up or signing in - they are redirected to the join page.

{% img src="/assets/images/docs/walkthrough/sign-up-invite.webp" width="3420" height="2142" /%}

### Join Page

The join page is where users can join a team account.

{% img src="/assets/images/docs/walkthrough/join-team.webp" width="3420" height="2142" /%}

## Super Admin

The super admin is the administrator of the SaaS. They have access to a special set of features that allow them to manage the accounts of the SaaS.

### Home Page of the super admin

The home page is a small overview of the SaaS.

You can easily customize this page to show the most important metrics of your SaaS.

{% img src="/assets/images/docs/walkthrough/super-admin-dashboard.jpg" width="3420" height="2142" /%}

### Listing the accounts of the SaaS

The super admin can view all the accounts of the SaaS. They can filter the accounts by personal accounts, team accounts, or all accounts.

{% img src="/assets/images/docs/walkthrough/super-admin-accounts.webp" width="3420" height="2142" /%}

### Viewing the account details

The super admin can view the details of an account. They can see the account information, the members of the account, and the billing information.

Additionally, they can perform the following actions:

1. Ban the account (or unban)
2. Delete the account

{% img src="/assets/images/docs/walkthrough/super-admin-account.webp" width="3420" height="2142" /%}

## Conclusion

This concludes the functional walkthrough of the Next.js Supabase Turbo Starter Kit. You should now have a good understanding of the features of the kit and how to use it. If you have any questions, feel free to reach out to us. We're here to help!


-----------------
FILE PATH: kits\next-supabase-turbo\installation\introduction.mdoc

---
status: "published"
title: 'Introduction to Next.js Supabase SaaS Kit Turbo Repository'
label: 'Introduction'
order: 1
description: 'Introducing the Next.js Supabase SaaS Kit Turbo repository, which is a Next.js starter kit for building SaaS applications with Supabase.'
---

Welcome to the latest version of the Next.js Supabase SaaS Kit! This starter kit is designed to help you build robust SaaS applications using the powerful combination of Next.js and Supabase.

Whether you're a beginner or an experienced developer, this documentation will guide you through the setup, configuration, and deployment of your SaaS application.

## What's New? 🎉

Compared to previous versions, this kit includes:

- **More Functionality:** Enhanced features to meet modern SaaS requirements.
- **Organized Code:** A cleaner, more maintainable codebase.
- **Modern Dependencies:** Up-to-date libraries and tools.

## Features

### Authentication

- **Sign Up, Sign In, Sign Out:** Easy and secure user authentication.
- **Forgot Password:** Simple password recovery process.

### Multi-Factor Authentication (MFA)

- **Enable MFA:** Add an extra layer of security to your account.

### Account Management

- **Personal Accounts:** Users have their own personal accounts in addition to organization accounts.
- **Organization Accounts:** Users can create and join organizations.
- **Organization Roles:** Assign different roles to users within organizations.
- **Organization Invitations:** Invite others to join your organization.

### Billing

- **Personal Account Billing:** Manage billing information for personal accounts.
- **Organization Billing:** Handle billing for organization accounts.
- **Subscriptions:** Support for various subscription plans, including flat, tiered, and per-seat models.
- **Multiple Billing Providers:** Out-of-the-box support for Stripe, Lemon Squeezy, and (soon) Paddle.

### Content Management Systems (CMS)

- **Multiple CMS Support:** Integration with Keystatic and WordPress.

### Monitoring

- **Monitoring Tools:** Built-in support for Sentry and Baselime.

## A Modular, Extensible, and Modern SaaS Starter Kit

### Modular Structure

The biggest change in this kit is its modular structure, which allows for:

- **🧩 Easy Feature Integration:** Plug new features into the kit with minimal changes.
- **🛠️ Simplified Maintenance:** Keep the codebase clean and maintainable.
- **🎯 Core Feature Separation:** Distinguish between core features and custom features.
- **🌟 Additional Modules:** Easily add modules like billing, CMS, monitoring, logger, mailer, and more.

## Scope of This Documentation

### Focus on the Kit

While building a SaaS application involves many moving parts, this documentation focuses specifically on the Next.js Supabase SaaS Kit. For in-depth information on the underlying technologies (Next.js, Supabase, Turbo, etc.), please refer to their respective official documentation.

This documentation will guide you through configuring, running, and deploying the kit, and will provide links to the official documentation of the underlying technologies where necessary. To fully grasp the kit's capabilities, it's essential to understand these technologies, so be sure to explore their documentation as well.

For anything strictly related to the Next.js Supabase SaaS Kit, this documentation has you covered!

### Refer to Official Documentation

For in-depth understanding of the underlying technologies, refer to their official documentation:

- **Next.js:** [Next.js Documentation](https://nextjs.org)
- **Supabase:** [Supabase Documentation](https://supabase.com)
- **Stripe:** [Stripe Documentation](https://stripe.com)

Understanding these technologies is crucial for building a successful SaaS application.

## Conclusion

This Next.js Supabase SaaS Kit is designed to accelerate your SaaS development with a robust, modular, and extensible foundation. Dive into the documentation, explore the features, and start building your SaaS application today! 🚀


