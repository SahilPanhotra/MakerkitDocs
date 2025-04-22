-----------------
FILE PATH: kits\remix-supabase\data-fetching\uploading-data-to-storage.mdoc

---
status: "published"
title: "Uploading data to Storage | Remix Supabase SaaS Kit"
description: "Learn how to upload your data to Supabase Storage in your Remix application"
label: "Uploading data to Storage"
order: 4
---

To upload data to Supabase Storage, we can use the Supabase JavaScript client. 

## Uploading a file

To upload a file, we can use the `upload` method.

For example, the code below is the code that uploads a user's profile image to the `avatars` bucket:

```ts
async function uploadUserProfilePhoto(
  client: SupabaseClient,
  photoFile: File,
  userId: string
) {
  const bytes = await photoFile.arrayBuffer();
  const bucket = client.storage.from('avatars');
  const extension = photoFile.name.split('.').pop();
  const fileName = `${userId}.${extension}`;

  const result = await bucket.upload(fileName, bytes, {
    upsert: true,
  });

  if (!result.error) {
    return bucket.getPublicUrl(fileName).data.publicUrl;
  }

  throw result.error;
}
```

You can also combine the above with `useMutation` from `react-query` when you use the code above from a React component:

```ts
export function useUploadUserProfilePhotoMutation() {
  const client = useSupabase();

  return useMutation(async (file: File, userId: string) => {
    return uploadUserProfilePhoto(client, file, userId);
  });
}
```

And then, you can use it like this:

```tsx
const updateProfilePhoto = useUploadUserProfilePhotoMutation();

<Form onUpload={(file: File, userId: string) => {
  return updateProfilePhoto.mutateAsync(file, userId)
} />
```

-----------------
FILE PATH: kits\remix-supabase\data-fetching\writing-data-to-supabase.mdoc

---
status: "published"
title: "Writing data to Database | Remix Supabase SaaS Kit"
label: "Writing data to Database"
order: 0
description: "Learn how to write, update and delete data using the Supabase Database in Remix Supabase SaaS Kit project."
---

Similarly to reading data, we may want to create a `custom
hook` also when we write, update or delete data to our Supabase database.

Assuming we have an entity called `tasks`, and we want to create a hook to
create a new `Task`, we can do the following.

1. First, we create a new hook at `lib/tasks/hooks/use-create-task.ts`
2. We add the `tasks` table name to `lib/db-tables.ts`
3. We create a `mutations.ts` file within `lib/tasks`

```ts
export const TASKS_TABLE = `tasks`;
```

In our mutations file, we will add all the mutations we want to perform on
the `tasks` table. In this case, we will add a `createTask` mutation.

```ts
export function createTask(
  client: Client,
  task: Task,
) {
  return client.from(TASKS_TABLE).insert(task).throwOnError();
}
````

Now, we can use the `createTask` mutation in our `useCreateTask` hook.

We will be using the `useMutation` hook from `react-query` to create our hook.

 ```tsx {% title="lib/tasks/hooks/use-create-task.ts" %}
function useCreateTaskMutation(task: Task) {
  const client = useSupabase();

  return useMutation(
    async (task: Task) => {
      return createTask(client, task);
    }
  );
}

export default useCreateTaskMutation;
```

Let's take a look at a complete example of a form that makes a request using
the hook above:

 ```tsx {% title="components/tasks/CreateTaskForm.tsx" %}
import { useNavigate } from '@remix-run/react';
import type { FormEventHandler } from 'react';
import { useCallback } from 'react';
import { toast } from 'sonner';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';
import useCreateTaskMutation from '~/lib/tasks/hooks/use-create-task';

import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';

const CreateTaskForm = () => {
  const createTaskMutation = useCreateTaskMutation();
  const navigate = useNavigate();
  const organization = useCurrentOrganization();
  const organizationId = organization?.id as number;

  const onCreateTask: FormEventHandler<HTMLFormElement> = useCallback(
    async (event) => {
      event.preventDefault();

      const target = event.currentTarget;
      const data = new FormData(target);
      const name = data.get('name') as string;
      const dueDate = (data.get('dueDate') as string) || getDefaultDueDate();

      if (name.trim().length < 3) {
        toast.error('Task name must be at least 3 characters long');

        return;
      }

      const task = {
        organizationId,
        name,
        dueDate,
        done: false,
      };

      // create task
      await createTaskMutation.mutateAsync(task);

      // redirect to /tasks
      return navigate(`/tasks`);
    },
    [navigate, createTaskMutation, organizationId]
  );

  return (
    <form onSubmit={onCreateTask}>
      <div>
        <TextField.Label>
          Name
          <TextField.Input
            required
            name={'name'}
            placeholder={'ex. Launch on IndieHackers'}
          />
          <TextField.Hint>Hint: whatever you do, ship!</TextField.Hint>
        </TextField.Label>

        <TextField.Label>
          Due date
          <TextField.Input name={'dueDate'} type={'date'} />
        </TextField.Label>

        <div>
          <Button>Create Task</Button>
        </div>
      </div>
    </form>
  );
};

export default CreateTaskForm;

function getDefaultDueDate() {
  const date = new Date();
  date.setDate(date.getDate() + 1);
  date.setHours(23, 59, 59);

  return date.toDateString();
}
```


-----------------
FILE PATH: kits\remix-supabase\data-fetching.mdoc

---
status: "published"
title: "Data Fetching in Remix Supabase"
label: "Data Fetching"
order: 5
description: "Data Fetching in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\remix-supabase\development\code-style.mdoc

---
status: "published"
title: "Adjust the codebase style according to your preferences | Remix Supabase Kit"
label: Code Style
order: 2
---

The Makerkit's boilerplate codebase is formatted with *Prettier*. You can configure it as you wish by updating the file Prettier configuration in the `package.json` file:

The default settings look like the below:

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "arrowParens": "always",
  "parser": "typescript",
  "printWidth": 80,
  "singleQuote": true
}
```

For example, to remove semicolons, update the `semi` and set it to `false`.

After adjusting the Prettier configuration, you can reformat the whole project by running the following command:

```
npm run format
```


-----------------
FILE PATH: kits\remix-supabase\development\commands.mdoc

---
status: "published"
title: "All the commands to use for your Makerkit app | Remix Supabase"
description: "Use these commands to run the development server, build the application, and more in your Remix Supabase application"
label: Commands
order: 0
---

Here are all the commands defined in the MakerKit's template:

### Run the development server

Run the command:

```
npm run dev
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

This is optional as it is automatically called after the `build` command.

### Format all the files

Run the command:

```
npm run format
```

### Type checking

Run the command:

```
npm run typecheck
```

### Linting

Run the command:

```
npm run lint
```

### Start the Supabase Local Environment

Run the command:

```
npm run supabase:start
```

This is needed during development. It requires Docker to be up and running.

### Stopping the Supabase Local Environment

Run the command:

```
npm run supabase:stop
```

### Reset the Supabase Local Environment Database

Run the command:

```
npm run supabase:db:reset
```

### Running the Supabase Database Tests

Run the command:

```
npm run test:db
```

### Running and resetting the Supabase Database Tests

Run the command:

```
npm run test:reset:db
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

### Run E2E Tests and Exit

Run the command:

```
npm test:e2e
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

### Kill Ports

The following commands kills all the ports that need to be free to run the Makerkit stack. This can be necessary after running the tests, for example when the emulators don't free up the ports after shutting down.

Run the command:

```
npm run killports
```


-----------------
FILE PATH: kits\remix-supabase\development\cors.mdoc

---
status: "published"
title: "Enable CORS | Remix Supabase SaaS Kit"
label: Enable CORS
order: 9
description: 'Learn how to enable CORS in your Remix Supabase SaaS Kit application'
---

Enabling CORS is required if you want to allow serving HTTP request to
external clients.

For example, if you want to expose an API to some
consumers: JS libraries, headless clients, and so on.

The code to enable CORS in Remix is very simple. In fact, you can enable it using the following code:

```tsx
function withCors() {
  const headers = new Headers();

  headers.append('Access-Control-Allow-Origin', '*');

  headers.append(
    'Access-Control-Allow-Headers',
    'Origin, X-Requested-With, Content-Type, Accept, referer-path'
  );

  return headers;
}
```

Additionally, you need to handle `OPTIONS` requests appropriately.

```tsx
if (request.method === `OPTIONS`) {
  return new Response(null, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, HEAD, POST, PUT, DELETE',
    },
  });
}
```

In your Makerkit codebase, this function is already available in the
`~/core/middleware/with-cors.ts` file.

To enable CORS, you can simply call it in your handler. If it fails, it will
throw an exception with the appropriate HTTP status code.

```tsx
import withCors from '~/core/middleware/with-cors';

export const action: ActionFunction = async ({}) => {
  withCors();
  // your logic
}

export default apiHandler;
```


-----------------
FILE PATH: kits\remix-supabase\development\emails.mdoc

---
status: "published"
title: "Sending emails with a Next.js Supabase SaaS application"
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

{% alert type="warning" %}
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

InBucket runs automatically when starting the local Supabase environment.

InBucket will now be running at [localhost:54324](http://localhost:54324). When we send an email using the
`sendEmail` function in the kit, we can visualize it using the InBucket UI.

InBucket is used by default during development. Instead, for production
usage, you will need to set up a real SMTP service.

#### Ethereal

In previous versions Makerkit used Ethereal. If you are running an older version, please refer to the below.

Makerkit used Ethereal to allow you testing your emails without using a real email account. To use Ethereal, you should add the following environment variables to your project, using a secure environment:

```bash
ETHEREAL_EMAIL=
ETHEREAL_PASSWORD=
```

If not set, Makerkit will automatically create an account for you on-the-fly and print the credentials in the console. You can then use these credentials to log in to Ethereal and see your emails.


-----------------
FILE PATH: kits\remix-supabase\development\encrypting-secrets.mdoc

---
status: "published"
title: "Encrypting Secrets | Remix Supabase SaaS Kit"
label: Encrypting Secrets
order: 9
description: "When storing sensitive data, such as API keys, you must ensure
to encrypt your data in the Remix Supabase database. Here is how we can encrypt and
decrypt it."
---

When storing sensitive data, such as API keys, you must ensure to encrypt your data in Database.

There are two ways to encrypt your data in Supabase:
- [Transparent Column Encryption with Postgres](https://supabase.com/blog/transparent-column-encryption-with-postgres)
- **Supabase Vault**: This is a generic solution for encrypting data in your database. We have an article about it. [Read more about Supabase Vault](/blog/tutorials/supabase-vault).


-----------------
FILE PATH: kits\remix-supabase\development\i18n-translations.mdoc

---
status: "published"

title: "Adding and updating locales in your Makerkit application"
label: Translations and Locales
order: 5
---


Much of the MakerKit's codebase uses translations using the packages`react-i18next` and `i18next`:

- this package allows you to create multi-language apps effortlessly
- it also helps easily renaming the Makerkit's entities according to your preferences: for example, renaming "organizations" to another entity such as "team" or "workspace"
- in a way, it helps write cleaner HTML

To add or update the default strings, you must use the locale files at `public/locales/{lang}/{file}.json`. For example, all the locale files for the English language are placed under `public/locales/en`.

{% alert type="warning" %}
When adding new keys, you'll need to restart the MakerKit server (`npm run dev`) to see the changes. Also, you may need to clear the browser cache since the client is hydrated with the translation files fetched from the JSON files.
{% /alert %}

The locales are split by functionality, page, or entity: as your application grows, you may not want to load every JSON file to reduce your bundle size.

For example, the translations for the authentication are placed in `auth.json,` while the ones used across every page are placed under `common.json.`

If you are unsure where to add some translations, simply add them to `common.json`.

{% alert type="info" %}
To simplify things, the MakerKit's starter uses all the available JSON files, and we recommend doing so until you have too many files.
{% /alert %}

### Adding new languages

By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:
1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **i18n config**: We need to also add a new language to the Remix configuration at `app/i18n/i18n.config.ts`. Simply add your new language's code to the `languages` array.

The configuration will look like the below:

 ```js {% title="app/i18n/i18n.config.ts" %}
const i18Config = {
  fallbackLanguage: DEFAULT_LOCALE,
  supportedLanguages: [DEFAULT_LOCALE, 'es'],
  defaultNS: ['common', 'auth', 'organization', 'profile', 'subscription'],
  react: { useSuspense: false },
};
```

To add new locale files by default, add them to the `i18Config.supportedLanguages` array.

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```

## Using the Language Switcher

The MakerKit kits come with a language switcher that allows users to switch between the available languages. This is not enabled by default, but you can easily add it anywhere in your application by importing the `LanguageSwitcherDropdown` component from `~/components/LanguageSwitcherDropdown`. 

It does not require any props, and it will automatically detect the available languages and display them in a dropdown.

If you want to customize the language switcher or use a different component, you can leverage the `useChangeLanguage` hook to change the language and store the selection as a cookie, which will then be used to set the default language.


-----------------
FILE PATH: kits\remix-supabase\development\logging.mdoc

---
status: "published"
title: "Logging | Remix Supabase SaaS Kit"
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

Let's assume you're writing an API function to write to your Supabase database:

```tsx
function addIntegrationHandler() {
  return writeToDb(data);
}
```

Let's rewrite the above by adding logging to your function:

```tsx
import logger from '~/core/logger';

async function addIntegrationHandler(
  userId: string,
  organizationId: number,
  integrationId: number,
) {
  // this is the context that every log will print out
  const loggingContext = {
    integrationId,
    organizationId,
    userId,
  };

  // Here we log what we're doing
  logger.log(loggingContext, `Adding new integration to organization`);

  try {
    await writeToDb(data);

    // Here we log that the result of the operation
    // was successful
    logger.log(loggingContext, `Integration successfully added`);

    // return successful response

    return {
      success: true
    };
  } catch (e) {
    // Here we log that the operation failed
    logger.error(loggingContext, `Encountered an error while adding integration`);

    // Logging errors can be okay but
    // ensure not to leak important information!
    logger.debug(e);

    // return error response
    return {
      success: false,
    };
  }
}
```

If you use the filter `withExceptionFilter`, it will automatically
log eventual exceptions thrown by the function, which makes the try/catch
block optional.

NB: If you're using Vercel, logs are not persisted by default! [Check out the
Vercel integrations](https://vercel.com/integrations#logging) for adding persisting logs to your projects.


-----------------
FILE PATH: kits\remix-supabase\development\user-roles.mdoc

---
status: "published"
title: "Extend User Roles | Remix Supabase SaaS Starter Kit"
description: "Learn how to extend the User Roles in the Remix Supabase SaaS Starter Kit"
label: User Roles
order: 9
---

The MakerKit Starter has three default roles:

```tsx
enum MembershipRole {
  Member = 0,
  Admin = 1,
  Owner = 2,
}

export default MembershipRole;
```

We use an `enum`, but you can convert this to an object if you need more granular permissions.

The permissions are hierarchical, which means that if we had a role with a lower level (`Readonly`), we would add it before `Member`:

```tsx
export enum MembershipRole {
  Readonly = 0,
  Member = 1,
  Admin = 2,
  Owner = 3,
}
```

When writing permissions between users, we can check if the user performing the action has a greater role than the target user.

You can extend the role above easily by adding your own, for example:

```tsx
export enum MembershipRole {
  Readonly = 0,
  AccountManager = 1,
  Owner = 2,
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

The role is stored in the database as a column in the `memberships` tabel.

-----------------
FILE PATH: kits\remix-supabase\development\validating-api-input-zod.mdoc

---
status: "published"

title: "Validating the Remix API inputs with Zod and Typescript"
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

export async function POST(request: Request) {
  try {
     // we can safely use data with the interface Body
    const body = await request.json();
    const bodyResult = await getBodySchema().parseAsync(body);
    const { displayName, email } = bodyResult.data;

    return sendInvite({ displayName, email });
  } catch(e) {
    return throwBadRequestException();
  }
}
```

You can also use `safeParse` if you prefer not to throw an error when the
validation fails:

```tsx
export async function POST(request: Request) {
  const body = await request.json();
  const result = await getBodySchema().parseAsync(body);

  // we use result.success as a type guard
  // when false, we throw an exception
  if (!result.success) {
    return throwBadRequestException();
  }

  // TS correctly infers result.data now
  return sendInvite(result.data);
}
```

To learn more about validating data with Zod, we suggest you
check out [the Zod official documentation on GitHub](https://github.com/colinhacks/zod).


-----------------
FILE PATH: kits\remix-supabase\development.mdoc

---
status: "published"
title: "Development in Remix Supabase"
label: "Development"
order: 6
description: "This section will teach you everything you need to know about developing with Remix Supabase."
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\remix-supabase\getting-started\clone-repository.mdoc

---
status: "published"
title: Clone the Remix Supabase SaaS boilerplate repository
label: Clone the repository
order: 2
description: 'Learn how to clone the  Remix Supabase SaaS kit repository'
---


If you have bought a license for MakerKit, you have access to all the
repositories built by the MakerKit team. In this document, we will learn how
to fetch and install the codebase.

### Requirements

To get started with the Remix and Supabase SaaS template, we need to ensure
you install the required software.

- Node.js
- Git
- Docker

### Remix Versions

The current main uses Remix v2.

### Cloning the repository

To get the codebase on your local machine, clone the repository with the
following command:

```
git clone git@github.com:makerkit/remix-supabase-saas-kit.git my-saas
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
git remote add upstream git@github.com:makerkit/remix-supabase-saas-kit.git
git add .
git commit -a -m "Initial Commit"
```

In this way, to fetch updates (after committing your files), simply run:

```
git pull upstream main
```

If it doesn't work, try with:

```
git pull upstream main --allow-unrelated-histories
```

You'll likely run into conflicts when running this command, so carefully choose the changes (sorry!).

### Installing the Node dependencies

Finally, we can install the NodeJS dependencies with `npm`:

```
npm i
```

While the application code is fully working, we now need to set up your Supabase
project.

So let's jump on to the next step!


-----------------
FILE PATH: kits\remix-supabase\getting-started\introduction.mdoc

---
status: "published"

title: 'Introduction to MakerKit: a SaaS boilerplate for Remix and Supabase'
label: 'Introduction'
order: 0
description: 'Introduction to MakerKit: a SaaS starter built with Remix and Supabase'
---


MakerKit is **the Remix.js boilerplate project for SaaS** built to get you
started on the right foot.

The MakerKit kit is a fully-featured Remix application that uses Supabase for
authentication, database (with Postgres), and storage.

MakerKit is ideal for any SaaS application or dynamic websites such as walled Online Publications.

It particularly shines when you wish to provide uniform navigation between your marketing site, documentation, and application.

## What does the boilerplate provide?

### Tech Stack

We built MakerKit with some of the best technologies available today, such as
Remix, Tailwind CSS, and Supabase:

- **Scalable Remix** structure template, perfect for the most ambitious
projects
- Beautiful **Tailwind 3** CSS theme - with dark mode!
- **Full Supabase** setup, including Authentication, Database and Storage
- **Reusable UI components** as building blocks (which you can easily swap
with your own or your favorite library) based on [Radix UI](https://radix-ui.
com)
- **SSR-compatible Authentication flow**, including 3rd-party providers (Google,
Facebook, Twitter, GitHub, etc.)

### Template Features

MakerKit is fully functioning from the beginning and comes with the
following features:

- **Payments** with Stripe, and support for **subscriptions** (or, alternatively, using Lemon Squeezy)
- **Onboarding flow**, which allows your users to set up their accounts and
create their organization
- **Organizations**: users can create and edit organizations. An organization is a group of users. You can rename the `Organization` entity according to your domain (for example, teams, projects, etc.)
- **Team Members**: users can invite other users to join their organization and assign them a role
- **MDX-powered Blog and Documentation** generators - already [Coming Soon]
**SEO-optimized** for you

### Marketing

Marketing matters! But don't get hung up on the technicalities. MakerKit provides:

- **Newsletter Sign-up form** for *ConvertKit*, but easy to adapt to more
providers
- **Email Templates** you can write with React using [React.email](https://react.email)

### Debugging and Testing

Never waste time chasing bugs again.

Assuming you have created a Sentry account, MakerKit can catch exceptions (and possible bugs) and  report them to Sentry whenever they happen in both client and server-side code:

- **Error Tracking** set up with Sentry
- Comprehensive **E2E Tests** suite with Cypress, which you can use and learn
from

### After you buy

You're not going to be left alone. We offer help and support:

- Access to the team for feedback, requests, and general support. **We're here to support you build your SaaS**.

This documentation introduces each of the points above and guides you through
so that you can easily adjust MakerKit to your application's domain.

MakerKit has plenty of resources for using both Remix and Supabase - and if you're stuck, you can always reach out to me for help.


