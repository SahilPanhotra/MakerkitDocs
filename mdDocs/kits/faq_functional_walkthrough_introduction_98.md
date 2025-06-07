-----------------
FILE PATH: kits/remix-supabase-turbo/installation/faq.mdoc

---
status: "published"
title: "FAQ - Questions about the Remix SaaS Boilerplate"
label: "FAQ"
order: 11
description: "Frequently asked questions about the Remix SaaS Boilerplate."
---


The below is a technical FAQ about this kit. For general questions about Makerkit, please see the [Makerkit FAQ](/faq).

## Do I need to know Supabase to use the Remix SaaS Boilerplate?

Yes - you should have a basic understanding of Supabase to use the Remix SaaS Boilerplate. You'll need to know how to create a Supabase project, set up the database, Postgres knowledge, and use the Supabase client in your Remix app.

Sure - you can use the kit to learn, but ultimately the kit does not teach you how to use Supabase. You should refer to the [Supabase documentation](https://supabase.com/docs) for that.

## Do I need to know Remix to use the Remix SaaS Boilerplate?

Yes - you should have a basic understanding of Remix to use the Remix SaaS Boilerplate.

## I don't know Supabase! Should I buy the Remix SaaS Boilerplate?

You either accept there is a learning curve, or you can learn Supabase first. The Remix SaaS Boilerplate is built on top of Supabase, so you'll need to know how to use Supabase to use the kit.

## I don't know Turborepo! Should I buy the Remix SaaS Boilerplate?

Yes! You don't need to know Turborepo to use the Remix SaaS Boilerplate. Turborepo is used by me to manage the monorepo structure of the boilerplate. Your focus should be on building your SaaS product which resides in `apps/web`, not on the tools used to build the boilerplate.

Even if you don't have experience using Turborepo or a monorepo, you won't need to be using it directly unless you're going to customize the core code of Makerkit in `packages`.

## Will you add more packages in the future?

Very likely! This kit is meant to be modular - which allows me to build new stuff without interfering with your existing code. I have a lot of ideas for new packages and features that I want to add in the future.

## Can I use this kit for a non-SaaS project?

This kit is meant to be used for SaaS projects. If you're building a non-SaaS project, you might find the Remix SaaS Boilerplate to be overkill. You can still use it, but you might need to remove some features that are specific to SaaS projects.

## Can I use personal accounts only?

Sure! You can set a feature flag to disable team accounts.

## Can I use the React package X with this kit?

This kit is a simple Remix application. You can use any React package you want. Generally speaking, you're only constrained by the underlying technologies (Remix, Stripe, Supabase, etc.) and not by the kit itself. Since you own and can edit all the code, you can always adapt the kit to your needs. However, if the underlying technology has a limitation, you might need to work around it.


-----------------
FILE PATH: kits/remix-supabase-turbo/installation/functional-walkthrough.mdoc

---
status: "published"

title: "Functional Walkthrough - Remix Supabase Turbo Starter Kit"
label: "Walkthrough"
order: 8
description: "A functional walkthrough of the Remix Supabase Turbo Starter Kit. Understand the features and how to use the kit."
---


This is a functional walkthrough of the Remix Supabase Turbo Starter Kit. In this guide, you'll learn about the functional aspects of the kit.

## Overview of the Remix Supabase Turbo Starter Kit

We can break down the Remix Supabase Turbo Starter Kit into the following functional sections:

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

This concludes the functional walkthrough of the Remix Supabase Turbo Starter Kit. You should now have a good understanding of the features of the kit and how to use it. If you have any questions, feel free to reach out to us. We're here to help!

-----------------
FILE PATH: kits/remix-supabase-turbo/installation/introduction.mdoc

---
status: "published"

title: "Introduction to Remix Supabase SaaS Kit Turbo Repository"
label: "Introduction"
order: 1
description: "Introducing the Remix Supabase SaaS Kit Turbo repository, which is a Remix starter kit for building SaaS applications with Supabase."
---


This kit is the latest version of the Remix Supabase SaaS Kit, which is a Remix starter kit for building SaaS applications with Supabase.

Compared to the previous kit, this kit has been updated with more functionality, more organized code, and mode modern dependencies.

## Features

- **Authentication**: Sign up, sign in, sign out, and forgot password.
- **Multi Factor Authentication (MFA)**: Enable MFA for your account.
- **Personal Accounts**: Users have their own personal accounts (besides the organization account).
- **Organization Accounts**: Users can create and join organizations.
- **Organization Roles**: Users can have different roles in different organizations.
- **Organization Invitations**: Users can invite others to join their organization.
- **Personal Account Billing**: Users can manage their billing information for their personal account.
- **Organization Billing**: Users can manage their billing information for their organization.
- **Subscriptions**: Users can subscribe to a multitude of plans, such as flat subscriptions, tiered, per-seat, and more.
- **Multiple Billing Providers**: The kit supports Stripe, Lemon Squeezy and Paddle (soon) out of the box.
- **CMSs**: The kit supports multiple CMSs, such as Keystatic and Wordpress
- **Monitoring**: The kit supports monitoring with Sentry and Baselime

## A Modular, Extensible and Modern SaaS Starter Kit

The biggest change in this kit is the modular structure. This allows us to:

- plug new features easily into the kit with no (or minimal) changes to the existing codebase.
- maintain the kit more easily.
- better separate the "core" features from your custom features.
- add additional modules (billing, CMS, monitoring, logger, mailer, etc.) to the kit.

And more!

## A word on this documentation's scope

This documentation teaches you how to configure, run and deploy the kit. This documentation does not replace the documentation of the underlying technologies (Remix, Supabase, Turbo, etc.). Instead, it focuses on the specific configuration and usage of the kit.

If you need help with Remix, Supabase, Turbo, etc., please refer to their respective documentation. We will provide links to the official documentation where necessary. For anything strictly related to the kit, you can refer to this documentation.

-----------------
FILE PATH: kits/remix-supabase-turbo/installation/migration-from-makerkit-v1.mdoc

---
status: "published"
title: "Guidelines for migrating from Makerkit v1 | Remix Supabase SaaS Starter Kit"
label: "Migrating from v1"
order: 9
description: "Guidelines for migrating from Makerkit v1 to Remix SaaS Boilerplate."
---

This guide will help you migrate from Makerkit v1 to the Remix SaaS Boilerplate. The Remix SaaS Boilerplate is a complete rewrite of Makerkit v1, and there are significant changes between the two versions. This guide will help you understand the changes and how to migrate your project to the new v2.

Overall, this is what you will need to do:

1. **Bootstrap a new project**: Create a new project using the Remix SaaS Boilerplate v2
2. **Update Schema**: Update the Postgres schema foreign key references, and add your existing tables
3. **Move files from older app**: Move your files to the new app structure
4. **Update imports to existing files**: Update imports to the new file structure
5. **Update configuration**: Update the configuration files to match the new schemas

## 1. Bootstrap a new project

The Remix SaaS Boilerplate is a complete rewrite of Makerkit v1. You will need to create a new project using the new boilerplate. You can follow the [installation guide](clone-repository) to create a new project.

## 2. Update Schema

The schema in the Remix SaaS Boilerplate has changed significantly from Makerkit v1. You will need to update your Postgres schema to match the new schema.

The main difference is that before, you'd have a foreign key set to the organization ID:

```sql
organization_id bigint not null references organizations(id) on delete cascade,
```

Now, you'll have a foreign key set to the account ID as a UUID:

```sql
account_id uuid not null references public.accounts(id) on delete cascade,
```

In fact, an account can be both a user or a organization. While in v1 you referenced the organization ID, in v2 you'll reference the account ID. It's the same if you were using the Lite kit and referencing a user ID: now you'll reference the account ID related to the user ID.

## 3. Move files from older app

In v2 you can choose to add these files to either the "user scope" (`/home`) or the "account scope" (`/home/[account]`).

If you choose to add the files to the "user scope", you can add them to the `/home` directory. If you choose to add the files to the "account scope", you can add them to the `/home/[account]` directory.

## 4. Update imports to existing files

You will need to update the imports to your existing files to match the new file structure.

This is the case for all the components and utilities that you imported from Makerkit.

For example, if you were importing a button from `~/core/ui/Button`, you will now import this from `@kit/ui/button` - and so on.

## 5. Update configuration

You will need to update the configuration files to match the new schemas. The configuration is now split across various files at `apps/web/config`. Most notably, you will need to update the billing schema, which is now much more versatile (and a bit more complex).

-----------------
FILE PATH: kits/remix-supabase-turbo/installation/navigating-codebase.mdoc

---
status: "published"
title: "Navigating the your Remix Supabase Turbo Starter Kit codebase"
label: "Navigating the Codebase"
order: 7
description: "Learn how to navigate the codebase of the Remix Supabase Turbo Starter Kit. Understand the project structure and how to update the codebase."
---


As mentioned before, this SaaS Starter Kit is modular, thanks to Turborepo - which makes it easy to manage multiple packages in a single repository.

## Project Structure

The main directories in the project are:
1. `apps` - the location of the Remix app
2. `packages` - the location of the shared code and the API

### `apps` Directory

This is where the app (or the apps, should you add more) lives. The main app is a Remix app, which is a React framework that allows you to build static and server-rendered web applications.

The Remix app defines:

1. the configuration of the project
2. the routes of the application

### `packages` Directory

This is where the shared code and the API live. The shared code is a collection of functions that are used by both the Remix app and the API. The API is a serverless function that is used to interact with the Supabase database.

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

The `apps/web` directory is where the Remix app lives. Here are some of the key directories and files you should be aware of:

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

### Diving into `(dashboard)`

The `(dashboard)` directory is where the internal dashboard of the application lives. Here are some of the key files you should be aware of:

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
FILE PATH: kits/remix-supabase-turbo/installation/running-project.mdoc

---
status: "published"
title: "Running the Remix Supabase Turbo project"
label: "Running the Project"
order: 4
description: "Learn how to run the Remix Supabase Turbo project on your local machine."
---

To run the project - there are some commands you need to run.

1. Start the development server (web application)
2. Start Supabase (ensure Docker is running)
3. Start Stripe (optional, if you want to test the billing system)

## 1. Start the development server

```bash
# Start the development server
pnpm dev
```

This command will run the web application.

Please refer to `apps/web/README.md` for more information about the web application.

To get started right away, use the credentials below:

- **Email**: `test@makerkit.dev`
- **Password**: `testingpassword`

To confirm email addresses, please visit [Inbucket](http://localhost:54324/status): Supabase uses Inbucket to capture emails sent during the authentication process.

## 2. Start Supabase

Run the following command to start Supabase (or use your IDE to run the command in the package.json)

```bash
pnpm run supabase:web:start
```

This command will start the Supabase web server.

## 3. Start Stripe

If you want to test the billing system, you can start Stripe by running the following command:

```bash
pnpm run stripe:listen
```

This will route webhooks to your local machine.

