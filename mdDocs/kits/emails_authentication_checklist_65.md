-----------------
FILE PATH: kits\react-router-supabase-turbo\emails.mdoc

---
status: "published"
title: "Emails in React Router Supabase Turbo"
label: "Emails"
order: 11
description: "Emails in React Router Supabase Turbo"
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
FILE PATH: kits\react-router-supabase-turbo\going-to-production\authentication.mdoc

---
status: "published"
title: "Setting the Supabase Auth Settings in Production | React Router Supabase SaaS Kit"
label: "Authentication"
description: "Configuring Authentication's production configuration in the React Router Supabase SaaS kit"
order: 4
---

Supabase needs a few settings to be configured in their Dashboard to work correctly.

This guide will walk you through the steps to get your Supabase authentication setup in your Production environment. The dev environment does not require any configuration.

{% alert type="warning" %}
Skipping this step will result in your users not being able to login or sign up.
{% /alert %}

## Authentication URLs

The first thing you need to do is to set the authentication URLs in the Supabase Dashboard. These URLs are used to redirect users to the correct page after they have logged in or signed up.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Authentication** tab.
4. Click on **URL Configuration**.
5. Add your Site URL to the **Site URL** field. This is the URL of your MakerKit site (e.g. `https://my-site.com`).
6. Add your Redirect URLs to the **Redirect URLs** field. This is the URL of your MakerKit site with `/auth/callback` appended to it (e.g. `https://my-site.com/auth/callback`).

Failure to set these URLs will result in your users not being able to log in or sign up.

Please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp) for more information.

## Custom SMTP

Supabase's SMTP settings have low limits and low deliverability. If you want your emails to go out and be delivered, please remember to set an alternative SMTP provider in the Supabase settings.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Project Settings** tab.
4. Click on **Auth**.
5. Tweak the `SMTP Settings` settings to your liking according to your provider's documentation.

### Troubleshooting

If you are having authentication issues, ensure that the Site URL and Redirect URLs are correct. If you are using a custom domain, ensure that you are using the correct domain in the Site URL and Redirect URLs.

{% alert type="warning" %}
NB: if your domain includes "www", ensure you include it in the Site URL and Redirect URLs. If your domain does not
  include "www", ensure you do not include it in the Site URL and Redirect URLs. If these do not match, your users will
  not be able to login.
{% /alert %}

If something is still not working, please open a support ticket with any useful information (such as server logs).


-----------------
FILE PATH: kits\react-router-supabase-turbo\going-to-production\checklist.mdoc

---
status: "published"
title: "Checklist for deploying your React Router Supabase SaaS to Production"
label: "Checklist"
description: "Let's deploy your React Router Supabase SaaS app to production!"
order: 0
---

When you are ready to go to production, please follow the checklist below. This is an overview, a more detailed guide will be provided in the future.

{% alert type="warning" title="Please complete the following steps" %}
Not completing these steps will result in your project not working correctly. Please make sure to complete these steps before testing your project.
{% /alert %}

This could take a couple of hours and some trial and error, so buckle up - you're almost there!

1. **Create a Supabase project**. [Link the project locally](https://supabase.com/docs/guides/cli/local-development#link-your-project) using the Supabase CLI (`supabase link`).
2. **Migrations**: [Push the migrations](https://supabase.com/docs/guides/cli/local-development#link-your-project) to the remote Supabase instance (`supabase db push`).
3. **Auth**: [Set your APP URL in the Supabase project settings](https://supabase.com/docs/guides/auth/redirect-urls). This is required for the OAuth flow. Make sure to add the path `/auth/callback` to the allowed URLs. If you don't have it yet, wait.
4. **Auth Providers**: [Set the OAuth providers in the Supabase](https://supabase.com/docs/guides/auth/social-login) project settings. If you use Google Auth, make sure to set it up. This requires creating a Google Cloud project and setting up the OAuth credentials.
5. **Auth Emails**: It is very much recommended to update the auth emails using the [following documentation](authentication-emails). The kit already implements the `confirm` route, but you need to update the emails in your Supabase settings.
6. **Deploy React Router**: Deploy the React Router app to Vercel or any another hosting provider. Copy the URL and set it in the Supabase project settings.
7. **Environment Variables**: The initial deploy **will likely fail** because you may not yet have a URL to set in your environment variables. This is normal. Once you have the URL, set the URL in the environment variables and redeploy.
8. **Webhooks**: [Set the DB Webhooks in Supabase](configuring-supabase-database-webhooks) pointing against your React Router app at `/api/db/webhooks`.
9. **Emails**: Get some SMTP details from an email service provider like SendGrid or Mailgun or Resend and configure the emails in both the Environment Variables and the [Supabase project settings](https://supabase.com/docs/guides/auth/auth-smtp).
10. **Billing**: Create a Stripe/Lemon Squeezy account, make sure to update the environment variables with the correct values. Point webhooks from these to `/api/billing/webhook`. Please use the relative documentation for more details.

Other minor things to consider:

1. Update the legal pages with your company's information (privacy policy, terms of service, etc.).
2. Remove the placeholder blog and documentation content / or replace it with your own
3. Update the favicon and logo with your own branding
4. Update the FAQ and other static content with your own information

---


1. **Create a Supabase Project**
   - **Why it's necessary:** Creating a Supabase project is the foundational step to set up your database, authentication, and storage services in the cloud. This will serve as the backend for your SaaS application.
   - **Action:** Create a new Supabase project in the Supabase dashboard. Once created, [link the project locally](https://supabase.com/docs/guides/cli/local-development#link-your-project) using the Supabase CLI:
     ```bash
     supabase link
     ```
2. **Migrations**
   - **Why it's necessary:** Pushing database migrations ensures that your database schema in the remote Supabase instance is configured to match Makerkit's requirements. This step is crucial for the application to function correctly.
   - **Action:** Push the database migrations to your remote Supabase instance:
     ```bash
     supabase db push
     ```
3. **Auth Configuration**
   - **Why it's necessary:** Setting your APP URL in the Supabase project settings is critical for enabling OAuth flows and redirecting users correctly during authentication.
   - **Action:** [Set your APP URL in the Supabase project settings](https://supabase.com/docs/guides/auth/redirect-urls). Add the path `/auth/callback` to the allowed URLs.
4. **OAuth Providers**
   - **Why it's necessary:** Configuring OAuth providers like Google ensures that users can log in using their existing accounts, enhancing user convenience and security. This is all done externally, in both Google and Supabase - not in the application code.
   - **Action:** [Set up the OAuth providers](https://supabase.com/docs/guides/auth/social-login) in your Supabase project settings. For Google Auth, create a Google Cloud project and set up the OAuth credentials.
5. **Auth Emails**
   - **Why it's necessary:** To provide a correct user experience with Makerkit's SSR authentication, you need to update the authentication emails to include the token hash and prevent errors usually related to PKCE - i.e when users click on the email and are redirected to a different browser - resulting in an error.
   - **Action:** Update the authentication emails using the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp). The kit implements the `confirm` route, but you need to update the email templates in your Supabase settings.
6. **Deploy React Router**
   - **Why it's necessary:** Because your users are waiting! Deploying your React Router app to a hosting provider makes it accessible to users worldwide, allowing them to interact with your application.
   - **Action:** Deploy your React Router app to Vercel or another hosting provider. Copy the deployment URL and set it in your Supabase project settings.
7. **Environment Variables**
   - **Why it's necessary:** Setting the correct environment variables is essential for the application to function correctly. These variables include API keys, database URLs, and other configuration details required for your app to connect to various services.
   - **Action:** Please [generate the environment variables using our script](production-environment-variables) and then add them to your hosting provider's environment variables. Redeploy the app once you have the URL to set in the environment variables.
8. **Webhooks**
   - **Why it's necessary:** Configuring database webhooks allows your application to respond to changes in the database in real-time, such as sending notifications or updating records, ensuring your app stays in sync with the database.
   - **Action:** [Set up the database webhooks in Supabase](https://supabase.com/docs/guides/database/webhooks) to point to your React Router app at `/api/db/webhooks`.
9. **Emails Configuration**
   - **Why it's necessary:** Properly configuring your email service ensures that your application can send emails for account verification, password resets, and other notifications, which are crucial for user communication.
   - **Action:** Obtain SMTP details from an email service provider such as SendGrid, Mailgun, or Resend. Configure the emails in both the environment variables and the [Supabase project settings](https://supabase.com/docs/guides/auth/auth-smtp).
10. **Billing Setup**
    - **Why it's necessary:** Well - you want to get paid, right? Setting up billing ensures that you can charge your users for using your SaaS application, enabling you to monetize your service and cover operational costs. This can take a while.
    - **Action:** Create a Stripe or Lemon Squeezy account. Update the environment variables with the correct values for your billing service. Point webhooks from Stripe or Lemon Squeezy to `/api/billing/webhook`. Refer to the relevant documentation for more details on setting up billing.

**Note**: Please note that these steps are essential for setting up Makerkit and ensuring that your SaaS application functions correctly. Omitting any of these steps may result in errors or unexpected behavior in your application.

-----------------
FILE PATH: kits\react-router-supabase-turbo\going-to-production\configuring-supabase-database-webhooks.mdoc

---
status: "published"

title: "Setting the Database Webhooks required to run the React Router Supabase SaaS kit"
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
FILE PATH: kits\react-router-supabase-turbo\going-to-production\fly.mdoc

---
status: "published"
title: 'Deploy React Router Supabase Turbo to Fly.io'
label: 'Deploy to Fly.io'
order: 7
description: 'Guide to deploy the React Router  SaaS boilerplate to Fly.io'
---

This guide was shared by a community member to deploy the React Router Supabase Turbo Kit to Fly.io.

First, create the `fly.toml` in the root directory (configure fly server to your needs):

```toml
app = "react-router-turbo-kit"
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
FILE PATH: kits\react-router-supabase-turbo\going-to-production\production-environment-variables.mdoc

---
status: "published"
title: "Generating and validating the environment variables needed for deployment | React Router Supabase SaaS Kit"
label: "Environment Variables"
description: "Guide to generate and validate the environment variables needed for deployment in your React Router Supabase SaaS Kit project"
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
FILE PATH: kits\react-router-supabase-turbo\going-to-production\vercel.mdoc

---
status: "published"
title: "Deploy React Router Supabase to Vercel"
label: "Deploy to Vercel"
order: 6
description: "Guide to deploy the React Router SaaS boilerplate to Vercel"
---

Deploying to Vercel should be straightforward. You can deploy the React Router SaaS boilerplate to Vercel by following these steps:

1. Connect your Repository to Vercel
2. Configure Environment Variables: the first time this may fail if you don't yet have a custom domain connected since you cannot place it in the environment variables yet. It's fine. Make the first deployment fail, then pick the domain and add it. Redeploy.
3. Deploy the Project

## Connect your Repository to Vercel

To deploy the React Router SaaS boilerplate to Vercel, you need to connect your repository to Vercel.

## Configure Environment Variables

Add the environment variables required for the project to work correctly. You can build them from [our environment variables generation](production-environment-variables).

## Set the correct Preset and Root Directory

Please make sure to set the "React Router" preset in the Vercel settings and to set the root directory to `apps/web`.

{% alert type="warning" %}
Failure to set the Preset to "React Router" will result in the project not working correctly.
{% /alert %}

## Update the Preset in your React Router Config

Install the `vercel-preset` package:

```bash
pnpm add --filter web --save-dev "@vercel/react-router"
```

Then, update the preset in your `react-router.config.ts` file:

```ts {% title="apps/web/react-router.config.ts" %}
import type { Config } from '@react-router/dev/config';

import { vercelPreset } from '@vercel/react-router/vite';

export default {
  ssr: true,
  presets: [
    vercelPreset()
  ],
} satisfies Config;
```

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

The above should be all! By following the steps, you should be able to deploy the React Router SaaS boilerplate to Vercel's Edge Functions with zero cold starts, better speed and lower costs!

Please be mindful of the limitations of the Edge runtime:

1. potentially higher latency to your database
2. limited Node.js features
3. limited access to the Node.js ecosystem

If you are fine with the above, then Vercel is a great choice for deploying your React Router SaaS boilerplate.

## I have more apps - how do I deploy them?

Vercel should automatically take care of deploying the app named `web`.

If you have multiple apps, you may need to customize the build command to point to the app being deployed.

```
cd ../.. && turbo run build --filter=<app-name>
```

Please replace `<app-name>` with the name of the app you want to deploy.

For more info refer to the [Vercel documentation](https://vercel.com/docs/monorepos/turborepo).


