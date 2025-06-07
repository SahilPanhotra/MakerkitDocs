-----------------
FILE PATH: kits/remix-supabase-turbo/billing/paddle.mdoc

---
status: "published"
label: "Paddle"
title: "Configuring Paddle Billing | Remix Supabase SaaS Kit Turbo"
order: 4
description: "Paddle is a payment processor that allows you to charge your users for your SaaS product. It is a Merchant of Record, which means they handle all the billing and compliance for you."
---

{% alert type="info" title="Paddle is not yet available" %}
The integration will be developed after Paddle releases their Customer Portal.
{% /alert %}

Stay tuned!


-----------------
FILE PATH: kits/remix-supabase-turbo/billing/per-seat-billing.mdoc

---
status: "published"

label: "Per Seat Billing"
title: "How to configure per seat billing in Makerkit"
order: 6
description: "Learn how to setup per seat billing in Makerkit to charge customers for additional seats (users) in your application."
---


Per Seat billing is a common pricing model for SaaS applications where customers are charged based on the number of seats (users) they have in the application. In this guide, we'll show you how to configure per seat billing in Makerkit.

Makerkit will automatically:
1. understand if your pricing model depends on the number of seats (using the `per_seat` line item type in your pricing model)
2. calculate the number of seats based on the members count in a Team account
3. report the number of seats to your billing system when subscribing
4. report the number of seats to your billing system when adding or removing members

## Define a per-seat billing schema

Per-seat billing is defined by the following fields:

```tsx
export default createBillingSchema({
  provider,
  products: [
    {
      id: 'starter',
      name: 'Starter',
      description: 'The perfect plan to get started',
      currency: 'USD',
      badge: `Value`,
      plans: [
        {
          name: 'Starter Monthly',
          id: 'starter-monthly',
          trialDays: 7,
          paymentType: 'recurring',
          interval: 'month',
          lineItems: [
            {
              id: 'price_1NNwYHI1i3VnbZTqI2UzaHIe',
              name: 'Addon 2',
              cost: 0,
              type: 'per_seat',
              tiers: [
                {
                    upTo: 3,
                    cost: 0,
                },
                {
                    upTo: 5,
                    cost: 7.99,
                },
                {
                    upTo: 'unlimited',
                    cost: 5.99,
                }
              ]
            },
          ],
        }
      ],
    }
  ]
});
```

Let's break down the fields:
- **id**: The unique identifier for the line item. **This must match the price ID in the billing provider**. The schema will validate this, but please remember to set it correctly.
- **name**: The name of the line item
- **cost**: The cost of the line item. This can be set to `0` as the cost is calculated based on the tiers.
- **type**: The type of the line item (e.g., `flat`, `metered`, `per-seat`). In this case, it's `per-seat`.
- **tiers**: The tiers of the line item. Each tier is defined by the following fields:
- **upTo**: The upper limit of the tier. If the usage is below this limit, the cost is calculated based on this tier.
- **cost**: The cost of the tier. This is the cost per unit.

If you set the first tier to `0`, it basically means that the first `n` seats are free. This is a common practice in per-seat billing.

Please remember that the cost is set for UI purposes. **The billing provider will handle the actual billing** - therefore, **please make sure the cost is correctly set in the billing provider**.

## Report the number of seats

This is done automatically for you. Makerkit will report the number of seats to your billing provider when subscribing, and when adding or removing members.

-----------------
FILE PATH: kits/remix-supabase-turbo/billing/stripe.mdoc

---
status: "published"
label: "Stripe"
title: "Configuring Stripe Billing in the Remix Supabase SaaS Kit"
description: "Learn how to configure Stripe in the Makerkit Remix Supabase SaaS Kit"
order: 2
---

Stripe is the default billing provider in both the local config and the DB, so you don't need to set these values if you want to use Stripe.

For Stripe, you'll need to set the following environment variables:

```bash
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
VITE_STRIPE_PUBLISHABLE_KEY=
```

While the `VITE_STRIPE_PUBLISHABLE_KEY` is public and can be added anywhere - **please do not add the secret keys** to the `.env` file.

During development, you can place them in `.env.local` as it's not committed to the repository. In production, you can set them in the environment variables of your hosting provider.

## Stripe CLI

The Stripe CLI which allows you to listen to Stripe events straight to your own localhost. You can install and use the CLI using a variety of methods, but we recommend using Docker - since you already have it installed.

First - login to your Stripe account using the project you want to run:

```bash
docker run --rm -it --name=stripe -v ~/.config/stripe:/root/.config/stripe stripe/stripe-cli:latest login
```

Now, you can listen to Stripe events running the following command:

```bash
pnpm run stripe:listen
```

**If you have not logged in - the first time you set it up, you are required to sign in**. This is a one-time process. Once you sign in, you can use the CLI to listen to Stripe events.

**Please sign in and then re-run the command**. Now, you can listen to Stripe events.

Once you start listening to Stripe events, you can use the CLI to listen to Stripe events.

Now, please copy the webhook secret displayed in the terminal and set it as the `STRIPE_WEBHOOK_SECRET` environment variable in your `.env.local` file:

```bash
STRIPE_WEBHOOK_SECRET=*your_webhook_secret*
```

If you're not receiving events, please make sure that:

1. the webhook secret is correct
2. the account you signed in is the same as the one you're using in your app

## Configuring the Stripe Customer Portal

Stripe requires you to set up the Customer Portal so that users can manage their billing information, invoices and plan settings from there.

{% img src="/assets/images/docs/stripe-customer-portal.webp" width="2712" height="1870" /%}

1. Please make sure to enable the setting that lets users switch plans
2. Configure the behavior of the cancellation according to your needs

## Setting Production Webhooks in Stripe

When going to production, you will need to set the webhook URL and the events you want to listen to in Stripe.

The webhook path is `/api/billing/webhook`. If your app is hosted at `https://myapp.com` then you need to enter `https://myapp.com/api/billing/webhook`.

{% alert title="Use a public URL" type="warning" %}
Please make sure to use a public URL for your webhooks. If you use a private URL (such as Vercel preview URLs), you will not be able to receive webhooks from Stripe.
{% /alert %}

To ensure the URL is publicly accessible, please visit it using Incognito mode in your browser.

Makerkit needs the following events to work:

1. `checkout.session.completed`
2. `customer.subscription.updated`
3. `customer.subscription.deleted`

Only if you're using one-off payments, please add:

1. `checkout.session.async_payment_failed`
2. `checkout.session.async_payment_succeeded`

If your application needs more events, please add them, [but remember to handle them](billing-webhooks).

You can [handle additional events](billing-webhooks) by adding the required handlers in the `api/billing/webhook/route.ts` file.


-----------------
FILE PATH: kits/remix-supabase-turbo/billing.mdoc

---
status: "published"
label: "Billing"
title: "How Billing works in the Remix Supabase SaaS kit"
description: "A quick introduction to billing in Makerkit and how to set it up in the Remix Supabase SaaS kit"
order: 7
collapsible: true
collapsed: true
---

The billing package is used to manage subscriptions, one-off payments, and more.

The billing package is abstracted from the billing gateway package, which is used to manage the payment gateway (e.g., Stripe, Lemon Squeezy, etc.).

To set up the billing package, you need to set the following environment variables:

```bash
NEXT_PUBLIC_BILLING_PROVIDER=stripe # or lemon-squeezy
```

## Billing in Makerkit

Makerkit implements two packages for managing billing:

1. `core`: this package is responsible for exporting the billing service and the schema getFontDefinitionFromNetwork
2. `gateway`: the gateway is a router that handles the billing service and the billing provider (e.g., Stripe, Lemon Squeezy, etc.)

Then, we define a package for each provider:

1. `stripe`: this package is responsible for handling the Stripe API
2. `lemon-squeezy`: this package is responsible for handling the Lemon Squeezy API
3. `paddle`: this package is responsible for handling the Paddle API (Coming soon)

To summarize:

1. core defines the service and the schema
2. the provider's packages define the API calls based on the provider's API
3. the gateway package is responsible for routing the requests to the correct provider

Whatever provider you choose, you need to set the environment variable `NEXT_PUBLIC_BILLING_PROVIDER` to the provider you want to use. The Gateway Service will then route the requests to the correct provider. This means you can switch between providers without changing the code. The schema is the same for all providers - but the details of the API calls are different - so some details apply.

## Subscriptions vs. One-off payments

Makerkit supports both one-off payments and subscriptions. You have the choice to use one or both. What Makerkit cannot assume with certainty is the billing mode you want to use. By default, we assume you want to use subscriptions, as this is the most common billing mode for SaaS applications.

This means that - by default - Makerkit will be looking for a subscription plan when visiting the billing section of the personal or team account. This means we fetch data from the tables `subscriptions` and `subscription_items`.

If you want to use one-off payments, you need to set the billing mode to `one-time`:

```bash
BILLING_MODE=one-time
```

By doing so, Makerkit will be looking for one-off payments when visiting the billing section of the personal or team account. This means we fetch data from the tables `orders` and `order_items`.

### But - I want to use both

Perfect - you can, but you need to customize the pages to display the correct data.

---


Depending on the service you use, you will need to set the environment variables accordingly. By default - the billing package uses Stripe. Alternatively, you can use Lemon Squeezy. In the future, we will also add Paddle.


-----------------
FILE PATH: kits/remix-supabase-turbo/components/bordered-navigation-menu.mdoc

---
status: "published"

label: "Bordered Navigation Menu"
title: "Bordered Navigation Menu Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Bordered Navigation Menu component in the Remix Supabase SaaS kit"
order: 6
---


The BorderedNavigationMenu components provide a stylish and interactive navigation menu with a bordered, underline-style active state indicator. These components are built on top of the NavigationMenu from Shadcn UI and are designed to work seamlessly with Remix routing.

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

1. **Automatic Active State**: Uses Remix `useLocation` to automatically determine if the item is active based on the current route.
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

These components provide a sleek, accessible way to create navigation menus in your Remix application, with built-in support for styling active states and internationalization.

-----------------
FILE PATH: kits/remix-supabase-turbo/components/card-button.mdoc

---
status: "published"
label: "Card Button"
title: "Card Button Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Card Button component in the Remix Supabase SaaS kit"
order: 7
---

The CardButton components provide a set of customizable, interactive card-like buttons for use in React applications. These components are built with flexibility in mind, allowing for easy composition and styling.

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
4. Leverage the `asChild` prop when you need to change the underlying element (e.g., for routing with Remix `Link` component).

These CardButton components provide a flexible and customizable way to create interactive card-like buttons in your React application, suitable for various use cases such as feature showcases, navigation elements, or clickable information cards.

-----------------
FILE PATH: kits/remix-supabase-turbo/components/cookie-banner.mdoc

---
status: "published"

label: "Cookie Banner"
title: "Cookie Banner Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Cookie Banner component in the Remix Supabase SaaS kit"
order: 7
---


This module provides a `CookieBanner` component and a `useCookieConsent` hook for managing cookie consent in React applications.

## CookieBanner Component

The CookieBanner component displays a consent banner for cookies and tracking technologies.

### Usage

```jsx
import { CookieBanner } from '@kit/ui/cookie-banner';

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
FILE PATH: kits/remix-supabase-turbo/components/data-table.mdoc

---
status: "published"

label: "Data Table"
title: "Data Table Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Data Table component in the Remix Supabase SaaS kit"
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

export function loader() {
}

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
FILE PATH: kits/remix-supabase-turbo/components/if.mdoc

---
status: "published"

label: "Conditional Rendering"
title: "Dynamic Conditional Rendering in the Remix Supabase SaaS kit"
description: "Learn how to use the If component in the Remix Supabase SaaS kit"
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

The `If` component is fully typed - this allows for type-safe usage of the render props pattern:

```typescript
<If condition={user}>
  {(user) => <UserProfile name={user.name} email={user.email} />}
</If>
```

The `If` component provides a clean and efficient way to handle conditional rendering in React applications, improving code readability and maintainability.

-----------------
FILE PATH: kits/remix-supabase-turbo/components/loading-overlay.mdoc

---
status: "published"

label: "Loading Overlay"
title: "Loading Overlay Component in the Remix Supabase SaaS kit"
description: "Learn how to use the Loading Overlay component in the Remix Supabase SaaS kit"
order: 3
---


The LoadingOverlay component is a versatile UI element designed to display a loading state with a spinner and optional content. It's perfect for indicating background processes or page loads in your application.

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


