-----------------
FILE PATH: kits\next-supabase-turbo\recipes\stripe-billing-checkout-addons.mdoc

---
status: "published"
title: 'Checkout Addons with Stripe Billing'
label: 'Checkout Addons with Stripe Billing'
order: 3
description: 'Learn how to create a subscription with addons using Stripe Billing.'
---

Stripe allows us to add multiple line items to a single subscription. This is useful when you want to offer additional features or services to your customers.

This feature is not supported by default in Makerkit. However, in this guide, I will show you how to create a subscription with addons using Stripe Billing, and how to customize Makerkit to support this feature.

Let's get started!

## 1. Personal Account Checkout Form

File: `apps/web/app/home/(user)/billing/_components/personal-account-checkout-form.tsx`

Update your `PersonalAccountCheckoutForm` component to pass addon data to the checkout session creation process:

 ```typescript {% title="home/(user)/billing/_components/personal-account-checkout-form.tsx" %}
<CheckoutForm
  // ...existing props
  onSubmit={({ planId, productId, addons }) => {
    startTransition(async () => {
      try {
        const { checkoutToken } = await createPersonalAccountCheckoutSession({
          planId,
          productId,
          addons, // Add this line
        });
        setCheckoutToken(checkoutToken);
      } catch {
        setError(true);
      }
    });
  }}
/>
```

This change allows the checkout form to handle addon selections and pass them to the checkout session creation process.

## 2. Personal Account Checkout Schema

Let's add addon support to the personal account checkout schema. The `addons` is an array of objects, each containing a `productId` and `planId`. By default, the `addons` array is empty.

Update your `PersonalAccountCheckoutSchema`:

 ```typescript {% title="home/(user)/billing/_lib/schema/personal-account-checkout.schema.ts" %}
export const PersonalAccountCheckoutSchema = z.object({
  planId: z.string().min(1),
  productId: z.string().min(1),
  addons: z
    .array(
      z.object({
        productId: z.string().min(1),
        planId: z.string().min(1),
      }),
    )
    .default([]),
});
```

This schema update ensures that the addon data is properly validated before being processed.

## 3. User Billing Service

Update your `createCheckoutSession` method. This method is responsible for creating a checkout session with the billing gateway. We need to pass the addon data to the billing gateway:

 ```typescript {% title="home/(user)/billing/_lib/server/user-billing.service.ts" %}
async createCheckoutSession({
  planId,
  productId,
  addons,
}: z.infer<typeof PersonalAccountCheckoutSchema>) {
  // ...existing code

  const checkoutToken = await this.billingGateway.createCheckoutSession({
    // ...existing props
    addons,
  });

  // ...rest of the method
}
```

This change ensures that the addon information is passed to the billing gateway when creating a checkout session.

## 4. Team Account Checkout Form

File: `apps/web/app/home/[account]/billing/_components/team-account-checkout-form.tsx`

Make similar changes to the `TeamAccountCheckoutForm` as we did for the personal account form.

## 5. Team Billing Schema

File: `apps/web/app/home/[account]/billing/_lib/schema/team-billing.schema.ts`

Update your `TeamCheckoutSchema` similar to the personal account schema.

## 6. Team Billing Service

File: `apps/web/app/home/[account]/billing/_lib/server/team-billing.service.ts`

Update the `createCheckoutSession` method similar to the user billing service.

## 7. Billing Configuration

We can now add addons to our billing configuration. Update your billing configuration file to include addons:

 ```typescript {% title="apps/web/config/billing.sample.config.ts" %}
plans: [
  {
    // ...existing plan config
    addons: [
      {
        id: 'price_1J4J9zL2c7J1J4J9zL2c7J1',
        name: 'Extra Feature',
        cost: 9.99,
        type: 'flat' as const,
      },
    ],
  },
],
```

NB: the `ID` of the addon should match the `planId` in your Stripe account.

## 8. Localization

Add a new translation key for translating the term "Add-ons" in your billing locale file:

 ```json {% title="apps/web/public/locales/en/billing.json" %}
{
  // ...existing translations
  "addons": "Add-ons"
}
```

## 9. Billing Schema

File: `packages/billing/core/src/create-billing-schema.ts`

The billing schema has been updated to include addons. You don't need to change this file, but be aware that the schema now supports addons.

## 10. Create Billing Checkout Schema

File: `packages/billing/core/src/schema/create-billing-checkout.schema.ts`

The checkout schema now includes addons. Again, you don't need to change this file, but your checkout process will now support addons.

## 11. Plan Picker Component

File: `packages/billing/gateway/src/components/plan-picker.tsx`

This component has been significantly updated to handle addons. It now displays addons as checkboxes and manages their state.

Here's the updated Plan Picker component:

 ```tsx {% title="packages/billing/gateway/src/components/plan-picker.tsx" %}
'use client';

import { useMemo } from 'react';

import { zodResolver } from '@hookform/resolvers/zod';
import { ArrowRight, CheckCircle } from 'lucide-react';
import { useForm } from 'react-hook-form';
import { useTranslation } from 'react-i18next';
import { z } from 'zod';

import {
  BillingConfig,
  type LineItemSchema,
  getPlanIntervals,
  getPrimaryLineItem,
  getProductPlanPair,
} from '@kit/billing';
import { formatCurrency } from '@kit/shared/utils';
import { Badge } from '@kit/ui/badge';
import { Button } from '@kit/ui/button';
import { Checkbox } from '@kit/ui/checkbox';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@kit/ui/form';
import { If } from '@kit/ui/if';
import { Label } from '@kit/ui/label';
import {
  RadioGroup,
  RadioGroupItem,
  RadioGroupItemLabel,
} from '@kit/ui/radio-group';
import { Separator } from '@kit/ui/separator';
import { Trans } from '@kit/ui/trans';
import { cn } from '@kit/ui/utils';

import { LineItemDetails } from './line-item-details';

const AddonSchema = z.object({
  name: z.string(),
  id: z.string(),
  productId: z.string(),
  planId: z.string(),
  cost: z.number(),
});

type OnSubmitData = {
  planId: string;
  productId: string;
  addons: z.infer<typeof AddonSchema>[];
};

export function PlanPicker(
  props: React.PropsWithChildren<{
    config: BillingConfig;
    onSubmit: (data: OnSubmitData) => void;
    canStartTrial?: boolean;
    pending?: boolean;
  }>,
) {
  const { t } = useTranslation(`billing`);

  const intervals = useMemo(
    () => getPlanIntervals(props.config),
    [props.config],
  ) as string[];

  const form = useForm({
    reValidateMode: 'onChange',
    mode: 'onChange',
    resolver: zodResolver(
      z
        .object({
          planId: z.string(),
          productId: z.string(),
          interval: z.string().optional(),
          addons: z.array(AddonSchema).optional(),
        })
        .refine(
          (data) => {
            try {
              const { product, plan } = getProductPlanPair(
                props.config,
                data.planId,
              );

              return product && plan;
            } catch {
              return false;
            }
          },
          { message: t('noPlanChosen'), path: ['planId'] },
        ),
    ),
    defaultValues: {
      interval: intervals[0],
      planId: '',
      productId: '',
      addons: [] as z.infer<typeof AddonSchema>[],
    },
  });

  const { interval: selectedInterval } = form.watch();
  const planId = form.getValues('planId');

  const { plan: selectedPlan, product: selectedProduct } = useMemo(() => {
    try {
      return getProductPlanPair(props.config, planId);
    } catch {
      return {
        plan: null,
        product: null,
      };
    }
  }, [props.config, planId]);

  const addons = form.watch('addons');

  const onAddonAdded = (data: z.infer<typeof AddonSchema>) => {
    form.setValue('addons', [...addons, data], { shouldValidate: true });
  };

  const onAddonRemoved = (id: string) => {
    form.setValue(
      'addons',
      addons.filter((item) => item.id !== id),
      { shouldValidate: true },
    );
  };

  // display the period picker if the selected plan is recurring or if no plan is selected
  const isRecurringPlan =
    selectedPlan?.paymentType === 'recurring' || !selectedPlan;

  const locale = useTranslation().i18n.language;

  return (
    <Form {...form}>
      <div
        className={
          'flex flex-col space-y-4 lg:flex-row lg:space-x-4 lg:space-y-0'
        }
      >
        <form
          className={'flex w-full max-w-xl flex-col space-y-6'}
          onSubmit={form.handleSubmit(props.onSubmit)}
        >
          <If condition={intervals.length}>
            <div
              className={cn('transition-all', {
                ['pointer-events-none opacity-50']: !isRecurringPlan,
              })}
            >
              <FormField
                name={'interval'}
                render={({ field }) => {
                  return (
                    <FormItem className={'rounded-md border p-4'}>
                      <FormLabel htmlFor={'plan-picker-id'}>
                        <Trans i18nKey={'common:billingInterval.label'} />
                      </FormLabel>

                      <FormControl id={'plan-picker-id'}>
                        <RadioGroup name={field.name} value={field.value}>
                          <div className={'flex space-x-2.5'}>
                            {intervals.map((interval) => {
                              const selected = field.value === interval;

                              return (
                                <label
                                  htmlFor={interval}
                                  key={interval}
                                  className={cn(
                                    'flex items-center space-x-2 rounded-md border border-transparent px-4 py-2 transition-colors',
                                    {
                                      ['border-primary']: selected,
                                      ['hover:border-primary']: !selected,
                                    },
                                  )}
                                >
                                  <RadioGroupItem
                                    id={interval}
                                    value={interval}
                                    onClick={() => {
                                      form.setValue('interval', interval, {
                                        shouldValidate: true,
                                      });

                                      form.setValue('addons', [], {
                                        shouldValidate: true,
                                      });

                                      if (selectedProduct) {
                                        const plan = selectedProduct.plans.find(
                                          (item) => item.interval === interval,
                                        );

                                        form.setValue(
                                          'planId',
                                          plan?.id ?? '',
                                          {
                                            shouldValidate: true,
                                            shouldDirty: true,
                                            shouldTouch: true,
                                          },
                                        );
                                      }
                                    }}
                                  />

                                  <span
                                    className={cn('text-sm', {
                                      ['cursor-pointer']: !selected,
                                    })}
                                  >
                                    <Trans
                                      i18nKey={`billing:billingInterval.${interval}`}
                                    />
                                  </span>
                                </label>
                              );
                            })}
                          </div>
                        </RadioGroup>
                      </FormControl>

                      <FormMessage />
                    </FormItem>
                  );
                }}
              />
            </div>
          </If>

          <FormField
            name={'planId'}
            render={({ field }) => (
              <FormItem>
                <FormLabel>
                  <Trans i18nKey={'common:planPickerLabel'} />
                </FormLabel>

                <FormControl>
                  <RadioGroup value={field.value} name={field.name}>
                    {props.config.products.map((product) => {
                      const plan = product.plans.find((item) => {
                        if (item.paymentType === 'one-time') {
                          return true;
                        }

                        return item.interval === selectedInterval;
                      });

                      if (!plan || plan.custom) {
                        return null;
                      }

                      const planId = plan.id;
                      const selected = field.value === planId;

                      const primaryLineItem = getPrimaryLineItem(
                        props.config,
                        planId,
                      );

                      if (!primaryLineItem) {
                        throw new Error(`Base line item was not found`);
                      }

                      return (
                        <RadioGroupItemLabel
                          selected={selected}
                          key={primaryLineItem.id}
                        >
                          <RadioGroupItem
                            data-test-plan={plan.id}
                            key={plan.id + selected}
                            id={plan.id}
                            value={plan.id}
                            onClick={() => {
                              if (selected) {
                                return;
                              }

                              form.setValue('planId', planId, {
                                shouldValidate: true,
                              });

                              form.setValue('productId', product.id, {
                                shouldValidate: true,
                              });

                              form.setValue('addons', [], {
                                shouldValidate: true,
                              });
                            }}
                          />

                          <div
                            className={
                              'flex w-full flex-col content-center space-y-2 lg:flex-row lg:items-center lg:justify-between lg:space-y-0'
                            }
                          >
                            <Label
                              htmlFor={plan.id}
                              className={
                                'flex flex-col justify-center space-y-2'
                              }
                            >
                              <div className={'flex items-center space-x-2.5'}>
                                <span className="font-semibold">
                                  <Trans
                                    i18nKey={`billing:plans.${product.id}.name`}
                                    defaults={product.name}
                                  />
                                </span>

                                <If
                                  condition={
                                    plan.trialDays && props.canStartTrial
                                  }
                                >
                                  <div>
                                    <Badge
                                      className={'px-1 py-0.5 text-xs'}
                                      variant={'success'}
                                    >
                                      <Trans
                                        i18nKey={`billing:trialPeriod`}
                                        values={{
                                          period: plan.trialDays,
                                        }}
                                      />
                                    </Badge>
                                  </div>
                                </If>
                              </div>

                              <span className={'text-muted-foreground'}>
                                <Trans
                                  i18nKey={`billing:plans.${product.id}.description`}
                                  defaults={product.description}
                                />
                              </span>
                            </Label>

                            <div
                              className={
                                'flex flex-col space-y-2 lg:flex-row lg:items-center lg:space-x-4 lg:space-y-0 lg:text-right'
                              }
                            >
                              <div>
                                <Price key={plan.id}>
                                  <span>
                                    {formatCurrency({
                                      currencyCode:
                                        product.currency.toLowerCase(),
                                      value: primaryLineItem.cost,
                                      locale,
                                    })}
                                  </span>
                                </Price>

                                <div>
                                  <span className={'text-muted-foreground'}>
                                    <If
                                      condition={
                                        plan.paymentType === 'recurring'
                                      }
                                      fallback={
                                        <Trans i18nKey={`billing:lifetime`} />
                                      }
                                    >
                                      <Trans
                                        i18nKey={`billing:perPeriod`}
                                        values={{
                                          period: selectedInterval,
                                        }}
                                      />
                                    </If>
                                  </span>
                                </div>
                              </div>
                            </div>
                          </div>
                        </RadioGroupItemLabel>
                      );
                    })}
                  </RadioGroup>
                </FormControl>

                <FormMessage />
              </FormItem>
            )}
          />

          <If condition={selectedPlan?.addons}>
            <div className={'flex flex-col space-y-2.5'}>
              <span className={'text-sm font-medium'}>Addons</span>

              <div className={'flex flex-col space-y-2'}>
                {selectedPlan?.addons?.map((addon) => {
                  return (
                    <div
                      className={'flex items-center space-x-2 text-sm'}
                      key={addon.id}
                    >
                      <Checkbox
                        value={addon.id}
                        onCheckedChange={() => {
                          if (addons.some((item) => item.id === addon.id)) {
                            onAddonRemoved(addon.id);
                          } else {
                            onAddonAdded({
                              productId: selectedProduct.id,
                              planId: selectedPlan.id,
                              id: addon.id,
                              name: addon.name,
                              cost: addon.cost,
                            });
                          }
                        }}
                      />

                      <span>{addon.name}</span>
                    </div>
                  );
                })}
              </div>
            </div>
          </If>

          <div>
            <Button
              data-test="checkout-submit-button"
              disabled={props.pending ?? !form.formState.isValid}
            >
              {props.pending ? (
                t('redirectingToPayment')
              ) : (
                <>
                  <If
                    condition={selectedPlan?.trialDays && props.canStartTrial}
                    fallback={t(`proceedToPayment`)}
                  >
                    <span>{t(`startTrial`)}</span>
                  </If>

                  <ArrowRight className={'ml-2 h-4 w-4'} />
                </>
              )}
            </Button>
          </div>
        </form>

        {selectedPlan && selectedInterval && selectedProduct ? (
          <PlanDetails
            selectedInterval={selectedInterval}
            selectedPlan={selectedPlan}
            selectedProduct={selectedProduct}
            addons={addons}
          />
        ) : null}
      </div>
    </Form>
  );
}

function PlanDetails({
  selectedProduct,
  selectedInterval,
  selectedPlan,
  addons = [],
}: {
  selectedProduct: {
    id: string;
    name: string;
    description: string;
    currency: string;
    features: string[];
  };

  selectedInterval: string;

  selectedPlan: {
    lineItems: z.infer<typeof LineItemSchema>[];
    paymentType: string;
  };

  addons: z.infer<typeof AddonSchema>[];
}) {
  const isRecurring = selectedPlan.paymentType === 'recurring';
  const { i18n } = useTranslation(`billing`);

  // trick to force animation on re-render
  const key = Math.random();

  return (
    <div
      key={key}
      className={
        'fade-in animate-in zoom-in-95 flex w-full flex-col space-y-4 py-2 lg:px-8'
      }
    >
      <div className={'flex flex-col space-y-0.5'}>
        <span className={'text-sm font-medium'}>
          <b>
            <Trans
              i18nKey={`billing:plans.${selectedProduct.id}.name`}
              defaults={selectedProduct.name}
            />
          </b>{' '}
          <If condition={isRecurring}>
            / <Trans i18nKey={`billing:billingInterval.${selectedInterval}`} />
          </If>
        </span>

        <p>
          <span className={'text-muted-foreground text-sm'}>
            <Trans
              i18nKey={`billing:plans.${selectedProduct.id}.description`}
              defaults={selectedProduct.description}
            />
          </span>
        </p>
      </div>

      <If condition={selectedPlan.lineItems.length > 0}>
        <Separator />

        <div className={'flex flex-col space-y-2'}>
          <span className={'text-sm font-semibold'}>
            <Trans i18nKey={'billing:detailsLabel'} />
          </span>

          <LineItemDetails
            lineItems={selectedPlan.lineItems ?? []}
            selectedInterval={isRecurring ? selectedInterval : undefined}
            currency={selectedProduct.currency}
          />
        </div>
      </If>

      <Separator />

      <div className={'flex flex-col space-y-2'}>
        <span className={'text-sm font-semibold'}>
          <Trans i18nKey={'billing:featuresLabel'} />
        </span>

        {selectedProduct.features.map((item) => {
          return (
            <div key={item} className={'flex items-center space-x-1 text-sm'}>
              <CheckCircle className={'h-4 text-green-500'} />

              <span className={'text-secondary-foreground'}>
                <Trans i18nKey={item} defaults={item} />
              </span>
            </div>
          );
        })}
      </div>

      <If condition={addons.length > 0}>
        <div className={'flex flex-col space-y-2'}>
          <span className={'text-sm font-semibold'}>
            <Trans i18nKey={'billing:addons'} />
          </span>

          {addons.map((addon) => {
            return (
              <div
                key={addon.id}
                className={'flex items-center space-x-1 text-sm'}
              >
                <CheckCircle className={'h-4 text-green-500'} />

                <span className={'text-secondary-foreground'}>
                  <Trans i18nKey={addon.name} defaults={addon.name} />
                </span>

                <span>-</span>

                <span className={'text-xs font-semibold'}>
                  {formatCurrency({
                    currencyCode: selectedProduct.currency.toLowerCase(),
                    value: addon.cost,
                    locale: i18n.language,
                  })}
                </span>
              </div>
            );
          })}
        </div>
      </If>
    </div>
  );
}

function Price(props: React.PropsWithChildren) {
  return (
    <span
      className={
        'animate-in slide-in-from-left-4 fade-in text-xl font-semibold tracking-tight duration-500'
      }
    >
      {props.children}
    </span>
  );
}
```


## 12. Stripe Checkout Creation

File: `packages/billing/stripe/src/services/create-stripe-checkout.ts`

The Stripe checkout creation process now includes addons:

```typescript
if (params.addons.length > 0) {
  lineItems.push(
    ...params.addons.map((addon) => ({
      price: addon.planId,
      quantity: 1,
    })),
  );
}
```

This change ensures that selected addons are included in the Stripe checkout session.

## Conclusion

These changes introduce a flexible addon system to Makerkit. By implementing these updates, you'll be able to offer additional features or services alongside your main subscription plans.

Remember, while adding addons to the checkout process is now straightforward, managing them post-purchase (like allowing users to add or remove addons from an active subscription) will require additional custom development. Consider your specific use case and user needs when implementing this feature.


-----------------
FILE PATH: kits\next-supabase-turbo\recipes\subscription-entitlements.mdoc

---
status: "published"
title: 'Managing entitlements based on subscriptions in your Next.js Supabase app'
label: 'Subscription Entitlements'
order: 3
description: 'Learn how to effectively manage access and entitlements based on subscriptions in your Next.js Supabase app.'
---

As your SaaS grows, the complexity of managing user entitlements increases. In this guide, we’ll build a flexible, performant, and secure entitlements system using Makerkit, Supabase, and PostgreSQL. 

This solution leverages the power of PostgreSQL functions and Supabase RPCs, enforcing business logic at the database level while integrating seamlessly with your Next.js app.

NB: this is a generic solution, but you can use it as a starting point to build your own custom entitlements system. In fact - I recommend you to do so, as your needs will evolve and this solution might not cover all your requirements.

### Why a Custom Entitlements System?

Makerkit is built to be flexible and extensible. Instead of offering a one-size-fits-all entitlements system, Makerkit provides a foundation you can customize. 

This article will walk you through the complete process:
- **Flexibility & Extensibility:** Easily handle different entitlement types (flat or usage quotas).
- **Performance:** Offload entitlement checks to the database.
- **Consistency & Security:** Ensure rules are enforced both in your app code and via Row Level Security (RLS).

## Step 1: Define Your Database Schema

We start by creating two tables: one to declare the entitlements for each plan variant and another to track feature usage per account. Both tables are set up with strict security rules using RLS policies.

### Creating the `plan_entitlements` Table

This table stores entitlement definitions. Each row defines which features are enabled for a specific plan variant. Note the use of a unique constraint to avoid duplicate entries and strict permission controls to ensure data security.

```sql {% title="apps/web/supabase/migrations/20250205034829_subscription-entitlements.sql" %}
-- Table to store plan entitlements with strict security and RLS policies
CREATE TABLE public.plan_entitlements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  variant_id VARCHAR(255) NOT NULL,
  feature VARCHAR(255) NOT NULL,
  entitlement JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE (variant_id, feature)
);

-- Revoke all and then selectively grant minimal permissions
REVOKE ALL ON public.plan_entitlements FROM public;
ALTER TABLE public.plan_entitlements ENABLE ROW LEVEL SECURITY;
GRANT SELECT ON public.plan_entitlements TO authenticated;

-- Policy ensuring authenticated users can select from the table
CREATE POLICY select_plan_entitlements
    ON public.plan_entitlements
    FOR SELECT
    TO authenticated
    USING (true);
```

### Creating the `feature_usage` Table

This table tracks the usage of features for each account. We use JSONB to support flexible usage metrics and add an index for efficient lookups.

```sql {% title="apps/web/supabase/migrations/20250205034829_subscription-entitlements.sql" %}
-- Table to track feature usage per account with RLS enforcement
CREATE TABLE public.feature_usage (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id UUID NOT NULL REFERENCES public.accounts(id) ON DELETE CASCADE,
  feature VARCHAR(255) NOT NULL,
  usage JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE (account_id, feature)
);

REVOKE ALL ON public.feature_usage FROM public;
GRANT SELECT ON public.feature_usage TO authenticated;
ALTER TABLE public.feature_usage ENABLE ROW LEVEL SECURITY;

-- Policy providing access to accounts based on ownership or team roles
CREATE POLICY select_feature_usage
    ON public.feature_usage
    FOR SELECT
    TO authenticated
    USING (
      public.has_role_on_account(account_id) OR (SELECT auth.uid()) = account_id
    );

-- Add an index to optimize lookup by account and feature
CREATE INDEX idx_feature_usage_account_id ON public.feature_usage(account_id, feature);
```

### Automatically Creating a Usage Row for New Accounts

Use a trigger to ensure that as soon as an account is created, a corresponding row in `feature_usage` is created:

```sql {% title="apps/web/supabase/migrations/20250205034829_subscription-entitlements.sql" %}
-- Function to auto-create a feature_usage row upon account creation
CREATE OR REPLACE FUNCTION public.create_feature_usage_row()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.feature_usage (account_id, feature)
  VALUES (NEW.id, '');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Create the trigger to execute the above function after account creation
CREATE TRIGGER create_feature_usage_row
AFTER INSERT ON public.accounts
FOR EACH ROW
EXECUTE FUNCTION public.create_feature_usage_row();
```
---

## Step 2: Develop PostgreSQL Functions for Entitlements

Encapsulate the core entitlement logic inside PostgreSQL functions. Each function is designed to be secure (using `SECURITY INVOKER` or `SECURITY DEFINER` where needed) and atomic.

### Check if an Account Can Use a Feature

This function determines if an account meets the criteria to use a given feature based on its subscription and plan entitlements.

```sql
-- Function to check if the account is eligible to use a feature
CREATE OR REPLACE FUNCTION public.can_use_feature(
  p_account_id UUID, 
  p_feature VARCHAR
)
RETURNS BOOLEAN 
SECURITY INVOKER 
AS $$
DECLARE
  eligible_variant VARCHAR(255);
BEGIN
  SELECT si.variant_id 
    INTO eligible_variant
  FROM public.subscription_items si
  JOIN public.subscriptions s ON s.id = si.subscription_id
  WHERE s.account_id = p_account_id
    AND si.type = 'flat'
    AND EXISTS (
      SELECT 1 
      FROM public.plan_entitlements ent
      WHERE ent.feature = p_feature
        AND ent.variant_id = si.variant_id
    )
  LIMIT 1;

  RETURN eligible_variant IS NOT NULL;
END;
$$ LANGUAGE plpgsql;
```

### Retrieve Entitlement Details

The following function returns the details of an entitlement along with any usage data for the account.

```sql {% title="apps/web/supabase/migrations/20250205034829_subscription-entitlements.sql" %}
-- Function to get entitlement details for a feature
CREATE OR REPLACE FUNCTION public.get_entitlement(
  p_account_id UUID, 
  p_feature VARCHAR
)
RETURNS TABLE(variant_id VARCHAR(255), entitlement JSONB) 
SECURITY INVOKER 
AS $$
BEGIN
  RETURN QUERY
  SELECT si.variant_id,
         ent.entitlement AS entitlement
  FROM public.subscriptions s
  JOIN public.subscription_items si ON s.id = si.subscription_id
  JOIN public.plan_entitlements ent ON ent.variant_id = si.variant_id
  WHERE s.account_id = p_account_id
    AND ent.feature = p_feature;
END;
$$ LANGUAGE plpgsql;
```

### Update Feature Usage

These functions update the `feature_usage` table. The first function handles merging JSON usage data, and the second one atomically updates quota usage using an UPSERT pattern.

```sql
-- Function to update feature usage with custom JSON merge
CREATE OR REPLACE FUNCTION public.update_feature_usage(
  p_account_id UUID, 
  p_feature VARCHAR, 
  p_usage JSONB
)
RETURNS VOID AS $$
BEGIN
  PERFORM 1 FROM public.accounts WHERE id = p_account_id;
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Cannot update feature usage for non-existent account';
  END IF;
  
  INSERT INTO public.feature_usage (account_id, feature, usage)
  VALUES (p_account_id, p_feature, p_usage)
  ON CONFLICT (account_id, feature)
  DO UPDATE SET usage = public.feature_usage.usage || p_usage,
                updated_at = NOW();
END;
$$ LANGUAGE plpgsql;

-- Function to update feature quota usage atomically using UPSERT
CREATE OR REPLACE FUNCTION public.update_feature_quota_usage(
  p_account_id UUID, 
  p_feature VARCHAR, 
  p_count INTEGER
)
RETURNS VOID 
SECURITY INVOKER 
AS $$
BEGIN
  INSERT INTO public.feature_usage (account_id, feature, usage, updated_at)
  VALUES (
    p_account_id, 
    p_feature, 
    jsonb_build_object('count', p_count),
    NOW()
  )
  ON CONFLICT (account_id, feature)
  DO UPDATE SET usage = jsonb_set(
      COALESCE(public.feature_usage.usage, '{}'::jsonb),
      '{count}',
      to_jsonb(
        COALESCE((public.feature_usage.usage->>'count')::INTEGER, 0) + p_count
      )
    ),
    updated_at = NOW();
END;
$$ LANGUAGE plpgsql;
```

Grant execute permissions securely:

```sql
-- Granting execute permissions to appropriate roles
GRANT EXECUTE ON FUNCTION public.can_use_feature(UUID, VARCHAR) TO authenticated;
GRANT EXECUTE ON FUNCTION public.get_entitlement(UUID, VARCHAR) TO authenticated;
GRANT EXECUTE ON FUNCTION public.update_feature_usage(UUID, VARCHAR, JSONB) TO service_role;
GRANT EXECUTE ON FUNCTION public.update_feature_quota_usage(UUID, VARCHAR, INTEGER) TO service_role;
```

## Step 3: Create the Entitlements Service in TypeScript

Encapsulate your entitlements logic in a TypeScript service. This service communicates with the Supabase backend via RPC and provides a clean API for your application.

```typescript {% title="apps/web/lib/server/entitlements.service.ts" %}
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '~/lib/database.types';

export function createEntitlementsService(
  client: SupabaseClient<Database>,
  accountId: string
) {
  return new EntitlementsService(client, accountId);
}

class EntitlementsService {
  constructor(
    private readonly client: SupabaseClient<Database>,
    private readonly accountId: string
  ) {}

  async canUseFeature(feature: string) {
    const { data, error } = await this.client.rpc('can_use_feature', {
      p_account_id: this.accountId,
      p_feature: feature,
    });

    if (error) throw error;

    return data;
  }

  async getEntitlement(feature: string) {
    const { data, error } = await this.client.rpc('get_entitlement', {
      p_account_id: this.accountId,
      p_feature: feature,
    });

    if (error) throw error;

    return data;
  }

  async updateFeatureUsage(feature: string, usage: Record<string, unknown>) {
    const { error } = await this.client.rpc('update_feature_usage', {
      p_account_id: this.accountId,
      p_feature: feature,
      p_usage: usage,
    });

    if (error) throw error;
  }
}
```

### How to Use the Service

In your API route or server component, you can use the service to check for entitlements and update usage data. For example:

```tsx
// Example API route or server action for handling an API request
import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { createEntitlementsService } from '~/lib/server/entitlements.service';

export async function handleApiRequest(accountId: string, endpoint: string) {
  const client = getSupabaseServerClient();
  const entitlementsService = createEntitlementsService(client, accountId);

  const canUseAPI = await entitlementsService.canUseFeature('api_access');
  
  if (!canUseAPI) {
    throw new Error('No access to API');
  }

  const entitlement = await entitlementsService.getEntitlement('api_calls');
  // Adjust processing based on entitlement type (flat, quota, etc.)
  if (entitlement && entitlement.entitlement.type === 'flat') {
    return processApiRequest(endpoint);
  } else if (entitlement && entitlement.entitlement.type === 'quota') {
    const currentUsage = Number(entitlement.usage?.count ?? 0);
    const limit = entitlement.entitlement.limit;

    if (currentUsage < limit) {
      // Atomically update usage count
      await entitlementsService.updateFeatureUsage('api_calls', { count: currentUsage + 1 });

      return processApiRequest(endpoint);
    } else {
      throw new Error('API call quota exceeded');
    }
  }
  throw new Error('Invalid entitlement state');
}
```

## Step 4: Enforcing Entitlements in Row Level Security

One of the major benefits of this approach is that you can enforce entitlements at the database level using RLS policies. For example, to restrict access to a table based on entitlement checks:

```sql
-- Example RLS policy using the can_use_feature function
CREATE POLICY "users_can_access_feature" ON public.some_table
  FOR SELECT
  TO authenticated
  USING (
    public.can_use_feature(auth.uid(), 'some_feature')
  );
```

This ensures that only users with the correct entitlements can access sensitive data.

## Step 5: Integrating with Billing Webhooks

When a billing event occurs (such as an invoice being paid), use a webhook to update entitlements accordingly. Below is an example using a Next.js API route with structured logging for observability:

```typescript
'use server';
import { enhanceRouteHandler } from '@kit/next/routes';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';
import { createEntitlementsService } from '~/lib/server/entitlements.service';
import { getLogger } from '@kit/shared/logger';
import { billingConfig } from '~/config/billing';

export const POST = enhanceRouteHandler(
  async ({ request }) => {
    const provider = billingConfig.provider;
    const logger = await getLogger();
    const ctx = { name: 'billing.webhook', provider };

    logger.info(ctx, 'Received billing webhook. Processing...');

    try {
      // Handle billing event using your billing event handler service...
      await handleInvoicePaidEvent(request, ctx);
      logger.info(ctx, 'Successfully processed billing webhook');
      return new Response('OK', { status: 200 });
    } catch (error) {
      logger.error({ ...ctx, error }, 'Failed to process billing webhook');
      return new Response('Failed to process billing webhook', { status: 500 });
    }
  },
  { auth: false }
);

async function handleInvoicePaidEvent(request: Request, ctx: Record<string, unknown>) {
  // Assume the request contains the account_id for which the invoice was paid
  const accountId = 'extracted-account-id'; // Extract account id securely from the request payload

  const entitlementsService = createEntitlementsService(getSupabaseServerAdminClient(), accountId);
  const entitlement = await entitlementsService.getEntitlement('api_calls');

  if (!entitlement) {
    ctx['error'] = `No entitlement found for "api_calls"`;
    throw new Error(ctx['error']);
  }

  const count = entitlement?.entitlement?.limit ?? 0;
  if (!count) {
    ctx['error'] = 'No limit found for "api_calls" entitlement';
    throw new Error(ctx['error']);
  }

  await entitlementsService.updateFeatureUsage('api_calls', { count });
  return;
}
```

## PgTap tests

Please add the following tests to your project and modify them as needed:

```sql
begin;
create extension "basejump-supabase_test_helpers" version '0.0.6';

select no_plan();

select tests.create_supabase_user('foreigner', 'foreigner@makerkit.dev');

-- Create test users
select makerkit.set_identifier('primary_owner', 'test@makerkit.dev');
select makerkit.set_identifier('member', 'member@makerkit.dev');
select makerkit.set_identifier('foreigner', 'foreigner@makerkit.dev');

-- Setup test data
set local role postgres;

-- Insert test plan entitlements
insert into public.plan_entitlements (variant_id, feature, entitlement)
values 
  ('basic_plan', 'api_calls', '{"limit": 1000, "period": "month"}'::jsonb),
  ('pro_plan', 'api_calls', '{"limit": 10000, "period": "month"}'::jsonb),
  ('basic_plan', 'storage', '{"limit": 5, "unit": "GB"}'::jsonb),
  ('pro_plan', 'storage', '{"limit": 50, "unit": "GB"}'::jsonb);

-- Create test billing customers and subscriptions
INSERT INTO public.billing_customers(account_id, provider, customer_id)
VALUES (makerkit.get_account_id_by_slug('makerkit'), 'stripe', 'cus_test');

-- Create a subscription with basic plan
SELECT public.upsert_subscription(
  makerkit.get_account_id_by_slug('makerkit'),
  'cus_test',
  'sub_test_basic',
  true,
  'active',
  'stripe',
  false,
  'usd',
  now(),
  now() + interval '1 month',
  '[{
    "id": "sub_basic",
    "product_id": "prod_basic",
    "variant_id": "basic_plan",
    "type": "flat",
    "price_amount": 1000,
    "quantity": 1,
    "interval": "month",
    "interval_count": 1
  }]'
);

-- Test as primary owner
select tests.authenticate_as('primary_owner');

-- Test reading plan entitlements
select isnt_empty(
  $$ select * from plan_entitlements where variant_id = 'basic_plan' $$,
  'Primary owner can read plan entitlements'
);

-- Test can_use_feature function
select is(
  (select public.can_use_feature(makerkit.get_account_id_by_slug('makerkit'), 'api_calls')),
  true,
  'Account with basic plan can use api_calls feature'
);

-- Test get_entitlement function
select row_eq(
  $$ select entitlement->>'limit' from public.get_entitlement(makerkit.get_account_id_by_slug('makerkit'), 'api_calls') $$,
  row('1000'::text),
  'Get entitlement returns correct limit for api_calls'
);

set local role service_role;

-- Test feature usage tracking
select lives_ok(
  $$ select public.update_feature_quota_usage(makerkit.get_account_id_by_slug('makerkit'), 'api_calls', 100) $$,
  'Can update feature quota usage'
);

-- Test as primary owner
select tests.authenticate_as('primary_owner');

-- Verify feature usage was recorded
select row_eq(
  $$ select usage->>'count' from feature_usage where account_id = makerkit.get_account_id_by_slug('makerkit') and feature = 'api_calls' $$,
  row('100'::text),
  'Feature usage is recorded correctly'
);

-- Test as member
select tests.authenticate_as('member');

-- Members can read plan entitlements
select isnt_empty(
  $$ select * from plan_entitlements $$,
  'Members can read plan entitlements'
);

-- Members can read feature usage for their account
select isnt_empty(
  $$ select * from feature_usage where account_id = makerkit.get_account_id_by_slug('makerkit') $$,
  'Members can read feature usage for their account'
);

-- Test as foreigner
select tests.authenticate_as('foreigner');

-- Foreigners can read plan entitlements (public info)
select isnt_empty(
  $$ select * from plan_entitlements $$,
  'Foreigners can read plan entitlements'
);

-- Foreigners cannot read feature usage for other accounts
select is_empty(
  $$ select * from feature_usage where account_id = makerkit.get_account_id_by_slug('makerkit') $$,
  'Foreigners cannot read feature usage for other accounts'
);

-- Test updating to pro plan
set local role postgres;

SELECT public.upsert_subscription(
  makerkit.get_account_id_by_slug('makerkit'),
  'cus_test',
  'sub_test_basic',
  true,
  'active',
  'stripe',
  false,
  'usd',
  now(),
  now() + interval '1 month',
  '[{
    "id": "sub_pro",
    "product_id": "prod_pro",
    "variant_id": "pro_plan",
    "type": "flat",
    "price_amount": 2000,
    "quantity": 1,
    "interval": "month",
    "interval_count": 1
  }]'
);

select tests.authenticate_as('primary_owner');

-- Verify pro plan entitlements
select row_eq(
  $$ select entitlement->>'limit' from public.get_entitlement(makerkit.get_account_id_by_slug('makerkit'), 'api_calls') $$,
  row('10000'::text),
  'Get entitlement returns updated limit for api_calls after plan upgrade'
);

-- Test edge cases
-- Test non-existent feature
select is(
  (select public.can_use_feature(makerkit.get_account_id_by_slug('makerkit'), 'non_existent_feature')),
  false,
  'Cannot use non-existent feature'
);

-- Test non-existent account
select is(
  (select public.can_use_feature('12345678-1234-1234-1234-123456789012'::uuid, 'api_calls')),
  false,
  'Cannot use feature for non-existent account'
);

-- Test updating feature usage with invalid data
set local role postgres;

select throws_ok(
  $$ select public.update_feature_usage('12345678-1234-1234-1234-123456789012'::uuid, 'api_calls', '{"invalid": true}'::jsonb) $$,
  'Cannot update feature usage for non-existent account'
);

-- Additional tests for subscription entitlements

--------------------------------------------------------------------
-- Additional tests for update_feature_quota_usage (storage feature)
--------------------------------------------------------------------
set local role postgres;

-- Create or update a subscription for storage feature if not already set
-- We'll use the basic plan for storage
SELECT public.upsert_subscription(
  makerkit.get_account_id_by_slug('makerkit'),
  'cus_test',
  'sub_test_storage',
  true,
  'active',
  'stripe',
  false,
  'usd',
  now(),
  now() + interval '1 month',
  '[{
    "id": "sub_storage",
    "product_id": "prod_storage",
    "variant_id": "basic_plan",
    "type": "flat",
    "price_amount": 500,
    "quantity": 1,
    "interval": "month",
    "interval_count": 1
  }]'
);

-- Reset storage usage by updating its quota
select lives_ok(
  $$ select public.update_feature_quota_usage(makerkit.get_account_id_by_slug('makerkit'), 'storage', 5) $$,
  'Initial storage quota update sets usage to 5'
);

select row_eq(
  $$ select usage->>'count' from feature_usage where account_id = makerkit.get_account_id_by_slug('makerkit') and feature = 'storage' $$,
  row('5'::text),
  'Storage usage should be 5 after initial update'
);

-- Update storage usage by adding 3 more units
select lives_ok(
  $$ select public.update_feature_quota_usage(makerkit.get_account_id_by_slug('makerkit'), 'storage', 3) $$,
  'Additional storage quota update adds 3 units'
);

select row_eq(
  $$ select usage->>'count' from feature_usage where account_id = makerkit.get_account_id_by_slug('makerkit') and feature = 'storage' $$,
  row('8'::text),
  'Accumulated storage usage should be 8'
);

set local role service_role;

-- Update api_calls usage by adding an extra field
select lives_ok(
  $$ select public.update_feature_usage(makerkit.get_account_id_by_slug('makerkit'), 'api_calls', '{"extra": 100}'::jsonb) $$,
  'Feature usage update concatenates new JSON data for api_calls'
);

-- Verify that the api_calls usage JSON now contains the extra field by checking the "extra" key value directly
select is(
  (select usage::json->>'extra' from feature_usage 
    where account_id = makerkit.get_account_id_by_slug('makerkit') 
      and feature = 'api_calls'),
  '100',
  'Feature usage for api_calls contains extra field after update'
);

--------------------------------------------------------------------
-- Additional test for non-existent subscription item entitlement
--------------------------------------------------------------------
select is_empty(
  $$ select * from public.get_entitlement(makerkit.get_account_id_by_slug('makerkit'), 'nonexistent_feature') $$,
  'Get entitlement returns empty for a non-existent feature'
);

--------------------------------------------------------------------
-- Additional test for atomicity of updating feature usage
--------------------------------------------------------------------
set local role postgres;

CREATE OR REPLACE FUNCTION test_atomicity_feature_usage() RETURNS text AS $$
DECLARE
    baseline text;
    current_usage text;
BEGIN
    -- Capture the baseline storage usage for the 'storage' feature
    SELECT usage->>'count' INTO baseline
    FROM feature_usage
    WHERE account_id = makerkit.get_account_id_by_slug('makerkit')
      AND feature = 'storage';

    BEGIN
        -- Perform a valid update: add 10 units to storage usage
        PERFORM public.update_feature_quota_usage(makerkit.get_account_id_by_slug('makerkit'), 'storage', 10);
        -- Force an error by updating usage for a non-existent account
        PERFORM public.update_feature_usage('00000000-0000-0000-0000-000000000000'::uuid, 'storage', '{"bad":1}'::jsonb);
        -- If no error is raised, return an error message (this should not happen)
        RETURN 'error not raised';
    EXCEPTION WHEN OTHERS THEN
        -- Exception caught; the subtransaction should be rolled back
        NULL;
    END;

    -- Capture the current usage after the forced error
    SELECT usage->>'count' INTO current_usage
    FROM feature_usage
    WHERE account_id = makerkit.get_account_id_by_slug('makerkit') AND feature = 'storage';

    IF current_usage = baseline THEN
         RETURN 'ok';
    ELSE
         RETURN 'failed';
    END IF;
END;
$$ LANGUAGE plpgsql;

select is((select test_atomicity_feature_usage()), 'ok', 'Atomicity of updating feature usage is preserved');

-- End of additional atomicity tests

select * from finish();

rollback; 
```

## Benefits of This Approach

1. **Flexibility:**  
   Handle different entitlement types—from simple feature flags to complex usage quotas—without locking into a rigid model.
2. **Performance:**  
   Offload entitlement checks to PostgreSQL. This minimizes round-trips between your app and the database.
3. **Consistency & Security:**  
   The same functions are used in both your application code and in RLS policies, ensuring a uniform level of security.
4. **Maintainability:**  
   Encapsulating logic in PostgreSQL functions and a dedicated TypeScript service simplifies updates and helps prevent bugs.

## Conclusion

By moving entitlement logic into PostgreSQL functions and encapsulating access in a dedicated TypeScript service, you create a robust and secure system that scales with your application. This approach not only meets the complex needs of SaaS applications but also adheres to best practices for performance, security, and maintainability.

Feel free to evolve these patterns further to suit your specific billing scenarios and business logic needs. Happy coding!

