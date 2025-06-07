-----------------
FILE PATH: kits/next-supabase-turbo/going-to-production/supabase.mdoc

---
status: "published"
title: "Deploy Supabase to Production"
label: "Deploy Supabase"
order: 1
description: "Guide on how to setup your Supabase environment for production deployment."
---

{% alert type="warning" title="Please complete the following steps" %}
Not completing these steps will result in your project not working correctly. Please make sure to complete these steps before testing your project.
{% /alert %}

{% sequence title="How to deploy Supabase to Production" description="Guide on how to setup your Supabase environment for production deployment." %}

[Create a Supabase project](#create-a-supabase-project)

[Keeping your Supabase Database Password safe](#keeping-your-supabase-database-password-safe)

[Navigating to the Supabase API settings](#navigating-to-the-supabase-api-settings)

[Setting up your Supabase project authentication](#setting-up-your-supabase-project-authentication)

[Setting up your Supabase project SMTP service](#setting-up-your-supabase-project-smtp-service)

[Setting the Emails in your Supabase project](#setting-the-emails-in-your-supabase-project)

[Linking your Supabase project](#linking-your-supabase-project)

[Pushing the Migration to Supabase for production](#pushing-the-migration-to-supabase-for-production)

[Setting up the Database Webhooks](#database-webhooks)

[Set up Google Auth (Optional)](#set-up-google-auth-[optional])

{% /sequence %}

## Create a Supabase project

Unless you're self-hosting Supabase, you will need to create an account on Supabase. You can do so by visiting [supabase.com](https://supabase.com).

Once you have created an account, you will need to create a project. You can do so by clicking on the "Create Project" button.

Supabase will promptly create a project and the API Keys required to connect your project to the Supabase API.

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
FILE PATH: kits/next-supabase-turbo/going-to-production/vercel.mdoc

---
status: "published"
title: "Deploy Next.js Supabase to Vercel"
label: "Deploy to Vercel"
order: 5
description: "This gude will help you deploy the Next.js SaaS boilerplate to Vercel - the native platform for deploying Next.js applications."
---

{% sequence title="How to deploy the Next.js SaaS boilerplate to Vercel" description="This gude will help you deploy the Next.js SaaS boilerplate to Vercel - the native platform for deploying Next.js applications." %}

[Connect your Repository to Vercel](#connect-your-repository-to-vercel)

[Configure Environment Variables](#configure-environment-variables)

[Deploy the Project](#deploy-the-project)

[Troubleshooting](#troubleshooting)

[Deploying to Vercel Edge Functions](#deploying-to-vercel-edge-functions)

[I have more apps - how do I deploy them?](#i-have-more-apps---how-do-i-deploy-them)

{% /sequence %}

Deploying to Vercel should be straightforward. You can deploy the Next.js SaaS boilerplate to Vercel by following these steps:

## Connect your Repository to Vercel

{% img src="/assets/images/docs/vercel-turbo-preset.webp" width="1744" height="854" /%}

As you can see in the image, please make sure to:
1. use Next.js as the framework preset
2. point the root directory to the `apps/web` folder

{% alert type="warning" %}
Please don't miss the steps above - they are crucial for the project to deploy correctly.
{% /alert %}

## Configure Environment Variables

The first time this may fail if you don't yet have a custom domain connected since you cannot place it in the environment variables yet. It's fine. Make the first deployment fail, then pick the domain and add it. Redeploy.

Please make sure to set all the environment variables required for the project to work correctly.

A production deployment should be setting the below environment variables:

{% img src="/assets/images/docs/vercel-env-variables-turbo.webp" width="1694" height="1874" /%}

Failure to set the environment variables will result in the project not working correctly.

If the build fails, deep dive into the logs to see what is the issue. Our Zod configuration will validate and report any missing environment variables. To find out which environment variables are missing, please check the logs.

## Deploy the Project

Vercel should be able to automatically infer the project settings and deploy it correctly.

### Troubleshooting

In some cases, users have reported issues with the deployment to Vercel using the default parameters. If this is the case, please try the following:

1. Make sure to set the root directory to `apps/web`
2. Make sure to set the preset to Next.js
3. Manually enter the commands for installing and running the build, respectively: `pnpm i` for installing and `pnpm run build` for building the project. This is the case when Vercel tries to use `npm` instead of `pnpm`.

If the above steps don't work, please check the logs and see what the issue is. The logs should give you a hint on what is wrong.

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
FILE PATH: kits/next-supabase-turbo/going-to-production/vps.mdoc

---
status: "published"
title: "Deploy Next.js Supabase to a VPS"
label: "Deploy to a VPS"
order: 9
description: "This guide will help you deploy the Next.js SaaS boilerplate to a VPS."
---

If you want to self-host your application, you can deploy the Next.js SaaS boilerplate to a VPS. In this specific guide, **we will use Digital Ocean**, a popular cloud provider in the VPS space.

Much of the setup in this guide is going to similarly apply to other platforms, such as Hetzner, Linode, Vultr, etc.

## Generate the Dockerfile image in your project

In your own project, run the following command to generate the Dockerfile.

```bash
pnpm run turbo gen docker
```

This will create a `Dockerfile` in the root of your project. In addition, it will install some dependencies that may be required to build the project.

## 0. Create a Digital Ocean Droplet (or other platform)

From Digital Ocean (or any other platform), create a new Droplet. Use the specifications and settings you prefer.

## 1. Install Docker

To Install Docker, please follow the [official Docker installation guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04) in the Digital Ocean Community.

## 2. Add Firewall Rules

Add a firewall rule to allow inbound traffic on port 3000.

Please refer to the [Digital Ocean documentation](https://docs.digitalocean.com/products/networking/firewalls/how-to/create/) for more information.

## 3. Building the Docker image

We can **build the Docker image on the Droplet** or **build the image locally** and push it to a container registry.

If you don't have a powerful VPS, **it's recommended to build the image locally and push it to a container registry** (such as DockerHub, Github Container Registry, etc.).

Ideally, you can set up a Github Actions workflow to build the image and push it to a container registry automatically when you push a new commit to the repository.

In the next steps, we will build the image on the Digital Ocean Droplet. Only follow this section if you have a powerful VPS, otherwise the build will fail due to lack of memory/CPU.

### 3.1. Connect with Github repository

#### 3.1.1. Create a Personal Access Token

First, you need to create a Personal Access Token in Github to allow access to the repository. We will reference this token below as `<PAT>`.

#### 3.1.2. Pull the repository

Next, you need to pull the repository to the Digital Ocean Droplet.

```bash
git clone https://<PAT>@github.com/username/repo.git
```

NB: Replace `<PAT>` with the Personal Access Token you created in step 3.1.

#### 3.1.3. Install Node.js and dependencies

Navigate to the repository and install the dependencies.

First, set up Node.js and pnpm.

Download and install `nvm`:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```

Then run:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

You can now install Node.js:

```
nvm install 20
```

Next, install PNPM globally:

```bash
npm install -g pnpm
```

Now, we can install the dependencies:

```bash
cd repo # Replace repo with the name of the repository
pnpm install
```

## 4. Create the Dockerfile

```bash
pnpm run turbo gen docker
```

## 5. Build the Docker Image on the Digital Ocean Droplet

Navigate to the repository and build the Docker image.
```bash
docker build -t next-supabase-turbo .
```

If the build fails, it's possible it's out of memory. You can try to add more memory to the Digital Ocean Droplet.

```bash
docker build -t --memory 2g -e next-supabase-turbo .
```

If the Droplet size allows it, you can try to add more memory to the build.

## 6. Environment Variables

Using the Dev Tools, generate the environment variables for the production environment. Copy the environment variables and paste them in the `apps/web/.env.production.local` file within the repository. This file gets never committed to the repository since it's ignored by the `.gitignore` file.

## 7. Run the Docker Container

First, ensure that the `.env.production.local` file is present in the `apps/web` directory.

Then, run the following command to run the Docker container:

```bash
docker run -d -p 3000:3000 next-supabase-turbo --env-file apps/web/.env.production.local
```

## 8. Access the Application

You can now access the application at `http://<your-docker-host>:3000`.

## 9. Next Steps

Now that your application is running, you can start to configure the production environment. You will want to do the following:

- Configure Nginx to reverse proxy requests to the Docker container
- Add a custom domain name to Digital Ocean
- Set up a Firewall in Digital Ocean to allow inbound traffic on port 3000
- Set up a SSL certificate for your custom domain name with Let's Encrypt

After doing this, you will have a production-ready application that is fully self-hosted!

-----------------
FILE PATH: kits/next-supabase-turbo/going-to-production.mdoc

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
FILE PATH: kits/next-supabase-turbo/installation/clone-repository.mdoc

---
status: "published"
title: "Clone the Next.js Supabase SaaS Kit Turbo Repository"
label: "Clone the Repository"
order: 4
description: "Clone the Next.js Supabase SaaS Kit Turbo repository to your local machine."
---

{% sequence title="Cloning the repository" description="Clone the Next.js Supabase SaaS Kit Turbo repository to your local machine." %}

[Prerequisites](#prerequisites)

[Have Github configured with SSH](#have-github-configured-with-ssh)

[Clone the repository](#cloning-the-repository)

[Install Pnpm](#install-pnpm)

[Setup Dependencies](#setup-dependencies)

[Automatic Setup](#automatic-setup-(recommended))

[Manual Setup](#manual-setup-(if-automatic-setup-fails))

{% /sequence %}

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

## Install Pnpm

Install Pnpm globally with the following command:

```bash
npm i -g pnpm
```

## Setup Dependencies

Install the necessary dependencies for the project:

```bash
pnpm i
```

## Automatic Setup (recommended)

Makerkit provides a script to automatically clone the repository and setup the project. 

If you have a repository created in version 2.9.0 or later, you can use the following command to setup the project (otherwise, you can manually setup the project by following the steps below):

```bash
pnpm turbo gen setup
```

If anything goes wrong or you have a repository created below version 2.9.0, you can manually setup the project by following the steps below.

## Manual Setup (if automatic setup fails)

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

## Keeping Your Repository Up to Date

To pull updates from the upstream repository, run the following command daily (preferably with your morning coffee ☕):

```bash
git pull upstream main
```

This ensures your repository stays up to date with the latest changes.

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
FILE PATH: kits/next-supabase-turbo/installation/code-health.mdoc

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


