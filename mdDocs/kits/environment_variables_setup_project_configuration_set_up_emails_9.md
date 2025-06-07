-----------------
FILE PATH: kits/next-fire/how-to/setup/environment-variables-setup.mdoc

---
status: "published"
title: "Understanding and setting the environment variables in your Next.js Firebase project"
label: "Environment variables"
order: 5
description: "Learn how to set environment variables in your Next.js Firebase project and how to use them in your code."
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
FILE PATH: kits/next-fire/how-to/setup/project-configuration.mdoc

---
status: "published"
title: "How to edit your Makerkit project configuration | Next.js Firebase"
label: Project Configuration
order: 0
description: "Learn how and where to edit your project configuration in your Next.js Firebase Makerkit SaaS application."
---

Makerkit stores your project configuration in a Typescript file named `configuration.ts`, in which we store **non-secret** data needed in the client-side bundles of your project.

## What is stored in the configuration file?

In this configuration file, we store data that is safe to be exposed in the client-side bundles of your project.

The configuration file contains the following data:

```tsx
import { LayoutStyle } from '~/core/layout-style';
import { GoogleAuthProvider } from 'firebase/auth';

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
    siteUrl: process.env.NEXT_PUBLIC_SITE_URL as string,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: process.env.DEFAULT_LOCALE,
  },
  firebase: {
    apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
    authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
    storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
    appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
    measurementId: process.env.NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID,
  },
  auth: {
    // Enable MFA. You must upgrade to GCP Identity Platform to use it.
    // see: https://cloud.google.com/identity-platform/docs/product-comparison
    enableMultiFactorAuth: true,
    // When enabled, users will be required to verify their email address
    // before being able to access the app
    requireEmailVerification:
      process.env.NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION === 'true',
    // NB: Enable the providers below in the Firebase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: [GoogleAuthProvider],
    },
  },
  environment: process.env.NEXT_PUBLIC_VERCEL_ENV ?? 'development',
  emulatorHost: process.env.NEXT_PUBLIC_EMULATOR_HOST,
  emulator: process.env.NEXT_PUBLIC_EMULATOR === 'true',
  production: process.env.NODE_ENV === 'production',
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Dark,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    emailLinkSignIn: '/auth/link',
    onboarding: `/onboarding`,
    appHome: '/dashboard',
    settings: {
      profile: '/settings/profile',
      authentication: '/settings/profile/authentication',
      email: '/settings/profile/email',
      password: '/settings/profile/password',
    },
    api: {
      checkout: `/api/stripe/checkout`,
      billingPortal: `/api/stripe/portal`,
    },
  },
  navigation: {
    style: LayoutStyle.Sidebar,
  },
  appCheckSiteKey: process.env.NEXT_PUBLIC_APPCHECK_KEY,
  sentry: {
    dsn: process.env.SENTRY_DSN,
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
FILE PATH: kits/next-fire/how-to/setup/set-up-emails.mdoc

---
status: "published"
title: "How to setup emails in your Makerkit SaaS application | Next.js Firebase SaaS Kit"
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

NB: this does not refer to emails sent by Firebase.

### Where do I get these values?

These values are normally provided by your service provider.

### Which services are supported?

Any service that supports SMTP should work. We recommend using [Resend](https://resend.com/).

### How do I test emails locally?

Makerkit uses [InBucket](https://inbucket.org) - a platform to testing emails.

InBucket saves time during development since we can test
our emails without setting up a real SMTP service - and works locally and
offline.

To run the InBucket platform, **we need Docker running on our machine**.

To start the InBucket service, run the following command:

```
npm run inbucket:start
```

InBucket will now be running at [localhost:9000](http://localhost:9000). When we send an email using the
`sendEmail` function in the kit, we can visualize it using the InBucket UI.

InBucket is used by default during development. Instead, for production
usage, you will need to set up a real SMTP service.

#### Ethereal

In previous versions Makerkit used Ethereal. If you are running an older version, please refer to the below.

Makerkit used Ethereal to allow you testing your emails without using a real
email account. To use Ethereal, you should add the following environment variables to your project, using a secure environment:

```bash
ETHEREAL_EMAIL=
ETHEREAL_PASSWORD=
```

If not set, Makerkit will automatically create an account for you on-the-fly and print the credentials in the console. You can then use these credentials to log in to Ethereal and see your emails.

-----------------
FILE PATH: kits/next-fire/how-to/setup/updating-favicon.mdoc

---
status: "published"
title: "Updating the favicons of your Makerkit site | Next.js Firebase SaaS Kit"
label: Updating the Favicons
order: 4
description: "Learn how to update the favicons of your Next.js Firebase SaaS Kit landing pages"
---

If you have a logo for your company, you can easily update the favicons of your Makerkit applications. Favicons are the small icons that appear in the browser tab of your website.

### Where to find the favicons

Favicons are stored at `public/assets/images/favicon`. You can replace the existing files with your own images. If you keep the same names and sizes, then you don't need to update the code.

### How to generate favicons

If you do, I suggest to generate them using any o the online favicon generator, and then dropping the resulting images in the `public/assets/images/favicon` folder.

If you have different sizes and names, then you need to update the code to reflect these changes.

## How to update the favicons in Makerkit

This code is stored in `Layout` component at `src/core/ui/Layout.tsx`. This component has many default settings that includes the favicons.

```tsx
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

Update the code above to match your new favicons.

-----------------
FILE PATH: kits/next-fire/how-to/setup/updating-fonts.mdoc

---
status: "published"
title: "Updating the fonts of your Makerkit site"
label: Updating the Fonts
order: 3
description: "Learn how to update the fonts of your Makerkit landing pages"
---

By default, Makerkit defines the site's fonts using the package `next/font/google`.

The fonts are defined at `src/_app.tsx` and you can update them to your liking.

### What fonts are used by default?

By default, **Makerkit uses the system Apple font on Apple devices**, the `Inter` font as fallback.

### Removing Apple font as default

If you want to remove the `Apple` font as default, simply update the `FontsFamily` component at `src/_app.tsx` to remove the `Apple` font.

 ```tsx {% title="src/_app.tsx" %}
function FontFamily() {
  return (
    <style jsx global>
      {`
        html {
          --font-family-sans: ${fontFamilySans.style.fontFamily}, 'Segoe UI', 'Roboto', 'Ubuntu',
            'sans-serif';

          --font-family-heading: ${fontFamilyHeading.style.fontFamily};
        }
      `}
    </style>
  );
}
```

If you compare the above with the version of the file you have in your project, you'll notice that we removed the `--apple-system` fonts.

### Using a different font

If you want to use a different font, you can import it from the `next/font/google` package.

For example, below, we import the font `Manrope` from Google Fonts. It replaces `Inter` as the default font.

 ```tsx {% title="src/_app.tsx" %}
import { Manrope as SansFont } from 'next/font/google';
```

You can use two different fonts for the `--font-family-sans` and `--font-family-heading` variables. One is used an the main body font, the other as the heading font.

-----------------
FILE PATH: kits/next-fire/how-to/setup/updating-logo.mdoc

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
FILE PATH: kits/next-fire/how-to/site/adding-marketing-pages.mdoc

---
status: "published"

title: "Adding pages to the marketing site of your Makerkit Next.js Firebase project"
label: Adding Pages
order: 3
description: "Learn how to add new pages to the marketing site of your Makerkit Next.js Firebase project."
---


To add a new page to the marketing site of your Next.js project, you can create a new page in the `pages` directory and using the required `getStaticProps` function.

By "marketing site", we mean the public pages of your project, such as the homepage, the about page, the contact page, etc.

{% alert type="warning" %}
      Use the below code only for public pages accessible by everyone, such as the homepage, the about page, the contact
      page, etc.

{% /alert %}

### Adding a new page to the marketing site

For example, if you want to add a page at `/about`, you can create a new file at `pages/about.tsx` with the following content:

```tsx
import type { GetStaticPropsContext } from "next";
import { withTranslationProps } from '~/lib/props/with-translation-props';

function AboutPage() {
  return <div>About page</div>;
}

export function getStaticProps(ctx: GetStaticPropsContext) {
  return withTranslationProps(ctx);
}
```

The `withTranslationProps` function is required to make sure that the page is translated correctly.

### Adding basic layout components

If you want to add a basic layout to your page, you can use the `SiteHeader` and `Footer` components:

```tsx
import type { GetStaticPropsContext } from "next";
import { withTranslationProps } from '~/lib/props/with-translation-props';

import Footer from '~/components/Footer';
import SiteHeader from '~/components/SiteHeader';
import Layout from '~/core/ui/Layout';
import Container from '~/core/ui/Container';

function AboutPage() {
  return (
    <Layout>
      <SiteHeader />
      <Container>
        <div>About page</div>
      </Container>
      <Footer />
    </Layout>
  );
}

export function getStaticProps(ctx: GetStaticPropsContext) {
  return withTranslationProps(ctx);
}
```

-----------------
FILE PATH: kits/next-fire/how-to/site/updating-navigation-menu.mdoc

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
FILE PATH: kits/next-fire/how-to/translations/adding-locales.mdoc

---
status: "published"
title: "Adding a new language | Next.js Firebase SaaS Kit"
label: "Adding a new language in the Next.js Firebase SaaS Kit"
order: 2
description: "How to add a new language to your Next.js Firebase application."
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
FILE PATH: kits/next-fire/how-to/translations/adding-translation.mdoc

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
1. Using the `Trans` component from `next-i18next`
2. Using the `useTranslation` hook from `next-i18next`

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
FILE PATH: kits/next-fire/how-to/translations/default-language.mdoc

---
status: "published"

title: "Setting a Default Language"
label: "Setting a Default Language"
order: 1
description: "Learn how to set a default language for your Makerkit application."
---


Setting a default language in your Next.js Firebase application is easy.

Simply set the environment variable `DEFAULT_LOCALE` to the language code you want to use as the default language.

By default, we use `en` as the default language. If you want to change it, simply add the following line to your `.env` file. For example, if you want to use Spanish as the default language, you would add the following line to your `.env` file:

```bash
DEFAULT_LOCALE=es
```

-----------------
FILE PATH: kits/next-fire/how-to/translations/detect-current-locale.mdoc

---
status: "published"
title: "Detecting the Currently selected locale | Next.js Firebase SaaS Kit"
label: "Detect current Locale"
order: 5
description: "How to detect the currently selected locale so that you can use it in your React components."
---

To detect the currently selected locale from the client or the server, you can use the `useTranslation` hook from `next-i18next`.

```tsx
import { useTranslation } from 'next-i18next';

function Component() {
  const { i18n } = useTranslation();
  const { language } = i18n;

  // use language here
}
```


-----------------
FILE PATH: kits/next-fire/how-to/translations/language-switcher.mdoc

---
status: "published"
title: "Using the Language Switcher | Next.js Firebase SaaS Kit"
label: "Using the Language Switcher"
order: 4
description: "Import and use the LanguageSwitcher component in your Next.js Firebase application."
---


By default, the language switcher is not used within the Next.js Firebase starter. To use it, you will need to import it into your application and add it to your layout.

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
FILE PATH: kits/next-fire/how-to/ui/dark-theme.mdoc

---
status: "published"
title: "Setting up the dark theme in Next.js Firebase"
label: Dark Theme
order: 1
description: "How to set up the dark theme in Next.js Firebase SaaS Kit"
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

In the above, we flipped the `features.enableThemeSwitcher` flag to `false` and
set the `theme` to `Themes.Light`.

## Detecting the current theme

To detect the currently selected theme, you can use the following snippet:

```tsx
import { ThemeContext } from '~/core/contexts/theme';

<ThemeContext.Consumer>
  {(theme) => {
     const isDark = theme === 'dark';
  }
</ThemContext.Consumer>
```

-----------------
FILE PATH: kits/next-fire/how-to/ui/theming.mdoc

---
status: "published"
title: Theming | Next.js Firebase SaaS Kit
label: Theming
order: 1
description: "Discover how to apply the ShadCN UI theme to your Next.js Firebase SaaS Kit project."
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
FILE PATH: kits/next-fire/how-to.mdoc

---
status: "published"
title: "Introduction to the How to section of the Makerkit Next.js Firebase documentation"
label: How To
order: 16
description: "Recipes to help you get started with the platform and handle common tasks."
collapsible: true
collapsed: true
---

The "How to" documentation of Makerkit Next.js Firebase is a collection of tutorials that will help you to build your own web application using Next.js and Firebase using small and practical examples.

## What you will learn

1. Setting up your Makerkit Next.js Firebase project
2. Authentication
3. Updating the marketing pages of your SaaS
4. Interacting with the Database
5. Updating the Marketing pages of your SaaS
6. Updating the Internal pages of your SaaS
7. Setting up Payments
8. Updating Translations
9. ... and a lot more.

This documentation is under development and will be updated regularly. If you have any questions, please contact me.

-----------------
FILE PATH: kits/next-fire/migrations/version-0.11.0.mdoc

---
status: "published"

title: "Migrating to 0.11.0"
label: "Migrating to 0.11.0"
order: 0
---


The 0.11.0 release of the kit introduces a number of breaking changes. This guide will help you migrate your code to the new version.

NB: If you have cloned the repository after the 0.11.0 release, you can skip this guide.

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

Then, copy this script into your project named "migrate-0.11.js":

```js title="migrate-0.11.js
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
node migrate-0.11.js
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

