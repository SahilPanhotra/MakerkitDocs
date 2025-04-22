-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\cloudflare.mdoc

---
status: "published"
title: 'Deploy Next.js Supabase to Cloudflare'
label: 'Deploy to Cloudflare'
order: 6
description: 'This guide will help you deploy the Next.js SaaS boilerplate to Cloudflare Pages using the Edge runtime.'
---

{% alert type="warning" %}
Cloudflare Workers support is temporarily impacted by this issue: [React Email/Cloudflare](https://github.com/resend/react-email/issues/1630).
{% /alert %}

To deploy the application to Cloudflare, you need to do the following:

1. Opt-in to the Edge runtime
2. Using the Cloudflare Mailer
3. Switching CMS
4. Setting Node.js Compatibility Flags
5. Environment variables

### Before you start: update to the paid Cloudflare tier

Deploying to Cloudflare requires a subscription to the Cloudflare Workers paid service due to the size limitations of the free tier. It starts at $5.

### 0. Limitations

Before you continue, **please evaluate the limitations of the Edge runtime**. The Edge runtime does not support all Node.js features, so you may need to adjust your application accordingly.

Cloudflare is cheaper and faster than many other providers, but running your application on Cloudflare Pages means not having access to the vast Node.js ecosystem.

Makerkit uses Cloudflare as a baseline, so you can deploy it to Cloudflare Pages without any issues. However, you will need to keep in mind the limitations of the Workers/Edge runtime when adding new features.

One more thing to consider is that the Edge runtime does run close to your users, but may run far from your database. Consider read replicas or other strategies to reduce latency in all situations.

If your mind is set on using Cloudflare, please follow the instructions below.

### 1. Opting into the Edge runtime

To opt-in to the Edge runtime, you need to do the following: open the root layout file of your app `apps/web/app/layout.tsx` and export the const runtime as `edge`:

```tsx
export const runtime = 'edge';
```

This will enable the Edge runtime for all your application.

### 2. Using the Resend Mailer or create a custom mailer

Since the default library `nodemailer` relies on Node.js, we cannot use it in the Edge runtime. Instead, we will Resend Mailer.

Set up SPF and DKIM records in your DNS settings.

Please follow [the Vercel Email documentation](https://github.com/Sh4yy/vercel-email?tab=readme-ov-file#setup-spf) to set up the SPF and DKIM records.

Alternatively, you can use the Resend Mailer. Set the `MAILER_PROVIDER` environment variable to `resend` in the `apps/web/.env` file:

```bash
MAILER_PROVIDER=resend
```

And provide the Resend API key:

```bash
RESEND_API_KEY=your-api-key
```

Alternatively, implement your own mailer using the abstract class we provide in the `packages/mailers` package.

This is very simple, but you need to ensure that the mailer is compatible with the Edge runtime. As usual, HTTP-based APIs are preferable.

### 3. Switching CMS

By default, Makerkit uses Keystatic as a CMS. Keystatic's local mode (which relies on the file system) is not supported in the Edge runtime. Therefore, you will need to switch to another CMS or use Keystatic's Github mode - which uses Github as data provider.

At this time, the other CMS supported is WordPress. Set `CMS_CLIENT` to `wordpress` in the `apps/web/.env` file:

```bash
CMS_CLIENT=wordpress
```

Alternatively, use the Keystatic remote mode through Github.

Please check the CMS documentation for more information.

### 4. Select "apps/web" as path to build

When prompted to select the path to build, select `apps/web` as the path to build - since our Next.js app is located in the `apps/web` directory.

If you have multiple apps, you can select the app you want to deploy.

### 5. Setting Node.js Compatibility Flags

Cloudflare requires you to set the Node.js compatibility flags. Please follow the instructions on the [Cloudflare documentation](https://developers.cloudflare.com/workers/runtime-apis/nodejs).

Please don't miss this step, as it's crucial for the application to work in the Edge runtime.

Always use the very latest compatibility flags since it supports more and more Node.js APIs.

### 6. Environment variables

When adding environment variables, make sure to add all of them from the Cloudflare dashboard instead of the `.env` file. I cannot understand why, but the Workers did not pick up the environment variables from the `.env` file. So please do make sure to add all the environment variables from the Cloudflare dashboard.


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\production-environment-variables.mdoc

---
status: "published"
title: "Generating and validating the environment variables needed for deployment | Next.js Supabase SaaS Kit"
label: "Environment Variables"
description: "Guide to generate and validate the environment variables needed for deployment in your Next.js Supabase SaaS Kit project"
order: 1
---

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
FILE PATH: kits\next-supabase-turbo\going-to-production\supabase.mdoc

---
status: "published"
title: "Deploy Supabase to Production"
label: "Deploy Supabase"
order: 1
description: "Guide on how to setup your Supabase environment for production deployment."
---

Unless you're self-hosting Supabase, you will need to create an account on Supabase. You can do so by visiting [supabase.com](https://supabase.com).

Once you have created an account, you will need to create a project. You can do so by clicking on the "Create Project" button.

Supabase will promptly create a project and the API Keys required to connect your project to the Supabase API.

{% alert type="warning" title="Please complete the following steps" %}
Not completing these steps will result in your project not working correctly. Please make sure to complete these steps before testing your project.
{% /alert %}

### Keeping your Supabase Database Password safe

When you create a new Supabase project, Supabase will prompt you to create a Database Password.

Please store the Database Password and keep it safe - you will need it to authenticate your project with the Supabase CLI later on.

### Navigating to the Supabase API settings

To retrieve the environment variables you will need to retrieve the following:

1. **Supabase URL**: this is the URL of your Supabase project.
2. **Anon Key**: this is the public key of your Supabase project. The client will use it to access your Supabase project.
3. **Service Role Key**: this is the secret key that grants administrative access to your Supabase project.

Navigate to `Project Settings` -> `API` to retrieve the Supabase URL, Anon Key, and Service Role Key.

{% img src="/assets/courses/next-turbo/supabase-api-settings.webp" width="2500" height="1262" /%}

Keep this values safe, you will need them to set up your environment variables.

### Setting up your Supabase project authentication

We need to set up the authentication for our Supabase project. This involves setting our application's URL, and a return URL for callbacks when a user logs in or signs up.

Navigate to `Authentication` -> `URL Configuration` to set up the authentication settings.

Please fill the `Site URL` and `Redirect URL` fields with your application's URL if you have it.

If you don't yet have a URL (as you may still need to deploy your project), you can fill in the URL later, but remember to update it before you go live with your project.

#### Redirect URL

The Redirect URL is the URL that Supabase will redirect the user to after they log in or sign up. This URL should be the URL of your application.

You can use the following Redirect URL format: `https://<your-url>/auth/callback**`. This is the default endpoint Makerkit uses for authentication.

### Setting up your Supabase project SMTP service

If you don't yet have an SMTP service, please skip this step. However - please remember to set up an SMTP service before you go live with your project.

Navigate to `Project Settings` -> `Authentication` -> `SMTP Settings` to set up your SMTP service.

Enable the SMTP service and fill in the SMTP settings. You can use a service like [Resend](https://resend.com/) to set up your SMTP service, which Makerkit supports natively.

This is extremely important to ensure that your users receive emails from your application, as Supabase's email service is not reliable and only meant for development purposes.

### Setting the Emails in your Supabase project

Makerkit provides with pre-designed emails that you can use in your Supabase project. You can find the emails in the `apps/web/supabase/templates` folder in your project.

These emails are nicer than the default Supabase emails and provide a better user experience. I do recommend using them in your project.

For more information, please visit the [official documentation to set up emails](/docs/next-supabase-turbo/authentication-emails).

### Linking your Supabase project

Now that you have set up your Supabase project, you need to link it to your project with the Supabase CLI.

To do this, you need to run the following command:

```bash
pnpm --filter web supabase login
```

Please follow the prompts and login to your Supabase account.

Now, you need to link your project to your Supabase project. To do this, run the following command:

```bash
pnpm --filter web supabase link
```

You can choose the project you want to link to, and the Supabase CLI will link your project to your Supabase project.

The CLI will prompt you for the Database Password you created earlier. Please enter it to authenticate your project with Supabase.

### Pushing the Migration to Supabase for production

If you have followed the previous steps, you have linked your project to your Supabase project. Now, you need to push the migration to your Supabase project.

To do this, run the following command:

```bash
pnpm --filter web supabase db push
```

The Supabase CLI will now ask you to confirm the migrations. If it all looks good (you should see the core migrations and the migration wr created for this course) please proceed.

{% img src="/assets/courses/next-turbo/supabase-webhooks.webp" width="2062" height="876" /%}

## Database Webhooks

Makerkit uses Database Webhooks in response to changes in the database.

This allows the app to intercept events and handle them for various purposes, such as sending emails after an invitation, or canceling a subscription after a user deletes their account.

As such, it is mandatory to set up the Database Webhooks in your Supabase instance for the app to work correctly.

First, you need to generate a secret and set it using the environment variable `SUPABASE_DB_WEBHOOK_SECRET`.

We use this variable in two places:
1. In the Supabase webhooks headers, so the server can authenticate the request is coming from Supabase
2. In the server, so we can authenticate the request is coming from Supabase

Please set the `SUPABASE_DB_WEBHOOK_SECRET` environment variable:

```bash
SUPABASE_DB_WEBHOOK_SECRET=**************************************************
```

{% alert title="Make the key very strong" %}
Make sure to keep the secret key secret and very complex. If someone gets access to the secret key, they can use it to send webhooks to your server.
{% /alert %}

Once you have set the environment variable, you need to deploy the Supabase DB webhooks to your Supabase instance.

Make sure to add the following header `X-Supabase-Event-Signature` with the value of the `SUPABASE_DB_WEBHOOK_SECRET` to the request.

In this way - your server will be able to authenticate the request and be sure it's coming from your Supabase instance.

As the endpoint, remember to use the `/api/db/webhook` endpoint. If your APP URL is `https://myapp.vercel.app`, the endpoint will be `https://myapp.vercel.app/api/db/webhook`. Please be sure to use your real APP URL.

#### Adding Database Webhooks from Supabase Studio

The below is only needed when going to production. The local development seed.sql script will add the webhooks for you.

While you can create a migration to add the database webhooks, you can also add them from the Supabase Studio.

Here are the steps to add the webhooks from the Supabase Studio:

1. Go to the Supabase Studio
2. Go to Database->Webhooks
3. Click on "Enable Webhooks"
4. Click on "Create a new hook"

Now, replicate the webhooks at `apps/web/supabase/seed.sql` using the UI:

1. Please remember to set the `X-Supabase-Event-Signature` header with the value of the `SUPABASE_DB_WEBHOOK_SECRET` to the request.
2. Please remember to set the endpoint to `/api/db/webhook` using your real APP URL. If your APP URL is `https://myapp.vercel.app`, the endpoint will be `https://myapp.vercel.app/api/db/webhook`.
3. Use 5000 as the timeout.

Alternatively, you can also set these using a migration - but it's not recommended since you'd need to store the secret in the migration file.

{% alert title="Use a public URL" type="warning" %}
Please make sure to use a public URL for your webhooks. If you use a private URL (such as Vercel preview URLs), you will not be able to receive webhooks from Supabase.
{% /alert %}

To ensure the URL is publicly accessible, please visit it using Incognito mode in your browser.

##### Webhooks to add

We need to add the following webhooks:

1. `delete` on `public.accounts`
2. `delete` on `public.subscriptions`
3. `insert` on `public.invitations`

Please make sure to add these webhooks to your Supabase instance.

## Set up Google Auth [Optional]

If you want to use Google Auth in your project, [you need to set up Google Auth in your Supabase project](https://supabase.com/docs/guides/auth/social-login/auth-google).

This is a little bit more convoluted as you need a Google Cloud project, so I recommend to do this later on.

However, I am writing it here as a reminder to do it before you go live with your project.


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\vercel.mdoc

---
status: "published"
title: "Deploy Next.js Supabase to Vercel"
label: "Deploy to Vercel"
order: 5
description: "This gude will help you deploy the Next.js SaaS boilerplate to Vercel - the native platform for deploying Next.js applications."
---

Deploying to Vercel should be straightforward. You can deploy the Next.js SaaS boilerplate to Vercel by following these steps:

1. Connect your Repository to Vercel
2. Configure Environment Variables: the first time this may fail if you don't yet have a custom domain connected since you cannot place it in the environment variables yet. It's fine. Make the first deployment fail, then pick the domain and add it. Redeploy.
3. Deploy the Project

Vercel should be able to automatically infer the project settings and deploy it correctly.

{% img src="/assets/images/docs/vercel-turbo-preset.webp" width="1744" height="854" /%}

As you can see in the image, please make sure to:
1. use Next.js as the framework preset
2. point the root directory to the `apps/web` folder

{% alert type="warning" %}
Please don't miss the steps above - they are crucial for the project to deploy correctly.
{% /alert %}

### Troubleshooting

In some cases, users have reported issues with the deployment to Vercel using the default parameters. If this is the case, please try the following:

1. Make sure to set the root directory to `apps/web`
2. Make sure to set the preset to Next.js
3. Manually enter the commands for installing and running the build, respectively: `pnpm i` for installing and `pnpm run build` for building the project. This is the case when Vercel tries to use `npm` instead of `pnpm`.

If the above steps don't work, please check the logs and see what the issue is. The logs should give you a hint on what is wrong.

## Environment Variables

Please make sure to set all the environment variables required for the project to work correctly.

A production deployment should be setting the below environment variables:

{% img src="/assets/images/docs/vercel-env-variables-turbo.webp" width="1694" height="1874" /%}

Failure to set the environment variables will result in the project not working correctly.

If the build fails, deep dive into the logs to see what is the issue. Our Zod configuration will validate and report any missing environment variables. To find out which environment variables are missing, please check the logs.

## Deploying to Vercel Edge Functions

If you want to deploy the project with Edge Functions, then the same steps as Cloudflare apply.

Please follow the same steps as for [Cloudflare](cloudflare) and then deploy the project to Vercel.

1. Switch to an HTTP-based mailer (Cloudflare Mailer or Resend Mailer)
2. Switch to a remote CMS (Wordpress or Keystatic Github mode)

The above should be all! By following the steps, you should be able to deploy the Next.js SaaS boilerplate to Vercel's Edge Functions with zero cold starts, better speed and lower costs!

Please be mindful of the limitations of the Edge runtime:

1. potentially higher latency to your database
2. limited Node.js features
3. limited access to the Node.js ecosystem

If you are fine with the above, then Vercel is a great choice for deploying your Next.js SaaS boilerplate.

## I have more apps - how do I deploy them?

Vercel should automatically take care of deploying the app named `web`.

If you have multiple apps, you may need to customize the build command to point to the app being deployed.

```
cd ../.. && turbo run build --filter=<app-name>
```

Please replace `<app-name>` with the name of the app you want to deploy.

For more info refer to the [Vercel documentation](https://vercel.com/docs/monorepos/turborepo).


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production.mdoc

---
status: "published"
title: "Going to Production in Next.js Supabase Turbo"
label: "Going to Production"
order: 15
description: "Going to Production in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

# Going to Production Guide

Ready to launch? Here's everything you need to know about deploying your Makerkit application:

1. [**Production Checklist**](/docs/next-supabase-turbo/checklist)
   Your comprehensive checklist for a successful production deployment.
2. [**Authentication Setup**](/docs/next-supabase-turbo/authentication)
   Configure authentication for your production environment.
3. [**Environment Variables**](/docs/next-supabase-turbo/production-environment-variables)
   Set up and manage your production environment variables.
4. [**Deployment Options**](/docs/next-supabase-turbo/deployment)
   - [Deploy to Vercel](/docs/next-supabase-turbo/vercel)
   - [Deploy to Cloudflare](/docs/next-supabase-turbo/cloudflare)
   - [Deploy to Supabase](/docs/next-supabase-turbo/supabase)
5. [**Database Setup**](/docs/next-supabase-turbo/supabase)
   Configure your production Supabase instance.

Time to ship your product! 🚀 Follow these guides to ensure a smooth deployment.

-----------------
FILE PATH: kits\next-supabase-turbo\installation\clone-repository.mdoc

---
status: "published"
title: "Clone the Next.js Supabase SaaS Kit Turbo Repository"
label: "Clone the Repository"
order: 4
description: "Clone the Next.js Supabase SaaS Kit Turbo repository to your local machine."
---

## Prerequisites

To get started with Makerkit, ensure you have the following prerequisites installed and set up:

- **Node.js 18.x or later** (recommended to use Node.js 20.x or later)
- **Docker** (required for running Supabase locally)
- **PNPM**
- **Payment Gateway account** (Stripe or Lemon Squeezy)
- **Email Service account** (optional for local development)

### Why Docker?

Docker is required for running Supabase locally. While this is not a strict requirement, it is recommended to use Docker for local development. You don't need to know Docker to use Supabase, you simply need to have it running so that you can spin up the services locally.

## Have Github configured with SSH

To clone the repository using SSH, you need to have your Github account configured with SSH. If you don't have it configured, please refer to the [official documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) to set it up.

Alternatively, you can use HTTPS to clone the repository. For example, `git clone https://github.com/makerkit/next-supabase-saas-kit-turbo`.

Another alternative is to use the Github CLI or Github Desktop to clone the repository.

## Verify your Git username

To verify you have access, you need to check that your local Git username is the same as set up in the Makerkit's Github organization.

Please run the following command to check your Git username:

```bash
git config user.username
```

If the output is not your Github username, or does not match the username registered in Makerkit's Github organization, you can set it using the following command:

```bash
git config --global user.username <your-github-username>
```

**NB**: You must replace `<your-github-username>` with your actual Github username in between quotes.

For example, if your Github username is `johndoe`, you can set it using the following command:

```bash
git config --global user.username "johndoe"
```

This is important to ensure you can run the repository.

## Cloning the Repository

Clone the repository using the following command:

```bash
git clone git@github.com:makerkit/next-supabase-saas-kit-turbo
```

**Note:** If your SSH key isn't set up, use HTTPS instead:

```bash
git clone https://github.com/makerkit/next-supabase-saas-kit-turbo
```

#### Windows Users: place the repository near the root of your drive

Some Windows users hit errors loading certain modules due to very long paths. To avoid this, try placing the repository near the root of your drive.

Avoid using OneDrive, as it can cause issues with Node.js.

### Important: Use HTTPS for All Commands if Not Using SSH

If you are not using SSH, ensure you switch to HTTPS for all Git commands, not just the clone command.

## Configuring Git Remotes

After cloning the repository, remove the original `origin` remote:

```bash
git remote rm origin
```

Add the upstream remote pointing to the original repository to pull updates:

```bash
git remote add upstream git@github.com:makerkit/next-supabase-saas-kit-turbo
```

Once you have your own repository set up, add your repository as the `origin`:

```bash
git remote add origin <your-repository-url>
```

### Keeping Your Repository Up to Date

To pull updates from the upstream repository, run the following command daily (preferably with your morning coffee ☕):

```bash
git pull upstream main
```

This ensures your repository stays up to date with the latest changes.

## 0.1. Install Pnpm

Install Pnpm globally with the following command:

```bash
npm i -g pnpm
```

## 1. Setup Dependencies

Install the necessary dependencies for the project:

```bash
pnpm i
```

With these steps completed, your development environment will be set up and ready to go! 🚀

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


