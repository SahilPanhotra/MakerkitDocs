-----------------
FILE PATH: kits/react-router-supabase-turbo/components/stepper.mdoc

---
status: "published"

label: "Stepper"
title: "Stepper Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Stepper component in the React Router Supabase SaaS kit"
order: 1
---


The Stepper component is a versatile UI element designed to display a series of steps in a process or form. It provides visual feedback on the current step and supports different visual styles.

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
FILE PATH: kits/react-router-supabase-turbo/components.mdoc

---
status: "published"
title: "Components in React Router Supabase Turbo"
label: "Components"
order: 9
description: "Components in React Router Supabase Turbo"
collapsible: true
collapsed: true
---

Components in React Router Supabase Turbo


-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/application-configuration.mdoc

---
status: "published"
label: "Application Configuration"
title: "Setting your application configuration | React Router Supabase SaaS Kit"
description: "Learn how to setup the overall settings of your React Router Supabase application"
order: 1
---

The application configuration is set at `apps/web/config/app.config.ts`. This configuration stores some overall variables for your application.

This configuration is set at application-level. The configuration gets propagated to the packages that the app imports, so you can control the behavior and logic of the package. This also allows you to host multiple apps in the same monorepo, as every application defines its own configuration.

The recomendation is **to not update this directly** - instead, please define the environment variables below and override the default behavior. The configuration is validated using the Zod schema `AppConfigSchema`, so if something is off, you'll see the errors.

```tsx
const appConfig = AppConfigSchema.parse({
  name: import.meta.env.VITE_PRODUCT_NAME,
  title: import.meta.env.VITE_SITE_TITLE,
  description: import.meta.env.VITE_SITE_DESCRIPTION,
  url: import.meta.env.VITE_SITE_URL,
  locale: import.meta.env.VITE_DEFAULT_LOCALE,
  theme: import.meta.env.VITE_DEFAULT_THEME_MODE,
  themeColor: import.meta.env.VITE_THEME_COLOR,
  themeColorDark: import.meta.env.VITE_THEME_COLOR_DARK,
  production,
});
```

For example, to set the product name and app URL, you'd update the variables:

```bash
VITE_SITE_URL=https://myapp.com
VITE_PRODUCT_NAME="My wonderful AI App"
```

-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/authentication-configuration.mdoc

---
status: "published"
label: "Authentication Configuration"
title: "Setting your authentication configuration"
description: "Learn how to setup the authentication configuration of your React Router Supabase application"
order: 2
---

Makerkit supports three different authentication methods:

1. **Password** - the traditional email/password method, set to `true` by default
2. **Magic Link** - magic link, set to `false` by default
3. **oAuth** - oAuth providers, by default we set Google Auth

The authentication configuration is set at `apps/web/config/auth.config.ts`.

The recomendation is **to not update this directly** - instead, please define the environment variables below and override the default behavior. The configuration is validated using the Zod schema `AuthConfigSchema`, so if something is off, you'll see the errors.

```tsx
const authConfig = AuthConfigSchema.parse({
  // NB: This is a public key, so it's safe to expose.
  // Copy the value from the Supabase Dashboard.
  captchaTokenSiteKey: import.meta.env.VITE_CAPTCHA_SITE_KEY,

  // NB: Enable the providers below in the Supabase Console
  // in your production project
  providers: {
    password: import.meta.env.VITE_AUTH_PASSWORD === 'true',
    magicLink: import.meta.env.VITE_AUTH_MAGIC_LINK === 'true',
    oAuth: ['google'],
  },
} satisfies z.infer<typeof AuthConfigSchema>);
```

For example, if you wanted to switch from password auth to magic link, you'd set the below variables:

```bash
VITE_AUTH_PASSWORD=false
VITE_AUTH_MAGIC_LINK=true
```

## Password Requirements

To set the password requirements, you can set the following environment variables:

```bash
VITE_PASSWORD_REQUIRE_UPPERCASE=true
VITE_PASSWORD_REQUIRE_NUMBERS=true
VITE_PASSWORD_REQUIRE_SPECIAL_CHARS=true
```

The above will enforce the following rules:
1. At least one uppercase letter
2. At least one number
3. At least one special character

## MFA

To enforce MFA for RLS (Row Level Security) you need to customize the RLS policies as per [the Supabase documentation](https://supabase.com/blog/mfa-auth-via-rls).

-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/environment-variables.mdoc

---
status: "published"
title: Environment Variables in the React Router Supabase Starter Kit
label: Environment Variables
order: 0
description: Learn how to configure environment variables in the React Router Supabase Starter Kit.
---

Environment variables are defined in the `.env` file in the root of the `apps/web` package.

These environment variables are only used during development and are not used in production. In production, the environment variables are set in the CI/CD system.

The `.env` file is not committed to git - so you **could** place sensitive information in there for local development. Just be careful not to commit it to git.

## Public vs Private Environment Variables

I use the following convention:

1. **Public Environment Variables** are prefixed with `VITE_` and are available to the client-side code.
2. **Private Environment Variables** are not prefixed and are only available to the server-side code. Please make this the default. If you need to add a public environment variable, please add it to the `apps/web/lib/public-env.ts` file.

## Sample Environment Variables

Below is a sample `.env` file with the environment variables you need to set for the React Router Supabase Starter Kit.

```bash
# SITE
VITE_SITE_URL=http://localhost:5173
VITE_PRODUCT_NAME=Makerkit
VITE_SITE_TITLE="Makerkit - The easiest way to build and manage your SaaS"
VITE_SITE_DESCRIPTION="Makerkit is the easiest way to build and manage your SaaS. It provides you with the tools you need to build your SaaS, without the hassle of building it from scratch."
VITE_DEFAULT_THEME_MODE=light
VITE_THEME_COLOR="#ffffff"
VITE_THEME_COLOR_DARK="#0a0a0a"

# AUTH
VITE_AUTH_PASSWORD=true
VITE_AUTH_MAGIC_LINK=false
VITE_CAPTCHA_SITE_KEY=

# BILLING
VITE_BILLING_PROVIDER=stripe

# CMS
CMS_CLIENT=keystatic

# KEYSTATIC
VITE_KEYSTATIC_CONTENT_PATH=./content

# LOCALES PATH
VITE_LOCALES_PATH=apps/web/public/locales

# PATHS (to be used in "packages")
SIGN_IN_PATH=/auth/sign-in
SIGN_UP_PATH=/auth/sign-up
TEAM_ACCOUNTS_HOME_PATH=/home
INVITATION_PAGE_PATH=/join

# FEATURE FLAGS
VITE_ENABLE_THEME_TOGGLE=true
VITE_ENABLE_PERSONAL_ACCOUNT_DELETION=true
VITE_ENABLE_PERSONAL_ACCOUNT_BILLING=true
VITE_ENABLE_TEAM_ACCOUNTS_DELETION=true
VITE_ENABLE_TEAM_ACCOUNTS_BILLING=true
VITE_ENABLE_TEAM_ACCOUNTS=true
VITE_ENABLE_TEAM_ACCOUNTS_CREATION=true
```

#### Supabase

For Supabase, you'll need to add the following environment variables:

```bash
SUPABASE_SERVICE_ROLE_KEY=
```

#### Stripe

Please check the [Stripe documentation](billing/stripe).

#### Lemon Squeezy

Please check the [Lemon Squeezy documentation](lemon-squeezy).

#### Mailers

Please check the [Mailers documentation](email-configuration).

#### Monitoring

Please check the [Monitoring documentation](monitoring/overview).

#### CMS

Please check the [CMS documentation](content/cms).

### Feature Flags

To enable or disable certain application features, please toggle the values below:

```bash
VITE_ENABLE_THEME_TOGGLE=true
VITE_ENABLE_PERSONAL_ACCOUNT_DELETION=true
VITE_ENABLE_PERSONAL_ACCOUNT_BILLING=true
VITE_ENABLE_TEAM_ACCOUNTS_DELETION=true
VITE_ENABLE_TEAM_ACCOUNTS_BILLING=true
VITE_ENABLE_TEAM_ACCOUNTS=true
VITE_ENABLE_TEAM_ACCOUNTS_CREATION=true
```

The following feature flags are available:

1. **VITE_ENABLE_THEME_TOGGLE** - you can hide the theme toggle (if you want to force a single theme)
2. **VITE_ENABLE_PERSONAL_ACCOUNT_DELETION** - to prevent users from self-deleting their personal account
3. **VITE_ENABLE_PERSONAL_ACCOUNT_BILLING** - to enable/disable billing for personal accounts
4. **VITE_ENABLE_TEAM_ACCOUNTS_DELETION** - to prevent team accounts from self-deleting the account
5. **VITE_ENABLE_TEAM_ACCOUNTS_BILLING** - to enable/disable billing for team accounts
6. **VITE_ENABLE_TEAM_ACCOUNTS** - to disable team accounts
7. **VITE_ENABLE_TEAM_ACCOUNTS_CREATION** - to enable/disable the ability to create a team account

#### Personal Accounts vs Team Accounts

The starter kit supports two types of accounts:

1. Personal accounts are accounts that are owned by a single user.
2. Team accounts are accounts that group multiple users together.

This allows you to:

1. Serve B2C customers (personal accounts)
2. Serve B2B customers (team accounts)
3. Allow both (for example, like GitHub)

In the vast majority of cases, **you will want to enable billing for personal OR team accounts**. I have not seen many cases where billing both was required.

To do so, please set the following variables accordingly.

For enabling personal accounts billing only:

```bash
VITE_ENABLE_PERSONAL_ACCOUNT_BILLING=true
VITE_ENABLE_TEAM_ACCOUNTS_BILLING=false
```

You may also want to fully disable team accounts:

```bash
VITE_ENABLE_TEAM_ACCOUNTS=false
```

To enable team accounts billing only:

```bash
VITE_ENABLE_PERSONAL_ACCOUNT_BILLING=false
VITE_ENABLE_TEAM_ACCOUNTS_BILLING=true
```

To enable both, leave them both as `true`.

Please remember that for ensuring DB consistency, you need to also set them at DB level by adjusting the table `config`:

```sql
create table if not exists public.config(
    enable_team_accounts boolean default true not null,
    enable_account_billing boolean default true not null,
    enable_team_account_billing boolean default true not null,
    billing_provider public.billing_provider default 'stripe' not null
);
```

To enable personal account billing:

```sql
update config set enable_account_billing = true;
```

To enable team account billing:

```sql
update config set enable_team_account_billing = true;
```

To disable team accounts:

```sql
update config set enable_team_accounts = false;
```

To leave them both enabled, just leave them as they are.

## Contact Form submissions

To receive submissions from the contact form, you need to set up the following environment variables:

```bash
CONTACT_EMAIL=
```

This email will receive the submissions from the contact form.


-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/feature-flags-configuration.mdoc

---
status: "published"
label: "Feature Flags"
title: "Setting your feature flags configuration | React Router Supabase SaaS Kit"
description: "Learn how to setup the feature flags configuration of your React Router Supabase application"
order: 4
---

The features flags configuration is set at `apps/web/config/feature-flags.config.ts`. We use this configuration to store feature flags, i.e. to enable or disable certain features in the app.

This configuration is set at application level. The configuration gets propagated to the packages that the app imports, so you can control the behavior and logic of the package. This also allows you to host multiple apps in the same monorepo, as every application defines its own configuration.

The recommendation is **to not update this directly** - instead, please define the environment variables below and override the default behavior. The configuration is validated using the Zod schema `FeatureFlagsSchema`, so if something is off, you'll see the errors.

```tsx
const featuresFlagConfig = FeatureFlagsSchema.parse({
  enableThemeToggle: getBoolean(
    import.meta.env.VITE_ENABLE_THEME_TOGGLE,
    true,
  ),
  enableAccountDeletion: getBoolean(
    import.meta.env.VITE_ENABLE_PERSONAL_ACCOUNT_DELETION,
    false,
  ),
  enableTeamDeletion: getBoolean(
    import.meta.env.VITE_ENABLE_TEAM_ACCOUNTS_DELETION,
    false,
  ),
  enableTeamAccounts: getBoolean(
    import.meta.env.VITE_ENABLE_TEAM_ACCOUNTS,
    true,
  ),
  enableTeamCreation: getBoolean(
    import.meta.env.VITE_ENABLE_TEAM_ACCOUNTS_CREATION,
    true,
  ),
  enablePersonalAccountBilling: getBoolean(
    import.meta.env.VITE_ENABLE_PERSONAL_ACCOUNT_BILLING,
    false,
  ),
  enableTeamAccountBilling: getBoolean(
    import.meta.env.VITE_ENABLE_TEAM_ACCOUNTS_BILLING,
    false,
  ),
  languagePriority: import.meta.env.VITE_LANGUAGE_PRIORITY as
    | 'user'
    | 'application',
} satisfies z.infer<typeof FeatureFlagsSchema>);

export default featuresFlagConfig;
```

You can update the following environment variables to override the default behavior:

```bash
VITE_ENABLE_THEME_TOGGLE=
VITE_ENABLE_PERSONAL_ACCOUNT_DELETION=
VITE_ENABLE_TEAM_ACCOUNTS=
VITE_ENABLE_TEAM_ACCOUNTS_DELETION=
VITE_ENABLE_TEAM_ACCOUNTS_CREATION=
VITE_ENABLE_PERSONAL_ACCOUNT_BILLING=
VITE_ENABLE_TEAM_ACCOUNTS_BILLING=
VITE_LANGUAGE_PRIORITY=
```

### Explanation

1. **VITE_ENABLE_THEME_TOGGLE**: use this feature if you want to set a specific theme mode (dark or light) and disallow switching to another mode
2. **VITE_ENABLE_PERSONAL_ACCOUNT_DELETION**: use this feature flag if you don't want users to self-delete their accounts
3. **VITE_ENABLE_TEAM_ACCOUNTS**: use this feature flag to enable or disable team accounts. For B2C apps, disabling may be your preference.
4. **VITE_ENABLE_TEAM_ACCOUNTS_DELETION**: use this feature flag if you don't want users to self-delete their team accounts
5. **VITE_ENABLE_TEAM_ACCOUNTS_CREATION**: use this feature flag to prevent users from creating a team account. This can be useful if you wish to manage the onboarding yourself.
6. **VITE_ENABLE_PERSONAL_ACCOUNT_BILLING**: use this feature to enable or disable the ability of personal accounts to subscribe to a plan.
7. **VITE_ENABLE_TEAM_ACCOUNTS_BILLING**: use this feature to enable or disable the ability of team accounts to subscribe to a plan.
8. **VITE_LANGUAGE_PRIORITY**: use this feature to set the language priority. If set to `user`, the user's preferred language will be used. If set to `application`, the application's default language will be used.

### Note

It is unlikely that both `VITE_ENABLE_PERSONAL_ACCOUNT_BILLING` and `VITE_ENABLE_TEAM_ACCOUNTS_BILLING` would both be enabled at once. In most cases, you want to enable it for users or team accounts.

-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/paths-configuration.mdoc

---
status: "published"
label: "Paths Configuration"
title: "Setting your paths configuration"
description: "Learn how to setup the paths configuration of your React Router Supabase application"
order: 3
---

The paths configuration is set at `apps/web/config/paths.config.ts`. This configuration stores all the paths that you'll be using in your application. It is a convenient way to store them in a central place rather than scatter them in the codebase using magic strings.

The configuration is validated using the Zod schema `PathsSchema`, so if something is off, you'll see the errors.

It **is unlikely you'll need to change this** unless you're heavily editing the codebase.

```tsx
import { z } from 'zod';

const PathsSchema = z.object({
  auth: z.object({
    signIn: z.string().min(1),
    signUp: z.string().min(1),
    verifyMfa: z.string().min(1),
    callback: z.string().min(1),
    passwordReset: z.string().min(1),
    passwordUpdate: z.string().min(1),
  }),
  app: z.object({
    home: z.string().min(1),
    personalAccountSettings: z.string().min(1),
    personalAccountBilling: z.string().min(1),
    personalAccountBillingReturn: z.string().min(1),
    accountHome: z.string().min(1),
    accountSettings: z.string().min(1),
    accountBilling: z.string().min(1),
    accountMembers: z.string().min(1),
    accountBillingReturn: z.string().min(1),
    joinTeam: z.string().min(1),
  }),
});

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

export default pathsConfig;

```

-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/personal-account-sidebar-configuration.mdoc

---
status: "published"
label: "Account Navigation Configuration"
title: "Setting the personal account navigation configuration | React Router Supabase SaaS Kit"
description: "Learn how to setup the personal account navigation of your React Router Supabase application"
order: 5
---

The personal account navigation is set at `apps/web/config/personal-account-navigation.config.tsx`. We use this configuration to define the navigation menu of the personal account. By default, it has three routes: home page, settings, and billing (if enabled).

We define it in one place so we can build different views at once (for example, the mobile menu).

**Please update this file to add more routes to the sidebar.**

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
  style: import.meta.env.VITE_USER_NAVIGATION_STYLE,
});
```

You can choose the style of the navigation by setting the `VITE_USER_NAVIGATION_STYLE` environment variable. The default style is `sidebar`.

```bash
VITE_USER_NAVIGATION_STYLE=sidebar
```

Alternatively, you can set the style to `header`:

```bash
VITE_TEAM_NAVIGATION_STYLE=header
```



-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration/team-account-sidebar-configuration.mdoc

---
status: "published"
label: "Team Account Navigation Configuration"
title: "Setting your team account navigation configuration | React Router Supabase SaaS Kit"
description: "Learn how to setup the team account navigation of your React Router Supabase application"
order: 6
---

The team account navigation is set at `apps/web/config/team-account-navigation.config.tsx`. We use this configuration to define the menu of the team account. By default, it has four routes: dashboard, settings, members, and billing (if enabled).

We define it in one place so we can build different views at once (for example, the mobile menu).

**Please update this file to add more routes to the sidebar.**

 ```tsx {% title="apps/web/config/team-account-navigation.config.tsx" %}
const getRoutes = (account: string) => [
  {
    label: 'common:dashboardTabLabel',
    path: pathsConfig.app.accountHome.replace('[account]', account),
    Icon: <LayoutDashboard className={iconClasses} />,
    end: true,
  },
  {
    label: 'common:settingsTabLabel',
    collapsible: false,
    children: [
      {
        label: 'common:settingsTabLabel',
        path: createPath(pathsConfig.app.accountSettings, account),
        Icon: <Settings className={iconClasses} />,
      },
      {
        label: 'common:accountMembers',
        path: createPath(pathsConfig.app.accountMembers, account),
        Icon: <Users className={iconClasses} />,
      },
      featureFlagsConfig.enableTeamAccountBilling
        ? {
            label: 'common:billingTabLabel',
            path: createPath(pathsConfig.app.accountBilling, account),
            Icon: <CreditCard className={iconClasses} />,
          }
        : undefined,
    ].filter(Boolean),
  },
];

export function getTeamAccountSidebarConfig(account: string) {
  return SidebarConfigSchema.parse({
    routes: getRoutes(account),
    style: import.meta.env.VITE_TEAM_NAVIGATION_STYLE,
  });
}

function createPath(path: string, account: string) {
  return path.replace('[account]', account);
}
```

You can choose the style of the navigation by setting the `VITE_TEAM_NAVIGATION_STYLE` environment variable. The default style is `sidebar`.

```bash
VITE_TEAM_NAVIGATION_STYLE=sidebar
```

Alternatively, you can set the style to `header`:

```bash
VITE_TEAM_NAVIGATION_STYLE=header
```

-----------------
FILE PATH: kits/react-router-supabase-turbo/configuration.mdoc

---
status: "published"
title: "Configuration in React Router Supabase Turbo"
label: "Configuration"
order: 2
description: "Configuration in React Router Supabase Turbo"
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

[Learn more about Production Configuration →](going-to-production/checklist)

-----------------
FILE PATH: kits/react-router-supabase-turbo/content/cms-api.mdoc

---
status: "published"
label: "CMS API"
title: "Introducing the CMS API that allows you to pull content from your CMS | React Router Supabase SaaS Kit"
description: "Introducing the CMS API that allows you to pull content from your CMS"
order: 3
---

To fetch content from your CMS, you can use the CMS API. The CMS API is an interface that allows you to pull content from your CMS and display it on your website. This is useful if you want to display dynamic content on your website that is stored in your CMS.

### Creating a CMS client

To create a CMS client, you can use the `createCmsClient` function. This function returns a client that you can use to fetch content from your CMS.

```tsx
import { createCmsClient } from '@kit/cms';

const client = await createCmsClient();
```

The implementation depends on the CMS you are using. By default, the CMS client uses the `keystatic` CMS. If you are using a different CMS, you can specify the CMS client to use by setting the `CMS_CLIENT` environment variable.

```
CMS_CLIENT=keystatic
```

### Fetching content items

To fetch content items from your CMS, you can use the `getContentItems` function.

```tsx
// import the createCmsClient function
import { createCmsClient } from '@kit/cms';

// create a client
const client = await createCmsClient();

// Fetch content items
const { items, count } = await client.getContentItems({
  collection: 'posts',
});
```

The `getContentItems` function takes an object with the following properties:
1. `collection` - The collection to fetch content items from.
2. `limit` - The number of content items to fetch.
3. `offset` - The offset to start fetching content items from.
4. `language` - The language to fetch content items in.
5. `sortBy` - The field to sort content items by.
6. `sortDirection` - The direction to sort content items in.

```tsx
import { createCmsClient } from '@kit/cms';

const getContentItems = cache(
  async (language: string | undefined, limit: number, offset: number) => {
    const client = await createCmsClient();

    return client.getContentItems({
      collection: 'posts',
      limit,
      offset,
      language,
      sortBy: 'publishedAt',
      sortDirection: 'desc',
    });
  },
);
```

NB: how these values are used is entirely dependent on the CMS you are using. For example, Wordpress will only support `posts` or `pages` as collections.

Additionally: how the language filtering is implemented is also dependent on the CMS you are using.

### Fetching a single content item

To fetch a single content item from your CMS, you can use the `getContentItemBySlug` function.

```tsx
import { createCmsClient } from '@kit/cms';

const client = await createCmsClient();

// Fetch a single content item
const item = await client.getContentItemBySlug({
  slug: 'hello-world',
  collection: 'posts'
});
```

The `getContentItemBySlug` function takes an object with the following properties:

1. `slug` - The slug of the content item to fetch.
2. `collection` - The collection to fetch the content item from.

#### Rendering pages using content from your CMS

You can use the `getContentItemBySlug` function to fetch content from your CMS and render pages using that content.

For example, if you want to store your Terms and Conditions in your CMS, you can fetch the content item for the Terms and Conditions page and render the page using that content.

{% alert type="info" title="Create the 'pages' collection in your CMS" %}
This example assumes that you have created the 'pages' collection in your CMS. By default, the kit includes the `posts` and `documents` collections.
{% alert /%}

```tsx
import { createCmsClient } from '@kit/cms';

async function TermsAndConditionsPage() {
  const client = await createCmsClient();

  const { content, title } = await client.getContentItemBySlug({
    slug: 'terms-and-conditions',
    collection: 'pages',
  });

  return (
    <div>
      <h1>{title}</h1>
      <div>{content}</div>
    </div>
  );
}
```

-----------------
FILE PATH: kits/react-router-supabase-turbo/content/cms.mdoc

---
status: "published"
title: "Using the CMS interface in Makerkit | React Router Supabase SaaS Kit"
label: "CMS"
description: "The CMS library in Makerkit abstracts the implementation from where you store your data. It provides a simple API to interact with your data, and it's easy to extend and customize."
order: 0
---

Makerkit implements a CMS interface that abstracts the implementation from where you store your data. It provides a simple API to interact with your data, and it's easy to extend and customize.

By default, Makerkit ships with two CMS implementations:

1. [Keystatic](https://keystatic.com) - a CMS that stores data in a JSON file or on your Github repository
2. [Wordpress](https://wordpress.org) - needs no introduction

You can also create your own CMS implementation by extending the `CMS` class.

By default, we use Keystatic using `local` mode (eg. storing data in a JSON file). You can change the mode to `github` to store data in your Github repository or `cloud` to use their cloud service.

Local mode is the easiest way to get started since you need no setup. However, assuming you want to use edge-rendering, you'll need to switch to a remote mode (eg. Github) or a remote-only CMS (eg. Wordpress).


-----------------
FILE PATH: kits/react-router-supabase-turbo/content/creating-your-own-cms-client.mdoc

---
status: "published"
label: "Creating your own CMS client | React Router Supabase SaaS Kit"
title: "Creating your own CMS client with the CMS API"
description: "Learn how to create your own CMS client using the CMS API in Makerkit (React Router Supabase SaaS Kit) to fetch content from your CMS"
order: 4
---

Makerkit may not be able to accomodate everyone's preferences when it comes to CMSs. After all - there are too many to support! But that's okay - you can create your own CMS client using the CMS API in Makerkit.

To create your own CMS API, you need to implement the following methods:

```tsx
export abstract class CmsClient {
  /**
   * Retrieves content items based on the provided options.
   * @param options - Options for filtering and pagination.
   * @returns A promise that resolves to an array of content items.
   */
  abstract getContentItems(options?: Cms.GetContentItemsOptions): Promise<{
    total: number;
    items: Cms.ContentItem[];
  }>;

  /**
   * Retrieves a content item by its ID and type.
   * @returns A promise that resolves to the content item, or undefined if not found.
   */
  abstract getContentItemBySlug(params: {
    slug: string;
    collection: string;
  }): Promise<Cms.ContentItem | undefined>;

  /**
   * Retrieves categories based on the provided options.
   * @param options - Options for filtering and pagination.
   * @returns A promise that resolves to an array of categories.
   */
  abstract getCategories(
    options?: Cms.GetCategoriesOptions,
  ): Promise<Cms.Category[]>;

  /**
   * Retrieves a category by its slug.
   * @param slug - The slug of the category.
   * @returns A promise that resolves to the category, or undefined if not found.
   */
  abstract getCategoryBySlug(slug: string): Promise<Cms.Category | undefined>;

  /**
   * Retrieves tags based on the provided options.
   * @param options - Options for filtering and pagination.
   * @returns A promise that resolves to an array of tags.
   */
  abstract getTags(options?: Cms.GetTagsOptions): Promise<Cms.Tag[]>;

  /**
   * Retrieves a tag by its slug.
   * @param slug - The slug of the tag.
   * @returns A promise that resolves to the tag, or undefined if not found.
   */
  abstract getTagBySlug(slug: string): Promise<Cms.Tag | undefined>;
}
```

For a detailed view of the CMS interface, please refer to the API at `packages/cms/core/src/cms-client.ts`.

For example, let' assume you have an HTTP API to a private repository you can access at `https://my-cms-api.com`. You can create a CMS client that interacts with this API as follows:

```tsx
import { CmsClient } from '@kit/cms';

export class MyCmsClient extends CmsClient {
  async getContentItems(options) {
    const response = await fetch('https://my-cms-api.com/content', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(options),
    });

    const { total, items } = await response.json();

    return { total, items };
  }

  async getContentItemBySlug({ slug, collection }) {
    const response = await fetch(
      `https://my-cms-api.com/content/${collection}/${slug}`,
    );

    if (response.status === 404) {
      return undefined;
    }

    return response.json();
  }

  async getCategories(options) {
    const response = await fetch('https://my-cms-api.com/categories', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(options),
    });

    return response.json();
  }

  async getCategoryBySlug(slug) {
    const response = await fetch(`https://my-cms-api.com/categories/${slug}`);

    if (response.status === 404) {
      return undefined;
    }

    return response.json();
  }

  async getTags(options) {
    const response = await fetch('https://my-cms-api.com/tags', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(options),
    });

    return response.json();
  }

  async getTagBySlug(slug) {
    const response = await fetch(`https://my-cms-api.com/tags/${slug}`);

    if (response.status === 404) {
      return undefined;
    }

    return response.json();
  }
}
```

Of course, you can do the same using SDKs of your preferred CMS client, such as Payload CMS, Strapi, Sanity, and so on.


