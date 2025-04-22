-----------------
FILE PATH: kits\react-router-supabase-turbo\development\database-webhooks.mdoc

---
status: "published"

label: "Database Webhooks"
order: 6
title: "How to handle custom database webhooks in your React Router Supabase application"
description: "Learn how to handle custom database webhooks in your React Router Supabase application"
---


Database webhooks allow you to listen to changes in your database and trigger custom logic when a change occurs. This is useful for sending notifications, updating caches, or triggering other services.

Makerkit handles some webhooks by default for functionalities such as deleting a subscription following a user deletion, or sending emails after a user signs up.

You can extend this functionality by adding your own webhook handlers:

 ```tsx {% title="apps/web/app/api/db/webhook/route.ts" %}
import {
  getDatabaseWebhookHandlerService,
} from '@kit/database-webhooks';

/**
 * @name POST
 * @description POST handler for the webhook route that handles the webhook event
 */
export async function POST(request: Request) {
  const service = getDatabaseWebhookHandlerService();

  try {
    // handle the webhook event
    await service.handleWebhook(request, {
      handleEvent(change) {
        if (change.type === 'INSERT' && change.table === 'invitations') {
          // do something with the invitation
        }
      },
    });

    return new Response(null, { status: 200 });
  } catch {
    return new Response(null, { status: 500 });
  }
}
```

As you can see above - the `handleEvent` function is where you can add your custom logic to handle the webhook event. In this example, we check if the event is an `INSERT` event on the `invitations` table and then do something with the invitation.

The `change` object is of type `RecordChange` and contains the following properties:

```tsx
import { Database } from '@kit/supabase/database';

export type Tables = Database['public']['Tables'];

export type TableChangeType = 'INSERT' | 'UPDATE' | 'DELETE';

export interface RecordChange<
  Table extends keyof Tables,
  Row = Tables[Table]['Row'],
> {
  type: TableChangeType;
  table: Table;
  record: Row;
  schema: 'public';
  old_record: null | Row;
}
```

You may need to cast the type manually:

```tsx
type AccountChange = RecordChange<'accounts'>;
```

Now, the `AccountChange` type can be used to type the `change` object in the `handleEvent` function and is typed to the `accounts` table.

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\legal-pages.mdoc

---
status: "published"
label: "Legal Pages"
title: "Legal Pages in the React Router Supabase Turbo Starter Kit"
description: "Learn how to create and update legal pages in the React Router Supabase Turbo Starter Kit."
order: 9
---

Legal pages are defined in the `apps/web/app/routes/marketing` directory.

Makerkit comes with the following legal pages:

1. Terms and Conditions
2. Privacy Policy
3. Cookie Policy

For obvious reasons, **these pages are empty and you need to fill in the content**.

Do yourself a favor and do not use ChatGPT to generate these pages.

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\marketing-pages.mdoc

---
status: "published"
label: "Marketing Pages"
title: "Marketing Pages in the React Router Supabase Turbo Starter Kit"
description: "Learn how to create and update marketing pages in the React Router Supabase Turbo Starter Kit."
order: 7
---

Makerkit comes with pre-defined marketing pages to help you get started with your SaaS application. These pages are built with React Router and Tailwind CSS and are located in the `apps/web/app/(marketing)` directory.

Makerkit comes with the following marketing pages:

- Home Page
- Contact Form
- Pricing Page
- FAQ
- Contact Page (with a contact form)

## Adding a new marketing page

To add a new marketing page to your Makerkit application, you need to follow these steps.

Create a file in the `apps/web/app/routes/marketing` directory:

```tsx
// apps/web/app/routes/marketing/about.tsx
export default function AboutPage() {
  return <div></div>
}
```

Then add the route to the `apps/web/app/routes.ts` file:

```tsx {% title="apps/web/app/routes.ts" %}
const marketingLayout = layout('routes/marketing/layout.tsx', [
  index('routes/marketing/index.tsx'),
  route('terms-of-service', 'routes/marketing/terms-of-service.tsx'),
  route('privacy-policy', 'routes/marketing/privacy-policy.tsx'),
  route('pricing', 'routes/marketing/pricing.tsx'),
  route('contact', 'routes/marketing/contact/index.tsx'),
  route('faq', 'routes/marketing/faq.tsx'),
  route('blog', 'routes/marketing/blog/index.tsx'),
  route('blog/:slug', 'routes/marketing/blog/$slug.tsx'),
  route('cookie-policy', 'routes/marketing/cookie-policy.tsx'),
  route('about', 'routes/marketing/about.tsx'),  // <-- add this line
  layout('routes/marketing/docs/layout.tsx', [
    route('docs', 'routes/marketing/docs/index.tsx'),
    route('docs/*', 'routes/marketing/docs/$slug.tsx'),
  ]),
]);
```

This page inherits the layout at `apps/web/app/routes/marketing/layout.tsx`. You can customize the layout by editing this file - but remember that it will affect all marketing pages.

## Contact Form

To make the contact form work, you need to add the following environment variables:

```bash
CONTACT_EMAIL=
```

In this variable, you need to set the email address where you want to receive the contact form submissions. The sender will be the same as the one configured in your [mailing configuration](email-configuration).

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\migrations.mdoc

---
status: "published"
label: "Migrations"
order: 1
title: "How to create new migrations and update the database schema in your React Router Supabase application"
description: "Learn how to create new migrations and update the database schema in your React Router Supabase application"
---


Creating a schema for your data is one of the primary tasks when building a new application. In this guide, we'll walk through how to create new migrations and update the database schema in your React Router Supabase application.

## A quick word on migrations

Supabase's hosted Studio is pretty great - but I don't think it should be used to perform schema changes. Instead, I recommend using your local Supabase Studio to make the changes and then generate the migration file. Then, you can push the migration to the remote Supabase instance.

At this point, you have two options:

1. create a migration with `pnpm --filter web supabase migration new <name>` and update the code manually
2. or, use the local Supabase Studio to make the changes and then run `pnpm --filter web supabase db diff -f <name>` which will generate the migration file for you. DOUBLY CHECK THE FILE!

Once you've tested it all and are happy with your local changes, push the migration to the remote Supabase instance with `pnpm --filter web supabase db push`.

Doing the opposite is also okay - but:

1. You're making changes against the production database - which is risky
2. You're not testing the changes locally - which is risky
3. You need to pull the changes from the remote Supabase instance to your local instance so they are in sync

## Creating a Migration

The first step towards building your app schema is to create a new migration.

To do so, run the command:

```
pnpm --filter web supabase migration new <name>
```

The migration will be generated at `apps/web/supabase/migrations`.

Once added some SQL commands, you need to reset the schema for it to take effect:

```
pnpm run supabase:web:reset
```

The schema is now populated! Yay!

### Generating Supabase types

Now that your schema is populated, you need to generate the types so that your Supabase client can work correctly.

Please run the following command:

```
pnpm run supabase:web:typegen
```

Your Supabase client will now correctly infer the types with your schema changes.

-----------------
FILE PATH: kits\react-router-supabase-turbo\development\seo.mdoc

---
status: "published"

label: "SEO"
title: SEO - Improve your React Router application's search engine optimization"
description:  "Learn how to improve your Makerkit application's search engine optimization (SEO)"
order: 10
---


SEO is an important part of building a website. It helps search engines understand your website and rank it higher in search results. In this guide, you'll learn how to improve your Makerkit application's search engine optimization (SEO).

Makerkit is already optimized for SEO out of the box. However, there are a few things you can do to improve your application's SEO:

1. **Content**: The most important thing you can do to improve your application's SEO is to **create high-quality content**. No amount of technical optimization can replace good content. Make sure your content is relevant, useful, and engaging - and make sure it's updated regularly.
2. **Write content helpful to your customers**: To write good content, the kit comes with a blog and documentation feature. You can use these features to create high-quality content that will help your website rank higher in search results - and help your customers find what they're looking for.
3. **Use the correct keywords**: Use descriptive titles and meta descriptions for your pages. Titles and meta descriptions are the first things users see in search results, so make sure they are descriptive and relevant to the content on the page. Use keywords that your customers are likely to search for.
4. **Optimize images**: Use descriptive filenames and alt text for your images. This helps search engines understand what the image is about and can improve your website's ranking in image search results.
5. **Website Speed**: This is much less important than it used to be, but it's still a factor. Make sure your website loads quickly and is mobile-friendly. You can use tools like Google's PageSpeed Insights to check your website's speed and get suggestions for improvement. Optimize all images and assets to reduce load times.
6. **Mobile Optimization**: Make sure your website is mobile-friendly. Google ranks mobile-friendly websites higher in search results. You can use Google's Mobile-Friendly Test to check if your website is mobile-friendly.
7. **Sitemap and Robots.txt**: Makerkit generates a sitemap and robots.txt file for your website. These files help search engines understand your website's structure and what pages they should index. Make sure to update the sitemap as you add new pages to your website.
8. **Backlinks**: Backlinks are links from other websites to your website. It's touted to be **the single most important factor in SEO** these days. The more backlinks you have from high-quality websites, the higher your website will rank in search results. You can get backlinks by creating high-quality content that other websites want to link to.

### Adding pages to the sitemap

Generally speaking, Google will find your pages without a sitemap as it follows the link in your website. However, you can add pages to the sitemap by adding them to the `apps/web/app/server-sitemap.xml/route.ts` route, which is used to generate the sitemap.

If you add more static pages to your website, you can add them to the `getPaths` function in the `apps/web/app/routes/sitemap/route.tsx` file. For example, if you add a new page at `/about`, you can add it to the `getPaths` function like this:

 ```tsx {% title="apps/web/app/routes/sitemap/route.tsx" %}
function getPaths() {
  const paths = [
    '/',
    '/faq',
    '/blog',
    '/docs',
    '/pricing',
    '/contact',
    '/cookie-policy',
    '/terms-of-service',
    '/privacy-policy',
    // add more paths here,
    '/about', // <-- add the new page here
  ];

  return paths.map((path) => {
    return {
      loc: new URL(path, appConfig.url).href,
      lastmod: new Date().toISOString(),
    };
  });
}
```

All the blog and documentation pages are automatically added to the sitemap. You don't need to add them manually.

### Adding your website to Google Search Console

Once you've optimized your website for SEO, you can add it to Google Search Console. Google Search Console is a free tool that helps you monitor and maintain your website's presence in Google search results.

You can use it to check your website's indexing status, submit sitemaps, and get insights into how Google sees your website.

The first thing you need to do is verify your website in Google Search Console. You can do this by adding a meta tag to your website's HTML or by uploading an HTML file to your website.

Once you've verified your website, you can submit your sitemap to Google Search Console. This will help Google find and index your website's pages faster.

Please submit your sitemap to Google Search Console by going to the `Sitemaps` section and adding the URL of your sitemap. The URL of your sitemap is `https://your-website.com/server-sitemap.xml`.

Of course, please replace `your-website.com` with your actual website URL.

### Backlinks

Backlinks are said to be the most important factor in modern SEO. The more backlinks you have from high-quality websites, the higher your website will rank in search results - and the more traffic you'll get.

How do you get backlinks? The best way to get backlinks is to create high-quality content that other websites want to link to. However, you can also get backlinks by:

1. **Guest posting**: Write guest posts for other websites in your niche. This is a great way to get backlinks and reach a new audience.
2. **Link building**: Reach out to other websites and ask them to link to your website. You can offer to write a guest post or provide a testimonial in exchange for a backlink.
3. **Social media**: Share your content on social media to reach a wider audience and get more backlinks.
4. **Directories**: Submit your website to online directories to get backlinks. Make sure to choose high-quality directories that are relevant to your niche.

All these methods can help you get more backlinks and improve your website's SEO - and help you rank higher in search results.

### Content

When it comes to internal factors, content is king. Make sure your content is relevant, useful, and engaging. Make sure it's updated regularly and optimized for SEO.

What should you write about? Most importantly, you want to think about how your customers will search for the problem your SaaS is solving. For example, if you're building a project management tool, you might want to write about project management best practices, how to manage a remote team, or how to use your tool to improve productivity.

You can use the blog and documentation features in Makerkit to create high-quality content that will help your website rank higher in search results - and help your customers find what they're looking for.

### Indexing and ranking take time

New websites can take a while to get indexed by search engines. It can take anywhere from a few days to a few weeks (in some cases, even months!) for your website to show up in search results. Be patient and keep updating your content and optimizing your website for search engines.

---

By following these tips, you can improve your Makerkit application's search engine optimization (SEO) and help your website rank higher in search results. Remember, SEO is an ongoing process, so make sure to keep updating your content and optimizing your website for search engines. Good luck!


-----------------
FILE PATH: kits\react-router-supabase-turbo\development.mdoc

---
status: "published"
title: "Development in React Router Supabase Turbo"
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
FILE PATH: kits\react-router-supabase-turbo\emails\authentication-emails.mdoc

---
status: "published"

title: "Setting the Supabase Auth Email Templates in Production"
label: "Authentication Emails"
description: "Configure the Supabase authentication URLs and emails in the React Router Supabase Starter Kit."
order: 3
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
FILE PATH: kits\react-router-supabase-turbo\emails\email-configuration.mdoc

---
status: "published"
label: "Email Configuration"
description: "Learn how to configure the mailer provider to start sending emails from your React Router Supabase Starter Kit."
title: "Email Configuration in the React Router Supabase Starter Kit"
order: 0
---

## Makerkit Emails vs Supabase Auth Emails

Before we dive into the configuration, it's important to understand the difference between Makerkit emails and Supabase Auth emails.

1. **Makerkit Emails**: Makerkit emails are used to send transactional emails, such as inviting a member to join a team, and a delete account confirmation, plus all the ones you will be adding
2. **Supabase Auth Emails**: Supabase Auth emails are used to send emails for authentication purposes, such as email verification and password reset.

You will need to set up both Makerkit and Supabase Auth emails to have a complete email setup in your application.

The guide below refers to Makerkit emails.

For Supabase Auth, please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/auth-smtp) for more information.

---

Makerkit allows you to configure and send emails using the `@kit/mailers` package. The package provides a simple API to send emails.

Makerkit provides three implementations of the mailer:

1. `nodemailer`: The default mailer that uses the `nodemailer` library. Perfect for Node.js environments as it works with any SMTP server - so you're not locked into any specific provider.
2. `resend`: A mailer that uses the [Resend](https://resend.com) API using HTTP. It's a good choice if you choose to use Resend.

Below, we will show you how to configure the mailer provider to start sending emails from your React Router Supabase Starter Kit.

## Configuration

To set the mailer provider, you need to set the `MAILER_PROVIDER` environment variable in the `apps/web/.env` file. For example, to use the `nodemailer` mailer, set the `MAILER_PROVIDER` environment variable to `nodemailer`:

```bash
MAILER_PROVIDER=nodemailer
```

By default, we use `nodemailer`.

### SMTP Configuration

If you want to use the `nodemailer` mailer, you need to set the SMTP configuration in your environment variables. Here is an example of the SMTP configuration:

```bash
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_HOST=
EMAIL_PORT=
EMAIL_TLS=
```

The variables are:

1. `EMAIL_USER`: The user of the email address. This is provider-specific so please check your email provider's documentation.
2. `EMAIL_PASSWORD`: The password of the account provided by the email provider.
3. `EMAIL_HOST`: The SMTP server host. This is provider-specific so please check your email provider's documentation.
4. `EMAIL_PORT`: The SMTP server port. This is provider-specific so please check your email provider's documentation.
5. `EMAIL_TLS`: The TLS configuration. This is provider-specific so please check your email provider's documentation. Generally, you can set it to `true`.
6. `EMAIL_SENDER`: The sender of the emails. This is usually the email address of your application. It's usually in the format `Sender Name <email@app.com>`.

## Resend API

Alternatively, you can use Resend.

Set the `MAILER_PROVIDER` environment variable to `resend` in the `apps/web/.env` file:

```bash
MAILER_PROVIDER=resend
```

And provide the Resend API key:

```bash
RESEND_API_KEY=your-api-key
EMAIL_SENDER=your-email
```

That's it! You're now ready to send emails from your React Router Supabase Starter Kit using the configured mailer provider.

-----------------
FILE PATH: kits\react-router-supabase-turbo\emails\email-templates.mdoc

---
status: "published"

label: "Email Templates"
description: "Learn how to write email templates in the React Router Supabase Starter Kit with React.Email"
title: "Email Templates in the React Router Supabase Starter Kit"
order: 2
---


Email templates are a great way to send beautiful and consistent emails to your users. In the React Router Supabase Starter Kit, we use [React.Email](https://react.email) to create email templates.

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
FILE PATH: kits\react-router-supabase-turbo\emails\inbucket.mdoc

---
status: "published"
title: "InBucket for local development in the React Router Supabase Starter Kit"
label: "Local Development"
description: "Learn how to use InBucket for local development in the React Router Supabase Starter Kit."
order: 4
---

When developing locally, Supabase uses [InBucket](https://www.inbucket.org/) to capture emails sent during development. This allows you to test email functionality without actually sending emails to real addresses.

Makerkit also uses InBucket for sending non-authentication emails, such as team invitations and account deletion confirmations.

## What is InBucket?

InBucket is a disposable email service that captures all emails sent during local development. It's particularly useful for testing:

- Email verification flows
- Password reset emails
- Team invitations
- Notification emails
- Any other transactional emails

## Accessing InBucket

When running Supabase locally, InBucket is automatically available at:

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
2. Go to InBucket at [http://localhost:54324](http://localhost:54324)
3. Find the verification email
4. Click the verification link or copy the verification code

### 2. Password Reset

To test password reset flows:
1. Request a password reset using any email
2. Check InBucket for the reset email
3. Use the reset link or code to complete the process

### 3. Team Invitations

When testing team invites:
1. Send an invitation to any email address
2. Check InBucket for the invitation email
3. Use the invitation link to accept

## InBucket vs Production Emails

Remember that InBucket is for development only. In production:

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

NB: Supabase authentication emails will still be sent using InBucket.

## Conclusion

InBucket is a valuable tool for local development, allowing you to:
- Test email flows without sending real emails
- Iterate quickly on email templates
- Debug email-related issues
- Develop with confidence

Remember to switch to a proper email provider when deploying to production!

-----------------
FILE PATH: kits\react-router-supabase-turbo\emails\sending-emails.mdoc

---
status: "published"
label: "Sending Emails"
description: "Learn how to send emails in the React Router Supabase Starter Kit."
title: "Sending Emails in the React Router Supabase Starter Kit"
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

Et voilà! You are now ready to send emails from your React Router Supabase Starter Kit. 🚀

