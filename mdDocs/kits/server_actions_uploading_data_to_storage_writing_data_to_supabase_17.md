-----------------
FILE PATH: kits/next-supabase/data-fetching/server-actions.mdoc

---
status: "published"
title: "Mutating data using Server Actions in the Next.js Supabase Lite Starter Kit"
label: "Mutating data using Server Actions"
order: 3
description: "Learn how mutate data using the new Next.js Server Actions in the Makerkit Supabase Next.js starter kit (Lite)"
---

Starting from the version 0.2.0, the Makerkit Supabase Next.js starter kit comes with support for Server Actions. Next.js Server Actions allow us to call functions on the server side, which is useful for mutating data.

This is the default way of mutating data in Makerkit - so **I recommend you to use this approach instead of the client-side approach**.

Makerkit offers support for some built-in features that allow you to protect your endpoints form unauthorized access and CSRF attacks.

## Creating a Server Action

You have two ways of creating a Server Action:

1. Defining a server function inline within a server component
2. Defining server functions into a separate file and exporting them to be used inside client components

### Defining a file for server functions

Assuming you have a `lib/tasks` folder, you can create a file called `lib/tasks/actions.ts` and define server actions inside it.

#### How to define a server action

Actions can be defined in two ways:

1. If you want to pass it to a `form` element, you need to use `FormData` as the parameter type.
2. Alternatively, you can customize the parameters as you wish - and call it imperatively from a client component just like a normal function

#### Utility functions

We can use two utilities:

1. `withSession` - this utility will check if the request contains a valid session
2. `withAdminSession` - this utility will check if the request contains a valid session from a Super Admin that can access the admin panel

#### Defining a Form Action

When defining server actions that will be used by a `form` element, you need to use the `FormData` type as the parameter type.

```ts
'use server';

import { z } from 'zod';

import { withSession } from '~/core/generic/actions-utils';
import getSupabaseServerActionClient from '~/core/supabase/action-client';

const zodSchema = z.object({
  task: z.object({
    name: z.string().min(1),
  }),
});

export const insertNewTask =
  withSession(
    async (data: FormData) => {
      const client = getSupabaseServerActionClient();
      const body = zodSchema.parse(Object.fromEntries(data.entries()));

      await insertNewTask(client, body.task);

      return {
        success: true,
      };
    }
  );
```

To call server actions from Forms, you can use the `action` attribute. This attribute will be used to call the server action.

```tsx
function TaskForm() {
  return (
    <form action={insertNewTask}>
    ...
    </form>
  );
}

export default TaskForm;
```

### Defining a Custom Server Action

Below we define a custom server action that takes two parameters: `task` and `csrfToken`. This action will insert a new task into the database.

```ts
'use server';

import { withSession } from '~/core/generic/actions-utils';
import getSupabaseServerActionClient from '~/core/supabase/action-client';

export const insertNewTask = withSession(
    async (params: {
      task: Task;
    }) => {
      const client = getSupabaseServerActionClient();

      await insertNewTask(client, params);

      return {
        success: true,
      };
    }
);
```

To call this function from a client component, you can do the following:

```tsx
import {FormEventHandler, useCallback, useTransition } from "react";

function TaskForm() {
  const [isPending, startTransition] = useTransition();

  const onSubmit: FormEventHandler = useCallback(e => {
    e.preventDefault();

    const data = new FormData(e.target as HTMLFormElement);
    const taskName = data.get("name") as string;

    startTransition(async () => {
      await insertNewTask({
        task: {
          name: taskName,
        }
      });
    });
  });

  return (
    <form onSubmit={onSubmit}>
    ...
    </form>
  );
}

export default TaskForm;
```

## Learn more about Server Actions

To learn more about server actions, we have a simple tutorial that you can follow: [Introduction to Next.js Server Actions](/blog/tutorials/nextjs-server-actions)

-----------------
FILE PATH: kits/next-supabase/data-fetching/uploading-data-to-storage.mdoc

---
status: "published"
title: "Uploading data to Storage | Next.js Supabase"
description: "Learn how to upload your data to Supabase Storage in your Next.js application"
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

You can also combine the above with `useSWRMutation` from `SWR` when you use the code above from a React component:

```ts
import useSWRMutation from 'swr/mutation';

function useUpdateProfile() {
  const client = useSupabase();
  const key = 'useUpdateProfile';

  return useSWRMutation(key, async (_, { arg: data }: { arg: {
    file: File;
    userId: string;
  } }) => {
    return updateUserData(client, data);
  });
}
```

And then, you can use it like this:

```tsx
const updateProfile = useUpdateProfile();

<Form onUpload={(file: File, userId: string) => {
  return updateProfile.trigger({ file, userId })
} />
```

-----------------
FILE PATH: kits/next-supabase/data-fetching/writing-data-to-supabase.mdoc

---
status: "published"
title: "Writing data to Database | Next.js Supabase SaaS Kit"
label: "Writing data to Database"
order: 0
description: "Learn how to write, update and delete data using the Supabase Database in  Next.js Supabase SaaS project"
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
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '../../database.types';
import type Task from '~/lib/tasks/types/task';
import { TASKS_TABLE } from '~/lib/db-tables';

type Client = SupabaseClient<Database>;

export function createTask(client: Client, task: Omit<Task, 'id'>) {
  return client.from(TASKS_TABLE).insert({
    name: task.name,
    organization_id: task.organizationId,
    due_date: task.dueDate,
    done: task.done,
  });
}
```

### Calling mutation using a Server Action

The best way to call a mutation is to use a Next.js Server Action. This way, we can use the `revalidatePath` function to revalidate the page that we want to revalidate after the mutation is completed.

```tsx
'use server';

import { revalidatePath } from 'next/cache';

import { createTask } from '~/lib/tasks/mutations';
import type Task from '~/lib/tasks/types/task';
import { withSession } from '~/core/generic/actions-utils';
import getSupabaseServerActionClient from '~/core/supabase/action-client';

type CreateTaskParams = {
  task: Omit<Task, 'id'>;
  csrfToken: string;
};

export const createTaskAction = withSession(
  async (params: CreateTaskParams) => {
    const client = getSupabaseServerActionClient();

    await createTask(client, params.task);

    revalidatePath('/dashboard/[organization]/tasks');
  }
);
```

Then, we call the action from a form:

```tsx
'use client';

import type { FormEventHandler } from 'react';
import { useCallback, useTransition } from 'react';
import { toast } from 'sonner';
import { useRouter } from 'next/navigation';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';
import If from '~/core/ui/If';
import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';
import { createTaskAction } from '~/lib/tasks/actions';
import useCsrfToken from '~/core/hooks/use-csrf-token';

const CreateTaskForm = () => {
  const [isMutating, startTransition] = useTransition();
  const organization = useCurrentOrganization();
  const organizationId = organization?.id as number;
  const csrfToken = useCsrfToken();
  const router = useRouter();

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

      startTransition(async () => {
        await createTaskAction({ task, csrfToken });

        router.push('../tasks');
      });
    },
    [csrfToken, organizationId, router]
  );

  return (
    <form onSubmit={onCreateTask}>
      <div className={'flex flex-col space-y-2'}>
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

        <div
          className={
            'flex flex-col space-y-2 md:flex-row md:space-x-2 md:space-y-0'
          }
        >
          <Button loading={isMutating}>
            <If condition={isMutating} fallback={<>Create Task</>}>
              Creating Task...
            </If>
          </Button>

          <Button color={'transparent'} href={'../tasks'}>
            Go back
          </Button>
        </div>
      </div>
    </form>
  );
};

function getDefaultDueDate() {
  const date = new Date();
  date.setDate(date.getDate() + 1);
  date.setHours(23, 59, 59);

  return date.toDateString();
}

export default CreateTaskForm;
```

### Calling mutation using SWR

Alternatively, we can use the `createTask` mutation in our `useCreateTask` hook.

We will be using the `useSWRMutation` hook from `swr` to create our hook.

 ```tsx {% title="lib/tasks/hooks/use-create-task.ts" %}
import useSWRMutation from 'swr/mutation';
import { useRouter } from 'next/navigation';

import useSupabase from '~/core/hooks/use-supabase';
import { createTask } from '~/lib/tasks/mutations';
import type Task from '~/lib/tasks/types/task';

function useCreateTaskMutation() {
  const client = useSupabase();
  const router = useRouter();
  const key = 'tasks';

  return useSWRMutation(key, async (_, { arg: task }: { arg: Omit<Task, 'id'> }) => {
    return createTask(client, task);
  }, {
    onSuccess: () => router.refresh()
  });
}

export default useCreateTaskMutation;
```

Let's take a look at a complete example of a form that makes a request using
the hook above:

 ```tsx {% title="components/tasks/CreateTaskForm.tsx" %}
import { useRouter } from 'next/router';
import type { FormEventHandler } from 'react';
import { useCallback } from 'react';
import { toast } from 'sonner';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';
import useCreateTaskMutation from '~/lib/tasks/hooks/use-create-task';

import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';

const CreateTaskForm = () => {
  const createTaskMutation = useCreateTaskMutation();
  const router = useRouter();
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
      await createTaskMutation.trigger(task);

      // redirect to /tasks
      return router.push(`/tasks`);
    },
    [router, createTaskMutation, organizationId]
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
FILE PATH: kits/next-supabase/data-fetching.mdoc

---
status: "published"
title: "Data Fetching in Next.js Supabase"
label: "Data Fetching"
order: 5
description: "Data Fetching in Next.js Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits/next-supabase/development/code-style.mdoc

---
status: "published"
title: "Adjust the codebase style according to your preferences | Next.js Supabase Kit"
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
FILE PATH: kits/next-supabase/development/commands.mdoc

---
status: "published"
title: "All the commands to use for your Makerkit app | Next.js Supabase"
description: "Use these commands to run the development server, build the application, and more in your Next.js Supabase application"
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

-----------------
FILE PATH: kits/next-supabase/development/cors.mdoc

---
status: "published"
title: "Enable CORS | Next.js Supabase SaaS Kit"
label: Enable CORS
order: 9
description: 'Learn how to enable CORS in your Next.js Supabase Saa S Kit application'
---

Enabling CORS is required if you want to allow serving HTTP request to
external clients.

For example, if you want to expose an API to some
consumers: JS libraries, headless clients, and so on.

The code to enable CORS in Next.js is very simple. In fact, you can enable it
using the following code:

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
import { NextRequest } from "next/server";

export async function GET(request: NextRequest) {
  if (request.method === `OPTIONS`) {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, HEAD, POST, PUT, DELETE',
      },
    });
  }
}
```

In your Makerkit codebase, this function is already available in the
`~/core/middleware/with-cors.ts` file.

To enable CORS, you can simply call it in your handler. If it fails, it will
throw an exception with the appropriate HTTP status code.

```tsx
import withCors from '~/core/middleware/with-cors';

export async function GET(request: NextRequest) {
  const headers = withCors();

  if (request.method === `OPTIONS`) {
    return new Response(null, {
      headers,
    });
  }
}
```


-----------------
FILE PATH: kits/next-supabase/development/emails.mdoc

---
status: "published"
title: "Sending emails | Next.js Supabase"
label: Sending Emails
order: 6
---

MakerKit uses the popular package `nodemailer` to allow you to send emails.

Normally, you would use a service for sending transactional emails such as SendGrid, Mailjet, MailGun, Postmark, and so on.

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
When running the emulators, by default, Makerkit uses InBucket for sending
  emails, and not your production service. Read below for more info.
{% /alert %}

This configuration will only be used for sending emails for inviting team members. For emails related to authentication, you will need to update the configuration in the Supabase Dashboard.

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

Makerkit uses [InBucket](https://inbucket.org) - a platform to testing emails.

InBucket saves time during development since we can test
our emails without setting up a real SMTP service - and works locally and
offline.

To run the InBucket platform, **we need Docker running on our machine**. InBucket runs when the Supabase development environment is started, so you don't need to do anything else.

InBucket will now be running at [localhost:54324](http://localhost:54324). When we send an email using the
`sendEmail` function in the kit, we can visualize it using the InBucket UI.

InBucket is used by default during development. Instead, for production
usage, you will need to set up a real SMTP service.

#### Ethereal

In **previous versions** Makerkit used Ethereal. If you are running an older version, please refer to the below.

Makerkit used Ethereal to allow you testing your emails without using a real
email account. To use Ethereal, you should add the following environment variables to your project, using a secure environment:

```bash
ETHEREAL_EMAIL=
ETHEREAL_PASSWORD=
```

If not set, Makerkit will automatically create an account for you on-the-fly and print the credentials in the console. You can then use these credentials to log in to Ethereal and see your emails.



-----------------
FILE PATH: kits/next-supabase/development/encrypting-secrets.mdoc

---
status: "published"
title: "Encrypting Secrets | Next.js Supabase SaaS Kit"
label: Encrypting Secrets
order: 9
description: "When storing sensitive data, such as API keys, you must ensure
to encrypt your data in Next.js Supabase. Here is how we can encrypt and decrypt it."
---

When storing sensitive data, such as API keys, you must ensure to encrypt your data in Database.

There are two ways to encrypt your data in Supabase:
- [Transparent Column Encryption with Postgres](https://supabase.com/blog/transparent-column-encryption-with-postgres)
- **Supabase Vault**: This is a generic solution for encrypting data in your database. We have an article about it. [Read more about Supabase Vault](/blog/tutorials/supabase-vault).


-----------------
FILE PATH: kits/next-supabase/development/i18n-translations.mdoc

---
status: "published"

title: "Adding and updating locales in your Makerkit application"
label: Translations and Locales
order: 5
---


The MakerKit's codebase uses translations using the package `i18next`.

- this package allows you to create multi-language apps effortlessly
- it also helps easily renaming the Makerkit's entities according to your preferences: for example, renaming "organizations" to another entity such as "team" or "workspace"
- in a way, it helps write cleaner HTML

To add or update the default strings, you must use the locale files at `public/locales/{lang}.json`. For example, all the locale files for the English language are placed under `public/locales/en`.

{% alert type="warning" %}
When adding new keys, you'll need to restart the MakerKit server (`npm run dev`) to see the changes. Also, you may need to clear the browser cache since the client is hydrated with the translation files fetched from the JSON files.
{% /alert %}

The locales are split by functionality, page, or entity: as your application grows, you may not want to load every JSON file to reduce your bundle size.

For example, the translations for the authentication are placed in `auth.json,` while the ones used across every page are placed under `common.json.`

If you are unsure where to add some translations, simply add them to `common.json`.

{% alert type="info" %}
To simplify things, the MakerKit's starter uses all the available JSON files, and we recommend doing so until you have too many files.
{% /alert %}

### Using Translations in Server Components

To use translations in Server Components, you can use the `withI18n` higher order server component function, which wraps the server component and initializes i18n before rendering the component.

```tsx
import { withI18n } from '~/i18n/with-i18n';

function DashboardPage() {
  return (
    <>
     Your page...
    </>
  );
}

export default withI18n(DashboardPage);
```

### Adding new languages

By default, Makerkit uses English for translating the website's text. All the files are stored in the files at `/public/locales/en`.

Adding a new language is very simple:
1. **Translation Files**: First, we need to create a new folder, such as `/public/locales/es`, and then copy over the files from the English version and start translating files.
2. **Next.js i18n config**: We need to also add a new language to the i18n configuration at `src/i18n/i18n.settings.ts`. Now add your new language's code to the `languages` array.

The configuration will look like the below:

 ```js {% title="src/i18n/i18n.settings.ts" %}
const languages: string[] = [fallbackLng, 'es'];
```

#### Setting the default Locale

To set the default locale, simply update the environment variable `DEFAULT_LOCALE` stored in `.env`.

So, open the `.env` file, and update the variable:

 ```txt {% title=".env" %}
DEFAULT_LOCALE=de
```

## Using the Language Switcher

The MakerKit kits come with a language switcher that allows users to switch between the available languages. This is not enabled by default, but you can easily add it anywhere in your application by importing the `LanguageSwitcherDropdown` component from `~/components/LanguageSwitcherDropdown`. 

It does not require any props, and it will automatically detect the available languages and display them in a dropdown.


-----------------
FILE PATH: kits/next-supabase/development/logging.mdoc

---
status: "published"
title: "Logging | Next.js Supabase SaaS Kit"
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

Let's assume you're writing an API function to write to your database:

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
      integrationId,
      success: true
    };
  } catch (e) {
    // Here we log that the operation failed
    logger.error(loggingContext, `Encountered an error while adding integration`);

    // Logging errors can be okay but
    // ensure not to leak important information!
    logger.debug(e);

    throw e;
  }
}
```

NB: If you're using Vercel, logs are not persisted by default! [Check out the
Vercel integrations](https://vercel.com/integrations#logging) for adding persisting logs to your projects.



