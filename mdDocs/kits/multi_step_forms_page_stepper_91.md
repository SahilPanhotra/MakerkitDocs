-----------------
FILE PATH: kits\remix-supabase-turbo\components\multi-step-forms.mdoc

---
status: "published"

label: "Multi Step Forms"
title: "Multi Step Forms in the Remix Supabase SaaS kit"
description: "Building multi-step forms in the Remix Supabase SaaS kit"
order: 0
---


The Multi-Step Form Component is a powerful and flexible wrapper around React Hook Form, Zod, and Shadcn UI. It provides a simple API to create multi-step forms with ease, perfect for complex registration processes, surveys, or any scenario where you need to break down a long form into manageable steps.

## Features

- Easy integration with React Hook Form and Zod for form management and validation
- Built-in step management
- Customizable layout and styling
- Progress tracking with optional Stepper component
- TypeScript support for type-safe form schemas

## Usage

Here's a basic example of how to use the Multi-Step Form Component:

```tsx
import { MultiStepForm, MultiStepFormStep } from '@kit/ui/multi-step-form';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const FormSchema = createStepSchema({
  step1: z.object({ /* ... */ }),
  step2: z.object({ /* ... */ }),
});

export function MyForm() {
  const form = useForm({
    resolver: zodResolver(FormSchema),
    // ...
  });

  const onSubmit = (data) => {
    // Handle form submission
  };

  return (
    <MultiStepForm schema={FormSchema} form={form} onSubmit={onSubmit}>
      <MultiStepFormStep name="step1">
        {/* Step 1 fields */}
      </MultiStepFormStep>
      <MultiStepFormStep name="step2">
        {/* Step 2 fields */}
      </MultiStepFormStep>
    </MultiStepForm>
  );
}
```

## Key Components

### MultiStepForm

The main wrapper component that manages the form state and step progression.

Props:
- `schema`: Zod schema for form validation
- `form`: React Hook Form's `useForm` instance
- `onSubmit`: Function to handle form submission
- `className`: Optional CSS classes

### MultiStepFormStep

Represents an individual step in the form.

Props:
- `name`: Unique identifier for the step (should match a key in your schema)
- `children`: Step content

### MultiStepFormHeader

Optional component for adding a header to your form, often used with the Stepper component.

### MultiStepFormContextProvider

Provides access to form context within child components.

### useMultiStepFormContext

The hook returns an object with the following properties:

- `form: UseFormReturn<z.infer<Schema>>` - The original form object.
- `currentStep: string` - The name of the current step.
- `currentStepIndex: number` - The index of the current step (0-based).
- `totalSteps: number` - The total number of steps in the form.
- `isFirstStep: boolean` - Whether the current step is the first step.
- `isLastStep: boolean` - Whether the current step is the last step.
- `nextStep: (e: React.SyntheticEvent) => void` - Function to move to the next step.
- `prevStep: (e: React.SyntheticEvent) => void` - Function to move to the previous step.
- `goToStep: (index: number) => void` - Function to jump to a specific step by index.
- `direction: 'forward' | 'backward' | undefined` - The direction of the last step change.
- `isStepValid: () => boolean` - Function to check if the current step is valid.
- `isValid: boolean` - Whether the entire form is valid.
- `errors: FieldErrors<z.infer<Schema>>` - Form errors from React Hook Form.
- `mutation: UseMutationResult` - A mutation object for handling form submission.

## Example

Here's a more complete example of a multi-step form with three steps: Account, Profile, and Review. The form uses Zod for schema validation and React Hook Form for form management.

```tsx
'use client';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

import { Button } from '@kit/ui/button';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@kit/ui/form';
import { Input } from '@kit/ui/input';
import {
  MultiStepForm,
  MultiStepFormContextProvider,
  MultiStepFormHeader,
  MultiStepFormStep,
  createStepSchema,
  useMultiStepFormContext,
} from '@kit/ui/multi-step-form';
import { Stepper } from '@kit/ui/stepper';

const FormSchema = createStepSchema({
  account: z.object({
    username: z.string().min(3),
    email: z.string().email(),
  }),
  profile: z.object({
    password: z.string().min(8),
    age: z.coerce.number().min(18),
  }),
});

type FormValues = z.infer<typeof FormSchema>;

export function MultiStepFormDemo() {
  const form = useForm<FormValues>({
    resolver: zodResolver(FormSchema),
    defaultValues: {
      account: {
        username: '',
        email: '',
      },
      profile: {
        password: '',
      },
    },
    reValidateMode: 'onBlur',
    mode: 'onBlur',
  });

  const onSubmit = (data: FormValues) => {
    console.log('Form submitted:', data);
  };

  return (
    <MultiStepForm
      className={'space-y-10 p-8 rounded-xl border'}
      schema={FormSchema}
      form={form}
      onSubmit={onSubmit}
    >
      <MultiStepFormHeader
        className={'flex w-full flex-col justify-center space-y-6'}
      >
        <h2 className={'text-xl font-bold'}>Create your account</h2>

        <MultiStepFormContextProvider>
          {({ currentStepIndex }) => (
            <Stepper
              variant={'numbers'}
              steps={['Account', 'Profile', 'Review']}
              currentStep={currentStepIndex}
            />
          )}
        </MultiStepFormContextProvider>
      </MultiStepFormHeader>

      <MultiStepFormStep name="account">
        <AccountStep />
      </MultiStepFormStep>

      <MultiStepFormStep name="profile">
        <ProfileStep />
      </MultiStepFormStep>

      <MultiStepFormStep name="review">
        <ReviewStep />
      </MultiStepFormStep>
    </MultiStepForm>
  );
}

function AccountStep() {
  const { form, nextStep, isStepValid } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex flex-col gap-4'}>
        <FormField
          name="account.username"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Username</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          name="account.email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <div className="flex justify-end">
          <Button onClick={nextStep} disabled={!isStepValid()}>
            Next
          </Button>
        </div>
      </div>
    </Form>
  );
}

function ProfileStep() {
  const { form, nextStep, prevStep } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex flex-col gap-4'}>
        <FormField
          name="profile.password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Password</FormLabel>
              <FormControl>
                <Input type="password" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          name="profile.age"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Age</FormLabel>
              <FormControl>
                <Input type="number" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <div className="flex justify-end space-x-2">
          <Button type={'button'} variant={'outline'} onClick={prevStep}>
            Previous
          </Button>

          <Button onClick={nextStep}>Next</Button>
        </div>
      </div>
    </Form>
  );
}

function ReviewStep() {
  const { prevStep, form } = useMultiStepFormContext<typeof FormSchema>();
  const values = form.getValues();

  return (
    <div className={'flex flex-col space-y-4'}>
      <div className={'flex flex-col space-y-4'}>
        <div>Great! Please review the values.</div>

        <div className={'flex flex-col space-y-2 text-sm'}>
          <div>
            <span>Username</span>: <span>{values.account.username}</span>
          </div>
          <div>
            <span>Email</span>: <span>{values.account.email}</span>
          </div>
          <div>
            <span>Age</span>: <span>{values.profile.age}</span>
          </div>
        </div>
      </div>

      <div className="flex justify-end space-x-2">
        <Button type={'button'} variant={'outline'} onClick={prevStep}>
          Back
        </Button>

        <Button type={'submit'}>Create Account</Button>
      </div>
    </div>
  );
}
```

The inner components `AccountStep`, `ProfileStep`, and `ReviewStep` represent the individual steps of the form. They use the `useMultiStepFormContext` hook to access form utilities like `nextStep`, `prevStep`, and `isStepValid`.

These are built using ShadcnUI - so please [do refer to the ShadcnUI documentation](https://ui.shadcn.com/docs/components/form) for more information on how to use the components.

## Tips

1. Use the `createStepSchema` helper to easily create Zod schemas for your multi-step form.
2. Leverage the `useMultiStepFormContext` hook in your step components to access form utilities.
3. Combine with the Stepper component for visual progress indication.
4. Customize the look and feel using the provided `className` props and your own CSS.

The Multi-Step Form Component simplifies the creation of complex, multi-step forms while providing a great user experience. It's flexible enough to handle a wide variety of use cases while keeping your code clean and maintainable.




-----------------
FILE PATH: kits\remix-supabase-turbo\components\page.mdoc

---
status: "published"

label: "Page"
title: "Page Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Page component in the Remix Supabase SaaS kit"
order: 5
---


The Page component is a versatile layout component that provides different page structures based on the specified style. It's designed to create consistent layouts across your application with support for sidebar and header-based designs.

## Usage

```jsx
import { Page, PageNavigation, PageBody, PageHeader } from '@kit/ui/page';

function MyPage() {
  return (
    <Page style="sidebar">
      <PageNavigation>
        {/* Navigation content */}
      </PageNavigation>
      <PageHeader title="Dashboard" description="Welcome to your dashboard">
        {/* Optional header content */}
      </PageHeader>
      <PageBody>
        {/* Main page content */}
      </PageBody>
    </Page>
  );
}
```

## Page Component Props

- `style?: 'sidebar' | 'header' | 'custom'`: Determines the layout style (default: 'sidebar')
- `contentContainerClassName?: string`: Custom class for the content container
- `className?: string`: Additional classes for the main container
- `sticky?: boolean`: Whether to make the header sticky (for 'header' style)

## Sub-components

### PageNavigation

Wraps the navigation content, typically used within the Page component.

### PageMobileNavigation

Wraps the mobile navigation content, displayed only on smaller screens.

### PageBody

Contains the main content of the page.

Props:
- `className?: string`: Additional classes for the body container

### PageHeader

Displays the page title and description.

Props:
- `title?: string | React.ReactNode`: The page title
- `description?: string | React.ReactNode`: The page description
- `className?: string`: Additional classes for the header container

### PageTitle

Renders the main title of the page.

### PageDescription

Renders the description text below the page title.

## Layout Styles

### Sidebar Layout

The default layout, featuring a sidebar navigation and main content area.

### Header Layout

A layout with a top navigation bar and content below.

### Custom Layout

Allows for complete custom layouts by directly rendering children.

## Examples

### Sidebar Layout

```jsx
<Page style="sidebar">
  <PageNavigation>
    <SidebarContent />
  </PageNavigation>
  <PageHeader title="Dashboard" description="Overview of your account">
    <UserMenu />
  </PageHeader>
  <PageBody>
    <DashboardContent />
  </PageBody>
</Page>
```

### Header Layout

```jsx
<Page style="header" sticky={true}>
  <PageNavigation>
    <HeaderNavLinks />
  </PageNavigation>
  <PageMobileNavigation>
    <MobileMenu />
  </PageMobileNavigation>
  <PageBody>
    <PageHeader title="Profile" description="Manage your account settings" />
    <ProfileSettings />
  </PageBody>
</Page>
```

## Customization

The Page component and its sub-components use Tailwind CSS classes for styling. You can further customize the appearance by passing additional classes through the `className` props or by modifying the default classes in the component implementation.

## Best Practices

1. Use consistent layout styles across similar pages for a cohesive user experience.
2. Leverage the PageHeader component to provide clear page titles and descriptions.
3. Utilize the PageNavigation and PageMobileNavigation components to create responsive navigation experiences.
4. When using the 'custom' style, ensure you handle responsive behavior manually.

The Page component and its related components provide a flexible system for creating structured, responsive layouts in your React application, promoting consistency and ease of maintenance across your project.

-----------------
FILE PATH: kits\remix-supabase-turbo\components\stepper.mdoc

---
status: "published"

label: "Stepper"
title: "Stepper Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Stepper component in the Remix Supabase SaaS kit"
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
FILE PATH: kits\remix-supabase-turbo\components.mdoc

---
status: "published"
title: "Components in Remix Supabase Turbo"
label: "Components"
order: 9
description: "Components in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Components in Remix Supabase Turbo


-----------------
FILE PATH: kits\remix-supabase-turbo\configuration\application-configuration.mdoc

---
status: "published"
label: "Application Configuration"
title: "Setting your application configuration | Remix Supabase SaaS Kit"
description: "Learn how to setup the overall settings of your Remix Supabase application"
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
FILE PATH: kits\remix-supabase-turbo\configuration\authentication-configuration.mdoc

---
status: "published"

label: "Authentication Configuration"
title: "Setting your authentication configuration"
description: "Learn how to setup the authentication configuration of your Remix Supabase application"
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
FILE PATH: kits\remix-supabase-turbo\configuration\environment-variables.mdoc

---
status: "published"

title: Environment Variables in the Remix Supabase Starter Kit
label: Environment Variables
order: 0
description: Learn how to configure environment variables in the Remix Supabase Starter Kit.
---


Environment variables are defined in the `.env` file in the root of the `apps/web` package.

These environment variables are only used during development and are not used in production. In production, the environment variables are set in the CI/CD system.

The `.env` file is not committed to git - so you **could** place sensitive information in there for local development. Just be careful not to commit it to git.

## Public vs Private Environment Variables

I use the following convention:

1. **Public Environment Variables** are prefixed with `VITE_` and are available to the client-side code. They are added to the `window` object when added to `apps/web/lib/public-env.ts`. This is copied from Next.js - with the difference we add them to the client side manually. I like the convention.
2. **Private Environment Variables** are not prefixed and are only available to the server-side code. Please make this the default. If you need to add a public environment variable, please add it to the `apps/web/lib/public-env.ts` file.

## Adding public environment variables to the client-side

To add public environment variables to the client-side, you need to add them to the `apps/web/lib/public-env.ts` file.

```typescript
/**
 * @name PUBLIC_ENV
 * @description This is the public environment variables that are available to the client.
 * We use this to expose public environment variables to the client.
 */
export const PUBLIC_ENV = {
  // ... default public vars,

  // Add your public environment variables here ...
};
```

Please DO NOT add sensitive information to this file.

## Sample Environment Variables

Below is a sample `.env` file with the environment variables you need to set for the Remix Supabase Starter Kit.

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
FILE PATH: kits\remix-supabase-turbo\configuration\feature-flags-configuration.mdoc

---
status: "published"
label: "Feature Flags"
title: "Setting your feature flags configuration | Remix Supabase SaaS Kit"
description: "Learn how to setup the feature flags configuration of your Remix Supabase application"
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
FILE PATH: kits\remix-supabase-turbo\configuration\paths-configuration.mdoc

---
status: "published"

label: "Paths Configuration"
title: "Setting your paths configuration"
description: "Learn how to setup the paths configuration of your Remix Supabase application"
order: 3
---


The paths configuration is set at `apps/web/config/paths.config.ts`. This configuration stores all the paths that you'll be using in your application. It is a convenient way to store them in a central place rather than scatter them in the codebase using magic strings.

The configuration is validated using the Zod schema `PathsSchema`, so if something is off, you'll see the errors.

It **is unlikely you'll need to change this** unless you're heavily editing the codebase.

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
FILE PATH: kits\remix-supabase-turbo\configuration\personal-account-sidebar-configuration.mdoc

---
status: "published"
label: "Account Navigation Configuration"
title: "Setting the personal account navigation configuration | Remix Supabase SaaS Kit"
description: "Learn how to setup the personal account navigation of your Remix Supabase application"
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



