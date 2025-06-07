-----------------
FILE PATH: kits/next-fire/data-fetching/custom-fetch.mdoc

---
status: "published"

title: "Writing your own Fetch"
label: "Writing your own Fetch"
order: 5
description: "Guide to writing your own fetch implementation"
---

By default, MakerKit uses a custom hook that wraps the browser's `fetch` to make it easier to use with React's components named `useApiRequest`. 

Additionally, this custom hook automatically adds two headers for security reasons: 
- `X-Firebase-AppCheck` - which is the token generated when using **Firebase AppCheck**
- `x-csrf-token` - the page's CSRF token generated when the page is server-rendered. It's needed to protect the page from CSRF attacks.

Since the `useApiRequest` hook is pretty basic, if you make complex queries to your API, you may want to use a more complete implementation such as `react-query` or `swc`. However, If you don't make complex queries, it can be unnecessary since most of your queries will be to Firestore, which uses its own client-side data-fetching method.

To make these libraries seamlessly work with the API, you only need to consider the headers above if the API uses them to protect your application's endpoints.

### Generating an App Check Token

To generate an App Check token, you can use the `useGetAppCheckToken` hook:

```tsx
const getAppCheckToken = useGetAppCheckToken();
const appCheckToken = await getAppCheckToken();

console.log(appCheckToken) // token
```

You will need to send a header `X-Firebase-AppCheck` with the resolved value returned by the promise `getAppCheckToken()`.

### Sending the CSRF Token

To retrieve the page's CSRF token, you can use the `useGetCsrfToken` hook:

```tsx
const getCsrfToken = useGetCsrfToken();
const csrfToken = getCsrfToken();

console.log(csrfToken) // token
```

You will need to send a header `x-csrf-token` with the value returned by `getCsrfToken()`.

-----------------
FILE PATH: kits/next-fire/data-fetching/fetching-data-firestore.mdoc

---
status: "published"

title: "Fetching data from Firestore"
label: "Fetching data from Firestore"
order: 1
description: "Learn how fetch data from your Firestore database in Makerkit"
---


When fetching data from Firestore, we use the hooks provided by `reactfire`,
although you could also use plain functions from the Firebase SDK if you
preferred.

To make data-fetching from Firestore more reusable, our convention is to
create a custom React hook for each query we make.

For example, let's take a look at the hook to fetch an organization from
Firestore:

 ```tsx {% title="use-fetch-organization.ts" %}
import { useFirestore, useFirestoreDocData } from 'reactfire';
import { doc, DocumentReference } from 'firebase/firestore';
import { Organization } from '~/lib/organizations/types/organization';
import { ORGANIZATIONS_COLLECTION } from '~/lib/firestore-collections';

type Response = WithId<Organization>;

export function useFetchOrganization(
  organizationId: string
) {
  const firestore = useFirestore();

  const ref = doc(
    firestore,
    ORGANIZATIONS_COLLECTION,
    organizationId
  ) as DocumentReference<Response>;

  return useFirestoreDocData(ref, { idField: 'id' });
}

export default useFetchOrganization;
```

Let's explain the above:

1. First, we get the Firestore instance using `useFirestore`
2. We create a Firestore reference to the path of the organization collection
3. Strong-type the document reference `as DocumentReference<Response>` so
that Typescript will correctly infer the following usages of the reference
4. Finally, we request the data using the hook `useFirestoreDocData,` which will
 return the record's data.

NB: Additionally, we tell Firestore to
add the `id` of the field to the object's dat using `{ idField: 'id' }`.
This is why the interface returned is `WithId<Organization>`. We're
basically telling Typescript that the data returned is the combination of
the following interfaces:

```tsx
interface WithId {
  id: string;
}

interface Organization {
  // ...
}
```

This is very useful, as often times you will need the IDs of your Firestore
documents on the client-side.

When you create a new Firestore collection, remember to:
1. Add the required Firestore rules
2. Store and read the name of the collection from
`~/lib/firestore-collections`:
this helps you avoid magic strings when typing your collection names
3. Create hooks to fetch data. Usually, we store these in `lib/<entity>/hooks`

### Consuming data from the Firestore hooks

Reactfire's custom hooks help us fetch data from Firestore effortlessly, but
we need to pay attention when using them due to their asynchronous nature.

For example, let's see how we can use the hook above correctly by displaying
a different UI based on the status of the hook:

```tsx
import { useFetchOrganization } from './use-fetch-organization';

function OrganizationCard({ organizationId }) {
  const {
    data: organization,
    status,
  } = useFetchOrganization(organizationId);

  /* data is loading */
  if (status === `loading`) {
    return <div>Loading...</div>
  }

  /* request errored */
  if (status === `error`) {
    return <div>Error!</div>
  }

  /* request successful, we can access "organization" */
  return <div>{organization.name}</div>;
}
```

### Fetching data from a collection using a Firestore query

Fetching data from a collection using a query is a common scenario.

For example, we assume you have a collection named `tasks`, with a foreign key
`organizationId` represents the ID of the organization to which it belongs.

Of course, you will want to write a query to fetch a list of tasks for your
organization.

When we define our entity, we will need to add a key `organizationId` so we
can match it against the ID of the organization it belongs to. For example,
take a look at the `Task` interface below:

```tsx
export interface Task {
  name: string;
  description: string;
  dueDate: string;
  done: boolean;

  // here we use organizationId as a foreign key
  organizationId: string;
}
```

Now, we can write a hook that gets a list of `tasks` that have
`organizationId` equal to the one we pass:

```tsx
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

### Security Rules

What about securing our Firestore database? We will use Firestore's security
rules. During development, you can update the `firestore.rules` file in your
root folder, but remember that you need to propagate these to your Firebase
instance using the console.

The functions `userIsMemberByOrganizationId` and `isSignedIn` are added by
default to the Makerkit's starter:

 ```js {% title="firestore.rules" %}
match /tasks/{taskId} {
  allow create: if isSignedIn();
  allow read, update, delete: if userIsMemberByOrganizationId(existingData().organizationId);
}
```

## Strong Typing

When reading data, it's necessary to strong-type your queries. While the
Firebase SDK isn't exactly type-friendly in some situations; we can make it
work by casting references to the correct type.

### Casting Documents References

When casting Firestore documents, you can use `as DocumentReference<T>`:

```tsx
const organizationRef = doc(
  firestore,
  ORGANIZATIONS_COLLECTION,
  organizationId
) as DocumentReference<WithId<Organization>>;
```

When getting the response from Firestore, the type will be correctly
inferred as `WithId<Organization>`.

### Casting Collections References

When casting Firestore collections, you can use `as CollectionReference<T>`:

```tsx
const collectionRef = collection(
  firestore,
  tasksCollection
) as CollectionReference<WithId<Task>>;
```


-----------------
FILE PATH: kits/next-fire/data-fetching/making-api-requests.mdoc

---
status: "published"

title: "API requests"
label: "API requests"
order: 2
description: "Learn how to make requests to your Next.js Firebase API in Makerkit"
---

To make requests to the API functions in the `api` folder, we provide a
custom hook `useApiRequest`, a wrapper around `fetch`.

It has the following functionality:

1. Automatically adds a `Firebase AppCheck Token` to the request headers if you
enabled Firebase AppCheck
2. Automatically adds a `CSRF token` to the request headers

We use this in conjunction with the [swr](https://swr.vercel.app/) to simplify its usage using React components.

 ```tsx {% title="src/core/hooks/use-create-session.ts" %}
import useApiRequest from '~/core/hooks/use-api';

interface Body {
  idToken: string;
}

export function useCreateSession() {
  const endpoint = '/api/session/sign-in';
  const fetcher = useApiRequest<void, Body>();

  return useSWRMutation(endpoint, (path, { arg }) => {
    return fetcher({
      path,
      body: arg,
    });
  });
}
```

You can use this hook in your components:

```tsx
import { useCreateSession } from '~/core/hooks/use-create-session';

function Component() {
  const { trigger, ...mutationState } = useCreateSession();

  return (
    <>
      { mutationState.isMutating ? `Mutating...` : null }
      { mutationState.error ? `Error :(` : null }
      { mutationState.data ? `Yay, success!` : null }

      <SignInForm onSignIn={(idToken) => trigger({ idToken })} />
    </>
  );
}
```

-----------------
FILE PATH: kits/next-fire/data-fetching/reading-data-storage.mdoc

---
status: "published"

title: "Reading data from Storage"
label: "Reading data from Storage"
order: 3
---

## 1) Add the Storage Provider

First, we need to wrap the component using Firebase Storage with the
`FirebaseStorageProvider`, which is responsible for initializing the
Firebase Storage SDK.

```tsx
import FirebaseStorageProvider from '~/core/firebase/components/FirebaseStorageProvider';

<FirebaseStorageProvider>
  <ComponentThatUsesStorage />
</FirebaseStorageProvider>
```

## 2) Create a Hook to fetch data from Storage

Let's assume we want to fetch a list of files from storage by getting their
`url`. This is a common scenario for retrieving images stored in Firebase
Storage.

```tsx
function useOrganizationAssets() {
  const storage = useStorage();

  const { setData, setError, setLoading, state } =
    useRequestState<MediaItem[]>();

  const path = `/${organizationId}/uploads`;
  const reference = ref(storage, path);

  useEffect(() => {
    void (async () => {
      try {
        const result = await list(reference);

        const items = await Promise.all(
          result.items.map(async (item) => {
            const url = await getDownloadURL(item);

            return url;
          })
        );

        setData(items);
      } catch (e) {
        setError(e);
      }
    })();
  }, [reference, setData, setError]);

  return state;
}
```

## 3) Use the custom hook in your components

And then we can use the hook in our components:

```tsx
function MyImages() {
  const { data, loading, error } = useOrganizationAssets();

  if (loading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>We could not fetch your images :(</p>;
  }

  return (
    <div className={'flex flex-col space-y-2'}>
      {data.map(image => {
        return <img src={image} key={image} />
      })}
    </div>
  );
}
```


-----------------
FILE PATH: kits/next-fire/data-fetching/uploading-data-to-storage.mdoc

---
status: "published"
title: "Uploading data to Storage | Next.js Firebase SaaS Kit"
description: "Learn how to upload your data to Firebase Storage"
label: "Uploading data to Storage"
order: 4
---

When uploading data to storage, the data needs to be secured so that it
can only be accessed by the organization that created it; we need to add the
organization ID to the file's custom data.

By doing so, we will be able to
compare the custom data ID with the user's organization ID, and verify they
match.

## 1) Add the Storage Provider

First, we need to wrap the component using Firebase Storage with the
`FirebaseStorageProvider`, which is responsible for initializing the
Firebase Storage SDK.

```tsx
import FirebaseStorageProvider from '~/core/firebase/components/FirebaseStorageProvider';

<FirebaseStorageProvider>
  <ComponentThatUsesStorage />
</FirebaseStorageProvider>
```

## 2) Using the Firebase Storage SDK

Now we can use the Firebase Storage SDK by using the `useStorage` hook. From
Here, we can simply use the SDK described by the Firebase documentation.

```tsx
const storage = useStorage();
```

Let's assume we have a function `uploadFile` that receives a `File`
parameter, and that we need to upload to Firebase Storage:

```tsx
import { ref, uploadBytes } from 'firebase/storage';
import { toast } from 'sonner';
import { useStorage } from 'reactfire';
import { useCurrentOrganization } from './use-current-organization';

function Component() {
  const storage = useStorage();
  const organization = useCurrentOrganization();

  async function uploadFile(file: File) {
    if (!organization) return;

    const organizationId = organization.id;
    const path = `/${organizationId}/uploads/${file.name}`;
    const reference = ref(storage, path);

    const promise = uploadBytes(reference, file, {
      cacheControl: 'max-age=31536000',
      customMetadata: {
        organizationId,
      },
    });

    toast.promise(promise, {
      success: `Yay, uploaded!`,
      loading: `Uploading...`,
      error: `Error :(`
    });
  }

  return <UploadForm onFileChosen={uploadFile} />
}
```

## 3) Security Rules for Storage

As you can see from the code above, we are setting some custom metadata when we
upload the file:

```ts
customMetadata: {
  organizationId,
},
```
By doing this, we're effectively telling the file to store `organizationId`
as metadata on the file.

This will help us with the Storage security rules
for disallowing users in other organizations from reading the file:

```js
match /organizations/{organizationId}/{fileName=**} {
  allow read: if resource.metadata.organizationId == request.auth.token.organizationId;
  allow write: if request.auth.token.organizationId == organizationId;
}
```


-----------------
FILE PATH: kits/next-fire/data-fetching/writing-data-to-firestore.mdoc

---
status: "published"

title: "Writing data to Firestore"
label: "Writing data to Firestore"
order: 0
description: "Learn how to write, update and delete data using Firestore in
Makerkit"
---

Similarly to reading data from Firestore, we may want to create a `custom
hook`` also when we write, update or delete data.

Assuming we have an entity called `tasks`, and we want to create a hook to
create a new `Task`, we can do the following.

1. First, we create a new hook at `lib/tasks/hooks/use-create-task.ts`
2. We add the `tasks` collection name to `lib/firestore-collections`
3. Wrap the `addDoc` function from Firestore within a `useCallback` hook

 ```tsx {% title="lib/tasks/hooks/use-create-task.ts" %}
import { TASKS_COLLECTION } from '~/lib/firestore-collections';

function useCreateTask() {
  const firestore = useFirestore();
  const tasksCollection = collection(firestore, TASKS_COLLECTION);

  return useCallback(
    (task: Task) => {
      return addDoc(tasksCollection, task);
    },
    [tasksCollection]
  );
}
```

And now, we could use this hook within a component:

```tsx
function Component() {
  const createTask = useCreateTask();

  return <MyForm onSubmit={task => createTask(task)} />
}
```

Let's take a look at a complete example of a form that makes a request using
the hook above:

 ```tsx {% title="components/tasks/NewTaskForm.tsx" %}
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
      const dueDate = data.get('dueDate') as string;

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

      await toaster.promise(promise, {
        success: `Task created!`,
        error: `Ops, error!`,
        loading: `Creating task...`,
      });
    },
    [router, createTask, organizationId, setLoading]
  );

  return (
    <div>
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
            <TextField.Input
              required
              defaultValue={getDefaultDueDate()}
              name={'dueDate'}
              type={'date'}
            />
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
  const year = date.getFullYear();
  const month = pad(date.getMonth() + 1);
  const day = pad(date.getDate() + 1);

  return `${year}-${month}-${day}`;
}

function pad(n: number) {
  return n < 10 ? `0${n}` : n.toString();
}

export default CreateTaskForm;
```


-----------------
FILE PATH: kits/next-fire/data-fetching.mdoc

---
status: "published"
title: "Data Fetching in Next.js Firebase"
label: "Data Fetching"
order: 5
description: "Everything you need to know about data fetching in Next.js Firebase"
collapsible: true
collapsed: true
---

In this section, you will learn everything you need to know about data fetching in Next.js Firebase.

- [Fetching data from Firestore](data-fetching/fetching-data-firestore): this is the guide to fetch data from Firestore
- [Fetching data from the Firebase Storage](data-fetching/fetching-data-from-storage): this is the guide to fetch data from Firebase Storage
- [Writing data to Firestore](data-fetching/writing-data-to-firestore): this is the guide to write data to Firestore
- [Reading Data from Firebase Storage](data-fetching/reading-data-storage): this is the guide to read data from Firebase Storage
- [Uploading data from Firebase Storage](data-fetching/uploading-data-storage): this is the guide to upload data to Firebase Storage
- [Writing your own Fetch](data-fetching/writing-data-to-firestore): this is the guide to write your own fetch implementation

-----------------
FILE PATH: kits/next-fire/development/api-routes.mdoc

---
status: "published"

title: "Writing a Next.js API route with Makerkit"
label: Writing API Routes
order: 7
description: "Makerkit offers utilities to reduce the boilerplate needed to
write Next.js API routes. Learn the techniques to write well-coded API
handlers."
---

Makerkit provides various utilities to reduce the boilerplate needed to write API functions with Next.js.

The simpler API route you could write is the following:

```tsx
export default function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  return res.send(`Hello World`)
}
```

Of course, there isn't much in there! But any SaaS will need various checks
before executing an handler, for example:

1. checking if the request is authenticated
2. validating the request isn't from a bot
3. validating the payload matches your expected schema
4. validating the endpoint matches the expected method
5. ...and so on!

Thankfully, Makerkit allows you to do all the above thanks to `middlewares`,
simple function that accept the Next,js `req` and `res` API parameters.

Normally, as you will see in the codebase, we writeAPI handlers
using the conventions below:

 ```tsx {% title="/pages/api/members.ts" %}
import { withAuthedUser } from '~/core/middleware/with-authed-user';
import { withPipe } from '~/core/middleware/with-pipe';
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';
import { withExceptionFilter } from '~/core/middleware/with-exception-filter';
import { withAppCheck } from '~/core/middleware/with-app-check';
import { withAdmin } from '~/core/middleware/with-admin';

const SUPPORTED_METHODS: HttpMethod[] = ['GET', 'POST'];

export default function members(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    // throw if method is not in SUPPORTED_METHODS array
    withMethodsGuard(SUPPORTED_METHODS),
    // initialize Firebase Admin
    withAdmin,
    // check request is genuine with Firebase AppCheck
    withAppCheck,
    // check user is authenticated
    withAuthedUser,
    // execute API logic
    membersHandler
  );

  // manage exceptions
  return withExceptionFilter(req, res)(handler);
}

async function membersHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const members = await fetchOrganizationMembers();

  res.json(members);
}
```

Let's take a look at these functions individually.

### Piping functions

Piping functions can be useful to augment a function, validate data, or do any other check before getting to the function handler, where we write the bulk of the logic.

The utility we use is `withPipe`, a dead-simple function that iterates over functions until one of them stops (or calls functions such as `req.end()`, `req.send()`, `req.json()`).

```tsx
export default withPipe(
  (req, res) => {
    if (req.method !== 'POST') {
      res.status(409).end();
    }
  },
  (req, res) => {
    // this will never execute!
  }
)
```

### Guarding against unsupported HTTP methods

This utility helps reject requests whose method is not defined in the array.

For example, let's assume we only want to reply to `GET` and `POST` requests:

 ```tsx {% title="/api/hello.ts" %}
import { withMethodsGuard } from '~/core/middleware/with-methods-guard';

export default withPipe(
  withMethodsGuard(['GET', 'POST'])
);
```

Alternatively, call it at the top of an API route:

```tsx
export default async function apiHandler() {
  await withMethodsGuard(['PUT']);
}
```

If the API is called with any other HTTP method, this function will throw a 405 error and add an `Allow` HTTP header with the list of allowed methods.

### Initializing the Firebase Admin

To be able to use the Firebase Admin functions in your Next.js API, you need to initialize it before calling its services, such as Firestore, Auth, etc.

```tsx
import { withAdmin } from '~/core/middleware/with-admin';

export default withPipe(
  withAdmin,
);
```

Alternatively, call it at the top of an API route:

```tsx
export default async function apiHandler() {
  await withAdmin();
}
```

### Rejecting requests from non-authenticated users

This is one of the most important functions you will be using, i.e. ensuring the API request is coming from an authenticated user:

```tsx
import { withAuthedUser } from '~/core/middleware/with-authed-user';

export default withPipe(
  withAuthedUser,
);
```

Alternatively, call it at the top of an API route:

```tsx
export default async function apiHandler() {
  await withAuthedUser();
}
```

### Rejecting requests from bots

If you enabled AppCheck in your application, you can disallow requests from bad actors and spammy clients:

```tsx
import { withAppCheck } from '~/core/middleware/with-app-check';

export default withPipe(
  withAdmin,
  withAppCheck,
);
```

Alternatively, call it at the top of an API route:

```tsx
export default async function apiHandler() {
  await withAdmin();
  await withAppCheck();
}
```

### Handling Exceptions

The utility `withExceptionFilter` is useful for managing exceptions from an API handler.

This utility will report the exception to `Sentry.io` (if configured), log the exception so that you can debug it, and return a JSON with some metadata to **avoid leaking information to the clients**: therefore, I highly encourage you to use this utility where possible.

Check out the example below:

```tsx
export default function apiHandler() {
  const handler = withPipe(
    withMethodsGuard(SUPPORTED_METHODS),
    withAuthedUser,
    (req, res) => {
      return await getData();
    }
  );

  // manage exceptions
  return withExceptionFilter(req, res)(handler);
}
```


-----------------
FILE PATH: kits/next-fire/development/building-features.mdoc

---
status: "published"

title: "Building features for your Next.js Makerkit application"
label: Building Features
order: 0
description: "Learn the flow and patterns that you can use to build new features"
---

As we've mentioned before, the Starter Kit uses various conventions around how the code is split.

While you're free to use your own conventions, in this post I want to describe how to build new features using the existing conventions of the Next.js and Firebase kit.

Let's take a high-level overview of how we can go about adding new features. In this example, we want to add "events" to our application:

### 1) Adding the business logic

First, we create a new folder within `lib`. For example, such as `lib/events`. This folder will contain all the code related to events.

In this folder, we will add types, hooks, utilities, and **anything
regarding the domain of this feature**.

For example, let's assume we want to write a feature that allows users to fetch and list `events`. We will create a `lib/events/hooks/use-fetch-events.ts` file that will contain the hook for fetching a list of events from Firestore.

 ```tsx {% title="lib/events/hooks/use-fetch-events.ts" %}
export default function useFetchEvents() {
  const firestore = useFirestore();
  const eventsCollection = 'events';

  const collectionRef = collection(
    firestore,
    eventsCollection
  ) as CollectionReference<WithId<Task>>;

  const path = `organizationId`;
  const operator = '==';
  const constraint = where(path, operator, organizationId);
  const organizationsQuery = query(collectionRef, constraint);

  return useFirestoreCollectionData(organizationsQuery, {
    idField: 'id',
  });
}
```

We will now use the hook above within our React components to fetch the events.

### 2) Adding the "events" components 

We will add a new folder within `src/components` called `events`. This folder will contain all the components related to events.

For example, `EventsContainer.tsx`, `EventDetail.tsx` and so on. These components will be imported within our Next.js pages components.

 ```tsx {% title="components/events/EventsContainer.tsx" %}
export default function EventsContainer() {
  const { data, status } = useFetchEvents();

  if (status === `loading`) {
    return <Loading />;
  }

  if (status === `error`) {
    return <Error />;
  }

  return (
    <div>
      {data.map((event) => (
        <EventDetail key={event.id} event={event} />
      ))}
    </div>
  );
}
```

### 3) Finally, we add the events Next.js Pages

Finally, **we need to add the pages to our Next.js application**. We can do this by adding the events pages to `pages/events/**.tsx`. 

The page components will then import the components as building blocks from `components/events`. 

 ```tsx {% title="pages/events/page.tsx" %}
export default function EventsPage() {
  return (
    <RouteShell>
      <EventsContainer />
    </RouteShell>
  );
}

export function getServerSideProps(ctx: GetServerSidePropsContext) {
  return withAppProps(ctx);
}
```

### 4) Using API Requests

Let's now assume you need to **use an API request** to fetch your data; since this can be due to various factors:

1. You need to fetch data from an external API
2. You need to fetch data from a different database
3. You need to fetch from Firestore that requires elevated permissions
4. You need to fetch from particularly complex Firestore queries

These are all good reasons! The flow explained above still makes a lot of sense, but the convention used by the kit is slightly different. 

In fact, we store the server queries within `~lib/server/events`. Of course, you can also add these below your domain `~/lib/events/server`. The choice is yours.

 ```tsx {% title="lib/server/events/fetch-events.ts" %}
export default function fetchEvents() {
  return makeExternalDatabaseRequest();
}
```

Now that we have a function to retrieve data from the external service, we can finally create an API handler to call this function.

 ```tsx {% title="pages/api/events.ts" %}
export default async function eventsHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const events = await fetchEvents();

  res.status(200).json(events);
}
```

From the client side, we can use the hook `useApiRequest` to call the API handler.

 ```tsx {% title="lib/events/hooks/use-fetch-events.ts" %}
import useQuery from 'swr';

export default function useFetchEventsFromApi() {
  const fetcher = useApiRequest();

  return useQuery([`events`], () => fetcher(`/api/events`));
}
```

Finally, we can use the hook within our React components.

 ```tsx {% title="components/events/EventsContainer.tsx" %}
export default function EventsContainer() {
  const { data, loading, error } = useFetchEventsFromApi();

  if (loading) {
    return <Loading />;
  }

  if (error) {
    return <Error />;
  }

  return (
    <div>
      {data.map((event) => (
        <EventDetail key={event.id} event={event} />
      ))}
    </div>
  );
}
```

-----------------
FILE PATH: kits/next-fire/development/code-style.mdoc

---
status: "published"
title: "Adjust the codebase style according to your preferences | Next.js Firebase Kit"
label: Code Style
order: 2
---

The Makerkit's boilerplate codebase is formatted with *Prettier*. You can configure it as you wish by updating the file `.prettierrc` you find in the project's root.

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


