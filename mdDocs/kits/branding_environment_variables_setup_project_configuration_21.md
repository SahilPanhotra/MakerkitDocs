-----------------
FILE PATH: kits\next-supabase\how-to\setup\branding.mdoc

---
status: "published"

title: "Update your Makerkit application Branding"
label: "Branding"
order: 1
description: "A unique branding is essential to any SaaS. Learn how to update your Makerkit application branding."
---


Most of the branding details of your SaaS will be stored in the main configuration file of your application. This file is located at `src/configurations.ts`.

The `site` object contains most of your branding details. You can update the following properties:

```tsx
site: {
  name: 'Awesomely - Your SaaS Title',
  description: 'Your SaaS Description',
  themeColor: '#ffffff',
  themeColorDark: '#0a0a0a',
  siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
  siteName: 'Awesomely',
  twitterHandle: '',
  githubHandle: '',
  language: 'en',
  convertKitFormId: '',
  locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
},
```

1. The `name` and `description` properties will be used in the meta tags of your application
2. The `themeColor` and `themeColorDark` properties will be used to update the color of the browser bar on mobile devices.
3. The `siteName` property is a short version of your SaaS title, without the subtitle.
4. The `twitterHandle` and `githubHandle` properties will be used to link to your social media accounts (if you have any).
5. The `locale` property is the default language of your application
6. The `language` property is deprecated and will be removed in a future version of Makerkit. Please use the `locale` property instead.
7. The `convertKitFormId` property is used to link your application to your ConvertKit account if you use the Newsletter form.

## Design Branding

Please refer to the [Theming](theming) section to learn how to update the design of your application.

-----------------
FILE PATH: kits\next-supabase\how-to\setup\environment-variables-setup.mdoc

---
status: "published"
title: "Understanding and setting the environment variables in your Next.js Supabase project"
label: "Environment variables"
order: 5
description: "Learn how to set environment variables in your Next.js Supabase project and how to use them in your code."
---

Makerkit ships with some environment variables pre-configured for you.

If you don't have experience with environment variables, you can think of them as a way to store configuration values for your project.

Here are the types of environment variables you can use in your Next.js project:
1. **.env**: This file is used to store environment variables that are common to all environments (development, staging, production). **Only use it for public and shared variables**.
2. **.env.local**: This file is used to store environment variables that are specific to your local development environment. **Do not commit this file to your repository** (this is already ignored by the kit's defaults).
3. **.env.development**: This file is used to store environment variables that are specific to your development environment. As long as you don't use it for sensitive data, you can commit this file to your repository.
4. **.env.production**: This file is used to store environment variables that are specific to your production environment. Do not add sensitive data to this file. Instead, use your CI provider.

## If I set mt variables in Vercel/another hosting provider, do I still need to set them in my project?

No. If you set your environment variables in your hosting provider, you don't need to set them in your project.

## Can I add secrets to my environment variables, like my OpenAI API key?

**No.** You should never add secrets to your environment variables. Instead, you should use your CI provider to set them.

## Does Next.js automatically load my environment variables?

Yes, Next.js automatically loads your environment variables. You can access them in your code using `process.env.VARIABLE_NAME`.

## I added a variable to my .env file, but I can't access it in my code. What's wrong?

Next.js only loads environment variables that start with `NEXT_PUBLIC_` if you are trying to access them from your client-side. If you want to access a variable in your client code, you need to prefix it with `NEXT_PUBLIC_`. **Note**: never add secrets to your client-side code, therefore do not prefix them with `NEXT_PUBLIC_` if it's private data.

-----------------
FILE PATH: kits\next-supabase\how-to\setup\project-configuration.mdoc

---
status: "published"
title: "How to edit your Makerkit project configuration | Next.js Supabase"
label: Project Configuration
order: 0
description: "Learn how and where to edit your project configuration in your Makerkit SaaS application."
---

Makerkit stores your project configuration in a Typescript file named `configuration.ts`, in which we store **non-secret** data needed in the client-side bundles of your project.

## What is stored in the configuration file?

In this configuration file, we store data that is safe to be exposed in the client-side bundles of your project.

The configuration file contains the following data:

```tsx
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

const production = process.env.NODE_ENV === 'production';

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
    siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
  },
  auth: {
    // ensure this is the same as your Supabase project. By default - it's true
    requireEmailConfirmation:
      process.env.NEXT_PUBLIC_REQUIRE_EMAIL_CONFIRMATION === 'true',
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
  environment: process.env.NEXT_PUBLIC_ENVIRONMENT,
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    signInMfa: '/auth/verify',
    onboarding: `/onboarding`,
    appPrefix: '/dashboard',
    appHome: '/dashboard',
    authCallback: '/auth/callback',
    settings: {
      profile: 'settings/profile',
      authentication: 'settings/profile/authentication',
      email: 'settings/profile/email',
      password: 'settings/profile/password',
    },
  },
  sentry: {
    dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  },
  stripe: {
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
          },
          {
            name: 'Yearly',
            price: '$90',
            stripePriceId: '<price_id>',
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
            stripePriceId: '<price_id>',
          },
          {
            name: 'Yearly',
            price: '$200',
            stripePriceId: '<price_id>',
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

Please refer to the Next.js official documentation on how to best store your secrets.

-----------------
FILE PATH: kits\next-supabase\how-to\setup\set-up-emails.mdoc

---
status: "published"
title: "How to setup emails in your Makerkit SaaS application | Next.js Supabase SaaS Kit"
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

Makerkit uses InBucket to allow you testing your emails without using a real
SMTP service. You can access InBucket at [localhost:54324](http://localhost:54324)

-----------------
FILE PATH: kits\next-supabase\how-to\setup\updating-favicon.mdoc

---
status: "published"
title: "Updating the favicons of your site | Next.js Supabase"
label: Updating the Favicons
order: 4
description: "Learn how to update the favicons of your Makerkit Next.js Supabase SaaS landing pages"
---

If you have a logo for your company, you can easily update the favicons of your Makerkit applications. Favicons are the small icons that appear in the browser tab of your website.

### Where to find the favicons

Favicons are stored at `public/assets/images/favicon`. You can replace the existing files with your own images. If you keep the same names and sizes, then you don't need to update the code.

### How to generate favicons

If you do, I suggest to generate them using any o the online favicon generator, and then dropping the resulting images in the `public/assets/images/favicon` folder.

If you have different sizes and names, then you need to update the code to reflect these changes.

## How to update the favicons in Makerkit

This code is stored in the root layout:

 ```tsx {% title="app/layout.tsx" %}
icons: {
  icon: '/assets/images/favicon/favicon.ico',
  shortcut: '/shortcut-icon.png',
  apple: '/assets/images/favicon/apple-touch-icon.png',
  other: {
    rel: 'apple-touch-icon-precomposed',
    url: '/apple-touch-icon-precomposed.png'
  },
}
```

Update the paths to match your new favicons.


-----------------
FILE PATH: kits\next-supabase\how-to\setup\updating-fonts.mdoc

---
status: "published"

title: "Updating the fonts of your Makerkit site"
label: Updating the Fonts
order: 3
description: "Learn how to update the fonts of your Makerkit landing pages"
---


By default, Makerkit defines the site's fonts using the package `next/font/google`.

The fonts are defined at `src/components/Fonts.tsx` and you can update them to your liking.

### What fonts are used by default?

By default, **Makerkit uses the system Apple font on Apple devices**, the `Inter` font as fallback.

### Removing Apple font as default

If you want to remove the `Apple` font as default, simply update the `FontsFamily` component at `src/components/Fonts.tsx` to remove the `Apple` font.

 ```tsx {% title="src/components/Fonts.tsx" %} {26-27}
'use client';

import { Inter as SansFont } from 'next/font/google';
import { useServerInsertedHTML } from 'next/navigation';

const sans = SansFont({
  subsets: ['latin'],
  variable: '--font-family-sans',
  fallback: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
  preload: true,
  weight: ['300', '400', '500', '600', '700', '800'],
});

// replace with your heading font
// by default, it will use the sans font
const heading = sans;

function Fonts() {
  useServerInsertedHTML(() => {
    return (
      <style
        key={'fonts'}
        dangerouslySetInnerHTML={{
          __html: `
          :root {
            --font-family-sans: ${sans.style.fontFamily}, 'Segoe UI', 'Roboto', 'Ubuntu', 'sans-serif';
            --font-family-heading: ${heading.style.fontFamily};
          }
        `,
        }}
      />
    );
  });

  return null;
}

export default Fonts;
```

If you compare the above with the version of the file you have in your project, you'll notice that we removed the `--apple-system` fonts.

### Using a different font

If you want to use a different font, you can import it from the `next/font/google` package.

For example, below, we import the font `Manrope` from Google Fonts. It replaces `Inter` as the default font.

```tsx
import { Manrope as SansFont } from 'next/font/google';
```

You can use two different fonts for the `--font-family-sans` and `--font-family-heading` variables. One is used an the main body font, the other as the heading font.

-----------------
FILE PATH: kits\next-supabase\how-to\setup\updating-logo.mdoc

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

1. `src/core/ui/Logo/LogoImage.tsx`
2. `src/core/ui/Logo/LogoImageMini.tsx`

Feel free to replace the default logo with your own logo.

### What sizes should my logo be?

The optimal sizes for the logs are:

1. The logos should be 20px tall to fit well in the header.
2. The logo mini should be 20px tall and 20px wide.

### I like Makerkit's logo! How can I reuse it for my own logo?

[Download the Makerkit Figma file from here](/assets/makerkit.fig) and use the logo from there.

-----------------
FILE PATH: kits\next-supabase\how-to\site\adding-marketing-pages.mdoc

---
status: "published"

title: "Adding pages to the marketing site of your Makerkit Next.js Supabase project"
label: Adding Pages
order: 3
description: "Learn how to add new pages to the marketing site of your Makerkit Next.js Supabase project."
---


To add a new page to the marketing site of your Next.js project, you can create a new page in the `src/app/(site)` directory.

By "marketing site", we mean the public pages of your project, such as the homepage, the about page, the contact page, etc.

{% alert type="warning" %}
      Use the below code only for public pages accessible by everyone, such as the homepage, the about page, the contact
      page, etc.

{% /alert %}

### Adding a new page to the marketing site

As you know already, we can add pages to our Next.js App Router application by creating files within the `src/app` directory named as `page.tsx`.

For example, if you want to add a page at `/about`, you can create a new file at `src/app/(site)/about/page.tsx` with the following content:

 ```tsx {% title="src/app/(site)/about/page.tsx" %}
function AboutPage() {
  return <div>About page</div>;
}

export default AboutPage;
```

This page will **automatically inherit the site layout** from `src/app/(site)/layout.tsx` - which means it will already display the header and footer.

#### I don't want to use the same layout though?

If you don't want to use the site layout, create a route group inside `src/app`, and add your page there.

### Adding a new page to the marketing site with translations

If you have translations on your marketing site - to ensure the page is translated correctly server-side, we use the `withI18n` HOC:

 ```tsx {% title="src/app/(site)/about/page.tsx" %}
import { withI18n } from '~/i18n/with-i18n';

function AboutPage() {
  return <div>About page</div>;
}

export default withI18n(AboutPage);
```


-----------------
FILE PATH: kits\next-supabase\how-to\site\updating-navigation-menu.mdoc

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
FILE PATH: kits\next-supabase\how-to\translations\adding-locales.mdoc

---
status: "published"
title: "Adding a new language | Next.js Supabase SaaS Kit"
label: "Adding a new language in the Next.js Supabase SaaS Kit"
order: 2
description: "How to add a new language to your Next.js Supabase application."
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
FILE PATH: kits\next-supabase\how-to\translations\adding-translation-file.mdoc

---
status: "published"
title: "Adding a new translation file | Next.js Supabase SaaS Kit"
label: "Adding a new translation file"
order: 3
description: "How to add a new translation file to your Next.js Supabase SaaS Kit application"
---

To add a new translation to your application next to the default ones (common, auth, profile, etc.) you need to:

1. create the new JSON file, for example `tasks.json` in the `public/locales/<lang>` folder
2. add the new bundle name to the translations list in the `src/i18n.settings.ts` file:

```typescript
export const defaultI18nNamespaces = [
  'common',
  'auth',
  'organization',
  'profile',
  'subscription',
];
```

We add the `tasks` bundle name to the `defaultI18nNamespaces`:

```typescript {7}
export const defaultI18nNamespaces = [
  'common',
  'auth',
  'organization',
  'profile',
  'subscription',
  'tasks'
];
```

Now, refresh the dev server and your new bundled will be picked up by the i18n module.

-----------------
FILE PATH: kits\next-supabase\how-to\translations\adding-translation.mdoc

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
FILE PATH: kits\next-supabase\how-to\translations\default-language.mdoc

---
status: "published"

title: "Setting a Default Language"
label: "Setting a Default Language"
order: 1
description: "Learn how to set a default language for your Makerkit application."
---


Setting a default language in your Next.js Supabase application is easy.

Simply set the environment variable `DEFAULT_LOCALE` to the language code you want to use as the default language.

By default, we use `en` as the default language. If you want to change it, simply add the following line to your `.env` file. For example, if you want to use Spanish as the default language, you would add the following line to your `.env` file:

```bash
DEFAULT_LOCALE=es
```

-----------------
FILE PATH: kits\next-supabase\how-to\translations\detect-current-locale.mdoc

---
status: "published"
title: "Detecting the Currently selected locale | Next.js Supabase SaaS Kit"
label: "Detect current Locale"
order: 5
description: "How to detect the currently selected locale so that you can use it in your React components."
---

To detect the currently selected locale from the client, you can use the `useTranslation` hook from `react-i18next`.

```tsx
import { useTranslation } from 'react-i18next';

function Component() {
  const { i18n } = useTranslation();
  const { language } = i18n;

  // use language here
}
```

### Detecting the locale from the server

When rendering on the server, you can use the `getLanguageCookie` object to detect the locale. This value represents the locale that was selected by the user.

NB: it can be null if the user has not selected a locale yet or if the user has never selected a locale. In this case, **you should use the default locale**.

```tsx
import getLanguageCookie from '~/i18n/get-language-cookie';
import configuration from '~/configuration';

function ServerComponent() {
  const locale = getCurrentLocale();

  // use locale here
}

function getCurrentLocale() {
  return getLanguageCookie() ?? configuration.site.locale;
}
```



-----------------
FILE PATH: kits\next-supabase\how-to\translations\language-switcher.mdoc

---
status: "published"
title: "Using the Language Switcher | Next.js Supabase SaaS Kit"
label: "Using the Language Switcher"
order: 4
description: "Import and use the LanguageSwitcher component in your Next.js Supabase application."
---

By default, the language switcher is not used within the Next.js Supabase starter. To use it, you will need to import it into your application and add it to your layout.

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
FILE PATH: kits\next-supabase\how-to\troubleshooting\403-error-actions.mdoc

---
status: "published"
title: "How to fix a 403 Error on Actions and API Routes | Next.js Supabase Kit"
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
FILE PATH: kits\next-supabase\how-to\troubleshooting\contentlayer-stuck.mdoc

---
status: "published"
title: "Contentlayer gets stuck when starting the Makerkit application | Next.js Supabase SaaS Kit"
label: "Contentlayer gets stuck"
order: 3
description: "Contentlayer gets stuck when starting the Next.js Supabase application"
---

A small percentage of customers have reported issues when starting the kit because [Contentlayer](https://contentlayer.dev) gets stuck.

The only known possible fix is to downgrade both the packages `contentlayer` and `next-contentlayer` to version `0.3.1`:

```bash
npm i contentlayer@0.3.1 next-contentlayer@0.3.1
```

If it keeps happening, please contact me and I'll help you out.

-----------------
FILE PATH: kits\next-supabase\how-to\troubleshooting\dynamic-server-usage-error.mdoc

---
status: "published"
title: "How to fix the Next.js error Dynamic Server Usage in Next.js Supabase"
label: "Dynamic server usage error"
order: 4
description: "The error Dynamic server usage Page couldn't be rendered statically because it used `cookies` is a common issue in Next.js. Here is the fix."
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
FILE PATH: kits\next-supabase\how-to\troubleshooting\supabase-not-starting.mdoc

---
status: "published"
title: "How to fix: Supabase is not starting | Next.js Supabase Kit"
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

