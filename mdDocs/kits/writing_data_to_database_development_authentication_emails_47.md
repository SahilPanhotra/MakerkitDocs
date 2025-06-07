-----------------
FILE PATH: kits\next-supabase-turbo\development\writing-data-to-database.mdoc

---
status: "published"
label: "Writing data to Database"
order: 5
title: "Learn how to write data to the Supabase database in your Next.js app"
description: "In this page we learn how to write data to the Supabase database in your Next.js app"
---

In this page, we will learn how to write data to the Supabase database in your Next.js app.

{% sequence title="How to write data to the Supabase database" description="In this page we learn how to write data to the Supabase database in your Next.js app" %}

[Writing a Server Action to Add a Task](#writing-a-server-action-to-add-a-task)

[Defining a Schema for the Task](#defining-a-schema-for-the-task)

[Writing the Server Action to Add a Task](#writing-the-server-action-to-add-a-task)

[Creating a Form to Add a Task](#creating-a-form-to-add-a-task)

[Using a Dialog component to display the form](#using-a-dialog-component-to-display-the-form)

{% /sequence %}


## Writing a Server Action to Add a Task

Server Actions are defined by adding `use server` at the top of the function or file. When we define a function as a Server Action, it will be executed on the server-side.

This is useful for various reasons:
1. By using Server Actions, we can revalidate data fetched through Server Components
2. We can execute server side code just by calling the function from the client side

In this example, we will write a Server Action to add a task to the database.

### Defining a Schema for the Task

We use Zod to validate the data that is passed to the Server Action. This ensures that the data is in the correct format before it is written to the database.

The convention in Makerkit is to define the schema in a separate file and import it where needed. We use the convention `file.schema.ts` to define the schema.

```tsx
import { z } from 'zod';

export const WriteTaskSchema = z.object({
  title: z.string().min(1),
  description: z.string().nullable(),
});
```

### Writing the Server Action to Add a Task

In this example, we write a Server Action to add a task to the database. We use the `revalidatePath` function to revalidate the `/home` page after the task is added.

```tsx
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

import { z } from 'zod';

import { getLogger } from '@kit/shared/logger';
import { requireUser } from '@kit/supabase/require-user';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

import { WriteTaskSchema } from '~/(dashboard)/home/(user)/_lib/schema/write-task.schema';

export async function addTaskAction(params: z.infer<typeof WriteTaskSchema>) {
  'use server';

  const task = WriteTaskSchema.parse(params);

  const logger = await getLogger();
  const client = getSupabaseServerClient();
  const auth = await requireUser(client);

  if (!auth.data) {
    redirect(auth.redirectTo);
  }

  logger.info(task, `Adding task...`);

  const { data, error } = await client
    .from('tasks')
    .insert({ ...task, account_id: auth.data.id });

  if (error) {
    logger.error(error, `Failed to add task`);

    throw new Error(`Failed to add task`);
  }

  logger.info(data, `Task added successfully`);

  revalidatePath('/home', 'page');

  return null;
}
```

Let's focus on this bit for a second:

```tsx
const { data, error } = await client
    .from('tasks')
    .insert({ ...task, account_id: auth.data.id });
```

Do you see the `account_id` field? This is a foreign key that links the task to the user who created it. This is a common pattern in database design.

Now that we have written the Server Action to add a task, we can call this function from the client side. But we need a form, which we define in the next section.

### Creating a Form to Add a Task

We create a form to add a task. The form is a React component that accepts a `SubmitButton` prop and an `onSubmit` prop.

```tsx
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@kit/ui/form';
import { Input } from '@kit/ui/input';
import { Textarea } from '@kit/ui/textarea';
import { Trans } from '@kit/ui/trans';

import { WriteTaskSchema } from '../_lib/schema/write-task.schema';

export function TaskForm(props: {
  task?: z.infer<typeof WriteTaskSchema>;
  onSubmit: (task: z.infer<typeof WriteTaskSchema>) => void;
  SubmitButton: React.ComponentType;
}) {
  const form = useForm({
    resolver: zodResolver(WriteTaskSchema),
    defaultValues: props.task,
  });

  return (
    <Form {...form}>
      <form
        className={'flex flex-col space-y-4'}
        onSubmit={form.handleSubmit(props.onSubmit)}
      >
        <FormField
          render={(item) => {
            return (
              <FormItem>
                <FormLabel>
                  <Trans i18nKey={'tasks:taskTitle'} />
                </FormLabel>

                <FormControl>
                  <Input required {...item.field} />
                </FormControl>

                <FormDescription>
                  <Trans i18nKey={'tasks:taskTitleDescription'} />
                </FormDescription>

                <FormMessage />
              </FormItem>
            );
          }}
          name={'title'}
        />

        <FormField
          render={(item) => {
            return (
              <FormItem>
                <FormLabel>
                  <Trans i18nKey={'tasks:taskDescription'} />
                </FormLabel>

                <FormControl>
                  <Textarea {...item.field} />
                </FormControl>

                <FormDescription>
                  <Trans i18nKey={'tasks:taskDescriptionDescription'} />
                </FormDescription>

                <FormMessage />
              </FormItem>
            );
          }}
          name={'description'}
        />

        <props.SubmitButton />
      </form>
    </Form>
  );
}
```

### Using a Dialog component to display the form

We use the Dialog component from the `@kit/ui/dialog` package to display the form in a dialog. The dialog is opened when the user clicks on a button.

```tsx
'use client';

import { useState, useTransition } from 'react';

import { PlusCircle } from 'lucide-react';

import { Button } from '@kit/ui/button';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@kit/ui/dialog';
import { Trans } from '@kit/ui/trans';

import { TaskForm } from '../_components/task-form';
import { addTaskAction } from '../_lib/server/server-actions';

export function NewTaskDialog() {
  const [pending, startTransition] = useTransition();
  const [isOpen, setIsOpen] = useState(false);

  return (
    <Dialog open={isOpen} onOpenChange={setIsOpen}>
      <DialogTrigger asChild>
        <Button>
          <PlusCircle className={'mr-1 h-4'} />
          <span>
            <Trans i18nKey={'tasks:addNewTask'} />
          </span>
        </Button>
      </DialogTrigger>

      <DialogContent>
        <DialogHeader>
          <DialogTitle>
            <Trans i18nKey={'tasks:addNewTask'} />
          </DialogTitle>

          <DialogDescription>
            <Trans i18nKey={'tasks:addNewTaskDescription'} />
          </DialogDescription>
        </DialogHeader>

        <TaskForm
          SubmitButton={() => (
            <Button>
              {pending ? (
                <Trans i18nKey={'tasks:addingTask'} />
              ) : (
                <Trans i18nKey={'tasks:addTask'} />
              )}
            </Button>
          )}
          onSubmit={(data) => {
            startTransition(async () => {
              await addTaskAction(data);
              setIsOpen(false);
            });
          }}
        />
      </DialogContent>
    </Dialog>
  );
}
```

We can now import `NewTaskDialog` in the `/home` page and display the dialog when the user clicks on a button.

Let's go back to the home page and add the component right next to the input filter:

```tsx {18}
<div className={'flex items-center justify-between'}>
  <div>
    <Heading level={4}>
      <Trans i18nKey={'tasks:tasksTabLabel'} defaults={'Tasks'} />
    </Heading>
  </div>

  <div className={'flex items-center space-x-2'}>
    <form className={'w-full'}>
      <Input
        name={'query'}
        defaultValue={query}
        className={'w-full lg:w-[18rem]'}
        placeholder={'Search tasks'}
      />
    </form>

    <NewTaskDialog />
  </div>
</div>
```

-----------------
FILE PATH: kits\next-supabase-turbo\development.mdoc

---
status: "published"
title: "Development in Next.js Supabase Turbo"
label: "Development"
order: 4
description: "This section will teach you everything you need to know about developing with Makerkit."
collapsible: true
collapsed: true
---

Your roadmap to building with Makerkit. Here's what you need to know about development:

1. [**Getting Started with Local Development**](/docs/next-supabase-turbo/approaching-local-development)
   Set up your local development environment and understand the development workflow.
2. [**Working with Turborepo**](/docs/next-supabase-turbo/adding-turborepo-app)
   - [Adding a Turborepo App](/docs/next-supabase-turbo/adding-turborepo-app)
   - [Adding a Turborepo Package](/docs/next-supabase-turbo/adding-turborepo-package)
3. [**Database Development**](/docs/next-supabase-turbo/database-schema)
   - [Database Schema](/docs/next-supabase-turbo/database-schema)
   - [Database Functions](/docs/next-supabase-turbo/database-functions)
   - [Database Webhooks](/docs/next-supabase-turbo/database-webhooks)
   - [Working with Migrations](/docs/next-supabase-turbo/migrations)
4. [**Data Operations**](/docs/next-supabase-turbo/loading-data-from-database)
   - [Loading Data from Database](/docs/next-supabase-turbo/loading-data-from-database)
   - [Writing Data to Database](/docs/next-supabase-turbo/writing-data-to-database)
5. [**Access Control**](/docs/next-supabase-turbo/permissions-and-roles)
   Learn about roles, permissions, and access control in your application.
6. [**Content Management**](/docs/next-supabase-turbo/marketing-pages)
   - [Marketing Pages](/docs/next-supabase-turbo/marketing-pages)
   - [Legal Pages](/docs/next-supabase-turbo/legal-pages)
   - [External Marketing Website](/docs/next-supabase-turbo/external-marketing-website)
   - [Search Engine Optimization](/docs/next-supabase-turbo/seo)

Ready to start building? Pick a topic above and dive in! 🚀

-----------------
FILE PATH: kits\next-supabase-turbo\emails\authentication-emails.mdoc

---
status: "published"
title: "Setting the Supabase Auth Email Templates in Production"
label: "Authentication Emails"
description: "Configure the Supabase authentication URLs and emails in the Next.js Supabase Starter Kit."
order: 4
---

{% sequence title="Steps to set the Supabase Auth Email Templates in Production" description="Learn how to set the Supabase Auth Email Templates in Production." %}

[Setting the Email Templates in Supabase](#setting-the-email-templates-in-supabase)

[Customizing the Email Templates](#customizing-the-email-templates)

{% /sequence %}

Makerkit provides a set of email templates that you can use for replacing the standard Supabase Auth emails. Additionally, you can customize these templates to match your brand.

**Please update the auth emails using the [following documentation in Supabase](https://supabase.com/docs/guides/auth/auth-email-templates#redirecting-the-user-to-a-server-side-endpoint)**.

{% alert type="warning" %}
Failure to do so will result in hiccups in the authentication flow when users click on an email and get redirected to a different browser than the one they used to sign up due to how the PKCE flow works.
{% /alert %}

## Setting the Email Templates in Supabase

Why should you use our email templates?

1. They will use the token hash strategy, which remediates the issue of users being redirected to a different browser than the one they used to sign up.
2. They look better than the default Supabase templates and you can customize them to match your brand.

## Customizing the Email Templates

Please clone the templates repository locally and customize them to your liking:

1. Clone our Emails Starter at [https://github.com/makerkit/makerkit-emails-starter](https://github.com/makerkit/makerkit-emails-starter)
2. Customize the templates as you see fit
3. Export the templates with your own information
4. Replace the templates in the `apps/web/supabase/templates` folder
5. Update the email templates in your Supabase settings

Now your emails from Supabase Auth will look great and match your brand! 🎉

-----------------
FILE PATH: kits\next-supabase-turbo\emails\custom-mailer.mdoc

---
status: "published"
title: "Creating a Custom Mailer in the Next.js Supabase Starter Kit"
label: "Custom Mailer"
description: "Learn how to create a custom mailer in the Next.js Supabase Starter Kit, so you can send emails using your own email provider."
order: 5
---

{% sequence title="Steps to create a custom mailer" description="Learn how to create a custom mailer in the Next.js Supabase Starter Kit, so you can send emails using your own email provider." %}

[Implementing the sendEmail method](#implementing-the-sendemail-method)

[Registering the Mailer](#registering-the-mailer)

[Setting a new Mailer Provider](#setting-a-new-mailer-provider)

[Moving Megamailer to a separate package](#moving-megamailer-to-a-separate-package)

{% /sequence %}

Makerkit implements both Nodemailer (which can potentially support any SMTP provider) and Resend, using its HTTP API, which allows Makerkit to be compatible with edge runtimes (such as Cloudflare Workers).

However, if you want to use your own email provider, you can create a custom mailer using the `packages/mailers` package.

The mailer class is very simple:

```tsx
export abstract class Mailer<Res = unknown> {
  abstract sendEmail(data: z.infer<typeof MailerSchema>): Promise<Res>;
}
```

As such, you can create your own mailer by extending the `Mailer` class and implementing the `sendEmail` method.

## Implementing the sendEmail method

The `sendEmail` method takes in a `z.infer<typeof MailerSchema>` object, which is a Zod schema that defines the shape of the email data. This allows you to validate the email data before sending it.

Here's an example of how to implement the `sendEmail` method. For simplicity, we will call our custom mailer `megamailer` in the following examples.

```tsx {% title="packages/mailers/core/src/megamailer.ts" %}
import { z } from 'zod';

import { MailerSchema } from '../../shared/src/schema/mailer.schema';

export function createMegaMailer() {
  return new MegaMailer();
}

class MegaMailer implements Mailer<unknown> {
  async sendEmail(data: z.infer<typeof MailerSchema>) {
    // Implement your email sending logic here
    // For example, you can use Nodemailer to send emails
    // or use a third-party email provider like SendGrid

    return {};
  }
}
```

NB: for simplicity, we've placed the `megamailer.ts` file in the core package. However, if you have 5 minutes, you can move it to a separate package and import it in the `index.ts` file, just like we do for Nodemailer and Resend.

## Registering the Mailer

Now that we have our custom mailer, we need to register it in the `packages/mailers/core/src/registry.ts` file:

```tsx {% title="packages/mailers/core/src/registry.ts" %}
import { Mailer } from '@kit/mailers-shared';
import { createRegistry } from '@kit/shared/registry';

import { MailerProvider } from './provider-enum';

const mailerRegistry = createRegistry<Mailer, MailerProvider>();

// here we add our custom mailer to the registry
mailerRegistry.register('megamailer', async () => {
  const { createMegaMailer } = await import('@kit/megamailer');

  return createMegaMailer();
});

mailerRegistry.register('nodemailer', async () => {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    const { createNodemailerService } = await import('@kit/nodemailer');

    return createNodemailerService();
  } else {
    throw new Error(
      'Nodemailer is not available on the edge runtime. Please use another mailer.',
    );
  }
});

mailerRegistry.register('resend', async () => {
  const { createResendMailer } = await import('@kit/resend');

  return createResendMailer();
});

export { mailerRegistry };
```

## Setting a new Mailer Provider

By setting the `MAILER_PROVIDER` environment variable to `megamailer`, you can use the Megamailer mailer in your application.

```
MAILER_PROVIDER=megamailer
```

## Moving Megamailer to a separate package

Let's move the Megamailer to a separate package. To do so, we create a new package called `@kit/megamailer` and move the `megamailer.ts` file to the new package.

Let's create a package at `packages/mailers/megamailer`.

1. Copy both the `package.json` and `tsconfig.json` files from the `resend` package to the new package and rename the package to `@kit/megamailer`
2. Copy the `megamailer.ts` file to the new package
3. Update the `index.ts` file to export the new package and use the Megamailer

#### Install the Megamailer package

To install the new package, run the following command:

```
pnpm i "@kit/megamailer:workspace:*" --filter "@kit/mailers"
```

#### Update the Mailer Registry

Now that we have the new package, we need to update the `registry.ts` file to export the new package and use the Megamailer.

```tsx {% title="packages/mailers/core/src/registry.ts" %}
// here we add our custom mailer to the registry
mailerRegistry.register('megamailer', async () => {
  const { createMegaMailer } = await import('@kit/megamailer');

  return createMegaMailer();
});
```

-----------------
FILE PATH: kits\next-supabase-turbo\emails\email-configuration.mdoc

---
status: "published"
label: "Email Configuration"
description: "Learn how to configure the mailer provider to start sending emails from your Next.js Supabase Starter Kit."
title: "Email Configuration in the Next.js Supabase Starter Kit"
order: 0
---

{% sequence title="Sending Emails in Makerkit" description="Learn how to configure the mailer provider to start sending emails from your Next.js Supabase Starter Kit." %}

[Understanding Makerkit and Supabase Auth emails](#understanding-makerkit-and-supabase-auth-emails)

[Configuring the mailer provider](#configuring-the-mailer-provider)

[Nodemailer](#nodemailer)

[Resend](#resend)

{% /sequence %}

## Understanding Makerkit and Supabase Auth emails

Before we delve into the configuration details, it's crucial to distinguish between Makerkit emails and Supabase Auth emails.

1. **Makerkit Emails**: These are transactional emails used for actions like team member invitations, account deletion confirmations, and any additional ones you'll be implementing.
2. **Supabase Auth Emails**: These emails are used for authentication-related actions, such as email verification and password reset.

To have a comprehensive email setup in your application, you'll need to configure both Makerkit and Supabase Auth emails.

This guide focuses on setting up Makerkit emails. For Supabase Auth, please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp).

## Configuring the mailer provider

Makerkit offers the `@kit/mailers` package to configure and send emails, providing a straightforward API for email operations.

There are three mailer implementations provided by Makerkit:

1. `nodemailer`: This is the default mailer that leverages the `nodemailer` library. It's ideal for Node.js environments as it's compatible with any SMTP server, ensuring you're not tied to a specific provider.
2. `resend`: This mailer uses the [Resend](https://resend.com) API via HTTP. It's a suitable choice if you opt for Resend.

The following sections will guide you on configuring the mailer provider to start sending emails from your Next.js Supabase Starter Kit.

## Nodemailer

To specify the mailer provider, set the `MAILER_PROVIDER` environment variable in the `apps/web/.env` file. 

For instance, to use the `nodemailer` mailer, set `MAILER_PROVIDER` to `nodemailer`:

```bash
MAILER_PROVIDER=nodemailer
```

By default, `nodemailer` is used, so you don't need to set the `MAILER_PROVIDER` environment variable by yourself.

### SMTP Configuration

If you're using the `nodemailer` mailer, you'll need to set the SMTP configuration in your environment variables. Here's an example of the SMTP configuration:

```bash
EMAIL_USER=User # refer to your email provider's documentation
EMAIL_PASSWORD=your-password # refer to your email provider's documentation
EMAIL_HOST=smtp.your-email-provider.com # refer to your email provider's documentation
EMAIL_PORT=587 # or 465 for SSL
EMAIL_TLS=true # or false for SSL (see provider documentation)
EMAIL_SENDER=Makerkit <your-email@app.com> # refer to your email provider's documentation
```

The variables are:

1. `EMAIL_USER`: The email address username. This is provider-specific, so refer to your email provider's documentation.
2. `EMAIL_PASSWORD`: The email address password. This is provider-specific, so refer to your email provider's documentation.
3. `EMAIL_HOST`: The SMTP server host. This is provider-specific, so refer to your email provider's documentation.
4. `EMAIL_PORT`: The SMTP server port. This is provider-specific, so refer to your email provider's documentation.
5. `EMAIL_TLS`: The TLS configuration. This is provider-specific, so refer to your email provider's documentation. Generally, you can set it to `true`.
6. `EMAIL_SENDER`: The sender of the emails. This is usually the email address of your application. It's usually in the format `Sender Name <email@app.com>`.

### EMAIL_SENDER format

Sometimes, `EMAIL_SENDER` should be written in the format `Sender Name <email@app.com>` so that the email provider can identify the sender of the email. This setting can be different for different email providers, so please refer to your email provider's documentation.

{% alert type="warn" title="Double check the settings" %}
Please double check the settings above and confirm them with your email provider. If the settings are not correct, you won't be able to send emails.
{% /alert %}

## Resend

As an alternative, you can use [Resend](https://resend.com), a modern email service that provides a simple API for sending emails.

Set the `MAILER_PROVIDER` environment variable to `resend` in the `apps/web/.env` file:

```bash
MAILER_PROVIDER=resend
```

And provide the Resend API key:

```bash
RESEND_API_KEY=your-api-key
EMAIL_SENDER=your-email
```

That's it! You're now ready to send emails from your Next.js Supabase Starter Kit using the configured mailer provider.

-----------------
FILE PATH: kits\next-supabase-turbo\emails\email-templates.mdoc

---
status: "published"
label: "Email Templates"
description: "Learn how to write email templates in the Next.js Supabase Starter Kit with React.Email"
title: "Email Templates in the Next.js Supabase Starter Kit"
order: 2
---

{% sequence title="Email Templates" description="Learn how to write email templates in the Next.js Supabase Starter Kit with React.Email" %}

[Where to find the email templates](#where-to-find-the-email-templates)

[Example of an email template in the kit](#example-of-an-email-template-in-the-kit)

[Creating your own email templates](#creating-your-own-email-templates)

{% /sequence %}

Email templates are a great way to send beautiful and consistent emails to your users. In the Next.js Supabase Starter Kit, we use [React.Email](https://react.email) to create email templates.

## Where to find the email templates

Templates are stored in the package `@kit/email-templates` which you can find in the `packages/email-templates` directory.

These templates use [React.Email](https://react.email) to create beautiful-looking email templates.

## Example of an email template in the kit

For example, here is our template for accepting an invitation to join a team:

```tsx
import {
  Body,
  Button,
  Column,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Img,
  Link,
  Preview,
  Row,
  Section,
  Tailwind,
  Text,
  render,
} from '@react-email/components';

interface Props {
  teamName: string;
  teamLogo?: string;
  inviter: string | undefined;
  invitedUserEmail: string;
  link: string;
  productName: string;
}

export function renderInviteEmail(props: Props) {
  const previewText = `Join ${props.invitedUserEmail} on ${props.productName}`;

  return render(
    <Html>
      <Head />
      <Preview>{previewText}</Preview>

      <Tailwind>
        <Body className="mx-auto my-auto bg-gray-50 font-sans">
          <Container className="mx-auto my-[40px] w-[465px] rounded-lg border border-solid border-[#eaeaea] bg-white p-[20px]">
            <Heading className="mx-0 my-[30px] p-0 text-center text-[24px] font-normal text-black">
              Join <strong>{props.teamName}</strong> on{' '}
              <strong>{props.productName}</strong>
            </Heading>
            <Text className="text-[14px] leading-[24px] text-black">
              Hello {props.invitedUserEmail},
            </Text>
            <Text className="text-[14px] leading-[24px] text-black">
              <strong>{props.inviter}</strong> has invited you to the{' '}
              <strong>{props.teamName}</strong> team on{' '}
              <strong>{props.productName}</strong>.
            </Text>
            {props.teamLogo && (
              <Section>
                <Row>
                  <Column align="center">
                    <Img
                      className="rounded-full"
                      src={props.teamLogo}
                      width="64"
                      height="64"
                    />
                  </Column>
                </Row>
              </Section>
            )}
            <Section className="mb-[32px] mt-[32px] text-center">
              <Button
                className="rounded bg-[#000000] px-[20px] py-[12px] text-center text-[12px] font-semibold text-white no-underline"
                href={props.link}
              >
                Join {props.teamName}
              </Button>
            </Section>
            <Text className="text-[14px] leading-[24px] text-black">
              or copy and paste this URL into your browser:{' '}
              <Link href={props.link} className="text-blue-600 no-underline">
                {props.link}
              </Link>
            </Text>
            <Hr className="mx-0 my-[26px] w-full border border-solid border-[#eaeaea]" />
            <Text className="text-[12px] leading-[24px] text-[#666666]">
              This invitation was intended for{' '}
              <span className="text-black">{props.invitedUserEmail}</span>.
            </Text>
          </Container>
        </Body>
      </Tailwind>
    </Html>,
  );
}
```

## Creating your own email templates

If you want to create your own email templates, you can create a new file in the `packages/email-templates/src/emails` directory and export a function that returns the email template.

Now, export the function from the package.

```tsx {% filename="packages/email-templates/src/index.ts" %}
export * from './emails/example';
```

Then, import the function and transform the template into HTML that you can send using the mailer.

```tsx
import { getMailer } from '@kit/mailers';
import { renderInviteEmail } from '@kit/email-templates';

async function sendEmail() {
  const emailHtml = renderInviteEmail({
    teamName: 'My Team',
    teamLogo: 'https://example.com/logo.png',
    inviter: 'John Doe',
    invitedUserEmail: ''
  });

  const mailer = await getMailer();

  return mailer.sendEmail({
    to: '',
    from: '',
    subject: 'Join the team!',
    html: emailHtml
  });
}
```

-----------------
FILE PATH: kits\next-supabase-turbo\emails\inbucket.mdoc

---
status: "published"
title: "InBucket for local development in the Next.js Supabase Starter Kit"
label: "Local Development"
description: "Learn how to use InBucket/Mailpit for local development in the Next.js Supabase Starter Kit."
order: 4
---

{% sequence title="Steps to use InBucket/Mailpit" description="Learn how to use InBucket/Mailpit for local development in the Next.js Supabase Starter Kit." %}

[What is Mailpit?](#what-is-mailpit)

[Accessing Mailpit](#accessing-mailpit)

[Common Use Cases](#common-use-cases)

[Production Emails](#mailpit-vs-production-emails)

{% /sequence %}

When developing locally, Supabase uses [InBucket](https://www.inbucket.org/) and [Mailpit](https://mailpit.axllent.org) (depending on which version you're running) to capture emails sent during development. This allows you to test email functionality without actually sending emails to real addresses.

Makerkit also uses InBucket/Mailpit for sending non-authentication emails, such as team invitations and account deletion confirmations.

## What is Mailpit?

Mailpit is a disposable email service that captures all emails sent during local development. It's particularly useful for testing:

- Email verification flows
- Password reset emails
- Team invitations
- Notification emails
- Any other transactional emails

## Accessing Mailpit

When running Supabase locally, Mailpit is automatically available at:

```bash
http://localhost:54324
```

You can view any captured emails by:

1. Going to [http://localhost:54324](http://localhost:54324)
2. Finding your test email in the list of received messages

## Common Use Cases

### 1. Email Verification

When testing user signup:
1. Create a new account using any email address
2. Go to InBucket/Mailpit at [http://localhost:54324](http://localhost:54324)
3. Find the verification email
4. Click the verification link or copy the verification code

### 2. Password Reset

To test password reset flows:
1. Request a password reset using any email
2. Check InBucket/Mailpit for the reset email
3. Use the reset link or code to complete the process

### 3. Team Invitations

When testing team invites:
1. Send an invitation to any email address
2. Check Mailpit for the invitation email
3. Use the invitation link to accept

## Mailpit vs Production Emails

Remember that Mailpit is for development only. In production:

1. Configure a proper email service provider (Resend, SendGrid, etc.)
2. Set up proper email authentication (SPF, DKIM)
3. Use production-grade SMTP settings
4. Monitor email deliverability

## Using a different email provider during local development

If you want to use a different email provider, switch the environment variables at `apps/web/.env.development`:

```bash
EMAIL_SENDER=test@makerkit.dev
EMAIL_PORT=
EMAIL_HOST=
EMAIL_TLS=
EMAIL_USER=
EMAIL_PASSWORD=
```

NB: Supabase authentication emails will still be sent using InBucket/Mailpit.

## Conclusion

InBucket/Mailpit are valuable tools for local development, allowing you to:
- Test email flows without sending real emails
- Iterate quickly on email templates
- Debug email-related issues
- Develop with confidence

Remember to switch to a proper email provider when deploying to production!

-----------------
FILE PATH: kits\next-supabase-turbo\emails\sending-emails.mdoc

---
status: "published"
label: "Sending Emails"
description: "Learn how to send emails in the Next.js Supabase SaaS Starter Kit."
title: "Sending Emails in the Next.js Supabase SaaS Starter Kit"
order: 1
---

{% sequence title="Steps to send emails" description="Learn how to send emails in the Next.js Supabase SaaS Starter Kit." %}

[Understanding the Mailer class](#understanding-the-mailer-class)

[Sending an email using the default mailer](#sending-an-email-using-the-default-mailer)

[Sending an HTML email](#sending-an-html-email)

{% /sequence %}

## Understanding the Mailer class

The Mailer class is extremely simple:

```tsx
import { z } from 'zod';

import { MailerSchema } from './schema/mailer.schema';

export abstract class Mailer<Res = unknown> {
  abstract sendEmail(data: z.infer<typeof MailerSchema>): Promise<Res>;
}
```

The `sendEmail` method is an abstract method that you need to implement in your mailer provider. The method receives an object with the following properties:

## Sending an email using the default mailer

Once you have configured the mailer provider, you can start sending emails using the `sendEmail` method. Here is an example of how to send an email using the default mailer:

```tsx
import { getMailer } from '@kit/mailers';

async function sendEmail(params: {
  from: string;
  to: string;
}) {
  const mailer = await getMailer();

  return mailer.sendEmail({
    to: params.from,
    from: params.to,
    subject: 'Hello',
    text: 'Hello, World!'
  });
}
```

The `sendEmail` method returns a promise that resolves when the email is sent successfully. If there is an error, the promise will be rejected with an error message.

## Sending an HTML email

If you want to send HTML emails, you can use the `html` property instead of the `text` property:

```tsx
import { getMailer } from '@kit/mailers';

async function sendEmail(params: {
  from: string;
  to: string;
}) {
  const mailer = await getMailer();

  return mailer.sendEmail({
    to: params.from,
    from: params.to,
    subject: 'Hello',
    html: '<h1>Hello, World!</h1>'
  });
}
```

Et voilà! You are now ready to send emails from your Next.js Supabase Starter Kit. 🚀

