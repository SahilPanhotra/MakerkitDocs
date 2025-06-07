-----------------
FILE PATH: kits/next-supabase/tutorials/running-project.mdoc

---
status: "published"
title: "Running the App | Next.js Supabase SaaS Kit"
label: Running the App
order: 3
description: "Learn how to run the app locally for development and testing."
---

When we run the stack of a Makerkit application, we will need to run a few commands before:

1. **Next.js**: First, we need to run the Next.js development server
2. **Supabase**: We need to run the Supabase local environment using Docker
3. **Stripe CLI** (optional): If you plan on interacting with Stripe, you are also required to run the Stripe CLI, which allows us to test the webhooks from Stripe against our local server.

### Why do we need to run the Supabase local environment?

The guide below will show you how to run the application locally. But why locally? Developing locally has many advantages over developing on a remote server:

**Speedy Development:** Local development lets you work without the hindrance of network latency or internet disruptions.

**Simplified Collaboration:** It's typically easier to collaborate with teammates when developing locally on the same project.

**Cost Efficiency:** Supabase offers a generous free tier, including two free projects. However, if you need more than two, local development comes into play. You can create countless local projects and establish links with live projects at your convenience for launch.

**Code-Based Configuration:** Any modifications to your tables via the Dashboard aren't reflected in code. Adopting local development workflows allows you to maintain all table schemas in code.

Once you are ready to deploy your application, you can do so by following the [production checklist guide](/docs/next-supabase/going-to-production-overview).

Fore more information, check out the [Supabase documentation](https://supabase.com/docs/guides/cli/local-development).

## Install and Run Docker

Before we can run the Supabase local environment, we need to run Docker, as Supabase uses it for running its local environment.

You can use Docker Desktop, Colima, OrbStack, or any other Docker-compatible solution.

### Running the Next.js development server

Run the following command from your IDE or from your terminal:

```
npm run dev
```

This command will start the Next.js server at [localhost:3000](http://localhost:3000). If you navigate to this page, you should be able to see the landing page of your application:

{% img src="/assets/images/posts/tutorial-landing-page.webp" width="2856" height="1972" /%}

### Running the Supabase Local environment

To run the Supabase local environment, **we need to first run Docker**. If you don't have Docker installed, you can download and install Docker Desktop or alternatives such as Colima or Orbstack.

Then, open a new terminal (or, better, from your IDE) and run the following
commands:

```
npm run supabase:start
```

If everything is working correctly, you will see the output below:

```
> supabase start

Applying migration 20221215192558_schema.sql...
Seeding data supabase/seed.sql...
Started supabase local development setup.

         API URL: http://localhost:54321
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
    Inbucket URL: http://localhost:54324
      JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
        anon key: ****************************************************
service_role key: ****************************************************
```

Now, we need to copy the `anon key` and `service_role key` values and add
them to the `.env.local` file:

```
NEXT_PUBLIC_SUPABASE_ANON_KEY=****************************************************
SUPABASE_SERVICE_ROLE_KEY=****************************************************
```

When you need to stop the development environment, you can run the following command:

```
npm run supabase:stop
```

#### Supabase Studio UI

To access the Supabase Studio UI, open a new tab in your browser and navigate to [http://localhost:54323](http://localhost:54323).

{% alert type="info" %}
Makerkit's template adds some data to your project by default for testing reasons. In fact, the data you see is used to run the Cypress E2E tests. This is why you'll see some pre-populated data in your project.
{% /alert %}

### Running the Stripe CLI (optional)

Optionally, if you want to run Stripe locally (e.g., sending webhooks to your local server), we need two more steps.

1. Have a Stripe account
2. Run the Stripe CLI (or install Stripe on your OS)

To run the CLI, run the following command:

```
npm run stripe:listen
```

When running the command above for the first time, **Stripe will ask you to login**. You can do so by following the instructions in the terminal.

After signing in, run the command again.

```
npm run stripe:listen
```

The above command runs the Stripe CLI and will route webhooks coming from
Stripe to your local endpoint. For example, in the Supabase Next.js SaaS starter, this
endpoint is at `/stripe/webhook`.

#### Adding the Stripe Webhook Secret

When running the command, it will print a webhook key used to sign the messages from Stripe. You will need to add this key to your local environment variables file as below:

```
STRIPE_WEBHOOK_SECRET=<KEY>
```

The webhook printed should not change, so you may only need to do this the first time.

### Signing In for the first time

You should now be able to sign in. To quickly get started, use the following credentials:

```
email = test@makerkit.dev
password = testingpassword
```

### Email Confirmations for Testing with InBucket

When signing up, Supabase sends an email confirmation to a testing account.

You can access the InBucket testing emails [at http://localhost:54324/monitor](http://localhost:54324/monitor), and can follow the links to complete the sign up process.

InBucket is started automatically when you run `npm run supabase:start`.

## More about developing locally with Supabase

For more information about developing locally with Supabase, check out the [Supabase documentation](https://supabase.com/docs/guides/cli/local-development).

-----------------
FILE PATH: kits/next-supabase/tutorials/supabase-data-fetching.mdoc

---
status: "published"

title: "Supabase: Data Fetching"
label: "Supabase: Data Fetching"
order: 8
description: "Learn how to fetch data from Supabase in your React applications."
---


With our tables and RLS in place, we can finally start writing our queries
using the Supabase Postgres client.

My convention (used throughout the boilerplate) is to **write
functions that inject the Supabase SDK Client as a parameter**: this allows us to
reuse these functions in both the browser and the server.

## Fetching a list of tasks

First, we want to write a function that is responsible for fetching the
paginated list of tasks using the Supabase Postgres client.

Below, we will write the query necessary to fetching the tasks for a given project:

- paginated using the `pageIndex` and `perPage` parameters
- filtered by the `query` parameter that is matched against the `name` column.

 ```tsx {% title="lib/tasks/queries.ts" %}
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '~/database.types';
import { TASKS_TABLE } from '~/lib/db-tables';
import Task from '~/lib/tasks/types/task';

type Client = SupabaseClient<Database>;

const TASKS_PAGE_SIZE = 10;

export function getTasks(
  client: Client,
  params: {
    organizationUid: string;
    pageIndex: number;
    perPage?: number;
    query?: string;
  },
) {
  const { organizationUid, perPage, pageIndex } = params;
  const { startOffset, endOffset } = getPaginationOffsets(pageIndex, perPage);

  let query = client
    .from(TASKS_TABLE)
    .select<string, Task>(
      `
      id,
      name,
      organizationId: organization_id,
      dueDate: due_date,
      done,
      description,
      organization: organization_id !inner (
        id,
        uuid
      )
    `, { count: 'exact' }
    )
    .eq('organization.uuid', organizationUid)
    .range(startOffset, endOffset);

  if (params.query) {
    query = query.textSearch('name', params.query);
  }

  return query;
}

function getPaginationOffsets(pageIndex: number, perPage?: number) {
  const pageSize = perPage || TASKS_PAGE_SIZE;
  const startOffset = pageIndex * pageSize;
  const endOffset = startOffset + pageSize;

  return {
    startOffset,
    endOffset,
  };
}
```

## Fetching a single task

Next, we want to write a function that is responsible for fetching a single task when viewing the task details page. We can place this function in the same file as the `getTasks` function.

 ```tsx {% title="lib/tasks/queries.ts" %}
export function getTask(client: Client, id: number) {
  return client
    .from(TASKS_TABLE)
    .select(
      `
      id,
      name,
      organizationId: organization_id,
      dueDate: due_date,
      description,
      done
    `,
    )
    .eq('id', id)
    .single();
}
```

---


We now have all the queries we need to fetch tasks from Supabase, both as a paginated list and as a single task. In the next section, we will learn how to use these queries from the React Server Components.

-----------------
FILE PATH: kits/next-supabase/tutorials/supabase-data-writing.mdoc

---
status: "published"
title: "Supabase: Data Writing in Next.js Supabase"
label: "Supabase: Data Writing"
order: 9
description: "Learn how to write data to the Supabase database in your Next.js applications."
---


In the section above, we saw how to fetch data from the Supabase Postgres.
Now, let's see how to write data to the Supabase database.

#### Creating a Task

First, we want to add a file named `mutations.ts` at `lib/tasks/`. Here, we
will add all the mutations that we will need to create, update, and delete
tasks.

In our mutations file, we will add all the mutations we want to perform on
the `tasks` table. In this case, we will add a `createTask` mutation.

```ts
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '../../database.types';
import type Task from '~/lib/tasks/types/task';
import { TASKS_TABLE } from '~/lib/db-tables';

type Client = SupabaseClient<Database>;

export function createTask(
  client: Client,
  task: Omit<Task, 'id'>
) {
  return client.from(TASKS_TABLE).insert({
    name: task.name,
    organization_id: task.organizationId,
    due_date: task.dueDate,
    description: task.description,
    done: task.done,
  });
}
```

You now have 2 alternatives to call the `createTask` mutation:

1. Use a Server Action (recommended)
2. Use an SWR Mutation

Let's explain the differences between the two:

1. **Server Actions** are functions executed server side - and allow us to revalidate the page after a mutation, so we can see the changes immediately.
2. **SWR** is a client-side library that allows us to fetch data and mutate it. Since the Supabase Client allows us to perform queries/mutations client-side, we can use it in conjunction with SWR to use it within client-side React components. There are times where you may not need to revalidate the page after a mutation, and in those cases, SWR is also a good option.

Most of the time, you will want to use a Server Action, but we will show both alternatives.

### 1. Using a Server Action

Using server actions is the simplest and most straightforward way to perform mutations in Next.js, which is why I recommend it. With that said, it's worth keeping in mind they're marked as experimental and may change in the future.

#### Creating a Server Action

First, we create a file `actions.ts` from which we can export all our server actions. In this case, we will create a `createTask` server action.

The utility `withSession` is used to ensure the user is authenticated before performing the mutation.

 ```ts {% title="lib/tasks/actions.ts" %}
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

    revalidatePath('/dashboard/[organization]/tasks', 'page');
  });
```

#### Using the Server Action

We will show how to use the server action in the `forms` section.

### 2. Using SWR

If you want to use SWR, we can use the `createTask` mutation in our `useCreateTask` hook.

We will be using the `useMutation` hook from `react-query` to create our hook.

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

And now, we could use this hook within a component. Below is a very simple example:

```tsx
function Component() {
  const createTaskMutation = useCreateTaskMutation();

  return <MyForm onSubmit={task => createTaskMutation.trigger(task)} />
}
```

#### Updating a Task

Now, let's write a hook to update an existing task.

We will write another mutation function to update a task:

 ```ts {% title="lib/tasks/mutations.ts" %}
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '../../database.types';
import type Task from '~/lib/tasks/types/task';
import { TASKS_TABLE } from '~/lib/db-tables';

type Client = SupabaseClient<Database>;

export function updateTask(
  client: Client,
  task: Partial<Task> & { id: number }
) {
  return client
    .from(TASKS_TABLE)
    .update({
      name: task.name,
      done: task.done,
      description: task.description,
    })
    .match({
      id: task.id,
    })
    .throwOnError();
}
```

### Updating a task using server actions

In addition to what we have written before, add the following code:

 ```ts {% title="lib/tasks/actions.ts" %}
import { revalidatePath } from 'next/cache';
import { createTask, updateTask } from '~/lib/tasks/mutations';

type UpdateTaskParams = {
  task: Partial<Task> & Pick<Task, 'id'>;
};

export const updateTaskAction = withSession(
  async (params: UpdateTaskParams) => {
    const client = getSupabaseServerActionClient();

    await updateTask(client, params.task);

    const path = `/dashboard/[organization]/tasks`;

    // revalidate the tasks page and the task page
    revalidatePath(path, 'page');
    revalidatePath(`${path}/[task]`, 'page');
  },
);
```

### Updating a task using SWR

And we can write a hook to use this mutation:

 ```tsx {% title="lib/tasks/hooks/use-update-task.ts" %}
import useSWRMutation from 'swr/mutation';
import useSupabase from '~/core/hooks/use-supabase';
import type Task from '~/lib/tasks/types/task';
import { updateTask } from '~/lib/tasks/mutations';

type TaskPayload = Partial<Task> & { id: number };

function useUpdateTaskMutation() {
  const client = useSupabase();
  const key = ['tasks'];

  return useSWRMutation(key, async (_, { arg: task }: { arg: TaskPayload }) => {
    return updateTask(client, task);
  });
}

export default useUpdateTaskMutation;
```

#### Deleting a Task

We will write another mutation function to delete a task:

 ```ts {% title="lib/tasks/mutations.ts" %}
import type { SupabaseClient } from '@supabase/supabase-js';
import type { Database } from '../../database.types';
import { TASKS_TABLE } from '~/lib/db-tables';

type Client = SupabaseClient<Database>;

export function deleteTask(
  client: Client,
  taskId: number
) {
  return client.from(TASKS_TABLE).delete().match({
    id: taskId
  }).throwOnError();
}
```

### Deleting a task using server actions

In addition to what we have written before, add the following code:

 ```ts {% title="lib/tasks/actions.ts" %}
import { createTask, deleteTask, updateTask } from '~/lib/tasks/mutations';
import revalidatePath from 'next/cache';

type DeleteTaskParams = {
  taskId: number;
};

export const deleteTaskAction = withSession(
  async (params: DeleteTaskParams) => {
    const client = getSupabaseServerActionClient();

    await deleteTask(client, params.taskId);

    revalidatePath('/dashboard/[organization]/tasks', 'page')
  });
```

Finally, we write a mutation to delete a task:

 ```tsx {% title="lib/entities/hooks/use-delete-task.ts" %}
import useSWRMutation from 'swr/mutation';

import useSupabase from '~/core/hooks/use-supabase';
import { deleteTask } from '~/lib/tasks/mutations';

function useDeleteTaskMutation() {
  const client = useSupabase();
  const taskId = ['tasks'];

  return useSWRMutation(
    taskId,
    async (_, { arg: taskId }: { arg: number }) => {
      return deleteTask(client, taskId);
  });
}

export default useDeleteTaskMutation;
```

---


We're done with the data writing section. In the next section, we will see how to use these functions in our pages and components.

-----------------
FILE PATH: kits/next-supabase/tutorials/tailwind-css.mdoc

---
status: "published"

title: "Tailwind CSS and Styling"
label: Tailwind CSS and Styling
order: 6
description: "Learn how to configure Tailwind CSS and tweak the Makerkit's theme."
---


This SaaS template uses Tailwind CSS for styling the application.

You will likely want to tweak the brand color of your application: to do so, open the file `tailwind.config.js` and replace the `primary` color (which it's `indigo` by default):

 ```js {% title="tailwind.config.js" %}
extend: {
  colors: {
    primary: {
      ...colors.indigo,
      contrast: '#fff',
    },
    dark: colors.gray
  },
},
```

1. The `primary` color is the main color of your application. It is used for the background of the navigation bar, the background of the sidebar, the background of the buttons, etc.
2. The `dark` color is the background color of the dark theme. You can use other types of dark colors in the Tailwind CSS Color palette (such as `slate`, `zinc`), or define your own colors.

### Updating the Primary color of your app

To update the `primary` color, you can either:

1. choose another color from the Tailwind CSS Color palette (for example, try `colors.blue`)
2. define your own colors, from `50` to `900`

The `contrast` color will be helpful to define the text color of some components, so tweaking it may be required.

Once updated, the Makerkit's theme will automatically adapt to your new color palette! For example, below, we set the primary color to `colors.blue`:

{% img src="/assets/images/posts/blue-dark-theme.webp" width="2842" height="1966" /%}


### Dark Mode

Makerkit also ships with a dark theme.

The light theme is the default, but users can manually switch thanks to the component `DarkModeToggle`.

If you wish to only use one theme, you can tweak the configuration using the following flag in the configuration file `~/configuration.ts`:

 ```ts {% title="~/configuration.ts" %}
{
  ...
  features: {
    enableThemeSwitcher: false,
  },
  theme: Themes.Dark,
  ...
}
```

The default Tailwind media switcher is not supported since Makerkit uses its own system that allows users to choose the System theme (light or dark) or the theme defined by the application.

## Installing Animations (optional)

If you want to use animations in your application, you can install the `tailwindcss-animate` library:

```bash
npm i tailwindcss-animate
```

Then, open the file `tailwind.config.js` and add the library as a plugin:

 ```js {% title="tailwind.config.js" %}
plugins: [
  require('tailwindcss-animate')
]
```

## Caveats

1. Setting the dark theme by default is currently not supported without some minor tweaks. It is being developed.
2. The default Tailwind media switcher is not supported since Makerkit uses its own system that allows users to choose the System theme (light or dark) or the theme defined by the application.

-----------------
FILE PATH: kits/next-supabase/tutorials/task-detail.mdoc

---
status: "published"
title: "Building the Task Detail page | Next.js Supabase"
label: "Building the Task Detail page"
order: 12
description: "In this tutorial, we will build the Task Detail page of the Tasks App."
---

The Task Detail page is the page that shows the details of a specific task. It is a child page of the Tasks page.

As you may know, we can define a dynamic path in Next.js by using square brackets. For example, we can define a dynamic path for the Tasks page by creating a file called `app/dashboard/[organization]/tasks/[task]/page.tsx`. This will create a dynamic path for the Tasks page that will be available at `/dashboard/:organization/tasks/:task`.

## Building the Task Detail Container

Before we build the page, we define a client component named `TaskItemContainer` that will be used by the page. This component will be responsible for rendering the form that will be used to update the task.

 ```tsx {% title="app/dashboard/[organization]/tasks/components/TaskItemContainer.tsx" %}
'use client';

import { FormEventHandler, useCallback, useTransition } from 'react';
import { ChevronLeftIcon } from '@heroicons/react/24/outline';

import Textarea from '~/core/ui/Textarea';
import Label from '~/core/ui/Label';
import Button from '~/core/ui/Button';
import Heading from '~/core/ui/Heading';
import { TextFieldInput, TextFieldLabel } from '~/core/ui/TextField';

import type Task from '~/lib/tasks/types/task';
import { updateTaskAction } from '~/lib/tasks/actions';

const TaskItemContainer: React.FC<{
  task: Task;
}> = ({ task }) => {
  const [isMutating, startTransition] = useTransition();

  const onUpdate: FormEventHandler<HTMLFormElement> = useCallback(
    (e) => {
      e.preventDefault();

      const data = new FormData(e.currentTarget);
      const name = data.get('name') as string;
      const description = data.get('description') as string;

      startTransition(async () => {
        await updateTaskAction({
          task: {
            name,
            description,
            id: task.id,
          },
          csrfToken,
        });
      });
    },
    [csrfToken, task.id],
  );

  return (
    <form onSubmit={onUpdate}>
      <div className={'flex flex-col space-y-4 max-w-xl'}>
        <Heading type={2}>{task.name}</Heading>

        <TextFieldLabel>
          Name
          <TextFieldInput required defaultValue={task.name} name={'name'} />
        </TextFieldLabel>

        <Label>
          Description
          <Textarea
            className={'h-32'}
            name={'description'}
            defaultValue={task.description || ''}
          />
        </Label>

        <div className={'flex space-x-2 justify-between'}>
          <Button href={'../tasks'} color={'transparent'}>
            <span className={'flex space-x-2 items-center'}>
              <ChevronLeftIcon className={'w-4'} />
              <span>Back to Tasks</span>
            </span>
          </Button>

          <Button loading={isMutating}>Update Task</Button>
        </div>
      </div>
    </form>
  );
};

export default TaskItemContainer;
```

## Building the Task Detail page

Now that we have the `TaskItemContainer` component, we can build the Task Detail page. The page will be responsible for fetching the task data and passing it to the `TaskItemContainer` component.

First, we define the function `loadTaskData` - responsible for fetching the task data - and then we use it in the `TaskPage` component.

If the task does not exist, we redirect the user back to the dashboard page. This is done by using the `redirect` function from the `next/navigation` module.

 ```tsx {% title="app/dashboard/[organization]/tasks/[task]/page.tsx" %}
async function loadTaskData(taskId: string) {
  const client = getSupabaseServerComponentClient();
  const { data: task } = await getTask(client, Number(taskId));

  if (!task) {
    redirect('/dashboard');
  }

  return {
    task,
  };
}
```

Below is the full code for the Task Detail page.

 ```tsx {% title="app/dashboard/[organization]/tasks/[task]/page.tsx" %}
import React, { use } from 'react';
import { redirect } from 'next/navigation';

import ArrowLeftIcon from '@heroicons/react/24/outline/ArrowLeftIcon';

import Button from '~/core/ui/Button';
import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';
import { getTask } from '~/lib/tasks/queries';
import AppHeader from '~/app/dashboard/[organization]/components/AppHeader';
import AppContainer from '~/app/dashboard/[organization]/components/AppContainer';
import TaskItemContainer from '~/app/dashboard/[organization]/tasks/components/TaskItemContainer';
import { withI18n } from '~/i18n/with-i18n';

interface Context {
  params: {
    task: string;
  };
}

export const metadata = {
  title: `Task`,
};

const TaskPage = ({ params }: Context) => {
  const data = use(loadTaskData(params.task));
  const task = data.task;

  return (
    <>
      <AppHeader>
        <TaskPageHeading />
      </AppHeader>

      <AppContainer>
        <TaskItemContainer task={task} />
      </AppContainer>
    </>
  );
};

function TaskPageHeading() {
  return (
    <div className={'flex items-center space-x-6'}>
      <span>Task</span>

      <Button size={'small'} color={'transparent'} href={'../tasks'}>
        <ArrowLeftIcon className={'mr-2 h-4'} />
        Back to Tasks
      </Button>
    </div>
  );
}

async function loadTaskData(taskId: string) {
  const client = getSupabaseServerComponentClient();
  const { data: task } = await getTask(client, Number(taskId));

  if (!task) {
    redirect('/dashboard');
  }

  return {
    task,
  };
}

export default withI18n(TaskPage);
```

---


Perfect, our Tasks App is now complete! 🎉

In the next steps, we take a look at some things you should now while keeping working on your app.

-----------------
FILE PATH: kits/next-supabase/tutorials/tasks-page.mdoc

---
status: "published"

title: "Building the Tasks page"
label: "Building the Tasks page"
order: 11
description: "Let's build the page that lists all the tasks for a given organization."
---


Okay, now that we have built the schema, queries and mutations for the tasks, let's build the page that lists all the tasks for a given organization.

In this page we will build a table using the `DataTable` component in the template (built using Tanstack Table) and display the tasks data. From a dropdown, we will be able to view a task, mark it as done/undone, or delete it.

Let's get started!

---


## Fetching the Tasks data

First, let's build the foundation of our page.

As you know - pages are Server Components. This means we can fetch data directly from our database, and pass it down to the client components.

To fetch data, we define a function `loadTasksData` - don't worry about writing this down yet, as we have a full example below. What matters is understanding the concept and how to fetch data from the Supabase database.

```tsx
export async function loadTasksData(params: {
  organizationUid: string;
  pageIndex: number;
  perPage: number;
  query?: string;
}) {
  const client = getSupabaseServerComponentClient();
  const { organizationUid, perPage, pageIndex, query } = params;

  const {
    data: tasks,
    error,
    count,
  } = await getTasks(client, {
    organizationUid,
    pageIndex,
    perPage,
    query,
  });

  if (error) {
    console.error(error);

    return {
      tasks: [],
      count: 0,
    };
  }

  return {
    tasks,
    count: count ?? 0,
  };
}
```

As you can see - we defined a Supabase client, and passed it to the `getTasks` function, which is a query we defined in the previous chapter.

Now, we will define the page `TasksPage`, retrieve the data from the server component, and pass it down to the client components.

For the time being - we don't display the tasks data, but we will do that in the next section - since the Table component requires a bit of work.

 ```tsx {% title="app/dashboard/[organization]/tasks/page.tsx" %}
import { use } from 'react';

import {
  PlusCircleIcon,
  RectangleStackIcon,
} from '@heroicons/react/24/outline';

import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';
import { getTasks } from '~/lib/tasks/queries';
import Trans from '~/core/ui/Trans';
import { withI18n } from '~/i18n/with-i18n';

import AppHeader from '~/app/dashboard/[organization]/components/AppHeader';
import AppContainer from '~/app/dashboard/[organization]/components/AppContainer';

export const metadata = {
  title: 'Tasks',
};

interface TasksPageParams {
  params: {
    organization: string;
  };

  searchParams: {
    page?: string;
    query?: string;
  };
}

function TasksPage({ params, searchParams }: TasksPageParams) {
  const pageIndex = Number(searchParams.page ?? '1') - 1;
  const perPage = 8;

  const { tasks, count } = use(
    loadTasksData({
      organizationUid: params.organization,
      pageIndex,
      perPage,
      query: searchParams.query || '',
    }),
  );

  const pageCount = Math.ceil(count / perPage);

  return (
    <>
      <AppHeader>
        <span className={'flex space-x-2'}>
          <RectangleStackIcon className="w-6" />

          <span>
            <Trans i18nKey={'common:tasksTabLabel'} />
          </span>
        </span>
      </AppHeader>

      <AppContainer>

      </AppContainer>
    </>
  );
}

export async function loadTasksData(params: {
  organizationUid: string;
  pageIndex: number;
  perPage: number;
  query?: string;
}) {
  const client = getSupabaseServerComponentClient();
  const { organizationUid, perPage, pageIndex, query } = params;

  const {
    data: tasks,
    error,
    count,
  } = await getTasks(client, {
    organizationUid,
    pageIndex,
    perPage,
    query,
  });

  if (error) {
    console.error(error);

    return {
      tasks: [],
      count: 0,
    };
  }

  return {
    tasks,
    count: count ?? 0,
  };
}

export default withI18n(TasksPage);
````

To complete this page, we need 3 supporting components that are defined in external files:
1. `TasksTable`: a table that displays the tasks data
2. `SearchTaskInput`: an input that allows the user to search for tasks
3. `CreateTaskModal`: a modal that allows the user to create a new task

We will define these components in the next sections.

## Building the Tasks table

Now that we have the data, let's build the table that displays it.

To define the table, we use the `DataTable` component, which is a wrapper around the Tanstack Table component. This component is a bit complex, so we will break it down into smaller pieces.

The component `TasksTable` accepts the following props:
1. `pageIndex` - the current page index
2. `pageCount` - the total number of pages
3. `tasks` - the tasks data

We will refresh the page when the user changes the page index, and we will pass the data to the `DataTable` component. As you can see we still don't have defined the `TABLE_COLUMNS` constant, but we will do that in the next section.

Since it's a client component, we define the `TasksTable` component in a separate file at `src/app/dashboard/[organization]/tasks/components/TasksTable.tsx`.

 ```tsx {% title="src/app/dashboard/[organization]/tasks/components/TasksTable.tsx" %}
function TasksTable(
  props: React.PropsWithChildren<{
    pageIndex: number;
    pageCount: number;
    tasks: Task[];
  }>,
) {
  const router = useRouter();
  const pathname = usePathname();

  return (
    <DataTable
      onPaginationChange={({ pageIndex }) => {
        router.push(`${pathname}?page=${pageIndex + 1}`);
      }}
      pageIndex={props.pageIndex}
      pageCount={props.pageCount}
      data={props.tasks}
      columns={TABLE_COLUMNS}
    />
  );
}
```

### Defining the table columns

Now, let's define the table columns. We will define the following columns:

1. `Name` - the name of the task, which is a link to the task page
2. `Description` - the description of the task, which is truncated to 50 characters
3. `Due Date` - the due date of the task, which is formatted using the `date-fns` library
4. `Actions` - a dropdown menu with the following actions:
 1. `View Task` - a link to the task page
 2. `Mark as Done` - a button that marks the task as done
 3. `Delete Task` - a button that deletes the task

Let's define the `TABLE_COLUMNS` constant. As you can see, we use the `ColumnDef` type from the Tanstack Table library. We define the `header` and `cell` properties, which are the header and cell components of the table.

```tsx
const TABLE_COLUMNS: ColumnDef<Task>[] = [
  {
    header: 'Name',
    cell: ({ row }) => {
      const task = row.original;

      return (
        <Link className={'hover:underline'} href={'tasks/' + task.id}>
          {task.name}
        </Link>
      );
    },
  },
  {
    header: 'Description',
    id: 'description',
    cell: ({ row }) => {
      const task = row.original;
      const length = task.description?.length ?? 0;

      return (
        <span className={'truncate max-w-[50px]'}>
          {length > 0 ? task.description : '-'}
        </span>
      );
    },
  },
  {
    header: 'Due Date',
    id: 'dueDate',
    cell: ({ row }) => {
      const task = row.original;

      const dueDate = formatDistance(new Date(task.dueDate), new Date(), {
        addSuffix: true,
      });

      return (
        <If
          condition={task.done}
          fallback={<span className={'capitalize'}>{dueDate}</span>}
        >
          <div className={'inline-flex'}>
            <Badge size={'small'} color={'success'}>
              Done
            </Badge>
          </div>
        </If>
      );
    },
  },
  {
    header: '',
    id: 'actions',
    cell: ({ row }) => {
      const task = row.original;

      return (
        <div className={'flex justify-end'}>
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <IconButton>
                <EllipsisVerticalIcon className="w-5" />
              </IconButton>
            </DropdownMenuTrigger>

            <DropdownMenuContent
              collisionPadding={{
                right: 20,
              }}
            >
              <DropdownMenuItem>
                <Link href={'tasks/' + row.original.id}>View Task</Link>
              </DropdownMenuItem>

              <UpdateStatusMenuItem task={task} />
              <DeleteTaskMenuItem task={task} />
            </DropdownMenuContent>
          </DropdownMenu>
        </div>
      );
    },
  },
];
```

### Defining the dropdown menu items

Now, let's define the dropdown menu items. Some of them are more complex, so we defined them into smaller components.

```tsx

function DeleteTaskMenuItem({ task }: { task: Task }) {
  const [, startTransition] = useTransition();
  const csrfToken = useCsrfToken();

  return (
    <DropdownMenuItem onSelect={(e) => e.preventDefault()}>
      <ConfirmDeleteTaskModal
        task={task.name}
        onConfirm={() => {
          startTransition(async () => {
            await deleteTaskAction({ taskId: task.id, csrfToken });
          });
        }}
      >
        <span className={'text-red-500'}>Delete Task</span>
      </ConfirmDeleteTaskModal>
    </DropdownMenuItem>
  );
}

function UpdateStatusMenuItem({
  task,
}: React.PropsWithChildren<{
  task: Task;
}>) {
  const [isPending, startTransition] = useTransition();
  const action = task.done ? 'Mark as Todo' : 'Mark as Done';

  return (
    <DropdownMenuItem
      disabled={isPending}
      onSelect={(e) => e.preventDefault()}
      onClick={() => {
        startTransition(async () => {
          await updateTaskAction({
            task: {
              id: task.id,
              done: !task.done,
            },
          });
        });
      }}
    >
      {action}
    </DropdownMenuItem>
  );
}
```

### Defining the Confirm Delete Task modal

Finally, we need to define the Modal component to confirm the deletion of a task. This component is defined in `src/core/ui/Modal.tsx`.

```tsx
function ConfirmDeleteTaskModal({
  children,
  onConfirm,
  task,
}: React.PropsWithChildren<{
  task: string;
  onConfirm: () => void;
}>) {
  return (
    <Modal heading={`Deleting Task`} Trigger={children}>
      <div className={'flex flex-col space-y-4'}>
        <div className={'text-sm flex flex-col space-y-2'}>
          <p>
            You are about to delete the task <b>{task}</b>
          </p>

          <p>Do you want to continue?</p>
        </div>

        <div className={'flex justify-end space-x-2'}>
          <Button variant={'flat'} color={'danger'} onClick={onConfirm}>
            Yep, delete task
          </Button>
        </div>
      </div>
    </Modal>
  );
}
```

### Full code for the Tasks table

Perfect - below is the full code for the Tasks table. We will now import it into the page, and display the data.

```tsx
'use client';

import Link from 'next/link';
import { usePathname, useRouter } from 'next/navigation';
import { useTransition } from 'react';
import { formatDistance } from 'date-fns';

import { ColumnDef } from '@tanstack/react-table';
import { EllipsisVerticalIcon } from '@heroicons/react/24/outline';

import Task from '~/lib/tasks/types/task';

import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '~/core/ui/Dropdown';

import DataTable from '~/core/ui/DataTable';
import IconButton from '~/core/ui/IconButton';
import Badge from '~/core/ui/Badge';
import If from '~/core/ui/If';

import { deleteTaskAction, updateTaskAction } from '~/lib/tasks/actions';
import useCsrfToken from '~/core/hooks/use-csrf-token';

const TABLE_COLUMNS: ColumnDef<Task>[] = [
  {
    header: 'Name',
    cell: ({ row }) => {
      const task = row.original;

      return (
        <Link className={'hover:underline'} href={'tasks/' + task.id}>
          {task.name}
        </Link>
      );
    },
  },
  {
    header: 'Description',
    id: 'description',
    cell: ({ row }) => {
      const task = row.original;
      const length = task.description?.length ?? 0;

      return (
        <span className={'truncate max-w-[50px]'}>
          {length > 0 ? task.description : '-'}
        </span>
      );
    },
  },
  {
    header: 'Due Date',
    id: 'dueDate',
    cell: ({ row }) => {
      const task = row.original;

      const dueDate = formatDistance(new Date(task.dueDate), new Date(), {
        addSuffix: true,
      });

      return (
        <If
          condition={task.done}
          fallback={<span className={'capitalize'}>{dueDate}</span>}
        >
          <div className={'inline-flex'}>
            <Badge size={'small'} color={'success'}>
              Done
            </Badge>
          </div>
        </If>
      );
    },
  },
  {
    header: '',
    id: 'actions',
    cell: ({ row }) => {
      const task = row.original;

      return (
        <div className={'flex justify-end'}>
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <IconButton>
                <EllipsisVerticalIcon className="w-5" />
              </IconButton>
            </DropdownMenuTrigger>

            <DropdownMenuContent
              collisionPadding={{
                right: 20,
              }}
            >
              <DropdownMenuItem>
                <Link href={'tasks/' + row.original.id}>View Task</Link>
              </DropdownMenuItem>

              <UpdateStatusMenuItem task={task} />
              <DeleteTaskMenuItem task={task} />
            </DropdownMenuContent>
          </DropdownMenu>
        </div>
      );
    },
  },
];

function TasksTable(
  props: React.PropsWithChildren<{
    pageIndex: number;
    pageCount: number;
    tasks: Task[];
  }>,
) {
  const router = useRouter();
  const pathname = usePathname();

  return (
    <DataTable
      onPaginationChange={({ pageIndex }) => {
        router.push(`${pathname}?page=${pageIndex + 1}`);
      }}
      pageIndex={props.pageIndex}
      pageCount={props.pageCount}
      data={props.tasks}
      columns={TABLE_COLUMNS}
    />
  );
}

function DeleteTaskMenuItem({ task }: { task: Task }) {
  const [, startTransition] = useTransition();
  const csrfToken = useCsrfToken();

  return (
    <DropdownMenuItem onSelect={(e) => e.preventDefault()}>
      <ConfirmDeleteTaskModal
        task={task.name}
        onConfirm={() => {
          startTransition(async () => {
            await deleteTaskAction({ taskId: task.id, csrfToken });
          });
        }}
      >
        <span className={'text-red-500'}>Delete Task</span>
      </ConfirmDeleteTaskModal>
    </DropdownMenuItem>
  );
}

function UpdateStatusMenuItem({
  task,
}: React.PropsWithChildren<{
  task: Task;
}>) {
  const [isPending, startTransition] = useTransition();
  const csrfToken = useCsrfToken();
  const action = task.done ? 'Mark as Todo' : 'Mark as Done';

  return (
    <DropdownMenuItem
      disabled={isPending}
      onSelect={(e) => e.preventDefault()}
      onClick={() => {
        startTransition(async () => {
          await updateTaskAction({
            task: {
              id: task.id,
              done: !task.done,
            },
          });
        });
      }}
    >
      {action}
    </DropdownMenuItem>
  );
}

function ConfirmDeleteTaskModal({
  children,
  onConfirm,
  task,
}: React.PropsWithChildren<{
  task: string;
  onConfirm: () => void;
}>) {
  return (
    <Modal heading={`Deleting Task`} Trigger={children}>
      <div className={'flex flex-col space-y-4'}>
        <div className={'text-sm flex flex-col space-y-2'}>
          <p>
            You are about to delete the task <b>{task}</b>
          </p>

          <p>Do you want to continue?</p>
        </div>

        <div className={'flex justify-end space-x-2'}>
          <Button variant={'flat'} color={'danger'} onClick={onConfirm}>
            Yep, delete task
          </Button>
        </div>
      </div>
    </Modal>
  );
}

export default TasksTable;
```

## Building the Search input

Now, let's build the search input. This input will allow the user to search for tasks by name.

We will use a text input to refresh the current route using a query parameter `query`:

```tsx src/app/dashboard/[organization]/tasks/components/SearchTaskInput.tsx
'use client';

import { usePathname, useRouter } from 'next/navigation';
import { FormEventHandler, useCallback } from 'react';
import { TextFieldInput } from '~/core/ui/TextField';

function SearchTaskInput({
  query,
}: React.PropsWithChildren<{
  query: Maybe<string>;
}>) {
  const router = useRouter();
  const pathName = usePathname();

  const onSubmit: FormEventHandler<HTMLFormElement> = useCallback(
    (event) => {
      event.preventDefault();

      const formData = new FormData(event.currentTarget);
      const query = formData.get('query') as string;
      const url = new URL(pathName, window.location.origin);

      url.searchParams.set('query', query);

      const path = url.pathname + url.search;

      router.push(path);
    },
    [pathName, router],
  );

  return (
    <form className={'w-full max-w-sm'} onSubmit={onSubmit}>
      <TextFieldInput
        defaultValue={query}
        name={'query'}
        className={'w-full'}
        placeholder={'Search for task...'}
      />
    </form>
  );
}

export default SearchTaskInput;
```

## Building the Create Task Form

To create a task - we will use a simple form that will be displayed in a modal - which we will define later.

 ```tsx {% title="src/app/dashboard/[organization]/tasks/components/TaskForm.tsx" %}
'use client';

import type { FormEventHandler } from 'react';
import { useCallback, useTransition } from 'react';
import { toast } from 'sonner';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';
import If from '~/core/ui/If';
import Label from '~/core/ui/Label';
import Textarea from '~/core/ui/Textarea';

import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';
import { createTaskAction } from '~/lib/tasks/actions';
import useCsrfToken from '~/core/hooks/use-csrf-token';

const TaskForm: React.FC = () => {
  const [isMutating, startTransition] = useTransition();
  const organization = useCurrentOrganization();
  const organizationId = organization?.id as number;
  const csrfToken = useCsrfToken();

  const onCreateTask: FormEventHandler<HTMLFormElement> = useCallback(
    async (event) => {
      event.preventDefault();

      const target = event.currentTarget;
      const data = new FormData(target);
      const name = data.get('name') as string;
      const description = data.get('description') as string;
      const dueDate = (data.get('dueDate') as string) || getDefaultDueDate();

      if (name.trim().length < 3) {
        toast.error('Task name must be at least 3 characters long');

        return;
      }

      const task = {
        organizationId,
        name,
        dueDate,
        description,
        done: false,
      };

      startTransition(async () => {
        await createTaskAction({ task, csrfToken });
      });
    },
    [csrfToken, organizationId],
  );

  return (
    <form className={'flex flex-col'} onSubmit={onCreateTask}>
      <div className={'flex flex-col space-y-4 w-full'}>
        <TextField.Label>
          Name
          <TextField.Input
            required
            name={'name'}
            placeholder={'ex. Launch on IndieHackers'}
          />
        </TextField.Label>

        <Label>
          Description
          <Textarea
            name={'description'}
            className={'h-32'}
            placeholder={'Describe the task...'}
          />
        </Label>

        <TextField.Label>
          Due date
          <TextField.Input name={'dueDate'} type={'date'} />
          <TextField.Hint>
            Leave empty to set the due date to tomorrow
          </TextField.Hint>
        </TextField.Label>

        <div className={'flex justify-end'}>
          <Button variant={'flat'} loading={isMutating}>
            <If condition={isMutating} fallback={<>Create Task</>}>
              Creating Task...
            </If>
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

export default TaskForm;
```

## Building the Create Task modal

We will leverage the `Modal` component to display the form in a modal. This component is defined in `src/core/ui/Modal.tsx`.

 ```tsx {% title="src/app/dashboard/[organization]/tasks/components/CreateTaskModal.tsx" %}
import Modal from '~/core/ui/Modal';
import TaskForm from './TaskForm';

function CreateTaskModal(props: React.PropsWithChildren) {
  return (
    <Modal heading={`Create Task`} Trigger={props.children}>
      <TaskForm />
    </Modal>
  );
}

export default CreateTaskModal;
```

## Adding the Tasks table to the page

Now that we have the table, let's add it to the page. We will define a further component named `TasksTableContainer` which will wrap the table, and add a search input and a button to create a new task.

 ```tsx {% title="app/dashboard/[organization]/tasks/page.tsx" %}
function TasksTableContainer({
  tasks,
  pageCount,
  pageIndex,
  query,
}: React.PropsWithChildren<{
  tasks: Task[];
  pageCount: number;
  pageIndex: number;
  query?: string;
}>) {
  return (
    <div className={'flex flex-col space-y-4'}>
      <div className={'flex space-x-4 justify-between items-center'}>
        <div className={'flex'}>
          <CreateTaskModal>
            <Button color={'transparent'}>
              <span className={'flex space-x-2 items-center'}>
                <PlusCircleIcon className={'w-4'} />

                <span>New Task</span>
              </span>
            </Button>
          </CreateTaskModal>
        </div>

        <SearchTaskInput query={query} />
      </div>

      <TasksTable pageIndex={pageIndex} pageCount={pageCount} tasks={tasks} />
    </div>
  );
}

function TasksEmptyState() {
  return (
    <div className={'flex flex-col space-y-8 p-4'}>
      <div className={'flex flex-col'}>
        <Heading type={2}>
          <span className={'font-semibold'}>
            Hey, it looks like you don&apos;t have any tasks yet... 🤔
          </span>
        </Heading>

        <Heading type={4}>
          Create your first task by clicking on the button below
        </Heading>
      </div>
    </div>
  );
}
```

Finally, we can add these components to the page by adding them as children of the `<AppContainer>` component.

 ```tsx {% title="app/dashboard/[organization]/tasks/page.tsx" %}
<AppContainer>
  <If condition={!count}>
    <TasksEmptyState />
  </If>

  <TasksTableContainer
    pageIndex={pageIndex}
    pageCount={pageCount}
    tasks={tasks}
    query={searchParams.query}
  />
</AppContainer>
```

Check the full source code below.

### Full Source Code for the Tasks page

Perfect - below is the full source code for the Tasks page. We will now add it to the dashboard.

 ```tsx {% title="app/dashboard/[organization]/tasks/page.tsx" %}
import { use } from 'react';

import {
  PlusCircleIcon,
  RectangleStackIcon,
} from '@heroicons/react/24/outline';

import getSupabaseServerComponentClient from '~/core/supabase/server-component-client';
import { getTasks } from '~/lib/tasks/queries';
import Trans from '~/core/ui/Trans';
import { withI18n } from '~/i18n/with-i18n';

import AppHeader from '~/app/dashboard/[organization]/components/AppHeader';
import AppContainer from '~/app/dashboard/[organization]/components/AppContainer';

import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';
import If from '~/core/ui/If';

import type Task from '~/lib/tasks/types/task';
import TasksTable from '~/app/dashboard/[organization]/tasks/components/TasksTable';
import SearchTaskInput from '~/app/dashboard/[organization]/tasks/components/SearchTaskInput';
import CreateTaskModal from '~/app/dashboard/[organization]/tasks/components/CreateTaskModal';

export const metadata = {
  title: 'Tasks',
};

interface TasksPageParams {
  params: {
    organization: string;
  };

  searchParams: {
    page?: string;
    query?: string;
  };
}

function TasksPage({ params, searchParams }: TasksPageParams) {
  const pageIndex = Number(searchParams.page ?? '1') - 1;
  const perPage = 8;

  const { tasks, count } = use(
    loadTasksData({
      organizationUid: params.organization,
      pageIndex,
      perPage,
      query: searchParams.query || '',
    }),
  );

  const pageCount = Math.ceil(count / perPage);

  return (
    <>
      <AppHeader>
        <span className={'flex space-x-2'}>
          <RectangleStackIcon className="w-6" />

          <span>
            <Trans i18nKey={'common:tasksTabLabel'} />
          </span>
        </span>
      </AppHeader>

      <AppContainer>
        <If condition={!count}>
          <TasksEmptyState />
        </If>

        <TasksTableContainer
          pageIndex={pageIndex}
          pageCount={pageCount}
          tasks={tasks}
          query={searchParams.query}
        />
      </AppContainer>
    </>
  );
}

export async function loadTasksData(params: {
  organizationUid: string;
  pageIndex: number;
  perPage: number;
  query?: string;
}) {
  const client = getSupabaseServerComponentClient();
  const { organizationUid, perPage, pageIndex, query } = params;

  const {
    data: tasks,
    error,
    count,
  } = await getTasks(client, {
    organizationUid,
    pageIndex,
    perPage,
    query,
  });

  if (error) {
    console.error(error);

    return {
      tasks: [],
      count: 0,
    };
  }

  return {
    tasks,
    count: count ?? 0,
  };
}

export default withI18n(TasksPage);

function TasksTableContainer({
  tasks,
  pageCount,
  pageIndex,
  query,
}: React.PropsWithChildren<{
  tasks: Task[];
  pageCount: number;
  pageIndex: number;
  query?: string;
}>) {
  return (
    <div className={'flex flex-col space-y-4'}>
      <div className={'flex space-x-4 justify-between items-center'}>
        <div className={'flex'}>
          <CreateTaskModal>
            <Button color={'transparent'}>
              <span className={'flex space-x-2 items-center'}>
                <PlusCircleIcon className={'w-4'} />

                <span>New Task</span>
              </span>
            </Button>
          </CreateTaskModal>
        </div>

        <SearchTaskInput query={query} />
      </div>

      <TasksTable pageIndex={pageIndex} pageCount={pageCount} tasks={tasks} />
    </div>
  );
}

function TasksEmptyState() {
  return (
    <div className={'flex flex-col space-y-8 p-4'}>
      <div className={'flex flex-col'}>
        <Heading type={2}>
          <span className={'font-semibold'}>
            Hey, it looks like you don&apos;t have any tasks yet... 🤔
          </span>
        </Heading>

        <Heading type={4}>
          Create your first task by clicking on the button below
        </Heading>
      </div>
    </div>
  );
}
```

