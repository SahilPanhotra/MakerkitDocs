-----------------
FILE PATH: kits/next-supabase-turbo/components/data-table.mdoc

---
status: "published"

label: "Data Table"
title: "Data Table Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Data Table component in the Next.js Supabase SaaS kit"
order: 2
---


The DataTable component is a powerful and flexible table component built on top of TanStack Table (React Table v8). It provides a range of features for displaying and interacting with tabular data, including pagination, sorting, and custom rendering.

## Usage

```tsx
import { DataTable } from '@kit/ui/enhanced-data-table';

function MyComponent() {
  const columns = [
    // Define your columns here
  ];

  const data = [
    // Your data array
  ];

  return (
    <DataTable
      columns={columns}
      data={data}
      pageSize={10}
      pageIndex={0}
      pageCount={5}
    />
  );
}
```

## Props

- `data: T[]` (required): An array of objects representing the table data.
- `columns: ColumnDef<T>[]` (required): An array of column definitions.
- `pageIndex?: number`: The current page index (0-based).
- `pageSize?: number`: The number of rows per page.
- `pageCount?: number`: The total number of pages.
- `onPaginationChange?: (pagination: PaginationState) => void`: Callback function for pagination changes.
- `tableProps?: React.ComponentProps<typeof Table>`: Additional props to pass to the underlying Table component.

## Pagination

The DataTable component handles pagination internally but can also be controlled externally. It provides navigation buttons for first page, previous page, next page, and last page.

## Sorting

Sorting is handled internally by the component. Click on column headers to sort by that column.

## Filtering

The component supports column filtering, which can be implemented in the column definitions.

## Example with ServerDataLoader

Here's an example of how to use the DataTable component with ServerDataLoader:

```jsx
import { ServerDataLoader } from '@makerkit/data-loader-supabase-nextjs';
import { DataTable } from '@kit/ui/enhanced-data-table';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

function AccountsPage({ searchParams }) {
  const client = getSupabaseServerAdminClient();
  const page = searchParams.page ? parseInt(searchParams.page) : 1;
  const filters = getFilters(searchParams);

  return (
    <ServerDataLoader
      table={'accounts'}
      client={client}
      page={page}
      where={filters}
    >
      {({ data, page, pageSize, pageCount }) => (
        <DataTable
          columns={[
            // Define your columns here
          ]}
          data={data}
          page={page}
          pageSize={pageSize}
          pageCount={pageCount}
        />
      )}
    </ServerDataLoader>
  );
}
```

This example demonstrates how to use ServerDataLoader to fetch data from a Supabase table and pass it to the DataTable component. The ServerDataLoader handles the data fetching and pagination, while the DataTable takes care of rendering and client-side interactions.

## Customization

The DataTable component is built with customization in mind. You can customize the appearance using Tailwind CSS classes and extend its functionality by passing custom props to the underlying Table component.

## Internationalization

The component uses the `Trans` component for internationalization. Ensure you have your i18n setup correctly to leverage this feature.

The DataTable component provides a powerful and flexible solution for displaying tabular data in your React applications, with built-in support for common table features and easy integration with server-side data loading.

-----------------
FILE PATH: kits/next-supabase-turbo/components/empty-state.mdoc

---
status: "published"
label: "Empty State"
title: "Empty State Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Empty State component in the Next.js Supabase SaaS kit"
order: 7
---

The `EmptyState` component is a flexible and reusable UI element designed to display when there's no content to show. It's perfect for scenarios like empty lists, search results with no matches, or initial states of features.

{% component path="empty-state" /%}

## Components

1. `EmptyState`: The main wrapper component
2. `EmptyStateHeading`: For the main heading
3. `EmptyStateText`: For descriptive text
4. `EmptyStateButton`: For a call-to-action button

## Usage

```jsx
import { EmptyState, EmptyStateHeading, EmptyStateText, EmptyStateButton } from '@kit/ui/empty-state';

function MyComponent() {
  return (
    <EmptyState>
      <EmptyStateHeading>No results found</EmptyStateHeading>
      <EmptyStateText>Try adjusting your search or filter to find what you're looking for.</EmptyStateText>
      <EmptyStateButton>Clear filters</EmptyStateButton>
    </EmptyState>
  );
}
```

## Component Details

### EmptyState

The main container that wraps all other components.

- **Props**: Accepts all standard `div` props
- **Styling**:
- Flex container with centered content
- Rounded corners with a dashed border
- Light shadow for depth

### EmptyStateHeading

Used for the main heading of the empty state.

- **Props**: Accepts all standard `h3` props
- **Styling**:
- Large text (2xl)
- Bold font
- Tight letter spacing

### EmptyStateText

For descriptive text explaining the empty state or providing guidance.

- **Props**: Accepts all standard `p` props
- **Styling**:
- Small text
- Muted color for less emphasis

### EmptyStateButton

A button component for primary actions.

- **Props**: Accepts all props from the base `Button` component
- **Styling**:
- Margin top for spacing
- Inherits styles from the base `Button` component

## Features

1. **Flexible Structure**: Components can be used in any order, and additional custom elements can be added.
2. **Automatic Layout**: The component automatically arranges its children in a centered, vertical layout.
3. **Customizable**: Each subcomponent accepts className props for custom styling.
4. **Type-Safe**: Utilizes TypeScript for prop type checking.

## Customization

You can customize the appearance of each component by passing a `className` prop:

```tsx
<EmptyState className="bg-gray-100">
  <EmptyStateHeading className="text-primary">Custom Heading</EmptyStateHeading>
  <EmptyStateText className="text-lg">Larger descriptive text</EmptyStateText>
  <EmptyStateButton className="bg-secondary">Custom Button</EmptyStateButton>
</EmptyState>
```

This `EmptyState` component provides a clean, consistent way to handle empty states in your application. Its modular design allows for easy customization while maintaining a cohesive look and feel across different use cases.

-----------------
FILE PATH: kits/next-supabase-turbo/components/if.mdoc

---
status: "published"

label: "Conditional Rendering"
title: "Dynamic Conditional Rendering in the Next.js Supabase SaaS kit"
description: "Learn how to use the If component in the Next.js Supabase SaaS kit"
order: 4
---


The `If` component is a utility component for conditional rendering in React applications. It provides a clean, declarative way to render content based on a condition, with support for fallback content.

## Features

- Conditional rendering based on various types of conditions
- Support for render props pattern
- Optional fallback content
- Memoized for performance optimization

## Usage

```jsx
import { If } from '@kit/ui/if';

function MyComponent({ isLoggedIn, user }) {
  return (
    <If condition={isLoggedIn} fallback={<LoginPrompt />}>
      {(value) => <WelcomeMessage user={user} />}
    </If>
  );
}
```

## Props

The `If` component accepts the following props:

- `condition: Condition<Value>` (required): The condition to evaluate. Can be any value, where falsy values (`false`, `null`, `undefined`, `0`, `''`) are considered false.
- `children: React.ReactNode | ((value: Value) => React.ReactNode)` (required): The content to render when the condition is truthy. Can be a React node or a function (render prop).
- `fallback?: React.ReactNode` (optional): Content to render when the condition is falsy.

## Types

```typescript
type Condition<Value = unknown> = Value | false | null | undefined | 0 | '';
```

## Examples

### Basic usage

```jsx
<If condition={isLoading}>
  <LoadingSpinner />
</If>
```

### With fallback

```jsx
<If condition={hasData} fallback={<NoDataMessage />}>
  <DataDisplay data={data} />
</If>
```

### Using render props

```jsx
<If condition={user}>
  {(user) => <UserProfile username={user.name} />}
</If>
```

## Performance

The `If` component uses `useMemo` to optimize performance by memoizing the rendered output. This means it will only re-render when the `condition`, `children`, or `fallback` props change.

## Best Practices

1. Use the `If` component for simple conditional rendering to improve readability.
2. Leverage the render props pattern when you need to use the condition's value in the rendered content.
3. Provide a fallback for better user experience when the condition is false.
4. Remember that the condition is re-evaluated on every render, so keep it simple to avoid unnecessary computations.

## Typescript Support

The `If` component is fully typed - This allows for type-safe usage of the render props pattern:

```typescript
<If condition={user}>
  {(user) => <UserProfile name={user.name} email={user.email} />}
</If>
```

The `If` component provides a clean and efficient way to handle conditional rendering in React applications, improving code readability and maintainability.

-----------------
FILE PATH: kits/next-supabase-turbo/components/loading-overlay.mdoc

---
status: "published"

label: "Loading Overlay"
title: "Loading Overlay Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Loading Overlay component in the Next.js Supabase SaaS kit"
order: 3
---

The LoadingOverlay component is a versatile UI element designed to display a loading state with a spinner and optional content. It's perfect for indicating background processes or page loads in your application.

{% component path="overlays/loading-overlay" /%}

## Features

- Customizable appearance through CSS classes
- Option for full-page overlay or inline loading indicator
- Spinner animation with customizable styling
- Ability to include additional content or messages

## Usage

```jsx
import { LoadingOverlay } from '@kit/ui/loading-overlay';

function MyComponent() {
  return (
    <LoadingOverlay>
      Loading your content...
    </LoadingOverlay>
  );
}
```

## Props

The LoadingOverlay component accepts the following props:

- `children?: React.ReactNode`: Optional content to display below the spinner.
- `className?: string`: Additional CSS classes to apply to the container.
- `spinnerClassName?: string`: CSS classes to apply to the spinner component.
- `fullPage?: boolean`: Whether to display as a full-page overlay. Defaults to `true`.

## Examples

### Full-page overlay

```jsx
<LoadingOverlay>
  Please wait while we load your dashboard...
</LoadingOverlay>
```

### Inline loading indicator

```jsx
<LoadingOverlay fullPage={false} className="h-40">
  Fetching results...
</LoadingOverlay>
```

### Customized appearance

```jsx
<LoadingOverlay
  className="bg-gray-800 text-white"
  spinnerClassName="text-blue-500"
>
  Processing your request...
</LoadingOverlay>
```

## Styling

The LoadingOverlay uses Tailwind CSS for styling. Key classes include:

- Flex layout with centered content: `flex flex-col items-center justify-center`
- Space between spinner and content: `space-y-4`
- Full-page overlay (when `fullPage` is true):
```
fixed left-0 top-0 z-[100] h-screen w-screen bg-background
  ```

You can further customize the appearance by passing additional classes through the `className` and `spinnerClassName` props.

## Accessibility

When using the LoadingOverlay, consider adding appropriate ARIA attributes to improve accessibility, such as `aria-busy="true"` on the parent element that's in a loading state.

## Best Practices

1. Use full-page overlays sparingly to avoid disrupting user experience.
2. Provide clear, concise messages to inform users about what's loading.
3. Consider using inline loading indicators for smaller UI elements or partial page updates.
4. Ensure sufficient contrast between the overlay and the spinner for visibility.

The LoadingOverlay component provides a simple yet effective way to indicate loading states in your application, enhancing user experience by providing visual feedback during asynchronous operations or content loads.


-----------------
FILE PATH: kits/next-supabase-turbo/components/marketing-components.mdoc

---
status: "published"
label: "Marketing Components"
title: "Marketing Components in the Next.js Supabase SaaS kit"
description: "Learn how to use the Marketing components in the Next.js Supabase SaaS kit"
order: 8
---

Marketing components are designed to help you create beautiful and engaging marketing pages for your SaaS application. These components are built on top of the Shadcn UI library and are designed to work seamlessly with Next.js routing.

## Hero

The Hero component is a versatile and customizable landing page hero section for React applications.

{% component path="marketing/hero" /%}

### Import

```jsx
import { Hero } from '@kit/ui/marketing';
```

### Usage

```jsx
import { Hero, Pill, CtaButton } from '@kit/ui/marketing';
import Image from 'next/image';

function LandingPage() {
  return (
    <Hero
      pill={<Pill>New Feature</Pill>}
      title="Welcome to Our App"
      subtitle="Discover the power of our innovative solution"
      cta={<CtaButton>Get Started</CtaButton>}
      image={
        <Image
          src="/hero-image.jpg"
          alt="Hero Image"
          width={1200}
          height={600}
        />
      }
    />
  );
}
```

### Styling

The Hero component uses Tailwind CSS for styling. You can customize its appearance by:

1. Modifying the default classes in the component.
2. Passing additional classes via the `className` prop.
3. Overriding styles in your CSS using the appropriate selectors.

### Animations

By default, the Hero component applies entrance animations to its elements. You can disable these animations by setting the `animate` prop to `false`.

### Accessibility

The Hero component uses semantic HTML elements and follows accessibility best practices:

- The main title uses an `<h1>` tag (via the `HeroTitle` component).
- The subtitle uses an `<h3>` tag for proper heading hierarchy.

Ensure that any images passed via the `image` prop include appropriate `alt` text for screen readers.

### Notes

- The Hero component is designed to be flexible and can accommodate various content types through its props.
- For optimal performance, consider lazy-loading large images passed to the `image` prop.
- The component is responsive and adjusts its layout for different screen sizes.

### A Larger example straight from the kit

Below is a larger example of a Hero component with additional elements like a pill, CTA button, and image:

```tsx
import { Hero, Pill, CtaButton, GradientSecondaryText } from '@kit/ui/marketing';
import { Trans } from '@kit/ui/trans';
import { LayoutDashboard } from 'lucide-react';
import Image from 'next/image';

<Hero
  pill={
    <Pill label={'New'}>
      <span>The leading SaaS Starter Kit for ambitious developers</span>
    </Pill>
  }
  title={
    <>
      <span>The ultimate SaaS Starter</span>
      <span>for your next project</span>
    </>
  }
  subtitle={
    <span>
      Build and Ship a SaaS faster than ever before with the next-gen SaaS
      Starter Kit. Ship your SaaS in days, not months.
    </span>
  }
  cta={<MainCallToActionButton />}
  image={
    <Image
      priority
      className={
        'delay-250 rounded-2xl border border-gray-200 duration-1000 ease-out animate-in fade-in zoom-in-50 fill-mode-both dark:border-primary/10'
      }
      width={3558}
      height={2222}
      src={`/images/dashboard.webp`}
      alt={`App Image`}
    />
  }
/>

function MainCallToActionButton() {
  return (
    <div className={'flex space-x-4'}>
      <CtaButton>
        <Link href={'/auth/sign-up'}>
          <span className={'flex items-center space-x-0.5'}>
            <span>
              <Trans i18nKey={'common:getStarted'} />
            </span>

            <ArrowRightIcon
              className={
                'h-4 animate-in fade-in slide-in-from-left-8' +
                ' delay-1000 duration-1000 zoom-in fill-mode-both'
              }
            />
          </span>
        </Link>
      </CtaButton>

      <CtaButton variant={'link'}>
        <Link href={'/contact'}>
          <Trans i18nKey={'common:contactUs'} />
        </Link>
      </CtaButton>
    </div>
  );
}
```

## HeroTitle

The `HeroTitle` component is a specialized heading component used within the Hero component to display the main title.

{% component path="marketing/hero-title" /%}

### Props

The `HeroTitle` component accepts the following props:

1. `asChild?: boolean`: Whether to render the component as a child of the `Slot` component.
2. `HTMLAttributes<HTMLHeadingElement>`: Additional attributes to apply to the heading element.

### Usage

```tsx
import { HeroTitle } from '@kit/ui/marketing';

function LandingPage() {
  return (
    <HeroTitle asChild>
      Welcome to Our App
    </HeroTitle>
  );
}
```

## Pill

The `Pill` component is a small, rounded content container often used for highlighting or categorizing information.

{% component path="marketing/pill" /%}

### Usage

Use the `Pill` component to create a small, rounded content container with optional label text.

```tsx
import { Pill } from '@kit/ui/marketing';

function LandingPage() {
  return (
    <Pill label="New">
      Discover the power of our innovative
    </Pill>
  );
}
```

## Features

The `FeatureShowcase`, `FeatureShowcaseIconContainer`, `FeatureGrid`, and `FeatureCard` components are designed to showcase product features on marketing pages.

### FeatureShowcase

The `FeatureShowcase` component is a layout component that showcases a feature with an icon, heading, and description.

### FeatureShowcaseIconContainer

The `FeatureShowcaseIconContainer` component is a layout component that contains an icon for the `FeatureShowcase` component.

### FeatureGrid

The `FeatureGrid` component is a layout component that arranges `FeatureCard` components in a grid layout.

### FeatureCard

The `FeatureCard` component is a card component that displays a feature with a label, description, and optional image.

### Usage

Use the `FeatureShowcase` component to showcase a feature with an icon, heading, and description.

```tsx
 <div className={'container mx-auto'}>
  <div
    className={'flex flex-col space-y-16 xl:space-y-32 2xl:space-y-36'}
  >
    <FeatureShowcase
      heading={
        <>
          <b className="font-semibold dark:text-white">
            The ultimate SaaS Starter Kit
          </b>
          .{' '}
          <GradientSecondaryText>
            Unleash your creativity and build your SaaS faster than ever
            with Makerkit.
          </GradientSecondaryText>
        </>
      }
      icon={
        <FeatureShowcaseIconContainer>
          <LayoutDashboard className="h-5" />
          <span>All-in-one solution</span>
        </FeatureShowcaseIconContainer>
      }
    >
      <FeatureGrid>
        <FeatureCard
          className={
            'relative col-span-2 overflow-hidden bg-violet-500 text-white lg:h-96'
          }
          label={'Beautiful Dashboard'}
          description={`Makerkit provides a beautiful dashboard to manage your SaaS business.`}
        >
          <Image
            className="absolute right-0 top-0 hidden h-full w-full rounded-tl-2xl border border-border lg:top-36 lg:flex lg:h-auto lg:w-10/12"
            src={'/images/dashboard-header.webp'}
            width={'2061'}
            height={'800'}
            alt={'Dashboard Header'}
          />
        </FeatureCard>

        <FeatureCard
          className={
            'relative col-span-2 w-full overflow-hidden lg:col-span-1'
          }
          label={'Authentication'}
          description={`Makerkit provides a variety of providers to allow your users to sign in.`}
        >
          <Image
            className="absolute left-16 top-32 hidden h-auto w-8/12 rounded-l-2xl lg:flex"
            src={'/images/sign-in.webp'}
            width={'1760'}
            height={'1680'}
            alt={'Sign In'}
          />
        </FeatureCard>

        <FeatureCard
          className={
            'relative col-span-2 overflow-hidden lg:col-span-1 lg:h-96'
          }
          label={'Multi Tenancy'}
          description={`Multi tenant memberships for your SaaS business.`}
        >
          <Image
            className="absolute right-0 top-0 hidden h-full w-full rounded-tl-2xl border lg:top-28 lg:flex lg:h-auto lg:w-8/12"
            src={'/images/multi-tenancy.webp'}
            width={'2061'}
            height={'800'}
            alt={'Multi Tenancy'}
          />
        </FeatureCard>

        <FeatureCard
          className={'relative col-span-2 overflow-hidden lg:h-96'}
          label={'Billing'}
          description={`Makerkit supports multiple payment gateways to charge your customers.`}
        >
          <Image
            className="absolute right-0 top-0 hidden h-full w-full rounded-tl-2xl border border-border lg:top-36 lg:flex lg:h-auto lg:w-11/12"
            src={'/images/billing.webp'}
            width={'2061'}
            height={'800'}
            alt={'Billing'}
          />
        </FeatureCard>
      </FeatureGrid>
    </FeatureShowcase>
  </div>
</div>
```

## SecondaryHero

The `SecondaryHero` component is a secondary hero section that can be used to highlight additional features or content on a landing page.

```tsx
<SecondaryHero
  pill={<Pill>Get started for free. No credit card required.</Pill>}
  heading="Fair pricing for all types of businesses"
  subheading="Get started on our free plan and upgrade when you are ready."
/>
```

{% component path="marketing/secondary-hero" /%}

## Header

The `Header` component is a navigation header that can be used to display links to different sections of a marketing page.

```tsx
export function SiteHeader(props: { user?: User | null }) {
  return (
    <Header
      logo={<AppLogo />}
      navigation={<SiteNavigation />}
      actions={<SiteHeaderAccountSection user={props.user ?? null} />}
    />
  );
}
```

## Footer

The `Footer` component is a footer section that can be used to display links, social media icons, and other information on a marketing page.

```tsx
import { Footer } from '@kit/ui/marketing';
import { Trans } from '@kit/ui/trans';

import { AppLogo } from '~/components/app-logo';
import appConfig from '~/config/app.config';

export function SiteFooter() {
  return (
    <Footer
      logo={<AppLogo className="w-[85px] md:w-[95px]" />}
      description={<Trans i18nKey="marketing:footerDescription" />}
      copyright={
        <Trans
          i18nKey="marketing:copyright"
          values={{
            product: appConfig.name,
            year: new Date().getFullYear(),
          }}
        />
      }
      sections={[
        {
          heading: <Trans i18nKey="marketing:about" />,
          links: [
            { href: '/blog', label: <Trans i18nKey="marketing:blog" /> },
            { href: '/contact', label: <Trans i18nKey="marketing:contact" /> },
          ],
        },
        {
          heading: <Trans i18nKey="marketing:product" />,
          links: [
            {
              href: '/docs',
              label: <Trans i18nKey="marketing:documentation" />,
            },
          ],
        },
        {
          heading: <Trans i18nKey="marketing:legal" />,
          links: [
            {
              href: '/terms-of-service',
              label: <Trans i18nKey="marketing:termsOfService" />,
            },
            {
              href: '/privacy-policy',
              label: <Trans i18nKey="marketing:privacyPolicy" />,
            },
            {
              href: '/cookie-policy',
              label: <Trans i18nKey="marketing:cookiePolicy" />,
            },
          ],
        },
      ]}
    />
  );
}
```

## CtaButton

The `CtaButton` component is a call-to-action button that can be used to encourage users to take a specific action.

{% component path="marketing/cta-button" /%}

```tsx
function MainCallToActionButton() {
  return (
    <div className={'flex space-x-4'}>
      <CtaButton>
        <Link href={'/auth/sign-up'}>
          <span className={'flex items-center space-x-0.5'}>
            <span>
              <Trans i18nKey={'common:getStarted'} />
            </span>

            <ArrowRightIcon
              className={
                'h-4 animate-in fade-in slide-in-from-left-8' +
                ' delay-1000 duration-1000 zoom-in fill-mode-both'
              }
            />
          </span>
        </Link>
      </CtaButton>

      <CtaButton variant={'link'}>
        <Link href={'/contact'}>
          <Trans i18nKey={'common:contactUs'} />
        </Link>
      </CtaButton>
    </div>
  );
}
```

## GradientSecondaryText

The `GradientSecondaryText` component is a text component that applies a gradient color to the text.

{% component path="marketing/gradient-secondary-text" /%}

```tsx
function GradientSecondaryTextExample() {
  return (
    <p>
      <GradientSecondaryText>
        Unleash your creativity and build your SaaS faster than ever with
        Makerkit.
      </GradientSecondaryText>
    </p>
  );
}
```

## GradientText

The `GradientText` component is a text component that applies a gradient color to the text.

{% component path="marketing/gradient-text" /%}

```tsx
function GradientTextExample() {
  return (
    <p>
      <GradientText className={'from-primary/60 to-primary'}>
        Unleash your creativity and build your SaaS faster than ever with
        Makerkit.
      </GradientText>
    </p>
  );
}
```

You can use the Tailwind CSS gradient utility classes to customize the gradient colors.

```tsx
<GradientText className={'from-violet-500 to-purple-700'}>
  Unleash your creativity and build your SaaS faster than ever with Makerkit.
</GradientText>
```

## NewsletterSignupContainer

The `NewsletterSignupContainer` is a comprehensive component for handling newsletter signups in a marketing context. It manages the entire signup flow, including form display, loading state, and success/error messages.

{% component path="marketing/newsletter-sign-up" /%}

### Import

```jsx
import { NewsletterSignupContainer } from '@kit/ui/marketing';
```

### Props

The `NewsletterSignupContainer` accepts the following props:
- `onSignup`: the callback function that will notify you of a submission
- `heading`: the heading of the component
- `description`: the description below the heading
- `successMessage`: the text to display on successful submissions
- `errorMessage`: the text to display on errors

The component also accepts all standard HTML div attributes.

### Usage

```tsx
'use client';

import { NewsletterSignupContainer } from '@kit/ui/marketing';

function WrapperNewsletterComponent() {
  const handleNewsletterSignup = async (email: string) => {
    // Implement your signup logic here
    await apiClient.subscribeToNewsletter(email);
  };

  return (
    <NewsletterSignupContainer 
      onSignup={handleNewsletterSignup}
      heading="Join Our Community"
      description="Be the first to know about new features and updates."
      successMessage="You're all set! Check your inbox for a confirmation email."
      errorMessage="Oops! Something went wrong. Please try again later."
      className="max-w-md mx-auto"
    />
  );
}
```

Wrap the component into a parent `client` component as you'll need to pass the `onSignup` function to the component.

The `onSignup` function should handle the signup process, such as making an API request to subscribe the user to the newsletter, whichever provider you're using.

### Behavior

1. Initially displays the newsletter signup form.
2. When the form is submitted, it shows a loading spinner.
3. On successful signup, displays a success message.
4. If an error occurs during signup, shows an error message.

### Styling

The component uses Tailwind CSS for styling. The container is flexbox-based and centers its content. You can customize the appearance by passing additional classes via the `className` prop.

### Accessibility

- Uses semantic HTML structure with appropriate headings.
- Provides clear feedback for form submission states.
- Error and success messages are displayed using the `Alert` component for consistent styling and accessibility.

### Notes

- It integrates with other Makerkit UI components like `Alert`, `Heading`, and `Spinner`.
- The actual signup logic is decoupled from the component, allowing for flexibility in implementation.


-----------------
FILE PATH: kits/next-supabase-turbo/components/multi-step-forms.mdoc

---
status: "published"
label: "Multi Step Forms"
title: "Multi Step Forms in the Next.js Supabase SaaS kit"
description: "Building multi-step forms in the Next.js Supabase SaaS kit"
order: 0
---

The Multi-Step Form Component is a powerful and flexible wrapper around React Hook Form, Zod, and Shadcn UI. It provides a simple API to create multi-step forms with ease, perfect for complex registration processes, surveys, or any scenario where you need to break down a long form into manageable steps.

## Features

- Easy integration with React Hook Form and Zod for form management and validation
- Built-in step management
- Customizable layout and styling
- Progress tracking with optional Stepper component
- TypeScript support for type-safe form schemas

{% component path="multi-step-form" /%}

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




