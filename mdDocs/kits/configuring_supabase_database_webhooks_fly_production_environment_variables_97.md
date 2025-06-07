-----------------
FILE PATH: kits/remix-supabase-turbo/going-to-production/configuring-supabase-database-webhooks.mdoc

---
status: "published"

title: "Setting the Database Webhooks required to run the Remix Supabase SaaS kit"
label: "Database Webhooks"
description: "Database Webhooks from Supabase allow our app to intercept events and handle them. Learn how to setup database webhooks."
order: 2
---


Makerkit uses Database Webhooks in response to changes in the database. This allows the app to intercept events and handle them for various purposes, such as sending emails after an invitation, or canceling a subscription after a user deletes their account. As such, it is mandatory to set up the Database Webhooks in your Supabase instance for the app to work correctly.

First, you need to set a secret `SUPABASE_DB_WEBHOOK_SECRET` that your server and your Supabase instance will share to authenticate the requests.

```bash
SUPABASE_DB_WEBHOOK_SECRET=**************************************************
```

Make it a strong secret key - and make sure to keep it secret!

Now, you need to deploy the Supabase DB webhooks to your Supabase instance.

Please copy the webhooks (written with Postgres SQL) from apps/web/supabase/seed.sql and make sure to replicate them to the Supabase instance.

Make sure to add the following header `X-Supabase-Event-Signature` with the value of the `SUPABASE_DB_WEBHOOK_SECRET` to the request.

In this way - your server will be able to authenticate the request and be sure it's coming from your Supabase instance.

As the endpoint, remember to use the `/api/db/webhook` endpoint. If your APP URL is `https://myapp.vercel.app`, the endpoint will be `https://myapp.vercel.app/api/db/webhook`. Please be sure to use your real APP URL.

#### Adding Database Webhooks from Supabase Studio

The below is only needed when going to production. The local development seed.sql script will add the webhooks for you.

While you can create a migration to add the database webhooks, you can also add them from the Supabase Studio.

1. Go to the Supabase Studio
2. Go to Database->Webhooks
3. Click on "Enable Webhooks"
4. Click on "Create a new hook"

Now, replicate the webhooks at `apps/web/supabase/seed.sql` using the UI:

1. Please remember to set the `X-Supabase-Event-Signature` header with the value of the `SUPABASE_DB_WEBHOOK_SECRET` to the request.
2. Please remember to set the endpoint to `/api/db/webhook` using your real APP URL. If your APP URL is `https://myapp.vercel.app`, the endpoint will be `https://myapp.vercel.app/api/db/webhook`.
3. Use 5000 as the timeout.

Alternatively, you can also set these using a migration - but it's not recommended since you'd need to store the secret in the migration file.

##### Webhooks to add

We need to add the following webhooks:

1. `delete` on `public.accounts`
2. `delete` on `public.subscriptions`
3. `insert` on `public.invitations`

Please make sure to add these webhooks to your Supabase instance.


-----------------
FILE PATH: kits/remix-supabase-turbo/going-to-production/fly.mdoc

---
status: "published"

title: 'Deploy Remix Supabase Turbo to Fly.io'
label: 'Deploy to Fly.io'
order: 7
description: 'Guide to deploy the Remix SaaS boilerplate to Fly.io'
---


This guide was shared by a community member to deploy the Remix Supabase Turbo Kit to Fly.io. 

First, create the `fly.toml` in the root directory (configure fly server to your needs):

```toml
app = "remix-turbo-kit"
primary_region = "lax"
kill_signal = "SIGINT"
kill_timeout = 5

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"

[experimental]
  allowed_public_ports = []
  auto_rollback = true

[[services]]
  http_checks = []
  internal_port = 3000
  processes = ["app"]
  protocol = "tcp"
  script_checks = []
  [services.concurrency]
    hard_limit = 25
    soft_limit = 20
  [[services.ports]]
    force_https = true
    handlers = ["http"]
    port = 80
  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443
  [[services.tcp_checks]]
    grace_period = "1s"
    interval = "15s"
    restart_limit = 0
    timeout = "2s"
```

then we add this dockerfile to your root:

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1: Builder
ARG NODE_VERSION=20.10.0
FROM node:${NODE_VERSION}-slim AS builder

# Set the working directory in the container
WORKDIR /app

# Install packages needed to build node modules
RUN apt-get update -qq && \
    apt-get install -y python-is-python3 pkg-config build-essential

# Install pnpm
RUN npm install -g pnpm

# Copy workspace files
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

# Install dependencies for the entire workspace
RUN pnpm install

# Change working directory to web app
WORKDIR /app/apps/web

# Copy all files
COPY . .

# Install dependencies for the web app
RUN pnpm install

# Define build arguments
ARG NODE_ENV
ARG VITE_SITE_URL
ARG VITE_PRODUCT_NAME
ARG VITE_SITE_TITLE
ARG VITE_SITE_DESCRIPTION
ARG VITE_DEFAULT_THEME_MODE
ARG VITE_DEFAULT_LOCALE
ARG VITE_AUTH_PASSWORD
ARG VITE_AUTH_MAGIC_LINK
ARG CONTACT_EMAIL
ARG VITE_ENABLE_THEME_TOGGLE
ARG VITE_ENABLE_PERSONAL_ACCOUNT_DELETION
ARG VITE_ENABLE_PERSONAL_ACCOUNT_BILLING
ARG VITE_ENABLE_TEAM_ACCOUNTS
ARG VITE_ENABLE_TEAM_ACCOUNT_DELETION
ARG VITE_ENABLE_TEAM_ACCOUNTS_BILLING
ARG VITE_ENABLE_TEAM_ACCOUNTS_CREATION
ARG VITE_ENABLE_NOTIFICATIONS
ARG VITE_REALTIME_NOTIFICATIONS
ARG VITE_SUPABASE_URL
ARG VITE_SUPABASE_ANON_KEY
ARG SUPABASE_SERVICE_ROLE_KEY
ARG VITE_BILLING_PROVIDER
ARG VITE_STRIPE_PUBLISHABLE_KEY
ARG STRIPE_SECRET_KEY
ARG STRIPE_WEBHOOK_SECRET
ARG VITE_CMS_CLIENT
ARG VITE_KEYSTATIC_CONTENT_PATH
ARG VITE_LOCALES_PATH
ARG MAILER_PROVIDER
ARG EMAIL_SENDER
ARG EMAIL_HOST
ARG EMAIL_PORT
ARG EMAIL_USER
ARG EMAIL_PASSWORD
ARG EMAIL_TLS
ARG GADGET_SECRET_KEY
ARG VITE_TEAM_NAVIGATION_STYLE
ARG VITE_USER_NAVIGATION_STYLE
ARG VITE_THEME_COLOR
ARG VITE_THEME_COLOR_DARK
ARG SIGN_IN_PATH
ARG SIGN_UP_PATH
ARG TEAM_ACCOUNTS_HOME_PATH
ARG INVITATION_PAGE_PATH
ARG VITE_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX

# Set environment variables
ENV NODE_ENV=${NODE_ENV}
ENV VITE_SITE_URL=${VITE_SITE_URL}
ENV VITE_PRODUCT_NAME=${VITE_PRODUCT_NAME}
ENV VITE_SITE_TITLE=${VITE_SITE_TITLE}
ENV VITE_SITE_DESCRIPTION=${VITE_SITE_DESCRIPTION}
ENV VITE_DEFAULT_THEME_MODE=${VITE_DEFAULT_THEME_MODE}
ENV VITE_DEFAULT_LOCALE=${VITE_DEFAULT_LOCALE}
ENV VITE_AUTH_PASSWORD=${VITE_AUTH_PASSWORD}
ENV VITE_AUTH_MAGIC_LINK=${VITE_AUTH_MAGIC_LINK}
ENV CONTACT_EMAIL=${CONTACT_EMAIL}
ENV VITE_ENABLE_THEME_TOGGLE=${VITE_ENABLE_THEME_TOGGLE}
ENV VITE_ENABLE_PERSONAL_ACCOUNT_DELETION=${VITE_ENABLE_PERSONAL_ACCOUNT_DELETION}
ENV VITE_ENABLE_PERSONAL_ACCOUNT_BILLING=${VITE_ENABLE_PERSONAL_ACCOUNT_BILLING}
ENV VITE_ENABLE_TEAM_ACCOUNTS=${VITE_ENABLE_TEAM_ACCOUNTS}
ENV VITE_ENABLE_TEAM_ACCOUNT_DELETION=${VITE_ENABLE_TEAM_ACCOUNT_DELETION}
ENV VITE_ENABLE_TEAM_ACCOUNTS_BILLING=${VITE_ENABLE_TEAM_ACCOUNTS_BILLING}
ENV VITE_ENABLE_TEAM_ACCOUNTS_CREATION=${VITE_ENABLE_TEAM_ACCOUNTS_CREATION}
ENV VITE_ENABLE_NOTIFICATIONS=${VITE_ENABLE_NOTIFICATIONS}
ENV VITE_REALTIME_NOTIFICATIONS=${VITE_REALTIME_NOTIFICATIONS}
ENV VITE_SUPABASE_URL=${VITE_SUPABASE_URL}
ENV VITE_SUPABASE_ANON_KEY=${VITE_SUPABASE_ANON_KEY}
ENV SUPABASE_SERVICE_ROLE_KEY=${SUPABASE_SERVICE_ROLE_KEY}
ENV VITE_BILLING_PROVIDER=${VITE_BILLING_PROVIDER}
ENV VITE_STRIPE_PUBLISHABLE_KEY=${VITE_STRIPE_PUBLISHABLE_KEY}
ENV STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
ENV STRIPE_WEBHOOK_SECRET=${STRIPE_WEBHOOK_SECRET}
ENV VITE_CMS_CLIENT=${VITE_CMS_CLIENT}
ENV VITE_KEYSTATIC_CONTENT_PATH=${VITE_KEYSTATIC_CONTENT_PATH}
ENV VITE_LOCALES_PATH=${VITE_LOCALES_PATH}
ENV MAILER_PROVIDER=${MAILER_PROVIDER}
ENV EMAIL_SENDER=${EMAIL_SENDER}
ENV EMAIL_HOST=${EMAIL_HOST}
ENV EMAIL_PORT=${EMAIL_PORT}
ENV EMAIL_USER=${EMAIL_USER}
ENV EMAIL_PASSWORD=${EMAIL_PASSWORD}
ENV EMAIL_TLS=${EMAIL_TLS}
ENV GADGET_SECRET_KEY=${GADGET_SECRET_KEY}
ENV VITE_TEAM_NAVIGATION_STYLE=${VITE_TEAM_NAVIGATION_STYLE}
ENV VITE_USER_NAVIGATION_STYLE=${VITE_USER_NAVIGATION_STYLE}
ENV VITE_THEME_COLOR=${VITE_THEME_COLOR}
ENV VITE_THEME_COLOR_DARK=${VITE_THEME_COLOR_DARK}
ENV SIGN_IN_PATH=${SIGN_IN_PATH}
ENV SIGN_UP_PATH=${SIGN_UP_PATH}
ENV TEAM_ACCOUNTS_HOME_PATH=${TEAM_ACCOUNTS_HOME_PATH}
ENV INVITATION_PAGE_PATH=${INVITATION_PAGE_PATH}
ENV VITE_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX=${VITE_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX}

# Build the web app
RUN pnpm run build

# Stage 2: Runner
FROM node:${NODE_VERSION}-slim AS runner

# Set the working directory in the container
WORKDIR /app

# Copy built application from the builder stage
COPY --from=builder /app/apps/web /app

# Install pnpm in the final stage to ensure it's available for runtime
RUN npm install -g pnpm
RUN pnpm install --prod

# Expose the port the app runs on
EXPOSE 8080

# Start the server by default, this can be overwritten at runtime
CMD ["pnpm", "start"]
```

Lastly, add this workflow file for the deployment:

```yaml
name: Fly Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy app
    runs-on: ubuntu-latest
    concurrency: deploy-group
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: |
          flyctl deploy --remote-only \
            --build-arg NODE_ENV="${{ vars.NODE_ENV }}" \
            --build-arg VITE_SITE_URL="${{ vars.VITE_SITE_URL }}" \
            --build-arg VITE_PRODUCT_NAME="${{ vars.VITE_PRODUCT_NAME }}" \
            --build-arg VITE_SITE_TITLE="${{ vars.VITE_SITE_TITLE }}" \
            --build-arg VITE_SITE_DESCRIPTION="${{ vars.VITE_SITE_DESCRIPTION }}" \
            --build-arg VITE_DEFAULT_THEME_MODE="${{ vars.VITE_DEFAULT_THEME_MODE }}" \
            --build-arg VITE_DEFAULT_LOCALE="${{ vars.VITE_DEFAULT_LOCALE }}" \
            --build-arg VITE_AUTH_PASSWORD="${{ vars.VITE_AUTH_PASSWORD }}" \
            --build-arg VITE_AUTH_MAGIC_LINK="${{ vars.VITE_AUTH_MAGIC_LINK }}" \
            --build-arg CONTACT_EMAIL="${{ secrets.CONTACT_EMAIL }}" \
            --build-arg VITE_ENABLE_THEME_TOGGLE="${{ vars.VITE_ENABLE_THEME_TOGGLE }}" \
            --build-arg VITE_ENABLE_PERSONAL_ACCOUNT_DELETION="${{ vars.VITE_ENABLE_PERSONAL_ACCOUNT_DELETION }}" \
            --build-arg VITE_ENABLE_PERSONAL_ACCOUNT_BILLING="${{ vars.VITE_ENABLE_PERSONAL_ACCOUNT_BILLING }}" \
            --build-arg VITE_ENABLE_TEAM_ACCOUNTS="${{ vars.VITE_ENABLE_TEAM_ACCOUNTS }}" \
            --build-arg VITE_ENABLE_TEAM_ACCOUNT_DELETION="${{ vars.VITE_ENABLE_TEAM_ACCOUNT_DELETION }}" \
            --build-arg VITE_ENABLE_TEAM_ACCOUNTS_BILLING="${{ vars.VITE_ENABLE_TEAM_ACCOUNTS_BILLING }}" \
            --build-arg VITE_ENABLE_TEAM_ACCOUNTS_CREATION="${{ vars.VITE_ENABLE_TEAM_ACCOUNTS_CREATION }}" \
            --build-arg VITE_ENABLE_NOTIFICATIONS="${{ vars.VITE_ENABLE_NOTIFICATIONS }}" \
            --build-arg VITE_REALTIME_NOTIFICATIONS="${{ vars.VITE_REALTIME_NOTIFICATIONS }}" \
            --build-arg VITE_SUPABASE_URL="${{ vars.VITE_SUPABASE_URL }}" \
            --build-arg VITE_SUPABASE_ANON_KEY="${{ vars.VITE_SUPABASE_ANON_KEY }}" \
            --build-arg SUPABASE_SERVICE_ROLE_KEY="${{ secrets.SUPABASE_SERVICE_ROLE_KEY }}" \
            --build-arg VITE_BILLING_PROVIDER="${{ vars.VITE_BILLING_PROVIDER }}" \
            --build-arg VITE_STRIPE_PUBLISHABLE_KEY="${{ vars.VITE_STRIPE_PUBLISHABLE_KEY }}" \
            --build-arg STRIPE_SECRET_KEY="${{ secrets.STRIPE_SECRET_KEY }}" \
            --build-arg STRIPE_WEBHOOK_SECRET="${{ secrets.STRIPE_WEBHOOK_SECRET }}" \
            --build-arg VITE_CMS_CLIENT="${{ vars.VITE_CMS_CLIENT }}" \
            --build-arg VITE_KEYSTATIC_CONTENT_PATH="${{ vars.VITE_KEYSTATIC_CONTENT_PATH }}" \
            --build-arg VITE_LOCALES_PATH="${{ vars.VITE_LOCALES_PATH }}" \
            --build-arg MAILER_PROVIDER="${{ secrets.MAILER_PROVIDER }}" \
            --build-arg EMAIL_SENDER="${{ secrets.EMAIL_SENDER }}" \
            --build-arg EMAIL_HOST="${{ secrets.EMAIL_HOST }}" \
            --build-arg EMAIL_PORT="${{ secrets.EMAIL_PORT }}" \
            --build-arg EMAIL_USER="${{ secrets.EMAIL_USER }}" \
            --build-arg EMAIL_PASSWORD="${{ secrets.EMAIL_PASSWORD }}" \
            --build-arg EMAIL_TLS="${{ secrets.EMAIL_TLS }}" \
            --build-arg VITE_TEAM_NAVIGATION_STYLE="${{ vars.VITE_TEAM_NAVIGATION_STYLE }}" \
            --build-arg VITE_USER_NAVIGATION_STYLE="${{ vars.VITE_USER_NAVIGATION_STYLE }}" \
            --build-arg VITE_THEME_COLOR="${{ vars.VITE_THEME_COLOR }}" \
            --build-arg VITE_THEME_COLOR_DARK="${{ vars.VITE_THEME_COLOR_DARK }}" \
            --build-arg SIGN_IN_PATH="${{ vars.SIGN_IN_PATH }}" \
            --build-arg SIGN_UP_PATH="${{ vars.SIGN_UP_PATH }}" \
            --build-arg TEAM_ACCOUNTS_HOME_PATH="${{ vars.TEAM_ACCOUNTS_HOME_PATH }}" \
            --build-arg INVITATION_PAGE_PATH="${{ vars.INVITATION_PAGE_PATH }}" \
            --build-arg VITE_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX="${{ vars.VITE_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX }}"
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

Configure your secrets and environment variables in GitHub Actions.

Note: It's recommended to use GitHub's secrets and variables for this deployment process, as Fly.io's secrets may not function as expected in this context.

Please review the list of environment variables in the deployment command above and adjust them according to your project's specific requirements. Remove any variables that are not applicable to your setup.

Ensure all necessary secrets and variables are properly set in your GitHub repository settings before running the deployment workflow.

-----------------
FILE PATH: kits/remix-supabase-turbo/going-to-production/production-environment-variables.mdoc

---
status: "published"
title: "Generating and validating the environment variables needed for deployment | Remix Supabase SaaS Kit"
label: "Environment Variables"
description: "Guide to generate and validate the environment variables needed for deployment in your Remix Supabase SaaS Kit project"
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

This command will guide you through the process of generating the environment variables. It will ask you for the values of the environment variables and generate a `.env` at `turbo/generators/templates/env/.env`.

This file is to never be committed to the repository. It is a template file that you can use to set the environment variables in your deployment environment.

For example, if you're using Vercel, you can copy/paste the contents of the `.env` file into the Vercel environment variables. Depending on your deployment environment, you may need to set the environment variables in a different way - but the `.env` file will guide you on what to set.

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
FILE PATH: kits/remix-supabase-turbo/going-to-production/vercel.mdoc

---
status: "published"

title: "Deploy Remix Supabase to Vercel"
label: "Deploy to Vercel"
order: 6
description: "Guide to deploy the Remix SaaS boilerplate to Vercel"
---


Deploying to Vercel should be straightforward. You can deploy the Remix SaaS boilerplate to Vercel by following these steps:

1. Connect your Repository to Vercel
2. Configure Environment Variables: the first time this may fail if you don't yet have a custom domain connected since you cannot place it in the environment variables yet. It's fine. Make the first deployment fail, then pick the domain and add it. Redeploy.
3. Deploy the Project

## Connect your Repository to Vercel

To deploy the Remix SaaS boilerplate to Vercel, you need to connect your repository to Vercel.

## Configure Environment Variables

Add the environment variables required for the project to work correctly. You can build them from [our environment variables generation](production-environment-variables).

## Set the correct Preset and Root Directory

Please make sure to set the "Remix" preset in the Vercel settings and to set the root directory to `apps/web`.

{% alert type="warning" %}
Failure to set the Preset to "Remix" will result in the project not working correctly.
{% /alert %}

## Update the Preset in your Vite Config

Please do not use the Vercel preset in your Vite config. It breaks because it is not able to detect our index page (_marketing._index) correctly.

## Environment Variables

Please make sure to set all the environment variables required for the project to work correctly.

A production deployment should be setting the below environment variables:

{% img src="/assets/images/docs/vercel-env-variables-turbo.webp" width="1694" height="1874" /%}

Failure to set the environment variables will result in the project not working correctly.

## Deploying to Vercel Edge Functions

If you want to deploy the project with Edge Functions, then the same steps as Cloudflare apply.

Please follow the same steps as for [Cloudflare](cloudflare) and then deploy the project to Vercel.

1. Switch to an HTTP-based mailer (Cloudflare Mailer or Resend Mailer)
2. Switch to a remote CMS (Wordpress or Keystatic Github mode)

The above should be all! By following the steps, you should be able to deploy the Remix SaaS boilerplate to Vercel's Edge Functions with zero cold starts, better speed and lower costs!

Please be mindful of the limitations of the Edge runtime:

1. potentially higher latency to your database
2. limited Node.js features
3. limited access to the Node.js ecosystem

If you are fine with the above, then Vercel is a great choice for deploying your Remix SaaS boilerplate.

## I have more apps - how do I deploy them?

Vercel should automatically take care of deploying the app named `web`.

If you have multiple apps, you may need to customize the build command to point to the app being deployed.

```
cd ../.. && turbo run build --filter=<app-name>
```

Please replace `<app-name>` with the name of the app you want to deploy.

For more info refer to the [Vercel documentation](https://vercel.com/docs/monorepos/turborepo).


-----------------
FILE PATH: kits/remix-supabase-turbo/going-to-production.mdoc

---
status: "published"
title: "Going to Production in Remix Supabase Turbo"
label: "Going to Production"
order: 15
description: "Going to Production in Remix Supabase Turbo"
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
FILE PATH: kits/remix-supabase-turbo/installation/clone-repository.mdoc

---
status: "published"
title: "Clone the Remix Supabase SaaS Kit Turbo Repository"
label: "Clone the Repository"
order: 3
description: "Clone the Remix Supabase SaaS Kit Turbo repository to your local machine."
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
git clone git@github.com:makerkit/remix-supabase-saas-kit-turbo
```

NB: If your SSH key isn't set - then use the HTTPS.

```bash
git clone https://github.com/makerkit/remix-supabase-saas-kit-turbo
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
git remote add upstream git@github.com:makerkit/remix-supabase-saas-kit-turbo
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
FILE PATH: kits/remix-supabase-turbo/installation/code-health.mdoc

---
status: "published"
label: "Code Health and Testing"
title: "Code Health and Testing in Makerkit. Set up Github Actions, pre-commit hooks | Remix Supabase SaaS Kit"
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
FILE PATH: kits/remix-supabase-turbo/installation/common-commands.mdoc

---
status: "published"

title: 'Common commands you need to know for the Remix Supabase Turbo Starter Kit'
label: 'Common Commands'
order: 5
description: 'Learn about the common commands you need to know to work with the Remix Supabase Turbo Starter Kit.'
---


Here are some common commands you need to know to work with the Remix Supabase Turbo Starter Kit.

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
pnpm --filter web supabase link
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
FILE PATH: kits/remix-supabase-turbo/installation/conventions.mdoc

---
status: "published"
description: "A collection of conventions used in the core kit"
title: "Conventions in the Remix Supabase Turbo Starter Kit"
label: "Conventions"
order: 7
---


Below are some of the conventions used in the Remix Supabase Turbo Starter Kit.

---


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

The files localed in `server` folders are to be assumed as server-side code. They are not meant to be used in the client-side code. This helps everyone understanding where the code is meant to be run, since the lines are very blurry in Remix.

For example, you will find the server code related to a part of the app at `_lib/server` folder.

