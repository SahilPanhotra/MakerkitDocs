-----------------
FILE PATH: kits\remix-supabase\how-to\setup\environment-variables-setup.mdoc

---
status: "published"
title: "Understanding and setting the environment variables in your Remix Supabase project"
label: "Environment variables"
order: 5
description: "Learn how to set environment variables in your Remix Supabase project and how to use them in your code."
---

Makerkit ships with some environment variables pre-configured for you.

If you don't have experience with environment variables, you can think of them as a way to store configuration values for your project.

Remix ships with a `.env` file that contains the following variables:

```bash
DEFAULT_LOCALE=en
SITE_URL=http://localhost:3000

# set the below to "production" in your production environment
ENVIRONMENT=development

# SUPABASE
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# STRIPE
STRIPE_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# EMAIL
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

NB: Remix only used the `.env` file during development. If you want to use environment variables in production, you need to set them in your hosting provider.

## If I set mt variables in Vercel/another hosting provider, do I still need to set them in my project?

No. If you set your environment variables in your hosting provider, you don't need to set them in your project.

## Can I add secrets to my environment variables, like my OpenAI API key?

**No.** You should never add secrets to your environment variables. Instead, you should use your CI provider to set them.

## Does Remix automatically load my environment variables?

No - or better - only during development. Which means that you need to set them in your hosting provider if you want to use them in production.


-----------------
FILE PATH: kits\remix-supabase\how-to\setup\project-configuration.mdoc

---
status: "published"

title: "How to edit your Makerkit project configuration"
label: Project Configuration
order: 0
description: "Learn how and where to edit your project configuration in your Makerkit SaaS application."
---


Makerkit stores your project configuration in a Typescript file named `configuration.ts`, in which we store **non-secret** data needed in the client-side bundles of your project.

## What is stored in the configuration file?

In this configuration file, we store data that is safe to be exposed in the client-side bundles of your project.

The configuration file contains the following data:

```tsx
import getEnv from '~/core/get-env';
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

const env = getEnv() ?? {};
const production = env.NODE_ENV === 'production';

enum Themes {
  Light = 'light',
  Dark = 'dark',
}

const configuration = {
  site: {
    name: 'Awesomely - Your SaaS Title',
    description: 'Your SaaS Description',
    themeColor: '#ffffff',
    themeColorDark: '#0a0a0a',
    siteUrl: env.SITE_URL,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: env.DEFAULT_LOCALE,
  },
  auth: {
    // ensure this is the same as your Supabase project
    requireEmailConfirmation: true,
    // NB: Enable the providers below in the Supabase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: ['google'] as Provider[],
    },
  },
  production,
  environment: env.ENVIRONMENT,
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Light,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    signInMfa: '/auth/verify',
    signInFromLink: '/auth/link',
    callback: '/auth/callback',
    onboarding: `/onboarding`,
    appHome: '/dashboard',
    settings: {
      profile: '/settings/profile',
      authentication: '/settings/profile/authentication',
      email: '/settings/profile/email',
      password: '/settings/profile/password',
    },
    api: {
      checkout: `/resources/ls/checkout`,
      customerPortal: `/resources/ls/customer-portal`,
      organizations: {
        create: `/resources/organizations/create`,
        transferOwnership: `/resources/organizations/transfer-ownership`,
        members: `/resources/organizations/members`,
      },
    },
  },
  sentry: {
    dsn: env.SENTRY_DSN,
  },
  subscriptions: {
    products: [
      {
        name: 'Basic',
        productId: 57713,
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
            variantId: 55476,
          },
          {
            name: 'Yearly',
            price: '$90',
            variantId: 55512,
          },
        ],
      },
      {
        name: 'Pro',
        badge: `Most Popular`,
        recommended: true,
        productId: 57719,
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
            variantId: 55483,
            trialPeriodDays: 0,
          },
          {
            name: 'Yearly',
            price: '$200',
            variantId: 55482,
            trialPeriodDays: 0,
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
            variantId: 0,
            label: `Contact us`,
            href: `/contact`,
          },
        ],
      },
    ],
  },
};

export default configuration;
```

## How should I store my secrets?

You should store your secrets in your **non public** environment variables, and ideally set up from your CI or Hosting Provider (such as Vercel).

Never add secrets to the configuration file, as it is exposed in the client-side bundles of your project.

Please refer to the Remix official documentation on how to best store your
secrets.

-----------------
FILE PATH: kits\remix-supabase\how-to\setup\set-up-emails.mdoc

---
status: "published"
title: "How to setup emails in your Makerkit SaaS application | Remix Supabase SaaS Kit"
label: Setting up Emails
order: 6
description: "How to setup emails in your Makerkit SaaS application"
---

Sending emails is a critical part of your SaaS application. Makerkit uses emails to send invites to your users only, but it's likely you'll want to send other emails to your users.

To set up emails in your Makerkit application, you should add the following environment variables to your project, using a secure environment:

```bash
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

Makerkit will use these values to send emails on your behalf using the node library `nodemailer`.

NB: this does not refer to emails sent by Supabase. These have to be setup
from within the Supabase Dashboard. You can reuse the same SMTP settings.

### Where do I get these values?

These values are normally provided by your service provider.

### Which services are supported?

Any service that supports SMTP should work. We recommend using [Resend](https://resend.com/).

### How do I test emails locally?

Makerkit uses InBucket to test emails locally without the need for an SMTP
service. You can access InBucket at [localhost:54324](https://localhost:54324).

-----------------
FILE PATH: kits\remix-supabase\how-to\setup\updating-favicon.mdoc

---
status: "published"
title: "Updating the favicons of your Makerkit site | Remix Supabase"
label: Updating the Favicons
order: 4
description: "Learn how to update the favicons of your Makerkit Remix Supabase landing pages"
---

If you have a logo for your company, you can easily update the favicons of your Makerkit applications. Favicons are the small icons that appear in the browser tab of your website.

### Where to find the favicons

Favicons are stored at `public/assets/images/favicon`. You can replace the existing files with your own images. If you keep the same names and sizes, then you don't need to update the code.

### How to generate favicons

If you do, I suggest to generate them using any o the online favicon generator, and then dropping the resulting images in the `public/assets/images/favicon` folder.

If you have different sizes and names, then you need to update the code to reflect these changes.

## How to update the favicons in Makerkit

This code is stored in the root `Head` component at `app/core/ui/Head.tsx`. You can update the code to match your new favicons.

 ```tsx {% title="app/core/ui/Head.tsx" %}
<link rel="shortcut icon" href="/assets/images/favicon/favicon.ico" />

<link
  rel="apple-touch-icon"
  sizes="144x144"
  href="/assets/images/favicon/apple-touch-icon.png"
/>

<link
  rel="icon"
  type="image/png"
  sizes="16x16"
  href="/assets/images/favicon/favicon-16x16.png"
/>

<link
  rel="icon"
  type="image/png"
  sizes="32x32"
  href="/assets/images/favicon/favicon-32x32.png"
/>

<link rel="manifest" href="/assets/images/favicon/site.webmanifest" />

<link
  rel="mask-icon"
  href="/assets/images/favicon/safari-pinned-tab.svg"
  color="#000000"
/>
```

Update the paths to match your new favicons.


-----------------
FILE PATH: kits\remix-supabase\how-to\setup\updating-fonts.mdoc

---
status: "published"

title: "Updating the fonts of your Makerkit site"
label: Updating the Fonts
order: 3
description: "Learn how to update the fonts of your Makerkit landing pages"
---


### What fonts are used by default?

By default, **Makerkit uses the system Apple font on Apple devices**, the `Inter` font as fallback.

### Removing Apple font as default

If you want to remove the `Apple` font as default, update the fonts in the Tailwind config file.

```js
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
}
```

If you are importing a font from Google Fonts, update the font in the `index.css` file.

 ```css {% title="app/styles/index.css" %}
@import url('//fonts.googleapis.com/css2?family=Inter:wght@300;400;500;700;800&display=swap&display=swap');
```

The above uses Inter, but you can update with the value set in the Tailwind config file.

NB: the font loaded in the CSS file **must match the font name in the Tailwind config file**.

-----------------
FILE PATH: kits\remix-supabase\how-to\setup\updating-logo.mdoc

---
status: "published"

title: "Updating the logo of your Makerkit site"
label: Updating the Logo
order: 2
description: "Learn how to update the logos of your Makerkit site"
---


Makerkit by default provides an SVG logo with the default Makerkit logo. In addition, we also provide a `mini` version of the logo that is used when the sidebar is collapsed.

You should change both the `LogoImage` and `LogoImageMini` to match your brand.

### Where are the logos defined?

The logos are defined in two files:

1. `app/core/ui/Logo/LogoImage.tsx`
2. `app/core/ui/Logo/LogoImageMini.tsx`

Feel free to replace the default logo with your own logo.

### What sizes should my logo be?

The optimal sizes for the logs are:

1. The logos should be 20px tall to fit well in the header.
2. The logo mini should be 20px tall and 20px wide.

### I like Makerkit's logo! How can I reuse it for my own logo?

[Download the Makerkit Figma file from here](/assets/makerkit.fig) and use the logo from there.

-----------------
FILE PATH: kits\remix-supabase\how-to\site\adding-marketing-pages.mdoc

---
status: "published"

title: "Adding pages to the marketing site of your Makerkit Remix Supabase project"
label: Adding Pages
order: 3
description: "Learn how to add new pages to the marketing site of your Makerkit Remix Supabase project."
---


To add a new page to the marketing site of your Remix project, you can create a new page in the `app/routes/_site` layout.

By "marketing site", we mean the public pages of your project, such as the homepage, the about page, the contact page, etc.

{% alert type="warning" %}
      Use the below code only for public pages accessible by everyone, such as the homepage, the about page, the contact
      page, etc.

{% /alert %}


### Adding a new page to the marketing site

For example, if you want to add a page at `/about`, you can create a new file at `app/routes/_site.about._index.tsx` with the following content:

 ```tsx {% title="app/_site.about._index.tsx" %}
function AboutPage() {
  return <div>About page</div>;
}

export default AboutPage;
```

This page will **automatically inherit the site layout** from `app/routes/_site.tsx` - which means it will already display the header and footer.

#### I don't want to use the same layout though?

If you don't want to use the site layout, create a new group inside `app/routes`, and add your page there. This replaces the `_site` layout with your own layout.


-----------------
FILE PATH: kits\remix-supabase\how-to\site\updating-navigation-menu.mdoc

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
FILE PATH: kits\remix-supabase\how-to\translations\adding-locales.mdoc

---
status: "published"
title: "Adding a new language | Remix Supabase SaaS Kit"
label: "Adding a new language in the Remix Supabase SaaS Kit"
order: 2
description: "How to add a new language to your Remix Supabase application."
---

To add a new locale to your application, you need to create a new file in the `public/locales` directory.

For example, to add a new language called `fr` (French), you would create a new file called `fr.json` in the `public/locales` directory.

```json
{
  "hello": "Bonjour"
}
```

That's it! Now you can use the `hello` key in your application and it will be translated to `Bonjour` when the locale is set to `fr`.

-----------------
FILE PATH: kits\remix-supabase\how-to\translations\adding-translation-file.mdoc

---
status: "published"
title: "Adding a new translation file | Remix Supabase SaaS Kit"
label: "Adding a new translation file"
order: 3
description: "How to add a new translation file to your Remix Supabase SaaS Kit application"
---

To add a new translation to your application next to the default ones (common, auth, profile, etc.) you need to:

1. create the new JSON file, for example `tasks.json` in the `public/locales/<lang>` folder
2. add the new bundle name to the translations list in the `app/i18n/i18next.config.ts` file:

```typescript
import isBrowser from '~/core/generic/is-browser';

const env = isBrowser() ? window.ENV : process.env;
const DEFAULT_LOCALE = env.DEFAULT_LOCALE ?? 'en';

const i18Config = {
  fallbackLanguage: DEFAULT_LOCALE,
  supportedLanguages: [DEFAULT_LOCALE],
  defaultNS: ['common', 'auth', 'organization', 'profile', 'subscription'],
  react: { useSuspense: false },
};

export default i18Config;

```

We add the `tasks` bundle name to the `i18Config.defaultNS` array:

```typescript {4}
const i18Config = {
  fallbackLanguage: DEFAULT_LOCALE,
  supportedLanguages: [DEFAULT_LOCALE],
  defaultNS: ['common', 'auth', 'organization', 'profile', 'subscription', 'tasks'],
  react: { useSuspense: false },
};
```

Now, refresh the dev server and your new bundled will be picked up by the i18n module.

-----------------
FILE PATH: kits\remix-supabase\how-to\translations\adding-translation.mdoc

---
status: "published"

title: "Adding a new translation string"
label: "Adding a new translation string"
order: 0
description: "Adding new translations to your application is easy. This guide will show you how to add a new translation string to your application."
---


The translations of the application are stored at `public/locales/<en>`.

We have various translations languages JSON files by default (only in English):
- `public/locales/en/common.json` - where we store the common translations
- `public/locales/en/auth.json` - where we store the auth translations
- `public/locales/en/profile.json` - where we store the profile settings translations
- `public/locales/en/organization.json` - where we store the organization settings translations
- `public/locales/en/subscription.json` - where we store the subscription settings translations

You can keep adding translations to these files if you wish, or create new bundles for your translations.

## Adding a new translation string

Let's say we want to add a new translation string to the `common.json` file.

1. Open the `public/locales/en/common.json` file and add the new translation string:

```json
{
  "new_translation_string": "This is a new translation string"
}
```

You can then reference this key in your application like this:
1. Using the `Trans` component from `~/core/ui/Trans`
2. Using the `useTranslation` hook from `react-i18next`

For example, using the `Trans` component:

```tsx
<Trans i18nKey="common:new_translation_string" />
```

or using the `useTranslation` hook:

```tsx
const { t } = useTranslation();
const newTranslationString = t('common:new_translation_string');
```


-----------------
FILE PATH: kits\remix-supabase\how-to\translations\default-language.mdoc

---
status: "published"

title: "Setting a Default Language"
label: "Setting a Default Language"
order: 1
description: "Learn how to set a default language for your Makerkit application."
---


Setting a default language in your Remix Supabase application is easy.

Simply set the environment variable `DEFAULT_LOCALE` to the language code you want to use as the default language.

By default, we use `en` as the default language. If you want to change it, simply add the following line to your `.env` file. For example, if you want to use Spanish as the default language, you would add the following line to your `.env` file:

```bash
DEFAULT_LOCALE=es
```

Make sure to set this environment variable in your hosting provider as well for your production application.

-----------------
FILE PATH: kits\remix-supabase\how-to\translations\language-switcher.mdoc

---
status: "published"
title: "Using the Language Switcher | Remix Supabase SaaS Kit"
label: "Using the Language Switcher"
order: 4
description: "Import and use the LanguageSwitcher component in your Remix Supabase application."
---

By default, the language switcher is not used within the Remix Supabase starter. To use it, you will need to import it into your application and add it to your layout.

To do so, import the `LanguageSwitcher` component from the `components` directory and add it to any component you want to display it in.

```tsx
import { LanguageSwitcher } from '~/components/LanguageSwitcher';

export const YourComponent = ({ children }) => {
  return (
    <div>
      <LanguageSwitcher />
      {children}
    </div>
  );
};
```

The language switcher will automatically display the current language and a list of available languages. When a user clicks on a language, the language will be updated in the user's profile and the page will be reloaded.

-----------------
FILE PATH: kits\remix-supabase\how-to\troubleshooting\contentlayer-stuck.mdoc

---
status: "published"
title: "Contentlayer gets stuck when starting the Remix Supabase SaaS Kit application"
label: "Contentlayer gets stuck"
order: 3
description: "Contentlayer gets stuck when starting the Makerkit application (Remix Supabase)"
---

A small percentage of customers have reported issues when starting the kit because [Contentlayer](https://contentlayer.dev) gets stuck.

The only known possible fix is to downgrade both the packages `contentlayer` and `next-contentlayer` to version `0.3.1`:

```bash
npm i contentlayer@0.3.1 next-contentlayer@0.3.1
```

If it keeps happening, please contact me and I'll help you out.

-----------------
FILE PATH: kits\remix-supabase\how-to\troubleshooting\supabase-not-starting.mdoc

---
status: "published"
title: "How to fix: Supabase is not starting | Remix Supabase Kit"
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
FILE PATH: kits\remix-supabase\how-to\troubleshooting\supabase-not-stopping.mdoc

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
FILE PATH: kits\remix-supabase\how-to\troubleshooting\tables-not-found.mdoc

---
status: "published"
title: "How to fix: Supabase tables not found | Remix Supabase SaaS Kit"
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
FILE PATH: kits\remix-supabase\how-to\ui\dark-theme.mdoc

---
status: "published"
title: "Setting up the dark theme in Remix Supabase"
label: Dark Theme
order: 1
description: "How to set up the dark theme in Remix Supabase"
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
    enableThemeSwitcher: false,
  },
  theme: Themes.Light,
}
```

In the above, we flipped the `enableThemeSwitcher` to `false` and set the `theme` to `Themes.Light`.

-----------------
FILE PATH: kits\remix-supabase\how-to\ui\theming.mdoc

---
status: "published"
title: Theming | Remix Supabase SaaS Kit
label: Theming
order: 1
description: "Discover how to apply the ShadCN UI theme to your Remix Supabase SaaS Kit project."
---

Makerkit's theming is defined in two places:

1. the `global.css` file
2. the `tailwind.config.js` file.

## Colors

To update the theme's colors, please update the `global.css` file in your project and update the CSS variables with your color palette.

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

