-----------------
FILE PATH: kits\next-supabase-turbo\emails.mdoc

---
status: "published"
title: "Emails in Next.js Supabase Turbo"
label: "Emails"
order: 11
description: "Emails in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Everything you need to know about handling emails in your Makerkit application:

1. [**Getting Started with Email**](/docs/next-supabase-turbo/email-configuration)
   Configure your email provider and understand the email system architecture.
2. [**Authentication Emails**](/docs/next-supabase-turbo/authentication-emails)
   Set up and customize authentication-related emails like verification, password reset, and login notifications.
3. [**Email Templates**](/docs/next-supabase-turbo/email-templates)
   Learn how to create and customize email templates using React.Email.
4. [**Sending Emails**](/docs/next-supabase-turbo/sending-emails)
   Implement email sending functionality in your application using the Makerkit mailer system.

Whether you're setting up transactional emails, authentication notifications, or marketing communications, this guide has you covered! 📧


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\authentication-emails.mdoc

---
title: "Setting the Supabase Auth Email Templates in Production"
label: "Authentication Emails"
description: "Configure the Supabase authentication URLs and emails in the Next.js Supabase Starter Kit."
order: 4
---

Makerkit provides a set of email templates that you can use for replacing the standard Supabase Auth emails. Additionally, you can customize these templates to match your brand.

{% sequence title="How to set the Supabase Auth Email Templates in Production" description="Makerkit provides a set of email templates that you can use for replacing the standard Supabase Auth emails. Additionally, you can customize these templates to match your brand." %}

[Setting the Email Templates in Supabase](#setting-the-email-templates-in-supabase)

[Customizing the Email Templates](#customizing-the-email-templates)

{% /sequence %}

## Setting the Email Templates in Supabase

Why should you use our email templates?

**Please update the auth emails using the [following documentation in Supabase](https://supabase.com/docs/guides/auth/auth-email-templates#redirecting-the-user-to-a-server-side-endpoint)**.

{% alert type="warning" %}
Failure to do so will result in hiccups in the authentication flow when users click on an email and get redirected to a different browser than the one they used to sign up due to how the PKCE flow works.
{% /alert %}

1. They will use the token hash strategy, which remediates the issue of users being redirected to a different browser than the one they used to sign up.
2. They look better than the default Supabase templates and you can customize them to match your brand.

## Customizing the Email Templates

Please clone the templates repository locally and customize them to your liking:

1. **Emails Starter**: Clone our Emails Starter at [https://github.com/makerkit/makerkit-emails-starter](https://github.com/makerkit/makerkit-emails-starter). This repository contains a [React.Email](https://react.email/) project with the email templates already set up.
2. **Customize the templates**: Next, you want to customize the templates as you see fit. For example, adding your product name, logo, and other branding elements.
3. **Export the templates**: Export the templates with your own information
4. **Replace the templates**: Replace the templates in the `apps/web/supabase/templates` folder
5. **Update the email templates in your Supabase settings**: Update the email templates in your Supabase's instance settings so that they use your custom templates.

Now your emails from Supabase Auth will look great and match your brand! 🎉


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\authentication.mdoc

---
status: "published"
title: "Setting the Supabase Auth Settings in Production | Next.js Supabase SaaS Kit"
label: "Authentication"
description: "Configuring Authentication's production configuration in the Next.js Supabase SaaS kit"
order: 3
---

{% sequence title="How to configure the Supabase Auth Settings in Production" description="Configuring Authentication's production configuration in the Next.js Supabase SaaS kit" %}

[Authentication URLs](#authentication-urls)

[Third Party Providers](#third-party-providers)

[Custom SMTP](#custom-smtp)

[Troubleshooting](#troubleshooting)

{% /sequence %}

Supabase needs a few settings to be configured in their Dashboard to work correctly.

This guide will walk you through the steps to get your Supabase authentication set up in your Production environment. The dev environment does not require any configuration.

{% alert type="warning" title="Don't skip this step" %}
Skipping this step will result in your users not being able to log in or sign up.
{% /alert %}

## Authentication URLs

The first thing you need to do is to [set the authentication URLs in the Supabase Dashboard](https://supabase.com/docs/guides/auth/redirect-urls). These URLs are used to redirect users to the correct page after they have logged in or signed up.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Authentication** tab.
4. Click on **URL Configuration**.
5. Add your Site URL to the **Site URL** field. This is the URL of your MakerKit site (e.g. `https://my-site.com`).
6. Add your Redirect URLs to the **Redirect URLs** field. This is the URL of your MakerKit site with `/auth/callback` appended to it (e.g. `https://my-site.com/auth/callback`).

Failure to set these URLs will result in your users not being able to log in or sign up.

## Custom SMTP

Supabase's SMTP settings have low limits and low deliverability. If you want your emails to go out and be delivered, please remember to set an alternative SMTP provider in the Supabase settings.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Project Settings** tab.
4. Click on **Auth**.
5. Tweak the `SMTP Settings` settings to your liking according to your provider's documentation.

Please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp) for more information.

## Third Party Providers

Third Party providers need to be configured, managed and enabled fully on the provider's and Supabase's side. Makerkit does not need any configuration (beyond setting the provider to be displayed in the UI).

Please [read Supabase's documentation](https://supabase.com/docs/guides/auth/social-login) on how to set up third-party providers.


### Troubleshooting

If you are having authentication issues, ensure that the Site URL and Redirect URLs are correct. If you are using a custom domain, ensure that you are using the correct domain in the Site URL and Redirect URLs.

{% alert type="warning" %}
NB: if your domain includes "www", ensure you include it in the Site URL and Redirect URLs. If your domain does not
  include "www", ensure you do not include it in the Site URL and Redirect URLs. If these do not match, your users will
  not be able to login.
{% /alert %}

If something is still not working, please open a support ticket with any useful information (such as server logs).


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\checklist.mdoc

---
status: "published"
title: "Checklist for deploying your Next.js Supabase SaaS to Production"
label: "Checklist"
description: "This guide will help you deploy your Next.js Supabase SaaS app to production! Please follow this checklist to ensure everything is set up correctly."
order: 0
---

When you're ready to deploy your project to production, follow this checklist.

{% alert type="warning" title="Please complete the following steps" %}
Not completing these steps will result in your project not working correctly. Please make sure to complete these steps before testing your project.
{% /alert %}

This process may take a few hours and some trial and error, so buckle up—you're almost there!

{% sequence title="Deployment Checklist" description="Follow these steps in order to deploy your Next.js Supabase SaaS to production." %}
Create a Supabase project and [link it locally](https://supabase.com/docs/guides/cli/local-development#link-your-project) using the Supabase CLI (`supabase link`).

[Push the migrations](https://supabase.com/docs/guides/cli/local-development#link-your-project) to the remote Supabase instance (`supabase db push`).

[Set your APP URL in the Supabase project settings](https://supabase.com/docs/guides/auth/redirect-urls). This is required for the OAuth flow. Make sure to add the path `/auth/callback**` to the allowed URLs. If you don't have it yet, wait.

[Set the OAuth providers in the Supabase](https://supabase.com/docs/guides/auth/social-login) project settings. If you use Google Auth, make sure to set it up. This requires creating a Google Cloud project and setting up the OAuth credentials.

Update the auth emails using the [following documentation](authentication-emails). The kit already implements the `confirm` route, but you need to update the emails in your Supabase settings.

Deploy the Next.js app to Vercel or any another hosting provider. Copy the URL and set it in the Supabase project settings.

Set up environment variables. The initial deploy **will likely fail** because you may not yet have a URL to set in your environment variables. This is normal. Once you have the URL, set the URL in the environment variables and redeploy.

[Set the DB Webhooks in Supabase](supabase) pointing against your Next.js app at `/api/db/webhooks`.

Get some SMTP details from an email service provider like SendGrid or Mailgun or Resend and configure the emails in both the Environment Variables and the [Supabase project settings](https://supabase.com/docs/guides/auth/auth-smtp).

Create a Stripe/Lemon Squeezy account, make sure to update the environment variables with the correct values. Point webhooks from these to `/api/billing/webhook`. Please use the relative documentation for more details.

Clear OTPs using a Cron Job or do so manually (optional).
{% /sequence %}

{% sequence title="Additional Tasks" description="Consider these minor but important tasks to complete your setup." %}
Update the legal pages with your company's information (privacy policy, terms of service, etc.).

Remove the placeholder blog and documentation content / or replace it with your own.

Update the favicon and logo with your own branding.

Update the FAQ and other static content with your own information.
{% /sequence %}

---

## Detailed Steps

{% sequence title="1. Create a Supabase Project" description="Set up your database, authentication, and storage services in the cloud." %}
Creating a Supabase project is the foundational step to set up your database, authentication, and storage services in the cloud. This will serve as the backend for your SaaS application.

Create a new Supabase project in the Supabase dashboard. Once created, [link the project locally](https://supabase.com/docs/guides/cli/local-development#link-your-project) using the Supabase CLI:
```bash
supabase link
```
{% /sequence %}

{% sequence title="2. Migrations" description="Configure your database schema." %}
Pushing database migrations ensures that your database schema in the remote Supabase instance is configured to match Makerkit's requirements. This step is crucial for the application to function correctly.

Push the database migrations to your remote Supabase instance:
```bash
supabase db push
```
{% /sequence %}

{% sequence title="3. Auth Configuration" description="Set up OAuth flows and redirect URLs." %}
Setting your APP URL in the Supabase project settings is critical for enabling OAuth flows and redirecting users correctly during authentication.

[Set your APP URL in the Supabase project settings](https://supabase.com/docs/guides/auth/redirect-urls). Add the path `/auth/callback` to the allowed URLs.
{% /sequence %}

{% sequence title="4. OAuth Providers" description="Set up external OAuth providers." %}
Configuring OAuth providers like Google ensures that users can log in using their existing accounts, enhancing user convenience and security. This is all done externally, in both Google and Supabase - not in the application code.

[Set up the OAuth providers](https://supabase.com/docs/guides/auth/social-login) in your Supabase project settings. For Google Auth, create a Google Cloud project and set up the OAuth credentials.
{% /sequence %}

{% sequence title="5. Auth Emails" description="Update authentication emails." %}
To provide a correct user experience with Makerkit's SSR authentication, you need to update the authentication emails to include the token hash and prevent errors usually related to PKCE - i.e when users click on the email and are redirected to a different browser - resulting in an error.

Update the authentication emails using the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp). The kit implements the `confirm` route, but you need to update the email templates in your Supabase settings.
{% /sequence %}

{% sequence title="6. Deploy Next.js" description="Deploy your application to a hosting provider." %}
Deploying your Next.js app to a hosting provider makes it accessible to users worldwide, allowing them to interact with your application.

Deploy your Next.js app to Vercel or another hosting provider. Copy the deployment URL and set it in your Supabase project settings.
{% /sequence %}

{% sequence title="7. Environment Variables" description="Set up environment variables." %}
Setting the correct environment variables is essential for the application to function correctly. These variables include API keys, database URLs, and other configuration details required for your app to connect to various services.

Please [generate the environment variables using our script](production-environment-variables) and then add them to your hosting provider's environment variables. Redeploy the app once you have the URL to set in the environment variables.
{% /sequence %}

{% sequence title="8. Webhooks" description="Set up database webhooks." %}
Configuring database webhooks allows your application to respond to changes in the database in real-time, such as sending notifications or updating records, ensuring your app stays in sync with the database.

[Set up the database webhooks in Supabase](https://supabase.com/docs/guides/database/webhooks) to point to your Next.js app at `/api/db/webhooks`.
{% /sequence %}

{% sequence title="9. Emails Configuration" description="Set up email service." %}
Properly configuring your email service ensures that your application can send emails for account verification, password resets, and other notifications, which are crucial for user communication.

Obtain SMTP details from an email service provider such as SendGrid, Mailgun, or Resend. Configure the emails in both the environment variables and the [Supabase project settings](https://supabase.com/docs/guides/auth/auth-smtp).
{% /sequence %}

{% sequence title="10. Billing Setup" description="Set up billing." %}
Setting up billing ensures that you can charge your users for using your SaaS application, enabling you to monetize your service and cover operational costs. This can take a while.

Create a Stripe or Lemon Squeezy account. Update the environment variables with the correct values for your billing service. Point webhooks from Stripe or Lemon Squeezy to `/api/billing/webhook`. Refer to the relevant documentation for more details on setting up billing.
{% /sequence %}

{% sequence title="11. OTPs (optional)" description="Clear unused DB records." %}
to remove unused DB records, we provide a function `kit.cleanup_expired_nonces()` to automatically clear unused or expired tokens

From your Supabase Studio, launch `select kit.cleanup_expired_nonces()` manually, or create a cron job to do so at a frequency of your choice
{% /sequence %}


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\cloudflare.mdoc

---
status: "published"
title: 'Deploy Next.js Supabase to Cloudflare'
label: 'Deploy to Cloudflare'
order: 6
description: 'This guide will help you deploy the Next.js SaaS boilerplate to Cloudflare Pages using the Edge runtime.'
---

{% sequence title="How to deploy the Next.js SaaS boilerplate to Cloudflare" description="To deploy the application to Cloudflare, you need to do the following:" %}

[0. Before you start: update to the paid Cloudflare tier](#before-you-start-update-to-the-paid-cloudflare-tier)

[1. Run the Turbo command to scaffold the project for Cloudflare](#1-run-the-turbo-command-to-scaffold-the-project-for-cloudflare)

[2. Use Console logger](#2-use-console-logger)

[3. Update Stripe Client](#up)

[4. Using the Resend Mailer or create a custom mailer](#3-using-the-resend-mailer-or-create-a-custom-mailer)

[5. Switching CMS](#4-switching-cms)

[6. Environment variables](#5-environment-variables)

[7. Deploy the application](#6-deploy-the-application)

[8. Other useful commands](#7-other-useful-commands)

{% /sequence %}

### 0. Before you start: update to the paid Cloudflare tier

Deploying to Cloudflare requires a subscription to the Cloudflare Workers paid service due to the size limitations of the free tier. It starts at $5.

### 1. Run the Turbo command to scaffold the project for Cloudflare

Run the following command to scaffold the project for Cloudflare:

```bash
pnpm run turbo gen cloudflare
```

This command will add some configuration files and install the necessary dependencies for Cloudflare.

### 2. Use Console logger

Cloudflare doesn't support the `pino` logger, so you will need to use the `console` logger:

```bash
LOGGER=console
```

### 3. Update Stripe Client

The Stripe client is not compatible with Cloudflare by default, so we must add `Stripe.createFetchHttpClient()` as property to the Stripe SDK:

```tsx {% title="packages/billing/stripe/src/services/stripe-sdk.ts" %}
import 'server-only';

import { StripeServerEnvSchema } from '../schema/stripe-server-env.schema';

const STRIPE_API_VERSION = '2025-03-31.basil';

/**
 * @description returns a Stripe instance
 */
export async function createStripeClient() {
  const { default: Stripe } = await import('stripe');

  // Parse the environment variables and validate them
  const stripeServerEnv = StripeServerEnvSchema.parse({
    secretKey: process.env.STRIPE_SECRET_KEY,
    webhooksSecret: process.env.STRIPE_WEBHOOK_SECRET,
  });

  return new Stripe(stripeServerEnv.secretKey, {
    apiVersion: STRIPE_API_VERSION,
    httpClient: Stripe.createFetchHttpClient()
  });
}
```

### 4. Using the Resend Mailer or create a custom mailer

Since the default library `nodemailer` relies on Node.js, we cannot use it in the Edge runtime. Instead, we will use the Resend Mailer.

```bash
MAILER_PROVIDER=resend
```

And provide the Resend API key:

```bash
RESEND_API_KEY=your-api-key
```

Alternatively, **implement your own mailer** using the abstract class we provide in the `packages/mailers` package.

This is very simple, but you need to ensure that the mailer is compatible with the Cloudflare runtime.

### 5. Switching CMS

By default, Makerkit uses Keystatic as a CMS. Keystatic's local mode (which relies on the file system) is not supported in Cloudflare's runtime.

Therefore, you will need to switch to another CMS or use Keystatic's Github mode - which uses Github as data provider.

At this time, the other CMS supported is WordPress. If you want to use WordPress, set `CMS_CLIENT` to `wordpress` in the `apps/web/.env` file:

```bash
CMS_CLIENT=wordpress
```

Alternatively, use the Keystatic remote mode through Github.

Please check the CMS documentation for more information.

### 6. Environment variables

Create a new file named `.env.production.local` in the `apps/web` directory and make sure to add the required environment variables.

You can use the Dev Tools to generate the environment variables that you can then paste into the `.env.production.local` file.

### 7. Deploy the application

At this time, you can deploy the application using the following command:

```bash
pnpm --filter web run deploy
```

At the time of writing, the Cloudflare Dashboard does not support deploying the application since OpenNext is not supported yet in Cloudflare.

### 8. Other useful commands

You can also use the following commands to deploy the application:

```bash
pnpm --filter web run preview
```

This command will preview the application in a Cloudflare-like environment before deploying it. It is highly recommended to run this command before deploying the application to understand if there are any issues.

```bash
pnpm --filter web run cf-typegen
```

This command will generate the TypeScript types for the Cloudflare environment.

-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\docker.mdoc

---
status: "published"
title: "Deploy Next.js Supabase using Docker"
label: "Deploy using Docker"
order: 10
description: "This guide will help you deploy the Next.js SaaS boilerplate to any Docker-compatible platform. Easily self-host your application with Docker."
---

If you want to self-host your application, you can deploy it using Docker. 

This guide will help you deploy the Next.js SaaS boilerplate to any Docker-compatible platform. For the sake of this guide, we will build and run the Docker container on a local machine.

### 0. Generate the Dockerfile

Run the following command to generate the Dockerfile:

```bash
pnpm run turbo gen docker
```

This command will generate a `Dockerfile` in the root directory of the project. In addition, it will install some dependencies that may be required to build the project, and set `output=standalone` in the `apps/web/next.config.mjs` file. If not, please do it manually.

### 1. Generate Environment Variables

Using the Dev Tools, generate the environment variables for the production environment. Place the environment variables in the `.env.production.local` file. You will then need to add them to the Docker container.

#### Exclude secrets during the build time

To avoid exposing secrets during the build time, make sure to temporarily
remove the secret keys from the env file. Open the `.env.production.local` file in the
`apps/web` directory and remove the following (but keep them around for later):

```bash
CAPTCHA_SECRET_TOKEN=***
OPENAI_API_KEY=***
RESEND_API_KEY=***
STRIPE_SECRET_KEY=***
STRIPE_WEBHOOK_SECRET=***
SUPABASE_DB_WEBHOOK_SECRET=***
SUPABASE_SERVICE_ROLE_KEY=***
```

### 2. Build the Docker Image

Run the following command to build the Docker image:

```bash
docker buildx build --platform linux/amd64 -t next-supabase-turbo .
```

The above command will build the Docker image for the `linux/amd64` architecture. This is the most common architecture for Docker images, however modify the command to build for the architecture you need.

### 3. Run the Docker Container

#### 3.1. Make sure the .env.production.local file exists

First, ensure that the `.env.production.local` file is present in the `apps/web` directory.


#### 3.2 Add secret environment variables

We can now add back the environment variables containing secret values that
will only be used at runtime:

```bash
CAPTCHA_SECRET_TOKEN=***
OPENAI_API_KEY=***
RESEND_API_KEY=***
STRIPE_SECRET_KEY=***
STRIPE_WEBHOOK_SECRET=***
SUPABASE_DB_WEBHOOK_SECRET=***
SUPABASE_SERVICE_ROLE_KEY=***
```

#### 3.3 Run the Docker container
Then, run the following command to run the Docker container:

```bash
docker run -d -p 3000:3000 --env-file .env.production.local next-supabase-turbo
```

### 4. Access the Application

You can now access the application at `http://<your-docker-host>:3000`.

### 5. Push the Docker Image to a Container Registry

If you want to push the Docker image to a container registry, you can run the following command:

```bash
docker push <your-docker-image>
```

If you're using the Github Container Registry, you can run the following command:

1. **Create a personal access token**: Generate a personal access token from Github - so that you can login to the Container Registry.
2. **Login to the Container Registry**: Login into the Github Container Registry:

```bash
docker login ghcr.io
```

### 6. Pull the Docker Image from the Github Container Registry in your VPS

Now, you can pull the Docker image from the Github Container Registry in your VPS.

1. **Login into your VPS**: Login into your VPS using SSH or a VPS management tool.
2. **Pull the Docker image**: Pull the Docker image from the Github Container Registry:

```bash
docker pull ghcr.io/your-username/repo
```

### 7. Run the Docker Container

Then, you can run the Docker container:

```bash
docker run -d -p 3000:3000 --env-file .env.production.local next-supabase-turbo
```

### 8. Access the Application

You can now access the application at `http://<your-docker-host>:3000`.



-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\production-environment-variables.mdoc

---
status: "published"
title: "Generating and validating the environment variables needed for deployment | Next.js Supabase SaaS Kit"
label: "Environment Variables"
description: "Guide to generate and validate the environment variables needed for deployment in your Next.js Supabase SaaS Kit project"
order: 1
---

{% sequence title="How to generate and validate the environment variables needed for deployment in your Next.js Supabase SaaS Kit project" description="Guide to generate and validate the environment variables needed for deployment in your Next.js Supabase SaaS Kit project" %}

[Before you start](#before-you-start)

[Generating the environment variables](#generating-the-environment-variables)

[Validating the environment variables](#validating-the-environment-variables)

{% /sequence %}

Makerkit has a lot of environment variables that need to be set for the project to work correctly. The environment variables are validated using Zod, so if you miss any, the project will not build correctly - at least in most cases.

While this can be annoying, you will agree with me that it's better to have a project that doesn't build than a project that doesn't work correctly.

## Before you start

Before you get started, make sure you have the following:

1. A Stripe/Lemon Squeezy account with the API keys and Webhook secret
2. A Supabase account with the API keys
3. A Mailer account with the API keys

If you don't have these, you can sign up for them. The project will not work correctly without these (well, you can fake them and deploy anyway, but it won't work correctly).

Why is Makerkit requiring these variables? I just want to deploy the project! Well, Makerkit tries quite hard to validate the project to make sure it works correctly. The environment variables are crucial for the project to work correctly. If you don't have them, the project will not work correctly. If you don't have the required services, maybe it's just not yet time to deploy the project.

## Generating the environment variables

Makerkit provides a script that will guide you through the process of generating the environment variables. To generate the environment variables, run the following command:

```bash
turbo gen env
```

This command will guide you through the process of generating the environment variables. It will ask you for the values of the environment variables and generate a `.env.local` at `turbo/generators/templates/env/.env.local`.

This file is to never be committed to the repository. It is a template file that you can use to set the environment variables in your deployment environment.

For example, if you're using Vercel, you can copy/paste the contents of the `.env.local` file into the Vercel environment variables. Depending on your deployment environment, you may need to set the environment variables in a different way - but the `.env.local` file will guide you on what to set.

## Validating the environment variables

After generating the environment variables, you can validate them by running the following command:

```bash
turbo gen validate-env
```

By default, this command will validate the environment variables in the `.env.local` file `turbo/generators/templates/env`.

The generator (at this time) validates each single variable, not the schema as a whole. It is assumed the previous step has generated the correct variables and the validation is just to ensure they are set. In the future, I plan on making the validation even smarter.

## Validate, Validate, Validate

The environment variables are crucial for the project to work correctly. If you miss any, the project will not work correctly. So please make sure to validate the environment variables before deploying the project.


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\sherpa.mdoc

---
status: "published"
title: "Deploy Next.js Supabase to sherpa.sh"
label: "Deploy to sherpa.sh"
order: 6
description: "This guide will help you deploy the Next.js SaaS boilerplate to sherpa.sh - a high performance, cost-effective platform for deploying Next.js applications."
---

Sherpa.sh is a cost-effective platform for dev teams who hate wasting time on infrastructure. 

Deploying your Next.js app to sherpa.sh is easy, and you can deploy the Next.js SaaS boilerplate to sherpa.sh by following these steps:

{% alert type="success" %}
Sherpa.sh is a bootstrapped startup. If you need help, the founder of sherpa.sh is in our discord server as user `@Zach @ sherpa.sh`.  You can message him directly there for assistance.
{% /alert %}

# Video Tutorial

Below you will find a video tutorial that will guide you through the process of deploying the Next.js SaaS boilerplate to sherpa.sh. You can also skip ahead and follow the written steps below.

{% youtube id="6K5JzPD8wKU" /%}

# Getting Started

## Import Project

1. Go to [sherpa.sh](https://www.sherpa.sh) and create a new account.
2. Link the github account/organization that contains your Makerkit project repo.

## Configure Build Settings

Configure your build settings for Makerkit.  

{% alert type="info" %}
Be sure to enter the project settings exactly. They are crucial for the project to deploy correctly.
{% /alert %}

{% img src="/assets/images/docs/sherpa-build-settings.webp" width="1744"
height="854" /%}

As you can see in the image, please make sure to:
1. use Next.js as the framework preset
2. use pnpm for the install and build commands
3. point the outpur directory to the `apps/web` folder
4. point the root directory to the `apps/web` folder

Video tutorial: [Build settings](https://youtu.be/6K5JzPD8wKU?feature=shared&t=71)

## Environment Variables

Please make sure to set all the environment variables required for the project to work correctly.

To generate the environment variables, please follow the steps in the [Environment Variables](https://makerkit.dev/docs/next-supabase-turbo/going-to-production/production-environment-variables) guide.

A production deployment should be setting the below environment variables. Failure to set the environment variables will result in the project not working correctly.

If the build fails, deep dive into the logs to see what is the issue. Our Zod configuration will validate and report any missing environment variables. To find out which environment variables are missing, please check the logs.

Video tutorial: [Environment Variables](https://youtu.be/6K5JzPD8wKU?feature=shared&t=126)

## I have more apps - how do I deploy them?

If you have multiple apps, customize the build command to point to the app being deployed.

Where `app/web` is set in the output directory and root directory, change that to the path of the app you want to deploy.

# Getting Help & Requesting Features

Sherpa.sh is a bootstrapped startup with a responsive team.  The founder of Sherpa.sh is in our discord server as user `@Zach @ sherpa.sh`.  You can message him directly with feature requests, any questions, or issues.

Otherwise, you can find the sherpa.sh documentation [here](https://sherpa-sh.gitbook.io/sherpa.sh-docs).

