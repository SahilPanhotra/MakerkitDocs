-----------------
FILE PATH: kits/next-fire/tutorials/deploying-production.mdoc

---
status: "published"
title: "Deploying to Production | Next.js Firebase"
label: "Deploying to Production"
order: 19
description: "Learn how to deploy your Next.js Firebase app to your hosting provider."
---

Before deploying to production or any remote server, you need to [follow all the steps outlined in the documentation](/docs/next-fire/going-to-production).

**Much of the work below needs to be done externally**, not in the Makerkit codebase.

Generally, you will need to:

1. **Firebase**: Create a new Firebase project (database, storage bucket, getting the private keys)
2. **Security Rules**: Update the remote security rules (both Firestore and Storage)
3. **Environment Variables**: Ensure that your environment variables are set correctly, and that you've added the relevant environment variables to your hosting provider
4. **Auth**: Enable the authentication providers you want to use from the Firebase Console
5. **Indexes**: Update all the Firestore indexes required by your application in the Firebase Console
6. **SMTP**: Set up an SMTP server to send emails by adding the required configuration to `src/configuration.ts`
7. **Payments**: Set up your Stripe or Lemon Squeezy accounts and add the relevant environment variables
8. **Deployment**: Finally, deploy your application to your hosting provider (Vercel, Firebase, Netlify, etc.): please follow the instructions provided by your hosting provider to deploy a Next.js application.

-----------------
FILE PATH: kits/next-fire/tutorials/development-getting-started.mdoc

---
status: "published"

title: "Development: adding custom features"
label: "Development: adding custom features"
order: 8
description: "Learn how to get started developing new features in your app"
---

It's time to work on our application's value proorder: adding and tracking tasks! This is likely the most exciting part for you because **it's where you get to change things and add your SaaS features to the template**.

### Page Structure

Before getting started, let's take a look at the default page structure of the boilerplate is the following.

```txt
├── pages
  └── api
    └── onboarding
    └── organizations
    └── stripe
    └── session
    └── user

  └── auth
    └── invite
      └── [code].tsx

    └── link.tsx
    └── password-reset.tsx
    └── sign-in.tsx
    └── sign-up.tsx

  └── dashboard
    └── page.tsx

  └── onboarding
    └── page.tsx

  └── settings
    └── organization
      └── members
        └── page.tsx
        └── invite.tsx

    └── profile
      └── page.tsx
      └── email.tsx
      └── password.tsx
      └── authentication.tsx
    └── subscription

  └── blog
    └── [collection]
      └── [...slug].tsx

  └── docs
    └── [page]
      └── slug.tsx
    └── page.tsx

  └── 500.tsx
  └── 404.tsx
  └── _document.tsx
  └── _app.tsx
  └── page.tsx
  └── faq.tsx
  └── pricing.tsx
```

### Setting the application's Home Page

By default, the application's home page is `/dashboard`; every time the user logs in, they're redirected to the page `src/pages/dashboard/page.tsx`.

You can update the above by setting the application's home page path at `configuration.paths.appHome`. The configuration file can be found at `src/configuration.ts`.

Unless you want to change the application's home page, **you can skip this section**, but it's good to know.

### Demo: Tasks Application

Our demo application will be a simple tasks management application. We will be able to create tasks, list them, mark them as completed, and delete them.

Here is a quick demo of what we will build:

{% video src="/assets/images/posts/tasks-demo.mp4" /%}

### Routing: Adding the Tasks Pages

Ok, so we want to add three pages to our application:

1. **Tasks List**: A page to list all our tasks
2. **Create New Task**: Another page to create a new task
3. **Task Page**: A page specific to the selected task

To create these two pages, we will create a folder named `tasks` at `src/pages/tasks.`

In the folder `src/pages/tasks` we will create three **Page Components**:

#### 1. List Page

We create a page `page.tsx`, which is accessible at the path `/tasks`.

NB: we will be creating the `TasksContainer` in the next steps.

```tsx filename=src/pages/tasks/page.tsx {4,16}
import { GetServerSidePropsContext } from 'next';
import { withAppProps } from '~/lib/props/with-app-props';
import RouteShell from '~/components/RouteShell';
import TasksContainer from '~/components/tasks/TasksContainer';
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const Tasks = () => {
  const organization = useCurrentOrganization();

  if (!organization) {
    return null;
  }

  return (
    <RouteShell title={'Tasks'}>
      <TasksContainer organizationId={organization.id} />
    </RouteShell>
  );
};

export default Tasks;

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  return await withAppProps(ctx);
}
```

Notice: we have added a `getServerSideProps` function to the page. This is because we need to fetch the current organization of the user before rendering the page. We will be using this organization ID to fetch the tasks. Additionally, the `withAppProps` function will:
- ensure the user is logged in
- ensure the user has an organization and is onboarded

As a rule of thumb, every page that requires the user to be logged in should use a `getServerSideProps` function with `withAppProps`.

```tsx /withAppProps/
import { GetServerSidePropsContext } from "next";
import { withAppProps } from "~/lib/props/with-app-props";

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  return await withAppProps(ctx);
}
```

#### 2. Create Task Page

We create a page `new.tsx`, which is accessible at the path `/tasks/new`. We will be creating the `CreateTaskForm` component in the next steps.

```tsx filename=src/pages/tasks/new.tsx {4,12}
import { GetServerSidePropsContext } from 'next';
import { withAppProps } from '~/lib/props/with-app-props';
import RouteShell from '~/components/RouteShell';
import CreateTaskForm from '~/components/tasks/CreateTaskForm';

const NewTaskPage = () => {
  return (
    <RouteShell title={'New Task'}>
      <div
        className={'max-w-2xl border border-gray-50 p-8 dark:border-black-400'}
      >
        <CreateTaskForm />
      </div>
    </RouteShell>
  );
};

export default NewTaskPage;

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  return await withAppProps(ctx);
}
```

#### 3. Task Page

We create a page `[id].tsx`, which is accessible at the path `/tasks/<taskID>` where `taskID` is a dynamic variable that refers to the actual ID of the task.

```txt
├── pages
  └── tasks
    └── page.tsx
    └── new.tsx
    └── [id].tsx
```

Below is what the page looks like. We will be defining the `TaskItemContainer` component in the next steps.

```tsx filename=src/pages/tasks/[id].tsx {5,19}
import { GetServerSidePropsContext } from 'next';
import ArrowLeftIcon from '@heroicons/react/24/outline/ArrowLeftIcon';

import { withAppProps } from '~/lib/props/with-app-props';
import TaskItemContainer from '~/components/tasks/TaskItemContainer';

import RouteShell from '~/components/RouteShell';
import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';
import Alert from '~/core/ui/Alert';
import ErrorBoundary from '~/core/ui/ErrorBoundary';

const TaskPage: React.FC<{ taskId: string }> = ({ taskId }) => {
  return (
    <RouteShell title={<TaskPageHeading />}>
      <ErrorBoundary
        fallback={<Alert type={'error'}>Ops, an error occurred :(</Alert>}
      >
        <TaskItemContainer taskId={taskId} />
      </ErrorBoundary>
    </RouteShell>
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

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  const appProps = await withAppProps(ctx);
  const taskId = ctx.query.id;

  if ('props' in appProps) {
    return {
      props: {
        ...(appProps.props ?? {}),
        taskId,
      },
    };
  }

  return appProps;
}
````

Above, we're extending `withAppProps` to include the `taskId` in the props. This is just an example, it's not strictly needed as we would be able to fetch the task ID from the URL in the `TaskItemContainer` component.

With that said, assuming you want to fetch data in the `getServerSideProps` function, you can do so and then pass it to the component as props.

### Adding Functionalities to your application

To add new functionalities to your application (in our case, tasks management), usually, you'd need the following things:

1. **Add a new page**, and the links to it in `pages`
2. **Add the components of the domain** and import them into the pages in `components`
3. Add data-fetching and writing to this domain's entities in `lib`

The above are the things we will do in the following few sections.

To clarify further the conventions of this boilerplate, here is what you should know:

1. **Data Model**: first, we want to define the data model of the feature you want to add
2. **Firestore Hooks**: once we're happy with the data model, we can create our Firestore hooks to write new tasks and then fetch the ones we created
3. **Import hooks into components**: then, we import and use our hooks within the components
4. **Add feature components into the pages**: finally, we add the components to the pages

### Adding page links to the Navigation Menu

To update the navigation menu, we need to update the `NAVIGATION_CONFIG` object in `src/navigation.config.tsx`.

```ts
const NAVIGATION_CONFIG = {
  items: [
    {
      label: 'common:dashboardTabLabel',
      path: configuration.paths.appHome,
      Icon: ({ className }: { className: string }) => {
        return <Squares2X2Icon className={className} />;
      },
    },
    {
      label: 'common:settingsTabLabel',
      path: '/settings',
      Icon: ({ className }: { className: string }) => {
        return <Cog8ToothIcon className={className} />;
      },
    },
  ],
};
```

To add a new link to the header menu, we can add the following item in the `NAVIGATION_CONFIG` object:

```tsx
{
  label: 'common:tasksTabLabel',
  path: '/tasks',
  Icon: ({ className }: { className: string }) => {
    return <Squares2X2Icon className={className} />;
  },
},
```

Now, also add the `common:tasksTabLabel` key to the `public/locales/en/common.json` file:

```json
{
   "tasksTabLabel": "Tasks"
}
```

Save the file and restart the Next server by rerunning the `npm run dev` command.

The result will be similar to the images below:

{% img src="/assets/images/posts/sidebar-layout-link.webp" width="1280" height="984" /%}

-----------------
FILE PATH: kits/next-fire/tutorials/environment-variables.mdoc

---
status: "published"

title: "Environment Variables"
label: Environment Variables
order: 5
description: "Learn how to use environment variables in your Next.js project."
---


The starter project comes with three different environment variables files:

1. **.env**: this is a shared environment file. Here, you can add variables shared across all the environments.
2. **.env.development**: the configuration file loaded when running the `dev` command: by default, we add the Firebase Emulators settings to this file.
 - Use this file for your **development configuration**.
3. **.env.production**: the configuration file loaded when running the `build` command locally or deployed to Vercel.
 - Use this file for your **actual project's configuration** (without private keys!).
4. **.env.test**: this environment file is loaded when running the Cypress E2E tests. You would rarely need to use this.

Additionally, you can add another environment file named `.env.local`: this file is never added to Git and can be used to store the secrets you need during development.

{% alert type="warning" title="Never add secrets to your .env files" %}
<span className='block'>
    Never ever add secrets to your `.env` files. It's not safe, and you risk leaking confidential data.
  </span>

  <span>
    Instead, add your secrets using your Vercel Console, or use the `.env.local` file during development.
  </span>
{% /alert %}

## Adding Environment Variables

The production environment variables template looks like the below:

```bash
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_SITE_URL=
NEXT_PUBLIC_APPCHECK_KEY=
NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION=false

SERVICE_ACCOUNT_CLIENT_EMAIL=
GCLOUD_PROJECT=

## SECRET KEYS ARE BEST ADDED TO YOUR CI
SERVICE_ACCOUNT_PRIVATE_KEY=

STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

-----------------
FILE PATH: kits/next-fire/tutorials/firestore-data-fetching.mdoc

---
status: "published"

title: "Firestore: Data Fetching"
label: "Firestore: Data Fetching"
order: 9
description: "Learn how to fetch data from Firestore in your React applications."
---

Before writing the Components, we want to tackle the task of fetching data from Firestore.

This involves a couple of steps:

1. We need to define, on paper, what the data model of the Firestore collections looks like
2. We need to update the Firestore rules so we can read and write data to the collection
3. We need to write the hooks to fetch data from Firebase Firestore

#### Firestore Data Model

First, let's write the data model. But... what does it mean?

When thinking about the Firestore data model, we need to answer the following questions:

1. Where does the data live?
2. How is the data structured?
3. How do we protect the data with security rules?

Ok, let's reply to these questions.

##### Where does the data live?

We can place `tasks` as a Firestore top-level collection. Because tasks belong to an organization, we can ensure that only users of that organization can read and write those tasks by adding a foreign key to each `task` named `organizationId`.

##### How is the data structured?

We can define a `Task` model at `src/lib/tasks/types/task.ts`:

 ```tsx {% title="src/lib/tasks/types/task.ts" %}
export interface Task {
  name: string;
  description: string;
  organizationId: string;
  dueDate: string;
  done: boolean;
}
```

##### How do we protect the data with security rules?

To write our Security Rules, we will update the file `firestore.rules`:

```
match /tasks/{taskId} {
  allow create: if userIsMemberByOrganizationId(incomingData().organizationId);
  allow read, update, delete: if userIsMemberByOrganizationId(existingData().organizationId);
}
```

The function `userIsMemberByOrganizationId` is pre-defined in the Makerkit's template: basically, it will verify that the current user is a member of the organization with ID `organizationId`.

If you use VSCode, take a look at the extension `toba.vsfire`.

### Data Fetching: React Hooks to read data from Firestore

Now that our security rules are updated, we can write Hooks to read data from Firestore.

I recommend writing your entities' hooks at `src/lib/tasks/hooks`. Let's start with a `React Hook` to fetch all the organization's tasks:

 ```tsx {% title="src/lib/tasks/hooks/use-fetch-tasks.ts" %}
import { useFirestore, useFirestoreCollectionData } from 'reactfire';

import {
  collection,
  CollectionReference,
  query,
  where,
} from 'firebase/firestore';

import { Task } from '~/lib/tasks/types/task';

function useFetchTasks(organizationId: string) {
  const firestore = useFirestore();
  const tasksCollection = 'tasks';

  const collectionRef = collection(
    firestore,
    tasksCollection
  ) as CollectionReference<WithId<Task>>;

  const path = `organizationId`;
  const operator = '==';
  const constraint = where(path, operator, organizationId);
  const organizationsQuery = query(collectionRef, constraint);

  return useFirestoreCollectionData(organizationsQuery, {
    idField: 'id',
  });
}

export default useFetchTasks;
```

Let's take a deep breath and slowly go through the above hook.

1. The hook above uses `useFirestoreCollectionData`, a `React hook` from the library `reactfire` to fetch real-time collection data from Firestore
2. We build a query that checks that the `organizationId` parameter matches the `organizationId` property of each task
3. Finally, we add `{ idField: 'id' }`. This special parameter decorates the data with the ID of the document. Thanks to the interface `WithId`, we add an `id` property to the `Task` interface returned from Firestore

### Displaying Firestore data in Components

Now that we can fetch our data using the hook `useFetchTasks,` we can build a component that lists each `Task.`

To do so, let's create a new component named `TasksListContainer`:

 ```tsx {% title="components/tasks/TasksContainer.tsx" %}
import PageLoadingIndicator from '~/core/ui/PageLoadingIndicator';
import Alert from '~/core/ui/Alert';
import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';

import useFetchTasks from '~/lib/tasks/hooks/use-fetch-tasks';
import TasksList from '~/components/tasks/TasksList';

const TasksContainer: React.FC<{
  organizationId: string;
}> = ({ organizationId }) => {
  const { status, data: tasks } = useFetchTasks(organizationId);

  if (status === `loading`) {
    return <PageLoadingIndicator>Loading Tasks...</PageLoadingIndicator>;
  }

  if (status === `error`) {
    return (
      <Alert type={'error'}>
        Sorry, we encountered an error while fetching your tasks.
      </Alert>
    );
  }

  if (tasks.length === 0) {
    return <TasksEmptyState />;
  }

  return (
    <div className={'flex flex-col space-y-4'}>
      <div className={'flex justify-end'}>
        <CreateTaskButton>New Task</CreateTaskButton>
      </div>

      <TasksList tasks={tasks} />
    </div>
  );
};

function TasksEmptyState() {
  return (
    <div
      className={
        'flex flex-col items-center justify-center space-y-4' + ' h-full p-24'
      }
    >
      <div>
        <Heading type={5}>No tasks found</Heading>
      </div>

      <CreateTaskButton>Create your first Task</CreateTaskButton>
    </div>
  );
}

function CreateTaskButton(props: React.PropsWithChildren) {
  return <Button href={'/tasks/new'}>{props.children}</Button>;
}

export default TasksContainer;
```

The data above will automatically update when the `tasks` collection changes!

{% alert type="warning" title="Hold on, we're not done yet!" %}

  Since the component "TaskListItem" is pretty complex and requires more files, we will define it separately. You will find it below after we explain the mutation hooks that the component will use.
{% /alert %}


-----------------
FILE PATH: kits/next-fire/tutorials/firestore-data-writing.mdoc

---
status: "published"

title: "Firestore: Data Writing"
label: "Firestore: Data Writing"
order: 10
description: "Learn how to write data to Firestore in your React applications."
---

While `reactfire` does not provide hooks for writing data to Firestore, we can still make our own.

In fact, **I encourage you to make a custom hook for each mutation**.

## Mutations

#### Creating a Task

In the example below, we write a custom hook to add a document to the `tasks` collection:

 ```tsx {% title="src/lib/tasks/hooks/use-create-task.ts" %}
import { useFirestore } from 'reactfire';
import { useCallback } from 'react';
import { addDoc, collection } from 'firebase/firestore';
import { Task } from '~/lib/tasks/types/task';

function useCreateTask() {
  const firestore = useFirestore();
  const tasksCollection = collection(firestore, `/tasks`);

  return useCallback(
    (task: Task) => {
      return addDoc(tasksCollection, task);
    },
    [tasksCollection]
  );
}

export default useCreateTask;
```

The hook above returns a callback. Let's take a quick look at its usage:

```tsx
import useCreateTask from '~/lib/tasks/hooks/use-create-task';

function Component() {
  const createTask = useCreateTask();

  return <Form onCreate={task => createTask(task)} />
}
```

#### Updating a Task

Now, let's write a hook to update an existing task.

To update Firestore documents, we will import the function `updateDoc` from Firestore.

 ```tsx {% title="src/lib/tasks/hooks/use-update-task.ts" %}
import { useCallback } from 'react';
import { useFirestore } from 'reactfire';
import { doc, updateDoc } from 'firebase/firestore';
import { Task } from '~/lib/tasks/types/task';

function useUpdateTask(taskId: string) {
  const firestore = useFirestore();
  const tasksCollection = 'tasks';

  const docRef = doc(firestore, tasksCollection, taskId);

  return useCallback(
    (task: Partial<Task>) => {
      return updateDoc(docRef, task);
    },
    [docRef]
  );
}

export default useUpdateTask;
```

#### Deleting a Task

Finally, we write a mutation to delete a task:

 ```tsx {% title="src/lib/tasks/hooks/use-delete-task.ts" %}
import { useFirestore } from 'reactfire';
import { deleteDoc, doc } from 'firebase/firestore';
import { useCallback } from 'react';

function useDeleteTask(taskId: string) {
  const firestore = useFirestore();
  const collection = `tasks`;
  const task = doc(firestore, collection, taskId);

  return useCallback(() => {
    return deleteDoc(task);
  }, [task]);
}

export default useDeleteTask;
```

### Using the Modal component for confirming actions

The `Modal` component is excellent for asking for confirmation before performing a destructive action. In our case, we want the users to confirm before deleting a task.

The `ConfirmDeleteTaskModal` is defined below using the core `Modal` component:

 ```tsx {% title="src/components/tasks/ConfirmDeleteTaskModal.tsx" %}
import Modal from '~/core/ui/Modal';
import Button from '~/core/ui/Button';

const ConfirmDeleteTaskModal: React.FC<{
  isOpen: boolean;
  setIsOpen: (isOpen: boolean) => void;
  task: string;
  onConfirm: () => void;
}> = ({ isOpen, setIsOpen, onConfirm, task }) => {
  return (
    <Modal heading={`Deleting Task`} isOpen={isOpen} setIsOpen={setIsOpen}>
      <div className={'flex flex-col space-y-4'}>
        <p>
          You are about to delete the task <b>{task}</b>
        </p>

        <p>Do you want to continue?</p>

        <Button block color={'danger'} onClick={onConfirm}>
          Yep, delete task
        </Button>
      </div>
    </Modal>
  );
};

export default ConfirmDeleteTaskModal;
```

### Wrapping all up: listing each Task Item

Now that we have defined the mutations we can perform on each task item, we can wrap it up.

The component below represents a `task` item and allows users to mark it as `done` or `not done` - or delete them.

 ```tsx {% title="src/components/tasks/TasksListItem.tsx" %}
import { useCallback, useState } from 'react';
import Link from 'next/link';
import { TrashIcon } from '@heroicons/react/24/outline';
import { toast } from 'sonner';
import { formatDistance } from 'date-fns';

import { Task } from '~/lib/tasks/types/task';
import Heading from '~/core/ui/Heading';
import IconButton from '~/core/ui/IconButton';
import Tooltip from '~/core/ui/Tooltip';
import useDeleteTask from '~/lib/tasks/hooks/use-delete-task';
import ConfirmDeleteTaskModal from '~/components/tasks/ConfirmDeleteTaskModal';
import useUpdateTask from '~/lib/tasks/hooks/use-update-task';

const TasksListItem: React.FC<{
  task: WithId<Task>;
}> = ({ task }) => {
  const getTimeAgo = useTimeAgo();
  const deleteTask = useDeleteTask(task.id);
  const updateTask = useUpdateTask(task.id);

  const [isDeleting, setIsDeleting] = useState(false);

  const onDelete = useCallback(() => {
    const deleteTaskPromise = deleteTask();

    return toast.promise(deleteTaskPromise, {
      success: `Task deleted!`,
      loading: `Deleting task...`,
      error: `Ops, error! We could not delete task`,
    });
  }, [deleteTask]);

  const onDoneChange = useCallback(
    (done: boolean) => {
      const promise = updateTask({ done });

      return toast.promise(promise, {
        success: `Task updated!`,
        loading: `Updating task...`,
        error: `Ops, error! We could not update task`,
      });
    },
    [updateTask]
  );

  return (
    <>
      <div
        className={'rounded border p-4 transition-colors dark:border-black-400'}
      >
        <div className={'flex items-center space-x-4'}>
          <div>
            <Tooltip content={task.done ? `Mark as not done` : `Mark as done`}>
              <input
                className={'Toggle cursor-pointer'}
                type="checkbox"
                defaultChecked={task.done}
                onChange={(e) => {
                  return onDoneChange(e.currentTarget.checked);
                }}
              />
            </Tooltip>
          </div>

          <div className={'flex flex-1 flex-col space-y-0.5'}>
            <Heading type={5}>
              <Link
                className={'hover:underline'}
                href={`/tasks/[id]`}
                as={`/tasks/${task.id}`}
                passHref
              >
                {task.name}
              </Link>
            </Heading>

            <div>
              <p className={'text-xs text-gray-400 dark:text-gray-500'}>
                Due {getTimeAgo(new Date(task.dueDate))}
              </p>
            </div>
          </div>

          <div className={'flex justify-end'}>
            <Tooltip content={`Delete Task`}>
              <IconButton
                onClick={(e) => {
                  e.stopPropagation();
                  setIsDeleting(true);
                }}
              >
                <TrashIcon className={'h-5 text-red-500'} />
              </IconButton>
            </Tooltip>
          </div>
        </div>
      </div>

      <ConfirmDeleteTaskModal
        isOpen={isDeleting}
        setIsOpen={setIsDeleting}
        task={task.name}
        onConfirm={onDelete}
      />
    </>
  );
};

export default TasksListItem;

function useTimeAgo() {
  return useCallback((date: Date) => {
    return formatDistance(date, new Date(), {
      addSuffix: true,
    });
  }, []);
}
```

-----------------
FILE PATH: kits/next-fire/tutorials/forms.mdoc

---
status: "published"

title: "Forms"
label: "Forms"
order: 11
description: "Learn how to build forms in the Next.js Firebase kit"
---

Now that we have created a custom hook to write data to Firestore, we need to build a form to create a new `task`.

The example below is straightforward:

1. We have a form with two input fields
2. When the form is submitted, we read the data using `FormData`
3. Finally, we add the document using the callback returned by the hook `useCreateTask`

 ```tsx {% title="components/tasks/CreateTaskForm.tsx" %}
import { useRouter } from 'next/router';
import { FormEventHandler, useCallback } from 'react';
import { toast } from 'sonner';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';
import If from '~/core/ui/If';
import Heading from '~/core/ui/Heading';

import useCreateTask from '~/lib/tasks/hooks/use-create-task';
import { useRequestState } from '~/core/hooks/use-request-state';
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const CreateTaskForm = () => {
  const createTask = useCreateTask();
  const { setLoading, state } = useRequestState();
  const router = useRouter();
  const organization = useCurrentOrganization();
  const organizationId = organization?.id as string;

  const onCreateTask: FormEventHandler<HTMLFormElement> = useCallback(
    async (event) => {
      event.preventDefault();

      const target = event.currentTarget;
      const data = new FormData(target);
      const name = data.get('name') as string;

      const dueDate = (data.get('dueDate') as string) || getDefaultDueDate();

      setLoading(true);

      const task = {
        organizationId,
        name,
        dueDate,
        description: ``,
        done: false,
      };

      const promise = createTask(task).then(() => {
        return router.push(`/tasks`);
      });

      await toast.promise(promise, {
        success: `Task created!`,
        error: `Ops, error!`,
        loading: `Creating task...`,
      });
    },
    [router, createTask, organizationId, setLoading]
  );

  return (
    <div className={'flex flex-col space-y-4'}>
      <div>
        <Heading type={2}>Create a new Task</Heading>
      </div>

      <form onSubmit={onCreateTask}>
        <div className={'flex flex-col space-y-3'}>
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
              'flex flex-col space-y-2 md:space-y-0 md:space-x-2' +
              ' md:flex-row'
            }
          >
            <Button loading={state.loading}>
              <If condition={state.loading} fallback={<>Create Task</>}>
                Creating Task...
              </If>
            </Button>

            <Button color={'transparent'} href={'/tasks'}>
              Go back
            </Button>
          </div>
        </div>
      </form>
    </div>
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

-----------------
FILE PATH: kits/next-fire/tutorials/functions.mdoc

---
status: "published"

title: "Functions you need to know"
label: "Functions you need to know"
order: 15
description: "Learn the most important functions you need to know to use the Next.js Firebase kit effectively."
---

This is an extremely important section of the documentation: it will teach you the most important functions you need to know to use the Next.js Firebase kit effectively.

We will cover both client-side and server-side functions.

## Client-Side Functions

### ReactFire Hooks

The Next.js Firebase kit uses [ReactFire](https://github.com/firebaseextended/reactfire) extensively throughout the kit. ReactFire is a library that allows you to bind Firebase data to React components. This allows you to easily display data from Firebase in your React components.

For the majority of use cases, you will be using Reactfire's React hooks to interact with your Firebase data, as we've seen in the previous sections.

These include:
- `useFirestore`: this will allow you to interact with your Firestore database
- `useAuth`: this will allow you to interact with your Firebase authentication
- `useStorage`: this will allow you to interact with your Firebase storage (you are required to add the FirebaseStorageProvider above your component tree for this to work)

### Makerkit Hooks

#### 1. `useCurrentOrganization`: getting the current user organization

This hook will allow you to get the current user organization. It is populated with the data fetched server-side in the `getServerSideProps` function, and is injected using the Context API.

For example, let's see how we can use this hook to get the current user organization name:

```ts
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

export function useCurrentOrganizationName() {
  const organization = useCurrentOrganization();

  return organization?.name;
}
```

There are a variety of other hooks that you can use to get the current user organization data. You can find them in the `~/lib/organizations/hooks` folder.

#### 2. `useCurrentUserRole`: getting the current user role within the current organization

In case you need to access the current user role within the current organization, you can use the `useCurrentUserRole` hook.

For example, let's see how we can use this hook to get the current user role within the current organization:

```tsx
import { useCurrentUserRole } from '~/lib/organizations/hooks/use-current-user-role';

function MyComponent() {
  const role = useCurrentUserRole();

  return (
    <div>
      <p>Current user role: {role}</p>
    </div>
  );
}
```

#### 3. `useUserSession`: getting the current user session information

In case you need to access the current user session data, you can use the `useUserSession` hook.

This hook will return the current user session data, which includes the following information:
- `auth`: the data that comes from Firebase Authentication
- `data`: the data that comes from the user's Firestore document

```tsx
export interface UserSession {
  auth: AuthUser | undefined;
  data: UserData | undefined;
}
```

For example, let's see how we can use this hook to get the current user ID:

```tsx
import { useUserSession } from '~/core/hooks/use-user-session';

function MyComponent() {
  const userSession = useUserSession();
  const userId = userSession?.auth?.uid;

  return (
    <div>
      <p>Current user ID: {userId}</p>
    </div>
  );
}
```

#### 4. `useIsSubscriptionActive`: getting the current organization subscription status and checking if it's active

A subscription is considered active if it's either `active` or `trialing`. The `useIsSubscriptionActive` hook will return `true` if the current organization is on any paid subscription, regardless of plan.

This can be useful if you want to display a message to the user if they are on a paid subscription, or if you want to restrict access to certain pages.

You can customize this hook to fit your needs.

```tsx
import type { Stripe } from 'stripe';
import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const ACTIVE_STATUSES: Stripe.Subscription.Status[] = ['active', 'trialing'];

/**
 * @name useIsSubscriptionActive
 * @description Returns whether the organization is on any paid
 * subscription, regardless of plan.
 */
function useIsSubscriptionActive() {
  const organization = useCurrentOrganization();
  const status = organization?.subscription?.status;

  if (!status) {
    return false;
  }

  return ACTIVE_STATUSES.includes(status);
}

export default useIsSubscriptionActive;
```

## Server-Side Functions

### Props Functions

Props functions are functions that are executed server-side and are used to populate the props of a page. They are executed before the page is rendered, and are used to fetch data that is required for the page to render.

#### 1. `withAppProps`: populating the application pages data

The `withAppProps` function is used to populate the props of the application pages. It is used in the `getServerSideProps` function of the pages you want to gate.

If you take a look at the previous sections, we used this function to protect the `tasks` application so that only authenticated users can access it.

```tsx
import { withAppProps } from "~/lib/props/with-app-props";
import { GetServerSidePropsContext } from "next";

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  return await withAppProps(ctx);
}
```

#### 2. `withTranslationsProps`: populating the application translations

This function is used to populate the translations of the application. All the other props functions include this by default, so you don't need to use it directly unless it's on a page that does not use any of the other ones.

You will be using this for the majority of your marketing pages.

```tsx
import { withTranslationProps } from "~/lib/props/with-translation-props";
import { GetStaticPropsContext } from "next";

export async function getStaticProps(
  context: GetStaticPropsContext
) {
  const { props } = await withTranslationProps(context);

  return {
    props,
  };
}
```

#### 3. `withAuthProps`: populating the authentication pages data

The `withAuthProps` function is used to populate the props of the authentication pages. You may never need to use it unless you introduce new authentication pages.

This function is used in the `getServerSideProps` function of the authentication pages.

```tsx
import { withAuthProps } from "~/lib/props/with-auth-props";
import { GetServerSidePropsContext } from "next";

export async function getServerSideProps(ctx: GetServerSidePropsContext) {
  return await withAuthProps(ctx);
}
```

This function ensures **authenticated users won't be able to access the authentication pages**.

### API Functions

API Routes have several utilities that help you build your API endpoints, and save you from writing boilerplate code.

#### 1. `withPipe`: piping functions when handling API requests

The `withPipe` function is used to pipe functions when handling API requests.

```ts {12}
import { NextApiRequest, NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withMethodsGuard(['GET']),
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

This helps you write the utility functions we will see next in a more readable way.

#### 2. `withMethodsGuard`: guarding API endpoints by HTTP methods

The `withMethodsGuard` function is used to guard API endpoints by HTTP methods. For example, in the example below, we only allow `GET` and `POST` requests to the `/api/hello-world` endpoint.

```ts {13}
import { NextApiRequest, NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withMethodsGuard(['GET', 'POST']),
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

#### 3. `withAuthedUser`: require authentication to access an API endpoint

The `withAuthedUser` function is used to require authentication to access an API endpoint. It will return a `401` error if the user is not authenticated.

```ts {12}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

#### 4. `withExceptionFilter`: handle exceptions in API endpoints

The `withExceptionFilter` function is used to handle exceptions in API endpoints. It will log errors and report to Sentry when errors are encountered.

```ts {18}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';

export default function helloWorld(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

#### 5. `withCsrf`:  protect API endpoints from CSRF attacks

The `withCsrf` function is used to protect API endpoints from CSRF attacks. It will return a `403` error if the CSRF token is invalid.

```ts {10}
import { NextApiRequest,NextApiResponse } from "next";

import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';
import { withCsrf } from '~/core/middleware/with-csrf';

export default function owner(req: NextApiRequest, res: NextApiResponse) {
  const handler = withPipe(
    withCsrf(),
    withAuthedUser,
    (req, res) => {
      res.status(200).json({ message: 'Hello World!' });
    }
  );

  return withExceptionFilter(req, res)(handler);
}
```

