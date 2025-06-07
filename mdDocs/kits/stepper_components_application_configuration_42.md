-----------------
FILE PATH: kits\next-supabase-turbo\components\stepper.mdoc

---
status: "published"
label: "Stepper"
title: "Stepper Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Stepper component in the Next.js Supabase SaaS kit"
order: 1
---

The Stepper component is a versatile UI element designed to display a series of steps in a process or form. It provides visual feedback on the current step and supports different visual styles.

{% component path="steppers/stepper" /%}

## Usage

```jsx
import { Stepper } from '@kit/ui/stepper';

function MyComponent() {
  return (
    <Stepper
      steps={['Step 1', 'Step 2', 'Step 3']}
      currentStep={1}
      variant="default"
    />
  );
}
```

## Props

The Stepper component accepts the following props:

- `steps: string[]` (required): An array of strings representing the labels for each step.
- `currentStep: number` (required): The index of the currently active step (0-based).
- `variant?: 'numbers' | 'default'` (optional): The visual style of the stepper. Defaults to 'default'.

## Variants

The Stepper component supports two visual variants:

1. `default`: Displays steps as a horizontal line with labels underneath.
2. `numbers`: Displays steps as numbered circles with labels between them.

{% component path="steppers/stepper-numbers" /%}

## Features

- Responsive design that adapts to different screen sizes
- Dark mode support
- Customizable appearance through CSS classes and variants
- Accessibility support with proper ARIA attributes

## Component Breakdown

### Main Stepper Component

The main `Stepper` function renders the overall structure of the component. It:

- Handles prop validation and default values
- Renders nothing if there are fewer than two steps
- Uses a callback to render individual steps
- Applies different CSS classes based on the chosen variant

### Steps Rendering

Steps are rendered using a combination of divs and spans, with different styling applied based on:

- Whether the step is currently selected
- The chosen variant (default or numbers)

### StepDivider Component

For the 'numbers' variant, a `StepDivider` component is used to render the labels between numbered steps. It includes:

- Styling for selected and non-selected states
- A divider line between steps (except for the last step)

## Styling

The component uses a combination of:

- Tailwind CSS classes for basic styling
- `cva` (Class Variance Authority) for managing variant-based styling
- `classNames` function for conditional class application

## Accessibility

- The component uses `aria-selected` to indicate the current step
- Labels are associated with their respective steps for screen readers

## Customization

You can further customize the appearance of the Stepper by:

1. Modifying the `classNameBuilder` function to add or change CSS classes
2. Adjusting the Tailwind CSS classes in the component JSX
3. Creating new variants in the `cva` configuration

## Example

```jsx
<Stepper
  steps={['Account', 'Personal Info', 'Review']}
  currentStep={1}
  variant="numbers"
/>
```

This will render a numbered stepper with three steps, where "Personal Info" is the current (selected) step.

The Stepper component provides a flexible and visually appealing way to guide users through multi-step processes in your application. Its support for different variants and easy customization makes it adaptable to various design requirements.

-----------------
FILE PATH: kits\next-supabase-turbo\components.mdoc

---
status: "published"
title: "UI Components in the Next.js Supabase Turbo SaaS Boilerplate"
label: "UI Components"
order: 9
description: "Components in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Components in Next.js Supabase Turbo


-----------------
FILE PATH: kits\next-supabase-turbo\configuration\application-configuration.mdoc

---
status: "published"
label: "Application Configuration"
title: "Setting your application configuration | Next.js Supabase SaaS Kit"
description: "Learn how to setup the overall settings of your Next.js Supabase application"
order: 1
---

The application configuration is set at `apps/web/config/app.config.ts`. This configuration stores some overall variables for your application.

This configuration is set at application-level. The configuration gets propagated to the packages that the app imports, so you can control the behavior and logic of the package. This also allows you to host multiple apps in the same monorepo, as every application defines its own configuration.

The recommendation is **to not update this directly** - instead, please define the environment variables below and override the default behavior. The configuration is validated using the Zod schema `AppConfigSchema`, so if something is off, you'll see the errors.

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

For example, to set the product name and app URL, you'd update the variables:

```bash
NEXT_PUBLIC_SITE_URL=https://myapp.com
NEXT_PUBLIC_PRODUCT_NAME="My wonderful AI App"
```

-----------------
FILE PATH: kits\next-supabase-turbo\configuration\authentication-configuration.mdoc

---
status: "published"
label: "Authentication Configuration"
title: "Setting your authentication configuration"
description: "Learn how to setup the authentication configuration of your Next.js Supabase application"
order: 2
---

{% sequence title="Authentication Configuration" description="Learn how to setup the authentication configuration of your Next.js Supabase application" %}

[Supported Authentication Methods](#supported-authentication-methods)

[Third Party Providers](#third-party-providers)

[Password Requirements](#password-requirements)

[Displaying Terms and Conditions](#displaying-terms-and-conditions)

{% /sequence %}

## Supported Authentication Methods

Makerkit supports three different authentication methods:

1. **Password** - the traditional email/password method, set to `true` by default
2. **Magic Link** - magic link, set to `false` by default
3. **oAuth** - oAuth providers, by default we set Google Auth

The authentication configuration is set at `apps/web/config/auth.config.ts`.

The recommendation is **to not update this directly** - instead, please define the environment variables below and override the default behavior. The configuration is validated using the Zod schema `AuthConfigSchema`, so if something is off, you'll see the errors.

```tsx
const authConfig = AuthConfigSchema.parse({
  // NB: This is a public key, so it's safe to expose.
  // Copy the value from the Supabase Dashboard.
  captchaTokenSiteKey: process.env.NEXT_PUBLIC_CAPTCHA_SITE_KEY,

  // whether to display the terms checkbox during sign-up
  displayTermsCheckbox:
    process.env.NEXT_PUBLIC_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX === 'true',

  // NB: Enable the providers below in the Supabase Console
  // in your production project
  providers: {
    password: process.env.NEXT_PUBLIC_AUTH_PASSWORD === 'true',
    magicLink: process.env.NEXT_PUBLIC_AUTH_MAGIC_LINK === 'true',
    oAuth: ['google'],
  },
} satisfies z.infer<typeof AuthConfigSchema>);
```

For example, if you wanted to switch from password auth to magic link, you'd set the below variables:

```bash
NEXT_PUBLIC_AUTH_PASSWORD=false
NEXT_PUBLIC_AUTH_MAGIC_LINK=true
```

## Third Party Providers

To display third-party providers in the UI, you need to set the `oAuth` array to include the provider you want to display. The default is Google.

```tsx
providers: {
  oAuth: ['google'],
}
```

**The configuration is done on Supabase's side - not on Makerkit's side**.

Third Party providers need to be configured, managed and enabled fully on the provider's and Supabase's side. Makerkit does not need any configuration (beyond setting the provider to be displayed in the UI).

Please [read Supabase's documentation](https://supabase.com/docs/guides/auth/social-login) on how to set up third-party providers.

For local development, also check out [Supabase's documentation on how to set up OAuth providers locally](https://supabase.com/docs/guides/local-development/managing-config).

### Scopes

Scopes are used to request specific permissions from the user. Different providers support and require different scopes.

To add your required scopes, please specify them in the `OAuthProviders` component.

 ```tsx {% title="packages/features/auth/src/components/oauth-providers.tsx" %}
/**
 * @name OAUTH_SCOPES
 * @description
 * The OAuth scopes are used to specify the permissions that the application is requesting from the user.
 *
 * Please add your OAuth providers here and the scopes you want to use.
 *
 * @see https://supabase.com/docs/guides/auth/social-login
 */
const OAUTH_SCOPES: Partial<Record<Provider, string>> = {
  azure: 'email',
  // add your OAuth providers here
};
```

For example, if you want to request the user's email address, you can add the `email` scope to the Google provider.

 ```tsx {% title="packages/features/auth/src/components/oauth-providers.tsx" %}
const OAUTH_SCOPES: Partial<Record<Provider, string>> = {
  azure: 'email',
  google: 'email',
};
```

## Password Requirements

To set the password requirements, you can set the following environment variables:

```bash
NEXT_PUBLIC_PASSWORD_REQUIRE_UPPERCASE=true
NEXT_PUBLIC_PASSWORD_REQUIRE_NUMBERS=true
NEXT_PUBLIC_PASSWORD_REQUIRE_SPECIAL_CHARS=true
```

The above will enforce the following rules:
1. At least one uppercase letter
2. At least one number
3. At least one special character

## Displaying Terms and Conditions

To display the terms and conditions checkbox during sign-up, set the following environment variable:

```bash
NEXT_PUBLIC_DISPLAY_TERMS_AND_CONDITIONS_CHECKBOX=true
```

This is off by default.

## MFA

To enforce MFA for RLS (Row Level Security) you need to customize the RLS policies as per [the Supabase documentation](https://supabase.com/blog/mfa-auth-via-rls).

-----------------
FILE PATH: kits\next-supabase-turbo\configuration\environment-variables.mdoc

---
status: "published"
title: Environment Variables in the Next.js Supabase Starter Kit
label: Environment Variables
order: 0
description: Learn how to configure environment variables in the Next.js Supabase Starter Kit.
---

{% sequence title="Environment Variables" description="Learn how to configure environment variables in the Next.js Supabase Starter Kit." %}

[Understanding Environment Variables](#understanding-environment-variables)

[Core Configuration Variables](#core-configuration-variables)

[Authentication Configuration](#authentication-configuration)

[Navigation and Layout Settings](#navigation-and-layout-settings)

[UI/UX](#ui/ux)

[Feature Flags](#feature-flags)

[Notifications](#notifications)

[Supabase Configuration](#supabase-configuration)

[Billing Configuration](#billing-configuration)

[Email Configuration](#email-configuration)

[CMS Configuration](#cms-configuration)

{% /sequence %}

## Understanding Environment Variables

Environment variables are defined in the `.env` file in the root of the `apps/web` package.

1. **Shared Environment Variables**: Shared environment variables are defined in the `.env` file. These are the env variables shared between environments (e.g., development, staging, production).
2. **Environment Variables**: Environment variables for a specific environment are defined in the `.env.development` and `.env.production` files. These are the env variables specific to the development and production environments.
3. **Secret Keys**: Secret keys and sensitive information are not stored in the `.env` file. Instead, they are stored in the environment variables of the CI/CD system.
4. **Secret keys to be used locally**: If you need to use secret keys locally, you can store them in the `.env.local` file. This file is not committed to Git, therefore it is safe to store sensitive information in it.

## Core Configuration Variables

Essential variables that define your application's basic settings:

```bash
NEXT_PUBLIC_SITE_URL=https://example.com
NEXT_PUBLIC_PRODUCT_NAME=Your Product
NEXT_PUBLIC_SITE_TITLE="Your Site Title"
NEXT_PUBLIC_SITE_DESCRIPTION="Your site description"
NEXT_PUBLIC_DEFAULT_LOCALE=en
```

### NEXT_PUBLIC_SITE_URL
The URL of your site, used for generating absolute URLs. Must include the protocol (e.g., `https://`) and be a valid URL.
```bash
NEXT_PUBLIC_SITE_URL=https://example.com
```

### NEXT_PUBLIC_PRODUCT_NAME
Your product's name, used consistently across the application interface.
```bash
NEXT_PUBLIC_PRODUCT_NAME=Makerkit
```

### NEXT_PUBLIC_SITE_TITLE
The site's title tag content, crucial for SEO and browser display.
```bash
NEXT_PUBLIC_SITE_TITLE="Makerkit - The easiest way to start your next project"
```

### NEXT_PUBLIC_SITE_DESCRIPTION
Your site's meta description, important for SEO optimization.
```bash
NEXT_PUBLIC_SITE_DESCRIPTION="Makerkit is the easiest way to start your next project"
```

### NEXT_PUBLIC_DEFAULT_LOCALE
Sets the default language for your application.
```bash
NEXT_PUBLIC_DEFAULT_LOCALE=en
```

## Authentication Configuration

Settings that control user authentication methods:

```bash
NEXT_PUBLIC_AUTH_PASSWORD=true
NEXT_PUBLIC_AUTH_MAGIC_LINK=false
NEXT_PUBLIC_CAPTCHA_SITE_KEY=
CAPTCHA_SECRET_TOKEN=
```

### NEXT_PUBLIC_AUTH_PASSWORD
Enables or disables password-based authentication.
```bash
NEXT_PUBLIC_AUTH_PASSWORD=true
```

### NEXT_PUBLIC_AUTH_MAGIC_LINK
Enables or disables magic link authentication.
```bash
NEXT_PUBLIC_AUTH_MAGIC_LINK=false
```

### NEXT_PUBLIC_CAPTCHA_SITE_KEY
Your Cloudflare Captcha site key for form protection.
```bash
NEXT_PUBLIC_CAPTCHA_SITE_KEY=your-captcha-site-key
```

### CAPTCHA_SECRET_TOKEN
Your Cloudflare Captcha secret token for backend verification.
```bash
CAPTCHA_SECRET_TOKEN=your-captcha-secret-token
```

## Navigation and Layout Settings

Settings that control the navigation and layout of your application:
```bash
NEXT_PUBLIC_USER_NAVIGATION_STYLE=sidebar
NEXT_PUBLIC_HOME_SIDEBAR_COLLAPSED=false
NEXT_PUBLIC_TEAM_NAVIGATION_STYLE=sidebar
NEXT_PUBLIC_TEAM_SIDEBAR_COLLAPSED=false
NEXT_PUBLIC_SIDEBAR_COLLAPSIBLE_STYLE=icon
```

### NEXT_PUBLIC_USER_NAVIGATION_STYLE
Controls user navigation layout. Options: `sidebar`, `header`, or `custom`.
```bash
NEXT_PUBLIC_USER_NAVIGATION_STYLE=sidebar
```

### NEXT_PUBLIC_HOME_SIDEBAR_COLLAPSED
Sets the default state of the home sidebar.
```bash
NEXT_PUBLIC_HOME_SIDEBAR_COLLAPSED=false
```

### NEXT_PUBLIC_TEAM_NAVIGATION_STYLE
Controls team navigation layout. Options: `sidebar`, `header`, or `custom`.
```bash
NEXT_PUBLIC_TEAM_NAVIGATION_STYLE=sidebar
```

### NEXT_PUBLIC_TEAM_SIDEBAR_COLLAPSED
Sets the default state of the team sidebar.
```bash
NEXT_PUBLIC_TEAM_SIDEBAR_COLLAPSED=false
```

### NEXT_PUBLIC_SIDEBAR_COLLAPSIBLE_STYLE
Defines sidebar collapse behavior. Options: `offcanvas`, `icon`, or `none`.
```bash
NEXT_PUBLIC_SIDEBAR_COLLAPSIBLE_STYLE=icon
```

## UI/UX

Settings that control the UI/UX of your application:

```
NEXT_PUBLIC_DEFAULT_THEME_MODE=light
NEXT_PUBLIC_ENABLE_THEME_TOGGLE=true
NEXT_PUBLIC_ENABLE_SIDEBAR_TRIGGER=true
```

### NEXT_PUBLIC_DEFAULT_THEME_MODE
Controls the default theme appearance. Options: `light`, `dark`, or `system`.
```bash
NEXT_PUBLIC_DEFAULT_THEME_MODE=light
```

### NEXT_PUBLIC_ENABLE_THEME_TOGGLE
Controls visibility of the theme toggle feature.
```bash
NEXT_PUBLIC_ENABLE_THEME_TOGGLE=true
```

### NEXT_PUBLIC_ENABLE_SIDEBAR_TRIGGER
Controls visibility of the sidebar trigger feature.
```bash
NEXT_PUBLIC_ENABLE_SIDEBAR_TRIGGER=true
```

## Feature Flags

```bash
NEXT_PUBLIC_ENABLE_THEME_TOGGLE=true
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_DELETION=true
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING=true
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS=true
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_CREATION=true
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_DELETION=true
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING=true
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=true
NEXT_PUBLIC_REALTIME_NOTIFICATIONS=true
```

### NEXT_PUBLIC_ENABLE_THEME_TOGGLE
Controls visibility of the theme toggle feature.
```bash
NEXT_PUBLIC_ENABLE_THEME_TOGGLE=true
```

### NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_DELETION
Allows users to delete their personal accounts.
```bash
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_DELETION=true
```

### NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING
Enables billing features for personal accounts.
```bash
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING=true
```

### NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS
Master switch for team account functionality.
```bash
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS=true
```

### NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_CREATION
Controls ability to create new team accounts.
```bash
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_CREATION=true
```

### NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_DELETION
Allows team account deletion.
```bash
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_DELETION=true
```

### NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING
Enables billing features for team accounts.
```bash
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING=true
```

## Notifications

Control notification behavior:

```bash
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=true
NEXT_PUBLIC_REALTIME_NOTIFICATIONS=true
```

### NEXT_PUBLIC_ENABLE_NOTIFICATIONS
Controls the notification system.
```bash
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=true
```

### NEXT_PUBLIC_REALTIME_NOTIFICATIONS
Enables real-time notifications using Supabase Realtime.
```bash
NEXT_PUBLIC_REALTIME_NOTIFICATIONS=true
```

## Supabase Configuration

Required variables for Supabase integration:

```bash
NEXT_PUBLIC_SUPABASE_URL=https://yourproject.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
SUPABASE_DB_WEBHOOK_SECRET=your-webhook-secret
```

### NEXT_PUBLIC_SUPABASE_URL
Your Supabase project URL.
```bash
NEXT_PUBLIC_SUPABASE_URL=https://yourproject.supabase.co
```

### NEXT_PUBLIC_SUPABASE_ANON_KEY
Your Supabase anonymous API key.
```bash
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

### SUPABASE_SERVICE_ROLE_KEY
Your Supabase service role key (keep this secret!).
```bash
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

### SUPABASE_DB_WEBHOOK_SECRET
Secret key for Supabase webhook verification.
```bash
SUPABASE_DB_WEBHOOK_SECRET=your-webhook-secret
```

## Billing Configuration

### NEXT_PUBLIC_BILLING_PROVIDER
Your chosen billing provider. Options: `stripe` or `lemon-squeezy`.
```bash
NEXT_PUBLIC_BILLING_PROVIDER=stripe
```

### Stripe Configuration

Required when using Stripe:
```bash
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your-publishable-key
STRIPE_SECRET_KEY=your-secret-key
STRIPE_WEBHOOK_SECRET=your-webhook-secret
```

#### NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

Your Stripe publishable key.
```bash
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your-publishable-key
```

#### STRIPE_SECRET_KEY

Your Stripe secret key.
```bash
STRIPE_SECRET_KEY=your-secret-key
```

#### STRIPE_WEBHOOK_SECRET

Your Stripe webhook secret.
```bash
STRIPE_WEBHOOK_SECRET=your-webhook-secret
```

### Lemon Squeezy Configuration
Required when using Lemon Squeezy:
```bash
LEMON_SQUEEZY_SECRET_KEY=your-secret-key
LEMON_SQUEEZY_STORE_ID=your-store-id
LEMON_SQUEEZY_SIGNING_SECRET=your-signing-secret
```

#### LEMON_SQUEEZY_SECRET_KEY

Your Lemon Squeezy secret key.
```bash
LEMON_SQUEEZY_SECRET_KEY=your-secret-key
```

#### LEMON_SQUEEZY_STORE_ID

Your Lemon Squeezy store ID.
```bash
LEMON_SQUEEZY_STORE_ID=your-store-id
```

#### LEMON_SQUEEZY_SIGNING_SECRET

Your Lemon Squeezy signing secret.
```bash
LEMON_SQUEEZY_SIGNING_SECRET=your-signing-secret
```

## Email Configuration

Setup email functionality:

```bash
MAILER_PROVIDER=nodemailer
EMAIL_SENDER=info@yourapp.com
CONTACT_EMAIL=contact@yourapp.com # contact form email

# For Resend
RESEND_API_KEY=your-resend-api-key

# For Nodemailer
EMAIL_HOST=smtp.provider.com
EMAIL_PORT=587
EMAIL_USER=your-email-user
EMAIL_PASSWORD=your-email-password
EMAIL_TLS=true
```

### MAILER_PROVIDER
Your email service provider. Options: `nodemailer` or `resend`.
```bash
MAILER_PROVIDER=nodemailer
```

### EMAIL_SENDER
Default sender email address.
```bash
EMAIL_SENDER=info@yourapp.com
```

### CONTACT_EMAIL
Email address for contact form submissions.
```bash
CONTACT_EMAIL=contact@yourapp.com
```

### Resend Configuration
Required when using Resend:
```bash
RESEND_API_KEY=your-resend-api-key
```

### Nodemailer Configuration
Required when using Nodemailer:
```bash
EMAIL_HOST=smtp.provider.com
EMAIL_PORT=587
EMAIL_USER=your-email-user
EMAIL_PASSWORD=your-email-password
EMAIL_TLS=true
```

## CMS Configuration

Setup CMS functionality:

```bash
CMS_CLIENT=keystatic
```

### CMS_CLIENT

Your chosen CMS system. Options: `wordpress` or `keystatic`.
```bash
CMS_CLIENT=keystatic
```

### Keystatic Configuration

```bash
NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND=local # local, cloud, github
KEYSTATIC_PATH_PREFIX=apps/web
NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH=./content # apps/web/content

# if Github mode
NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND=github
NEXT_PUBLIC_KEYSTATIC_STORAGE_REPO=makerkit/next-supabase-saas-kit-turbo-demo
NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH=./content
KEYSTATIC_GITHUB_TOKEN=github_**********************************************
KEYSTATIC_PATH_PREFIX=apps/web
```

#### NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND

Your Keystatic storage kind. Options: `local`, `cloud`, `github`.
```bash
NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND=local
```

#### NEXT_PUBLIC_KEYSTATIC_STORAGE_REPO

Your Keystatic storage repo.
```bash
NEXT_PUBLIC_KEYSTATIC_STORAGE_REPO=makerkit/next-supabase-saas-kit-turbo-demo
```

#### KEYSTATIC_GITHUB_TOKEN

Your Keystatic GitHub token.
```bash
KEYSTATIC_GITHUB_TOKEN=github_**********************************************
```

#### KEYSTATIC_PATH_PREFIX

Your Keystatic path prefix.
```bash
KEYSTATIC_PATH_PREFIX=apps/web
```

#### NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH

Your Keystatic content path.
```bash
NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH=./content
```

#### KEYSTATIC_PATH_PREFIX

Your Keystatic path prefix.
```bash
KEYSTATIC_PATH_PREFIX=apps/web
```

### Wordpress Configuration

### WORDPRESS_API_URL
Required when using WordPress:
```bash
WORDPRESS_API_URL=https://your-wordpress-site.com/wp-json
```

## Best Practices 🔑

1. **Environment File Structure**
   - `.env`: Shared, non-sensitive variables
   - `.env.development`: Development-specific settings
   - `.env.production`: Production-specific settings
   - `.env.local`: Local sensitive variables (git-ignored)
2. **Security Considerations**
   - Never commit secret keys
   - Use CI/CD environment variables for sensitive data
   - Rotate keys regularly

-----------------
FILE PATH: kits\next-supabase-turbo\configuration\feature-flags-configuration.mdoc

---
status: "published"
label: "Feature Flags"
title: "Setting your feature flags configuration | Next.js Supabase SaaS Kit"
description: "Learn how to setup the feature flags configuration of your Next.js Supabase application"
order: 4
---

The features flags configuration is set at `apps/web/config/feature-flags.config.ts`. We use this configuration to store feature flags, i.e. to enable or disable certain features in the app.

This configuration is set at application level. The configuration gets propagated to the packages that the app imports, so you can control the behavior and logic of the package. This also allows you to host multiple apps in the same monorepo, as every application defines its own configuration.

The recommendation is **to not update this directly** - instead, please define the environment variables below and override the default behavior. The configuration is validated using the Zod schema `FeatureFlagsSchema`, so if something is off, you'll see the errors.

```tsx

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

export default featuresFlagConfig;
```

You can update the following environment variables to override the default behavior:

```bash
NEXT_PUBLIC_ENABLE_THEME_TOGGLE=
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_DELETION=
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS=
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_DELETION=
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_CREATION=
NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING=
NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING=
NEXT_PUBLIC_LANGUAGE_PRIORITY=
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=
NEXT_PUBLIC_REALTIME_NOTIFICATIONS=
NEXT_PUBLIC_ENABLE_VERSION_UPDATER=
```

### Explanation

1. **NEXT_PUBLIC_ENABLE_THEME_TOGGLE**: use this feature if you want to set a specific theme mode (dark or light) and disallow switching to another mode
2. **NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_DELETION**: use this feature flag if you don't want users to self-delete their accounts
3. **NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS**: use this feature flag to enable or disable team accounts. For B2C apps, disabling may be your preference.
4. **NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_DELETION**: use this feature flag if you don't want users to self-delete their team accounts
5. **NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_CREATION**: use this feature flag to prevent users from creating a team account. This can be useful if you wish to manage the onboarding yourself.
6. **NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING**: use this feature to enable or disable the ability of personal accounts to subscribe to a plan.
7. **NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING**: use this feature to enable or disable the ability of team accounts to subscribe to a plan.
8. **NEXT_PUBLIC_LANGUAGE_PRIORITY**: use this feature to set the language priority. If set to `user`, the user's preferred language will be used. If set to `application`, the application's default language will be used.
9. **NEXT_PUBLIC_ENABLE_NOTIFICATIONS**: use this feature to enable or disable notifications in the app.
10. **NEXT_PUBLIC_REALTIME_NOTIFICATIONS**: use this feature to enable or disable real-time notifications in the app.
11. **NEXT_PUBLIC_ENABLE_VERSION_UPDATER**: use this feature to enable or disable the version updater in the app.

### Note

It is unlikely that both `NEXT_PUBLIC_ENABLE_PERSONAL_ACCOUNT_BILLING` and `NEXT_PUBLIC_ENABLE_TEAM_ACCOUNTS_BILLING` would both be enabled at once. In most cases, you want to enable it for users or team accounts.


-----------------
FILE PATH: kits\next-supabase-turbo\configuration\paths-configuration.mdoc

---
status: "published"

label: "Paths Configuration"
title: "Setting your paths configuration"
description: "Learn how to setup the paths configuration of your Next.js Supabase application"
order: 3
---

The paths configuration is set at `apps/web/config/paths.config.ts`. This configuration stores all the paths that you'll be using in your application. It is a convenient way to store them in a central place rather than scatter them in the codebase using magic strings.

The configuration is validated using the Zod schema `PathsSchema`, so if something is off, you'll see the errors.

```tsx
const pathsConfig = PathsSchema.parse({
  auth: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    verifyMfa: '/auth/verify',
    callback: '/auth/callback',
    passwordReset: '/auth/password-reset',
    passwordUpdate: '/update-password',
  },
  app: {
    home: '/home',
    personalAccountSettings: '/home/settings',
    personalAccountBilling: '/home/billing',
    personalAccountBillingReturn: '/home/billing/return',
    accountHome: '/home/[account]',
    accountSettings: `/home/[account]/settings`,
    accountBilling: `/home/[account]/billing`,
    accountMembers: `/home/[account]/members`,
    accountBillingReturn: `/home/[account]/billing/return`,
    joinTeam: '/join',
  },
} satisfies z.infer<typeof PathsSchema>);
```

-----------------
FILE PATH: kits\next-supabase-turbo\configuration\personal-account-sidebar-configuration.mdoc

---
status: "published"
label: "Account Navigation Configuration"
title: "Setting your personal account navigation configuration | Next.js Supabase SaaS Kit"
description: "Learn how to setup the personal account navigation of your Next.js Supabase application"
order: 5
---

{% sequence title="Personal Account Navigation Configuration" description="Learn how to setup the personal account navigation of your Next.js Supabase application" %}

[Configuration](#configuration)

[Layout Style](#layout-style)

{% /sequence %}

The personal account navigation is set at `apps/web/config/personal-account-navigation.config.tsx`. We use this configuration to define the navigation menu of the personal account. By default, it has three routes: home page, settings, and billing (if enabled).

We define it in one place so we can build different views at once (for example, the mobile menu).

## Configuration

You define the personal workspace routing configuration at `apps/web/config/personal-account-navigation.config.tsx`:

 ```tsx {% title="apps/web/config/personal-account-navigation.config.tsx" %}
import { CreditCard, Home, User } from 'lucide-react';
import { z } from 'zod';

import { NavigationConfigSchema } from '@kit/ui/navigation-schema';

import featureFlagsConfig from '~/config/feature-flags.config';
import pathsConfig from '~/config/paths.config';

const iconClasses = 'w-4';

const routes = [
  {
    label: 'common:routes.application',
    children: [
      {
        label: 'common:routes.home',
        path: pathsConfig.app.home,
        Icon: <Home className={iconClasses} />,
        end: true,
      },
    ],
  },
  {
    label: 'common:routes.settings',
    children: [
      {
        label: 'common:routes.profile',
        path: pathsConfig.app.personalAccountSettings,
        Icon: <User className={iconClasses} />,
      },
      featureFlagsConfig.enablePersonalAccountBilling
        ? {
            label: 'common:routes.billing',
            path: pathsConfig.app.personalAccountBilling,
            Icon: <CreditCard className={iconClasses} />,
          }
        : undefined,
    ].filter(route => !!route),
  },
] satisfies z.infer<typeof NavigationConfigSchema>['routes'];

export const personalAccountNavigationConfig = NavigationConfigSchema.parse({
  routes,
  style: process.env.NEXT_PUBLIC_USER_NAVIGATION_STYLE,
  sidebarCollapsed: process.env.NEXT_PUBLIC_HOME_SIDEBAR_COLLAPSED,
});
```

The configuration allows you to define route groups. You can add a route's children by setting the `children` attribute (for simplicity, I don't use i18n strings):

 ```tsx {% title="apps/web/config/personal-account-navigation.config.tsx" %}
const routes = [{
  label: 'Your Store',
  children: [
    {
      label: 'Dashboard',
      path: '/dashboard',
      Icon: <DashboardIcon className={iconClasses} />,
      children: [
        {
          label: 'Orders',
          path: '/orders',
          Icon: <ShoppingCartIcon className={iconClasses} />,
        },
        {
          label: 'Inventory',
          path: '/inventory',
          Icon: <InventoryIcon className={iconClasses} />,
        },
      ]
    },
  ]
}];
```

### Layout Style

You can choose the style of the navigation by setting the `NEXT_PUBLIC_USER_NAVIGATION_STYLE` environment variable. The default style is `sidebar`.

```bash
NEXT_PUBLIC_USER_NAVIGATION_STYLE=sidebar
```

Alternatively, you can set the style to `header`:

```bash
NEXT_PUBLIC_TEAM_NAVIGATION_STYLE=header
```

## Old Configuration

If you're running an older version (e.g prior to 25 October 2024), you're running the older version which allowed flat routes:

 ```tsx {% title="apps/web/config/personal-account-navigation.config.tsx" %}
const routes = [
  {
    label: 'common:homeTabLabel',
    path: pathsConfig.app.home,
    Icon: <Home className={iconClasses} />,
    end: true,
  },
  {
    label: 'account:accountTabLabel',
    path: pathsConfig.app.personalAccountSettings,
    Icon: <User className={iconClasses} />,
  },
];

if (featureFlagsConfig.enablePersonalAccountBilling) {
  routes.push({
    label: 'common:billingTabLabel',
    path: pathsConfig.app.personalAccountBilling,
    Icon: <CreditCard className={iconClasses} />,
  });
}

export const personalAccountSidebarConfig = SidebarConfigSchema.parse({
  routes,
  style: process.env.NEXT_PUBLIC_USER_NAVIGATION_STYLE,
});
```

NB: please update to the latest version to use the latest features and optimizations.


-----------------
FILE PATH: kits\next-supabase-turbo\configuration\team-account-sidebar-configuration.mdoc

---
status: "published"
label: "Team Account Navigation Configuration"
title: "Setting your team account navigation configuration | Next.js Supabase SaaS Kit"
description: "Learn how to setup the team account navigation of your Next.js Supabase application"
order: 6
---

{% sequence title="Team Account Navigation Configuration" description="Learn how to setup the team account navigation of your Next.js Supabase application" %}

[Configuration](#configuration)

[Layout Style](#layout-style)

{% /sequence %}

The team account navigation is set at `apps/web/config/team-account-navigation.config.tsx`. We use this configuration to define the menu of the team account. By default, it has four routes: dashboard, settings, members, and billing (if enabled).

We define it in one place so we can build different views at once (for example, the mobile menu).

**Please update this file to add more routes to the sidebar.**

 ```tsx {% title="apps/web/config/team-account-navigation.config.tsx" %}
import {
  CreditCard,
  LayoutDashboard,
  MessageSquare,
  Settings,
  Users,
} from 'lucide-react';

import { NavigationConfigSchema } from '@kit/ui/navigation-schema';

import featureFlagsConfig from '~/config/feature-flags.config';
import pathsConfig from '~/config/paths.config';

const iconClasses = 'w-4';

const getRoutes = (account: string) => [
  {
    label: 'common:routes.application',
    children: [
      {
        label: 'common:routes.dashboard',
        path: pathsConfig.app.accountHome.replace('[account]', account),
        Icon: <LayoutDashboard className={iconClasses} />,
        end: true,
      },
      {
        label: 'common:routes.chat',
        path: pathsConfig.app.accountHome.replace('[account]', account) + '/chat',
        Icon: <MessageSquare className={iconClasses} />,
        end: false,
      },
    ],
  },
  {
    label: 'common:settingsTabLabel',
    collapsible: false,
    children: [
      {
        label: 'common:routes.settings',
        path: createPath(pathsConfig.app.accountSettings, account),
        Icon: <Settings className={iconClasses} />,
      },
      {
        label: 'common:routes.members',
        path: createPath(pathsConfig.app.accountMembers, account),
        Icon: <Users className={iconClasses} />,
      },
      featureFlagsConfig.enableTeamAccountBilling
        ? {
            label: 'common:routes.billing',
            path: createPath(pathsConfig.app.accountBilling, account),
            Icon: <CreditCard className={iconClasses} />,
          }
        : undefined,
    ].filter(Boolean),
  },
];

export function getTeamAccountSidebarConfig(account: string) {
  return NavigationConfigSchema.parse({
    routes: getRoutes(account),
    style: process.env.NEXT_PUBLIC_TEAM_NAVIGATION_STYLE,
    sidebarCollapsed: process.env.NEXT_PUBLIC_TEAM_SIDEBAR_COLLAPSED,
  });
}

function createPath(path: string, account: string) {
  return path.replace('[account]', account);
}
```

### Layout Style

You can choose the style of the navigation by setting the `NEXT_PUBLIC_TEAM_NAVIGATION_STYLE` environment variable. The default style is `sidebar`.

```bash
NEXT_PUBLIC_TEAM_NAVIGATION_STYLE=sidebar
```

Alternatively, you can set the style to `header`:

```bash
NEXT_PUBLIC_TEAM_NAVIGATION_STYLE=header
```

### Old Configuration

If you're running an older version (e.g prior to 25 October 2024), you're running the older version which allowed flat routes that looked like the below:

```tsx
const routes = [
  {
    label: 'common:dashboardTabLabel',
    path: pathsConfig.app.accountHome.replace('[account]', account),
    Icon: <LayoutDashboard className={iconClasses} />,
    end: true,
  }
];
```

NB: please update to the latest version to use the latest features and optimizations.

-----------------
FILE PATH: kits\next-supabase-turbo\configuration.mdoc

---
status: "published"
title: "Configuration in Next.js Supabase Turbo"
label: "Configuration"
order: 2
description: "Configuration in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Makerkit uses a configuration-first approach, making it easy to customize your application without diving into the code. The configuration layer acts as a bridge between your environment variables and the application, ensuring type-safety and validation.

### How Configuration Works in Makerkit

Configuration in Makerkit follows a simple pattern:

1. Environment variables define the settings
2. Configuration files validate and parse these variables
3. The application consumes the validated configuration

This approach ensures your application never starts with invalid configuration, preventing runtime errors that could affect your users.

## Key Configuration Areas

### 📱 Application Configuration
The core settings of your application like name, URL, and metadata. This configuration defines how your application presents itself to users and search engines.

[Learn more about Application Configuration →](configuration/application-configuration)

### 🔐 Authentication Configuration
Control which authentication methods are available to your users - from password-based auth to magic links and OAuth providers.

[Learn more about Authentication Configuration →](configuration/authentication-configuration)

### 🚩 Feature Flags Configuration
Toggle features on/off across your application. Perfect for rolling out new features or maintaining different versions of your product.

[Learn more about Feature Flags →](configuration/feature-flags-configuration)

### 🛣️ Paths Configuration
Define the routing structure of your application. This configuration helps maintain consistent navigation across your app.

[Learn more about Paths Configuration →](configuration/paths-configuration)

### 📱 Navigation Configuration
Customize the navigation structure for both personal and team accounts. Define what appears in your sidebars and headers.

- [Personal Account Navigation →](configuration/personal-account-sidebar-configuration)
- [Team Account Navigation →](configuration/team-account-sidebar-configuration)

## Environment Variables

All configuration ultimately stems from environment variables. These variables let you maintain different settings across environments without changing code.

[Learn more about Environment Variables →](configuration/environment-variables)

## Production Configuration

When deploying to production, you'll need to ensure all your configuration is properly set up. We've created a comprehensive guide to help you get everything right.

[Learn more about Production Configuration →](../going-to-production/checklist)

