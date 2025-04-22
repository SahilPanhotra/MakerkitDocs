-----------------
FILE PATH: kits\next-supabase-turbo\components\app-breadcrumbs.mdoc

---
status: "published"

label: "App Breadcrumbs"
title: "App Breadcrumbs Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the App Breadcrumbs component in the Next.js Supabase SaaS kit"
order: 6
---


The `AppBreadcrumbs` component creates a dynamic breadcrumb navigation based on the current URL path. It's designed to work with Next.js and uses the `usePathname` hook from Next.js for routing information.

## Features

- Automatically generates breadcrumbs from the current URL path
- Supports custom labels for path segments
- Limits the number of displayed breadcrumbs with an ellipsis for long paths
- Internationalization support with the `Trans` component
- Responsive design with different text sizes for mobile and desktop

## Usage

```tsx
import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';

function MyPage() {
  return (
    <AppBreadcrumbs
      values={{
        "custom-slug": "Custom Label"
      }}
      maxDepth={4}
    />
  );
}
```

When you have IDs in your URL, you can use the `values` prop to provide custom labels for those segments. For example, if your URL is `/users/123`, you can set `values={{ "123": "User Profile" }}` to display "User Profile" instead of "123" in the breadcrumb.

```tsx
<AppBreadcrumbs
  values={{
    "123": "User"
  }}
/>
```

This will display "User" instead of "123" in the breadcrumb.

## Props

The component accepts two optional props:

1. `values`: An object where keys are URL segments and values are custom labels.
- Type: `Record<string, string>`
- Default: `{}`
2. `maxDepth`: The maximum number of breadcrumb items to display before using an ellipsis.
- Type: `number`
- Default: `6`

## Functionality

- The component splits the current path into segments and creates a breadcrumb item for each.
- If the number of segments exceeds `maxDepth`, it shows an ellipsis (...) to indicate hidden segments.
- The last breadcrumb item is not clickable and represents the current page.
- Custom labels can be provided through the `values` prop.
- For segments without custom labels, it attempts to use an i18n key (`common.routes.[unslugified-path]`). If no translation is found, it falls back to the unslugified path.

## Styling

- The component uses Tailwind CSS classes for styling.
- Breadcrumb items are capitalized.
- On larger screens (lg breakpoint), the text size is slightly smaller.

## Dependencies

This component relies on several other components and utilities:

- Next.js `usePathname` hook
- Custom UI components (Breadcrumb, BreadcrumbItem, etc.) from Shadcn UI
- `If` component for conditional rendering
- `Trans` component for internationalization

This component provides a flexible and easy-to-use solution for adding breadcrumb navigation to your Next.js application. It's particularly useful for sites with deep hierarchical structures or those requiring dynamic breadcrumb generation.

-----------------
FILE PATH: kits\next-supabase-turbo\components\bordered-navigation-menu.mdoc

---
status: "published"

label: "Bordered Navigation Menu"
title: "Bordered Navigation Menu Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Bordered Navigation Menu component in the Next.js Supabase SaaS kit"
order: 8
---


The BorderedNavigationMenu components provide a stylish and interactive navigation menu with a bordered, underline-style active state indicator. These components are built on top of the NavigationMenu from Shadcn UI and are designed to work seamlessly with Next.js routing.

## BorderedNavigationMenu

This component serves as a container for navigation menu items.

### Usage

```jsx
import { BorderedNavigationMenu, BorderedNavigationMenuItem } from '@kit/ui/bordered-navigation-menu';

function MyNavigation() {
  return (
    <BorderedNavigationMenu>
      <BorderedNavigationMenuItem path="/home" label="Home" />
      <BorderedNavigationMenuItem path="/about" label="About" />
      {/* Add more menu items as needed */}
    </BorderedNavigationMenu>
  );
}
```

### Props

- `children: React.ReactNode`: The navigation menu items to be rendered.

## BorderedNavigationMenuItem

This component represents an individual item in the navigation menu.

### Props

- `path: string` (required): The URL path for the navigation item.
- `label: React.ReactNode | string` (required): The text or content to display for the item.
- `end?: boolean | ((path: string) => boolean)`: Determines if the path should match exactly or use a custom function for active state.
- `active?: boolean`: Manually set the active state of the item.
- `className?: string`: Additional CSS classes for the menu item container.
- `buttonClassName?: string`: Additional CSS classes for the button element.

### Features

1. **Automatic Active State**: Uses Next.js's `usePathname` to automatically determine if the item is active based on the current route.
2. **Custom Active State Logic**: Allows for custom active state determination through the `end` prop.
3. **Internationalization**: Supports i18n through the `Trans` component for string labels.
4. **Styling**: Utilizes Tailwind CSS for styling, with active items featuring an underline animation.

### Example

```jsx
<BorderedNavigationMenuItem
  path="/dashboard"
  label="common:dashboardLabel"
  end={true}
  className="my-custom-class"
  buttonClassName="px-4 py-2"
/>
```

## Styling

The components use Tailwind CSS for styling. Key classes include:

- Menu container: `relative h-full space-x-2`
- Menu item button: `relative active:shadow-sm`
- Active indicator: `absolute -bottom-2.5 left-0 h-0.5 w-full bg-primary animate-in fade-in zoom-in-90`

You can further customize the appearance by passing additional classes through the `className` and `buttonClassName` props.

## Best Practices

1. Use consistent labeling and paths across your application.
2. Leverage the `Trans` component for internationalization of labels.
3. Consider the `end` prop for more precise control over the active state for nested routes.
4. Use the `active` prop sparingly, preferring the automatic active state detection when possible.

These components provide a sleek, accessible way to create navigation menus in your Next.js application, with built-in support for styling active states and internationalization.

-----------------
FILE PATH: kits\next-supabase-turbo\components\card-button.mdoc

---
status: "published"
label: "Card Button"
title: "Card Button Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Card Button component in the Next.js Supabase SaaS kit"
order: 7
---

The CardButton components provide a set of customizable, interactive card-like buttons for use in React applications. These components are built with flexibility in mind, allowing for easy composition and styling.

{% component path="card-button" /%}

## Components

### CardButton

The main wrapper component for creating a card-like button.

#### Props

- `asChild?: boolean`: If true, the component will render its children directly.
- `className?: string`: Additional CSS classes to apply to the button.
- `children: React.ReactNode`: The content of the button.
- `...props`: Any additional button props.

#### Usage

```jsx
<CardButton onClick={handleClick}>
  {/* Card content */}
</CardButton>
```

### CardButtonTitle

Component for rendering the title of the card button.

#### Props

- `className?: string`: Additional CSS classes for the title.
- `asChild?: boolean`: If true, renders children directly.
- `children: React.ReactNode`: The title content.

#### Usage

```jsx
<CardButtonTitle>My Card Title</CardButtonTitle>
```

### CardButtonHeader

Component for the header section of the card button.

#### Props

- `className?: string`: Additional CSS classes for the header.
- `asChild?: boolean`: If true, renders children directly.
- `displayArrow?: boolean`: Whether to display the chevron icon (default: true).
- `children: React.ReactNode`: The header content.

#### Usage

```jsx
<CardButtonHeader displayArrow={false}>
  <CardButtonTitle>Header Content</CardButtonTitle>
</CardButtonHeader>
```

### CardButtonContent

Component for the main content area of the card button.

#### Props

- `className?: string`: Additional CSS classes for the content area.
- `asChild?: boolean`: If true, renders children directly.
- `children: React.ReactNode`: The main content.

#### Usage

```jsx
<CardButtonContent>
  <p>Main card content goes here</p>
</CardButtonContent>
```

### CardButtonFooter

Component for the footer section of the card button.

#### Props

- `className?: string`: Additional CSS classes for the footer.
- `asChild?: boolean`: If true, renders children directly.
- `children: React.ReactNode`: The footer content.

#### Usage

```jsx
<CardButtonFooter>
  <span>Footer information</span>
</CardButtonFooter>
```

## Styling

These components use Tailwind CSS for styling. Key features include:

- Hover and active states for interactive feedback
- Responsive sizing and layout
- Dark mode support
- Customizable through additional class names

## Example

Here's a complete example of how to use these components together:

```jsx
import {
  CardButton,
  CardButtonTitle,
  CardButtonHeader,
  CardButtonContent,
  CardButtonFooter
} from '@kit/ui/card-button';

function MyCardButton() {
  return (
    <CardButton onClick={() => console.log('Card clicked')}>
      <CardButtonHeader>
        <CardButtonTitle>Featured Item</CardButtonTitle>
      </CardButtonHeader>
      <CardButtonContent>
        <p>This is a detailed description of the featured item.</p>
      </CardButtonContent>
      <CardButtonFooter>
        <span>Click to learn more</span>
      </CardButtonFooter>
    </CardButton>
  );
}
```

## Accessibility

- The components use semantic HTML elements when not using the `asChild` prop.
- Interactive elements are keyboard accessible.

## Best Practices

1. Use clear, concise titles in `CardButtonTitle`.
2. Provide meaningful content in `CardButtonContent` for user understanding.
3. Use `CardButtonFooter` for calls-to-action or additional information.
4. Leverage the `asChild` prop when you need to change the underlying element (e.g., for routing with Next.js `Link` component).

These CardButton components provide a flexible and customizable way to create interactive card-like buttons in your React application, suitable for various use cases such as feature showcases, navigation elements, or clickable information cards.

-----------------
FILE PATH: kits\next-supabase-turbo\components\coming-soon.mdoc

---
status: "published"
label: "Temporary Landing Page"
title: "A temporary minimal landing page for your SaaS"
description: "Looking to ship as quickly as possible? Use the Coming Soon component to showcase your product's progress."
order: 9
---

If you're rushing to launch your SaaS, you can use the Coming Soon component to showcase a minimal landing page for your product and generate buzz before you launch.

{% component path="coming-soon" /%}

My suggestions is to replace the whole `(marketing)` layout using the Coming Soon component.

This will save you a lot of time making sure the landing page and the links are filled with the right information.

```tsx {% title="apps/web/app/(marketing)/layout.tsx" %}
import Link from 'next/link';

import {
  ComingSoon,
  ComingSoonButton,
  ComingSoonHeading,
  ComingSoonLogo,
  ComingSoonText,
} from '@kit/ui/marketing';

import { AppLogo } from '~/components/app-logo';
import appConfig from '~/config/app.config';

export default function SiteLayout() {
  return (
    <ComingSoon>
      <ComingSoonLogo>
        <AppLogo />
      </ComingSoonLogo>

      <ComingSoonHeading>{appConfig.name} is coming soon</ComingSoonHeading>

      <ComingSoonText>
        We&apos;re building something amazing. Our team is working hard to bring
        you a product that will revolutionize how you work.
      </ComingSoonText>

      <ComingSoonButton asChild>
        <Link href="#">Follow Our Progress</Link>
      </ComingSoonButton>

      {/* Additional custom content */}
      <div className="mt-8 flex justify-center gap-4">
        {/* Social icons, etc */}
      </div>
    </ComingSoon>
  );
}
```

Even better, you can use an env variable to check if it's a production build or not and displaying the normal layout during development:

```tsx {% title="apps/web/app/(marketing)/layout.tsx" %}
import Link from 'next/link';

import {
  ComingSoon,
  ComingSoonButton,
  ComingSoonHeading,
  ComingSoonLogo,
  ComingSoonText,
} from '@kit/ui/marketing';

import { SiteFooter } from '~/(marketing)/_components/site-footer';
import { SiteHeader } from '~/(marketing)/_components/site-header';
import { withI18n } from '~/lib/i18n/with-i18n';

import { AppLogo } from '~/components/app-logo';
import appConfig from '~/config/app.config';

function SiteLayout() {
  if (!appConfig.production) {
    return (
      <div className={'flex min-h-[100vh] flex-col'}>
        <SiteHeader />

        {props.children}

        <SiteFooter />
      </div>
    );
  }

  return (
    <ComingSoon>
      <ComingSoonLogo>
        <AppLogo />
      </ComingSoonLogo>

      <ComingSoonHeading>{appConfig.name} is coming soon</ComingSoonHeading>

      <ComingSoonText>
        We&apos;re building something amazing. Our team is working hard to bring
        you a product that will revolutionize how you work.
      </ComingSoonText>

      <ComingSoonButton asChild>
        <Link href="#">Follow Our Progress</Link>
      </ComingSoonButton>

      {/* Additional custom content */}
      <div className="mt-8 flex justify-center gap-4">
        {/* Social icons, etc */}
      </div>
    </ComingSoon>
  );
}

export default withI18n(SiteLayout);
```

-----------------
FILE PATH: kits\next-supabase-turbo\components\cookie-banner.mdoc

---
status: "published"
label: "Cookie Banner"
title: "Cookie Banner Component in the Next.js Supabase SaaS kit"
description: "Learn how to use the Cookie Banner component in the Next.js Supabase SaaS kit"
order: 7
---

This module provides a `CookieBanner` component and a `useCookieConsent` hook for managing cookie consent in React applications.

{% component path="cookie-banner" /%}

## CookieBanner Component

The CookieBanner component displays a consent banner for cookies and tracking technologies.

### Usage

```jsx
import dynamic from 'next/dynamic';

const CookieBanner = dynamic(() => import('@kit/ui/cookie-banner').then(m => m.CookieBanner), {
  ssr: false
});

function App() {
  return (
    <div>
      {/* Your app content */}
      <CookieBanner />
    </div>
  );
}
```

### Features

- Displays only when consent status is unknown
- Automatically hides after user interaction
- Responsive design (different layouts for mobile and desktop)
- Internationalization support via the `Trans` component
- Animated entrance using Tailwind CSS

## useCookieConsent Hook

This custom hook manages the cookie consent state and provides methods to update it.

### Usage

```jsx
import { useCookieConsent } from '@kit/ui/cookie-banner';

function MyComponent() {
  const { status, accept, reject, clear } = useCookieConsent();

  // Use these values and functions as needed
}
```

### API

- `status: ConsentStatus`: Current consent status (Accepted, Rejected, or Unknown)
- `accept(): void`: Function to accept cookies
- `reject(): void`: Function to reject cookies
- `clear(): void`: Function to clear the current consent status

## ConsentStatus Enum

```typescript
enum ConsentStatus {
  Accepted = 'accepted',
  Rejected = 'rejected',
  Unknown = 'unknown'
}
```

## Key Features

1. **Persistent Storage**: Consent status is stored in localStorage for persistence across sessions.
2. **Server-Side Rendering Compatible**: Checks for browser environment before accessing localStorage.
3. **Customizable**: The `COOKIE_CONSENT_STATUS` key can be configured as needed.
4. **Reactive**: The banner automatically updates based on the consent status.

## Styling

The component uses Tailwind CSS for styling, with support for dark mode and responsive design.

## Accessibility

- Uses Radix UI's Dialog primitive for improved accessibility
- Autofocu s on the "Accept" button for keyboard navigation

## Internationalization

The component uses the `Trans` component for internationalization. Ensure you have the following keys in your i18n configuration:

- `cookieBanner.title`
- `cookieBanner.description`
- `cookieBanner.reject`
- `cookieBanner.accept`

## Best Practices

1. Place the `CookieBanner` component at the root of your application to ensure it's always visible when needed.
2. Use the `useCookieConsent` hook to conditionally render content or initialize tracking scripts based on the user's consent.
3. Provide clear and concise information about your cookie usage in the banner description.
4. Ensure your privacy policy is up-to-date and accessible from the cookie banner or nearby.

## Example: Conditional Script Loading

```jsx
function App() {
  const { status } = useCookieConsent();

  useEffect(() => {
    if (status === ConsentStatus.Accepted) {
      // Initialize analytics or other cookie-dependent scripts
    }
  }, [status]);

  return (
    <div>
      {/* Your app content */}
      <CookieBanner />
    </div>
  );
}
```

This cookie consent management system provides a user-friendly way to comply with cookie laws and regulations while maintaining a good user experience.

-----------------
FILE PATH: kits\next-supabase-turbo\components\data-table.mdoc

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
FILE PATH: kits\next-supabase-turbo\components\empty-state.mdoc

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
FILE PATH: kits\next-supabase-turbo\components\if.mdoc

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
FILE PATH: kits\next-supabase-turbo\components\loading-overlay.mdoc

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
FILE PATH: kits\next-supabase-turbo\components\marketing-components.mdoc

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


