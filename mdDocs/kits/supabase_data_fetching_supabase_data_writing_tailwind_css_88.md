-----------------
FILE PATH: kits\remix-supabase\tutorials\supabase-data-fetching.mdoc

---
status: "published"

title: "Supabase: Data Fetching"
label: "Supabase: Data Fetching"
order: 10
description: "Learn how to fetch data from Supabase in your Remix applications."
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
import { TASKS_TABLE } from '~/lib/db-tables';
import type Task from '~/lib/tasks/types/task';
import type { Database } from '../../database.types';

type Client = SupabaseClient<Database>;

const TASKS_PAGE_SIZE = 10;

export function getTasks(
  client: Client,
  params: {
    organizationId: number;
    pageIndex: number;
    perPage?: number;
    query?: string;
  },
) {
  const { organizationId, perPage, pageIndex } = params;
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
      description
    `,
      {
        count: 'exact',
      },
    )
    .order('name', { ascending: true })
    .eq('organization_id', organizationId)
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
FILE PATH: kits\remix-supabase\tutorials\supabase-data-writing.mdoc

---
status: "published"
title: "Supabase: Data Writing in Remix Supabase"
label: "Supabase: Data Writing"
order: 11
description: "Learn how to write data to the Supabase Database in your Remix Supabase React applications."
---

In the section above, we saw how to fetch data from the Supabase Postgres.
Now, let's see how to write data to the Supabase database.

#### Creating a Task

First, we want to add a file named `mutations.ts` at `lib/tasks/`. Here, we
will add all the mutations that we will need to create, update, and delete
tasks.

```ts
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

#### Updating a Task

Now, let's write a hook to update an existing task.

We will write another mutation function to update a task:

 ```ts {% title="lib/tasks/mutations.ts" %}
export function updateTask(
  client: Client,
  task: Partial<Task> & { id: number }
) {
  return client
    .from(TASKS_TABLE)
    .update({
      name: task.name,
      done: task.done,
    })
    .match({
      id: task.id,
    })
    .throwOnError();
}
```

#### Deleting a Task

We will write another mutation function to delete a task:

 ```ts {% title="lib/tasks/mutations.ts" %}
export function deleteTask(
  client: Client,
  taskId: number
) {
  return client.from(TASKS_TABLE).delete().match({
    id: taskId
  }).throwOnError();
}
```



-----------------
FILE PATH: kits\remix-supabase\tutorials\tailwind-css.mdoc

---
status: "published"
title: "Tailwind CSS and Styling | Remix Supabase SaaS Starter Kit"
label: Tailwind CSS and Styling
order: 6
description: "Learn how to configure Tailwind CSS and tweak the Remix Supabase SaaS Kit theme."
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
FILE PATH: kits\remix-supabase\tutorials\task-detail.mdoc

---
status: "published"
title: "Building the Task Detail page | Remix Supabase"
label: "Building the Task Detail page"
order: 13
description: "Learn how to build the Task Detail page in the Tasks App of the Remix Supabase project."
---

The Task Detail page is the page that shows the details of a specific task. It is a child page of the Tasks page.

As you may know, we can define a dynamic path in Remix using the convention `$param` denoted by the `$1` prefix. For example, we can define a dynamic path for the Tasks page by creating a file called `_app.tasks.$task.tsx`. This will create a dynamic path for the Tasks page that will be available at `/tasks/:task`.

## Building the Loader

First - we need to define the loader function for the Task Detail page. The loader function is responsible for fetching the data that will be used by the page.

In this case - we fetch the task data by using the `getTask` query. This query is defined in the `~/lib/tasks/queries` module.

As you can see below, we're able to access the `task` param by using the `args.params.task` property. This is possible because we defined the dynamic path as `_app.tasks.$task.tsx`.

```tsx
export const meta: MetaFunction = ({ data }) => {
  return [
    {
      title: data.task.name,
    },
  ];
};

export async function loader(args: LoaderArgs) {
  const client = getSupabaseServerClient(args.request);
  const taskId = args.params.task;

  const { data: task, error } = await getTask(client, Number(taskId));

  if (!task || error) {
    return throwNotFoundException();
  }

  return json({
    task,
  });
}
```

Don't worry about copying the code above, we will show the full code for the Task Detail page later in this step.

## Building the Action

From this page - we also allow users to update the task data. To do that, we define an action function that will be responsible for updating the task data.

```tsx
export async function action(args: ActionArgs) {
  const client = getSupabaseServerClient(args.request);
  const body = await args.request.json();

  switch (args.request.method) {
    case 'PUT':
      await updateTask(client, body);

      return json({ success: true });
  }

  return throwNotFoundException();
}
```

## Building the Task Detail Container

Before we build the page, we define a client component named `TaskItemContainer` that will be used by the page.

This component will be responsible for rendering the form that will be used to update the task - and also for handling the form submission using the `useFetcher` hook.

The `useFetcher` hook is a hook that is provided by Remix. When called, it will submit the `action` function defined above.

 ```tsx {% title="app/dashboard/[organization]/tasks/components/TaskItemContainer.tsx" %}
function TaskItemContainer({
  task,
}: React.PropsWithChildren<{
  task: Task;
}>) {
  const fetcher = useFetcher();
  const isSubmitting = fetcher.state === 'submitting';

  const onUpdate: FormEventHandler<HTMLFormElement> = useCallback(
    (e) => {
      e.preventDefault();

      const data = new FormData(e.currentTarget);
      const name = data.get('name') as string;
      const description = data.get('description') as string;

      fetcher.submit(
        {
          name,
          description,
        },
        {
          method: 'PUT',
          encType: 'application/json',
        },
      );
    },
    [fetcher],
  );

  return (
    <form onSubmit={onUpdate}>
      <div className={'flex flex-col space-y-4 max-w-xl'}>
        <Heading type={2}>{task.name}</Heading>

        <TextField.Label>
          Name
          <TextField.Input required name={'name'} defaultValue={task.name} />
        </TextField.Label>

        <Label>
          Description
          <Textarea
            className={'h-32'}
            name={'description'}
            defaultValue={task.description}
          />
        </Label>

        <div className={'flex space-x-2 justify-between'}>
          <Button href={'../tasks'} color={'transparent'}>
            <span className={'flex space-x-2 items-center'}>
              <ChevronLeftIcon className={'w-4'} />
              <span>Back to Tasks</span>
            </span>
          </Button>

          <Button loading={isSubmitting}>Update Task</Button>
        </div>
      </div>
    </form>
  );
}
```

## Building the Task Detail page

Now that we have the `TaskItemContainer` component, we can build the Task Detail page.

Below is the full code for the Task Detail page.

 ```tsx {% title="app/dashboard/[organization]/tasks/[task]/page.tsx" %}
import type { MetaFunction } from '@remix-run/react';
import { useFetcher, useLoaderData } from '@remix-run/react';
import type { ActionArgs } from '@remix-run/node';
import { json } from '@remix-run/node';
import type { LoaderArgs } from '@remix-run/server-runtime';
import type { FormEventHandler } from 'react';
import React, { useCallback } from 'react';

import { ChevronLeftIcon, ArrowLeftIcon } from '@heroicons/react/24/outline';

import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';
import { throwNotFoundException } from '~/core/http-exceptions';
import getSupabaseServerClient from '~/core/supabase/server-client';

import AppHeader from '~/components/AppHeader';
import AppContainer from '~/components/AppContainer';

import { getTask } from '~/lib/tasks/queries';
import type Task from '~/lib/tasks/types/task';
import Label from '~/core/ui/Label';
import Textarea from '~/core/ui/Textarea';
import TextField from '~/core/ui/TextField';
import { updateTask } from '~/lib/tasks/mutations';

export const meta: MetaFunction = ({ data }) => {
  return [
    {
      title: data.task.name,
    },
  ];
};

export async function loader(args: LoaderArgs) {
  const client = getSupabaseServerClient(args.request);
  const taskId = args.params.task;

  const { data: task, error } = await getTask(client, Number(taskId));

  if (!task || error) {
    return throwNotFoundException();
  }

  return json({
    task,
  });
}

const TaskPage = () => {
  const data = useLoaderData<typeof loader>();
  const task = data.task as Task;

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
      <Heading type={4}>
        <span>Task</span>
      </Heading>

      <Button size={'small'} color={'transparent'} href={'/tasks'}>
        <ArrowLeftIcon className={'mr-2 h-4'} />
        Back to Tasks
      </Button>
    </div>
  );
}

export default TaskPage;

function TaskItemContainer({
  task,
}: React.PropsWithChildren<{
  task: Task;
}>) {
  const fetcher = useFetcher();
  const isSubmitting = fetcher.state === 'submitting';

  const onUpdate: FormEventHandler<HTMLFormElement> = useCallback(
    (e) => {
      e.preventDefault();

      const data = new FormData(e.currentTarget);
      const name = data.get('name') as string;
      const description = data.get('description') as string;

      fetcher.submit(
        {
          name,
          description,
        },
        {
          method: 'PUT',
          encType: 'application/json',
        },
      );
    },
    [fetcher],
  );

  return (
    <form onSubmit={onUpdate}>
      <div className={'flex flex-col space-y-4 max-w-xl'}>
        <Heading type={2}>{task.name}</Heading>

        <TextField.Label>
          Name
          <TextField.Input required name={'name'} defaultValue={task.name} />
        </TextField.Label>

        <Label>
          Description
          <Textarea
            className={'h-32'}
            name={'description'}
            defaultValue={task.description}
          />
        </Label>

        <div className={'flex space-x-2 justify-between'}>
          <Button href={'../tasks'} color={'transparent'}>
            <span className={'flex space-x-2 items-center'}>
              <ChevronLeftIcon className={'w-4'} />
              <span>Back to Tasks</span>
            </span>
          </Button>

          <Button loading={isSubmitting}>Update Task</Button>
        </div>
      </div>
    </form>
  );
}

export async function action(args: ActionArgs) {
  const client = getSupabaseServerClient(args.request);
  const body = await args.request.json();

  switch (args.request.method) {
    case 'PUT':
      await updateTask(client, body);

      return json({ success: true });
  }

  return throwNotFoundException();
}

```

---


Perfect, our Tasks App is now complete! 🎉

In the next steps, we take a look at some things you should now while keeping working on your app.

-----------------
FILE PATH: kits\remix-supabase\tutorials\tasks-page.mdoc

---
status: "published"

title: "Building the Tasks page"
label: "Building the Tasks page"
order: 12
description: "Let's build the page that lists all the tasks for a given organization."
---


Okay, now that we have built the schema, queries and mutations for the tasks, let's build the page that lists all the tasks for a given organization.

In this page we will build a table using the `DataTable` component in the template (built using Tanstack Table) and display the tasks data. From a dropdown, we will be able to view a task, mark it as done/undone, or delete it.

Let's get started!

---


## Fetching the Tasks data using the Remix Loader

First, let's build the foundation of our page.

To fetch data, we define the `loader` function, the special hook that Remix uses to fetch data for the page. Don't worry about writing this down yet, as we have a full example below. What matters is understanding the concept and how to fetch data from the Supabase database.

 ```tsx {% title="app/routes/_app.tasks._index.tsx" %}
export async function loader(args: LoaderArgs) {
  const request = args.request;

  const url = new URL(request.url);
  const searchParams = url.searchParams;
  const page = searchParams.get('page') ?? '1';
  const query = searchParams.get('query') ?? '';

  const client = getSupabaseServerClient(request);
  const data = await getCurrentOrganization(client);

  const pageSize = 8;
  const pageIndex = Number(page) - 1;
  const organizationId = data?.organization.id;

  if (!organizationId) {
    return new Response('Not found', {
      status: 404,
    });
  }

  const {
    data: tasks,
    count,
    error,
  } = await getTasks(client, {
    organizationId: data.organization.id,
    pageIndex,
    query,
  });

  if (error) {
    console.error(error);

    return json({
      tasks: [],
      count: 0,
      pageCount: 0,
      query,
      pageIndex,
      pageSize,
    });
  }

  const pageCount = count ? Math.ceil(count / pageSize) : 0;

  return json({
    tasks,
    count,
    query,
    pageCount,
    pageIndex,
    pageSize,
  });
}
```

As you can see - we defined a Supabase client, and passed it to the `getTasks` function, which is a query we defined in the previous chapter.

## Defining the Remix action handler

Now, let's define the action handler. This handler will be used to create a new task.

 ```tsx {% title="app/routes/_app.tasks._index.tsx" %}
export async function action(args: ActionArgs) {
  const client = getSupabaseServerClient(args.request);
  const body = await args.request.json();

  const ok = () => json({ success: true });

  switch (args.request.method) {
    case 'POST':
      return createTask(client, body).then(ok);

    case 'DELETE':
      return deleteTask(client, body).then(ok);

    case 'PUT':
      return updateTask(client, body).then(ok);
  }

  return throwNotFoundException();
}
```

Now, we will define the page `TasksPage`.

For the time being - we don't display the tasks data, but we will do that in the next section - since the Table component requires a bit of work.

 ```tsx {% title="app/routes/_app.tasks._index.tsx" %}
import type { ActionArgs, MetaFunction } from '@remix-run/node';

import { json } from '@remix-run/node';
import { Form, useLoaderData } from '@remix-run/react';
import type { LoaderArgs } from '@remix-run/server-runtime';

import {
  PlusCircleIcon,
  RectangleStackIcon,
} from '@heroicons/react/24/outline';

import { Trans } from 'react-i18next';

import AppHeader from '~/components/AppHeader';
import AppContainer from '~/components/AppContainer';
import getSupabaseServerClient from '~/core/supabase/server-client';
import { getTasks } from '~/lib/tasks/queries';
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

import If from '~/core/ui/If';
import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';
import Modal from '~/core/ui/Modal';
import TextField from '~/core/ui/TextField';

import type Task from '~/lib/tasks/types/task';
import TasksTable from '~/components/tasks/TasksTable';
import TaskForm from '~/components/tasks/TaskForm';

import { createTask, deleteTask, updateTask } from '~/lib/tasks/mutations';
import { throwNotFoundException } from '~/core/http-exceptions';

export const meta: MetaFunction = () => {
  return [
    {
      title: 'Tasks',
    },
  ];
};

export async function loader(args: LoaderArgs) {
  const request = args.request;

  const url = new URL(request.url);
  const searchParams = url.searchParams;
  const page = searchParams.get('page') ?? '1';
  const query = searchParams.get('query') ?? '';

  const client = getSupabaseServerClient(request);
  const data = await getCurrentOrganization(client);

  const pageSize = 8;
  const pageIndex = Number(page) - 1;
  const organizationId = data?.organization.id;

  if (!organizationId) {
    return new Response('Not found', {
      status: 404,
    });
  }

  const {
    data: tasks,
    count,
    error,
  } = await getTasks(client, {
    organizationId: data.organization.id,
    pageIndex,
    query,
  });

  if (error) {
    console.error(error);

    return json({
      tasks: [],
      count: 0,
      pageCount: 0,
      query,
      pageIndex,
      pageSize,
    });
  }

  const pageCount = count ? Math.ceil(count / pageSize) : 0;

  return json({
    tasks,
    count,
    query,
    pageCount,
    pageIndex,
    pageSize,
  });
}

function TasksPage() {
  const { tasks, count, pageCount, query, pageIndex, pageSize } =
    useLoaderData<typeof loader>();

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

export default TasksPage;

function CreateTaskModal(props: React.PropsWithChildren) {
  return (
    <Modal heading={`Create Task`} Trigger={props.children}>
      <TaskForm />
    </Modal>
  );
}

export async function action(args: ActionArgs) {
  const client = getSupabaseServerClient(args.request);
  const body = await args.request.json();

  const ok = () => json({ success: true });

  switch (args.request.method) {
    case 'POST':
      return createTask(client, body).then(ok);

    case 'DELETE':
      return deleteTask(client, body).then(ok);

    case 'PUT':
      return updateTask(client, body).then(ok);
  }

  return throwNotFoundException();
}
```

To complete this page, we need 3 supporting components that are defined in external files:
1. `TasksTable`: a table that displays the tasks data
2. `TaskForm`: a form that allows the user to create a new task

We will define these components in the next sections.

## Building the Tasks table

Now that we have the data, let's build the table that displays it.

To define the table, we use the `DataTable` component, which is a wrapper around the Tanstack Table component. This component is a bit complex, so we will break it down into smaller pieces.

The component `TasksTable` accepts the following props:
1. `pageIndex` - the current page index
2. `pageCount` - the total number of pages
3. `tasks` - the tasks data

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
        <Link className={'hover:underline'} to={'/tasks/' + task.id}>
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
                <Link to={'/tasks/' + row.original.id}>View Task</Link>
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
  const fetcher = useFetcher();

  return (
    <DropdownMenuItem onSelect={(e) => e.preventDefault()}>
      <ConfirmDeleteTaskModal
        task={task.name}
        onConfirm={() => {
          fetcher.submit(task.id, {
            method: 'DELETE',
            encType: 'application/json',
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
  const fetcher = useFetcher();
  const submitting = fetcher.state === 'submitting';
  const action = task.done ? 'Mark as Todo' : 'Mark as Done';

  return (
    <DropdownMenuItem
      disabled={submitting}
      onSelect={(e) => e.preventDefault()}
      onClick={() => {
        return fetcher.submit(
          {
            id: task.id,
            done: !task.done,
          },
          {
            method: 'PUT',
            encType: 'application/json',
          },
        );
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

 ```tsx {% title="app/components/tasks/TasksTable.tsx" %}
import { Link, useFetcher, useSearchParams } from '@remix-run/react';
import { formatDistance } from 'date-fns';

import type { ColumnDef } from '@tanstack/react-table';
import { EllipsisVerticalIcon } from '@heroicons/react/24/outline';

import type Task from '~/lib/tasks/types/task';

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

import Modal from '~/core/ui/Modal';
import Button from '~/core/ui/Button';
import { useCallback } from 'react';

const TABLE_COLUMNS: ColumnDef<Task>[] = [
  {
    header: 'Name',
    cell: ({ row }) => {
      const task = row.original;

      return (
        <Link className={'hover:underline'} to={'/tasks/' + task.id}>
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
                <Link to={'tasks/' + row.original.id}>View Task</Link>
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
    pageSize: number;
    tasks: Task[];
  }>,
) {
  const [, setSearchParams] = useSearchParams();

  const onPaginationChange = useCallback(
    ({ pageIndex }: { pageIndex: number }) => {
      const page = (pageIndex + 1).toString();
      setSearchParams({ page });
    },
    [setSearchParams],
  );

  return (
    <DataTable
      onPaginationChange={onPaginationChange}
      pageIndex={props.pageIndex}
      pageCount={props.pageCount}
      pageSize={props.pageSize}
      data={props.tasks}
      columns={TABLE_COLUMNS}
    />
  );
}

function DeleteTaskMenuItem({ task }: { task: Task }) {
  const fetcher = useFetcher();

  return (
    <DropdownMenuItem onSelect={(e) => e.preventDefault()}>
      <ConfirmDeleteTaskModal
        task={task.name}
        onConfirm={() => {
          fetcher.submit(task.id, {
            method: 'DELETE',
            encType: 'application/json',
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
  const fetcher = useFetcher();
  const submitting = fetcher.state === 'submitting';
  const action = task.done ? 'Mark as Todo' : 'Mark as Done';

  return (
    <DropdownMenuItem
      disabled={submitting}
      onSelect={(e) => e.preventDefault()}
      onClick={() => {
        return fetcher.submit(
          {
            id: task.id,
            done: !task.done,
          },
          {
            method: 'PUT',
            encType: 'application/json',
          },
        );
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

## Building the Create Task Form

To create a task - we will use a simple form that will be displayed in a modal - which we will define later.

 ```tsx {% title="app/components/tasks/TaskForm.tsx" %}
import type { FormEventHandler } from 'react';
import { useCallback } from 'react';
import { toast } from 'sonner';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';

import If from '~/core/ui/If';
import Textarea from '~/core/ui/Textarea';
import Label from '~/core/ui/Label';

import useCurrentOrganization from '~/lib/organizations/hooks/use-current-organization';
import type Task from '~/lib/tasks/types/task';

function TaskForm({
  onSubmit,
  submitting,
}: React.PropsWithChildren<{
  onSubmit: (task: Omit<Task, 'id'>) => unknown;
  submitting?: boolean;
}>) {
  const organization = useCurrentOrganization();
  const organizationId = organization?.id as number;

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

      onSubmit(task);
    },
    [onSubmit, organizationId],
  );

  return (
    <form onSubmit={onCreateTask}>
      <div className={'flex flex-col space-y-4'}>
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
          <Button loading={submitting}>
            <If condition={submitting} fallback={<>Create Task</>}>
              Creating Task...
            </If>
          </Button>
        </div>
      </div>
    </form>
  );
}

function getDefaultDueDate() {
  const date = new Date();
  date.setDate(date.getDate() + 1);
  date.setHours(23, 59, 59);

  return date.toDateString();
}

export default TaskForm;
```

The form above will emit a `onSubmit` event, which we will be handled by the container component - and will submit the data to the `action` function.

## Adding the Tasks table to the page

Now that we have the table, let's add it to the page. We will define a further component named `TasksTableContainer` which will wrap the table, and add a search input and a button to create a new task.

```tsx
function TasksTableContainer({
  tasks,
  pageCount,
  pageIndex,
  pageSize,
  query,
}: React.PropsWithChildren<{
  tasks: Task[];
  pageCount: number;
  pageIndex: number;
  pageSize: number;
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

        <Form className={'w-full max-w-sm'} method={'GET'}>
          <TextField.Input
            defaultValue={query}
            name={'query'}
            className={'w-full'}
            placeholder={'Search for task...'}
          />
        </Form>
      </div>

      <TasksTable
        pageSize={pageSize}
        pageIndex={pageIndex}
        pageCount={pageCount}
        tasks={tasks}
      />
    </div>
  );
}
```

Finally, we can add these components to the page by adding them as children of the `<AppContainer>` component.

```tsx
<AppContainer>
  <If condition={!count}>
    <TasksEmptyState />
  </If>

  <TasksTableContainer
    pageSize={pageSize}
    pageIndex={pageIndex}
    pageCount={pageCount}
    tasks={tasks}
    query={query}
  />
</AppContainer>
```

Check the full source code below.

### Full Source Code for the Tasks page

Perfect - below is the full source code for the Tasks page. We will now add it to the dashboard.

 ```tsx {% title="app/routes/_app.tasks._index.tsx" %}
import type { ActionArgs, MetaFunction } from '@remix-run/node';

import { json } from '@remix-run/node';
import { Form, useFetcher, useLoaderData } from '@remix-run/react';
import type { LoaderArgs } from '@remix-run/server-runtime';

import { useCallback } from "react";

import {
  PlusCircleIcon,
  RectangleStackIcon,
} from '@heroicons/react/24/outline';

import { Trans } from 'react-i18next';

import AppHeader from '~/components/AppHeader';
import AppContainer from '~/components/AppContainer';
import getSupabaseServerClient from '~/core/supabase/server-client';
import { getTasks } from '~/lib/tasks/queries';
import getCurrentOrganization from '~/lib/server/organizations/get-current-organization';

import If from '~/core/ui/If';
import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';
import Modal from '~/core/ui/Modal';
import TextField from '~/core/ui/TextField';

import type Task from '~/lib/tasks/types/task';
import TasksTable from '~/components/tasks/TasksTable';
import TaskForm from '~/components/tasks/TaskForm';

import { createTask, deleteTask, updateTask } from '~/lib/tasks/mutations';
import { throwNotFoundException } from '~/core/http-exceptions';

export const meta: MetaFunction = () => {
  return [
    {
      title: 'Tasks',
    },
  ];
};

export async function loader(args: LoaderArgs) {
  const request = args.request;

  const url = new URL(request.url);
  const searchParams = url.searchParams;
  const page = searchParams.get('page') ?? '1';
  const query = searchParams.get('query') ?? '';

  const client = getSupabaseServerClient(request);
  const data = await getCurrentOrganization(client);

  const pageSize = 8;
  const pageIndex = Number(page) - 1;
  const organizationId = data?.organization.id;

  if (!organizationId) {
    return new Response('Not found', {
      status: 404,
    });
  }

  const {
    data: tasks,
    count,
    error,
  } = await getTasks(client, {
    organizationId: data.organization.id,
    pageIndex,
    query,
  });

  if (error) {
    console.error(error);

    return json({
      tasks: [],
      count: 0,
      pageCount: 0,
      query,
      pageIndex,
      pageSize,
    });
  }

  const pageCount = count ? Math.ceil(count / pageSize) : 0;

  return json({
    tasks,
    count,
    query,
    pageCount,
    pageIndex,
    pageSize,
  });
}

function TasksPage() {
  const { tasks, count, pageCount, query, pageIndex, pageSize } =
    useLoaderData<typeof loader>();

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
          pageSize={pageSize}
          pageIndex={pageIndex}
          pageCount={pageCount}
          tasks={tasks}
          query={query}
        />
      </AppContainer>
    </>
  );
}

export default TasksPage;

function TasksTableContainer({
  tasks,
  pageCount,
  pageIndex,
  pageSize,
  query,
}: React.PropsWithChildren<{
  tasks: Task[];
  pageCount: number;
  pageIndex: number;
  pageSize: number;
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

        <Form className={'w-full max-w-sm'} method={'GET'}>
          <TextField.Input
            defaultValue={query}
            name={'query'}
            className={'w-full'}
            placeholder={'Search for task...'}
          />
        </Form>
      </div>

      <TasksTable
        pageSize={pageSize}
        pageIndex={pageIndex}
        pageCount={pageCount}
        tasks={tasks}
      />
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

function CreateTaskModal(props: React.PropsWithChildren) {
  const fetcher = useFetcher();

  const submitting = fetcher.state === 'submitting';

  const onSubmit = useCallback(
    (task: Omit<Task, 'id'>) => {
      fetcher.submit(task, {
        method: 'POST',
        encType: 'application/json',
      });
    },
    [fetcher],
  );

  return (
    <Modal heading={`Create Task`} Trigger={props.children}>
      <TaskForm onSubmit={onSubmit} submitting={submitting} />
    </Modal>
  );
}

export async function action(args: ActionArgs) {
  const client = getSupabaseServerClient(args.request);
  const body = await args.request.json();

  const ok = () => json({ success: true });

  switch (args.request.method) {
    case 'POST':
      return createTask(client, body).then(ok);

    case 'DELETE':
      return deleteTask(client, body).then(ok);

    case 'PUT':
      return updateTask(client, body).then(ok);
  }

  return throwNotFoundException();
}
```

