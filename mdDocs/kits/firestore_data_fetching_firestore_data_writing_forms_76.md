-----------------
FILE PATH: kits\remix-fire\tutorials\firestore-data-fetching.mdoc

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

 ```tsx {% title="components/tasks/TasksListContainer.tsx" %}
import PageLoadingIndicator from '~/core/ui/PageLoadingIndicator';
import Alert from '~/core/ui/Alert';
import Heading from '~/core/ui/Heading';
import Button from '~/core/ui/Button';

import useFetchTasks from '~/lib/tasks/hooks/use-fetch-tasks';
import TasksList from '~/components/tasks/TasksList';
import { Task } from '~/lib/tasks/types/task';

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
      <div className={'mt-2 flex justify-end'}>
        <CreateTaskButton>New Task</CreateTaskButton>
      </div>

      <div className={'flex flex-col space-y-4'}>
        {tasks.map((task) => {
          return <TaskListItem task={task} key={task.id} />;
        })}
      </div>
    </div>
  );
};

function TasksEmptyState() {
  return (
    <div
      className={
        'flex flex-col items-center justify-center space-y-4 h-full p-24'
      }
    >
      <div>
        <Heading type={5}>No tasks found</Heading>
      </div>

      <CreateTaskButton>Create your first Task</CreateTaskButton>
    </div>
  );
}

function TasksList({ tasks }: React.PropsWithChildren<{
  tasks: Task[]
}>) {
  return (
    <div className={'flex flex-col space-y-4'}>
      {tasks.map((task) => {
        return <TaskListItem task={task} key={task.id} />;
      })}
    </div>
  );
}

export default TasksList;


function CreateTaskButton(props: React.PropsWithChildren) {
  return <Button href={'/tasks/new'}>{props.children}</Button>;
}

export default TasksContainer;
```

The data above will automatically update when the `tasks` collection changes!

{% alert type="warning" %}
Since the component "TaskListItem" is pretty complex and requires more files, we will define it separately. You will find it below after we explain the mutation hooks that the component will use.
{% /alert %}


-----------------
FILE PATH: kits\remix-fire\tutorials\firestore-data-writing.mdoc

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
FILE PATH: kits\remix-fire\tutorials\forms.mdoc

---
status: "published"

title: "Forms"
label: "Forms"
order: 11
description: "Learn how to build forms in the Remix Firebase kit"
---


Now that we have created a custom hook to write data to Firestore, we need to build a form to create a new `task`.

The example below is straightforward:
1. We have a form with two input fields
2. When the form is submitted, we read the data using `FormData`
3. Finally, we add the document using the callback returned by the hook `useCreateTask`

 ```tsx {% title="components/tasks/CreateTaskForm.tsx" %}
import { useNavigate } from '@remix-run/react';
import { FormEventHandler, useCallback } from 'react';

import TextField from '~/core/ui/TextField';
import Button from '~/core/ui/Button';
import useCreateTask from '~/lib/tasks/hooks/use-create-task';

import { useCurrentOrganization } from '~/lib/organizations/hooks/use-current-organization';

const CreateTaskForm = () => {
  const createTask = useCreateTask();
  const navigate = useNavigate();
  const organization = useCurrentOrganization();
  const organizationId = organization?.id as string;

  const onCreateTask: FormEventHandler<HTMLFormElement> = useCallback(
    async (event) => {
      event.preventDefault();

      const target = event.currentTarget;
      const data = new FormData(target);
      const name = data.get('name') as string;
      const dueDate = data.get('dueDate') as string;

      const task = {
        organizationId,
        name,
        dueDate,
        done: false,
      };

      // create task
      await createTask(task);

      // redirect to /tasks
      await navigate(`/tasks`);
    },
    [navigate, createTask, organizationId]
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
          <TextField.Input required name={'dueDate'} type={'date'} />
        </TextField.Label>

        <div>
          <Button>
            Create Task
          </Button>
        </div>
      </div>
    </form>
  );
};

export default CreateTaskForm;
```


-----------------
FILE PATH: kits\remix-fire\tutorials\initializing-project.mdoc

---
status: "published"
title: "Setting up the Remix Firebase SaaS template"
label: Initial Setup
order: 1
description: "This section describes how to set up your development environment to use the Remix Firebase SaaS template, from forking the repository to installing the Node modules"
---


## Initializing the Project

You have two ways to initialize the project:

1. Forking it from GitHub, or
2. Cloning it using Git

### 1. Forking the Repository

Fork this repository by clicking on the "Fork" button on the top right
corner in GitHub for the repository you want to use.

Once you have forked the repository, clone it locally. Then, set the
upstream repository to the original repository, so you can pull updates
when needed:

```
git remote add upstream git@github.com:makerkit/remix-firebase-saas-kit.git
```

### 2. Cloning the Repository

Alternatively, assuming you have accepted the invites and have access to the repository, open your terminal and run this command (replace `tasks-app` with your name of choice):

```
git clone git@github.com:makerkit/remix-firebase-saas-kit.git tasks-app
```

Once completed, we'll change into the `tasks-app` directory, and then we will install the Node modules:

```
cd tasks-app
npm i
```

### Reinitialize Git

As the Git repository's remote points to Makerkit's original repository, you can re-initialize (optionally!) it and set the Makerkit repository as `upstream`:

```
git remote rm origin
```

Then, we set the Makerkit repository as `upstream`:

```
git remote add upstream git@github.com:makerkit/remix-firebase-saas-kit.git
git add .
git commit -a -m "Initial Commit"
```

By adding the Makerkit's repository as `upstream` remote, you can fetch updates (after committing your files) by running the following command:

```
git pull upstream main --allow-unrelated-histories
```

Perfect! Now you can fire up your IDE and open the `tasks-app` project we just created.

-----------------
FILE PATH: kits\remix-fire\tutorials\introduction.mdoc

---
status: "published"
title: "Building a SaaS tasks application with Makerkit | Remix.js Firebase"
label: Introduction
order: 0
description: "Get a head start on your app development with our Makerkit SaaS boilerplate, built with Remix and Firebase. Follow our step-by-step guide for an easy setup!"
---

This tutorial is a comprehensive guide to getting started with the Remix and Firebase SaaS template to build a basic tasks application, from fetching the repository to deploying the application.

By the end, you'll have a fully working application with the minimum basics a SaaS needs:

1. Landing Page and Marketing pages (blog, documentation)
2. Authentication (sign in, sign up)
3. Payments (Stripe)
4. Profile and Organization management

## Prerequisites

To get started, you're going to need some things installed:

1. Git
2. Node.js version (>=16.0.0)
3. npm 7 or greater
4. A code editor (VSCode, WebStorm)
5. If you'd like to deploy your application, you'll also want an account on Vercel.

Experience with React, TypeScript/JavaScript, and Firebase would be advantageous but not strictly required. The codebase can also serve as a way to learn these topics more in-depth.

If you have all the above installed, I guess we can get started!

-----------------
FILE PATH: kits\remix-fire\tutorials\marketing-site-pages.mdoc

---
status: "published"
title: "Adding pages to the Marketing Site | Remix Firebase SaaS Kit"
label: "Adding pages to the Marketing Site"
order: 15
description: "Learn how to add pages to the Marketing Site."
---

To update the site's navigation menu, you can visit the `SiteNavigation.tsx` component and add a new entry to the menu.

By default, you will see the following object `links`:

 ```tsx {% title="components/SiteNavigation.tsx" %}
const links: Record<string, Link> = {
  Blog: {
    label: 'Blog',
    path: '/blog',
  },
  Docs: {
    label: 'Docs',
    path: '/docs',
  },
  Pricing: {
    label: 'Pricing',
    path: '/pricing',
  },
  FAQ: {
    label: 'FAQ',
    path: '/faq',
  },
};
```

The menu is defined in the render function:

```tsx
<NavigationMenu>
  <NavigationMenuItem link={links.Blog} />
  <NavigationMenuItem link={links.Docs} />
  <NavigationMenuItem link={links.Pricing} />
  <NavigationMenuItem link={links.FAQ} />
</NavigationMenu>
```

Assuming we want to add a new menu entry, say `About`, we would first add the link object:

```tsx
About: {
  label: 'About',
  path: '/about',
},
```

And then we update the menu:

```tsx
<NavigationMenu>
  <NavigationMenuItem link={links.About} />
  ...
</NavigationMenu>
```


-----------------
FILE PATH: kits\remix-fire\tutorials\onboarding-flow.mdoc

---
status: "published"
title: "Onboarding Flow | Remix Firebase SaaS Kit"
label: Onboarding Flow
order: 7
description: "Learn how to onboard your users to your app using the Onboarding Flow in the Remix Firebase SaaS Kit"
---

After sign-up, users are redirected to the onboarding flow.

Here, you can ask users questions, configure their accounts or simply show them around.

By default, Makerkit adds one single step, where it asks users to **create an organization**.

{% img src="/assets/images/posts/onboarding-flow-tutorial.webp" width="2940" height="1972" /%}

Of course, you can extend it and add as many steps as you wish. I would recommend replacing the image on the right-hand side with a screenshot of your application, a video, or something that can help users understand what they are signing up for.

### What happens when the user submits the form?

After the user submits the form, the API will receive the request and:

1. create the user Firestore record at `/users`
2. create the organization Firestore record at `/organizations` and assign the user as its owner

I encourage you to visit the [Firestore Emulator UI](http://localhost:4000/firestore/) and see the data created.

### What is an "Organization"?

Organizations are groups of users.

You can call them projects, teams, classrooms, or whatever feels suitable for your domain. But, generally speaking, organizations are the backbone of the data model because it's where we store most of the data shared among users.

Users can:

1. create new Organizations
2. be invited to other Organizations
3. switch between organizations using a dropdown

### Can I remove this step?

No, **but you can skip it** by automatically submitting the form.

Yes, you can follow this guide for [removing the onboarding flow and organizations](/recipes/removing-organizations).

-----------------
FILE PATH: kits\remix-fire\tutorials\project-configuration.mdoc

---
status: "published"
title: "Project Configuration | Remix Firebase"
label: Project Configuration
order: 4
description: "Learn how to set up up your project's configuration"
---

Makerkit can be configured from a single place: the `src/configuration.ts` object. This file exports a constant we use throughout the project to read our application's settings.

{% alert type="info" title="Use this file to configure your project" %}

  We recommend adding additional configuration to this file rather than reading it directly from the environment variables. For example, assuming you rename one of your environment variables, you will only need to update it in one place. Additionally, it helps maintain your codebase more explicit and understandable.
{% /alert %}

We retrieve some of this file's configuration using **environment variables**: these help us swap and tweak our settings based on which environment we're using, such as the Firebase configuration.

For example, in your Vercel console, you could run multiple projects:
- one for `production`
- and one for `staging`

Here is what the configuration file looks like:

 ```tsx {% title="src/configuration.ts" %}
const configuration = {
  site: {
    name: 'Awesomely - Your SaaS Title',
    description: 'Your SaaS Description',
    themeColor: '#ffffff',
    themeColorDark: '#0a0a0a',
    siteUrl: process.env.SITE_URL as string,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
  },
  firebase: {
    apiKey: process.env.FIREBASE_API_KEY,
    authDomain: process.env.FIREBASE_AUTH_DOMAIN,
    projectId: process.env.FIREBASE_PROJECT_ID,
    storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
    appId: process.env.FIREBASE_APP_ID,
    measurementId: process.env.FIREBASE_MEASUREMENT_ID,
  },
  auth: {
    // Enable MFA. You must upgrade to GCP Identity Platform to use it.
    // see: https://cloud.google.com/identity-platform/docs/product-comparison
    enableMultiFactorAuth: false,
    // NB: Enable the providers below in the Firebase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: [GoogleAuthProvider],
    },
  },
  emulatorHost: process.env.EMULATOR_HOST,
  emulator: process.env.EMULATOR === 'true',
  production: process.env.NODE_ENV === 'production',
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    emailLinkSignIn: '/auth/link',
    onboarding: `/onboarding`,
    appHome: '/dashboard',
    settings: {
      profile: '/settings/profile',
      authentication: '/settings/profile/authentication',
      email: '/settings/profile/email',
      password: '/settings/profile/password',
    },
    api: {
      checkout: `/resources/stripe/checkout`,
      billingPortal: `/resources/stripe/portal`,
    },
    searchIndex: `/public/search-index`,
  },
  navigation: {
    style: LayoutStyle.Sidebar,
  },
  appCheckSiteKey: process.env.APPCHECK_KEY,
  email: {
    host: '',
    port: 0,
    user: '',
    password: '',
    senderAddress: '',
  },
  emailEtherealTestAccount: {
    email: process.env.ETHEREAL_EMAIL,
    password: process.env.ETHEREAL_PASSWORD,
  },
  sentry: {
    dsn: process.env.SENTRY_DSN,
  },
  stripe: {
    plans: [
      {
        name: 'Basic',
        description: 'Description of your Basic plan',
        price: '$249/year',
        stripePriceId: 'basic-plan',
        features: [
          'Feature 1',
          'Feature 2',
          'common:plans.features.feature1'
        ],
      },
    ],
  }
};
```

Feel free to extend this configuration to your needs. For example, you could new properties to the `site` object to store your social media handles, or add new properties to the `paths` object to store the paths to your pages.


-----------------
FILE PATH: kits\remix-fire\tutorials\project-structure.mdoc

---
status: "published"
title: "Project Structure | Remix Firebase SaaS Kit"
label: Project Structure
order: 2
description: "A quick overview of the project structure and how to navigate it in Remix Firebase SaaS Kit."
---

When inspecting the project structure, you will find something similar to the below:

```
tasks-app
├── README.md
├── @types
├── src
│   ├── components
│   ├── core
│   ├── lib
│   └── routes
        └── __app
        └── __site
        └── auth
        └── invite
        └── onboarding
│       └── root.tsx
├── package-lock.json
├── package.json
├── public
│   └── favicon.ico
└── tsconfig.json
```

NB: we omitted a lot of the files for simplicity.

Let's take a deeper look at the structure we see above:

- `src/core`: this folder contains reusable building blocks of the template that **are not related to any domain**. This includes components, React hooks, functions, and so on.
- `src/components`: this folder contains the application's components, including those that belong to your application's domain. For example, we can add the `tasks` components to `src/components/tasks`.
- `src/lib`: this folder contains the business logic of your application's domain. For example, we can add the `tasks` hooks to write and fetch data from Firestore in `src/lib/tasks`.
- `src/routed`: this is Remix folder where we add our application
routes.

Don't worry if this isn't clear yet! We'll explain where we place our files in the sections ahead.

-----------------
FILE PATH: kits\remix-fire\tutorials\running-project.mdoc

---
status: "published"
title: "Running the App | Remix Firebase SaaS Kit"
label: Running the App
order: 3
description: "Learn how to run the app locally for development and testing in Remix Firebase SaaS Kit."
---

When we run the stack of a Makerkit application, we will need to run a few commands before:
1. **Remixs**: First, we need to run the Remix development server
2. **Firebase**: We need to run the Firebase Emulators. The emulators allow us to run a fully-local Firebase environment.
3. **Stripe CLI** (optional): If you plan on interacting with Stripe, you are also required to run the Stripe CLI, which allows us to test the webhooks from Stripe against our local server.

### Why do we need to run the Supabase local environment?

The guide below will show you how to run the application locally. But why locally? Developing locally has many advantages over developing on a remote server:

**Speedy Development:** Local development lets you work without the hindrance of network latency or internet disruptions.

**Simplified Collaboration:** It's typically easier to collaborate with teammates when developing locally on the same project.

**Code-Based Configuration:** Any modifications to your collections via the Dashboard aren't reflected in code. Adopting local development workflows allows you to maintain all table schemas in code.

Once you are ready to deploy your application, you can do so by following
the [production checklist guide](/docs/remix-fire/going-to-production-overview).

### Running the Remix development server

Run the following command from your IDE or from your terminal:

```
npm run dev
```

This command will start the Remix server at [localhost:3000](http://localhost:3000). If you navigate to this page, you should be able to see the landing page of your application:

{% img src="/assets/images/posts/tutorial-landing-page.webp" width="2856" height="1972" /%}

### Running the Firebase Emulators

To interact with Firebase's services, such as Auth, Firestore, and Storage, we need to run the Firebase emulators. To do so, open a new terminal (or, better, from your IDE) and run the following command:

```
npm run firebase:emulators:start
```

If everything is working correctly, you will see the output below:

```
┌────────────────┬────────────────┬─────────────────────────────────┐
│ Emulator       │ Host:Port      │ View in Emulator UI             │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Authentication │ localhost:9099 │ http://localhost:4000/auth      │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Firestore      │ localhost:8080 │ http://localhost:4000/firestore │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Pub/Sub        │ localhost:8085 │ n/a                             │
├────────────────┼────────────────┼─────────────────────────────────┤
│ Storage        │ localhost:9199 │ http://localhost:4000/storage   │
└────────────────┴────────────────┴─────────────────────────────────┘
  Emulator Hub running at localhost:4400
  Other reserved ports: 4500, 9150
```

By default, we run the emulator for the following services: Authentication, Firestore, and Storage. However, you can easily add Firebase Functions and PubSub.

#### Firebase Emulator UI

As you can see from the `View in Emulator UI` column, you have access to the Emulator UI, which helps you see and edit your Firebase Emulator data using a nifty UI that resembles the Firebase Console.

For example, to display the list of users that have signed up, navigate to the [Authentication Emulator](http://localhost:4000/auth).

{% alert type="info" %}
Makerkit's template adds some data to your project by default for testing reasons. In fact, the data you see is used to run the Cypress E2E tests. This is why you'll see some pre-populated data in your Firestore and Authentication tabs.
{% /alert %}

### Running the Stripe CLI (optional)

Optionally, if you want to run Stripe locally (e.g., sending webhooks to your local server), you will also need to run the following command:

```
npm run stripe:listen
```

This command requires Docker, but you can alternatively install Stripe on your OS and change the command to use `stripe` directly.

The above command runs the Stripe CLI and will route webhooks coming from Stripe to your local endpoint. For example, in the Makerkit starter, this endpoint is `/resources/stripe/webhook`.

When running the command, it will print a webhook key used to sign the messages from Stripe. You will need to add this key to your local environment variables file as below:

```
STRIPE_WEBHOOK_SECRET=<KEY>
```

The webhook printed should not change, so you may only need to do this the first time.

