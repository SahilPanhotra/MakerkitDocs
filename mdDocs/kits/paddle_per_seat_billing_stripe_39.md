-----------------
FILE PATH: kits\next-supabase-turbo\billing\paddle.mdoc

---
status: "published"
label: 'Paddle'
title: 'Configuring Paddle Billing | Next.js Supabase SaaS Kit Turbo'
order: 4
description: 'Paddle is a payment processor that allows you to charge your users for your SaaS product. It is a Merchant of Record, which means they handle all the billing and compliance for you.'
---

{% alert type="info" title="Paddle is not yet available" %}
The integration will be developed after Paddle releases their Customer Portal.
{% /alert %}

Stay tuned!


-----------------
FILE PATH: kits\next-supabase-turbo\billing\per-seat-billing.mdoc

---
status: "published"
label: "Per Seat Billing"
title: "How to configure per seat billing in Makerkit"
order: 6
description: "Learn how to setup per seat billing in Makerkit to charge customers for additional seats (users) in your application."
---

{% sequence title="How to configure per seat billing in Makerkit" description="Learn how to setup per seat billing in Makerkit to charge customers for additional seats (users) in your application." %}

[What the kit does for you](#what-the-kit-does-for-you)

[Define a per-seat billing schema](#define-a-per-seat-billing-schema)

[Report the number of seats](#report-the-number-of-seats)

{% /sequence %}

Per Seat billing is a common pricing model for SaaS applications where customers are charged based on the number of seats (users) they have in the application. In this guide, we'll show you how to configure per seat billing in Makerkit.

## What the kit does for you

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
FILE PATH: kits\next-supabase-turbo\billing\stripe.mdoc

---
status: "published"
label: "Stripe"
title: "Configuring Stripe Billing"
description: "Learn how to configure Stripe in the Makerkit Next.js Supabase SaaS Kit"
order: 2
---

Stripe is the default billing provider in both the local config and the DB, so you don't need to set these values if you want to use Stripe.

{% sequence title="Steps to configure Stripe" description="Learn how to configure Stripe in the Makerkit Next.js Supabase SaaS Kit" %}

[Environment variables](#environment-variables)

[Stripe CLI](#stripe-cli)

[Configuring the Stripe Customer Portal](#configuring-the-stripe-customer-portal)

[Setting Production Webhooks in Stripe](#setting-production-webhooks-in-stripe)

[Setting Stripe Trials without collecting a card](#setting-stripe-trials-without-collecting-a-card)
{% /sequence %}

## Environment variables

For Stripe, you'll need to set the following environment variables:

```bash
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
```

While the `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` is public and can be added anywhere - **please do not add the secret keys** to the `.env` file.

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

Once you start listening to Stripe events, you can use the CLI to listen to Stripe events.

Now, please copy the webhook secret displayed in the terminal and set it as the `STRIPE_WEBHOOK_SECRET` environment variable in your `.env.local` file:

```bash
STRIPE_WEBHOOK_SECRET=*your_webhook_secret*
```

**Please sign in and then re-run the command**. Now, you can listen to Stripe events.

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

### Setting Stripe Trials without collecting a card

Makerkit allows you to set trials without collecting a card. This is useful if you want to offer a free trial for your users.

To do so, you need to set the `STRIPE_ENABLE_TRIAL_WITHOUT_CC` environment variable to `true`.

```bash
STRIPE_ENABLE_TRIAL_WITHOUT_CC=true
```

Of course - you also must set a trial period for the plan you want to offer a free trial for.


-----------------
FILE PATH: kits\next-supabase-turbo\billing.mdoc

---
status: "published"
label: "Billing"
title: "How Billing works in the Next.js Supabase SaaS kit"
description: "A quick introduction to billing in Makerkit and how to set it up in the Next.js Supabase SaaS kit"
order: 7
collapsible: true
collapsed: true
canonical: '/docs/next-supabase-turbo/billing/overview'
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

