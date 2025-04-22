-----------------
FILE PATH: courses\nextjs-turbo\customization.mdoc

---
title: "How to customize your Makerkit Next.js Supabase Turbo project"
description: "Learn how to customize your Makerkit Next.js Supabase Turbo project and make it your own."
label: "Customization and Configuration"
order: 1
---

Hi there! 👋 Welcome to this module on customizing your Makerkit Next.js Supabase Turbo project. In this module, **we will explore how you can customize your project and make it your own**.

Makerkit is designed to be fully customizable. To achieve this, we use environment variables: in this way, you can make customizations to your project without having to modify the source code - where possible.

Once you have your project set up, you will want to update a few things, such as:

1. **Details**: Update your project details, such as the name, description, and logo
2. **Branding**: Customize the colors, fonts, and other branding elements
3. **Settings**: You will also want to customize the settings of your project, such as the default language, or logic configuration
4. **Features**: You can also enable or disable features, such as authentication methods, notifications, etc.

In this module, we will explore how you can customize your project using environment variables and other methods.

{% img src="/assets/courses/next-turbo/configuration-architecture.webp" width="788" height="377" /%}

## Makerkit's Configuration Architecture

1. **Environment variables**: Makerkit can access the vast majority of the configuration options through environment variables.
2. **Configuration**: The environment variables are passed to the configuration layer (`apps/web/config/*`) that uses [Zod](https://zod.dev) to validate the configuration. This ensures that the configuration is correct and that the application will not start if the configuration is invalid.
3. **Application Layer**: Finally, the configuration is passed to the application layer (`apps/web/src/*`) and the packages where it is used to customize the application.

{% img src="/assets/courses/next-turbo/env-variables-architecture.webp" width="333" height="362" /%}

## Environment variables

Environment variables are defined in the `.env` file in the root of the `apps/web` package.

1. **Shared Environment Variables**: Shared environment variables are defined in the `.env` file. These are the env variables shared between environments (e.g., development, staging, production).
2. **Environment Variables**: Environment variables for a specific environment are defined in the `.env.development` and `.env.production` files. These are the env variables specific to the development and production environments.
3. **Secret Keys**: Secret keys and sensitive information are not stored in the `.env` file. Instead, they are stored in the environment variables of the CI/CD system.
4. **Secret keys to be used locally**: If you need to use secret keys locally, you can store them in the `.env.local` file. This file is not committed to Git, therefore it is safe to store sensitive information in it. Don't change the ignore rule in the `.gitignore` file.

I'll reiterate for those in the back: **Do not store secret keys in the `.env` file**. Instead, store them in the environment variables of the CI/CD system.

#### Public and Private Environment Variables in Next.js

If you are new to Next.js - you need to know about public and private environment variables:

1. **Public environment variables** are prefixed with `NEXT_PUBLIC_` and can be accessed on the client-side. These are exposed to the browser, therefore should not be used for sensitive information.
2. **Private environment variables** can only be accessed on the server-side. These are not prefixed with `NEXT_PUBLIC_`.

```bash {% title="apps/web/.env" %}
# Public environment variables
NEXT_PUBLIC_PRODUCT_NAME="Makerdesk"

# Private environment variables
STRIPE_SECRET_KEY=********
```

**Always carefully consider when a variable should be made public** - and when it should be kept private. If you're not sure, it's best to keep it private.

### How do I decide where to store my environment variables?

Here's a general rule of thumb for storing environment variables:

1. **Shared and Public**: If the environment variable is shared between environments and is not sensitive, store it in the `.env` file. Things like the product name, colors, language, etc.Configuration
2. **Environment-specific and Public**: If the environment variable is specific to an environment (e.g., development, production), store it in the `.env.development` or `.env.production` file. Things like the API URL, analytics keys, etc.
3.  **Secret and Sensitive**: If the environment variable is secret and sensitive, store it in the environment variables of the CI/CD system. Things like the database URL, secret keys, etc. Locally, store them in the `.env.local` file.

**NB**: This is not Makerkit-specific. This is a general best practice for managing environment variables in a project, and it's a good practice to follow.

## Configuration Files

The configuration files are located in the `apps/web/config` directory. The configuration files are written in TypeScript and use the Zod library to validate the configuration.

Makerkit defines the following configuration files:

1. `app.config.ts`: This file stores the overall variables for your application. This includes the product name, site title, description, URL, locale, theme, etc.
2. `auth.config.ts`: This file stores the authentication configuration for your application. This includes the captcha token site key, providers (password, magic link, oAuth), etc.
3. `feature-flags.config.ts`: This file stores the feature flags configuration for your application. This includes the feature flags (enableThemeToggle, enableAccountDeletion, enableTeamDeletion, etc.), language priority, etc.
4. `billing.config.ts`: This file stores the billing schema for your application.
5. `paths.config.ts`: This file stores the paths configuration for your application
6. `personal-account-navigation.config.ts`: This file stores the global navigation configuration for the personal account's layout
7. `team-account-navigation.config.ts`: This file stores the global navigation configuration for the team account's layout

### Using Zod to validate the configuration

As mentioned, we use Zod to validate the configuration.

Zod is a TypeScript-first schema declaration and validation library. It's used to validate the configuration and ensure that it's correct. To avoid shipping invalid configuration, Makerkit goes out of its way to make sure you're setting a valid configuration.

Makerkit tries its best to be __an absolute pain in the neck__ when it comes to invalid configurations. It will throw an error if the configuration is invalid, and the application will not start.

I know you will curse my name while seeing red builds all over your CI - but this is for your own good. Trust me - an invalid configuration can lead to a lot of headaches down the road - and I don't want that for you.

{% img src="/assets/courses/next-turbo/env-variables-architecture-zod.webp" width="674" height="362" /%}

Let's take a quick look at how `app.config.ts` is structured. First, we define the schema using Zod:

```tsx
const AppConfigSchema = z
  .object({
    name: z
      .string({
        description: `This is the name of your SaaS. Ex. "Makerkit"`,
        required_error: `Please provide the variable NEXT_PUBLIC_PRODUCT_NAME`,
      })
      .min(1),
    title: z
      .string({
        description: `This is the default title tag of your SaaS.`,
        required_error: `Please provide the variable NEXT_PUBLIC_SITE_TITLE`,
      })
      .min(1),
    description: z.string({
      description: `This is the default description of your SaaS.`,
      required_error: `Please provide the variable NEXT_PUBLIC_SITE_DESCRIPTION`,
    }),
    url: z
      .string({
        required_error: `Please provide the variable NEXT_PUBLIC_SITE_URL`,
      })
      .url({
        message: `You are deploying a production build but have entered a NEXT_PUBLIC_SITE_URL variable using http instead of https. It is very likely that you have set the incorrect URL. The build will now fail to prevent you from from deploying a faulty configuration. Please provide the variable NEXT_PUBLIC_SITE_URL with a valid URL, such as: 'https://example.com'`,
      }),
    locale: z
      .string({
        description: `This is the default locale of your SaaS.`,
        required_error: `Please provide the variable NEXT_PUBLIC_DEFAULT_LOCALE`,
      })
      .default('en'),
    theme: z.enum(['light', 'dark', 'system']),
    production: z.boolean(),
    themeColor: z.string(),
    themeColorDark: z.string(),
  })
  .refine(
    (schema) => {
      const isCI = process.env.NEXT_PUBLIC_CI;

      if (isCI ?? !schema.production) {
        return true;
      }

      return !schema.url.startsWith('http:');
    },
    {
      message: `Please use a valid HTTPS URL in production.`,
      path: ['url'],
    },
  )
  .refine(
    (schema) => {
      return schema.themeColor !== schema.themeColorDark;
    },
    {
      message: `Please provide different theme colors for light and dark themes.`,
      path: ['themeColor'],
    },
  );
```

Then, we parse the configuration using the schema:

```tsx
const appConfig = AppConfigSchema.parse({
  name: process.env.NEXT_PUBLIC_PRODUCT_NAME,
  title: process.env.NEXT_PUBLIC_SITE_TITLE,
  description: process.env.NEXT_PUBLIC_SITE_DESCRIPTION,
  url: process.env.NEXT_PUBLIC_SITE_URL,
  locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
  theme: process.env.NEXT_PUBLIC_DEFAULT_THEME_MODE,
  themeColor: process.env.NEXT_PUBLIC_THEME_COLOR,
  themeColorDark: process.env.NEXT_PUBLIC_THEME_COLOR_DARK,
  production,
});
```

As you can see here, we use Next.js environment variables to set the configuration. Let's assume you want to set the product name and app URL. You'd update the variables in the `.env` file:

```bash {% title="apps/web/.env" %}
NEXT_PUBLIC_PRODUCT_NAME="Makerdesk"
NEXT_PUBLIC_SITE_URL=https://myapp.com
```

Now, the configuration is validated using the Zod schema - and your application can safely use `appConfig` without worrying about invalid configuration.

```tsx
import appConfig from '~/config/app.config';

const appUrl = appConfig.url;
const isProduction = appConfig.production;
```

Perfect! Now you know how to customize your Makerkit Next.js Supabase Turbo project using environment variables and configuration files.

To know the specific environment variables, [please head to the configuration's documentation](/docs/next-supabase-turbo/environment-variables).

## Customizing your project details

Okay - we have a new application, and we want to customize it. Let's start by updating the project details.

You will want to update the following details:

1. **Product Name**: This is the name of your application. It's displayed in the header, footer, and other places.
2. **Site Title**: This is the title of your application. It's displayed in the title tag of the browser.
3. **Site Description**: This is the description of your application. It's displayed in the meta description tag of the browser.
4. **Site URL**: This is the URL of your application. It's used for SEO and other purposes.
5. **Default Locale**: This is the default locale of your application. It's used for localization purposes.
6. **Theme**: This is the default theme of your application. It's used to set the theme of your application.
7. **Theme Color**: This is the color of your application. It's used to set the color of your application.

This is all done using environment variables. You can update the environment variables in the `.env` file:

```bash {% title="apps/web/.env" %}
NEXT_PUBLIC_PRODUCT_NAME="Makerdesk"
NEXT_PUBLIC_SITE_TITLE="Makerdesk - Customer Support for busy creators"
NEXT_PUBLIC_SITE_DESCRIPTION="Makerdesk is a customer support platform for creators. Receive and respond to tickets in real-time."
NEXT_PUBLIC_SITE_URL=https://myapp.com
```

Our application will now use these environment variables to set the project details throughout the application.

## Customizing your project branding

Next, let's customize the branding of your application. You will want to update the colors, fonts, and other branding elements.

### Updating the Theme

**Makerkit uses Shadcn UI as UI library for the components**. Shadcn UI itself uses Radix UI, a Headless UI library, to build the components. Here we are simply repeating the same instructions as for the [Shadcn UI Themes documentation](https://ui.shadcn.com/themes) for completeness.

By default, Makerkit uses the default colors of Shadcn UI using the New York style, albeit with some minor changes. However, you can customize the colors using their theme system. Please [visit the Shadcn UI Themes documentation](https://ui.shadcn.com/themes) to learn more about how to customize the theme.

To update the colors, please pick your favorite theme from the Shadcn UI themes, copy the CSS variables, and update the global CSS style at `apps/web/styles/shadcn-ui.css`.

I personally decided to apply the `Violet` theme to this application. **I therefore copied the CSS variables from the theme and updated the global CSS style** (only for light mode, as I prefer Makerkit's default dark colors):

```css {% title="apps/web/styles/shadcn-ui.css" %}
@layer base {
  :root {
    --background: hsl(0 0% 100%);
    --foreground: hsl(224 71.4% 4.1%);
    --card: hsl(0 0% 100%);
    --card-foreground: hsl(224 71.4% 4.1%);
    --popover: hsl(0 0% 100%);
    --popover-foreground: hsl(224 71.4% 4.1%);
    --primary: hsl(262.1 83.3% 57.8%);
    --primary-foreground: hsl(210 20% 98%);
    --secondary: hsl(220 14.3% 95.9%);
    --secondary-foreground: hsl(220.9 39.3% 11%);
    --muted: hsl(220 14.3% 95.9%);
    --muted-foreground: hsl(220 8.9% 46.1%);
    --accent: hsl(220 14.3% 95.9%);
    --accent-foreground: hsl(220.9 39.3% 11%);
    --destructive: hsl(0 84.2% 60.2%);
    --destructive-foreground: hsl(210 20% 98%);
    --border: hsl(220 13% 91%);
    --input: hsl(220 13% 91%);
    --ring: hsl(262.1 83.3% 57.8%);
    --radius: 0.75rem;

    --chart-1: hsl(12 76% 61%);
    --chart-2: hsl(173 58% 39%);
    --chart-3: hsl(197 37% 24%);
    --chart-4: hsl(43 74% 66%);
    --chart-5: hsl(27 87% 67%);

    --sidebar-background: hsl(0 0% 98%);
    --sidebar-foreground: hsl(240 5.3% 26.1%);
    --sidebar-primary: hsl(240 5.9% 10%);
    --sidebar-primary-foreground: hsl(0 0% 98%);
    --sidebar-accent: hsl(240 4.8% 95.9%);
    --sidebar-accent-foreground: hsl(240 5.9% 10%);
    --sidebar-border: hsl(220 13% 91%);
    --sidebar-ring: hsl(217.2 91.2% 59.8%);
  }

  .dark {
    --background: hsl(224 71% 4%);
    --foreground: hsl(213 31% 91%);
    --muted: hsl(223 47% 11%);
    --muted-foreground: hsl(215.4 16.3% 56.9%);
    --accent: hsl(216 34% 10%);
    --accent-foreground: hsl(210 40% 98%);
    --popover: hsl(224 71% 4%);
    --popover-foreground: hsl(215 20.2% 65.1%);
    --border: hsl(216 34% 17%);
    --input: hsl(216 34% 17%);
    --card: hsl(224 71% 4%);
    --card-foreground: hsl(213 31% 91%);
    --secondary: hsl(216 34% 10%);
    --secondary-foreground: hsl(210 40% 98%);
    --destructive: hsl(0 63% 31%);
    --destructive-foreground: hsl(210 40% 98%);
    --ring: hsl(216 34% 17%);

    --chart-1: hsl(220 70% 50%);
    --chart-2: hsl(160 60% 45%);
    --chart-3: hsl(30 80% 55%);
    --chart-4: hsl(280 65% 60%);
    --chart-5: hsl(340 75% 55%);

    --sidebar-background: hsl(224 71.4% 4.1%);
    --sidebar-foreground: hsl(240 4.8% 95.9%);
    --sidebar-primary: hsl(224.3 76.3% 48%);
    --sidebar-primary-foreground: hsl(0 0% 100%);
    --sidebar-accent: hsl(215 27.9% 13%);
    --sidebar-accent-foreground: hsl(240 4.8% 95.9%);
    --sidebar-border: hsl(240 3.7% 15.9%);
    --sidebar-ring: hsl(217.2 91.2% 59.8%);
  }
}
```

Please feel free to choose any color, or leave the default colors as they are.

### Updating the Logo

The logo is displayed in various parts of the application - and it's stored in a React component.

To update the logo, you can replace the logo in the `apps/web/components/app-logo.tsx` file:

1. **SVGs**:You can provide your own SVG
2. **Images**: You can also use an image and set the `src` attribute to the image URL
3. **Text**: You can also use a text logo and set the `children` attribute to the text

As long as the logo is placed in the `apps/web/components/app-logo.tsx` file, it will be used throughout the application - and can really be anything you want.

For my logo, I designed the following SVG in Figma:

```tsx {% title="apps/web/components/app-logo.tsx" %}
import Link from 'next/link';

import { cn } from '@kit/ui/utils';

function LogoImage({
  className,
  width = 105,
}: {
  className?: string;
  width?: number;
}) {
  return (
    <svg
      width={width}
      className={cn(`w-[95px]`, className)}
      viewBox="0 0 952 140"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        className="dark:hidden"
        d="M119.421 138V73.209C119.421 67.551 117.42 62.79 113.418 58.926C109.554 55.062 104.793 53.13 99.1349 53.13C93.6149 53.13 88.7849 55.062 84.6449 58.926C80.6429 62.652 78.6419 67.344 78.6419 73.002V138H59.8049V73.002C59.8049 67.206 57.8729 62.514 54.0089 58.926C49.8689 55.062 44.9699 53.13 39.3119 53.13C33.7919 53.13 29.0999 55.062 25.2359 58.926C21.0959 63.066 19.0259 67.965 19.0259 73.623V138H0.395899V36.984H19.0259V44.643C21.3719 41.745 24.4079 39.33 28.1339 37.398C31.9979 35.466 35.7239 34.5 39.3119 34.5C45.3839 34.5 50.9039 35.742 55.8719 38.226C60.9779 40.572 65.3939 43.884 69.1199 48.162C72.8459 43.884 77.2619 40.572 82.3679 38.226C87.4739 35.742 93.0629 34.5 99.1349 34.5C104.517 34.5 109.554 35.466 114.246 37.398C118.938 39.33 123.078 42.09 126.666 45.678C130.254 49.266 133.014 53.475 134.946 58.305C137.016 62.997 138.051 67.965 138.051 73.209V138H119.421ZM242.513 138V122.268C238.097 127.374 232.991 131.445 227.195 134.481C221.399 137.517 215.258 139.035 208.772 139.035C201.596 139.035 194.834 137.724 188.486 135.102C182.276 132.48 176.756 128.754 171.926 123.924C167.096 119.232 163.301 113.712 160.541 107.364C157.919 100.878 156.608 94.116 156.608 87.078C156.608 80.04 157.919 73.347 160.541 66.999C163.301 60.513 167.096 54.855 171.926 50.025C176.756 45.195 182.276 41.469 188.486 38.847C194.834 36.225 201.596 34.914 208.772 34.914C215.396 34.914 221.606 36.294 227.402 39.054C233.336 41.814 238.373 45.678 242.513 50.646V36.984H261.143V138H242.513ZM208.772 53.337C204.218 53.337 199.802 54.234 195.524 56.028C191.384 57.684 187.796 60.03 184.76 63.066C181.724 66.102 179.309 69.759 177.515 74.037C175.859 78.177 175.031 82.524 175.031 87.078C175.031 91.632 175.859 95.979 177.515 100.119C179.309 104.259 181.724 107.847 184.76 110.883C187.796 113.919 191.384 116.334 195.524 118.128C199.802 119.784 204.218 120.612 208.772 120.612C213.326 120.612 217.673 119.784 221.813 118.128C225.953 116.334 229.541 113.919 232.577 110.883C235.613 107.847 237.959 104.259 239.615 100.119C241.409 95.979 242.306 91.632 242.306 87.078C242.306 82.524 241.409 78.177 239.615 74.037C237.959 69.759 235.613 66.102 232.577 63.066C229.541 60.03 225.953 57.684 221.813 56.028C217.673 54.234 213.326 53.337 208.772 53.337ZM331.467 138L300.003 99.705V138H281.787V0.344996H300.003V59.754L328.155 33.258H355.272L306.213 78.798L355.479 138H331.467ZM379.639 94.116C379.639 97.428 380.812 100.878 383.158 104.466C385.642 108.054 388.471 111.09 391.645 113.574C397.441 118.128 404.203 120.405 411.931 120.405C424.213 120.405 434.218 114.471 441.946 102.603L457.678 111.918C452.296 120.612 445.672 127.305 437.806 131.997C429.94 136.689 421.315 139.035 411.931 139.035C404.893 139.035 398.2 137.724 391.852 135.102C385.504 132.342 379.915 128.547 375.085 123.717C370.255 118.887 366.46 113.298 363.7 106.95C361.078 100.602 359.767 93.909 359.767 86.871C359.767 79.833 361.078 73.14 363.7 66.792C366.46 60.306 370.255 54.648 375.085 49.818C379.777 44.988 385.297 41.262 391.645 38.64C398.131 36.018 404.893 34.707 411.931 34.707C418.969 34.707 425.662 36.018 432.01 38.64C438.496 41.262 444.085 44.988 448.777 49.818C458.989 60.306 464.095 72.45 464.095 86.25C464.095 88.734 463.888 91.356 463.474 94.116H379.639ZM411.931 51.681C406.273 51.681 400.96 52.923 395.992 55.407C391.024 57.891 387.022 61.203 383.986 65.343C381.088 69.345 379.639 73.623 379.639 78.177H444.223C444.223 73.623 442.705 69.345 439.669 65.343C436.771 61.203 432.838 57.891 427.87 55.407C422.902 52.923 417.589 51.681 411.931 51.681ZM528.883 54.372C525.571 52.854 522.604 52.095 519.982 52.095C514.462 52.095 509.908 54.027 506.32 57.891C502.456 62.031 500.524 66.792 500.524 72.174V138H482.722V72.174C482.722 64.722 484.585 57.891 488.311 51.681C492.175 45.471 497.419 40.641 504.043 37.191C509.011 34.845 514.324 33.672 519.982 33.672C524.536 33.672 528.883 34.5 533.023 36.156C537.163 37.812 541.51 40.503 546.064 44.229L528.883 54.372Z"
        fill="url(#paint0_linear_2621_2)"
      />
      <path
        className="hidden dark:block"
        d="M119.421 138V73.209C119.421 67.551 117.42 62.79 113.418 58.926C109.554 55.062 104.793 53.13 99.1349 53.13C93.6149 53.13 88.7849 55.062 84.6449 58.926C80.6429 62.652 78.6419 67.344 78.6419 73.002V138H59.8049V73.002C59.8049 67.206 57.8729 62.514 54.0089 58.926C49.8689 55.062 44.9699 53.13 39.3119 53.13C33.7919 53.13 29.0999 55.062 25.2359 58.926C21.0959 63.066 19.0259 67.965 19.0259 73.623V138H0.395899V36.984H19.0259V44.643C21.3719 41.745 24.4079 39.33 28.1339 37.398C31.9979 35.466 35.7239 34.5 39.3119 34.5C45.3839 34.5 50.9039 35.742 55.8719 38.226C60.9779 40.572 65.3939 43.884 69.1199 48.162C72.8459 43.884 77.2619 40.572 82.3679 38.226C87.4739 35.742 93.0629 34.5 99.1349 34.5C104.517 34.5 109.554 35.466 114.246 37.398C118.938 39.33 123.078 42.09 126.666 45.678C130.254 49.266 133.014 53.475 134.946 58.305C137.016 62.997 138.051 67.965 138.051 73.209V138H119.421ZM242.513 138V122.268C238.097 127.374 232.991 131.445 227.195 134.481C221.399 137.517 215.258 139.035 208.772 139.035C201.596 139.035 194.834 137.724 188.486 135.102C182.276 132.48 176.756 128.754 171.926 123.924C167.096 119.232 163.301 113.712 160.541 107.364C157.919 100.878 156.608 94.116 156.608 87.078C156.608 80.04 157.919 73.347 160.541 66.999C163.301 60.513 167.096 54.855 171.926 50.025C176.756 45.195 182.276 41.469 188.486 38.847C194.834 36.225 201.596 34.914 208.772 34.914C215.396 34.914 221.606 36.294 227.402 39.054C233.336 41.814 238.373 45.678 242.513 50.646V36.984H261.143V138H242.513ZM208.772 53.337C204.218 53.337 199.802 54.234 195.524 56.028C191.384 57.684 187.796 60.03 184.76 63.066C181.724 66.102 179.309 69.759 177.515 74.037C175.859 78.177 175.031 82.524 175.031 87.078C175.031 91.632 175.859 95.979 177.515 100.119C179.309 104.259 181.724 107.847 184.76 110.883C187.796 113.919 191.384 116.334 195.524 118.128C199.802 119.784 204.218 120.612 208.772 120.612C213.326 120.612 217.673 119.784 221.813 118.128C225.953 116.334 229.541 113.919 232.577 110.883C235.613 107.847 237.959 104.259 239.615 100.119C241.409 95.979 242.306 91.632 242.306 87.078C242.306 82.524 241.409 78.177 239.615 74.037C237.959 69.759 235.613 66.102 232.577 63.066C229.541 60.03 225.953 57.684 221.813 56.028C217.673 54.234 213.326 53.337 208.772 53.337ZM331.467 138L300.003 99.705V138H281.787V0.344996H300.003V59.754L328.155 33.258H355.272L306.213 78.798L355.479 138H331.467ZM379.639 94.116C379.639 97.428 380.812 100.878 383.158 104.466C385.642 108.054 388.471 111.09 391.645 113.574C397.441 118.128 404.203 120.405 411.931 120.405C424.213 120.405 434.218 114.471 441.946 102.603L457.678 111.918C452.296 120.612 445.672 127.305 437.806 131.997C429.94 136.689 421.315 139.035 411.931 139.035C404.893 139.035 398.2 137.724 391.852 135.102C385.504 132.342 379.915 128.547 375.085 123.717C370.255 118.887 366.46 113.298 363.7 106.95C361.078 100.602 359.767 93.909 359.767 86.871C359.767 79.833 361.078 73.14 363.7 66.792C366.46 60.306 370.255 54.648 375.085 49.818C379.777 44.988 385.297 41.262 391.645 38.64C398.131 36.018 404.893 34.707 411.931 34.707C418.969 34.707 425.662 36.018 432.01 38.64C438.496 41.262 444.085 44.988 448.777 49.818C458.989 60.306 464.095 72.45 464.095 86.25C464.095 88.734 463.888 91.356 463.474 94.116H379.639ZM411.931 51.681C406.273 51.681 400.96 52.923 395.992 55.407C391.024 57.891 387.022 61.203 383.986 65.343C381.088 69.345 379.639 73.623 379.639 78.177H444.223C444.223 73.623 442.705 69.345 439.669 65.343C436.771 61.203 432.838 57.891 427.87 55.407C422.902 52.923 417.589 51.681 411.931 51.681ZM528.883 54.372C525.571 52.854 522.604 52.095 519.982 52.095C514.462 52.095 509.908 54.027 506.32 57.891C502.456 62.031 500.524 66.792 500.524 72.174V138H482.722V72.174C482.722 64.722 484.585 57.891 488.311 51.681C492.175 45.471 497.419 40.641 504.043 37.191C509.011 34.845 514.324 33.672 519.982 33.672C524.536 33.672 528.883 34.5 533.023 36.156C537.163 37.812 541.51 40.503 546.064 44.229L528.883 54.372Z"
        fill="url(#paint0_linear_2621_2)"
      />
      <path
        d="M630.86 138V122.061C626.444 127.305 621.2 131.445 615.128 134.481C609.194 137.517 602.984 139.035 596.498 139.035C589.184 139.035 582.353 137.724 576.005 135.102C569.657 132.342 564.137 128.478 559.445 123.51C554.615 118.542 550.82 112.884 548.06 106.536C545.438 100.05 544.127 93.219 544.127 86.043C544.127 78.867 545.438 72.105 548.06 65.757C550.82 59.271 554.615 53.544 559.445 48.576C564.137 43.608 569.657 39.813 576.005 37.191C582.353 34.431 589.184 33.051 596.498 33.051C603.674 33.051 610.16 34.5 615.956 37.398C621.89 40.296 626.858 44.298 630.86 49.404V0.551993H649.076V138H630.86ZM596.498 51.888C591.944 51.888 587.597 52.785 583.457 54.579C579.317 56.235 575.729 58.65 572.693 61.824C569.657 64.998 567.242 68.724 565.448 73.002C563.654 77.142 562.757 81.489 562.757 86.043C562.757 95.289 566.069 103.362 572.693 110.262C575.729 113.436 579.317 115.92 583.457 117.714C587.597 119.508 591.944 120.405 596.498 120.405C601.052 120.405 605.399 119.508 609.539 117.714C613.817 115.92 617.474 113.436 620.51 110.262C627.134 103.362 630.446 95.289 630.446 86.043C630.446 81.489 629.549 77.142 627.755 73.002C625.961 68.724 623.546 64.998 620.51 61.824C617.474 58.65 613.817 56.235 609.539 54.579C605.399 52.785 601.052 51.888 596.498 51.888ZM687.511 94.116C687.511 97.428 688.684 100.878 691.03 104.466C693.514 108.054 696.343 111.09 699.517 113.574C705.313 118.128 712.075 120.405 719.803 120.405C732.085 120.405 742.09 114.471 749.818 102.603L765.55 111.918C760.168 120.612 753.544 127.305 745.678 131.997C737.812 136.689 729.187 139.035 719.803 139.035C712.765 139.035 706.072 137.724 699.724 135.102C693.376 132.342 687.787 128.547 682.957 123.717C678.127 118.887 674.332 113.298 671.572 106.95C668.95 100.602 667.639 93.909 667.639 86.871C667.639 79.833 668.95 73.14 671.572 66.792C674.332 60.306 678.127 54.648 682.957 49.818C687.649 44.988 693.169 41.262 699.517 38.64C706.003 36.018 712.765 34.707 719.803 34.707C726.841 34.707 733.534 36.018 739.882 38.64C746.368 41.262 751.957 44.988 756.649 49.818C766.861 60.306 771.967 72.45 771.967 86.25C771.967 88.734 771.76 91.356 771.346 94.116H687.511ZM719.803 51.681C714.145 51.681 708.832 52.923 703.864 55.407C698.896 57.891 694.894 61.203 691.858 65.343C688.96 69.345 687.511 73.623 687.511 78.177H752.095C752.095 73.623 750.577 69.345 747.541 65.343C744.643 61.203 740.71 57.891 735.742 55.407C730.774 52.923 725.461 51.681 719.803 51.681ZM840.737 65.343C840.323 62.859 839.426 60.513 838.046 58.305C835.424 53.751 830.249 51.474 822.521 51.474C818.381 51.474 815.207 51.957 812.999 52.923C810.101 54.027 807.893 55.614 806.375 57.684C804.995 59.754 804.305 62.031 804.305 64.515C804.305 67.551 805.409 70.311 807.617 72.795C809.963 75.417 813.758 76.797 819.002 76.935C828.248 77.349 835.7 78.453 841.358 80.247C847.154 82.041 851.57 85.146 854.606 89.562C857.78 93.978 859.367 100.188 859.367 108.192C859.367 116.334 856.262 123.372 850.052 129.306C847.016 132.204 843.428 134.55 839.288 136.344C835.286 138.138 831.077 139.035 826.661 139.035H823.556C812.516 139.035 803.615 135.861 796.853 129.513C791.333 124.407 787.883 117.231 786.503 107.985H804.926C805.616 111.297 806.996 113.988 809.066 116.058C812.24 119.094 816.656 120.612 822.314 120.612C823.832 120.612 824.936 120.543 825.626 120.405C830.456 120.129 834.251 118.818 837.011 116.472C839.771 113.988 841.151 110.952 841.151 107.364C841.151 102.396 839.633 99.153 836.597 97.635C833.561 96.117 827.696 95.358 819.002 95.358C808.1 95.358 799.751 91.908 793.955 85.008C789.263 79.764 786.917 73.14 786.917 65.136C786.917 63.342 786.986 61.962 787.124 60.996C788.228 52.026 791.885 45.195 798.095 40.503C804.443 35.673 812.792 33.258 823.142 33.258C830.732 33.258 837.356 34.914 843.014 38.226C847.016 40.71 850.466 44.16 853.364 48.576C856.124 52.992 857.918 58.512 858.746 65.136L858.332 65.343H840.737ZM927.602 138L896.138 99.705V138H877.922V0.344996H896.138V59.754L924.29 33.258H951.407L902.348 78.798L951.614 138H927.602Z"
        fill="url(#paint1_linear_2621_2)"
      />
      <defs>
        <linearGradient
          id="paint0_linear_2621_2"
          x1="501.5"
          y1="-47.5"
          x2="499"
          y2="214.5"
          gradientUnits="userSpaceOnUse"
        >
          <stop />
        </linearGradient>
        <linearGradient
          id="paint0_linear_2621_2_dark"
          x1="501.5"
          y1="-47.5"
          x2="499"
          y2="214.5"
          gradientUnits="userSpaceOnUse"
        >
          <stop stopColor={'#fff'} />
        </linearGradient>
        <linearGradient
          id="paint1_linear_2621_2"
          x1="501.5"
          y1="-47.5"
          x2="499"
          y2="214.5"
          gradientUnits="userSpaceOnUse"
        >
          <stop stopColor="#8E79DE" />
        </linearGradient>
      </defs>
    </svg>
  );
}

export function AppLogo({
  href,
  label,
  className,
}: {
  href?: string;
  className?: string;
  label?: string;
}) {
  return (
    <Link aria-label={label ?? 'Home Page'} href={href ?? '/'}>
      <LogoImage className={className} />
    </Link>
  );
}
```

You can replace the SVG with your own SVG. You can also use an image or text logo.

After updating both the colors and the logo, we start giving our SaaS product a new soul. It's starting to look like our own product.

{% img src="/assets/courses/next-turbo/makerdesk-logo.webp" width="1086" height="1264" /%}

### Updating the Favicons

The favicons, instead, are stored in the public folder at `apps/web/public/images/favicon`. You can replace the favicons with your own favicons.

For example, use the [Favicon Generator](https://favicon.io/) to generate the favicons and replace the favicons in the `apps/web/public/images/favicon` folder. Normally, it's just a matter of dropping the new files and overwrite the existing ones using the Makerkit logo.

### Updating the Fonts

Updating the fonts is well covered in [the kit's documentation](https://makerkit.dev/docs/next-supabase-turbo/fonts).

Let's update the font in our application so we can see the changes. I will replace the default font for the headings with `Quicksand`. It's extremely easy, we only need to update the import:

```tsx {% title="apps/web/lib/fonts.ts" %} {1}
import { Quicksand as HeadingFont, Inter as SansFont } from 'next/font/google';

/**
 * @sans
 * @description Define here the sans font.
 * By default, it uses the Inter font from Google Fonts.
 */
const sans = SansFont({
  subsets: ['latin'],
  variable: '--font-sans',
  fallback: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
  preload: true,
  weight: ['300', '400', '500', '600', '700'],
});

/**
 * @heading
 * @description Define here the heading font.
 * By default, it uses the Urbanist font from Google Fonts.
 */
const heading = HeadingFont({
  subsets: ['latin'],
  variable: '--font-heading',
  fallback: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
  preload: true,
  weight: ['500', '700'],
});

// we export these fonts into the root layout
export { sans, heading };
```

Great, the font has been updated.

NB: feel free to choose any other font (or keep the existing one). Our goal here is only to show you how to update the font should you want to.

By default, we use the `sans` font for the body text and the `heading` font for the headings.

## Home Page

Once you've updated the colors, logo, favicons, and fonts, you can update the home page to reflect your application's purpose.

I normally don't work on the home page right away, but I like to update the main heading and the subheading. So that's what we'll do now.

Since our application is a real-time customer support ticket SaaS, I tasked Claude Sonnet to write a catchy subheading for the home page. He came up with the following:

- **Main Heading**: "Customer Support for busy creators"
- **Subheading**: "Collect and manage customer support in one place. Solve customer issues faster and keep them happy."

We can now start updating our website's home page with this content.

As we've seen in the previous lesson, the marketing section lives in the `apps/web/app/(marketing)` folder. This folder uses the pathless routing feature of Next.js to render the marketing pages at the root of the project, while separating them into a different layout for UI consistency.

```tsx {% title="apps/web/app/(marketing)/page.tsx" %}
<HeroTitle>
  <span className={'text-primary'}>
    Customer Support
  </span>

  <span>
    <span>for busy creators</span>
  </span>
</HeroTitle>
```

And then we can update the subheading:

```tsx {% title="apps/web/app/(marketing)/page.tsx" %}
<div className={'flex flex-col items-center space-y-8'}>
  <HeroTitle>
    <span className={'text-primary'}>
      Customer Support
    </span>

    <span>
      <span>for busy creators</span>
    </span>
  </HeroTitle>

  <div className={'flex flex-col'}>
    <Heading
      level={2}
      className={
        'p-0 text-center font-sans text-2xl font-normal text-muted-foreground'
      }
    >
      <span>
        Collect and manage customer support in one place.
      </span>
    </Heading>

    <Heading
      level={2}
      className={
        'p-0 text-center font-sans text-2xl font-normal text-muted-foreground'
      }
    >
      <span>
        Solve customer issues faster and keep them happy.
      </span>
    </Heading>

    <Heading
      level={2}
      className={
        'p-0 text-center font-sans text-2xl font-normal text-muted-foreground'
      }
    >
      Start for free. No credit card required.
    </Heading>
  </div>

  <MainCallToActionButton />
</div>
```

{% img src="/assets/courses/next-turbo/landing-page.webp" width="2984" height="2224" /%}

Et voilà! We've updated the home page with the new content. This is enough for this module's purpose, but you can continue to update the home page with more content, such as testimonials, features, etc.

## Updating the Features Configuration

The features configuration is stored at `apps/web/config/feature-flags.config.ts`. This file contains the features of the application and their status (enabled or disabled).

```tsx {% title="apps/web/config/feature-flags.config.ts" %}

const featuresFlagConfig = FeatureFlagsSchema.parse({
  enableThemeToggle: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_THEME_TOGGLE,
    true,
  ),
  enableAccountDeletion: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_DELETION,
    false,
  ),
  enableTeamDeletion: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_DELETION,
    false,
  ),
  enableTeamAccounts: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS,
    true,
  ),
  enableTeamCreation: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_CREATION,
    true,
  ),
  enablePersonalAccountBilling: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING,
    false,
  ),
  enableTeamAccountBilling: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING,
    false,
  ),
  languagePriority: process.env
    .NEXT_PUBLIC_LANGUAGE_PRIORITY as LanguagePriority,
  enableNotifications: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_NOTIFICATIONS,
    true,
  ),
  realtimeNotifications: getBoolean(
    process.env.NEXT_PUBLIC_REALTIME_NOTIFICATIONS,
    false,
  ),
  enableVersionUpdater: getBoolean(
    process.env.NEXT_PUBLIC_ENABLE_VERSION_UPDATER,
    false,
  ),
} satisfies z.infer<typeof FeatureFlagsSchema>);
```

As you can see - we do have quite a few flags that you can enable or disable. You can update the flags to reflect the features of your application.

Since our application is meant to be B2B - we can disable the personal account billing:

```tsx {% title="apps/web/.env" %}
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING="false"
```

Team Account billing is enabled by default, so we keep it as is. Please change any other flags as you see fit.

You can also add new flags to the configuration if you want to add new features to your application.

## Exploring the Application

Okay, before we wrap up, let's explore the application to see the changes we've made - and to give you an overview of the application.

### Running Supabase

To run Supabase, you need to make sure Docker is currently running on your machine.

Is Docker running? If not, start Docker.

You can now start Supabase by running the following command:

```bash
pnpm run supabase:web:start
```

### Running the Application

To run the application, you can use the following command:

```bash
pnpm run dev
```

This will start the application in development mode.

You can then open your browser and navigate to `http://localhost:3000` to see the application - and you will see the changes we've made, with a new color, logo (and if you changed it) a new font.

### Logging in to the Application

You can log in to the application using the following credentials:

- **Email**: `test@makerkit.dev`
- **Password**: `testingpassword`

#### Signing up for a new account
You can also sign up for a new account if you want. If you do, make sure to navigate to [InBucket](http://localhost:54324/monitor) to confirm your email address.

Once you've logged in, you're signed into the user dashboard at `/home`. From there, you can navigate to the User Settings, or navigate to a Team Account using the dropdown in the sidebar.

Please feel free to familiarize yourself with the application and explore the features.

## Wrapping Up

In this module, **we've updated the colors, logo, favicons, and fonts of our application**. We've also updated the home page with new content. You can continue to update the home page with more content, such as testimonials, features, etc.

We also learned **how Makerkit stores and validates the configuration using environment variables and Zod**. This is (IMHO) a great way to manage the configuration of your application, and I recommend you use it in your projects as they grow.

In the next module, we'll learn **how to develop the Database schema for our application**. We'll use Supabase to create the database schema and set up the database for our application. See you there!


