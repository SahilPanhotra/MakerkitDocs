-----------------
FILE PATH: courses/nextjs-turbo/deploy-production.mdoc

---
title: "Going to Production with Makerkit Next.js Supabase Turbo"
description: "Learn how to configure your Makerkit Next.js Supabase Turbo project to production"
label: "Deploy to Production"
order: 8
---

Hi there! 👋 You made it to the end of the course! 🎉

Welcome to this module to deploying your Makerkit Next.js Supabase Turbo project to production.

In this module, we'll cover the following:

1. **Setting up your environment variables**: We'll cover how to set up your environment variables for your production environment.
2. **Deploying your project**: We'll cover how to deploy your project to Vercel.
3. **Setting up your Supabase project**: We'll cover how to set up your Supabase project for production.
4. **Stripe integration**: We'll cover how to set up your Stripe account for production.
5. **Minor changes for production**: We'll cover some minor changes you might need to make for your project to work in production.

In addition to this module, you will find the [official documentation](/docs/next-supabase-turbo/checklist) extremely helpful. The documentation covers everything you need to know about deploying your project to production.

The estimated time to complete this module is around 1 hour, but it could take longer if you're setting up your accounts for the first time.

Without further ado, let's get started! 🚀

## Requirements

Before we get started, make sure you have the following:

1. A Vercel account - you can sign up for one [here](https://vercel.com/signup).
2. A Supabase account - you can sign up for one [here](https://supabase.io/).
3. A Stripe account - you can sign up for one [here](https://stripe.com/).
4. Optionally, an SMTP server for sending emails.

Without these, you won't be able to complete this module. If you don't have these accounts, please sign up for them before continuing.

## High-level overview of our production deployment

Our production deployment will look like this 👇

{% img src="/assets/courses/next-turbo/production-services-overview.webp" width="707" height="790" /%}

1. **User**: the user visits your website that sits on Vercel / or on any other hosting provider you choose.
2. **Vercel**: hosts your Next.js application and serves it to the user.
3. **Supabase**: the application will communicate with Supabase for the database, authentication and storage.

## Creating a Supabase project for production

Before we deploy our project to production, we need to set up a Supabase project for production. This involves setting up the database, authentication, storage, and SMTP service.

If you haven't yet, please create a new [Supabase](https://supabase.com) account and create a new project. **You can do this on the free tier**, no need to upgrade to a paid plan.

Now that you have created a new Project, Supabase will provide you with some environment variables that you need to set up in your project.

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

For more information, please visit the [official documentation to set up emails in Supabase](/docs/next-supabase-turbo/supabase).

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

Once succeeded, your Supabase Database is now set up for production.

### Adding Supabase Database Webhooks pointing to your Vercel project

The last step is to add the Database Webhooks to your Supabase project. These webhooks will notify our application when certain records get created, updated, or deleted in the database. Makerkit uses these notifications for various reasons, such as cleaning up subscriptions after an account is deleted, or to send emails when a new invitation gets created.

To add the Database Webhooks, please [follow the official documentation](/docs/next-supabase-turbo/supabase).

Once set up, you should see the Webhooks in your Supabase project.

{% img src="/assets/courses/next-turbo/supabase-webhooks.webp" width="2062" height="876" /%}

### Set up Google Auth [Optional]

If you want to use Google Auth in your project, [you need to set up Google Auth in your Supabase project](https://supabase.com/docs/guides/auth/social-login/auth-google).

This is a little bit more convoluted as you need a Google Cloud project, so I recommend to do this later on.

However, I am writing it here as a reminder to do it before you go live with your project.

## Setting up Stripe

Before we deploy our project to production, we need to set up our Stripe account. This involves setting up our Stripe keys and adding a webhook to our Stripe account.

If you don't yet have a Stripe account, please create one [here](https://stripe.com/).

### Getting up your Stripe keys

Navigate to the `Developers` -> `API keys` section in your Stripe account to retrieve your Stripe keys.

In this section - you will retrieve two keys:

1. **Publishable Key**: this is the public key that the client will use to communicate with Stripe. We use it to display the embedded Stripe Checkout on the client-side.
2. **Secret Key**: this is the secret key that the server will use to communicate with Stripe server-side. We use it to create subscriptions, update subscriptions, and more.

{% img src="/assets/courses/next-turbo/stripe-api-keys.webp" width="1742" height="864" /%}

### Adding a Webhook to your Stripe account

We now need to add a webhook to our Stripe account. This webhook will notify our application when certain events happen in Stripe, such as a subscription being created, updated, or deleted.

Additionally, we will need the signing secret that we will use to verify the webhook events - to make sure they are coming from Stripe.

To add a webhook to your Stripe account, follow these steps:

1. Navigate to the `Developers` -> `Webhooks` section in your Stripe account.
2. Click on the `Add endpoint` button.
3. Fill in the `Endpoint URL` with your application's URL. You can use the following format: `https://<your-url>/api/billing/webhook`. If you don't have one yet, you can fill it in later and get back to this step.
4. Select the following events: `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`. Select more if your app depends on more events.

Once created a webhook, you will see the webhook in your Stripe account. Click on the webhook to display the `Signing Secret`. This key is what we name `STRIPE_WEBHOOK_SECRET` in our environment variables. Hold on it until you set up your environment variables.

{% img src="/assets/courses/next-turbo/stripe-signing-secret.webp" width="1310" height="208" /%}

## Create Stripe Products

Before you go live with your project, you need to create the products in your Stripe account. These products are what your users will subscribe to.

I will show you how to create one of the products, and you can create the rest of the products in the same way.

### Flat-rate Subscription

If you remember, we created a flat-rate subscription in our project. This subscription is a fixed price subscription that your users can subscribe to.

Let's add the Starter Products to your Stripe account. To do this, follow these steps:

1. Navigate to the `Product Catalog` section in your Stripe account.
2. Click on the `Add product` button.
3. Fill in the `Name` with `Starter Plan`.
4. Set it as a `Recurring` product at a `Monthly` interval with a price of `49.00` USD. You can also add a Yearly interval if you want.

Once you have created your `Product` (Starter Plan) and your `Prices` (Monthly and Yearly), you can retrieve the Price IDs which we will define in our environment variables as we've seen in the previous section.

```bash {% title="apps/web/.env.production" %}
NEXT_PUBLIC_STARTER_PLAN_MONTHLY_VARIANT_ID=<your-monthly-price-id>
NEXT_PUBLIC_STARTER_PLAN_YEARLY_VARIANT_ID=<your-yearly-price-id>
```

### Per Seat Subscription

Our application charges $10 per user per month. To set this up, you need to create a `Product` in your Stripe account.

To create a Per Seat Subscription, we can create a `Product` in our Stripe account. To do this, follow these steps:

1. Navigate to the `Product Catalog` section in your Stripe account.
2. Click on the `Add product` button.
3. Fill in the `Name` with `Per Seat Plan`.
4. Set it as a `Recurring` product at a `Monthly` interval with a price of `10.00` USD.

When each user gets added to a Team Account, Makerkit will increase the quantity of the subscription by 1. This way, you can charge $10 per user per month. When a user gets removed from a plan, Makerkit will decrease the quantity of the subscription by 1. When a team subscribes to a Pro Plan (say with 5 users), the quantity will be set to 5 and they will be charged $50 per month.

Once you have created your `Product` (Per Seat Plan) and your `Prices` (Monthly and Yearly), you can retrieve the Price IDs which we will define in our environment variables as we've seen in the previous section.

```bash {% title="apps/web/.env.production" %}
NEXT_PUBLIC_PER_SEAT_PLAN_MONTHLY_VARIANT_ID=<your-monthly-price-id>
NEXT_PUBLIC_PER_SEAT_PLAN_YEARLY_VARIANT_ID=<your-yearly-price-id>
```

You can set these environment variables in your `.env` files depending on which environment you are deploying to.

## Setting up your environment variables

One of the most important tasks in deploying you project to production is setting up your environment variables. A wrong environment variable can lead to your project not working as expected. In this section, we'll cover how to set up your environment variables for your production environment.

### Exporting environment variables using the Turbo generator

Makerkit provides a generator that will prompt you for your environment variables and generate a `.env.local` file for you, which you can then use within your Vercel/Cloudflare/any other hosting provider.

To use the generator, run the following command:

```bash
turbo gen env
```

The generator will prompt you for your environment variables and generate a `.env.local` file for you.

Now, you can validate the `.env.local` file and make sure all the environment variables are correct.

```
turbo gen validate-env
```

If it passes, you can now copy the environment variables into your Vercel project. You can also feel free to remove the ones that are already set in your `.env` and `.env.production` files, since they will simply be overwritten.

## Pushing your Repository to Github

Before deploying your project to Vercel, you need to push your repository to Github. If you haven't done this yet, you can do so by running the following commands:

### Deleting the origin remote

Delete the origin remote:

```bash
git remote rm origin
```

### Adding the Github remote

Add the Github remote to your project:

```bash
git remote add origin <your-github-repo-url>
```

Now, commit your changes and push your repository to Github:

```bash
git add .
git commit -m "<your-commit-message>"
git push origin main
```

Now that your repository is on Github, you can deploy your project to Vercel.

## Deploying your project to Vercel

Once we have a Supabase project set up and our environment variables configured, we can deploy our project to [Vercel](https://vercel.com). Vercel is a cloud platform that allows you to deploy your projects with ease.

### Connecting the Repository to Vercel

To deploy your project to Vercel, you need to connect your Github repository to Vercel. To do this, follow these steps:
1. Go to [Vercel](https://vercel.com/).
2. Click on the `Import Project` button.
3. Select the repository you want to deploy. You need to tweak your settings to make sure the repository can be read by Vercel.
4. Click on the `Import` button.
5. Configure your project settings.

When configuring your project settings, make sure to set the following settings:
1. The preset to `Next.js`.
2. The root directory to `apps/web`.

Now, copy/paste your environment variables from the `.env.local` file into the Vercel project settings.

Now you can deploy your project to Vercel by clicking on the `Deploy` button. If the build fails, you can check the logs to see what went wrong.

Once deployed, you should finally see your project live on Vercel. Congratulations! 🎉

### Trial and Error while deploying

If you encounter any issues during the deployment process, don't worry.

Deployment can be tricky, and it's normal to encounter issues along the way. I myself have to redeploy a few times due to some misconfigured option. It's also totally possible I'm forgetting something here, so please let me know if you encounter any issues.

Take a deep breath, analyze the error message, and try to fix the issue. If you can't fix it, don't hesitate to ask for help. The Makerkit community is here to help you.

## Minor changes for production

Before you go live with your project, there are some minor changes you might need to make to your project to make it production-ready.

### Update your Legal Pages

Before you go live with your project, you need to update your legal pages. This includes your Privacy Policy, Terms of Service, and Cookie Policy.

You can find the legal pages in the `apps/web/(marketing)/(legal)` folder in your project. You can update these pages to reflect your company's policies.

### Update your FAQ

Before you go live with your project, you need to update your FAQ. This includes the questions and answers that your users might have about your product.

You can find the FAQ in the `apps/web/(marketing)/faq/page.tsx` file in your project. You can update these questions and answers to reflect your product.

### Update the static content (Blog, Docs)

Before you go live with your project, you need to update the static content in your project and replace the placeholders used by Makerkit. This includes the blog posts and documentation that your users might read.

### Make it truly YOURS

Add your own touch to Makerkit and make it truly yours. The great thing about Makerkit is its vanilla nature - you can customize it to your heart's content. You can change the colors, the fonts, the layout, and more to make it your own.

## Deployment Checklist

**Supabase Setup**

1. Create a new Supabase project
2. Save the Database Password securely
3. Retrieve Supabase URL, Anon Key, and Service Role Key
4. Configure authentication settings (Site URL and Redirect URL)
5. Set up SMTP service for emails
6. Update email templates in Supabase
7. Link local project to Supabase using CLI
8. Push database migrations to Supabase
9. Add Database Webhooks pointing to your application

**Stripe Setup**

1. Create a Stripe account (if not already done)
2. Retrieve Publishable Key and Secret Key
3. Add webhook endpoint in Stripe dashboard
4. Save Stripe Webhook Signing Secret
5. Create Stripe products and pricing plans
6. Note down Price IDs for each plan

**Environment Variables**

1. Run turbo gen env to generate .env.local
2. Validate environment variables with turbo gen validate-env
3. Ensure all required variables are set correctly

**Github**

1. Remove existing origin remote
2. Add new Github remote
3. Commit and push changes to Github

**Vercel Deployment**

1. Connect Github repository to Vercel
2. Set project configuration (Next.js preset, root directory)
3. Copy environment variables to Vercel project settings
4. Deploy project
5. Verify successful deployment

**Post-Deployment Tasks**

1. Update legal pages (Privacy Policy, Terms of Service, Cookie Policy)
2. Update FAQ page
3. Replace placeholder content in blog and docs
4. Customize design elements (colors, fonts, layout)
5. Test all features in the production environment
6. Set up Google Auth (if using)
7. Verify all webhooks are working correctly
8. Check email functionality

**Final Checks**

1. Ensure all environment variables are correctly set
2. Verify database connections and queries are working
3. Test authentication flow (sign up, sign in, password reset)
4. Confirm Stripe integration is functioning (test subscription)
5. Check that all pages are loading correctly
6. Verify mobile responsiveness
7. Run final checks on security settings

This checklist covers all the major steps discussed in the deployment lesson. It can serve as a quick reference guide to ensure nothing is overlooked during the deployment process.

## You made it to the end!

Congratulations! 🎉 You made it to the end of the course!

You have successfully deployed your Makerkit Next.js Supabase Turbo project to production. You have set up your Supabase project, Stripe account, and environment variables. You have also made some minor changes to your project to make it production-ready.

I hope you enjoyed this course and learned a lot about deploying your project to production. If you have any questions or feedback, please let me know. I would love to hear from you!

Thank you for taking this course, and I wish you all the best with your project! 🚀


-----------------
FILE PATH: courses/nextjs-turbo/introduction.mdoc

---
title: "Introduction to Makerkit Turbo - the leading Next.js Supabase SaaS Starter Kit"
description: "Learn how to build and launch your SaaS product quickly using Makerkit Turbo - the leading Next.js Supabase SaaS Starter Kit."
label: "Introduction to Makerkit Turbo"
order: 0
---

Hi there! 👋 Giancarlo here. I'm the creator of Makerkit.

Welcome to this course on building and launching your SaaS product quickly using Makerkit, the leading Next.js Supabase SaaS Starter Kit. This course will guide you through the process of setting up your SaaS product using Makerkit.

Here is a quick preview of what you will learn:

1. **Setting up your project**:  How to set up your project using Makerkit Turbo and Supabase
2. **Customization**: How to customize the design of your app and apply your branding to it
3.**Features**: How to add new pages and features to your app using Makerkit's architecture
4. **JS Widget**: Building an embeddable Javascript widget and communicate with your app using an external-facing API
5. **Billing**: How to add billing to your app using Stripe and setup your pricing plans and how to handle limits and quotas for your users
6. **Production**: How to deploy your app to production using Vercel

### What do we build?

We will be building together **a SaaS for Customer Support** - a simple helpdesk application that allows users to create tickets and chat with support agents.

This guide can be used as documentation, but it's a more practical approach. For the nitty-gritty details, you can always refer to the [official documentation](/docs/next-supabase-turbo/introduction) - which should cover everything you need to know.

**Makerkit (Turbo version) is built on top of Next.js App Router and Supabase**, two powerful tools that make it easy to build and deploy web applications. Next.js is a React framework that provides a great developer experience, while Supabase is an open-source Firebase alternative that provides a powerful backend for your app.

Together, **they make a killer tech stack for building ambitious SaaS products** - for both large teams and solo developers.

---

This course will take approximately 8 hours to complete. It's divided into multiple sections, each covering a different aspect of building a SaaS product using Makerkit.

It is highly recommended to follow along using the complete repository at `https://github.com/makerkit/next-supabase-saas-kit-turbo-course`. You need a valid license to use the course, you will see a 404 page if you try to access the repository without a valid license.

Let's get started! 🚀

## Issues, bugs and feedback

Please use our [Discord channel](https://discord.gg/3hBn3nhbgD) to report any of these. I am striving to make this course as good as possible, and your feedback is invaluable.

## What is Makerkit?

Makerkit is **a Next.js SaaS Starter Kit that helps you build and launch a SaaS product**. Unlike other SaaS kits, Makerkit is opinionated, offers a scalable architecture, and is easily extendable.

The Turbo version is Makerkit's v2, built on top of the learnings and experience gained from the v1, used by thousands of customers every day.

Makerkit Turbo uses Turborepo, a tool that helps you manage multiple packages in a single repository. Turborepo makes it easy to manage the project and add new features - and keeping the codebase clean and organized.

Makerkit resembles a lot a framework that you can use to build your app.

## What you'll learn in this course

In this course, you will learn how to build and launch your SaaS product using Makerkit Turbo. Here's what you'll learn:

- **Set Up**: How to set up your project using Makerkit Turbo
- **Customization**: How to customize the design of your app
- **Supabase**: How to use Supabase: migrations, queries, mutations and deploying to production
- **Adding Features**: How to add new pages and features to your app
- **React Query**: How to use React Query for efficiently fetching and caching data
- **Realtime**: How to stream updates from Supabase Realtime to your app
- **JS Widget**: Building an embeddable Javascript widget and communicate with your app using an external-facing API
- **Billing**: How to add billing to your app using Stripe and setup your pricing plans
- **Deployment**: How to deploy your app to production using Vercel
- **Troubleshooting**: How to troubleshoot common issues and errors, and how to make sure your code is debuggable
- **A bit of the basics**: Of course, we will also be explaining some of the basics needed to understand the codebase

By the end of this course, **you should be able to build most of the features** you need for your SaaS product and launch it to production. You'll master Makerkit in no time!

## Why Makerkit?

There are countless SaaS starter kits out there, so why should you choose Makerkit? How is it different from the others?

Here are a few reasons:

1. **Opinionated**: Makerkit is opinionated, which means it makes decisions for you. This can be a good thing because it saves you time and helps you avoid common pitfalls.
2. **Educational**: Makerkit is designed to be educational. It's not just a starter kit - it's a learning tool that helps you understand how to build a SaaS product, and understand better Next.js/Remix and Supabase.
3. **Scalable**: Makerkit is built to scale. It's designed to handle large teams and large codebases. It uses Turborepo, a tool that helps you manage multiple packages in a single repository.
4. **Extendable**: Makerkit is easily extendable. You can add new features to your app by creating new packages or apps. You can also customize the design of your app by creating new components or styles.
5. **Community**: Makerkit has a great community. You can ask questions, share your work, and get feedback from other developers using Makerkit.
6. **Support**: I am here to help you get the most from Makerkit. If you have any questions or need help, you can reach out to me on Discord.

## Makerkit's Philosophy

Makerkit is built on a few core principles:

**Extensibility**: You should be able to edit the codebase without fear of breaking things. This means:
 - The codebase should be **easy to understand** - allowing anyone to extend it
 - The codebase should be **strictly typed** - e.g. fail in case of errors, so you can catch them early
 - The codebase should be **easy to extend** - as your app will inevitably diverge from the starter kit

**Education**: You should be able to learn from the codebase and understand how it works. This means:
 - The codebase should be **well-documented** - so you can understand how it works
 - The documentation should **inform AND educate** - so you can learn how to build your app. This course is a key part of that.
 - The documentation should be **complete** - so you can understand how to use the kit without needing to ask questions

**Support**: You should be able to get help when you need it. This means:
 - The community should be **welcoming** - so you can ask questions and get help
 - The creator should be **available** - so you can get help when you need it
 - The documentation should be **up-to-date** - so you can get the help you need


## Who is this course for?

This course is for **developers who want to build and launch a SaaS product quickly and have purchased a license**.

This course does not replace the (very extensive) documentation, but it's a more practical and streamlined approach to building a SaaS product.

You should use both in tandem - the official documentation for the details and this course for a more practical, step-by-step guide on building a SaaS product.

## What we will build together

In this course, **we will be building a SaaS for Customer Support** - a simple help-desk application that allows users to create tickets and chat with support agents.

It will have the following features:

1. **Support Tickets**: Users can create support tickets and see updates in real-time.
2. **Widgets**: Users can embed a widget on their website to allow their customers to create tickets.
3. **Support Agents Chat**: Support agents can chat with users in real-time
4. **Billing**: Users can subscribe to a plan and pay using Stripe
5. **Features Limitations**: Users can be limited by the features they can use based on their plan
6. **A Sprinkle of AI**: We will add a sprinkle of AI to the app to make it more interesting - when customers create a ticket, we will use AI to create the name of the ticket based on the content. We can add more AI features as we go.

## Prerequisites

Before you start this course, you should have the following:

- Basic knowledge of React, TypeScript, and Tailwind CSS
- Basic knowledge of Next.js App Router
- Basic knowledge of Supabase
- A Makerkit license to access the Makerkit Turbo repository

If you're new to Next.js, Supabase, or Makerkit, I suggest you take a look at the official documentation to get familiar with the tools. You can also check out the [Next.js](https://nextjs.org/docs/getting-started) and [Supabase](https://supabase.io/docs) documentation to get started. Once you have a good understanding of the tools, you can move on to the next section.

## Getting Started

To get started, you will need some software installed on your computer:

1. Git
2. Node.js (and Pnpm)
3. Docker

### Why Docker?

Docker is required for running Supabase locally. While this is not a strict requirement, it is recommended to use Docker for local development. You don't need to know Docker to use Supabase, you simply need to have it running so that you can spin up the services locally.

I suggest to install Docker Desktop or [Orbstack](https://orbstack.dev/) (a lighter alternative). Please make sure you have Docker installed and running before proceeding, otherwise you will not be able to run the course.

NB: you don't need to know Docker to use Supabase, you simply need to have it running so that you can spin up the services locally.

### Step 1: Clone the Makerkit Turbo repository

{% alert type="warning" title="Have Github configured with SSH" %}
The following commands assume that you have your Github account configured with SSH. If you don't have it configured, please refer to the [official documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) to set it up.
{% /alert %}

With the assumption that you have your Github account configured with SSH, you can clone the Makerkit Turbo repository from GitHub:

```bash
git clone git@github.com:makerkit/next-supabase-saas-kit-turbo.git
```

This will create a new directory called `next-supabase-saas-kit-turbo` with the project files. Please navigate into the directory:

```bash
cd next-supabase-saas-kit-turbo
```

And now you can install the dependencies. Please install `pnpm` if you haven't already:

```bash
npm install -g pnpm
```

And then install the dependencies:

```bash
pnpm i
```

Fantastic! You're now ready to start the project. Please proceed to the next step.

### Step 2: Start the project

{% alert type="warning" title="Have Docker running" %}
We need to have Docker running so that we can start the Supabase Docker container. If you don't have Docker running, please start it now.
{% /alert %}

We now need to start the project. To do this, we need to start the Supabase Docker container and the Next.js development server.

To start the Supabase Docker container, run the following command:

```bash
pnpm run supabase:web:start
```

This will start the Supabase Docker container - and will load the Database migrations and seed data.

Next, we need to start the Next.js development server. To do this, run the following command:

```bash
pnpm run dev
```

This will start the Next.js development server at `http://localhost:3000`. You can now open your browser and navigate to this URL to see the project running.

**Congratulations! You have successfully started the project**. You can now explore the project and start customizing it to your needs. Yay! 🎉

## Step 3. InBucket Email Server

To receive emails during local development, Supabase uses [InBucket](http://localhost:54324/status). It's from here that you will be receving emails from both Supabase Auth and transactional emails during local development.

You can bookmark the InBucket URL, as you will need it quite often.

#### Quick Introduction to Turborepo and PNPM

Turborepo is a monorepo tool that allows you to manage multiple packages in a single repository. 

It provides a set of commands to manage the packages and dependencies in your project. PNPM is the package manager we use in combination with Turborepo to manage the dependencies and packages in your project.

There's not too much you need to know to use Makerkit effectively, however it's worth taking a moment to understand the basics of Turborepo and PNPM.

Thanks to Turborepo, we can split the codebase into multiple packages (and apps) which allows us to manage the dependencies and packages in a more efficient way, and use incremental builds to speed up the development process (such as, linting, formatting, and type checking).

In fact, Turborepo is able to detect changes that affect the dependencies and packages in your project and cache. For example, try running `pnpm run typecheck`: the first time, it will check all the packages. Now, run it again: it will feel instantaneous, as all the packages were cached.

 However, let's make a change to any package: now, the package and its dependents will be computed again.

##### PNPM - A faster alternative to NPM

PNPM is a faster alternative to NPM, capable of using less disk space and being faster. It also provides great tooling for working with monorepos.

With PNPM, we can run commands inside apps or packages by using the `--filter` argument. This is something you'll be encountering often.

For example, the package `@kit/stripe` contains a script called `start`, which allows us to start the Stripe CLI.

You can call this command anywhere by specifying the package:

```bash
pnpm --filter '@kit/stripe' start
```

Similarly, you can run commands inside an app by specifying the app:

```bash
pnpm --filter web dev
```

### Step 3: Explore the project

Before we start building the project, let's take a moment to explore the project structure - so you can have a high-level understanding of how the project is organized.

**Makerkit is organized using Turborepo**, a monorepo tool that allows you to manage multiple packages in a single repository.

You can now explore the project. The project is divided into two main parts:
- **apps** - i.e. the Next.js apps
- **packages** - shared packages that can be used across multiple apps

{% img src="/assets/courses/next-turbo/turbo-architecture.webp" width="618" height="377" /%}

The `apps` directory contains the Next.js app, while the `packages` directory contains the shared packages used by the app.

You can think of the `packages` directory as a library of components, hooks, and utilities that can be shared across multiple projects - and it's basically Makerkit-core.

The `apps` directory contains the following apps:

1. `web` - the main Next.js app
2. `e2e` - the end-to-end tests using Playwright

{% img src="/assets/courses/next-turbo/turbo-architecture-apps.webp" width="618" height="377" /%}

The `packages` directory contains the following packages.

{% img src="/assets/courses/next-turbo/turbo-architecture-packages.webp" width="618" height="377" /%}

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

It's a lot to take in - but don't worry!

We will be exploring these packages in more detail as we build out the project - and generally speaking, you don't need to worry about the packages too much, think of it as a framework that you can use to build your app.

### How do apps and packages work together?

Generally speaking, the `packages` are the building blocks of the project. They contain the shared logic and utilities that can be used across multiple apps.

Instead, the apps within `apps` are the actual applications that use these packages to build the project.

The `apps` have two main responsibilities:

1. **Routing**: Define the routes and pages of the application
2. **Configuration**: Define the configuration that is specific to the application (e.g., environment variables, Supabase URL, etc.)

The `packages` will receive the configuration from the apps and use it to build the project. This allows you to reuse the packages across multiple apps without having to duplicate the code.

{% img src="/assets/courses/next-turbo/turbo-architecture-apps-packages.webp" width="1080" height="377" /%}

### Do I need to worry about the packages?

No, you don't need to worry about the packages too much - and generally speaking you **don't need any Turborepo knowledge** to build your app with Makerkit. You should think of the packages as a framework that you can use to build your app - just as you would by using any other library or framework.

The packages system is a technical implementation that helps write code decoupled from yours. Think of them as a framework that you can use to build your app. You can use the packages as they are, or you can customize them to fit your needs.

You can even fully ignore `packages` and just focus on the `apps` - and you will still be able to build your app. Actually - I'd say that's the best way to start.

Just focus on the `apps` and build your app - and as you get more comfortable with the project, you can start exploring the `packages` and see how they can help you build your app.

## Routing Overview

The codebase is structured in a way that **makes it easy to navigate and update**.

Before we get into the details, I want to refresh your memory on how Next.js App Router's routing system works.

### A primer on Next.js routing

Next.js uses a file-system-based routing system - which means we define the route of the application based on the file structure.

Here's how it works:

1. **Pages**: the pages of the applications are defined using the special file convention `page.tsx`. Every file named `page.tsx` in the `app` directory will be a route in the application. For example, `app/home/page.tsx` will be the route `/home`.
2. **Layouts**: the layouts of the application are defined using the special file convention `_layout.tsx`. Every file named `layout.tsx` in the `app` directory will be a layout in the application. For example, `app/layout.tsx` will be the layout for the entire application.
3. **Pathless routes**: whenever a folder is surrounded by parenthesis, it means it's a pathless route. For example, `(marketing)` will be a pathless route in the application. Pathless means that `marketing` will not be in the URL - it's a route that is not visible in the URL.
4. **Params routes**: whenever a folder is surrounded by brackets, it means it's a param route. For example, `[account]` will be a param route in the application. Params routes are dynamic routes that can be used to pass data to the route.
5. **Nested routes**: you can nest routes by creating a folder with the same name as the route. For example, `app/home/user/page.tsx` will be the route `/home/user`.
6. **Special Conventions**: other conventions include **loading.tsx** for displaying a loading screen and **error.tsx** for displaying an error screen.

The above is enough for understanding the routing in Makerkit, but for a more complete guide, you can refer to the [Next.js documentation](https://nextjs.org/docs/app/building-your-application/routing).

### Routing in Next.js Makerkit Turbo

Here are some of the key files and directories you should be aware of:

```
- apps
-- web
--- app
--- components
--- config
--- lib
--- content
--- styles
--- supabase
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
-- home
-- (marketing)
-- auth
-- join
-- admin
-- update-password
-- server-sitemap.xml
```

The image below shows the routing overview of the application for the directories that will matter the most.

1. `home` - this is where the internal home of the application lives. This is where you define the routes when the user is logged in.
2. `(marketing)` - this is where the marketing pages of the application live. This is where you define the routes when the user is not logged in. It's a pathless route, so you don't see it in the URL.
3. `auth` - this is where the authentication pages of the application live. This is where you define the routes for the login, signup, and forgot password pages.

{% img src="/assets/courses/next-turbo/routing-overview.webp" width="760" height="255" /%}

Let's break down the directories:

1. `home` - this is where the internal home of the application lives. This is where you define the routes when the user is logged in.
2. `(marketing)` - this is where the marketing pages of the application live. This is where you define the routes when the user is not logged in. It's a pathless route, so you don't see it in the URL.
3. `auth` - this is where the authentication pages of the application live. This is where you define the routes for the login, signup, and forgot password pages.
4. `join` - this is where the join pages of the application live. This route is the route where the user can join a team account following an invitation.
5. `admin` - this is where the admin pages of the application live. This is where you define the routes for the super admin pages.
6. `update-password` - this is where the update password pages of the application live. This is the route the user is redirected to after resetting their password.
7. `server-sitemap.xml` - this is where the sitemap of the application is defined.

### Diving into `marketing`

The `marketing` directory is where the marketing pages of the application live. Here are some of the key files you should be aware of:

```
- (marketing)
-- page.tsx <--- home page
-- pricing/page.tsx
-- faq/page.tsx
-- contact/page.tsx
-- blog/...
-- docs/...
-- (legal)/...
```

{% img src="/assets/courses/next-turbo/marketing-routing-overview.webp" width="703" height="260" /%}

### Diving into `home`

The `home` directory is where the internal dashboard of the application lives. Here are some of the key files you should be aware of:

```
- home
-- (user)
-- [account]
```

Let's break down the directories:

1. `(user)` - this is where the user pages of the application live. This is where you define the routes for the personal account pages.
2. `[account]` - this is where the account pages of the application live. This is where you define the routes for the team account pages.

The `home` path allows us to separate the marketing pages from the internal dashboard pages.

So the user home page would be `/home/user` and the account home page would be `/home/[account]`.

{% img src="/assets/courses/next-turbo/home-routing-overview.webp" width="548" height="399" /%}

#### When to use `(user)` and `[account]`

The `(user)` and `[account]` directories are used to separate the user pages from the account pages. The `(user)` directory is used for the personal account pages, while the `[account]` directory is used for the team account pages.

Imagine you want to add some settings that are specific to the user - you would add them to the `(user)` directory. If you want to add some settings that are specific to the account - you would add them to the `[account]` directory.

#### My SaaS is exclusively B2C, what should I do?

If your app is exclusively B2C, you have the freedom to ignore the `[account]` directory and just use the `(user)` directory.

### What's next?

In this course, we will be building a SaaS product using Makerkit Turbo. We will start by setting up the project, customizing the design, and deploying the app to production.

We will also add new features to the app and explore the different packages that Makerkit Turbo provides.

I hope you're excited to get started! Let's dive in and see you in the next section! 🚀


