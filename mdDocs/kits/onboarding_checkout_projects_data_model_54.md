-----------------
FILE PATH: kits\next-supabase-turbo\recipes\onboarding-checkout.mdoc

---
status: "published"
title: 'Creating an Onboarding and Checkout flows'
label: 'Onboarding Checkout'
order: 2
description: 'Learn how to create an onboarding and checkout flow in the Next.js Supabase Starter Kit.'
---

One popular request from customers is to have a way to onboard new users and guide them through the app, and have customers checkout before they can use the app.

In this guide, we'll show you how to create an onboarding and checkout flow in the Next.js Supabase Starter Kit.

In this guide, we will cover:

1. Creating an onboarding flow when a user signs up.
2. Creating a multi-step form to have customers create a new Team Account
3. Creating a checkout flow to have customers pay before they can use the app.
4. Use Webhooks to update the user's record after they have paid.

Remember: you can customize the onboarding and checkout flow to fit your app's needs. This is a starting point, and you can build on top of it.

NB: **please make sure you have pullest the latest changes from the main branch before you start this guide.**

## Step 0: Adding an Onboarding Table

Before we create the onboarding flow, let's add a new table to store the onboarding data.

Create a new migration using the following command:

```bash
pnpm --filter web supabase migration new onboarding
```

This will create a new migration file at `apps/web/supabase/migrations/<timestamp>_onboarding.sql`. Open this file and add the following SQL code:

 ```sql {% title="apps/web/supabase/migrations/<timestamp>_onboarding.sql" %}
create table if not exists public.onboarding (
  id uuid primary key default uuid_generate_v4(),
  account_id uuid references public.accounts(id) not null unique,
  data jsonb default '{}',
  completed boolean default false,
  created_at timestamp with time zone default current_timestamp,
  updated_at timestamp with time zone default current_timestamp
);

revoke all on public.onboarding from public, service_role;

grant select, update, insert on public.onboarding to authenticated;
grant select, delete on public.onboarding to service_role;

alter table onboarding enable row level security;

create policy read_onboarding
    on public.onboarding
    for select
    to authenticated
    using (account_id = (select auth.uid()));

create policy insert_onboarding
    on public.onboarding
    for insert
    to authenticated
    with check (account_id = (select auth.uid()));

create policy update_onboarding
    on public.onboarding
    for update
    to authenticated
    using (account_id = (select auth.uid()))
    with check (account_id = (select auth.uid()));
```

This migration creates a new `onboarding` table with the following columns:
- `id`: A unique identifier for the onboarding record.
- `account_id`: A foreign key reference to the `accounts` table.
- `data`: A JSONB column to store the onboarding data.
- `completed`: A boolean flag to indicate if the onboarding is completed.
- `created_at` and `updated_at`: Timestamps for when the record was created and updated.

The migration also sets up row-level security policies to ensure that users can only access their own onboarding records.

Update your DB schema by running the following command:

```bash
pnpm run supabase:web:reset
```

And update your DB types by running the following command:

```bash
pnpm run supabase:web:typegen
```

Now that we have the `onboarding` table set up, let's create the onboarding flow.

## Step 1: Create the Onboarding Page

First, let's create the main onboarding page. This will be the entry point for our onboarding flow.

Create a new file at `apps/web/app/onboarding/page.tsx`:

 ```tsx {% title="apps/web/app/onboarding/page.tsx" %}
import { AppLogo } from '~/components/app-logo';
import { OnboardingForm } from './_components/onboarding-form';

function OnboardingPage() {
  return (
    <div className="flex h-screen flex-col items-center justify-center space-y-16">
      <AppLogo />

      <div>
        <OnboardingForm />
      </div>
    </div>
  );
}

export default OnboardingPage;
```

This page is simple. It displays your app logo and the `OnboardingForm` component, which we'll create next.

## Step 2: Create the Onboarding Form Schema

Before we create the form, let's define its schema. This will help us validate the form data.

Create a new file at `apps/web/app/onboarding/_lib/onboarding-form.schema.ts`:

 ```typescript {% title="apps/web/app/onboarding/_lib/onboarding-form.schema.ts" %}
import { z } from 'zod';

export const OnboardingFormSchema = z.object({
  profile: z.object({
    name: z.string().min(1).max(255),
  }),
  team: z.object({
    name: z.string().min(1).max(255),
  }),
  checkout: z.object({
    planId: z.string().min(1),
    productId: z.string().min(1),
  }),
});
```

This schema defines the structure of our onboarding form. It has three main sections: profile, team, and checkout.

## Step 3: Create the Onboarding Form Component

Now, let's create the main `OnboardingForm` component. This is where the magic happens!

Create a new file at `apps/web/app/onboarding/_components/onboarding-form.tsx`:

 ```tsx {% title="apps/web/app/onboarding/_components/onboarding-form.tsx" %}
'use client';

import { useCallback, useRef, useState } from 'react';

import { createPortal } from 'react-dom';

import dynamic from 'next/dynamic';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

import { PlanPicker } from '@kit/billing-gateway/components';
import { Button } from '@kit/ui/button';
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
} from '@kit/ui/form';
import { If } from '@kit/ui/if';
import { Input } from '@kit/ui/input';
import {
  MultiStepForm,
  MultiStepFormContextProvider,
  MultiStepFormHeader,
  MultiStepFormStep,
  useMultiStepFormContext,
} from '@kit/ui/multi-step-form';
import { Stepper } from '@kit/ui/stepper';

import billingConfig from '~/config/billing.config';
import { OnboardingFormSchema } from '~/onboarding/_lib/onboarding-form.schema';
import { submitOnboardingFormAction } from '~/onboarding/_lib/server/server-actions';

const EmbeddedCheckout = dynamic(
  async () => {
    const { EmbeddedCheckout } = await import('@kit/billing-gateway/checkout');

    return {
      default: EmbeddedCheckout,
    };
  },
  {
    ssr: false,
  },
);

export function OnboardingForm() {
  const [checkoutToken, setCheckoutToken] = useState<string | undefined>(
    undefined,
  );

  const form = useForm({
    resolver: zodResolver(OnboardingFormSchema),
    defaultValues: {
      profile: {
        name: '',
      },
      team: {
        name: '',
      },
      checkout: {
        planId: '',
        productId: '',
      },
    },
    mode: 'onBlur',
  });

  const onSubmit = useCallback(
    async (data: z.infer<typeof OnboardingFormSchema>) => {
      try {
        const { checkoutToken } = await submitOnboardingFormAction(data);

        setCheckoutToken(checkoutToken);
      } catch (error) {
        console.error('Failed to submit form:', error);
      }
    },
    [],
  );

  const checkoutPortalRef = useRef<HTMLDivElement>(null);

  if (checkoutToken) {
    return (
      <EmbeddedCheckout
        checkoutToken={checkoutToken}
        provider={billingConfig.provider}
        onClose={() => setCheckoutToken(undefined)}
      />
    );
  }

  return (
    <div
      className={
        'w-full rounded-lg p-8 shadow-sm duration-500 animate-in fade-in-90 zoom-in-95 slide-in-from-bottom-12 lg:border'
      }
    >
      <MultiStepForm
        className={'space-y-8 p-1'}
        schema={OnboardingFormSchema}
        form={form}
        onSubmit={onSubmit}
      >
        <MultiStepFormHeader>
          <MultiStepFormContextProvider>
            {({ currentStepIndex }) => (
              <Stepper
                variant={'numbers'}
                steps={['Profile', 'Team', 'Complete']}
                currentStep={currentStepIndex}
              />
            )}
          </MultiStepFormContextProvider>
        </MultiStepFormHeader>

        <MultiStepFormStep name={'profile'}>
          <ProfileStep />
        </MultiStepFormStep>

        <MultiStepFormStep name={'team'}>
          <TeamStep />
        </MultiStepFormStep>

        <MultiStepFormStep name={'checkout'}>
          <If condition={checkoutPortalRef.current}>
            {(portalRef) => createPortal(<CheckoutStep />, portalRef)}
          </If>
        </MultiStepFormStep>
      </MultiStepForm>

      <div className={'p-1'} ref={checkoutPortalRef}></div>
    </div>
  );
}

function ProfileStep() {
  const { nextStep, form } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex flex-col space-y-6'}>
        <div className={'flex flex-col space-y-2'}>
          <h1 className={'text-xl font-semibold'}>Welcome to Makerkit</h1>

          <p className={'text-sm text-muted-foreground'}>
            Welcome to the onboarding process! Let&apos;s get started by
            entering your name.
          </p>
        </div>

        <FormField
          render={({ field }) => {
            return (
              <FormItem>
                <FormLabel>Your Name</FormLabel>

                <FormControl>
                  <Input {...field} placeholder={'Name'} />
                </FormControl>

                <FormDescription>Enter your full name here</FormDescription>
              </FormItem>
            );
          }}
          name={'profile.name'}
        />

        <div className={'flex justify-end'}>
          <Button onClick={nextStep}>Continue</Button>
        </div>
      </div>
    </Form>
  );
}

function TeamStep() {
  const { nextStep, prevStep, form } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex w-full flex-col space-y-6'}>
        <div className={'flex flex-col space-y-2'}>
          <h1 className={'text-xl font-semibold'}>Create Your Team</h1>

          <p className={'text-sm text-muted-foreground'}>
            Let&apos;s create your team. Enter your team name below.
          </p>
        </div>

        <FormField
          render={({ field }) => {
            return (
              <FormItem>
                <FormLabel>Your Team Name</FormLabel>

                <FormControl>
                  <Input {...field} placeholder={'Name'} />
                </FormControl>

                <FormDescription>
                  This is the name of your team.
                </FormDescription>
              </FormItem>
            );
          }}
          name={'team.name'}
        />

        <div className={'flex justify-end space-x-2'}>
          <Button variant={'ghost'} onClick={prevStep}>
            Go Back
          </Button>

          <Button onClick={nextStep}>Continue</Button>
        </div>
      </div>
    </Form>
  );
}

function CheckoutStep() {
  const { form, mutation } = useMultiStepFormContext();

  return (
    <Form {...form}>
      <div className={'flex w-full flex-col space-y-6 lg:min-w-[55rem]'}>
        <div className={'flex flex-col space-y-2'}>
          <PlanPicker
            pending={mutation.isPending}
            config={billingConfig}
            onSubmit={({ planId, productId }) => {
              form.setValue('checkout.planId', planId);
              form.setValue('checkout.productId', productId);

              mutation.mutate();
            }}
          />
        </div>
      </div>
    </Form>
  );
}
```

This component creates a multi-step form for the onboarding process. It includes steps for profile information, team creation, and plan selection.

## Step 4: Create the Server Action

Now, let's create the server action that will handle the form submission.

Create a new file at `apps/web/app/onboarding/_lib/server/server-actions.ts`:

 ```typescript {% title="apps/web/app/onboarding/_lib/server/server-actions.ts" %}
'use server';

import { redirect } from 'next/navigation';

import { createBillingGatewayService } from '@kit/billing-gateway';
import { enhanceAction } from '@kit/next/actions';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

import appConfig from '~/config/app.config';
import billingConfig from '~/config/billing.config';
import pathsConfig from '~/config/paths.config';
import { OnboardingFormSchema } from '~/onboarding/_lib/onboarding-form.schema';

export const submitOnboardingFormAction = enhanceAction(
  async (data, user) => {
    const logger = await getLogger();

    logger.info({ userId: user.id }, `Submitting onboarding form...`);

    const isOnboarded = user.app_metadata.onboarded === true;

    if (isOnboarded) {
      logger.info(
        { userId: user.id },
        `User is already onboarded. Redirecting...`,
      );

      redirect(pathsConfig.app.home);
    }

    const client = getSupabaseServerClient();

    const createTeamResponse = await client
      .from('accounts')
      .insert({
        name: data.team.name,
        primary_owner_user_id: user.id,
        is_personal_account: false,
      })
      .select('id')
      .single();

    if (createTeamResponse.error) {
      logger.error(
        {
          error: createTeamResponse.error,
        },
        `Failed to create team`,
      );

      throw createTeamResponse.error;
    } else {
        logger.info(
            { userId: user.id, teamId: createTeamResponse.data.id },
            `Team created. Creating onboarding data...`,
        );
    }

    const response = await client.from('onboarding').upsert(
      {
        account_id: user.id,
        data: {
          userName: data.profile.name,
          teamAccountId: createTeamResponse.data.id,
        },
        completed: true,
      },
      {
        onConflict: 'account_id',
      },
    );

    if (response.error) {
      throw response.error;
    }

    logger.info(
      { userId: user.id, teamId: createTeamResponse.data.id },
      `Onboarding data created. Creating checkout session...`,
    );

    const billingService = createBillingGatewayService(billingConfig.provider);

    const { plan } = getPlanDetails(
      data.checkout.productId,
      data.checkout.planId,
    );

    const returnUrl = new URL('/onboarding/complete', appConfig.url).href;

    const checkoutSession = await billingService.createCheckoutSession({
      returnUrl,
      customerEmail: user.email,
      accountId: createTeamResponse.data.id,
      plan,
      variantQuantities: [],
      metadata: {
        source: 'onboarding',
        userId: user.id,
      },
    });

    return {
      checkoutToken: checkoutSession.checkoutToken,
    };
  },
  {
    auth: true,
    schema: OnboardingFormSchema,
  },
);

function getPlanDetails(productId: string, planId: string) {
  const product = billingConfig.products.find(
    (product) => product.id === productId,
  );

  if (!product) {
    throw new Error('Product not found');
  }

  const plan = product?.plans.find((plan) => plan.id === planId);

  if (!plan) {
    throw new Error('Plan not found');
  }

  return { plan, product };
}
```

This server action handles the form submission, inserts the onboarding data into Supabase, create a team (so we can assign it a subscription), and creates a checkout session for the selected plan.

Once the checkout is completed, the user will be redirected to the `/onboarding/complete` page. This page will be created in the next step.

## Step 6: Enhancing the Stripe Webhook Handler

This change extends the functionality of the Stripe webhook handler to complete the onboarding process after a successful checkout. Here's what's happening:

In the `handleCheckoutSessionCompleted` method, we add new logic to handle onboarding-specific actions.

First, define the `completeOnboarding` function to process the onboarding data and create a team based on the user's input.

NB: this is valid for Stripe, but you can adapt it to any other payment provider.

 ```typescript {% title="packages/billing/stripe/src/services/stripe-webhook-handler.service.ts" %}
async function completeOnboarding(accountId: string) {
  const logger = await getLogger();
  const adminClient = getSupabaseServerAdminClient();

  logger.info(
    { accountId },
    `Checkout comes from onboarding. Processing onboarding data...`,
  );

  const onboarding = await adminClient
    .from('onboarding')
    .select('*')
    .eq('account_id', accountId)
    .single();

  if (onboarding.error) {
    logger.error(
      { error: onboarding.error, accountId },
      `Failed to retrieve onboarding data`,
    );

    // if there's an error, we can't continue
    return;
  } else {
    logger.info({ accountId }, `Onboarding data retrieved. Processing...`);

    const data = onboarding.data.data as {
      userName: string;
      teamAccountId: string;
    };

    const teamAccountId = data.teamAccountId;

    logger.info(
      { userId: accountId, teamAccountId },
      `Assigning membership...`,
    );

    const assignMembershipResponse = await adminClient
      .from('accounts_memberships')
      .insert({
        account_id: teamAccountId,
        user_id: accountId,
        account_role: 'owner',
      });

    if (assignMembershipResponse.error) {
      logger.error(
        {
          error: assignMembershipResponse.error,
        },
        `Failed to assign membership`,
      );
    } else {
      logger.info({ accountId }, `Membership assigned. Updating account...`);
    }

    const accountResponse = await adminClient
      .from('accounts')
      .update({
        name: data.userName,
      })
      .eq('id', accountId);

    if (accountResponse.error) {
      logger.error(
        {
          error: accountResponse.error,
        },
        `Failed to update account`,
      );
    } else {
      logger.info(
        { accountId },
        `Account updated. Cleaning up onboarding data...`,
      );
    }

    // set onboarded flag on user account
    const updateUserResponse = await adminClient.auth.admin.updateUserById(
      accountId,
      {
        app_metadata: {
          onboarded: true,
        },
      },
    );

    if (updateUserResponse.error) {
      logger.error(
        {
          error: updateUserResponse.error,
        },
        `Failed to update user`,
      );
    } else {
      logger.info({ accountId }, `User updated. Cleaning up...`);
    }

    // clean up onboarding data
    const deleteOnboardingResponse = await adminClient
      .from('onboarding')
      .delete()
      .eq('account_id', accountId);

    if (deleteOnboardingResponse.error) {
      logger.error(
        {
          error: deleteOnboardingResponse.error,
        },
        `Failed to delete onboarding data`,
      );
    } else {
      logger.info(
        { accountId },
        `Onboarding data cleaned up. Completed webhook handler.`,
      );
    }
  }
}
```

Now, we handle this function in the `handleCheckoutSessionCompleted` method, right before the `onCheckoutCompletedCallback` is called.

 ```typescript {% title="packages/billing/stripe/src/services/stripe-webhook-handler.service.ts" %}
// ...

const subscriptionData =
  await stripe.subscriptions.retrieve(subscriptionId);

const metadata = subscriptionData.metadata as {
  source: string;
  userId: string;
} | undefined;

// if the checkout comes from onboarding
// we need to complete the onboarding process
if (metadata?.source === 'onboarding') {
  const userId = metadata.userId;

  await completeOnboarding(userId);
}

return onCheckoutCompletedCallback(payload);
```

This enhanced webhook handler completes the onboarding process by creating a team account, updating the user's personal account, and marking the user as "onboarded" in their Supabase user metadata.

## Step 7: Handling the Onboarding Completion Page

The checkout will rediret to the `/onboarding/complete` page after the onboarding process is completed. This is because there is no telling when the webhook will be triggered.

In this page, we will start fetching the user data and verify if the user has been onboarded.

If the user has been onboarded correctly, we will redirect the user to the `/home` page. If not, we will show a loading spinner and keep checking the user's onboarded status until it is true.

 ```tsx {% title="/apps/web/app/onboarding/complete/page.tsx" %}
'use client';

import { useRef } from 'react';

import { useQuery } from '@tanstack/react-query';

import { useSupabase } from '@kit/supabase/hooks/use-supabase';
import { LoadingOverlay } from '@kit/ui/loading-overlay';

import pathsConfig from '~/config/paths.config';

export default function OnboardingCompletePage() {
  const { error } = useCheckUserOnboarded();

  if (error) {
    return (
      <div className={'flex flex-col items-center justify-center'}>
        <p>Something went wrong...</p>
      </div>
    );
  }

  return <LoadingOverlay>Setting up your account...</LoadingOverlay>;
}

/**
 * @description
 * This function checks if the user is onboarded
 * If the user is onboarded, it redirects them to the home page
 * it retries every second until the user is onboarded
 */
function useCheckUserOnboarded() {
  const client = useSupabase();
  const countRef = useRef(0);
  const maxCount = 10;
  const error = countRef.current >= maxCount;

  useQuery({
    queryKey: ['onboarding-complete'],
    refetchInterval: () => (error ? false : 1000),
    queryFn: async () => {
      if (error) {
        return false;
      }

      countRef.current++;

      const response = await client.auth.getUser();

      if (response.error) {
        throw response.error;
      }

      const onboarded = response.data.user.app_metadata.onboarded;

      // if the user is onboarded, redirect them to the home page
      if (onboarded) {
        return window.location.assign(pathsConfig.app.home);
      }

      return false;
    },
  });

  return {
    error,
  };
}
```

This page

## What This Means for Your Onboarding Flow

With these changes, your onboarding process now includes these additional steps:

1. When a user completes the checkout during onboarding, it triggers this enhanced webhook handler.
2. The handler retrieves the onboarding data that was saved earlier in the process.
3. It creates a new team account with the name provided during onboarding.
4. It updates the user's personal account with their name.
5. Finally, it marks the user as "onboarded" in their Supabase user metadata.

This completes the onboarding process, ensuring that all the information collected during onboarding is properly saved and the user's account is fully set up.

## Step 8: Update Your App's Routing

To integrate this onboarding flow into your app, you'll need to update your routing logic.

We can do this in the middleware, in the logic branch that handles the `/home` route. If the user is not logged in, we'll redirect them to the sign-in page. If the user is logged in but has not completed onboarding, we'll redirect them to the onboarding flow.

Update the `apps/web/middleware.ts` file:

 ```typescript {% title="apps/web/middleware.ts" %}
{
  pattern: new URLPattern({ pathname: '/home/*?' }),
  handler: async (req: NextRequest, res: NextResponse) => {
    const {
      data: { user },
    } = await getUser(req, res);

    const origin = req.nextUrl.origin;
    const next = req.nextUrl.pathname;

    // If user is not logged in, redirect to sign in page.
    if (!user) {
      const signIn = pathsConfig.auth.signIn;
      const redirectPath = `${signIn}?next=${next}`;

      return NextResponse.redirect(new URL(redirectPath, origin).href);
    }

    // verify if user has completed onboarding
    const isOnboarded = user?.app_metadata.onboarded;

    // If user is logged in but has not completed onboarding,
    if (!isOnboarded) {
      return NextResponse.redirect(new URL('/onboarding', origin).href);
    }

    const supabase = createMiddlewareClient(req, res);

    const requiresMultiFactorAuthentication =
      await checkRequiresMultiFactorAuthentication(supabase);

    // If user requires multi-factor authentication, redirect to MFA page.
    if (requiresMultiFactorAuthentication) {
      return NextResponse.redirect(
        new URL(pathsConfig.auth.verifyMfa, origin).href,
      );
    }
  },
}
```

This code checks if the user is logged in and has completed onboarding. If the user has not completed onboarding, they are redirected to the onboarding flow.

Also add the following pattern to make sure the user is authenticated when visiting the `/onboarding` route:

```typescript {% title="apps/web/middleware.ts" %}
{
  pattern: new URLPattern({ pathname: '/onboarding/*?' }),
  handler: async (req: NextRequest, res: NextResponse) => {
    const {
      data: { user },
    } = await getUser(req, res);

    // If user is not logged in, redirect to sign in page.
    if (!user) {
      const signIn = pathsConfig.auth.signIn;
      const redirectPath = `${signIn}?next=${next}`;

      return NextResponse.redirect(new URL(redirectPath, origin).href);
    }

    const supabase = createMiddlewareClient(req, res);

    const requiresMultiFactorAuthentication =
      await checkRequiresMultiFactorAuthentication(supabase);

    // If user requires multi-factor authentication, redirect to MFA page.
    if (requiresMultiFactorAuthentication) {
      return NextResponse.redirect(
        new URL(pathsConfig.auth.verifyMfa, origin).href,
      );
    }
  },
}
```

### Marking invited users as onboarded

When a user gets invited to a team account, you don't want to show them the onboarding flow again. You can use the `onboarded` property to mark the user as onboarded.

Update the `packages/features/team-accounts/src/server/actions/team-invitations-server-actions.ts` server action `acceptInvitationAction` at line 125 (eg. before increasing seats):

```tsx
// mark user as onboarded
await adminClient.auth.admin.updateUserById(user.id, {
  app_metadata: {
    onboarded: true,
  },
});
```

In this way, the user will be redirected to the `/home` page after accepting the invite.

### Considerations

Remember, the user can always unsubscribe from the plan they selected during onboarding. You should handle this case in your billing system if your app always requires a plan to be active.

## Conclusion

That's it! You've now added an onboarding flow to your Makerkit Next.js Supabase Turbo project.

This flow includes:

1. A profile information step
2. A team creation step
3. A plan selection step
4. Integration with your billing system

Remember to style your components, handle errors gracefully, and test thoroughly. Happy coding! 🚀


-----------------
FILE PATH: kits\next-supabase-turbo\recipes\projects-data-model.mdoc

---
status: "published"
title: 'Adding a Projects Data Model to Your Next.js Supabase Application'
label: 'Projects Data Model'
order: 4
description: 'Learn how to add a Projects data model to your Next.js Supabase application using Supabase'
---

Team Accounts allow you to manage several members in a single account. These usually share a subscription and other common attributes. However, you may need to go deeper than that, and allow team accounts to have a set of projects that they can collaborate on.

This guide will walk you through implementing a robust project system in your Makerkit application using Supabase. We'll cover everything from setting up the necessary database tables to implementing role-based access control.

## Overview of the Projects Data Model

The Projects feature allows users to create and manage projects within your application. It includes:

- Project creation and management
- Role-based access control (owner, admin, member)
- Permissions system
- Database schema and functions

Let's dive in!

## Database Schema and Functions

We'll start by setting up the database schema. This involves creating tables, enums, and functions.

### Enums

First, let's create two enum types:

```sql
-- Project roles
CREATE TYPE public.project_role AS ENUM ('owner', 'admin', 'member');

-- Project actions
CREATE TYPE public.project_action AS ENUM (
  'view_project',
  'edit_project',
  'delete_project',
  'invite_member',
  'remove_member'
);
```

These enums define the possible roles and actions within a project.

### Tables

Now, let's create the main tables.

We will create two tables: `projects` and `project_members`.

The `projects` table stores project information, while the `project_members` table manages the relationship between projects and users.

### Projects Table

```sql
-- Projects table
CREATE TABLE IF NOT EXISTS public.projects (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  account_id UUID NOT NULL REFERENCES public.accounts(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Let's dive into the `projects` table:

- `id` is a unique identifier for each project.
- `name` is the name of the project.
- `description` is an optional description for the project.
- `account_id` is a foreign key referencing the `accounts` table.
- `created_at` and `updated_at` are timestamps for when the project was created and updated, respectively.

### Project Members Table

The `project_members` table stores the relationship between projects and users:

```sql
-- Project members table
CREATE TABLE IF NOT EXISTS public.project_members (
  project_id UUID NOT NULL REFERENCES public.projects(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  role public.project_role NOT NULL DEFAULT 'member',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (project_id, user_id)
);
```

Let's clarify the `project_members` table:

- The `project_id` column is a foreign key referencing the `projects` table.
- The `user_id` column is a foreign key referencing the `auth.users` table.
- The `role` column is an enum type that can have three values: `owner`, `admin`, or `member`.

These columns ensure that each project has a unique owner, and that each member can have a role of `owner`, `admin`, or `member`.

### Indexes

To optimize query performance, let's add some indexes:

```sql
CREATE INDEX projects_account_id ON public.projects (account_id);
CREATE INDEX project_members_project_id ON public.project_members (project_id);
CREATE INDEX project_members_user_id ON public.project_members (user_id);
```

We also need a unique index to ensure only one owner per project:

```sql
CREATE UNIQUE INDEX projects_unique_owner ON public.project_members (project_id) WHERE role = 'owner';
```

### Permissions

Set up default permissions. This is useful to make sure we can granularly control access to the tables.

```sql
alter table public.projects enable row level security;
alter table public.project_members enable row level security;

-- Revoke all permissions by default
REVOKE ALL ON public.projects FROM public, service_role;
REVOKE ALL ON public.project_members FROM public, service_role;

-- Grant access to authenticated users
GRANT SELECT, INSERT, UPDATE, DELETE ON public.projects TO authenticated;
GRANT SELECT, INSERT, UPDATE, DELETE ON public.project_members TO authenticated;
```

## Functions

Now, let's create several helper functions to manage projects and permissions.

### Is Project Member

This function checks if a user is a member of a specific project:

```sql
CREATE OR REPLACE FUNCTION public.is_project_member(p_project_id UUID)
RETURNS BOOLEAN
SET search_path = ''
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.project_members
    WHERE project_id = p_project_id
    AND user_id = (SELECT auth.uid())
  );
$$ LANGUAGE sql SECURITY DEFINER;

GRANT EXECUTE ON FUNCTION public.is_project_member(UUID) TO authenticated;
```

### Is Project Admin

This function checks if a user is an admin or owner of a specific project:

```sql
CREATE OR REPLACE FUNCTION public.is_project_admin(p_project_id UUID)
RETURNS BOOLEAN
SET search_path = ''
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.project_members
    WHERE project_id = p_project_id
    AND user_id = (SELECT auth.uid())
    AND role IN ('owner', 'admin')
  );
$$ LANGUAGE sql;

GRANT EXECUTE ON FUNCTION public.is_project_admin TO authenticated;
```

### Is Project Owner

This function checks if a user is the owner of a project:

```sql
CREATE OR REPLACE FUNCTION public.is_project_owner(project_id UUID)
RETURNS BOOLEAN
SET search_path = ''
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.project_members
    WHERE project_id = $1
    AND user_id = (SELECT auth.uid())
    AND role = 'owner'
  );
$$ LANGUAGE sql;

GRANT EXECUTE ON FUNCTION public.is_project_owner TO authenticated;
```

### User Has Project Permission

This function checks if a user has the required permissions to perform a specific action on a project:

```sql
CREATE OR REPLACE FUNCTION public.user_has_project_permission(
  p_user_auth_id UUID,
  p_project_id UUID,
  p_action public.project_action
)
RETURNS BOOLEAN
SET search_path = ''
AS $$
DECLARE
  v_role public.project_role;
BEGIN
  -- First, check if the user is a member of the project
  SELECT role INTO v_role
  FROM public.project_members
  WHERE project_id = p_project_id AND user_id = p_user_auth_id;

  IF v_role IS NULL THEN
    RETURN FALSE;
  END IF;

  -- Check permissions based on role and action
  CASE v_role
    WHEN 'owner' THEN
      RETURN TRUE;  -- Owners can do everything
    WHEN 'admin' THEN
      RETURN p_action != 'delete_project';  -- Admins can do everything except delete the project
    WHEN 'member' THEN
      RETURN p_action IN ('view_project');
    ELSE
      RAISE EXCEPTION 'User must be a member of the project to perform this action';
  END CASE;
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION public.user_has_project_permission TO authenticated;
```

In the above, we use the `CASE` statement to check the role of the user and the action they are trying to perform. If the user is a member of the project and has the required permissions, we return `TRUE`. Otherwise, we raise an exception.

Please feel free to modify the `user_has_project_permission` function to fit your specific use case and your project's requirements.For example, you may want to add more roles or actions, or restrict the permissions based on the project's settings.

### Current User Has Project Permission

This function is a wrapper around `user_has_project_permission` for the current user:

```sql
CREATE OR REPLACE FUNCTION public.current_user_has_project_permission(
  p_project_id UUID,
  p_action public.project_action
)
RETURNS BOOLEAN
SET search_path = ''
AS $$
DECLARE
  v_role public.project_role;
BEGIN
  SELECT public.user_has_project_permission((SELECT auth.uid()), p_project_id, p_action);
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION public.current_user_has_project_permission TO authenticated;
```

### Current User Can Manage Project Member

This function checks if the current user can manage another user in a project:

```sql
CREATE OR REPLACE FUNCTION public.current_user_can_manage_project_member(
  p_target_member_role public.project_role,
  p_project_id UUID
)
RETURNS BOOLEAN
SET search_path = ''
AS $$
DECLARE
  v_current_user_role public.project_role;
BEGIN
  SELECT role INTO v_current_user_role
  FROM public.project_members
  WHERE project_id = p_project_id AND user_id = (SELECT auth.uid());

  IF v_current_user_role IS NULL OR p_target_member_role IS NULL THEN
    RAISE EXCEPTION 'User not found';
  END IF;

  -- Check if the manager has a higher role
  RETURN (v_current_user_role = 'owner' AND p_target_member_role != 'owner') OR
         (v_current_user_role = 'admin' AND p_target_member_role = 'member');
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION public.current_user_can_manage_project_member TO authenticated;
```

This project is extremely useful to verify that a user can perform actions that affect another user in a project.

### Update Project Member Role

This function updates the role of a project member:

```sql
CREATE OR REPLACE FUNCTION public.update_project_member_role(
  p_user_id UUID,
  p_new_role public.project_role,
  p_project_id UUID
)
RETURNS BOOLEAN
SET search_path = ''
AS $$
DECLARE
  v_current_role public.project_role;
BEGIN
  -- Get the current role of the member
  SELECT role INTO v_current_role
  FROM public.project_members
  WHERE project_id = p_project_id AND user_id = p_user_id;

  -- Check if the manager can manage this member
  IF NOT public.current_user_can_manage_project_member(v_current_role, p_project_id) THEN
    RAISE EXCEPTION 'Permission denied';
  END IF;

  IF p_new_role = 'owner' THEN
    RAISE EXCEPTION 'Owner cannot be updated to a different role';
  END IF;

  -- Update the member's role
  UPDATE public.project_members
  SET role = p_new_role
  WHERE project_id = p_project_id AND user_id = p_user_id;

  RETURN TRUE;
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION public.update_project_member_role TO authenticated;
```

### Can Edit Project

This function checks if a user can edit a project:

```sql
CREATE OR REPLACE FUNCTION public.can_edit_project(p_user_auth_id UUID, p_project_id UUID)
RETURNS BOOLEAN
SET search_path = ''
AS $$
  SELECT public.user_has_project_permission(p_user_auth_id, p_project_id, 'edit_project'::public.project_action);
$$ LANGUAGE sql;

GRANT EXECUTE ON FUNCTION public.can_edit_project TO authenticated;
```

This is a simple wrapper around the `user_has_project_permission` function that checks if the current user can edit a project.

### Can Delete Project

This function checks if a user can delete a project:

```sql
CREATE OR REPLACE FUNCTION public.can_delete_project(p_user_auth_id UUID, p_project_id UUID)
RETURNS BOOLEAN
SET search_path = ''
AS $$
  SELECT public.user_has_project_permission(p_user_auth_id, p_project_id, 'delete_project'::public.project_action);
$$ LANGUAGE sql;

GRANT EXECUTE ON FUNCTION public.can_delete_project TO authenticated;
```

This is a simple wrapper around the `user_has_project_permission` function that checks if the current user can delete a project.

### Can Invite Project Member

This function checks if a user can invite a new member to the project:

```sql
CREATE OR REPLACE FUNCTION public.can_invite_project_member(p_user_auth_id UUID, p_project_id UUID)
RETURNS BOOLEAN
SET search_path = ''
AS $$
  SELECT public.user_has_project_permission(p_user_auth_id, p_project_id, 'invite_member'::public.project_action);
$$ LANGUAGE sql;

GRANT EXECUTE ON FUNCTION public.can_invite_project_member TO authenticated;
```

This is also a simple wrapper around the `user_has_project_permission` function that checks if the current user can invite a new member to the project.

## Row Level Security (RLS) Policies

Now, let's set up RLS policies to secure our tables. RLS policies are used to control access to specific columns and rows in a table. The functions we defined earlier are used to check if a user has the required permissions to perform a specific action on a project.

### Projects Table Policies

```sql
-- SELECT
CREATE POLICY select_projects
  ON public.projects
  FOR SELECT
  TO authenticated
  USING (
    public.is_project_member(id)
  );

-- INSERT
CREATE POLICY insert_new_project
  ON public.projects
  FOR INSERT
  TO authenticated
  WITH CHECK (
    public.has_role_on_account(account_id)
  );

-- DELETE
CREATE POLICY delete_project
  ON public.projects
  FOR DELETE
  TO authenticated
  USING (
    public.can_delete_project((SELECT auth.uid()), id)
  );

-- UPDATE
CREATE POLICY update_project
  ON public.projects
  FOR UPDATE
  TO authenticated
  USING (
    public.can_edit_project((SELECT auth.uid()), id)
  )
  WITH CHECK (
    public.can_edit_project((SELECT auth.uid()), id)
  );
```

Alright, let's break down the policies:

1. **Select Projects**: This policy allows authenticated users to select projects from the `projects` table. If a user is not a member of the project, they will not be able to see the project.
2. **Insert New Project**: This policy allows authenticated users to insert new projects into the `projects` table. It does so by checking if the current user is currently a member of the team account associated with the project.
3. **Delete Project**: This policy allows authenticated users to delete projects from the `projects` table. It verifies that the user can delete the project using the `can_delete_project` function.
4. **Update Project**: This policy allows authenticated users to update projects in the `projects` table. It does so by checking if the current user has the required permissions to edit the project using the `can_edit_project` function.

### Project Members Table Policies

```sql
-- SELECT
CREATE POLICY select_project_members
  ON public.project_members
  FOR SELECT
  TO authenticated
  USING (
    public.is_project_member(project_id)
  );

-- INSERT
CREATE POLICY insert_project_member
  ON public.project_members
  FOR INSERT
  TO authenticated
  WITH CHECK (
    public.can_invite_project_member(
      (SELECT auth.uid()),
      project_id
    )
  );

-- UPDATE
CREATE POLICY update_project_members
  ON public.project_members
  FOR UPDATE
  TO authenticated
  USING (
    public.current_user_can_manage_project_member(
      role,
      project_id
    )
  )
  WITH CHECK (
    public.current_user_can_manage_project_member(
      role,
      project_id
    )
  );

-- DELETE
CREATE POLICY delete_project_members
  ON public.project_members
  FOR DELETE
  TO authenticated
  USING (
    public.current_user_can_manage_project_member(
      role,
      project_id
    )
  );
```

Let's break down the policies:

1. **Select Project Members**: This policy allows authenticated users to select project members from the `project_members` table. If a user is not a member of the project, they will not be able to see the project members.
2. **Insert Project Member**: This policy allows authenticated users to insert new project members into the `project_members` table. It does so by checking if the current user can invite a new member to the project using the `can_invite_project_member` function.
3. **Update Project Members**: This policy allows authenticated users to update project members in the `project_members` table. It does so by checking if the current user can manage the project member using the `current_user_can_manage_project_member` function.
4. **Delete Project Members**: This policy allows authenticated users to delete project members from the `project_members` table. It does so by checking if the current user can manage the project member using the `current_user_can_manage_project_member` function.

## Additional Functions

Let's add a few more helpful functions:

### Add Project Owner

This function adds the owner of the project as the first project member:

```sql
CREATE OR REPLACE FUNCTION kit.add_project_owner()
RETURNS TRIGGER
AS $$
BEGIN
  INSERT INTO public.project_members (project_id, user_id, role)
  VALUES (NEW.id, auth.uid(), 'owner'::public.project_role);

  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger to add owner of the project creator as the first project member
CREATE TRIGGER add_project_owner_on_insert
    AFTER INSERT ON public.projects
    FOR EACH ROW
    EXECUTE PROCEDURE kit.add_project_owner();
```

### Add Project Member

This function adds a new member to a project:

```sql
CREATE OR REPLACE FUNCTION public.add_project_member(
  p_project_id UUID,
  p_user_id UUID,
  p_role public.project_role DEFAULT 'member'
) RETURNS BOOLEAN
SET search_path = ''
AS $$
DECLARE
  v_account_id UUID;
BEGIN
  -- Check if the current user has permission to add members
  IF NOT public.is_project_admin(p_project_id) OR p_role = 'owner' THEN
    RAISE EXCEPTION 'Permission denied';
  END IF;

  -- Get the account_id for the project
  SELECT account_id INTO v_account_id
  FROM public.projects
  WHERE id = p_project_id;

  -- Check if the user is a member of the team account
  IF NOT EXISTS (
    SELECT 1 FROM public.accounts_memberships
    WHERE account_id = v_account_id AND user_id = p_user_id
  ) THEN
    RAISE EXCEPTION 'User is not a member of the team account';
  END IF;

  -- Add the new member (the trigger will enforce the team membership check)
  INSERT INTO public.project_members (project_id, user_id, role)
  VALUES (p_project_id, p_user_id, p_role)
  ON CONFLICT (project_id, user_id) DO UPDATE
  SET role = EXCLUDED.role;

  RETURN TRUE;
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION public.add_project_member TO authenticated;
```

### Check Project Member in Team

This trigger function ensures that a user being added to a project is already a member of the associated team account:

```sql
CREATE OR REPLACE FUNCTION kit.check_project_member_in_team()
RETURNS TRIGGER
AS $$
DECLARE
  v_account_id UUID;
BEGIN
  SELECT account_id FROM public.projects
  WHERE id = NEW.project_id
  INTO v_account_id;

  IF NOT EXISTS (
    SELECT 1 FROM public.accounts_memberships
    WHERE account_id = v_account_id AND user_id = NEW.user_id
  ) THEN
    RAISE EXCEPTION 'User must be a member of the team account to be added to the project';
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Create a trigger that uses the above function
CREATE TRIGGER ensure_project_member_in_team
BEFORE INSERT OR UPDATE ON public.project_members
FOR EACH ROW EXECUTE FUNCTION kit.check_project_member_in_team();
```

## Implementing the Projects Feature

Now that we've set up all the necessary database objects, let's walk through how to use this Projects feature in your Makerkit application.

### Creating a New Project

To create a new project, you'll insert a row into the `public.projects` table. The `kit.add_project_owner()` trigger will automatically add the creating user as the owner.

```sql
INSERT INTO public.projects (name, description, account_id)
VALUES ('My New Project', 'This is a description of my project', :account_id);
```

Replace `:account_id` with the actual account ID.

### Adding Members to a Project

Use the `public.add_project_member` function to add new members to a project:

```sql
SELECT public.add_project_member(:project_id, :user_id, 'member'::public.project_role);
```

Replace `:project_id` and `:user_id` with actual values. The role can be 'member' or 'admin'.

### Updating a Member's Role

To change a member's role, use the `public.update_project_member_role` function:

```sql
SELECT public.update_project_member_role(:user_id, 'admin'::public.project_role, :project_id);
```

### Querying Projects

To get all projects a user is a member of:

```sql
SELECT * FROM public.projects
WHERE id IN (
  SELECT project_id
  FROM public.project_members
  WHERE user_id = auth.uid()
);
```

### Checking Permissions

You can use the various permission-checking functions in your application logic. For example:

```sql
-- Check if the current user can edit a project
SELECT public.can_edit_project(auth.uid(), :project_id);

-- Check if the current user can delete a project
SELECT public.can_delete_project(auth.uid(), :project_id);

-- Check if the current user can invite members to a project
SELECT public.can_invite_project_member(auth.uid(), :project_id);
```

## Application-Level Logic

Now that we've covered the database schema and functions, let's move on to the application-level logic.

### The ProjectsService

To provide a better API for interacting with the database, we can create a service layer (`ProjectsService`) that encapsulates all the database operations related to projects. This is a great practice as it separates concerns and makes your code more maintainable.

The `ProjectsService` class encapsulates all the database operations related to projects. This is a great practice as it separates concerns and makes your code more maintainable.

 ```tsx {% title="apps/web/lib/server/projects/projects.service.ts" %}
import { SupabaseClient } from '@supabase/supabase-js';

import { Database } from '~/lib/database.types';

type ProjectAction = Database['public']['Enums']['project_action'];

export function createProjectsService(client: SupabaseClient<Database>) {
  return new ProjectsService(client);
}

class ProjectsService {
  constructor(private readonly client: SupabaseClient<Database>) {}

  async createProject(params: {
    name: string;
    description?: string;
    accountId: string;
  }) {
    const { data, error } = await this.client
      .from('projects')
      .insert({
        name: params.name,
        description: params.description,
        account_id: params.accountId,
      })
      .select('id')
      .single();

    if (error) {
      throw error;
    }

    return data;
  }

  async getProjects(accountSlug: string) {
    const { data, error } = await this.client
      .from('projects')
      .select('*, account: account_id ! inner (slug)')
      .eq('account.slug', accountSlug);

    if (error) {
      throw error;
    }

    return data;
  }

  async getProjectMembers(projectId: string) {
    const { data, error } = await this.client
      .from('project_members')
      .select('*')
      .eq('project_id', projectId);

    if (error) {
      throw error;
    }

    return data;
  }

  async getProject(projectId: string) {
    const { data, error } = await this.client
      .from('projects')
      .select('*')
      .eq('id', projectId)
      .single();

    if (error) {
      throw error;
    }

    return data;
  }

  async hasPermission(params: { projectId: string; action: ProjectAction }) {
    const { data, error } = await this.client.rpc(
      'current_user_has_project_permission',
      {
        p_project_id: params.projectId,
        p_action: params.action,
      },
    );

    if (error) {
      throw error;
    }

    return data;
  }

  async addProjectMember(params: {
    projectId: string;
    userId: string;
    role?: 'member' | 'admin';
  }) {
    const { error } = await this.client.rpc('add_project_member', {
      p_project_id: params.projectId,
      p_user_id: params.userId,
      p_role: params.role ?? 'member',
    });

    if (error) {
      throw error;
    }

    return true;
  }
}
```

Let's look at some key methods:

1. **createProject**: This method inserts a new project into the `projects` table.
2. **getProjects**: Fetches all projects for a given account.
3. **getProjectMembers**: Retrieves all members of a specific project.
4. **getProject**: Fetches details of a single project.
5. **hasPermission**: Checks if the current user has a specific permission for a project.
6. **addProjectMember**: Adds a new member to a project.

These methods utilize the Supabase client to interact with the database, leveraging the SQL functions and RLS policies we set up earlier.

### Using the ProjectsService in a Next.js Page

The `ProjectsPage` component demonstrates how to use the `ProjectsService` in a Next.js server component:

1. We create an instance of `ProjectsService` using the Supabase client:

   ```typescript
   const client = getSupabaseServerComponentClient();
   const service = createProjectsService(client);
   ```

2. We fetch the projects for the current account:

   ```typescript
   const projects = use(service.getProjects(params.account));
   ```

   The `use` function is a React hook that allows us to use async data in a synchronous-looking way in server components.

3. We render the projects, showing an empty state if there are no projects:

   ```tsx
   <If condition={projects.length === 0}>
     <EmptyState>
       {/* Empty state content */}
     </EmptyState>
   </If>

   <div className={'grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4'}>
     {projects.map((project) => (
       <CardButton key={project.id} asChild>
         <Link href={`/home/${params.account}/projects/${project.id}`}>
           {project.name}
         </Link>
       </CardButton>
     ))}
   </div>
   ```

Here is a page that shows a list of projects for a given account:

 ```tsx {% title="apps/web/app/home/[account]/projects/page.tsx" %}
import { use } from 'react';

import Link from 'next/link';

import { getSupabaseServerComponentClient } from '@kit/supabase/server-component-client';
import { Button } from '@kit/ui/button';
import { CardButton } from '@kit/ui/card-button';
import {
  EmptyState,
  EmptyStateButton,
  EmptyStateHeading,
  EmptyStateText,
} from '@kit/ui/empty-state';
import { If } from '@kit/ui/if';
import { PageBody, PageHeader } from '@kit/ui/page';

import { createProjectsService } from '~/lib/server/projects/projects.service';

interface ProjectsPageProps {
  params: {
    account: string;
  };
}

export default function ProjectsPage({ params }: ProjectsPageProps) {
  const client = getSupabaseServerComponentClient();
  const service = createProjectsService(client);

  const projects = use(service.getProjects(params.account));

  return (
    <>
      <PageHeader title="Projects" description="Manage your team's projects">
        <Link href={`/home/${params.account}/projects/new`}>
          <Button>New Project</Button>
        </Link>
      </PageHeader>

      <PageBody>
        <If condition={projects.length === 0}>
          <EmptyState>
            <EmptyStateHeading>No projects found</EmptyStateHeading>

            <EmptyStateText>
              You still have not created any projects. Create your first project
              now!
            </EmptyStateText>

            <EmptyStateButton>Create Project</EmptyStateButton>
          </EmptyState>
        </If>

        <div className={'grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4'}>
          {projects.map((project) => (
            <CardButton key={project.id} asChild>
              <Link href={`/home/${params.account}/projects/${project.id}`}>
                {project.name}
              </Link>
            </CardButton>
          ))}
        </div>
      </PageBody>
    </>
  );
}
```

The page will display a list of projects for the current account, with a link to create a new project. If there are no projects, it will display an empty state with a button to create a new project, which is a link to the new project page.

### Implementing Other Features

Based on the `ProjectsService`, you can implement other features:

1. **Creating a New Project**:
   You could create a form that calls `service.createProject()` when submitted.
2. **Project Details Page**:
   Use `service.getProject()` to fetch and display details of a single project.
3. **Managing Project Members**:
   Use `service.getProjectMembers()` to list members, and `service.addProjectMember()` to add new ones.
4. **Permission Checking**:
   Before performing actions, use `service.hasPermission()` to check if the user is allowed to do so.

## Going beyond Projects

The reason why you're adding Projects is likely because yoyu want to group more entities into a projects, so to better organizate a team's work. You can extend the Projects feature to support other entities, such as Tasks, Documents, and more.

To do so, your project-related entities will need to be related to the Projects entity. This can be done using the `project_id` column in the related tables.

For example, if you want to add Tasks to a Project, you can create a new table `tasks` and add a foreign key to the `project_id` column in the `projects` table.

```sql
-- Table: public.tasks
CREATE TABLE if not exists public.tasks (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  name text NOT NULL,
  description text,
  project_id uuid NOT NULL REFERENCES public.projects(id) ON DELETE CASCADE,
  created_at timestamptz NOT NULL DEFAULT NOW(),
  updated_at timestamptz NOT NULL DEFAULT NOW()
);
```

With this setup, you can now create Tasks that are related to a Project. You can also use the Supabase client to interact with the database, leveraging the SQL functions and RLS policies we set up earlier.

Remember, the database schema and functions we've created provide a solid foundation for these features. As you build out your UI, you'll be calling the methods in `ProjectsService` to interact with the database in a secure, permission-controlled way.

This approach allows you to create a robust, scalable application while keeping your business logic cleanly separated from your UI components.

### RLS Policies

When it comes to RLS, you can use the permissions system we've set up to control access to your database.

This allows you to create fine-grained access control for your users, ensuring that only those who should have access to certain data can do so.

```sql
create policy select_projects_tasks
  on public.tasks
  for select
  to authenticated
  using (
    public.is_project_member(project_id)
  );
```

In short, if a user can view a Project, they can also view the Tasks related to that Project.

You can extend the permissions system further to allow for more granular access control based on the specific requirements of your application, such as `can_edit_task` or `can_delete_task`.

### Best Practices

1. **Error Handling**: The service methods throw errors when something goes wrong. Make sure to catch and handle these errors appropriately in your components.
2. **Type Safety**: The service uses TypeScript, leveraging the `Database` type to ensure type safety when interacting with Supabase.
3. **Separation of Concerns**: By using a service layer, we keep our database logic separate from our UI components, making the code more maintainable and testable.
4. **Server Components**: By using Next.js server components, we can fetch data on the server, reducing the amount of JavaScript sent to the client and improving initial page load times.
5. **Reusability**: The `ProjectsService` can be used across different parts of your application, promoting code reuse.

## Full Schema

Below is the full schema:

```sql
-- enums for project roles: owner, admin, member
create type public.project_role as enum ('owner', 'admin', 'member');

-- enums for project actions
create type public.project_action as enum (
  'view_project',
  'edit_project',
  'delete_project',
  'invite_member',
  'remove_member'
);

/*
# public.projects
# This table stores the projects for the account
*/
create table if not exists public.projects (
  id uuid default gen_random_uuid() primary key,
  name varchar(255) not null,
  description text,
  account_id uuid not null references public.accounts(id) on delete cascade,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

-- revoke access by default to public.projects
revoke all on public.projects from public, service_role;
-- grant access to authenticated users
grant select, insert, update, delete on public.projects to authenticated;

-- indexes on public.projects
create index projects_account_id on public.projects (account_id);

-- RLS policies on public.projects
alter table public.projects enable row level security;

/*
# public.project_members
# This table stores the members of a project
*/
create table if not exists public.project_members (
  project_id uuid not null references public.projects(id) on delete cascade,
  user_id uuid not null references auth.users(id) on delete cascade,
  role public.project_role not null default 'member',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  primary key (project_id, user_id)
);

-- make sure there is only one owner per project
create unique index projects_unique_owner on public.project_members (project_id) where role = 'owner';

-- indexes on public.project_members
create index project_members_project_id on public.project_members (project_id);
create index project_members_user_id on public.project_members (user_id);

-- revoke access by default to public.project_members
revoke all on public.project_members from public, service_role;

-- grant access to authenticated users to public.project_members
grant select, insert, update, delete on public.project_members to authenticated;

-- RLS policies on public.project_members
alter table public.project_members enable row level security;

-- public.is_project_member
-- this function checks if a user is a member of a specific project
create or replace function public.is_project_member(p_project_id uuid)
returns boolean
set search_path = ''
as $$
  select exists (
    select 1 from public.project_members
    where project_id = p_project_id
    and user_id = (select auth.uid())
  );
$$ language sql security definer;

grant execute on function public.is_project_member(uuid) to authenticated;

-- public.is_project_admin
-- this function checks if a user is an admin or owner of a specific project
create or replace function public.is_project_admin(p_project_id uuid)
returns boolean
set search_path = ''
as $$
  select exists (
    select 1 from public.project_members
    where project_id = p_project_id
    and user_id = (select auth.uid())
    and role in ('owner', 'admin')
  );
$$ language sql;

grant execute on function public.is_project_admin to authenticated;

-- public.is_project_owner
-- check if a user is the owner of a project
create or replace function public.is_project_owner(project_id uuid)
returns boolean
set search_path = ''
as $$
  select exists (
    select 1 from public.project_members
    where project_id = $1
    and user_id = (select auth.uid())
    and role = 'owner'
  );
$$ language sql;

grant execute on function public.is_project_owner to authenticated;

-- public.user_has_project_permission
-- check if the current user has the required permissions to perform a specific action on a project
create or replace function public.user_has_project_permission(
  p_user_auth_id uuid,
  p_project_id uuid,
  p_action public.project_action
)
returns boolean
set search_path = ''
as $$
declare
  v_role public.project_role;
begin
  -- first, check if the user is a member of the project
  select role into v_role
  from public.project_members
  where project_id = p_project_id and user_id = p_user_auth_id;

  if v_role is null then
    return false;
  end if;

  -- check permissions based on role and action
  case v_role
    when 'owner' then
      return true;  -- owners can do everything
    when 'admin' then
      return p_action != 'delete_project';  -- admins can do everything except delete the project
    when 'member' then
      return p_action in ('view_project');
    else
      raise exception 'user must be a member of the project to perform this action';
  end case;
end;
$$ language plpgsql;

grant execute on function public.user_has_project_permission to authenticated;

-- public.current_user_has_project_permission
create or replace function public.current_user_has_project_permission(
  p_project_id uuid,
  p_action public.project_action
)
returns boolean
set search_path = ''
as $$
declare
  v_role public.project_role;
begin
  select public.user_has_project_permission((select auth.uid()), p_project_id, p_action);
end;
$$ language plpgsql;

grant execute on function public.current_user_has_project_permission to authenticated;

-- public.current_user_can_manage_project_member
-- Function to check if a user can manage another user in a project
create or replace function public.current_user_can_manage_project_member(
  p_target_member_role public.project_role,
  p_project_id uuid
)
returns boolean
set search_path = ''
as $$
declare
  v_current_user_role public.project_role;
begin
  select role into v_current_user_role
  from public.project_members
  where project_id = p_project_id and user_id = (select auth.uid());

  if v_current_user_role is null or p_target_member_role is null then
    raise exception 'User not found';
  end if;

  -- Check if the manager has a higher role
  return (v_current_user_role = 'owner' and p_target_member_role != 'owner') or
         (v_current_user_role = 'admin' and p_target_member_role = 'member');
end;
$$ language plpgsql;

grant execute on function public.current_user_can_manage_project_member to authenticated;

-- public.update_project_member_role
-- function to update the role of a project member
create or replace function public.update_project_member_role(
  p_user_id uuid,
  p_new_role public.project_role,
  p_project_id uuid
)
returns boolean
set search_path = ''
as $$
declare
  v_current_role public.project_role;
begin
  -- Get the current role of the member
  select role into v_current_role
  from public.project_members
  where project_id = p_project_id and user_id = p_user_id;

  -- Check if the manager can manage this member
  if not public.current_user_can_manage_project_member(v_current_role, p_project_id) then
    raise exception 'Permission denied';
  end if;

  if p_new_role = 'owner' then
    raise exception 'Owner cannot be updated to a different role';
  end if;

  -- Update the member's role
  update public.project_members
  set role = p_new_role
  where project_id = p_project_id and user_id = p_user_id;

  return true;
end;
$$ language plpgsql;

grant execute on function public.update_project_member_role to authenticated;

-- public.can_edit_project
-- check if the user can edit the project
create or replace function public.can_edit_project(p_user_auth_id uuid, p_project_id uuid)
returns boolean
set search_path = ''
as $$
  select public.user_has_project_permission(p_user_auth_id, p_project_id, 'edit_project'::public.project_action);
$$ language sql;

grant execute on function public.can_edit_project to authenticated;

-- public.can_delete_project
-- check if the user can delete the project
create or replace function public.can_delete_project(p_user_auth_id uuid, p_project_id uuid)
returns boolean
set search_path = ''
as $$
  select public.user_has_project_permission(p_user_auth_id, p_project_id, 'delete_project'::public.project_action);
$$ language sql;

grant execute on function public.can_delete_project to authenticated;

-- public.can_invite_project_member
-- check if the user can invite a new member to the project
create or replace function public.can_invite_project_member(p_user_auth_id uuid, p_project_id uuid)
returns boolean
set search_path = ''
as $$
  select public.user_has_project_permission(p_user_auth_id, p_project_id, 'invite_member'::public.project_action);
$$ language sql;

grant execute on function public.can_invite_project_member to authenticated;

/*
RLS POLICIES
*/

-- SELECT(public.projects)
create policy select_projects
  on public.projects
  for select
  to authenticated
  using (
    public.is_project_member(id)
  );

-- INSERT(public.projects)
create policy insert_new_project
  on public.projects
  for insert
  to authenticated
  with check (
    public.has_role_on_account(account_id)
  );

-- DELETE(public.projects)
create policy delete_project
  on public.projects
  for delete
  to authenticated
  using (
    public.can_delete_project((select auth.uid()), id)
  );

-- UPDATE(public.projects)
create policy update_project
  on public.projects
  for update
  to authenticated
  using (
    public.can_edit_project((select auth.uid()), id)
  )
  with check (
    public.can_edit_project((select auth.uid()), id)
  );

-- SELECT(public.project_members)
create policy select_project_members
  on public.project_members
  for select
  to authenticated
  using (
    public.is_project_member(project_id)
  );

-- INSERT(public.project_members)
create policy insert_project_member
  on public.project_members
  for insert
  to authenticated
  with check (
    public.can_invite_project_member(
      (select auth.uid()),
      project_id
    )
  );

-- UPDATE(public.project_members)
create policy update_project_members
  on public.project_members
  for update
  to authenticated
  using (
    public.current_user_can_manage_project_member(
      role,
      project_id
    )
  )
  with check (
    public.current_user_can_manage_project_member(
      role,
      project_id
    )
  );

-- DELETE(public.project_members)
create policy delete_project_members
  on public.project_members
  for delete
  to authenticated
  using (
    public.current_user_can_manage_project_member(
      role,
      project_id
    )
  );

/*
# FUNCTIONS
*/
-- function to add owner of the project creator as the first project member
create or replace function kit.add_project_owner()
returns trigger
as $$
begin
  insert into public.project_members (project_id, user_id, role)
  values (new.id, auth.uid(), 'owner'::public.project_role);

  return new;
end;
$$ language plpgsql security definer;

-- trigger to add owner of the project creator as the first project member
create trigger add_project_owner_on_insert
    after insert on public.projects
    for each row
    execute procedure kit.add_project_owner();

create or replace function public.add_project_member(
  p_project_id uuid,
  p_user_id uuid,
  p_role public.project_role default 'member'
) returns boolean
set search_path = ''
as $$
declare
  v_account_id uuid;
begin
  -- check if the current user has permission to add members
  if not public.is_project_admin(p_project_id) or p_role = 'owner' then
    raise exception 'permission denied';
  end if;

  -- get the account_id for the project
  select account_id into v_account_id
  from public.projects
  where id = p_project_id;

  -- check if the user is a member of the team account
  if not exists (
    select 1 from public.accounts_memberships
    where account_id = v_account_id and user_id = p_user_id
  ) then
    raise exception 'user is not a member of the team account';
  end if;

  -- add the new member (the trigger will enforce the team membership check)
  insert into public.project_members (project_id, user_id, role)
  values (p_project_id, p_user_id, p_role)
  on conflict (project_id, user_id) do update
  set role = excluded.role;

  return true;
end;
$$ language plpgsql;

grant execute on function public.add_project_member to authenticated;

-- TRIGGERS on public.project_members
-- this trigger function ensures that a user being added to a project
-- is already a member of the associated team account
create or replace function kit.check_project_member_in_team()
returns trigger
as $$
declare
  v_account_id uuid;
begin
  select account_id from public.projects
  where id = new.project_id
  into v_account_id;

  if not exists (
    select 1 from public.accounts_memberships
    where account_id = v_account_id and user_id = new.user_id
  ) then
    raise exception 'user must be a member of the team account to be added to the project';
  end if;

  return new;
end;
$$ language plpgsql security definer;

-- we create a trigger that uses the above function
create trigger ensure_project_member_in_team
before insert or update on public.project_members
for each row execute function kit.check_project_member_in_team();
```

## User Interface

Now that we have a data model, let's create the UI for managing projects. We will create a new page at `app/dashboard/[account]/projects/page.tsx` that will display a list of projects for the current account.

### Adding the Page to the Navigation

Let's update the navigation menu to add a new link to the projects page. We will create a new file at `apps/web/config/team-account-navigation.config.tsx` with the following content:

 ```tsx {% title="apps/web/config/team-account-navigation.config.tsx" %} {7-11}
{
    label: 'common:routes.dashboard',
    path: pathsConfig.app.accountHome.replace('[account]', account),
    Icon: <LayoutDashboard className={iconClasses} />,
    end: true,
},
{
    label: 'common:routes.projects',
    path: `/home/${account}/projects`,
    Icon: <FolderKanban className={iconClasses} />,
},
```

NB: you need to the component `FolderKanban` from `lucide-react` to use the icon.

### Creating the Projects List Page

We can now create the page at `app/dashboard/[account]/projects/page.tsx` with the following content:

 ```tsx {% title="app/dashboard/[account]/projects/page.tsx" %}
import { use } from 'react';

import Link from 'next/link';

import { getSupabaseServerComponentClient } from '@kit/supabase/server-component-client';
import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';
import { Button } from '@kit/ui/button';
import {
  CardButton,
  CardButtonHeader,
  CardButtonTitle,
} from '@kit/ui/card-button';
import {
  EmptyState,
  EmptyStateButton,
  EmptyStateHeading,
  EmptyStateText,
} from '@kit/ui/empty-state';
import { If } from '@kit/ui/if';
import { PageBody, PageHeader } from '@kit/ui/page';

import { CreateProjectDialog } from '~/home/[account]/projects/_components/create-project-dialog';
import { createProjectsService } from '~/lib/server/projects/projects.service';

interface ProjectsPageProps {
  params: {
    account: string;
  };
}

export default function ProjectsPage({ params }: ProjectsPageProps) {
  const client = getSupabaseServerComponentClient();
  const service = createProjectsService(client);

  const projects = use(service.getProjects(params.account));

  return (
    <>
      <PageHeader title="Projects" description={<AppBreadcrumbs />}>
        <Link href={`/home/${params.account}/projects/new`}>
          <CreateProjectDialog>
            <Button>New Project</Button>
          </CreateProjectDialog>
        </Link>
      </PageHeader>

      <PageBody>
        <If condition={projects.length === 0}>
          <EmptyState>
            <EmptyStateHeading>No projects found</EmptyStateHeading>

            <EmptyStateText>
              You still have not created any projects. Create your first project
              now!
            </EmptyStateText>

            <CreateProjectDialog>
              <EmptyStateButton>Create Project</EmptyStateButton>
            </CreateProjectDialog>
          </EmptyState>
        </If>

        <div className={'grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4'}>
          {projects.map((project) => (
            <CardButton key={project.id} asChild>
              <Link href={`/home/${params.account}/projects/${project.id}`}>
                <CardButtonHeader>
                  <CardButtonTitle>{project.name}</CardButtonTitle>
                </CardButtonHeader>
              </Link>
            </CardButton>
          ))}
        </div>
      </PageBody>
    </>
  );
}
```

This page is pretty straightforward, it just displays a list of projects for the account. We can now build the `ProjectsDataTable` component that will display the list of projects.

When we have a list of projects, we display them in the following way:

{% img src="/assets/images/docs/projects-list.webp" width="2858" height="1920" /%}

Instead, we can also display a message when there are no projects:

{% img src="/assets/images/docs/projects-empty-state.webp" width="2858" height="1920" /%}

### Creating the Dialog to Create a Project

We can now create the dialog to create a project. We will create a new file at `apps/web/home/[account]/projects/_components/create-project-dialog.tsx` with the following content:

 ```tsx {% title="apps/web/home/[account]/projects/_components/create-project-dialog.tsx" %}
'use client';

import { useState, useTransition } from 'react';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';

import { useTeamAccountWorkspace } from '@kit/team-accounts/hooks/use-team-account-workspace';
import { Button } from '@kit/ui/button';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@kit/ui/dialog';
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
} from '@kit/ui/form';
import { Input } from '@kit/ui/input';

import { CreateProjectSchema } from '../_lib/schema/create-project-schema';
import { createProjectAction } from '../_lib/server/server-actions';

export function CreateProjectDialog(props: React.PropsWithChildren) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <Dialog open={isOpen} onOpenChange={setIsOpen}>
      <DialogTrigger asChild>{props.children}</DialogTrigger>

      <DialogContent>
        <DialogHeader>
          <DialogTitle>Create Project</DialogTitle>

          <DialogDescription>
            Create a new project for your team.
          </DialogDescription>
        </DialogHeader>

        <CreateProjectDialogForm
          onCancel={() => setIsOpen(false)}
          onCreateProject={() => setIsOpen(false)}
        />
      </DialogContent>
    </Dialog>
  );
}

function CreateProjectDialogForm(props: {
  onCreateProject?: (name: string) => unknown;
  onCancel?: () => unknown;
}) {
  const {
    account: { id: accountId },
  } = useTeamAccountWorkspace();

  const form = useForm({
    resolver: zodResolver(CreateProjectSchema),
    defaultValues: {
      name: '',
      accountId,
    },
  });

  const [pending, startTransition] = useTransition();

  return (
    <Form {...form}>
      <form
        className={'flex flex-col space-y-4'}
        onSubmit={form.handleSubmit((data) => {
          startTransition(async () => {
            await createProjectAction(data);
          });
        })}
      >
        <FormField
          name={'name'}
          render={({ field }) => (
            <FormItem>
              <FormLabel>Project Name</FormLabel>

              <FormControl>
                <Input
                  data-test={'project-name-input'}
                  required
                  min={3}
                  max={50}
                  type={'text'}
                  placeholder={''}
                  {...field}
                />
              </FormControl>

              <FormDescription>Enter a name for your project (Ex. Accounting)</FormDescription>
            </FormItem>
          )}
        />

        <div className={'flex justify-end space-x-2'}>
          <Button variant={'outline'} type={'button'} onClick={props.onCancel}>
            Cancel
          </Button>

          <Button disabled={pending}>Create Project</Button>
        </div>
      </form>
    </Form>
  );
}
```

We can now create the server action to create a project. We will create a new file at `apps/web/home/[account]/projects/_lib/server/server-actions.ts` with the following content:

 ```tsx {% title="apps/web/home/[account]/projects/_lib/server/server-actions.ts" %}
'use server';

import { revalidatePath } from 'next/cache';

import { enhanceAction } from '@kit/next/actions';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

import { CreateProjectSchema } from '../schema/create-project-schema';

export const createProjectAction = enhanceAction(
  async (data) => {
    const client = getSupabaseServerClient();
    const logger = await getLogger();

    logger.info(
      {
        accountId: data.accountId,
        name: data.name,
      },
      'Creating project...',
    );

    const response = await client.from('projects').insert({
      account_id: data.accountId,
      name: data.name,
    });

    if (response.error) {
      logger.error(
        {
          accountId: data.accountId,
          name: data.name,
          error: response.error,
        },
        'Failed to create project',
      );

      throw response.error;
    }

    logger.info(
      {
        accountId: data.accountId,
        name: data.name,
      },
      'Project created',
    );

    revalidatePath('/home/[account]/projects', 'layout');
  },
  {
    schema: CreateProjectSchema,
    auth: true,
  },
);
```

When the user submits the form, we call the `createProjectAction` server action. This action will insert a new project into the database. To refresh the data, we use the `revalidatePath` function to revalidate the `/home/[account]/projects` path.

### Project Detail Page

Now, we want to create a detail page for each project.

In this case, we will use an outer layout to wrap the project detail page, so that we can display an inner navigation menu that will reuse the same layout.


We will create a new file at `apps/web/home/[account]/projects/[id]/layout.tsx` with the following content:

```tsx
import { use } from 'react';

import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';
import {
  BorderedNavigationMenu,
  BorderedNavigationMenuItem,
} from '@kit/ui/bordered-navigation-menu';
import { PageBody, PageHeader } from '@kit/ui/page';

import { getProject } from './_lib/server/get-project';

interface ProjectDetailLayoutProps {
  params: {
    id: string;
    account: string;
  };
}

export default function ProjectDetailLayout({
  params,
  children,
}: React.PropsWithChildren<ProjectDetailLayoutProps>) {
  const project = use(getProject(params.id));

  return (
    <>
      <PageHeader
        title={project.name}
        description={<AppBreadcrumbs values={{ [project.id]: project.name }} />}
      />

      <PageBody>
        <div className={'border-b px-4 pb-2.5'}>
          <BorderedNavigationMenu>
            <BorderedNavigationMenuItem
              path={`/home/${params.account}/projects/${project.id}`}
              label={'Documents'}
            />

            <BorderedNavigationMenuItem
              path={`/home/${params.account}/projects/${project.id}/members`}
              label={'Members'}
            />
          </BorderedNavigationMenu>
        </div>

        {children}
      </PageBody>
    </>
  );
}
```

#### Navigation Menu

Just as an example, we have added a navigation menu with two links: Documents and Members. You can customize this menu to fit your needs.

#### Using the "getProject" function

As you may have noticed, we have a `getProject` function that will fetch the project from the database.

We use this to make use of caching and reuse this function across all the pages in the Project layout, so we don't re-fetch the data from the database when avigating between pages.

 ```tsx {% title="apps/web/app/home/[account]/projects/[id]/_lib/server/get-project.ts" %}
import { cache } from 'react';

import { getSupabaseServerClient } from '@kit/supabase/server-client';

import { createProjectsService } from '~/lib/server/projects/projects.service';

export const getProject = cache(projectLoader);

async function projectLoader(id: string) {
  const client = getSupabaseServerClient();
  const service = createProjectsService(client);

  return service.getProject(id);
}
```

### Project Detail Page

Now, we want to create a detail page for each project.

We will create a new file at `apps/web/app/home/[account]/projects/[id]/page.tsx` with the following content:

 ```tsx {% title="apps/web/app/home/[account]/projects/[id]/page.tsx" %}
import { use } from 'react';

import { getSupabaseServerComponentClient } from '@kit/supabase/server-component-client';
import { AppBreadcrumbs } from '@kit/ui/app-breadcrumbs';
import { PageBody, PageHeader } from '@kit/ui/page';

import { createProjectsService } from '~/lib/server/projects/projects.service';

interface ProjectDetailPageProps {
  params: {
    id: string;
    account: string;
  };
}

export default function ProjectDetailPage({ params }: ProjectDetailPageProps) {
  const client = getSupabaseServerComponentClient();
  const service = createProjectsService(client);

  const project = use(service.getProject(params.id));

  return (
    <></>
  );
}
```

Generally speaking, it is up to you to decide how you want to display the project data. This is the inner part of the layout, so we can use the same layout for the detail page.

## Conclusion

In this tutorial, we've covered how to create a simple (yet powerful) data model for managing Projects in your Makerkit application. We've also explored how to implement this data model using Supabase and Next.js, and how to use the Supabase client to interact with the database.

By following these steps, you can create a robust, scalable application that leverages the power of Supabase and Next.js to manage your team's projects in a secure, permission-controlled way.


