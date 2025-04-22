-----------------
FILE PATH: kits\next-fire\tutorials\initializing-project.mdoc

---
status: "published"
title: "Setting up the Next.js Firebase SaaS template"
label: Initial Setup
order: 1
description: "This section describes how to set up your development environment to use the Next.js Firebase SaaS template, from forking the repository to installing the Node modules"
---

## Initializing the Project

You have two ways to initialize the project:

1. Forking it from GitHub, or
2. Cloning it using Git

### 1. Forking the Repository

Fork this repository by clicking on the "Fork" button on the top right
corner in GitHub for the repository you want to use.

Once you have forked the repository, clone it locally. Then, set the
upstream repository to the original repository, so you can pull updates
when needed:

```
git remote add upstream git@github.com:makerkit/next-firebase-saas-kit.git
```

### 2. Cloning the Repository

Alternatively, assuming you have accepted the invites and have access to the repository, open your terminal and run this command (replace `tasks-app` with your name of choice):

```
git clone git@github.com:makerkit/next-firebase-saas-kit.git tasks-app
```

Once completed, we'll change into the `tasks-app` directory, and then we will install the Node modules:

```
cd tasks-app
npm i
```

### Reinitialize Git

As the Git repository's remote points to Makerkit's original repository, you can re-initialize (optionally!) it and set the Makerkit repository as `upstream`:

```
git remote rm origin
```

Then, we set the Makerkit repository as `upstream`:

```
git remote add upstream git@github.com:makerkit/next-firebase-saas-kit.git
git add .
git commit -a -m "Initial Commit"
```

By adding the Makerkit's repository as `upstream` remote, you can fetch updates (after committing your files) by running the following command:

```
git pull upstream main --allow-unrelated-histories
```

Perfect! Now you can fire up your IDE and open the `tasks-app` project we just created.

-----------------
FILE PATH: kits\next-fire\tutorials\introduction.mdoc

---
status: "published"
title: "Building a SaaS tasks application with Makerkit | Next.js and Firebase"
label: Introduction
order: 0
description: "Get a head start on your app development with our Makerkit SaaS boilerplate, built with Next.js and Firebase. Follow our step-by-step guide for an easy setup."
---

This tutorial is a comprehensive guide to getting started with the Next.js and Firebase SaaS template to build a basic tasks application, from fetching the repository to deploying the application.

By the end, you'll have a fully working application with the minimum basics a SaaS needs:

1. Landing Page and Marketing pages (blog, documentation)
2. Authentication (sign in, sign up)
3. Payments (Stripe)
4. Profile and Organization management

## Prerequisites

To get started, you're going to need some things installed:

1. Git
2. Node.js version
3. npm 7 or greater
4. A code editor (VSCode, WebStorm)
5. If you'd like to deploy your application, you'll also want an account on Vercel (it's free and easy to use)

Experience with React, TypeScript/JavaScript, and Firebase would be advantageous but not strictly required. The codebase can also serve as a way to learn these topics more in-depth.

If you have all the above installed, I guess we can get started!

-----------------
FILE PATH: kits\next-fire\tutorials\marketing-site-pages.mdoc

---
status: "published"
title: "Adding pages to the Marketing Site | Next.js Firebase SaaS Kit"
label: "Adding pages to the Marketing Site"
order: 16
description: "Learn how to add pages to the Marketing Site of your Next.js Firebase application, and how to add them to the navigation menu."
---

Adding pages to the Marketing Site is easy.

You can add a new page by creating a new file in the `pages` directory. For example, if you want to add a new page called `About`, you can create a new file called `about.tsx` in the `pages` directory.

```
pages/
  about.tsx
```

The file should export a React component that will be rendered when the user visits the page.

For example:

 ```tsx {% title="pages/about.tsx" %}
import { GetStaticPropsContext } from "next";
import { withTranslationProps } from "~/lib/props/with-translation-props";

function About() {
  return <div>About</div>;
}

export function getStaticProps(
  context: GetStaticPropsContext
) {
  return withTranslationProps(context);
}

export default About;
```

NB: We've added the `withTranslationProps` function to the `getStaticProps` function. This is required for all pages to ensure that the page is translated correctly.
If you don't add this function, the components that use translations will not be translated, and the component will not be rendered correctly.

### Adding a page to the navigation menu

To update the site's navigation menu, you can visit the `SiteNavigation.tsx` component and add a new entry to the menu.

By default, you will see the following object `links`:

 ```tsx {% title="components/SiteNavigation.tsx" %}
const links: Record<string, Link> = {
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
};
```

The menu is defined in the render function:

```tsx
<NavigationMenu>
  <NavigationMenuItem link={links.Blog} />
  <NavigationMenuItem link={links.Docs} />
  <NavigationMenuItem link={links.Pricing} />
  <NavigationMenuItem link={links.FAQ} />
</NavigationMenu>
```

Assuming we want to add a new menu entry, say `About`, we would first add the link object:

```tsx
About: {
  label: 'About',
  path: '/about',
},
```

And then we update the menu:

```tsx
<NavigationMenu>
  <NavigationMenuItem link={links.About} />
  ...
</NavigationMenu>
```


-----------------
FILE PATH: kits\next-fire\tutorials\onboarding-flow.mdoc

---
status: "published"
title: "Onboarding Flow | Next.js Firebase SaaS Kit"
label: Onboarding Flow
order: 7
description: "Learn how to onboard your users to your app using the Onboarding Flow in the Next.js Firebase SaaS Kit"
---

After sign-up, users are redirected to the onboarding flow.

Here, you can ask users questions, configure their accounts or simply show them around.

By default, Makerkit adds one single step, where it asks users to **create an organization**.

{% img src="/assets/images/posts/onboarding-flow-tutorial.webp" width="2940" height="1972" /%}

Of course, you can extend it and add as many steps as you wish. I would recommend replacing the image on the right-hand side with a screenshot of your application, a video, or something that can help users understand what they are signing up for.

### What happens when the user submits the form?

After the user submits the form, the API will:

1. create the user Firestore record at `/users`
2. create the organization Firestore record at `/organizations` and assign the user as its owner

I encourage you to visit the [Firestore Emulator UI](http://localhost:4000/firestore/) and see the data created.

### What is an "Organization"?

Organizations are groups of users.

You can call them projects, teams, classrooms, or whatever feels suitable for your domain. But, generally speaking, organizations are the backbone of the data model because it's where we store most of the data shared among users.

Users can:

1. create new Organizations
2. be invited to other Organizations
3. switch between organizations using a dropdown

### Can I remove this step?

No, **but you can skip it** by automatically submitting the form.

Yes, you can follow this guide for [removing the onboarding flow and organizations](/recipes/removing-organizations).

-----------------
FILE PATH: kits\next-fire\tutorials\project-configuration.mdoc

---
status: "published"
title: "Project Configuration | Next.js Firebase"
label: Project Configuration
order: 4
description: "Learn how to set up up your project's configuration"
---

Makerkit can be configured from a single place: the `src/configuration.ts` object. This file exports a constant we use throughout the project to read our application's settings.

{% alert type="info" title="Use this file to configure your project" %}
  We recommend adding additional configuration to this file rather than reading it directly from the environment variables. For example, assuming you rename one of your environment variables, you will only need to update it in one place. Additionally, it helps maintain your codebase more explicit and understandable.
{% /alert %}

We retrieve some of this file's configuration using **environment variables**: these help us swap and tweak our settings based on which environment we're using, such as the Firebase configuration.

For example, in your Vercel console, you could run multiple projects:
- one for `production`
- and one for `staging`

Here is what the configuration file looks like:

 ```tsx {% title="src/configuration.ts" %}
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
    enableMultiFactorAuth: false,
    // NB: Enable the providers below in the Firebase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: [GoogleAuthProvider],
    },
  },
  emulatorHost: process.env.NEXT_PUBLIC_EMULATOR_HOST,
  emulator: process.env.NEXT_PUBLIC_EMULATOR === 'true',
  production: process.env.NODE_ENV === 'production',
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
    searchIndex: `/public/search-index`,
  },
  navigation: {
    style: LayoutStyle.Sidebar,
  },
  appCheckSiteKey: process.env.NEXT_PUBLIC_APPCHECK_KEY,
  email: {
    host: '',
    port: 0,
    user: '',
    password: '',
    senderAddress: '',
  },
  sentry: {
    dsn: process.env.SENTRY_DSN,
  },
  stripe: {
    plans: [
      {
        name: 'Basic',
        description: 'Description of your Basic plan',
        price: '$249/year',
        stripePriceId: 'basic-plan',
        features: [
          'Feature 1',
          'Feature 2',
          'common:plans.features.feature1'
        ],
      },
    ],
  }
};
```

Feel free to extend this configuration to your needs. For example, you could new properties to the `site` object to store your social media handles, or add new properties to the `paths` object to store the paths to your pages.


-----------------
FILE PATH: kits\next-fire\tutorials\project-structure.mdoc

---
status: "published"

title: "Project Structure"
label: Project Structure
order: 2
description: "A quick overview of the project structure and how to navigate it."
---

When inspecting the project structure, you will find something similar to the below:

```
tasks-app
├── README.md
├── @types
├── src
│   ├── components
│   ├── core
│   ├── lib
│   └── pages
        └── _document.tsx
        └── _app.tsx
│       └── page.tsx
├── package-lock.json
├── package.json
├── public
│   └── favicon.ico
├── next.config.mjs
└── tsconfig.json
```

NB: we omitted a lot of the files for simplicity.

Let's take a deeper look at the structure we see above:

- `src/core`: this folder contains reusable building blocks of the template that **are not related to any domain**. This includes components, React hooks, functions, and so on.
- `src/components`: this folder contains the application's components, including those that belong to your application's domain. For example, we can add the `tasks` components to `src/components/tasks`.
- `src/lib`: this folder contains the business logic of your application's domain. For example, we can add the `tasks` hooks to write and fetch data from Firestore in `src/lib/tasks`.
- `src/pages`: this is Next.js's special folder where we add our application routes.

Don't worry if this isn't clear yet! We'll explain where we place our files in the sections ahead.

-----------------
FILE PATH: kits\next-fire\tutorials\running-project.mdoc

---
status: "published"
title: "Running the App | Next.js Firebase SaaS Kit"
label: Running the App
order: 3
description: "Learn how to run the app locally for development and testing."
---

When we run the stack of a Makerkit application, we will need to run a few commands before:

1. **Next.js**: First, we need to run the Next.js development server
2. **Firebase**: We need to run the Firebase Emulators. The emulators allow us to run a fully-local Firebase environment.
3. **Stripe CLI** (optional): If you plan on interacting with Stripe, you are also required to run the Stripe CLI, which allows us to test the webhooks from Stripe against our local server.

### Running the Next.js development server

Run the following command from your IDE or from your terminal:

```bash
npm run dev
```

This command will start the Next.js server at [localhost:3000](http://localhost:3000). If you navigate to this page, you should be able to see the landing page of your application:

{% img src="/assets/images/posts/tutorial-landing-page.webp" width="2856" height="1972" /%}

### Running the Firebase Emulators

To interact with Firebase's services, such as Auth, Firestore, and Storage, we need to run the Firebase emulators. To do so, open a new terminal (or, better, from your IDE) and run the following command:

```
npm run firebase:emulators:start
```

If everything is working correctly, you will see the output below:

```
┌────────────────┬────────────────┬─────────────────────────────────┐
│ Emulator       │ Host:Port      │ View in Emulator UI             │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Authentication │ localhost:9099 │ http://localhost:4000/auth      │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Firestore      │ localhost:8080 │ http://localhost:4000/firestore │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Pub/Sub        │ localhost:8085 │ n/a                             │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Storage        │ localhost:9199 │ http://localhost:4000/storage   │
└────────────────┴────────────────┴─────────────────────────────────┘
  Emulator Hub running at localhost:4400
  Other reserved ports: 4500, 9150
```

By default, we run the emulator for the following services: Authentication, Firestore, and Storage. However, you can easily add Firebase Functions and PubSub.

#### Firebase Emulator UI

As you can see from the `View in Emulator UI` column, you have access to the Emulator UI, which helps you see and edit your Firebase Emulator data using a nifty UI that resembles the Firebase Console.

For example, to display the list of users that have signed up, navigate to the [Authentication Emulator](http://localhost:4000/auth).

{% alert type="info" title="Makerkit's template adds some data to your project by default for testing reasons" %}
Makerkit's template adds some data to your project by default for testing reasons. In fact, the data you see is used to run the Cypress E2E tests. This is why you'll see some pre-populated data in your Firestore and Authentication tabs.
{% /alert %}

### Running the Stripe CLI (optional)

Optionally, if you want to run Stripe locally (e.g., sending webhooks to your local server), you will also need to run the following command:

```
npm run stripe:listen
```

This command requires Docker, but you can alternatively install Stripe on your OS and change the command to use `stripe` directly.

The above command runs the Stripe CLI and will route webhooks coming from Stripe to your local endpoint. For example, in the Makerkit starter, this endpoint is `/api/stripe/webhook`.

When running the command, it will print a webhook key used to sign the messages from Stripe. You will need to add this key to your local environment variables file as below:

```
STRIPE_WEBHOOK_SECRET=<KEY>
```

The webhook printed should not change, so you may only need to do this the first time.

-----------------
FILE PATH: kits\next-fire\tutorials\translations.mdoc

---
status: "published"

title: "Translations"
label: "Translations"
order: 14
description: "Learn how to easily translate your Next.js Firebase Makerkit SaaS into multiple languages with our guide. Optimize your app and reach a wider audience."
---

Most strings in the Makerkit template's application use `next-18n`, a library to translate your application into multiple languages. Thanks to this library, we can store all the application's text in `json` files for each supported language.

For example, if you are serving your application in English and Spanish, you'd have the following files:

- **English** translation files at `/public/locales/en/{file}.json`
- **Spanish** translation files at `/public/locales/es/{file}.json`

There are valid reasons to use translation files, even if you are not planning on translating your application, such as:

1. it's easier to search and replace text
2. tidier render functions
3. easy update path if you do decide to support a new language

### Adding new languages

By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:
1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **Next.js i18n config**: We need to also add a new language to the Next.js configuration at `next-i18next.config.js`. Simply add your new language's code to the `locales` array.

The configuration will look like the below:

```js
const config = {
  i18n: {
    defaultLocale: DEFAULT_LOCALE,
    locales: [DEFAULT_LOCALE, 'es'],
  },
  fallbackLng: {
    default: [DEFAULT_LOCALE],
  },
  localePath: resolve('./public/locales'),
};
```

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```

### Using the Language Switcher component

The Language switcher component will automatically retrieve and translate the languages listed in your configuration.

When the user changes language, it will update the URL to reflect the user's preferences and rerender the application with the selected language.

{% video src="/assets/images/posts/language-selector.mp4" /%}

NB: The language selector is not added to the application by default; you will need to import it where you want to place it.


-----------------
FILE PATH: kits\next-fire\tutorials\updating-latest-version.mdoc

---
status: "published"

title: "Updating to the latest version"
label: "Updating to the latest version"
order: 20
description: "Learn how to update your Makerkit repository to the latest version."
---

If you have followed the steps to set up Git at the beginning of this tutorial, you've already set the original Makerkit repository as `upstream`.

As the repository is constantly updated, it's recommended to fetch updates regularly. You can do so by running the following Git command:

```
git pull upstream main --allow-unrelated-histories
```

Unfortunately, you may need to resolve eventual conflicts.

### Any questions?

If you have questions, please join our [Discord Community](https://discord.gg/BeJSyQJ6jP), even if you have not purchased any kit. We chat and help each other build products.

-----------------
FILE PATH: kits\next-fire\tutorials.mdoc

---
status: "published"
title: "Tutorials"
label: Tutorials
order: 15
description: "Tutorials"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\next-fire\ui\shadcn.mdoc

---
status: "published"
title: Shadcn UI | Next.js Firebase
label: Shadcn UI
order: 1
description: 'Learn how to add new components from Shadcn UI to your project.'
---

ShadCN UI is a template library based on Radix UI and Tailwind CSS. It provides a set of high-quality React components out of the box that are fully accessible, customizable, and themable.

Makerkit was an early adopter of ShadCN UI, and we use it to build our own UI components. With that, we used very different styling conventions early on, which means copy/pasting components from our docs to your project might not work as expected.

ShadCN UI is **configured** in all Makerkit projects as of the latest versions, while versions released before 28 September 2023 will not work seamlessly.

Makerkit only ships the components it uses in the core kit: to use other components, you will need to either use the CLI or import them manually.

## Theming

### Updating the colors in CSS

To update the theme's colors, please update the `src/styles/index.css` file in your project and update the CSS variables.

For examples, check out the [ShadCN UI Themes Page](https://ui.shadcn.com/themes).

### Updating the colors in Tailwind

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
FILE PATH: kits\next-fire\ui\tailwindcss.mdoc

---
status: "published"
title: "Tailwind CSS | Next.js Firebase SaaS Kit"
label: Tailwind CSS
order: 0
description: 'Learn how to customize Tailwind CSS in your Makerkit project.'
---

All the Makerkit kits use Tailwind CSS for styling. Tailwind is a utility-first CSS framework that allows you to build custom designs without leaving your HTML. It's a great choice for Makerkit because it allows you to customize the look and feel of your project without having to write any CSS.

The Tailwind CSS configuration file is located at `tailwind.config.js` in the root of your project.

By default, it looks like this:

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
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        dark: {
          ...colors.slate,
          DEFAULT: colors.slate[950],
          foreground: colors.slate[100],
        },
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
          ...colors.violet,
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: { height: 0 },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: 0 },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

The Tailwind configuration gets extended with two additional colors:
`primary` and `dark`.

1. `primary` is the main color of your project. It's used for buttons, links, and other interactive elements. It's also used for some backgrounds of the dark mode.
2. `dark` is the color palette for the dark mode. It's used for the background of the dark mode.

By updating these colors, you can customize the look and feel of your project.

For example:
1. If you want to change the primary color, you can update the `primary` color in the Tailwind configuration. Try changing it to `blue` and see what happens.
2. If you want to change the dark mode background color, you can update the `dark` color in the Tailwind configuration. Try changing it to `slate` and see what happens.

Of course - you can also pass your own palette as the `primary` and `dark` colors.

## Usage in practice

Let's say you want to change the primary color of your project to `indigo`. and the dark mode background color to `zinc`.

 ```js {% title="tailwind.config.js" %}
const colors = require('tailwindcss/colors');

extend: {
  colors: {
    dark: {
      ...colors.zinc,
      DEFAULT: colors.zinc[950],
      foreground: colors.zinc[100],
    },
    primary: {
      DEFAULT: 'hsl(var(--primary))',
      foreground: 'hsl(var(--primary-foreground))',
      ...colors.indigo,
    }
  },
},
```

Additionally, please update the CSS variables of your global CSS file `globals.css` to match the new colors by updating the `--primary` color to match using the `hsl` value of the `indigo` color.

To have a reference of the Tailwind palette as `hsl` values, you can use [this reference from the ShadcnUI repository](https://github.com/shadcn-ui/ui/issues/669).

When you want to use the primary color in your project, you can use the `primary` color class.

 ```tsx {% title="src/components/PrimaryButton.tsx" %}
import React from 'react';

export const PrimaryButton = () => {
  return (
    <button className="bg-primary text-white dark:bg-primary/10 dark:text-primary px-4 py-2 rounded">
      Click me
    </button>
  );
};
```

-----------------
FILE PATH: kits\next-fire\ui\themes.mdoc

---
status: "published"
title: Themes | Next.js Firebase SaaS Kit
label: Themes
order: 2
description: 'Learn how to configure the default theme of your Next.js Firebase application.'
---

By default, the Makerkit kits come with a light and a dark theme, and we provide a toggle to let users switch between them.

## Changing the default theme

To change the default theme, you need to update the theme in your `src/configuration.ts` file.

 ```ts {% title="src/configuration.ts" %} {2}
const configuration = {
  theme: Themes.Dark,
  ...
}
```

## Disabling the theme toggle

To disable the theme toggle, you need to update the `src/configuration.ts` file.

 ```tsx {% title="src/configuration.ts" %} {3}
const configuration = {
  theme: Themes.Dark,
  features: {
    enableThemeSwitcher: false,
  },
  ...
}
```

This will prevent the theme toggle from appearing in the application.

