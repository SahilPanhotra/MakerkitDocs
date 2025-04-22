-----------------
FILE PATH: kits\next-supabase-lite\how-to\site\updating-navigation-menu.mdoc

---
status: "published"

title: "Updating the navigation menu of your Makerkit SaaS application"
label: Updating the Navigation menu
order: 0
description: "Learn how to update the navigation menu of your Makerkit landing pages"
---


The navigation menu of your Makerkit SaaS application's landing pages (or marketing side) is defined in the `SiteNavigation` component defined at `src/components/SiteNavigation.tsx`.

### Adding a new menu entry to the navigation menu

To add a new menu entry, simply add a new entry to the `links` object.

The key of the object is the name of the menu entry, and the value is an object with the `label` and `path` properties.

 ```tsx {% title="src/components/SiteNavigation.tsx" %} {22-25}
const links = {
  SignIn: {
    label: 'Sign In',
    path: '/auth/sign-in',
  },
  Blog: {
    label: 'Blog',
    path: '/blog',
  },
  Docs: {
    label: 'Docs',
    path: '/docs',
  },
  Pricing: {
    label: 'Pricing',
    path: '/pricing',
  },
  FAQ: {
    label: 'FAQ',
    path: '/faq',
  },
  NewPage: {
    label: 'New Page',
    path: '/new-page',
  },
};
```

Then, add a new `NavigationMenuItem` component to the `NavigationMenu` component.

 ```tsx {% title="src/components/SiteNavigation.tsx" %} {12}
<NavigationMenu>
  <NavigationMenuItem
    className={'flex lg:hidden'}
    link={links.SignIn}
  />

  <NavigationMenuItem link={links.Blog} />
  <NavigationMenuItem link={links.Docs} />
  <NavigationMenuItem link={links.Pricing} />
  <NavigationMenuItem link={links.FAQ} />

  <NavigationMenuItem link={links.NewPage} />
</NavigationMenu>
```

Et voilà! Your new menu entry is now available in the navigation menu.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\troubleshooting\403-error-actions.mdoc

---
status: "published"
title: "How to fix a 403 Error on Actions and API Routes | Next.js Supabase Lite Kit"
label: "403 error with API/Actions"
order: 5
description: "One of the reasons why you're hitting a 403 error in your Server Actions is omitting a CSRF token. Let's see how to fix it"
---

If you're encountering a 403 error on Server Actions and API Routes, it is possible that your requests lack a CSRF Token.

By default, the Makerkit middleware at `src/middleware.ts` will automatically try to validate a CSRF token for HTTP methods such as `POST`, `PUT` and `DELETE`: failing to send a valid CSRF token will result in these requests being forbidden.

To fix this error, check out these pages:

1. **Server Actions**: [Add a CSRF token to your Server Actions](server-actions-csrf)
2. **API Routes**: [Add a CSRF token to your API Routes](api-routes)

### The CSRF Token is sent, but it's not defined

If the CSRF Token is being sent but it's null/undefined, it's likely a bug. In that case, please open a support ticket in our Discord channel.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\troubleshooting\contentlayer-stuck.mdoc

---
status: "published"
title: "Contentlayer gets stuck when starting the Makerkit application | Next.js Supabase Lite SaaS Kit"
label: "Contentlayer gets stuck"
order: 3
description: "Contentlayer gets stuck when starting the Makerkit application (Next.js Supabase Lite)"
---

A small percentage of customers have reported issues when starting the kit because [Contentlayer](https://contentlayer.dev) gets stuck.

The only known possible fix is to downgrade both the packages `contentlayer` and `next-contentlayer` to version `0.3.1`:

```bash
npm i contentlayer@0.3.1 next-contentlayer@0.3.1
```

If it keeps happening, please contact me and I'll help you out.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\troubleshooting\dynamic-server-usage-error.mdoc

---
status: "published"
title: "How to fix the Next.js error Dynamic Server Usage in Next.js Supabase Lite"
label: "Dynamic server usage error"
order: 4
description: "The error Dynamic server usage Page couldn't be rendered statically because it used `cookies` is a common issue in Next.js. Here is the fix for Next.js Supabase Lite."
---

If you encounter the following error when starting the Makerkit application:

```
[ERROR] Dynamic server usage Page couldn't be rendered statically because it used `cookies`.
```

You can fix it by adding the following line to the nearest layout of the page that is causing the error:

```ts
export const dynamic = 'force-dynamic';
```

If it keeps happening, try to see if it is happening in yet another layout and add the line there as well. Do it until the error goes away.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\troubleshooting\supabase-not-starting.mdoc

---
status: "published"
title: "How to fix: Supabase is not starting | Next.js Supabase Lite"
label: "Supabase is not starting"
order: 0
description: "Learn why Supabase may not be able to start in certain cases and how to fix it."
---

Sometimes, Supabase may not be able to start. This can happen for a number of reasons, but the most common is that the port you are trying to use is already in use by another application.

## Is Docker running?

To start Supabase in development mode, you need to have Docker running. If you don't have Docker running, Supabase will not be able to start.

Please download any Docker-compatible application (Docker Desktop, Colima, Orbstack) and make sure it is running before starting Supabase.

## Is Supabase already running?

If Supabase is already running, then it will not be able to start again. You can check if Supabase is already running by running the following command in your terminal:

```bash
docker ps
```

If yes, you can kill them all using the following command:

```bash
docker kill $(docker ps -q)
```

NB: this command will shut down **all running Docker containers**.

To shut down Supabase, you can run the following command from your application's root directory:

```bash
npm run supabase:stop
```

If you have 2 different Supabase projects running, you can stop them individually by running the following command:

```bash
npm run supabase:stop
```

If you run this command from a different project, it will not stop the Supabase instance running in your current project. So make sure you are in the right directory before running this command.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\troubleshooting\supabase-not-stopping.mdoc

---
status: "published"

title: "How to fix: Supabase is not stopping"
label: "Supabase is not stopping"
order: 1
description: "Learn why Supabase may not be able to stop in certain cases and how to fix it."
---


Sometimes, Supabase may not be able to stop. In most cases, this happens because you have multiple projects running, and are trying to stop it from the wrong project.

Try to run the following command to discover the projects that are running in Docker:

```bash
docker ps
```

If you want to kill them all at once, also making Supabase stop, you can run the following command:

```bash
docker kill $(docker ps -q)
```

NB: this command will shut down **all running Docker containers**.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\troubleshooting\tables-not-found.mdoc

---
status: "published"
title: "How to fix: Supabase tables not found | Next.js Supabase SaaS Kit (Lite)"
label: "Tables/Functions not found"
order: 2
description: "How to fix the error: Tables/Functions not found when running Supabase locally or remotely."
---

When you try to start your Makerkit SaaS and try to navigate to your application, you may hit an error that says that the SQL tables or functions were not found.

This can happen for two reasons:

1. You're running Supabase locally and the migrations were not applied correctly.
2. You're running Supabase remotely and the migrations were not pushed to Supabase.
3. Something else is wrong with your Supabase instance. In case contact me or the Supabase team.

## This is happening when running Supabase locally

If this is happening when running Supabase locally, it means that the migrations weren't applied correctly. In this case, I recommend restarting Supabase and running the migrations again.

```
npm run supabase:stop
npm run supabase:start
```

### This is happening in the Supabase Cloud Instance

Instead - if this is happening in your remote instance, it means you did not push your migrations to the remote instance. In this case, I recommend pushing your migrations to the remote instance.

To push our database to Supabase, we need to link the Supabase CLI to our project.

You'll need to grab your project's reference ID from the URL: it's the part that comes after `app.supabase.io/` in the URL or the segment that comes before `supabase.co/` in the URL, such as `*******.supabase.co`.

We can do this by running the following command:

```bash
./node_modules/.bin/supabase link --project-ref **************
```

When prompted for the database password, enter the password you created when creating your project.

At this point, we can proceed to push our database to Supabase. Run the following command to push our database to Supabase:

```bash
./node_modules/.bin/supabase db push
```

The output should look like this:

```bash
Applying migration 20230705082911_schema.sql...
Finished supabase db push.
```

To verify that our database was pushed successfully, navigate to your project's dashboard and verify the tables were created.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\ui\dark-theme.mdoc

---
status: "published"
title: "Setting up the dark theme in Next.js Supabase SaaS Kit (Lite)"
label: Dark Theme
order: 1
description: "How to set up the dark theme in Next.js Supabase SaaS Kit (Lite)"
---

Makerkit supports by default a dark theme. With that said, you can decide to opt-out of the dark theme and use the light theme only instead using the configuration below.

## Using the dark theme by default

To use the dark theme by default, you need to edit the following configuration to your `src/configuration.ts` file:

 ```tsx {% title="src/configuration.ts" %}
{
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
}
```

Users can still opt-out of the dark theme by using the theme switcher in the top right corner of the page.

## Using only the dark theme

To switch to the dark theme, you need to edit the following configuration to your `src/configuration.ts` file:

 ```tsx {% title="src/configuration.ts" %}
{
  features: {
    enableThemeSwitcher: false,
  },
  theme: Themes.Dark,
}
```

## Disabling the dark theme

To disable the dark theme and only use the light theme - you need to edit the following configuration to your `src/configuration.ts` file:

 ```tsx {% title="src/configuration.ts" %}
{
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Light,
}
```

In the above, we flipped the `features.enableThemeSwitcher` flag to `false` and
set the `theme` to `Themes.Light`.

-----------------
FILE PATH: kits\next-supabase-lite\how-to\ui\theming.mdoc

---
status: "published"
title: Theming | Next.js Supabase SaaS Kit Lite
label: Theming
order: 1
description: "Discover how to apply the ShadCN UI theme to your Next.js Supabase SaaS Kit Lite project."
---

Makerkit's theming is defined in two places:
1. the `global.css` file
2. the `tailwind.config.js` file.

## Colors

To update the theme's colors, please update the `global.css` file in your project and update the CSS variables.

### Using the ShadCN themes

The ShadCN UI themes are predefined themes that you can use in your Makerkit project too.

You can [copy/paste the ShadCN themes](https://ui.shadcn.com/themes) into your `global.css` file and update the CSS variables to use the Makerkit color palette.

Next - you need to update the `tailwind.config.js` file in your project and update the `colors` object. These should match the CSS variables you have defined in the `global.css` file.

### Updating the Tailwind configuration

Additionally, we need to update the Tailwind configuration: since Makerkit uses a color palette based on Tailwind's palette system, we need to update the `tailwind.config.js` file in your project and update the `colors` object.

For example, since the default color for the `primary` color is `violet.500`, we need to update the `colors` object to:

```js
primary: {
  DEFAULT: 'hsl(var(--primary))',
  foreground: 'hsl(var(--primary-foreground))',
  ...colors.violet,
}
```

The code above spreads the `violet` color palette into the `primary` object, which means that the `primary` color will be the `violet.500` color.

The variable `--primary` is the hsl value of the `violet.500` color.

Additionally, we can access other colors of the `primary` palette using `primary-500`, `primary-600`, etc. This helps us to access the color palette in a more semantic way.

### Dark Mode

We need to do the same for the dark mode. Makerkit has historically used the `dark` color palette convention to define the dark mode colors.

By default, Makerkit uses `slate` as the dark mode color palette, so we need to update the `colors` object to:

```js
dark: {
  ...colors.slate,
  DEFAULT: colors.slate[950],
  foreground: colors.slate[100],
}
```

As you can see, we spread the `slate` color palette into the `dark` object, which means that the `dark` color will be the `slate.950` color.

If you were to use another color such as `zinc`, you would need to update the `colors` object to:

```js
dark: {
  ...colors.zinc,
  DEFAULT: colors.zinc[950],
  foreground: colors.zinc[100],
}
```

## Icons

While ShadCN UI uses `lucide` as their icons library, Makerkit has historically used `heroicons`. We have configured ShadCN UI to use `heroicons` instead of `lucide` in all Makerkit projects, but for new components, you will need to use `lucide` instead if you don't want to make changes. As such, please install `lucide` as a dependency in your project.

-----------------
FILE PATH: kits\next-supabase-lite\how-to.mdoc

---
status: "published"
title: "How to in Next.js Lite Supabase"
label: "How to"
order: 16
description: "How to in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\next-supabase-lite\migrations\version-0.1.0.mdoc

---
status: "published"

title: "Migrating to 0.1.0"
label: "Migrating to 0.1.0"
order: 0
---


The 0.1.0 release of the kit introduces a number of breaking changes. This guide will help you migrate your code to the new version.

NB: If you have cloned the repository after the 0.1.0 release, you can skip this guide.

---


The changes impact the following areas:

1. Tailwind CSS classes
2. Installed Packages

## Tailwind CSS classes

We have updates the Tailwind CSS classes using a new convention named `dark`, which is used for dark mode. This is a breaking change, as it will impact the way you use the Tailwind CSS classes.

Before, we used to have a `black` color - but it is no longer available. Instead, we have a `dark` color, which is used for dark mode. This means that you will need to update your code to use the new `dark` color.

Additionally, the new color uses a different level of opacity, which is more suitable to the standard Tailwind CSS convention, and is more consistent with the other colors.

By default, the `dark` color is an alias for `colors.gray` from the Tailwind CSS palette. You can change it by updating the `colors.dark` property in the `tailwind.config.js` file.

For example, you can provide your own palette, or switch to another dark color such as `colors.slate`.

 ```js {% title="tailwind.config.js" %}
const colors = require('tailwindcss/colors');

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./app/**/*.{ts,tsx,jsx,js}'],
  darkMode: 'class',
  theme: {
    extend: {
      fontFamily: {
        serif: ['serif'],
        heading: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
        sans: [
          'system-ui',
          'BlinkMacSystemFont',
          'Inter',
          'Segoe UI',
          'Roboto',
          'Ubuntu',
        ],
        monospace: [`SF Mono`, `ui-monospace`, `Monaco`, 'Monospace'],
      },
      colors: {
        primary: {
          ...colors.indigo,
          contrast: '#fff',
        },
        dark: colors.slate,
      },
    },
  },
};
```

### Automating the migration

To automate the migration, you can use the following script. First, install the package (either globally or locally):

```bash
npm i replace-in-file
```

Then, copy this script into your project named "migrate-0.1.js":

```js title="migrate-0.1.js
const replace = require('replace-in-file');
const files = ['src/**/*.tsx', 'src/**/*.css'];

const promise = () =>
  new Promise((resolve, reject) => {
    replace({
      ...getConfigs(),
      files,
    })
      .then((results) => {
        results.forEach((result) => {
          if (result.hasChanged) {
            console.log('File changed:', result.file);
          }
        });

        resolve();
      })
      .catch((error) => {
        console.error('Error occurred:', error);

        reject(error);
      });
  });

(async () => {
  console.log(`Replacing classes in "${files}" ...`);
  await promise();
  console.log('Done!');
})();

function getConfigs() {
  return {
    from: [
      /black-600/g,
      /black-500/g,
      /black-400/g,
      /black-300/g,
      /black-200/g,
      /black-100/g,
      /black-50/g,
      "'classnames'",
    ],
    to: [
      'dark-900',
      'dark-900',
      'dark-800',
      'dark-700',
      'dark-600',
      'dark-500',
      'dark-400',
      "'clsx'",
    ],
  };
}
```

Then, run the script:

```bash
node migrate-0.1.js
```

This will replace all the classes in your project.

## Packages

### Replaced "classnames" with "clsx"

Additionally, we replaced the package `classNames` with a newer, smaller, faster and maintained package named `clsx`. This is a breaking change, as it will impact the way you import the package.

The command above will also replace the import statement for you. However, if you want to do it manually, rename every instance of `'classnames'` with `'clsx'` in your project.

### Removed "Headless UI"

We have removed the package `headlessui` from the kit. It's slowly maintained and we don't use it anymore.

This is a breaking change, as it will impact the way you import the package.

If you have components using `headlessui`, please keep it in your project. Otherwise, you can remove it.

## Conclusion

This guide should help you migrate your project to the new version of the kit. If you have any questions, please get in touch with me.

-----------------
FILE PATH: kits\next-supabase-lite\migrations.mdoc

---
status: "published"
title: "Migrations in Next.js Lite Supabase"
label: "Migrations"
order: 11
description: "Migrations in Next.js Lite Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\next-supabase-lite\organizations.mdoc

---
status: "published"
title: "Organizations in Next.js Lite Supabase (legacy)"
label: "Organizations"
order: 11
description: "Learn how to add organizations to your Next.js Lite Supabase application"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\next-supabase-lite\payments\stripe-configuration.mdoc

---
status: "published"
title: Set up Stripe Payments for Next.js and Supabase Lite SaaS applications
label: Stripe Configuration
order: 0
description: 'Learn how to configure Stripe with your Next.js and Supabase Lite SaaS application'
canonical: '/docs/next-supabase/stripe-configuration'
---

MakerKit offers support for subscriptions and payments using [Stripe](https://stripe.com).

To get started with [Stripe Checkout](https://stripe.com/docs/payments/checkout), you will need to create an account using the [Stripe website](https://stripe.com).

If you want, you can get your business details sorted right away. If you
want to continue with the set-up, you can skip it for now and get back to it later.

## Configuration

Visit the [Stripe Checkout Tutorial](https://stripe.com/docs/checkout/quickstart?client=next) where we can copy the
configuration for our development environment.

If you click on the file `.env` of the code editor on the right-hand side,
you will be able to see the variables needed:

- `STRIPE_SECRET_KEY`
- `STRIPE_WEBHOOK_SECRET`

Copy these values to your local `.env` file.

{% alert type="warning" %}
Make sure to keep your <code>.env</code> file private. It should not be committed to your repository. These environment variables should be added using your hosting provider's dashboard.
{% /alert %}

## Creating your SaaS plans in Stripe

Stripe allows us to create our plans using the UI in the Stripe Dashboard.

To get started, let's add a couple of testing plans that you can use while
developing your application.

Visit [the Stripe Dashboard](https://dashboard.stripe.com/test/products/prod_KpXDAnvbPkXkgp) and add a new product (for example, a one-time purchase or a SaaS subscription).

You can add multiple "prices": for example, if you're defining a
subscription, you could use two plans, "Basic" and "Pro".

## Creating a Webhook endpoint

Stripe allows us to create a webhook endpoint to receive events. To tell Stripe
where to send the events, we need to create a webhook endpoint that points to our application.

After creating a Stripe endpoint, we need to add the "Signing Secret" key in Stripe to the production environment variables. To do so, use the environment variables editor in your hosting provider's dashboard.

{% img src="/assets/images/docs/stripe-webhook-config.webp" width="1670" height="368" /%}

### Webhook API Endpoint

The API endpoint needs to point to your application's Stripe webhook: the path is `/app/api/stripe/webhook`. For example, if your application is hosted at `https://myapp.com`, the webhook endpoint would be `https://myapp.com/api/stripe/webhook`.

### Events

The events you want to receive from Stripe are:
- `checkout.session.completed`
- `customer.subscription.updated`
- `customer.subscription.deleted`

If you have any other events you want to receive, you need to add them here.

### Signing Secret

The signing secret is the value of the `STRIPE_WEBHOOK_SECRET` environment variable. 

{% alert type="warning" %}
If you do not follow these steps, you will not be able to receive events from Stripe. This will prevent you from being able to update the subscription status of your users. Please make sure to follow these steps.
{% /alert %}

Before you continue... **please make sure to follow these steps**:

1. Create a webhook endpoint in Stripe
2. Make sure to add the **events** you want to receive from Stripe
3. Copy the **signing secret** to your production environment variables as `STRIPE_WEBHOOK_SECRET`
4. Redeploy your application to apply the changes

### Pricing Table Configuration

Below is the default Pricing Table configuration of the kits. The `PricingTable` component will automatically generate the pricing table based on the Stripe plans you have created and added to the configuration (see below).

We have 3 products (Basic, Pro and Premium), each with 2 plans (monthly, yearly). By default, the Pro plan is set as the recommended plan.

Of course, the below is simply an example. You will need to customize this according to your application's plans.

```tsx
stripe: {
  embedded: true,
  displayMode: StripeCheckoutDisplayMode.Popup,
  products: [
    {
      name: 'Basic',
      description: 'Description of your Basic plan',
      badge: `Up to 20 users`,
      features: [
        'Basic Reporting',
        'Up to 20 users',
        '1GB for each user',
        'Chat Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$9',
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$90',
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
        },
      ],
    },
    {
      name: 'Pro',
      badge: `Most Popular`,
      recommended: true,
      description: 'Description of your Pro plan',
      features: [
        'Advanced Reporting',
        'Up to 50 users',
        '5GB for each user',
        'Chat and Phone Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$29',
          stripePriceId: 'pro-plan-mth',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$200',
          stripePriceId: 'pro-plan-yr',
          trialPeriodDays: 7,
        },
      ],
    },
    {
      name: 'Premium',
      description: 'Description of your Premium plan',
      badge: ``,
      features: [
        'Advanced Reporting',
        'Unlimited users',
        '50GB for each user',
        'Account Manager',
      ],
      plans: [
        {
          name: '',
          price: 'Contact us',
          stripePriceId: '',
          trialPeriodDays: 7,
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ],
}
```

{% alert type="warning" %}
The properties "stripePriceId" are placeholders. You will need to replace them with the IDs of your plans.
{% /alert %}

### Embedded Display Mode

You can choose between two layouts for displaying the embedded checkout:

1. **Popup [default]** - the default layout. The checkout will be displayed as a popup.
2. **Overlay** - the checkout will be displayed as a full page overlay.

### Hosted Stripe Checkout

If you prefer to redirect users to Stripe Checkout, you can set `embedded` to `false`. This is the default configuration in versions before 0.14.0 as embedded checkouts were not available (released in October 2023).

### Additional Configuration

You can also configure the following properties on each `product`:

- `recommended` - set to `true` if you want to highlight a plan as recommended
- `badge` - set to a string if you want to display a badge next to the plan name

You can also configure the following properties on each `plan`:

- `price` - any string that you want to display as the price (e.g. `$9.99` or `Free`)
- `label` - set to a custom string if you want to display a button label
- `href` - set to a custom URL if you want to link to a custom page (for example, a contact page)
- `trialPeriodDays` - set to the number of days you want to offer a free trial for

### Install The Stripe CLI

The Stripe CLI is required to help us test our integration.
Please [follow this guide to install the CLI on your system](https://stripe.com/docs/stripe-cli).

The default recommendation is to use Docker, which Makerkit uses by default.

### Connect CLI with your account

To connect the Stripe CLI with your account, run the following command:

```
npm run stripe:listen
```

This command requires Docker, but you can alternatively install Stripe on your OS and change the command to use `stripe` directly.

The CLI should prompt you to connect your account. If you signed in already, visit the link and follow the instructions.

If you followed the instructions, get back to the terminal, and then copy
the webhook secret to your `STRIPE_WEBHOOK_SECRET` environment variables in the
`.env` file.

Restart the next.js server to apply the changes.

## Billing Portal

MakerKit uses the [Stripe Billing Portal](https://dashboard.stripe.com/test/settings/billing/portal) to let your users manage their invoices and subscriptions.

The portal becomes accessible once the organization subscribes to a plan, which creates a `customerId` property, making it possible for the organization to access the portal.

### Enabling the Billing Portal in test mode

Please visit the [Stripe Billing Page](https://dashboard.stripe.com/test/settings/billing/portal) to enable the Billing Portal.

Choose the settings that best apply to your product and save your changes.

The only required fields to get started are `Terms of Service` and
`Privacy` URLs. You can add any URL at this stage. **However, remember to update these pages when you go live**.

For a technical deep-dive, we recommend reading our blog post where we [build the integration between Stripe and Next.js from scratch](/blog/tutorials/guide-nextjs-stripe). Because the result of the blog post is a simplified version of the Makerkit's implementation, it can be beneficial for understanding its code.


