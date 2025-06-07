-----------------
FILE PATH: kits\remix-supabase\authentication.mdoc

---
status: "published"
title: "Authentication in Remix Supabase (legacy)"
label: "Authentication"
order: 4
description: "Learn how to set up authentication in your Remix Supabase application"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\remix-supabase\blog-and-docs\blog.mdoc

---
status: "published"

title: Add Blog posts to your Remix Supabase application
label: Blog
order: 1
description: 'Learn how to add blog posts in your MakerKit Remix Supabase
application'
---


We can add your website's blog posts within the `src/content/posts` folder.

## Add a new blog post

To add a new blog post, create a new markdown file within the `src/content/posts` folder. The file name should be the slug of the blog post. For example, if you want to create a blog post with the slug `hello-world`, create a file called `hello-world.mdx`.

For example, the following file is a blog post with the slug `lorem-ipsum`:

```mdx
---

title: Lorem Ipsum
date: 2023-09-23
live: true
description: "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."
image: "/assets/images/posts/lorem-ipsum.webp"
---

```

## Blog post metadata

Each blog post should have the following metadata:

- `title`: The title of the blog post
- `date`: The date of the blog post
- `live`: Whether the blog post should be visible on the website
- `description`: The description of the blog post
- `image`: The image of the blog post

## Blog post content

The content of the blog post should be written in markdown. You can use the [MDX](https://mdxjs.com/) syntax to add React components within the markdown content.

Additionally, we included multiple components that you can use to add content to your blog post:

- `Image`: Add an image to your blog post (a wrapper around the `next/image` component)
- `Video`: Add a video to your blog post. This is lazy-loaded and includes a loading spinner while the video is loading
- `Alert`: Add an alert to your blog post
- `LazyRender`: Lazy-load a React component within your blog post
- `TweetEmbed`: Embed a tweet within your blog post

-----------------
FILE PATH: kits\remix-supabase\blog-and-docs\documentation.mdoc

---
status: "published"

title: Add a Documentation website to your Remix Supabase application
label: Documentation
order: 2
description: 'Learn how to add documentation pages in your Remix Supabase application'
---


The Remix Supabase Starter Kit comes with a documentation generator that you can use to create a documentation website for your SaaS application.

{% img src="/assets/images/posts/docs-index.webp" alt="Documentation Index Page" width="2796" height="1668" /%}

The documentation (like the blog) is built using the amazing [Contentlayer](https://contentlayer.dev). Take a look at the documentation if you're planning on extending the functionality of the documentation.

### Content Folder

Every kit now has a `content` folder, which contains the Markdown files for the documentation. The `content` folder is organized into sections, which are then rendered as pages on the documentation site.

Sections can be nested, so you can have a section with sub-sections. The sub-sections are then rendered as sub-pages.

### Initial Section page

Every section page can have an `index.mdx` file, which is the initial page for the section. This is useful if you want to have a landing page for the section, which then links to the sub-sections.

{% img src="/assets/images/posts/docs-section-page.webp" alt="Documentation Section Page" width="2818" height="1800" /%}

### Ordering

To organize the order of the sections, you shall prefix the folder name with a number, such as `01-getting-started`. The sections are then sorted alphabetically, so you can use numbers to control the order.

{% img src="/assets/images/posts/docs-ordering.webp" alt="Documentation Ordering" width="754" height="674" /%}

For example, the `content` folder for the Remix Supabase kit looks like this:

```bash
- content
  - docs
    - 01-getting-started
      - index.mdx
      - 01-installation.mdx
      - 02-configuration.mdx
        - 01-supabase.mdx
        - 02-firebase.mdx
      - 03-authentication.mdx
      - 04-database.mdx
      - 05-storage.mdx
      - 06-usage.mdx
      - 07-deployment.mdx
        - 01-vercel.mdx
        - 02-netlify.mdx
      - 08-questions.mdx
    - 02-faq
      - index.mdx
      - 01-what-is-supabase.mdx
```

As you can see, we can use nested folders to organize the content. The `01-getting-started` folder is the first section, and it has an `index.mdx` file, which is the initial page for the section. The `02-configuration` folder is the second section, and it has a sub-section `01-supabase` and `02-firebase`.

### Frontmatter

Every Markdown file can have a frontmatter, which is a YAML block at the top of the file. The frontmatter is used to define the title of the page, and the description of the page.

For example, the `index.mdx` file for the `01-getting-started` section looks like this:

```mdx
---

title: "Getting Started with the Makerkit SaaS Starter Kits"
label: "Getting Started"
description: "Learn how to get started with the Makerkit SaaS Starter Kits"
---

```

The `title` is the title of the page, which is used for the `title` tag in the HTML code. The `label` is the label of the page, which is used in the sidebar navigation (where it's likely you will want a less SEO-focused label). The `description` is the description of the page, which is used in the meta tags.

### Content

The content of the Markdown file is written in Markdown, and is rendered as HTML. You can use Markdown to write the content of the page.

Just like the blog post, you can use the following components to render the content:

- `Image`: Add an image to your blog post (a wrapper around the `next/image` component)
- `Video`: Add a video to your blog post. This is lazy-loaded and includes a loading spinner while the video is loading
- `Alert`: Add an alert to your blog post
- `LazyRender`: Lazy-load a React component within your blog post
- `TweetEmbed`: Embed a tweet within your blog post

### Navigation

The navigation for the documentation is automatically generated based on the folder structure of the `content` folder. Furthermore, page links appear at the bottom of the page, so you can easily navigate to the previous or

-----------------
FILE PATH: kits\remix-supabase\blog-and-docs.mdoc

---
status: "published"
title: "Blog and Docs in Remix Supabase"
label: "Blog and Docs"
order: 8
description: "Blog and Docs in Remix Supabase"
collapsible: true
collapsed: true
---

-----------------
FILE PATH: kits\remix-supabase\configuration\configuration.mdoc

---
status: "published"
title: "Define the global configuration of your MakerKit SaaS application
with Remix and Supabase"
label: "Global Configuration"
order: 2
description: "Learn how to define the global configuration of a MakerKit
application with Remix and Supabase"
---

We store the global configuration of a MakerKit application in `/configuration.ts`.

Within any application file, we can use the path `~/configuration` to
import it from any other file.

{% alert type="info" %}
You do not need any changes to start developing your application. Feel free
to complete the configuration once you want to deploy the app for the first
time.
{% /alert %}

The configuration has the following structure:

 ```ts {% title="configuration.ts" %}
import getEnv from '~/core/get-env';
import type { Provider } from '@supabase/gotrue-js/src/lib/types';

const env = getEnv() ?? {};
const production = env.NODE_ENV === 'production';

const configuration = {
  site: {
    name: 'Awesomely - Your SaaS Title',
    description: 'Your SaaS Description',
    themeColor: '#ffffff',
    themeColorDark: '#0a0a0a',
    siteUrl: env.SITE_URL,
    siteName: 'Awesomely',
    twitterHandle: '',
    githubHandle: '',
    language: 'en',
    convertKitFormId: '',
    locale: env.DEFAULT_LOCALE,
  },
  auth: {
    // ensure this is the same as your Supabase project
    // by default - it's true in production and false in development
    requireEmailConfirmation: production,
    // NB: Enable the providers below in the Supabase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: ['google'] as Provider[],
    },
  },
  production,
  environment: env.ENVIRONMENT,
  features: {
    enableThemeSwitcher: true,
  },
  theme: Themes.Light,
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    signInMfa: '/auth/verify',
    signInFromLink: '/auth/link',
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
      organizations: {
        create: `/resources/organizations/create`,
        transferOwnership: `/resources/organizations/transfer-ownership`,
        members: `/resources/organizations/members`,
      },
    },
  },
  sentry: {
    dsn: env.SENTRY_DSN,
  },
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
            stripePriceId: 'basic-plan-mth',
            trialPeriodDays: 7,
          },
          {
            name: 'Yearly',
            price: '$90',
            stripePriceId: 'basic-plan-yr',
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
          },
          {
            name: 'Yearly',
            price: '$200',
            stripePriceId: 'pro-plan-yr',
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
  },
};

export default configuration;
```

These values are used throughout the application instead of being hardcoded into the codebase.

This file will not be committed (at least not the "secret" variable). So instead, you should define those variables within your favorite CI/CD.


-----------------
FILE PATH: kits\remix-supabase\configuration\environment-variables.mdoc

---
status: "published"

title: "Environment Variables"
label: Environment Variables
order: 2
description: "Learn how to use environment variables in your Next.js Supabase project."
---


## Environment Variables

Makerkit provides templates for configuring your environment variables correctly and the **minimum** environment variables to run your local environment.

To push your project to production, **you must fill these variables by creating your Supabase project** and adding the required values.

Below are the required environment variables for your project to run locally and in production.

```
DEFAULT_LOCALE=en
SITE_URL=http://localhost:3000

# set the below to "production" in your production environment
ENVIRONMENT=development

# SUPABASE
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# STRIPE
STRIPE_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=
STRIPE_PUBLISHABLE_KEY=

# EMAIL
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

### Production Environment Variables

{% alert type="warning" %}
When you go to production, ensure you add the required environment variables using your CI or service provider.
{% /alert %}

Remix **does not bundle your ".env" variables when building your application in production mode**, so you need to ensure you are providing these variables using your CU or service provider.

## Website Configuration

The following variables are used to configure your website:

### DEFAULT_LOCALE

The default locale for your application. This is used by the `i18next` library to set the default locale for your application.

### SITE_URL

The URL of your application. This is used across the application to generate links to your application.

## Supabase Configuration

The following variables are used to configure your Supabase project:

### SUPABASE_URL

The URL of your Supabase project. This is used by the Supabase client to connect to your Supabase project. It is a public variable.

### SUPABASE_ANON_KEY

The anonymous key of your Supabase project. This is used by the Supabase client to connect to your Supabase project. It is a public variable.

### SUPABASE_SERVICE_ROLE_KEY

The service role key of your Supabase project. This is used by the Supabase client to connect to your Supabase project. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

## Stripe Configuration

The following variables are used to configure your Stripe project:

### STRIPE_WEBHOOK_SECRET

The webhook secret of your Stripe project (also referred to as Signing Secret within the Stripe dashboard). This is used by the Stripe client to connect to your Stripe project. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### STRIPE_SECRET_KEY

The secret key of your Stripe project. This is used by the Stripe client to connect to your Stripe project. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### STRIPE_PUBLISHABLE_KEY

The publishable key of your Stripe project. This is used by the Stripe client to connect to your Stripe project. It is a public variable and it's used to create the embedded checkout with the Stripe SDK.

## Email Configuration

The following variables are used to configure your email provider:

### EMAIL_HOST

The SMTP host of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_PORT

The SMTP port of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_USER

The SMTP user of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_PASSWORD

The SMTP password of your email provider. This is used by the email client to connect to your email provider. It is a secret variable - so please add it to your CI/CD environment and never add it to Git.

### EMAIL_SENDER

The sender of your emails. This is used by the email client to send emails to your users. It is a public variable. Use the format `Sender Name <email@app.com>`.

-----------------
FILE PATH: kits\remix-supabase\configuration\feature-flags.mdoc

---
status: "published"

title: Setting up Feature Flags with Remix.js and Supabase
label: 'Feature Flags'
order: 2
description: 'The Makerkit feature flagging system allows you to control the visibility of certain features in your application. This is useful for testing new features, or for rolling out features to a subset of your users.'
---


Makerkit uses feature flags to let you control the visibility of certain features in your application. We use a feature flag when we think a feature may not be 100% ready or when it's too opinionated and we want to give you the option to turn it off.

NB: In following releases, some flags may be removed and the feature will be enabled by default. Other flags may be removed and the feature will be removed as well.

## How it works

By default, Makerkit uses the following feature flags, defined in the `src/configurations.ts.ts` file:

```ts
{
  enableThemeSwitcher: true,
  enableAccountDeletion: false,
  enableOrganizationDeletion: false,
}
```

### Enable Theme Switcher

The theme switcher allows users to switch between light and dark mode. It's enabled by default, but you can disable it by setting the `enableThemeSwitcher` flag to `false`.

### Enable Account Deletion

The account deletion feature allows users to delete their account.

It's disabled by default, but you can enable it by setting the `enableAccountDeletion` flag to `true` or setting the environment variable `ENABLE_ACCOUNT_DELETION` to `true`.

```
ENABLE_ACCOUNT_DELETION=true
```

### Enable Organization Deletion

The organization deletion feature allows users to delete their organization.

It's disabled by default, but you can enable it by setting the `enableOrganizationDeletion` flag to `true` or setting the environment variable `ENABLE_ORGANIZATION_DELETION` to `true`.

```
ENABLE_ORGANIZATION_DELETION=true
```

-----------------
FILE PATH: kits\remix-supabase\configuration.mdoc

---
status: "published"
title: "Configuration in Remix Supabase"
label: "Configuration"
order: 2
description: "Configuration in Remix Supabase"
collapsible: true
collapsed: true
---


-----------------
FILE PATH: kits\remix-supabase\data-fetching\fetching-data-supabase.mdoc

---
status: "published"

title: "Fetching data from Supabase"
label: "Fetching data from Supabase"
order: 1
description: "Learn how fetch data from your Supabase database in Makerkit"
---


When fetching data from the Supabase Postgres Database, we use the Supabase
JS client.

To make data-fetching from Supabase more reusable, our convention is to
create a custom React hook for each query we make.

For example, let's take a look at the hook to fetch an organization from
Supabase.

First, we want to write a query that receives two parameters:
1. a Supabase client interface
2. an organization ID

 ```tsx {% title="organizations/queries.ts" %}
export function getOrganizationById(
  client: Client,
  organizationId: number
) {
  return client
    .from('organizations')
    .select(`
      id,
      name,
      logoURL: logo_url
    `)
    .eq('id', organizationId)
    .throwOnError()
    .single();
}
```

It's important that we pass the client as a parameter so that we can use the
function in both the browser and the server.

Now, if we want to use a React Hook to fetch the organization from one of
our components, we can use the `useQuery` hook from `react-query`:

```tsx
import useSupabase from '~/core/hooks/use-supabase';
import { getOrganizationById } from '~/lib/organizations/database/queries';
import { useQuery } from '@tanstack/react-query';

function useOrganizationQuery(
  organizationId: number
) {
  const client = useSupabase();
  const key = ['organization', organizationId];

  return useQuery(key, async () => {
    return getOrganizationById(client, organizationId).then(
      (result) => result.data
    );
  });
}

export default useOrganizationQuery;
```

### Retrieving data using a React hook

Now that we have a hook to fetch an organization, we can use it in our
components to retrieve the data.

```tsx
import { useOrganizationQuery } from './use-organization-query';

function OrganizationCard({ organizationId }) {
  const {
    data: organization,
    isLoading,
    error
  } = useOrganizationQuery(organizationId);

  /* data is loading */
  if (isLoading) {
    return <div>Loading...</div>
  }

  /* request errored */
  if (error) {
    return <div>Error!</div>
  }

  /* request successful, we can access "organization" */
  return <div>{organization.name}</div>;
}
```

### Retrieving data from the server

When we want to fetch data from the server, we can import a server Supabase
client and use it to fetch data.

```tsx
import {
  throwInternalServerErrorException,
} from '~/core/http-exceptions';

import { json } from '@remix-run/node';
import getSupabaseServerClient from '~/core/supabase/server-client';
import { parseOrganizationIdCookie } from '~/lib/server/cookies/organization.cookie';
import { getOrganizationById } from "~/lib/tasks/queries";

export async function loader({ request }: LoaderArgs) {
  const client = getSupabaseServerClient(request);
  const organizationId = Number(await parseOrganizationIdCookie(request));

  try {
    const { data } = await getOrganizationById(client, organizationId);

    return json(data);
  } catch (error) {
    return throwInternalServerErrorException();
  }
}
```

When you want to ensure your data is rendered on the server, you should ensure you're loading the data using the `loader` function. Then, you can co-ordinate data re-fetching on the client when the data changes using `useSubmit` or `useFetcher`.

When the data is only fetched on the client, you can use `useQuery` to fetch the data and re-fetch the stale queries.

-----------------
FILE PATH: kits\remix-supabase\data-fetching\making-api-requests.mdoc

---
status: "published"

title: "API requests"
label: "API requests"
order: 2
description: "Learn how to make requests to your Remix API in Makerkit"
---


Normally, Remix encourages you to colocate actions/loaders within the page where they are used. 

However, sometimes you may want to make a request to your API from a different page, or from a component that is not colocated with the API. This can be the case when the same action is used in multiple places, or when you want to make a reusable component that can be used in multiple places. 

In these cases, the recommended approach is to add your API routes under `routes/resources`. For example, the Stripe webhooks are added under `routes/resources/stripe`.

## Executing API requests with Remix

To make a request to your API, we recommend using one of the Remix data-fetching utilities, such as `Form`, `useFetcher`, or `useSubmit`. 

These are meant to work seamlessly with Remix's data-fetching utilities and will perform some optimizations like automatic cancellation of requests when the component unmounts, and automatic data re-fetching for the layouts that the page is using.

## Data Fetching

Let's see a simple example of using `useFetcher` to make a request to our API. First, we define a loader function that fetches some data and returns it:

 ```tsx {% title="app/resources/users.ts" %}
export async function loader() {
  const users = await loadUser();

  return json({ users });
}
```

At this point, you have various options for how to use this loader function. 

You can use a `fetch` call to make the request, or you can use one of the Remix data-fetching utilities, such as `useFetcher`.

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


