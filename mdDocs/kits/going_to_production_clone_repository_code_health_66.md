-----------------
FILE PATH: kits\react-router-supabase-turbo\going-to-production.mdoc

---
status: "published"
title: "Going to Production in React Router Supabase Turbo"
label: "Going to Production"
order: 15
description: "Going to Production in React Router Supabase Turbo"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits\react-router-supabase-turbo\installation\clone-repository.mdoc

---
status: "published"
title: "Clone the React Router Supabase SaaS Kit Turbo Repository"
label: "Clone the Repository"
order: 3
description: "Clone the React Router Supabase SaaS Kit Turbo repository to your local machine."
---

## 0. Prerequisites

- Node.js 18.x or later
- Docker
- Pnpm
- Supabase account (optional for local development)
- Payment Gateway account (Stripe/Lemon Squeezy)
- Email Service account (optional for local development)

## Verify your Git username

To verify you have access, you need to check that your local Git username is the same as set up in the Makerkit's Github organization.

Please run the following command to check your Git username:

```bash
git config user.username
```

If the output is not your Github username, or does not match the username registered in Makerkit's Github organization, you can set it using the following command:

```bash
git config --global user.username "Your Github Username"
```

Ex. if your Github username is `johndoe`, you can set it using the following command:

```bash
git config --global user.username "johndoe"
```

This is important to ensure you can run the repository.

## Cloning the Repository

Clone this repository with the command:

```bash
git clone git@github.com:makerkit/react-router-supabase-saas-kit-turbo
```

NB: If your SSH key isn't set - then use the HTTPS.

```bash
git clone https://github.com/makerkit/react-router-supabase-saas-kit-turbo
```

NB: please switch to HTTPS for ALL commands if you are not using SSH, not just the clone command.

**Windows Users**: place the repository near the root of your drive

Some Windows users hit errors loading certain modules due to very long paths. To avoid this, try placing the repository near the root of your drive.

Avoid using OneDrive, as it can cause issues with Node.js.

Now, remove the original `origin`:

```bash
git remote rm origin
```

Add upstream pointing to this repository so you can pull updates

```bash
git remote add upstream git@github.com:makerkit/react-router-supabase-saas-kit-turbo
```

Once you have your own repository, do the same but use `origin` instead of `upstream`

To pull updates (please do this daily with your morning coffee):

```bash
git pull upstream main
```

This will keep your repository up to date.

### 0.1. Install Pnpm

```bash
# Install pnpm
npm i -g pnpm
```

## 1. Setup dependencies

```bash
# Install dependencies
pnpm i
```

## 2. Post-merge Hooks

It's very useful to run the following command after pulling updates from the upstream repository:

```bash
pnpm i
```

This ensures that any new dependencies are installed and the project is up to date. We can run this command automatically after pulling updates by setting up a post-merge hook.

Create a new file named `post-merge` in the `.git/hooks` directory:

```bash
touch .git/hooks/post-merge
```

Add the following content to the `post-merge` file:

```bash
#!/bin/bash

pnpm i
```

Make the `post-merge` file executable:

```bash
chmod +x .git/hooks/post-merge
```

Now, every time you pull updates from the upstream repository, the `pnpm i` command will run automatically to ensure your project is up to date. This ensures you're always working with the latest changes and dependencies and avoid errors that may arise from outdated dependencies.


-----------------
FILE PATH: kits\react-router-supabase-turbo\installation\code-health.mdoc

---
status: "published"
label: "Code Health and Testing"
title: "Code Health and Testing in Makerkit. Set up Github Actions, pre-commit hooks | React Router Supabase SaaS Kit"
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

1. `VITE_STRIPE_PUBLISHABLE_KEY` - this is the publishable key for Stripe.
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
FILE PATH: kits\react-router-supabase-turbo\installation\common-commands.mdoc

---
status: "published"
title: 'Common commands you need to know for the React Router Supabase Turbo Starter Kit'
label: 'Common Commands'
order: 5
description: 'Learn about the common commands you need to know to work with the React Router Supabase Turbo Starter Kit.'
---

Here are some common commands you need to know to work with the React Router Supabase Turbo Starter Kit.

NB: You don't need these commands to kickstart your project - you just need to know they exist so you can use them when you need them.

## Installing dependencies

To install the dependencies, run the following command:

```bash
pnpm i
```

This command will install all the dependencies for the project.

## Starting the development server

```bash
pnpm run dev
```

## Running the Supabase CLI commands

Supabase is installed in the `apps/web` folder. To run commands with the Supabase CLI, you can use the command:

```
pnpm --filter web supabase <command>
```

For example, if the documentation in Supabase recommends a command such as:

```
supabase db link
```

You will use:

```
pnpm --filter web supabase db link
```

This command will start the development server for the web application.

## Starting Supabase

To start Supabase, run the following command:

```bash
pnpm run supabase:web:start
```

This command will start the Supabase web server.

## Starting Stripe

If you want to test the billing system, you can start Stripe by running the following command:

```bash
pnpm run stripe:listen
```

This will route webhooks to your local machine.

## Resetting Supabase

Resetting the Database is needed when you update the schema or need to start fresh.

If you need to reset the Supabase database, run the following command:

```bash
pnpm run supabase:web:reset
```

This will reset the Supabase database.

## Generate Supabase types

Generating the types is required when you update the Supabase schema. In this way, the client can have the latest types.

To generate the Supabase types, run the following command:

```bash
pnpm run supabase:web:typegen
```

This will generate the Supabase types for the project. This should be done every time you update the Supabase schema.

## Running tests

To run the tests, run the following command:

```bash
pnpm run test
```

This will run the tests for the project.

## Cleaning the project

To clean the project, run the following command:

```bash
pnpm run clean:workspaces
pnpm run clean
```

Then - reinstall the dependencies:

```bash
pnpm i
```

## Type-checkinh the project

To type-check the project, run the following command:

```bash
pnpm run typecheck
```

## Linting the project

To lint the project, run the following command:

```bash
pnpm run lint:fix
```

This will lint the project using ESLint.

## Formatting the project

To format the project, run the following command:

```bash
pnpm run format:fix
```

This will format the project using Prettier.


-----------------
FILE PATH: kits\react-router-supabase-turbo\installation\conventions.mdoc

---
status: "published"
description: "A collection of conventions used in the core kit"
title: "Conventions in the React Router Supabase Turbo Starter Kit"
label: "Conventions"
order: 7
---

Below are some of the conventions used in the React Router Supabase Turbo Starter Kit.

In this project, we use Turborepo packages to define reusable code that can be shared across multiple applications.

**Apps are used to define the main application**, including the routing, layout, and global styles.

Apps pass down the configuration to the packages - and the packages provide the corresponding logic and components.

#### Should I create a package for my app code?

I do not recommend creating a package for your app code unless you are planning to reuse it in multiple applications, or unless you're experienced writing library code.

If you are building an application that is not meant to be reused, you should keep all the code in the app folder.

This will save you time - and make you think less. Both are good things when you're trying to ship fast.

**If you are experienced** by all means, go for it!

#### Imports and Paths

When importing something from a package or an app, you will use the following paths:

- When you import something from a package, you will use `@kit/package-name` (e.g., `@kit/ui`, `@kit/shared`, etc.).
- When you import something from an app, you will use `~/` (e.g., `~/lib/components`, `~/config`, etc.).

#### Non-Route Folders

Non-route folders within `app` will be prefixed with an underscore (e.g., `_components`, `_lib`, etc.).

This makes it obvious that these folders are not routes and are used for shared components, utilities, etc.

#### Server Code

The files localed in `server` folders are to be assumed as server-side code. They are not meant to be used in the client-side code. This helps everyone understanding where the code is meant to be run, since the lines are very blurry in React Router.

For example, you will find the server code related to a part of the app at `_lib/server` folder.

-----------------
FILE PATH: kits\react-router-supabase-turbo\installation\faq.mdoc

---
status: "published"
title: "FAQ - Questions about the React Router SaaS Boilerplate"
label: "FAQ"
order: 11
description: "Frequently asked questions about the React Router SaaS Boilerplate."
---

The below is a technical FAQ about this kit. For general questions about Makerkit, please see the [Makerkit FAQ](/faq).

## Do I need to know Supabase to use the React Router SaaS Boilerplate?

Yes - you should have a basic understanding of Supabase to use the React Router SaaS Boilerplate. You'll need to know how to create a Supabase project, set up the database, Postgres knowledge, and use the Supabase client in your React Router app.

Sure - you can use the kit to learn, but ultimately the kit does not teach you how to use Supabase. You should refer to the [Supabase documentation](https://supabase.com/docs) for that.

## Do I need to know React Router to use the React Router SaaS Boilerplate?

Yes - you should have a basic understanding of React Router to use the React Router SaaS Boilerplate.

## I don't know Supabase! Should I buy the React Router SaaS Boilerplate?

You either accept there is a learning curve, or you can learn Supabase first. The React Router SaaS Boilerplate is built on top of Supabase, so you'll need to know how to use Supabase to use the kit.

## I don't know Turborepo! Should I buy the React Router SaaS Boilerplate?

Yes! You don't need to know Turborepo to use the React Router SaaS Boilerplate. Turborepo is used by me to manage the monorepo structure of the boilerplate. Your focus should be on building your SaaS product which resides in `apps/web`, not on the tools used to build the boilerplate.

Even if you don't have experience using Turborepo or a monorepo, you won't need to be using it directly unless you're going to customize the core code of Makerkit in `packages`.

## Will you add more packages in the future?

Very likely! This kit is meant to be modular - which allows me to build new stuff without interfering with your existing code. I have a lot of ideas for new packages and features that I want to add in the future.

## Can I use this kit for a non-SaaS project?

This kit is meant to be used for SaaS projects. If you're building a non-SaaS project, you might find the React Router SaaS Boilerplate to be overkill. You can still use it, but you might need to remove some features that are specific to SaaS projects.

## Can I use personal accounts only?

Sure! You can set a feature flag to disable team accounts.

## Can I use the React package X with this kit?

This kit is a simple React Router application. You can use any React package you want. Generally speaking, you're only constrained by the underlying technologies (React Router, Stripe, Supabase, etc.) and not by the kit itself. Since you own and can edit all the code, you can always adapt the kit to your needs. However, if the underlying technology has a limitation, you might need to work around it.


-----------------
FILE PATH: kits\react-router-supabase-turbo\installation\functional-walkthrough.mdoc

---
status: "published"
title: "Functional Walkthrough - React Router Supabase Turbo Starter Kit"
label: "Walkthrough"
order: 8
description: "A functional walkthrough of the React Router Supabase Turbo Starter Kit. Understand the features and how to use the kit."
---

This is a functional walkthrough of the React Router Supabase Turbo Starter Kit. In this guide, you'll learn about the functional aspects of the kit.

## Overview of the React Router Supabase Turbo Starter Kit

We can break down the React Router Supabase Turbo Starter Kit into the following functional sections:

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

### Blog

The blog is where the company can publish articles about the product, the industry, and other topics.

Below is the page where all the latest blog posts are listed:

{% img src="/assets/images/docs/walkthrough/blog.webp" width="3420" height="2142" /%}

And here is an example of a blog post:

{% img src="/assets/images/docs/walkthrough/blog-post.webp" width="3420" height="2142" /%}

### Documentation

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

1. Home Page - empty by default
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

Users can checkout and subscribe to a plan - or visit the billing portal - from the billing page. This is only visible if billing is enabled. In fact - you can
choose to disable billing for a personal account - and only enable it for team accounts - or vice versa.

{% img src="/assets/images/docs/walkthrough/user-billing.webp" width="3420" height="2142" /%}

Once choosing a plan - we load the embedded checkout form from Stripe (or Lemon Squeezy).

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

The home page is a small overview of the SaaS. You can easily customize this page to show the most important metrics of your SaaS.

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

This concludes the functional walkthrough of the React Router Supabase Turbo Starter Kit. You should now have a good understanding of the features of the kit and how to use it. If you have any questions, feel free to reach out to us. We're here to help!

