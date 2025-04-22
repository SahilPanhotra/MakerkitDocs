-----------------
FILE PATH: kits\next-fire\development\commands.mdoc

---
status: "published"
title: "All the commands to use for your Makerkit app | Next.js Firebase"
description: "Use these commands to run the development server, build the application, and more in your Next.js Firebase application"
label: Commands
order: 0
---

Here are all the commands defined in the MakerKit's template:

### Analyzing the Next.js bundle

Run the command:

```
npm run analyze
```

The command will automatically bundle your clients and open a page with an analysis of your bundles. This will allow you to understand which libraries are taking the most space.

### Run the development server

Run the command:

```
npm run dev
```

### Run the E2E testing server

This command is needed for running the E2E tests. Run the command:

```
npm run dev:test
```

### Build a production bundle

Run the command:

```
npm build
```


### Start a production server

Run the command after building the application with the `build` command:

```
npm start
```

### Build RSS feeds

Run the command:

```
npm run rss
```

This is optional as it is automatically called after the `build` command.

### Build Sitemap

Run the command:

```
npm run sitemap
```

This is optional as it is automatically called after the `build` command.

### Format all the files

Run the command:

```
npm run format
```

### Healthcheck: Lint code and check types

Run the command:

```
npm run healthcheck
```

### Start the Firebase Emulator

Run the command:

```
npm run firebase:emulators:start
```

This is needed during development.

### Export data from the Firebase Emulator

Run the command:

```
npm run firebase:emulators:export
```

### Run Cypress for E2E Tests (with UI)

Run the command:

```
npm run cypress
```

### Run Cypress for E2E Tests (Headless)

Run the command:

```
npm run cypress:headless
```

### Run Tests and Exit

Run the command:

```
npm test
```

### Run the Local Stripe Webhooks Server

This is needed if you are testing Stripe. This command requires Docker, but you can alternatively install Stripe on your OS and change the command to use `stripe` directly.

Run the command:

```
npm run stripe:listen
```

### Run the Mock Stripe Server

Run the command:

```
npm run stripe:mock-server
```

### Index blog and documentation pages for making the documents available for searching

Run the command:

```
npm run blog-docs-indexer
```

### Kill Ports

The following commands kills all the ports that need to be free to run the Makerkit stack. This can be necessary after running the tests, for example when the emulators don't free up the ports after shutting down.

Run the command:

```
npm run killports
```


-----------------
FILE PATH: kits\next-fire\development\cors.mdoc

---
status: "published"
title: "Enable CORS | Next.js Firebase SaaS Kit"
label: Enable CORS
order: 9
---

Enabling CORS is required if you want to allow serving HTTP request to
external clients. For example, if you want to expose an API to some
consumers: JS libraries, headless clients, and so on.

To enable CORS, you can use the `with-cors` middleware in two different ways.

You can simply call it in your handler:

```tsx
import withCors from '~/core/middleware/with-cors';

function apiHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
    withCors(res);

    // your logic
}

export default apiHandler;
```

Alternatively, you can use it with `withPipe`:

```tsx
import withCors from '~/core/middleware/with-cors';

function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
   // your logic
}

export default withPipe(
  withCors,
  apiHandler
);
```

Additionally, you need to respond to `OPTIONS` request appropriately.

```tsx
export function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  withCors(res);

  if (req.method === `OPTIONS`) {
     // add the method you want to allow
     res.setHeader('Access-Control-Allow-Methods', 'GET');

     return res.end();
  }
}
```

-----------------
FILE PATH: kits\next-fire\development\develop-firebase-security-rules.mdoc

---
status: "published"

title: "Setting up your Firebase Security Rules during development"
label: "Security Rules"
order: 4
---

When running the Firebase Emulator, you can update your Firebase Firestore and Storage security rules by simply changing the files `firestore.rules` and `storage.rules`.

### Firestore Security Rules

By default, Makerkit comes with pre-configured Security Rules that work with the original boilerplate's data structure. However, you will likely be updating your structure to fit your SaaS data model, and therefore you will probably be updating the Firestore Security Rules.

To change your Firestore security rules, simply edit the `firestore.rules` file in the root directory of your Makerkit repository; the Firebase Emulator will automatically pick the new changes up.

### Storage Security Rules

At the moment, Makerkit does not add any security rule for your Storage buckets; this will be added very soon.

### Publish your rules

Publishing your security rules is a critical pre-production step you should never forget; otherwise, your users will encounter runtime exceptions, ultimately leading to bugs in your app.

{% alert type="warning" title="Remember: updating your repository's rules does not automatically publish them!" %}
Remember: updating your repository's rules does not automatically publish them!
{% /alert %}

To publish your Security rules, open your Firebase Admin Console and copy-paste the content of your security rules. It can take some time before they fully propagate.


-----------------
FILE PATH: kits\next-fire\development\emails.mdoc

---
status: "published"
title: "Sending emails | Next.js Firebase"
label: Sending Emails
order: 6
---

MakerKit uses the popular package `nodemailer` to allow you to send emails.

Normally, **you would use a service for sending transactional emails** such as SendGrid, Mailjet, MailGun, Postmark, and so on.

These are all valid, and it doesn't really matter what you use. In
fact, as long as you provide the SMTP credentials for your service, you shouldn't be needing any other change.

## Email Configuration

To add your service's configuration, fill the environment variables in your production environment.

This is best done from your Hosting provider's safe environment variables, or from your CI/CD pipeline.

```tsx
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

The details above are provided by the service you're using.

**Note**: Some email providers don't like the `EMAIL_SENDER` format. If you're having issues, try using just the email address, such as `info@makerkit.dev`.

{% alert type="warning" title="Makerkit uses InBucket for sending emails, and not your production service" %}
When running the emulators, by default, Makerkit uses InBucket for sending emails, and not your production service.
  Read below for more info.
{% /alert %}

## Sending Emails

To send emails, import and use the `sendEmail` function, such as below:

```tsx
interface SendEmailParams {
  from: string;
  to: string;
  subject: string;
  text?: string;
  html?: string;
}

import { sendEmail } from '~/core/email/send-email';

function sendTransactionalEmail() {
  const sender = configuration.email.senderAddress;

  return sendEmail({
    to: `youruser@email.com`,
    from: sender,
    subject: `Achievement Unlocked!`,
    html: `Yay, you unlocked an achievement!`,
  });
}
```

## Using react.email to render emails

Makerkit's newest versions use [react-email](https://react.email) to render emails: this is a great library that allows us to write emails using React components.

By default, Makerkit's only email is the one sent when inviting members to a Makerkit application - but you can leverage this library to write your own emails for your application.

For example, here's the code for the email sent when inviting members:

 ```tsx {% title="src/lib/emails/invite.ts" %}
interface Props {
  organizationName: string;
  organizationLogo?: string;
  inviter: Maybe<string>;
  invitedUserEmail: string;
  link: string;
  productName: string;
}

export default function renderInviteEmail(props: Props) {
  const title = `You have been invited to join ${props.organizationName}`;

  return render(
    <Html>
      <Head>
        <title>{title}</title>
      </Head>
      <Preview>{title}</Preview>
      <Body style={{ width: '500px', margin: '0 auto', font: 'helvetica' }}>
        <EmailNavbar />
        <Section style={{ width: '100%' }}>
          <Column>
            <Text>Hi,</Text>

            <Text>
              {props.inviter} with {props.organizationName} has invited you to
              use {props.productName} to collaborate with them.
            </Text>

            <Text>
              Use the button below to set up your account and get started:
            </Text>
          </Column>
        </Section>

        <Section>
          <Column align="center">
            <CallToActionButton href={props.link}>
              Join {props.organizationName}
            </CallToActionButton>
          </Column>
        </Section>

        <Section>
          <Column>
            <Text>Welcome aboard,</Text>
            <Text>The {props.productName} Team</Text>
          </Column>
        </Section>
      </Body>
    </Html>
  );
}
```

## Testing Emails locally

Makerkit uses [InBucket](https://inbucket.org) - a platform to testing emails.

InBucket saves time during development since we can test
our emails without setting up a real SMTP service - and works locally and
offline.

To run the InBucket platform, **we need Docker running on our machine**.

To start the InBucket service, run the following command:

```
npm run inbucket:start
```

InBucket will now be running at [localhost:9000](http://localhost:9000). When we send an email using the
`sendEmail` function in the kit, we can visualize it using the InBucket UI.

InBucket is used by default during development. Instead, for production
usage, you will need to set up a real SMTP service.

#### Ethereal

In previous versions Makerkit used Ethereal. If you are running an older version, please refer to the below.

Makerkit used Ethereal to allow you testing your emails without using a real
email account. To use Ethereal, you should add the following environment variables to your project, using a secure environment:

```bash
ETHEREAL_EMAIL=
ETHEREAL_PASSWORD=
```

If not set, Makerkit will automatically create an account for you on-the-fly and print the credentials in the console. You can then use these credentials to log in to Ethereal and see your emails.


-----------------
FILE PATH: kits\next-fire\development\encrypting-secrets.mdoc

---
status: "published"
title: "Encrypting Secrets | Next.js Firebase SaaS Kit"
label: Encrypting Secrets
order: 9
description: "When storing sensitive data, such as API keys, you must ensure to encrypt your data in Firestore. Here is how we can encrypt and decrypt it."
---

When storing sensitive data, such as API keys, you must ensure to encrypt your data in Firestore.

To use these utilities, you must provide an environment variable named `SECRET_KEY`. This should be provided safely using your CI since it's a secret key.

MakerKit provides some utilities to encrypt and decrypt your data. 

```tsx
import { encrypt, decrypt } from '~/core/generic/crypto';

// storing secrets
function storeApiKey(key: string) {
  const encryptedKey = encrypt(key);

  return storeKeyInFirestore(encryptedKey);
}

// retrieving secrets
function getApiKey(id: string) {
  const encryptedKey = await getApiKeyFromFirestore(id);

  return decrypt(encryptedKey);
}
```


-----------------
FILE PATH: kits\next-fire\development\i18n-translations.mdoc

---
status: "published"

title: "Adding and updating locales in your Makerkit application"
label: Translations and Locales
order: 5
---

Much of the MakerKit's codebase uses translations using the package `next-i18next':

- this package allows you to create multi-language apps effortlessly
- it also helps easily renaming the Makerkit's entities according to your preferences: for example, renaming "organizations" to another entity such as "team" or "workspace"
- in a way, it helps write cleaner HTML

To add or update the default strings, you must use the locale files at `public/locales/{lang}.json`. For example, all the locale files for the English language are placed under `public/locales/en`.

{% alert type="warning" title="Restart the MakerKit server to see the changes" %}
When adding new keys, you'll need to restart the MakerKit server (`npm run dev`) to see the changes. Also, you may need to clear the browser cache since the client is hydrated with the translation files fetched from the JSON files.
{% /alert %}

The locales are split by functionality, page, or entity: as your application grows, you may not want to load every JSON file to reduce your bundle size.

For example, the translations for the authentication are placed in `auth.json,` while the ones used across every page are placed under `common.json.`

If you are unsure where to add some translations, simply add them to `common.json`.

{% alert type="info" title="All the JSON files are used by default" %}
To simplify things, the MakerKit's starter uses all the available JSON
  files, and we recommend doing so until you have too many files or too many
  keys.
{% /alert %}

### Adding new languages

By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:
1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **Next.js i18n config**: We need to also add a new language to the Next.js configuration at `next-i18next.config.js`. Simply add your new language's code to the `locales` array.

The configuration will look like the below:

```js
const config = {
  i18n: {
    defaultLocale: DEFAULT_LOCALE,
    locales: [DEFAULT_LOCALE, 'es'],
  },
  fallbackLng: {
    default: [DEFAULT_LOCALE],
  },
  localePath: resolve('./public/locales'),
};
```

### Adding new translation files

For example, let's assume you add a new namespace (e.g. a JSON file) named `editor.json`. Now, to automatically add the translations of this file, we will add it to the `DEFAULT_OPTIONS` variable:

```tsx
const DEFAULT_OPTIONS: Options = {
  locale: DEFAULT_LOCALE,
  localeNamespaces: [
    'common',
    'auth',
    'organization',
    'profile',
    'subscription',
    'editor' // <---
status: "published"
 HERE IT IS
  ],
};
```

If you want to simplify things, simply merge all the files into one; the only downside is that lazy-loading translation files will not be possible.

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```

### Using the Language Switcher component

The Language switcher component will automatically retrieve and translate the languages listed in your configuration.

When the user changes language, it will update the URL to reflect the user's preferences and rerender the application with the selected language.

{% video src="/assets/images/posts/language-selector.mp4" /%}

NB: The language selector is not added to the application by default; you will need to import it where you want to place it.

### Lazy Loading Locale files

Let's assume your `editor.json` file is only loaded in your `EditorPage` component; so it can make sense to only load translations on that page. To do so, we can use `withAppProps`:

 ```tsx {% title="pages/editor.tsx" %}
export default function EditorPage() {
  return <></>;
}

export function getServerSideProps() {
  return withAppProps({
    localeNamespaces: ['editor']
  });
}
```

The `editor` namespace will be added to the default ones specified in  `DEFAULT_OPTIONS`. This approach is excellent if the translations in the specified JSON file are only added to a specific page.


-----------------
FILE PATH: kits\next-fire\development\logging.mdoc

---
status: "published"
title: "Logging | Next.js Firebase SaaS Kit"
label: Logging
order: 8
description: "Logging is a fundamental part for your SaaS to understand and
monitor the behavior of your code at runtime. Let's learn how to add logging
to your Makerkit application."
---

The Makerkit Boilerplate uses `pino` as logging library. Pino is simple and
lightweight, and it's used across the API functions to log important
information that helps you debug your code.

Generally speaking, you will find that we log every function, especially for
asynchronous operations that could fail for a number of reasons: network
issues, API exceptions, and so on.

So, how to log your API functions effectively? Our recommendation is to log
**before executing an operation** and then log the **result of the
operation**, without leaking important data.

Furthermore, we want to log information such as:
- which user is performing the operation?
- which organization is the user part of?
- any other information that can help you understand and debug what is
happening.

Let's assume you're writing an API function to write to your Firestore database:

```tsx
function addIntegrationHandler() {
  return writeToFirestore(data);
}
```

Let's rewrite the above by adding logging to your function:

```tsx
import logger from '~/core/logger';

async function addIntegrationHandler(
  req: NextApiRequest,
  res: NextApiResponse,
) {
  const userId = req.firebaseUser.uid;
  const organizationId = req.cookies.organizationId;
  const integrationId = req.body.integration;

  // this is the context that every log will print out
  const loggingContext = {
    integrationId,
    organizationId,
    userId,
  };

  // Here we log what we're doing
  logger.log(loggingContext, `Adding new integration to organization`);

  try {
    await writeToFirestore(data);

    // Here we log that the result of the operation
    // was successful
    logger.log(loggingContext, `Integration successfully added`);

    // return successful response
    return res.json({
      integrationId,
      success: true
    });
  } catch (e) {
    // Here we log that the operation failed
    logger.error(loggingContext, `Encountered an error while adding integration`);

    // Logging errors can be okay but
    // ensure not to leak important information!
    logger.debug(e);

    // return 500
    return res.status(500).json({
      integrationId,
      success: false
    });
  }
}
```

If you use the filter `withExceptionFilter`, it will automatically
log eventual exceptions thrown by the function, which makes the try/catch
block optional.

NB: If you're using Vercel, logs are not persisted by default! [Check out the
Vercel integrations](https://vercel.com/integrations#logging) for adding persisting logs to your projects.


-----------------
FILE PATH: kits\next-fire\development\user-roles.mdoc

---
status: "published"
title: "Extend User Roles | Next.js Firenase SaaS Starter Kit"
description: "Learn how to extend the User Roles in the Next.js Firenase SaaS Starter Kit"
label: User Roles
order: 9
---

The MakerKit Starter has three default roles:

```tsx
export enum MembershipRole {
  Member,
  Admin,
  Owner,
}
```

We use an `enum`, but you can convert this to an object if you need more granular permissions.

The permissions are hierarchical, which means that if we had a role with a lower level (`Readonly`), we would add it before `Member`:

```tsx
export enum MembershipRole {
  Readonly,
  Member,
  Admin,
  Owner,
}
```

When writing permissions between users, we can check if the user performing the action has a greater role than the target user.

You can extend the role above easily by adding your own, for example:

```tsx
export enum MembershipRole {
  Readonly,
  AccountManager,
  Owner,
}
```

Afterward, remember to add the name and descriptions of these roles in the translations file `common.json`:

```json
"roles": {
  "owner": {
    "label": "Owner",
    "description": "Can change any setting, invite new members and manage billing"
  },
  "accountmanager": {
    "label": "Account Manager",
    "description": "Can change some settings, invite members, perform disruptive actions"
  },
  "readonly": {
    "label": "Readonly",
    "description": "Can only read information"
  }
}
```

Learn more about [using user roles in your permissions system](/docs/permissions).


-----------------
FILE PATH: kits\next-fire\development\validating-api-input-zod.mdoc

---
status: "published"

title: "Validating the API inputs with Zod and Typescript"
label: Validating API payload with Zod
order: 7
description: "Zod is a library for validating data with awesome support for
Typescript. Learn how to use it within your Makerkit project."
---

[Zod](https://github.com/colinhacks/zod) is a Typescript library that helps us
secure our API endpoints by validating the payloads sent
from the client and also facilitating the typing of the payloads with
Typescript.

Using Zod is the first line of defense to validate the data sent against our
API: as a result, it's something we recommend you keep doing. It ensures we
write safe, resilient, and valid code.

All Makerkit's API routes are secured with Zod: in this document, we want
to explain the conventions used by the SaaS Boilerplate, and how to use it
for your API endpoints.

When we write an API endpoint, we first define the schema of the payload:

```tsx
function getBodySchema() {
  return z.object({
    displayName: z.string(),
    email: z.string().email(),
  });
}
```

This function represents the schema, which will validate the following
interface:

```tsx
interface Body {
  displayName: string;
  email: Email;
}
```

Now, let's write the body of the API handler that validates the body of the
function, which we expect to be equal to the `Body` interface.

```tsx
import { throwBadRequestException } from `~/core/http-exceptions`;

function inviteMemberHandler(
  req: NextApiRequest,
  res: NextApiResponse,
) {
  try {
     // we can safely use data with the interface Body
    const schema = getBodySchema();
    const { displayName, email } = schema.parse(req.body);

    return sendInvite({ displayName, email });
  } catch(e) {
    return throwBadRequestException(res);
  }
}

export default function apiHandler() {
  const handler = withPipe(
    withMethodsGuard(['POST']),
    withAuthedUser,
    inviteMemberHandler,
  );

  // manage exceptions
  return withExceptionFilter(req, res)(handler);
}
```

You can also use `safeParse` if you prefer not to throw an error when the
validation fails:

```tsx
function inviteMemberHandler(
  req: NextApiRequest,
  res: NextApiResponse,
) {
  const schema = getBodySchema();
  const result = schema.safeParse(req.body);

  // we use result.success as a type guard
  // when false, we throw an exception
  if (!result.success) {
    return throwBadRequestException(res);
  }

  // TS correctly infers result.data now
  return sendInvite(result.data);
}
```

To learn more about validating data with Zod, we suggest you
check out [the Zod official documentation on GitHub](https://github.com/colinhacks/zod).


-----------------
FILE PATH: kits\next-fire\development.mdoc

---
status: "published"
title: "Common development tasks in the Next.js Firebase kit"
label: Development
order: 6
collapsible: true
collapsed: true
---

In this section, you will learn everything you need to know about common development tasks in the Next.js Firebase kit.

- [Code Style](development/code-style): this is the guide to set up the code style
- [API Routes](development/api-routes): this is the guide to configure API routes
- [Building Features](development/building-features): this is the guide to build features
- [Commands](development/commands): this is the guide to use the commands
- [Emails](development/emails): this is the guide to set up emails
- [Encrypting Secrets](development/encrypting-secrets): this is the guide to encrypt secrets
- [Validating API input with Zod](development/validating-api-input-zod): this is the guide to validate API input with Zod
- [CORS](development/cors): this is the guide to set up CORS
- [Logging](development/logging): this is the guide to set up logging
- [I18n](development/i18n-translations): this is the guide to set up i18n and translations

-----------------
FILE PATH: kits\next-fire\getting-started\clone-repository.mdoc

---
status: "published"
title: Clone the Next.js Firebase SaaS boilerplate repository
label: Clone the repository
order: 2
description: 'Learn how to clone the Next.js Firebase repository and getting started'
---

If you have bought a license for MakerKit, you have access to all the
repositories built by the MakerKit team. In this document, we will learn how
to fetch and install the codebase.

### Requirements

To get started with the Next.js and Firebase SaaS template, we need to ensure
you install the required software.

- Node.js (LTS recommended)
- Git
- Docker (optional but highly recommended)

### Cloning the repository

To get the codebase on your local machine, clone the repository with the
following command:

```
git clone git@github.com:makerkit/next-firebase-saas-kit.git
```

The command above clones the repository in the folder `my-saas` which
you can rename it with the name of your project.

### Initializing Git

Now, run the following commands for:

1. Moving into the folder
2. Adding the remote to your own Git repository (if you haven't created a Git repository yet, you can do it later on)
3. Adding the original Makerkit repository as "upstream" so we can fetch updates from the main repository:

```
cd my-saas
git remote rm origin
git remote add origin <your-git-repository>
git remote add upstream git@github.com:makerkit/next-firebase-saas-kit.git
```

In this way, to fetch updates (after committing your files), simply run:

```
git pull upstream main
```

Sometimes, you'll likely run into conflicts when running this command, so carefully choose the changes (sorry!).

If you want to use the Lemon Squeezy branch, you'll need to switch to the `main-ls` branch:

```
git checkout main-ls
```

Of course, when pulling updates, you'll need to pull from the `main-ls` branch:

```
git pull upstream main-ls
```

### Installing the Node dependencies

Finally, we can install the NodeJS dependencies with `npm`:

```
npm i
```

While the application code is fully working, we now need to set up your Firebase
project.

So let's jump on to the next step!


-----------------
FILE PATH: kits\next-fire\getting-started\coding-conventions.mdoc

---
status: "published"

title: SaaS template Coding Conventions
label: Coding Conventions
order: 5
description: 'An explanation of the coding conventions used in the boilerplate'
---

## React.FC vs React.FCC

Since React 18, `React.FC` no longer implies that components can accept a `children` prop. The alternative to that is using the interface `React.FC<React.PropsWithChildren<{}>>`, which is a bit long.

`React.FCC` is a shorthand that includes `children` as a possible property that works similarly to how it used to work. When a component needs `children`, `React.FCC` is preferred since it's faster to type.

## Exporting functions vs constants

We use both conventions in the boilerplate, i.e., exporting functions as components and constants.

My general rule is the following:

- if it's the exported component, use `const` typed with `React.FC` or `React.FCC`
- if it's not exported, use `function` since my personal preference is to define functions below the exported component

NB: sometimes, that may not be the case, and that's not necessarily intentional. The two constructs are essentially the same in most cases.

## Why is useCallback used in most components?

Most functions defined within a render function use `useCallback`. In my experience, this is the better default to avoid re-renders that I can't immediately explain.

Whether you want to adhere to this convention is totally up to you; just be aware of the potential issues.

## Interfaces, Types, and Inline 

I've gone through phases, so you may find some types defined as `type` and others as `interface`. I've since realized that I prefer `interface`, but you will find some instance that uses `type`.

Inline types are typically only defined when the type isn't reused anywhere. For example, when defining a function that accepts parameters as objects.

## Function parameters

I adhere to the principle that if a function accepts multiple parameters of the same type, then use an object. Unfortunately, Typescript can't save us if the types match.

Consider the example below:

```tsx
function makeDiv(width: number, height: number) {}

const height = 10;
const width = 40;

// correct according to TS, not in practice!
const div = makeDiv(height, width);
```

Now we rewrite it using an object:

```tsx
function makeDiv(params: { width: number, height: number}) {}

const height = 10;
const width = 40;

// Okay, now it looks better!
const div = makeDiv({ height, width });
```

## Custom React Hooks

The boilerplate uses Custom React Hooks when:

- fetching/writing using an API function
- fetching/writing using Firestore
- fetching/writing using Storage

It makes the code more understandable and reusable.

