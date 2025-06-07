-----------------
FILE PATH: kits/react-router-supabase-turbo/components/bordered-navigation-menu.mdoc

---
status: "published"

label: "Bordered Navigation Menu"
title: "Bordered Navigation Menu Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Bordered Navigation Menu component in the React Router Supabase SaaS kit"
order: 6
---


The BorderedNavigationMenu components provide a stylish and interactive navigation menu with a bordered, underline-style active state indicator. These components are built on top of the NavigationMenu from Shadcn UI and are designed to work seamlessly with React Router routing.

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

1. **Automatic Active State**: Uses React Router `useLocation` to automatically determine if the item is active based on the current route.
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

These components provide a sleek, accessible way to create navigation menus in your React Router application, with built-in support for styling active states and internationalization.

-----------------
FILE PATH: kits/react-router-supabase-turbo/components/card-button.mdoc

---
status: "published"
label: "Card Button"
title: "Card Button Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Card Button component in the React Router Supabase SaaS kit"
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
4. Leverage the `asChild` prop when you need to change the underlying element (e.g., for routing with React Router `Link` component).

These CardButton components provide a flexible and customizable way to create interactive card-like buttons in your React application, suitable for various use cases such as feature showcases, navigation elements, or clickable information cards.

-----------------
FILE PATH: kits/react-router-supabase-turbo/components/cookie-banner.mdoc

---
status: "published"

label: "Cookie Banner"
title: "Cookie Banner Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Cookie Banner component in the React Router Supabase SaaS kit"
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
FILE PATH: kits/react-router-supabase-turbo/components/data-table.mdoc

---
status: "published"

label: "Data Table"
title: "Data Table Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Data Table component in the React Router Supabase SaaS kit"
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

## Customization

The DataTable component is built with customization in mind. You can customize the appearance using Tailwind CSS classes and extend its functionality by passing custom props to the underlying Table component.

## Internationalization

The component uses the `Trans` component for internationalization. Ensure you have your i18n setup correctly to leverage this feature.

The DataTable component provides a powerful and flexible solution for displaying tabular data in your React applications, with built-in support for common table features and easy integration with server-side data loading.

-----------------
FILE PATH: kits/react-router-supabase-turbo/components/if.mdoc

---
status: "published"

label: "Conditional Rendering"
title: "Dynamic Conditional Rendering in the React Router Supabase SaaS kit"
description: "Learn how to use the If component in the React Router Supabase SaaS kit"
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
FILE PATH: kits/react-router-supabase-turbo/components/loading-overlay.mdoc

---
status: "published"

label: "Loading Overlay"
title: "Loading Overlay Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Loading Overlay component in the React Router Supabase SaaS kit"
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


-----------------
FILE PATH: kits/react-router-supabase-turbo/components/multi-step-forms.mdoc

---
status: "published"

label: "Multi Step Forms"
title: "Multi Step Forms in the React Router Supabase SaaS kit"
description: "Building multi-step forms in the React Router Supabase SaaS kit"
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
FILE PATH: kits/react-router-supabase-turbo/components/page.mdoc

---
status: "published"

label: "Page"
title: "Page Component in the React Router Supabase SaaS kit"
description: "Learn how to use the Page component in the React Router Supabase SaaS kit"
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

