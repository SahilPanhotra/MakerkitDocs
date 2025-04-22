-----------------
FILE PATH: kits\next-fire\how-to\data-fetching\reading-document.mdoc

---
status: "published"

title: "Reading a Document"
label: "Reading a Document"
order: 0
description: "Learn how to read a document from Firebase using both the Web SDK and the Admin SDK in your Next.js Firebase application"
---


There is an important distinction to be made when using the Firebase SDK. Indeed, the SDK can be used in two different ways:

1. **Client SDK** - On the client, with minimal permissions (using the Firebase Security Rules), that allows clients to read and write data directly to the database only if they have the right permissions.
2. **Admin SDK** - On the server, with full permissions, that allows the server to read and write data directly to the database without any restriction.

Depending on the use case, you will need to use one or the other.

## Reading a Document with the Client SDK

The client SDK uses Reactfire to make it easier to fetch data directly from React components using React Hooks.

### Reading a Document with "useFirestoreDocData"

To read a single document, we can use the React Hook `useFirestoreDocData` from Reactfire. This hook takes a `DocumentReference` as a parameter and returns the data of the document as well as the status of the request.

In the example below, we fetch the data of a user document from the Firestore database. We can do so because we have the right permissions to read this document and the User ID is available in the React Context.

```tsx
import { doc, DocumentReference } from 'firebase/firestore';
import { useFirestore, useFirestoreDocData } from 'reactfire';
import { UserData } from '~/core/session/types/user-data';
import { useUserId } from '~/core/hooks/use-user-id';
import { USERS_COLLECTION } from '~/lib/firestore-collections';

function useFetchUser() {
  const firestore = useFirestore();
  const userId = useUserId() as string;

  const ref = doc(
    firestore,
    USERS_COLLECTION,
    userId
  ) as DocumentReference<UserData>;

  return useFirestoreDocData(ref, { idField: 'id' });
}

export default useFetchUser;
```

### Breaking Down the Code

Let's break down the code above to understand what is happening.

#### 1. Importing the Reactfire Hooks

First, we import the Reactfire Hooks that we need to fetch the data from the Firestore database.

```tsx
import { doc, DocumentReference } from 'firebase/firestore';
import { useFirestore, useFirestoreDocData } from 'reactfire';
```

#### 2. Importing the User ID

We also import the User ID from the React Context. This is the ID of the user that is currently logged in.

```tsx
import { useUserId } from '~/core/hooks/use-user-id';
```

#### 3. Creating a Document Reference

We create a `DocumentReference` to the user document in the Firestore database. We use the `useFirestore` hook to get access to the Firestore instance.

```tsx
const firestore = useFirestore();
const userId = useUserId() as string;

const ref = doc(
  firestore,
  USERS_COLLECTION,
  userId
) as DocumentReference<UserData>;
```

The `USERS_COLLECTION` is a constant that contains the name of the collection in the Firestore database where the user documents are stored.

This would be `users` in Makerkit.

#### 4. Fetching the Data

Finally, we use the `useFirestoreDocData` hook to fetch the data of the user document. We pass the `DocumentReference` as a parameter and we specify the name of the field that contains the ID of the document.

```tsx
return useFirestoreDocData(ref, { idField: 'id' });
```

### Reading a Document with "useFirestore"

Let's now see how we can read a document using the `useFirestore` hook from Reactfire from within a React component.

The `useFirestoreDocData` hook will return three values:
- `data` - The data of the document.
- `status` - The status of the request

We can use these values to display the data of the document in the React component in a reactive way.

```tsx
function UserComponent() {
  const { data, status } = useFetchUser();

  if (status === 'loading') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Error: {error.message}</p>;
  }

  return (
    <div>
      <p>{data.name}</p>
    </div>
  );
}
```

## Reading a document using the Firebase Admin SDK

While the API is similar, the Admin SDK is used in a different way. Indeed, the Admin SDK is used on the server, not on the client. This means that we can't use Reactfire to fetch data from the Firestore database.

The Admin SDK is used only in two cases:
1. When using an API Route
2. When using `getServerSideProps`

**Remember: The Admin SDK is not used in React components**. Similarly, the Web SDK is not used in API Routes or `getServerSideProps`.

### Reading the user from an API Route

Let's see how we can read a user document from an API Route.

Assuming we have a query parameter `userId` in the API Route, we can use the following code to read the user document from the Firestore database.

```ts
import { NextApiRequest, NextApiResponse } from 'next';
import { withAdmin as withFirebaseAdmin } from '~/core/middleware/with-admin';
import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  await withFirebaseAdmin();

  const { userId } = req.query;
  const firestore = getRestFirestore();
  const usersCollection = firestore.collection('users');
  const user = await usersCollection.doc(userId).get();

  res.status(200).json(user);
}
```

Let's break down the code above to understand what is happening.

#### 1. Importing the Firebase Admin SDK

First, we import the Firebase Admin SDK.

```ts
import { withAdmin as withFirebaseAdmin } from '~/core/middleware/with-admin';
import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';
```

The function `getRestFirestore` returns a Firebase Admin Firestore instance that uses `REST` instead of `gRPC`, because it's much faster in a serverless environment.

#### 2. Using the Firebase Admin SDK

We then use the Firebase Admin SDK to read the user document from the Firestore database.

```ts
await withFirebaseAdmin();

const { userId } = req.query;

const firestore = getRestFirestore();
const usersCollection = firestore.collection('users');
const user = await usersCollection.doc(userId).get();
```

We use the `withFirebaseAdmin` middleware to initialize the Firebase Admin SDK. This middleware must be called before using the Firebase Admin SDK.

We then use the `getRestFirestore` function to get a Firestore instance that uses `REST` instead of `gRPC`.

Finally, we use the Firestore instance to read the user document from the Firestore database.

#### 3. Returning the Data

Finally, we return the data of the user document as JSON.

```ts
res.status(200).json(user);
```

---


To learn more about the above, check out both the Firebase and Reactfire respective documentations.

-----------------
FILE PATH: kits\next-fire\how-to\data-fetching\reading-documents-list.mdoc

---
status: "published"

title: "Reading a list of Documents"
label: "Reading a list of Documents"
order: 1
description: "Learn how to read a list of documents from Firebase using both the Web SDK and the Admin SDK in your Next.js Firebase application"
---


## Reading a List of Documents with the Client SDK

The client SDK uses Reactfire to make it easier to fetch data directly from React components using React Hooks.

### Reading a List of Documents with "useFirestoreCollectionData"

In the example below, we will fetch a list of user organizations using the Firebase client SDK and Reactfire.

To do so, we use the React hook `useFirestoreCollectionData`, which allows us to stream real-time changes from a collection based on the filters provided.

```tsx
import {
  collection,
  where,
  query,
  CollectionReference,
} from 'firebase/firestore';

import { useFirestore, useFirestoreCollectionData } from 'reactfire';
import { Organization } from '~/lib/organizations/types/organization';
import { ORGANIZATIONS_COLLECTION } from '~/lib/firestore-collections';

export function useFetchUserOrganizations(userId: string) {
  const firestore = useFirestore();

  const organizationsCollection = collection(
    firestore,
    ORGANIZATIONS_COLLECTION
  ) as CollectionReference<WithId<Organization>>;

  const userPath = `members.${userId}`;
  const operator = '!=';

  // we query Firestore for all the organizations
  // where the user is a member, therefore where he path
  // members.<user_id> is not null
  const constraint = where(userPath, operator, null);
  const organizationsQuery = query(organizationsCollection, constraint);

  return useFirestoreCollectionData(organizationsQuery, {
    idField: `id`,
    initialData: [],
  });
}
```

Breaking down the code above:

- We first import the `useFirestoreCollectionData` hook from `reactfire` and the `collection`, `where` and `query` functions from `firebase/firestore`.
- We then use the `useFirestore` hook to get access to the Firestore instance.
- We then create a reference to the `organizations` collection.
- We then create a query to fetch all the organizations where the user is a member.
- Finally, we use the `useFirestoreCollectionData` hook to fetch the data from Firestore.

### Using the Data

The `useFirestoreCollectionData` hook returns an object with the following properties:
- `data`: the data returned from Firestore
- `status`: the status of the request

```tsx
function UserOrganizations({
  userId,
}: React.PropsWithChildren<{
  userId: string;
}>) {
  const { data, status } = useFetchUserOrganizations(userId);

  if (status === 'loading') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Error</p>;
  }

  return (
    <ul>
      {data.map((organization) => (
        <li key={organization.id}>{organization.name}</li>
      ))}
    </ul>
  );
}
```

## Using the Admin SDK

The Admin SDK is used to read data from the server-side.

### Reading a List of Documents with "getDocs"

In the example below, we will fetch a list of user organizations using the Firebase Admin SDK.

```tsx
import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';

export async function getOrganizationsForUser(userId: string) {
  const firestore = getRestFirestore();
  const organizationsCollection = firestore.collection('organizations');

  const result = await organizationsCollection
    .where(`members.${userId}`, '!=', null)
    .get();

  return result.docs.map((doc) => {
    return {
      ...doc.data(),
      members: [],
      id: doc.id,
      role: doc.data().members[userId].role,
    };
  });
}
```

You can use the function above in an API route or in a `getServerSideProps` function.

---


To learn more about the above, check out both the Firebase and Reactfire respective documentations.

-----------------
FILE PATH: kits\next-fire\how-to\data-mutations\creating-document.mdoc

---
status: "published"

title: "Creating a Document"
label: "Creating a Document"
order: 0
description: "In this tutorial we show how you can create a document in Firestore using the Web and Admin SDKs."
---


Creating a document in Firestore can be achieved using both the Web (Client) and the Admin SDK (Server). Depending on the use-case, you will use one or the other.

1. **Client SDK**: When you want to create a document from the browser, you will use the Client SDK. This is the most common use-case, and can be done when the user has the permissions required to create a document based on their security rules.
2. **Admin SDK**: When you want to create a document from a server, you will use the Admin SDK. This is useful when you want to create a document from a server, or when you want to create a document that the user does not have permission to create.

## Client SDK

To create a document, we will use the `addDoc` function from the Web Firebase SDK.

We can use hooks to make sure we have access to the Firestore instance, and the collection we want to add the document to.

```tsx
import { useFirestore } from 'reactfire';
import { useCallback } from 'react';
import { addDoc, collection } from 'firebase/firestore';

function useCreateTask() {
  const firestore = useFirestore();
  const tasksCollection = collection(firestore, `/tasks`);

  return useCallback(
    (task: {
      title: string;
      description: string;
      completed: boolean;
    }) => {
      return addDoc(tasksCollection, task);
    },
    [tasksCollection]
  );
}
```

We can then use this hook in our component to create a new task.

```tsx
import { useCallback } from "react";

const CreateTaskForm = () => {
  const createTask = useCreateTask();

  // ... other code

  const onSubmit = useCallback(async (task: {
    title: string;
    description: string;
    completed: boolean;
  }) => {
    await createTask(task);
  }, [createTask]);

  // ... other code
};
```

## Admin SDK

To create a document, we will use the `add` function from the Admin Firebase SDK.

We can use the Admin SDK only in API Route handlers and the `getServerSideProps` function. For two reasons:

1. The Admin SDK is for Node.js environments, and cannot be used in the browser.
2. The Admin SDK requires a service account, which should not be exposed to the browser.

```tsx
import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';

async function addTaskFromServer(task: {
  title: string;
  description: string;
  completed: boolean;
}) {
  const firestore = getRestFirestore();
  const tasksCollection = firestore.collection(firestore, `tasks`);

  await tasksCollection.add(task);

  return task;
}
```

You can now use the function `addTaskFromServer` in your API Route handlers and `getServerSideProps` functions.

-----------------
FILE PATH: kits\next-fire\how-to\data-mutations\deleting-document.mdoc

---
status: "published"

title: "Deleting a Document"
label: "Deleting a Document"
order: 2
description: "Learn how to delete a document in Firestore in your Next.js Firebase app"
---


Deleting a document in Firestore can be achieved using both the Web (Client) and the Admin SDK (Server). Depending on the use-case, you will use one or the other.

1. **Client SDK**: When you want to update a document from the browser, you will use the Client SDK. This is the most common use-case, and can be done when the user has the permissions required to delete a document based on their security rules.
2. **Admin SDK**: When you want to update a document from a server, you will use the Admin SDK. This is useful when you want to delete a document from a server, or when you want to delete a document that the user does not have permission to delete.

## Client SDK

To delete a document, we will use the `deleteDoc` function from the Web Firebase SDK.

We can use hooks to make sure we have access to the Firestore instance, and the collection we want to delete the document from.

```tsx
import { useFirestore } from 'reactfire';
import { useCallback } from 'react';
import { deleteDoc, doc } from "firebase/firestore";

function useDeleteTask() {
  const firestore = useFirestore();

  return useCallback(
    (taskId: string) => {
      const tasksDoc = doc(firestore, `/tasks`, taskId);

      return deleteDoc(tasksDoc);
    },
    [firestore]
  );
}
```

We can then use this hook in our component to create a new task.

```tsx
import { useCallback } from "react";

const DeleteTask = ({ id }: { id: string }) => {
  const deleteTask = useDeleteTask(id);

  // ... other code

  const onSubmit = useCallback(async (taskId: string) => {
    await deleteTask(taskId);
  }, [deleteTask]);

  // ... other code
};
```

## Admin SDK

To delete a Firestore document, we will use the `delete` function from the Admin Firebase SDK.

We can use the Admin SDK only in API Route handlers and the `getServerSideProps` function. For two reasons:

1. The Admin SDK is for Node.js environments, and cannot be used in the browser.
2. The Admin SDK requires a service account, which should not be exposed to the browser.

```tsx
import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';

async function deleteTaskFromServer(task: {
  id: string;
}) {
  const firestore = getRestFirestore();
  const taskRef = firestore.collection(firestore, `tasks`).doc(task.id);

  await taskRef.delete();

  return task;
}
```

You can now use the function `deleteTaskFromServer` in your API Route handlers and `getServerSideProps` functions.

-----------------
FILE PATH: kits\next-fire\how-to\data-mutations\updating-document.mdoc

---
status: "published"

title: "Updating a Document"
label: "Updating a Document"
order: 1
description: "Learn how to update a document in Firestore in your Next.js Firebase app"
---


Updating a document in Firestore can be achieved using both the Web (Client) and the Admin SDK (Server). Depending on the use-case, you will use one or the other.

1. **Client SDK**: When you want to update a document from the browser, you will use the Client SDK. This is the most common use-case, and can be done when the user has the permissions required to update a document based on their security rules.
2. **Admin SDK**: When you want to update a document from a server, you will use the Admin SDK. This is useful when you want to update a document from a server, or when you want to update a document that the user does not have permission to update.

## Client SDK

To update a Firestore document, we will use the `addDoc` function from the Web Firebase SDK.

We can use hooks to make sure we have access to the Firestore instance, and the collection we want to add the document to.

```tsx
import { useFirestore } from 'reactfire';
import { useCallback } from 'react';
import { updateDoc, doc } from "firebase/firestore";

function useUpdateTask(id: string) {
  const firestore = useFirestore();
  const tasksDoc = doc(firestore, `/tasks`, id);

  return useCallback(
    (task: {
      title: string;
      description: string;
      completed: boolean;
    }) => {
      return updateDoc(tasksDoc, task);
    },
    [tasksDoc]
  );
}
```

We can then use this hook in our component to create a new task.

```tsx
import { useCallback } from "react";

const UpdateTaskForm = ({ id }: { id: string }) => {
  const updateTask = useUpdateTask(id);

  // ... other code

  const onSubmit = useCallback(async (task: {
    title: string;
    description: string;
    completed: boolean;
  }) => {
    await updateTask(task);
  }, [updateTask]);

  // ... other code
};
```

## Admin SDK

To create a document, we will use the `update` function from the Admin Firebase SDK.

We can use the Admin SDK only in API Route handlers and the `getServerSideProps` function. For two reasons:

1. The Admin SDK is for Node.js environments, and cannot be used in the browser.
2. The Admin SDK requires a service account, which should not be exposed to the browser.

```tsx
import getRestFirestore from '~/core/firebase/admin/get-rest-firestore';

async function updateTaskFromServer(task: {
  id: string;
  title: string;
  description: string;
  completed: boolean;
}) {
  const firestore = getRestFirestore();
  const taskRef = firestore.collection(firestore, `tasks`).doc(task.id);

  await taskRef.update(task);

  return task;
}
```

You can now use the function `updateTaskFromServer` in your API Route handlers and `getServerSideProps` functions.

-----------------
FILE PATH: kits\next-fire\how-to\payments\configuring-plans.mdoc

---
status: "published"
title: "Configuring your SaaS Stripe Plans in your Makerkit Next.js Firebase App"
label: Configuring Plans
order: 0
description: "Learn how to configure the Stripe plans in your SaaS application with Makerkit"
---

When taking payments from Stripe, you're required to configure your subscription's plans.

Your plans need to be configured in two places:

1. **Stripe** - first, you need to define your **products** and **plans** in Stripe itself. This requires you to have an account in Stripe.
2. **Configuration**: - then, you will copy your plans data in the Makerkit configuration, which is used to render the Pricing table and display the prices metadata

### Pricing Table Configuration

Below is the default Pricing Table configuration of the kits. The `PricingTable` component will automatically generate the pricing table based on the Stripe plans you have created and added to the configuration (see below).

We have 3 products (Basic, Pro and Premium), each with 2 plans (monthly, yearly). By default, the Pro plan is set as the recommended plan.

Of course, the below is simply an example. You will need to customize this according to your application's plans.

```tsx
stripe: {
  products: [
    {
      name: 'Basic',
      description: 'Description of your Basic plan',
      badge: `Up to 20 users`,
      features: [
        'Basic Reporting',
        'Up to 20 users',
        '1GB for each user',
        'Chat Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$9',
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$90',
          stripePriceId: '<price_id>',
          trialPeriodDays: 7,
        },
      ],
    },
    {
      name: 'Pro',
      badge: `Most Popular`,
      recommended: true,
      description: 'Description of your Pro plan',
      features: [
        'Advanced Reporting',
        'Up to 50 users',
        '5GB for each user',
        'Chat and Phone Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$29',
          stripePriceId: 'pro-plan-mth',
          trialPeriodDays: 7,
        },
        {
          name: 'Yearly',
          price: '$200',
          stripePriceId: 'pro-plan-yr'
        },
      ],
    },
    {
      name: 'Premium',
      description: 'Description of your Premium plan',
      badge: ``,
      features: [
        'Advanced Reporting',
        'Unlimited users',
        '50GB for each user',
        'Account Manager',
      ],
      plans: [
        {
          name: '',
          price: 'Contact us',
          stripePriceId: '',
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ],
}
```

#### Update the Stripe Price IDs values!

Please replace the property `stripePriceId` with the real price ID from the
Stripe Dashboard.

<div>
  {% alert type="warning" title="Update your real Stripe Price IDs" %}

         The properties "stripePriceId" are placeholders. You will need to replace them with the IDs of your plans.

{% /alert %}
</div>

### Additional Configuration

You can also configure the following properties on each `product`:

- `recommended` - set to `true` if you want to highlight a plan as recommended
- `badge` - set to a string if you want to display a badge next to the plan name

You can also configure the following properties on each `plan`:

- `price` - any string that you want to display as the price (e.g. `$9.99` or `Free`)
- `label` - set to a custom string if you want to display a button label
- `href` - set to a custom URL if you want to link to a custom page (for example, a contact page)
- `trialPeriodDays` - set to the number of days you want to offer a free trial for


-----------------
FILE PATH: kits\next-fire\how-to\payments\stripe-webhooks-locally.mdoc

---
status: "published"
title: "Running the Stripe Webhook locally"
label: Running the Stripe Webhook locally
order: 1
description: "Want to test and run the Stripe Webhook locally? This guide will show you how to do it."
---

When testing webhooks sent by Stripe, it's useful to redirect them to our local webhook endpoint so that we can test them locally.

Makerkit has a built-in NPM script that starts the Stripe CLI and redirects webhooks to our local endpoint.

## Prerequisites

The only prerequisite is to have Docker installed and running on your machine.

## Running the Stripe Webhook locally

To run the Stripe Webhook locally, run the following command:

```bash
npm run stripe:listen
```

The first time you run this command, it will ask you to login to your Stripe account. Follow the instructions on the screen to do so.

Once you're logged in, run the command again:

```bash
npm run stripe:listen
```

This time, the Stripe CLI will start and redirect webhooks to our local endpoint.

You can now test your webhooks locally when testing your Stripe integration with Makerkit 🎉


-----------------
FILE PATH: kits\next-fire\how-to\payments\using-lemon-squeezy.mdoc

---
status: "published"
title: "Using Lemon Squeezy instead of Stripe in your Next.js Firebase application"
label: Using Lemon Squeezy instead of Stripe
order: 4
description: "How to use Lemon Squeezy instead of Stripe in your Next.js Firebase application"
---

To use Lemon Squeezy instead of Stripe, you should use the branch `main-ls` to develop your SaaS.

The Lemon Squeezy configuration is slightly different from the Stripe configuration as it accepts a variant ID instead of a price ID.

```tsx
subscriptions: {
  plans: [
    {
      name: 'Basic',
      description: 'Description of your Basic plan',
      badge: `Up to 20 users`,
      features: [
        'Basic Reporting',
        'Up to 20 users',
        '1GB for each user',
        'Chat Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$9',
          variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$90',
           variantID: '<your variant ID>',
        },
      ],
    },
    {
      name: 'Pro',
      badge: `Most Popular`,
      recommended: true,
      description: 'Description of your Pro plan',
      features: [
        'Advanced Reporting',
        'Up to 50 users',
        '5GB for each user',
        'Chat and Phone Support',
      ],
      plans: [
        {
          name: 'Monthly',
          price: '$29',
           variantID: '<your variant ID>',
        },
        {
          name: 'Yearly',
          price: '$200',
          variantID: '<your variant ID>',
        },
      ],
    },
    {
      name: 'Premium',
      description: 'Description of your Premium plan',
      badge: ``,
      features: [
        'Advanced Reporting',
        'Unlimited users',
        '50GB for each user',
        'Account Manager',
      ],
      plans: [
        {
          name: '',
          price: 'Contact us',
          label: `Contact us`,
          href: `/contact`,
        },
      ],
    },
  ]
}
```

Please refer to the [complete Lemon Squeezy documentation](/docs/next-fire/lemon-squeezy) for more details.

-----------------
FILE PATH: kits\next-fire\how-to\setup\branding.mdoc

---
status: "published"

title: "Update your Makerkit application Branding"
label: "Branding"
order: 1
description: "A unique branding is essential to any SaaS. Learn how to update your Makerkit application branding."
---


Most of the branding details of your SaaS will be stored in the main configuration file of your application. This file is located at `src/configurations.ts`.

The `site` object contains most of your branding details. You can update the following properties:

```tsx
site: {
  name: 'Awesomely - Your SaaS Title',
  description: 'Your SaaS Description',
  themeColor: '#ffffff',
  themeColorDark: '#0a0a0a',
  siteUrl: process.env.NEXT_PUBLIC_SITE_URL,
  siteName: 'Awesomely',
  twitterHandle: '',
  githubHandle: '',
  language: 'en',
  convertKitFormId: '',
  locale: process.env.NEXT_PUBLIC_DEFAULT_LOCALE,
},
```

1. The `name` and `description` properties will be used in the meta tags of your application
2. The `themeColor` and `themeColorDark` properties will be used to update the color of the browser bar on mobile devices.
3. The `siteName` property is a short version of your SaaS title, without the subtitle.
4. The `twitterHandle` and `githubHandle` properties will be used to link to your social media accounts (if you have any).
5. The `locale` property is the default language of your application
6. The `language` property is deprecated and will be removed in a future version of Makerkit. Please use the `locale` property instead.
7. The `convertKitFormId` property is used to link your application to your ConvertKit account if you use the Newsletter form.

## Design Branding

Please refer to the [Theming](theming) section to learn how to update the design of your application.

