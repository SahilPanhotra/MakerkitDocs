-----------------
FILE PATH: kits\next-supabase-turbo\emails\custom-mailer.mdoc

---
status: "published"
title: "Creating a Custom Mailer in the Next.js Supabase Starter Kit"
label: "Custom Mailer"
description: "Learn how to create a custom mailer in the Next.js Supabase Starter Kit, so you can send emails using your own email provider."
order: 5
---

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

Now that we have our custom mailer, we need to register it in the `packages/mailers/core/src/index.ts` file:

```tsx {% title="packages/mailers/core/src/index.ts" %}
import { z } from 'zod';

const MAILER_PROVIDER = z
  .enum(['nodemailer', 'resend', 'megamailer'])
  .default('nodemailer')
  .parse(process.env.MAILER_PROVIDER);

/**
 * @description Get the mailer based on the environment variable.
 */
export async function getMailer() {
  switch (MAILER_PROVIDER) {
    case 'nodemailer':
      return getNodemailer();

    case 'resend': {
      const { createResendMailer } = await import('@kit/resend');

      return createResendMailer();
    }

    case 'megamailer': {
      const { createMegamailer } = await import('./megamailer');

      return createMegamailer();
    }

    default:
      throw new Error(`Invalid mailer: ${MAILER_PROVIDER as string}`);
  }
}

async function getNodemailer() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    const { createNodemailerService } = await import('@kit/nodemailer');

    return createNodemailerService();
  } else {
    throw new Error(
      'Nodemailer is not available on the edge runtime. Please use another mailer.',
    );
  }
}
```

By setting the `MAILER_PROVIDER` environment variable to `megamailer`, you can use the Megamailer mailer in your application.

```
MAILER_PROVIDER=megamailer
```

### Moving Megamailer to a separate package

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

#### Update the `index.ts` file

Now that we have the new package, we need to update the `index.ts` file to export the new package and use the Megamailer.

```tsx {% title="packages/mailers/core/src/index.ts" %}
import { z } from 'zod';

const MAILER_PROVIDER = z
  .enum(['nodemailer', 'resend', 'megamailer'])
  .default('nodemailer')
  .parse(process.env.MAILER_PROVIDER);

/**
 * @description Get the mailer based on the environment variable.
 */
export async function getMailer() {
  switch (MAILER_PROVIDER) {
    case 'nodemailer':
      return getNodemailer();

    case 'resend': {
      const { createResendMailer } = await import('@kit/resend');

      return createResendMailer();
    }

    case 'megamailer': {
      const { createMegamailer } = await import('@kit/megamailer');

      return createMegamailer();
    }

    default:
      throw new Error(`Invalid mailer: ${MAILER_PROVIDER as string}`);
  }
}

async function getNodemailer() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    const { createNodemailerService } = await import('@kit/nodemailer');

    return createNodemailerService();
  } else {
    throw new Error(
      'Nodemailer is not available on the edge runtime. Please use another mailer.',
    );
  }
}
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


Before we delve into the configuration details, it's crucial to distinguish between Makerkit emails and Supabase Auth emails.

1. **Makerkit Emails**: These are transactional emails used for actions like team member invitations, account deletion confirmations, and any additional ones you'll be implementing.
2. **Supabase Auth Emails**: These emails are used for authentication-related actions, such as email verification and password reset.

To have a comprehensive email setup in your application, you'll need to configure both Makerkit and Supabase Auth emails.

This guide focuses on setting up Makerkit emails. For Supabase Auth, please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp).

---

Makerkit offers the `@kit/mailers` package to configure and send emails, providing a straightforward API for email operations.

There are three mailer implementations provided by Makerkit:

1. `nodemailer`: This is the default mailer that leverages the `nodemailer` library. It's ideal for Node.js environments as it's compatible with any SMTP server, ensuring you're not tied to a specific provider.
2. `resend`: This mailer uses the [Resend](https://resend.com) API via HTTP. It's a suitable choice if you opt for Resend.

The following sections will guide you on configuring the mailer provider to start sending emails from your Next.js Supabase Starter Kit.

## Configuration

To specify the mailer provider, set the `MAILER_PROVIDER` environment variable in the `apps/web/.env` file. For instance, to use the `nodemailer` mailer, set `MAILER_PROVIDER` to `nodemailer`:

```bash
MAILER_PROVIDER=nodemailer
```

By default, `nodemailer` is used.

### SMTP Configuration

If you're using the `nodemailer` mailer, you'll need to set the SMTP configuration in your environment variables. Here's an example of the SMTP configuration:

```bash
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_HOST=
EMAIL_PORT=
EMAIL_TLS=
EMAIL_SENDER=your-email
```

The variables are:

1. `EMAIL_USER`: The email address user. This is provider-specific, so refer to your email provider's documentation.
2. `EMAIL_PASSWORD`: The password for the email account, provided by your email provider.
3. `EMAIL_HOST`: The SMTP server host. This is provider-specific, so refer to your email provider's documentation.
4. `EMAIL_PORT`: The SMTP server port. This is provider-specific, so refer to your email provider's documentation.
5. `EMAIL_TLS`: The TLS configuration. This is provider-specific, so refer to your email provider's documentation. Generally, you can set it to `true`.
6. `EMAIL_SENDER`: The sender of the emails. This is usually the email address of your application. It's usually in the format `Sender Name <email@app.com>`.

Sometimes, `EMAIL_SENDER` should be written in the format `Sender Name <email@app.com>`. It depends on your SMTP provider.

## Resend

As an alternative, you can use Resend.

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


Email templates are a great way to send beautiful and consistent emails to your users. In the Next.js Supabase Starter Kit, we use [React.Email](https://react.email) to create email templates.

Templates are stored in the package `@kit/email-templates` which you can find in the `packages/email-templates` directory.

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

If you want to create your own email templates, you can create a new file in the `packages/email-templates/src` directory and export a function that returns the email template.

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

When developing locally, Supabase uses [InBucket](https://www.inbucket.org/) and [Mailpit](https://mailpit.axllent.org) (depending on which version you're running) to capture emails sent during development. This allows you to test email functionality without actually sending emails to real addresses.

Makerkit also uses InBucket/Mailpit for sending non-authentication emails, such as team invitations and account deletion confirmations.

## What is InBucket/Mailpit?

InBucket/Mailpit are disposable email services that capture all emails sent during local development. It's particularly useful for testing:

- Email verification flows
- Password reset emails
- Team invitations
- Notification emails
- Any other transactional emails

## Accessing InBucket/Mailpit

When running Supabase locally, InBucket/Mailpit are automatically available at:

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
2. Check InBucket/Mailpit for the invitation email
3. Use the invitation link to accept

## InBucket/Mailpit vs Production Emails

Remember that InBucket/Mailpit is for development only. In production:

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

The Mailer class is extremely simple:

```tsx
import { z } from 'zod';

import { MailerSchema } from './schema/mailer.schema';

export abstract class Mailer<Res = unknown> {
  abstract sendEmail(data: z.infer<typeof MailerSchema>): Promise<Res>;
}
```

The `sendEmail` method is an abstract method that you need to implement in your mailer provider. The method receives an object with the following properties:

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

-----------------
FILE PATH: kits\next-supabase-turbo\emails.mdoc

---
status: "published"
title: "Emails in Next.js Supabase Turbo"
label: "Emails"
order: 11
description: "Emails in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Everything you need to know about handling emails in your Makerkit application:

1. [**Getting Started with Email**](/docs/next-supabase-turbo/email-configuration)
   Configure your email provider and understand the email system architecture.
2. [**Authentication Emails**](/docs/next-supabase-turbo/authentication-emails)
   Set up and customize authentication-related emails like verification, password reset, and login notifications.
3. [**Email Templates**](/docs/next-supabase-turbo/email-templates)
   Learn how to create and customize email templates using React.Email.
4. [**Sending Emails**](/docs/next-supabase-turbo/sending-emails)
   Implement email sending functionality in your application using the Makerkit mailer system.

Whether you're setting up transactional emails, authentication notifications, or marketing communications, this guide has you covered! 📧


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\authentication-emails.mdoc

---
title: "Setting the Supabase Auth Email Templates in Production"
label: "Authentication Emails"
description: "Configure the Supabase authentication URLs and emails in the Next.js Supabase Starter Kit."
order: 4
---

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
FILE PATH: kits\next-supabase-turbo\going-to-production\authentication.mdoc

---
status: "published"
title: "Setting the Supabase Auth Settings in Production | Next.js Supabase SaaS Kit"
label: "Authentication"
description: "Configuring Authentication's production configuration in the Next.js Supabase SaaS kit"
order: 3
---

Supabase needs a few settings to be configured in their Dashboard to work correctly.

This guide will walk you through the steps to get your Supabase authentication set up in your Production environment. The dev environment does not require any configuration.

{% alert type="warning" %}
Skipping this step will result in your users not being able to log in or sign up.
{% /alert %}

## Third Party Providers

Third Party providers need to be configured, managed and enabled fully on the provider's and Supabase's side. Makerkit does not need any configuration (beyond setting the provider to be displayed in the UI).

Please [read Supabase's documentation](https://supabase.com/docs/guides/auth/social-login) on how to set up third-party providers.

## Authentication URLs

The first thing you need to do is to [set the authentication URLs in the Supabase Dashboard](https://supabase.com/docs/guides/auth/redirect-urls). These URLs are used to redirect users to the correct page after they have logged in or signed up.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Authentication** tab.
4. Click on **URL Configuration**.
5. Add your Site URL to the **Site URL** field. This is the URL of your MakerKit site (e.g. `https://my-site.com`).
6. Add your Redirect URLs to the **Redirect URLs** field. This is the URL of your MakerKit site with `/auth/callback` appended to it (e.g. `https://my-site.com/auth/callback`).

Failure to set these URLs will result in your users not being able to log in or sign up.

## Custom SMTP

Supabase's SMTP settings have low limits and low deliverability. If you want your emails to go out and be delivered, please remember to set an alternative SMTP provider in the Supabase settings.

1. Go to the [Supabase Dashboard](https://app.supabase.io/).
2. Click on the project you want to use.
3. Go to the **Project Settings** tab.
4. Click on **Auth**.
5. Tweak the `SMTP Settings` settings to your liking according to your provider's documentation.

Please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp) for more information.

### Troubleshooting

If you are having authentication issues, ensure that the Site URL and Redirect URLs are correct. If you are using a custom domain, ensure that you are using the correct domain in the Site URL and Redirect URLs.

{% alert type="warning" %}
NB: if your domain includes "www", ensure you include it in the Site URL and Redirect URLs. If your domain does not
  include "www", ensure you do not include it in the Site URL and Redirect URLs. If these do not match, your users will
  not be able to login.
{% /alert %}

If something is still not working, please open a support ticket with any useful information (such as server logs).


-----------------
FILE PATH: kits\next-supabase-turbo\going-to-production\checklist.mdoc

---
status: "published"
title: "Checklist for deploying your Next.js Supabase SaaS to Production"
label: "Checklist"
description: "This guide will help you deploy your Next.js Supabase SaaS app to production! Please follow this checklist to ensure everything is set up correctly."
order: 0
---

When you're ready to deploy your project to production, follow this checklist.

{% alert type="warning" title="Please complete the following steps" %}
Not completing these steps will result in your project not working correctly. Please make sure to complete these steps before testing your project.
{% /alert %}

This process may take a few hours and some trial and error, so buckle up—you're almost there!

1. **Create a Supabase project**. [Link the project locally](https://supabase.com/docs/guides/cli/local-development#link-your-project) using the Supabase CLI (`supabase link`).
2. **Migrations**: [Push the migrations](https://supabase.com/docs/guides/cli/local-development#link-your-project) to the remote Supabase instance (`supabase db push`).
3. **Auth**: [Set your APP URL in the Supabase project settings](https://supabase.com/docs/guides/auth/redirect-urls). This is required for the OAuth flow. Make sure to add the path `/auth/callback**` to the allowed URLs. If you don't have it yet, wait.
4. **Auth Providers**: [Set the OAuth providers in the Supabase](https://supabase.com/docs/guides/auth/social-login) project settings. If you use Google Auth, make sure to set it up. This requires creating a Google Cloud project and setting up the OAuth credentials.
5. **Auth Emails**: It is very much recommended to update the auth emails using the [following documentation](authentication-emails). The kit already implements the `confirm` route, but you need to update the emails in your Supabase settings.
6. **Deploy Next.js**: Deploy the Next.js app to Vercel or any another hosting provider. Copy the URL and set it in the Supabase project settings.
7. **Environment Variables**: The initial deploy **will likely fail** because you may not yet have a URL to set in your environment variables. This is normal. Once you have the URL, set the URL in the environment variables and redeploy.
8. **Webhooks**: [Set the DB Webhooks in Supabase](supabase) pointing against your Next.js app at `/api/db/webhooks`.
9. **Emails**: Get some SMTP details from an email service provider like SendGrid or Mailgun or Resend and configure the emails in both the Environment Variables and the [Supabase project settings](https://supabase.com/docs/guides/auth/auth-smtp).
10. **Billing**: Create a Stripe/Lemon Squeezy account, make sure to update the environment variables with the correct values. Point webhooks from these to `/api/billing/webhook`. Please use the relative documentation for more details.
11. **OTP (optional)**: Clear OTPs using a Cron Job or do so manually

Other minor things to consider:

1. Update the legal pages with your company's information (privacy policy, terms of service, etc.).
2. Remove the placeholder blog and documentation content / or replace it with your own
3. Update the favicon and logo with your own branding
4. Update the FAQ and other static content with your own information

---

1. **Create a Supabase Project**
   - **Why it's necessary:** Creating a Supabase project is the foundational step to set up your database, authentication, and storage services in the cloud. This will serve as the backend for your SaaS application.
   - **Action:** Create a new Supabase project in the Supabase dashboard. Once created, [link the project locally](https://supabase.com/docs/guides/cli/local-development#link-your-project) using the Supabase CLI:
     ```bash
     supabase link
     ```
2. **Migrations**
   - **Why it's necessary:** Pushing database migrations ensures that your database schema in the remote Supabase instance is configured to match Makerkit's requirements. This step is crucial for the application to function correctly.
   - **Action:** Push the database migrations to your remote Supabase instance:
     ```bash
     supabase db push
     ```
3. **Auth Configuration**
   - **Why it's necessary:** Setting your APP URL in the Supabase project settings is critical for enabling OAuth flows and redirecting users correctly during authentication.
   - **Action:** [Set your APP URL in the Supabase project settings](https://supabase.com/docs/guides/auth/redirect-urls). Add the path `/auth/callback` to the allowed URLs.
4. **OAuth Providers**
   - **Why it's necessary:** Configuring OAuth providers like Google ensures that users can log in using their existing accounts, enhancing user convenience and security. This is all done externally, in both Google and Supabase - not in the application code.
   - **Action:** [Set up the OAuth providers](https://supabase.com/docs/guides/auth/social-login) in your Supabase project settings. For Google Auth, create a Google Cloud project and set up the OAuth credentials.
5. **Auth Emails**
   - **Why it's necessary:** To provide a correct user experience with Makerkit's SSR authentication, you need to update the authentication emails to include the token hash and prevent errors usually related to PKCE - i.e when users click on the email and are redirected to a different browser - resulting in an error.
   - **Action:** Update the authentication emails using the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp). The kit implements the `confirm` route, but you need to update the email templates in your Supabase settings.
6. **Deploy Next.js**
   - **Why it's necessary:** Because your users are waiting! Deploying your Next.js app to a hosting provider makes it accessible to users worldwide, allowing them to interact with your application.
   - **Action:** Deploy your Next.js app to Vercel or another hosting provider. Copy the deployment URL and set it in your Supabase project settings.
7. **Environment Variables**
   - **Why it's necessary:** Setting the correct environment variables is essential for the application to function correctly. These variables include API keys, database URLs, and other configuration details required for your app to connect to various services.
   - **Action:** Please [generate the environment variables using our script](production-environment-variables) and then add them to your hosting provider's environment variables. Redeploy the app once you have the URL to set in the environment variables.
8. **Webhooks**
   - **Why it's necessary:** Configuring database webhooks allows your application to respond to changes in the database in real-time, such as sending notifications or updating records, ensuring your app stays in sync with the database.
   - **Action:** [Set up the database webhooks in Supabase](https://supabase.com/docs/guides/database/webhooks) to point to your Next.js app at `/api/db/webhooks`.
9. **Emails Configuration**
   - **Why it's necessary:** Properly configuring your email service ensures that your application can send emails for account verification, password resets, and other notifications, which are crucial for user communication.
   - **Action:** Obtain SMTP details from an email service provider such as SendGrid, Mailgun, or Resend. Configure the emails in both the environment variables and the [Supabase project settings](https://supabase.com/docs/guides/auth/auth-smtp).
10. **Billing Setup**
    - **Why it's necessary:** Well - you want to get paid, right? Setting up billing ensures that you can charge your users for using your SaaS application, enabling you to monetize your service and cover operational costs. This can take a while.
    - **Action:** Create a Stripe or Lemon Squeezy account. Update the environment variables with the correct values for your billing service. Point webhooks from Stripe or Lemon Squeezy to `/api/billing/webhook`. Refer to the relevant documentation for more details on setting up billing.
11. **OTPs (optional):
    - **Why it's necessary:** to remove unused DB records, we provide a function `kit.cleanup_expired_nonces()` to automatically clear unused or expired tokens
    - **Action:** From your Supabase Studio, launch `select kit.cleanup_expired_nonces()` manually, or create a cron job to do so at a frequency of your choice

**Note**: Please note that these steps are essential for setting up Makerkit and ensuring that your SaaS application functions correctly. Omitting any of these steps may result in errors or unexpected behavior in your application.


