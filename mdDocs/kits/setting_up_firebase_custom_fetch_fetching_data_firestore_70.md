-----------------
FILE PATH: kits\remix-fire\configuration\setting-up-firebase.mdoc

---
status: "published"

title: Setting up a Firebase project for your MakerKit application
label: Setting up your Firebase Project
order: 1
description: 'Learn how to set up your Firebase project with a MakerKit application'
---


You should be all set if you have already installed NodeJS on your machine. If not, please install [NodeJS](https://nodejs.org) before continuing.

If any of the following commands do not work, please install the Firebase tools package globally:

```
npm i firebase-tools -g
```

Now, run this command to set up your Firebase project:

```
firebase init
```

Now, select all the products you would like to set up. If you're not sure yet, I recommend the following ones:
- Emulators
- Cloud Firestore
- Cloud Storage
- Remote Config

{% alert type="info" %}
You do not need any changes to start developing your application. Feel free
to complete the configuration once you want to deploy the app for the first
time.
{% /alert %}

By default, Makerkit starts the application with a demo project
`demo-makerkit`.

You can rename the demo name however you want, as long as you keep the
prefix "demo", which tells Firebase not to use any production service and
not to require additional configuration for developing the application. In
fact, you do not need to create a Firebase project to get started!

Additionally, remember to change the project in the environment variables.

**If you also want to create a real Firebase project, please read on.**

#### Create or select a Project

If you have already created a project using the Firebase Console, continue by
using "Use an existing project"; otherwise, you can create one right away by
selecting "Create a new project".

If you're more comfortable creating a new project using the Firebase Console,
 [create a new project from here](https://console.firebase.google.com/).

#### Emulators

The CLI should prompt you to choose the emulators to download and the ports for each. Next, select the emulators you need based on the products selected during the first step.

Otherwise, you don't need to make any changes because this starter already uses the Firebase default ports.

Once completed, you should have the following new files in your project:
- `.firebaserc`
- `firebase.json`
- `firestore.rules`
- `firestore.indexes.json`

## Firebase Project Set-up

If you have created a Firebase project, you should be able to access this
project using the [Firebase Console](https://console.firebase.google.com/).

We assume you're building a web app: let's create a new web project.

{% video src="/assets/images/docs/create-web-app-firebase.mp4" /%}

### Copying the Firebase configuration in the environment file

After the previous step, you should be seeing the Firebase configuration
object.

We need to add this to your `.env` file, which will automatically
populate the project configuration file `~/.configuration`.

```
FIREBASE_API_KEY=
FIREBASE_AUTH_DOMAIN=
FIREBASE_PROJECT_ID=
FIREBASE_STORAGE_BUCKET=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_APP_ID=
FIREBASE_MEASUREMENT_ID=
```

### Getting the service account

You can find the service account by clicking on `Settings->Service Accounts`.
You should copy the text highlighted in the video below:

{% video src="/assets/images/docs/find-service-account.mp4" /%}

Afterward, you need to populate the relative environment variable:

```
SERVICE_ACCOUNT_CLIENT_EMAIL=
```

You could safely add it using the file of your staging account. If it's
 your production environment, please add it using the Vercel console instead (or any other service you use to
 deploy the project).

### Generating the Private Key

The private key is fundamental to running your app with Admin permissions on the
server side. From the same page, click on "Generate Private key".

You should see the modal below:

{% img width="1368" height="1018" src="/assets/images/docs/generate-private-key.webp" alt="Generate Firebase Private Key" /%}

Once you open the generated JSON file, copy the `private_key` property and place it in your CI or your environment file as long as you do not commit the environment file.

The environment variable to populate is `SERVICE_ACCOUNT_PRIVATE_KEY`:

```
SERVICE_ACCOUNT_PRIVATE_KEY=
```

The key looks like this:

```
---
--BEGIN PRIVATE KEY---
--
***************************
---
--END PRIVATE KEY---
---
-
```

If you enter the wrong key, you will not be able to sign in server-side. In short, it won't work.

{% alert type="warning" %}
Do not commit the file if you add the private key to your local environment file
{% /alert %}

## Firestore

Now it's time to set up Firestore DB. From the main panel on the left-hand side of the Firebase Console, click on `Firestore Database`.

Once you click on the `Create Database` CTA, you will se a modal like in the
image below:

{% img width="1810" height="1132" src="/assets/images/docs/create-firestore-db.webp" alt="Create Firestore DB" /%}

Don't overthink what to choose: we use the default MakerKit Firestore
rules which use the production configuration by default.

### Location
The next step is to choose the Firestore Database's location.

Which option to choose here depends entirely on **where you think most of
your users come from**.

{% img width="1724" height="1034" src="/assets/images/docs/firestore-location.webp" alt="Choose Firestore Location" /%}

Be careful to the Firebase notice:

{% alert type="warning" %}
After you've set this location, you cannot change it later. Also, this location setting will be the location for your default Cloud Storage bucket.
{% /alert %}

If you're not sure, think it through. You do not need this step to start
working on your app with the Emulators, so feel free to come back to this
later when you have made a decision.

### Firestore Indexes

If you are setting up Makerkit for the first time, remember to set up an
index for querying an invitation without knowing its organization.

You can do so using the following link (replace
"\[\[YOUR_PROJECT_ID\]\]" with your real project ID):

https://console.firebase.google.com/u/0/project/\[\[YOUR_PROJECT_ID\]\]/firestore/indexes/single-field?create_exemption=ClBwcm9qZWN0cy9zdG9yeWpveS00NmVlMy9kYXRhYmFzZXMvKGRlZ
mF1bHQpL2NvbGxlY3Rpb25Hcm91cHMvaW52aXRlcy9maWVsZHMvY29kZRACGggKBGNvZGUQAQ

## Firestore and Storage Security Rules

Finally, you are required to publish the security rules to the production project. 

You can do so by logging in to the Firebase Console and pasting the rules from the `firestore.rules` and `storage.rules` files in their respective sections.

You don't need to do this right away, but you will need to do it before you deploy your application to production.

-----------------
FILE PATH: kits\remix-fire\data-fetching\custom-fetch.mdoc

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
FILE PATH: kits\remix-fire\data-fetching\fetching-data-firestore.mdoc

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
FILE PATH: kits\remix-fire\data-fetching\making-api-requests.mdoc

---
status: "published"

title: "API requests"
label: "API requests"
order: 2
description: "Learn how to make requests to your Remix API in Makerkit"
---


To make requests to the API functions in the `api` folder, we provide a
custom hook `useApiRequest`, a wrapper around `fetch`.

It has the following functionality:

1. Uses an internal `state` to keep track of the state of the request, like
`loading`, `error` and `success`. This helps you write fetch requests the
"hooks way"
2. Automatically adds a `Firebase AppCheck Token` to the request headers if you
enabled Firebase AppCheck
3. Automatically adds a `CSRF token` to the request headers

Similarly to making Firestore Requests, it's a convention to create a custom
hook for each request for readability/reusability reasons.

 ```tsx {% title="src/core/hooks/use-create-session.ts" %}
import useApiRequest from '~/core/hooks/use-api';

interface Body {
  idToken: string;
}

export function useCreateSession() {
  return useApiRequest<void, Body>('/api/session/sign-in');
}
```

The hook returns an array with:
1. the callback to make the request
2. the state object of the request

```tsx
const [createSession, createSessionState] = useCreateSession();
```

The state object internally uses `useApiRequest`, and has the following
interface:

```tsx
{
  success: boolean;
  loading: boolean;
  error: string;
  data: T | undefined;
}
```

When `success` is `true`, the property `data` is inferred with its correct type.

You can use this hook in your components:

```tsx
import { useCreateSession } from '~/core/hooks/use-create-session';

function Component() {
  const [createSession, createSessionState] = useCreateSession();

  return (
    <>
      { createSessionState.loading ? `Loading...` : null }
      { createSessionState.error ? `Error :(` : null }
      { createSessionState.success ? `Yay, success!` : null }

      <SignInForm onSignIn={(idToken) => createSession({ idToken })} />
    </>
  );
}
```

Similarly, you can also use it to fetch data:

```tsx
import { useCreateSession } from '~/core/hooks/use-create-session';

function Component() {
  const [fetchMembers, { loading, error, data }]
    = useFetchMembers();

  // fetch data when the component mounts
  useEffect(() => {
    fetchMembers();
  }, [fetchMembers]);

  if (loading) {
    return <div>Fetching members...</div>;
  }

  if (error) {
    return <div>{error}</div>;
  }

  return (
    <div>
      {data.map(member => <MemberItem member={member} />)}
    </div>
  );
}
```


-----------------
FILE PATH: kits\remix-fire\data-fetching\reading-data-storage.mdoc

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
FILE PATH: kits\remix-fire\data-fetching\uploading-data-to-storage.mdoc

---
status: "published"
title: "Uploading data to Storage | Remix.js Firebase SaaS Kit"
label: "Uploading data to Storage"
description: "Learn how to upload your data to Firebase Storage in your Remix application"
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

    return toast.promise(promise, {
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
FILE PATH: kits\remix-fire\data-fetching\writing-data-to-firestore.mdoc

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

      await toaster.promise(createTask(task), {
        success: `Task created!`,
        error: `Ops, error!`,
        loading: `Creating task...`,
      });

      await router.push(`/tasks`);
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
FILE PATH: kits\remix-fire\development\code-style.mdoc

---
status: "published"
title: "Adjust the codebase style according to your preferences | Remix Firebase Kit"
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
FILE PATH: kits\remix-fire\development\commands.mdoc

---
status: "published"
title: "All the commands to use for your Makerkit app | Remix Firebase"
description: "Use these commands to run the development server, build the application, and more in your Remix Firebase application"
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
FILE PATH: kits\remix-fire\development\cors.mdoc

---
status: "published"
title: "Enable CORS | Remix Firebase SaaS Kit"
label: Enable CORS
order: 9
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

export const action: ActionFunction = async ({request}) => {
  withCors(res);
  // your logic
}

export default apiHandler;
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

