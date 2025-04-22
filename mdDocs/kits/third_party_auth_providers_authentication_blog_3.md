-----------------
FILE PATH: kits\next-fire\authentication\third-party-auth-providers.mdoc

---
status: "published"
title: Set up Third party authentication providers with Next.js and Firebase
label: Third-Party Providers
order: 3
description: 'Learn how to setup Third-Paerty providers with Firebase Auth in
your MakerKit application'
---

By default, the kit uses Google as the third-party provider to allow users
to access the application.

With that said, you could decide to add other providers that Firebase provides or even custom ones.

## Customizing the Third Party providers

To add one or multiple oAuth providers, we will need to customize the global
configuration by adding the Firebase provider to the `oAuth` array.

For example, we could also add Facebook, Twitter, and GitHub:

 ```tsx {% title="configuration.ts" %}
import { FacebookAuthProvider, TwitterAuthProvider, GitHubAuthProvider } from
'firebase/auth';

{
  auth: {
    providers: {
      oAuth: [
        GoogleAuthProvider,
        FacebookAuthProvider,
        TwitterAuthProvider,
        GitHubAuthProvider
      ],
    }
  }
}
```

Additionally, we can add custom oAuth providers. First, we define them by
extending the `OAuthProvider` class:

```tsx
class MicrosoftAuthProvider extends OAuthProvider {
  constructor() {
    super('microsoft.com');
  }
}

class AppleAuthProvider extends OAuthProvider {
  constructor() {
    super('apple.com');
  }
}
```

And then, we simply add them to the `oAuth` array:

```tsx
oAuth: [
  GoogleAuthProvider,
  MicrosoftAuthProvider,
  AppleAuthProvider
],
```

{% alert type="warning" title="Remember to enable authentication methods" %}
<p>
    Remember that you will always need to enable the authentication methods
    you want to use from the Firebase Console once you deploy your application
    to production.
  </p>

  <p>
    Often, these need an additional configuration you need to
    enable on the provider's end.
  </p>
{% /alert %}

## Adding Additional Scopes

To add additional scopes to the oAuth provider, we can tweak the
`OAuthProviders.tsx` component and assign the scopes based on the instance type:

 ```tsx {% title="OAuthProviders.tsx" %}
import {
  GoogleAuthProvider,
  FacebookAuthProvider,
} from 'firebase/auth';

{OAUTH_PROVIDERS.map((OAuthProviderClass) => {
  const providerInstance = new OAuthProviderClass();
  const providerId = providerInstance.providerId;

  if (providerInstance instanceof GoogleAuthProvider) {
    providerInstance.addScope('scope');
  }

  if (providerInstance instanceof FacebookAuthProvider) {
    providerInstance.addScope('scope');
  }

   return (
     // render <AuthProviderButton />
   );
});
```

For example, if you are adding the scopes for Facebook Login, [you can check
out the Firebase documentation](https://firebase.google.com/docs/auth/web/facebook-login). Then, add them to the `addScope` method
as described above.


-----------------
FILE PATH: kits\next-fire\authentication.mdoc

---
status: "published"
title: Authentication in Next.js Firebase
label: Authentication
description: 'Authentication'
order: 4
collapsible: true
collapsed: true
---

Makerkit provides Server-Side Authentication with Firebase Authentication.

In this section, you will learn everything you need to know about the authentication flow of your SaaS.

- [Auth Overview](authentication/auth-overview): this is the overview of the authentication flow of your SaaS
- [Setting up Firebase Auth](authentication/setting-up-firebase-auth): this is the guide to set up Firebase Authentication in your project
- [Auth SSR](authentication/auth-ssr): this is the guide to set up SSR with Firebase Authentication
- [Auth Flow](authentication/auth-flow): this is the guide to set up the authentication flow of your SaaS
- [Auth Hooks](authentication/auth-hooks): this is the guide to use the authentication hooks in your project
- [Auth with Custom Providers](authentication/auth-third-party-auth-providers): this is the guide to set up custom authentication providers
- [Email Verification](authentication/email-verification): this is the guide to set up email verification
- [Email Link](authentication/email-link-authentication): this is the guide to set up email link authentication
- [Page Guards](authentication/page-guards): this is the guide to set up page guards
- [API Guards](authentication/api-guards): this is the guide to set up API guards
- [App Check](authentication/app-check): this is the guide to set up App Check
- [Troubleshooting](authentication/auth-troubleshooting): this is the guide to troubleshoot authentication issues

-----------------
FILE PATH: kits\next-fire\blog-and-docs\blog.mdoc

---
status: "published"
title: Add Blog posts to your Next.js application
label: Blog
order: 1
description: 'Learn how to add blog posts in
your MakerKit application'
---

We can add your website's blog posts within the `_posts` folder.

## Collections

Before writing a blog post, we have to define the post's collection. We use `collections` to organize our blog posts by their category.

For example, your blog could have collections such as changelog,
announcements, tutorials, etc.

MakerKit generates every collection with its page, listing all the articles
associated with it.

To change a collection page, please change the file `src/pages/blog/[collection].ts`.

### Adding collections

Let's see how we can define a `collection` in Typescript:

```tsx
interface Collection {
  name: string;

  // image
  logo?: string;
  emoji?: string;
}
````

As you can see, the only required property to create a collection is a name. You can also attach an image or a simple emoji for each collection.

A collection can be simply the following file:

```json
{
  "name":"Changelog"
}
```

## Writing a Blog Post
The interface of a blog post is the following:

```tsx
interface Post {
  title: string;
  excerpt: string;

  date: string;
  live: boolean;
  tags?: string[];

  coverImage?: string;

  ogImage?: {
    url: string;
  };

  author?: {
    name: string;
    picture: string;
    url: string;
  };

  canonical?: string;
}
```

These values can be defined in MDX files using the `frontmatter`, for example:

```yaml
---

title: An Awesome Post title
except: "Write here a short description for your blog post"
collection: changelog.json
date: '2022-01-05'
live: true
coverImage: '/assets/images/posts/announcement.webp'
tags:
  - changelog
  - coolstuff
---

```

#### Title
The property `title` is the title of the blog post.

NB: it does not correspond to the URL. The URL
segment, instead, is defined by the name of the file.

#### Excerpt
Write a short description for your blog post. This description will also
populate the relative meta tags.

#### Collection
The property `collection` is the path to the collection associated with the blog post.

NB: the `.json` extension needs to be part of the name.

#### Live
You can choose to publish or not a blog post by setting this property to `false`.

### Cover Image
The property `coverImage` is the path to the post's image.

### Tags
A list of tags you can attach to the post.

For each tag, MakerKit generates a page with every blog post associated, which you can update at `src/pages/blog/tags/[tag.tsx]`.


-----------------
FILE PATH: kits\next-fire\blog-and-docs\documentation.mdoc

---
status: "published"

title: Add a Documentation website to your Next.js application
label: Documentation
order: 2
description: 'Learn how to add documentation pages in your Next.js application'
---


We place the documentation pages within the `_docs` folder.

## Topics
The pages are defined hierarchically so that you can define your
documentation in the following way:

```
- _docs
  - [topic]
    - [topic.json]
    - [page].mdx
```

The documentation you're reading, for example, is defined as follows.

The MakerKit kit has a folder called `_docs`: in this folder, we have a list of
sub-folders for each topic we are describing.

Each folder has a metadata file named `meta.json`:

```json
{
  "title": "Blog and Docs",
  "position": 2,
  "description": "Learn how to configure and write your product's Blog and Documentation"
}
```

The `position` property defines the order of the topics. In
the case above, the topic `Blog and Docs` will be the third topic in the list.

## Pages

Within each topic, we define the collection pages defined as MDX files.
They share the same components as the blog posts' files.

A page is defined as follows:

```yaml
---

title: Blog
order: 1
---

```

The `position` property defines the order in which the page is listed within
the topic.


-----------------
FILE PATH: kits\next-fire\blog-and-docs\search.mdoc

---
status: "published"

title: Searching Posts and Documentation with Minisearch
label: Search Functionality
order: 3
description: 'Learn how Makerkit makes it easy to add a simple search engine
for your blog posts and documentation'
---

Makerkit's blog and documentation pages come with built-in search
functionality that does not reply on complex and expensive cloud services
such as Algolia and ElasticSearch.

Instead, we used [Minisearch](https://lucaong.github.io/minisearch/), a lightweight Javascript Search
Engine.

{% alert type="info" title="This feature has temporarily been deprecated" %}
This feature has temporarily been deprecated. Please do not use it.
{% /alert %}

### How does Minisearch work?

Instead of indexing our posts at runtime, we index all our
documentation and blog articles at build time, right after the Next.js
application has finished building.

Minisearch creates a simple index file in the `public` folder. The API loads
the index and responds to the user's query.

By default, we index two directories: `_docs` and `_posts`. Of course, it's
easy to extend this to other folders if you create more types of content.

Below you can find our indexer script that takes care of indexing
recursively our `.mdx` files in the folders provided:

 ```tsx {% title="search-indexer.ts" %}
const FIELDS = ['content', 'path', 'tag', 'title', 'collection', 'slug'];

export async function searchIndexer() {
  const miniSearch = new MiniSearch({
    fields: FIELDS,
    storeFields: FIELDS,
  });

  const engine = new SearchEngine(miniSearch);

  await engine.indexDirectory(`_posts`, (doc) => {
    return { ...doc, tag: `blog` };
  });

  await engine.indexDirectory(`_docs`, (doc) => {
    return { ...doc, tag: `docs` };
  });

  await engine.export();

  process.exit();
}
```

Assuming you have another folder named `_guides`, you could add:

 ```tsx {% title="search-indexer.ts" %}
await engine.indexDirectory(`guides`, (doc) => {
  return { ...doc, tag: `guide` };
});
```

{% video src="/assets/images/docs/search.mp4" /%}

### Does it scale?

It's important to keep the index not larger
than few MBs, otherwise it will exceed the maximum memory available to the
API function.

That could be more or less a few hundreds pieces of content (eg. articles),
so you may have some room before having to migrate.


-----------------
FILE PATH: kits\next-fire\blog-and-docs.mdoc

---
status: "published"
title: Add Blog posts and a Documentation website to your Next.js application
label: "Content"
order: 7
description: 'MakerKit allows you to easily add blog posts and a
documentation website for your Next.js application'
collapsible: true
collapsed: true
---

MakerKit has both Blogs and a Documentation site built in so that you can
start working on your content even before you launch your product.

We placed the sections together because, naturally, Blog and Docs share a good amount of code.

We can write the Blog posts and the documentation pages using [MDX
files](https://mdxjs.com/).

Thanks to MDX, we can augment the built-in components with richer styling and more complex components.


-----------------
FILE PATH: kits\next-fire\configuration\feature-flags.mdoc

---
status: "published"

title: Setting up Feature Flags with Next.js and Firebase
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

It's disabled by default, but you can enable it by setting the `enableAccountDeletion` flag to `true` or setting the environment variable `NEXT_PUBLIC_ENABLE_ACCOUNT_DELETION` to `true`.

```
NEXT_PUBLIC_ENABLE_ACCOUNT_DELETION=true
```

### Enable Organization Deletion

The organization deletion feature allows users to delete their organization.

It's disabled by default, but you can enable it by setting the `enableOrganizationDeletion` flag to `true` or setting the environment variable `NEXT_PUBLIC_ENABLE_ORGANIZATION_DELETION` to `true`.

```
NEXT_PUBLIC_ENABLE_ORGANIZATION_DELETION=true
```

-----------------
FILE PATH: kits\next-fire\configuration\firebase-functions.mdoc

---
status: "published"

title: Setting up Firebase Functions for your MakerKit application
label: Setting up Firebase Functions
order: 3
description: 'Learn how to set up Firebase Functions in your Makerkit SaaS project'
---


{% alert type="info" title="Setting Firebase Functions is optional" %}
Setting Firebase Functions is optional. You only need to set up Firebase Functions if you need certain features, such as Firestore Triggers, Storage Triggers, Authentication Blocking Functions, etc.
{% /alert %}

While you may want to use Next.js's own API functions for your application's API, there are many scenarios where you may be required to use Firebase Functions, for example:

1. You want to implement [Authentication Blocking Functions](/blog/tutorials/blocking-authentication-firebase-auth)
2. You need to use Firebase Storage Triggers
3. You need to use Firebase Firestore Triggers

Therefore, it's essential to know how to set up Firebase Functions in a Next.js environment, such as the Makerkit SaaS template.

## Initializing Firebase Functions

Assuming you have a working Makerkit project installed, let's type the following command in your terminal:

```
firebase init functions
```

When prompted about creating a new project, choose not to create a default project, as you may have already done it.

After completing the CLI prompts, you should have a few more files in your project:

```
✔  Wrote functions/package.json
✔  Wrote functions/tsconfig.json
✔  Wrote functions/src/index.ts
✔  Wrote functions/.gitignore
```

#### Adjust or delete the eslint configuration file

You may need to delete (or adjust) the `eslint` file generated by the CLI, as it may conflict with the one in the root of your project. We recommend using the one in the root of your project shipped with Makerkit.

If you leave it as is, you may see linting errors in your functions code, that can also prevent you from deploying your functions.

### Firebase Functions setup

I usually proceed to copy some of the scripts to the root `package.json`, such as the below:

```
"build:functions": "tsc -P functions/tsconfig.json",
"build:functions:watch": "tsc -P functions/tsconfig.json --watch",
"deploy:functions": "firebase deploy --only functions"
```

1. `build:functions` builds the functions bundle and then exits
2. `build:functions:watch` builds the functions in development mode, so it will re-bundle your code on changes
3. `deploy:functions` deploys your Functions to the Firebase cloud

### Writing your First Function

Let's write our first, very simple Firebase Function:

```tsx
import { https } from 'firebase-functions';

export const helloWorld = https.onRequest((req, res) => {
  res.send({ Hello: `World` })
});
```

### Running the Firebase Functions emulators

When using Makerkit, simply run the emulators:

```
npm run firebase:emulators:start
```

If working correctly you'll see the following output:

```
✔  functions: Loaded functions definitions from source: helloWorld.
```

🎉 And now you can finally use the full power of Firebase's Functions!

### Ignore the Firebase Functions folder when deploying to Vercel

Since Firebase Functions are to be deployed to Firebase, you can ignore the `functions` folder when deploying to Vercel.

To do so, create a file named `.vercelignore` in the root of your project and add the following line:

```
functions
```

-----------------
FILE PATH: kits\next-fire\configuration\setting-up-firebase.mdoc

---
status: "published"

title: Setting up a Firebase project for your MakerKit application
label: Setting up your Firebase Project
order: 1
description: 'Learn how to set up your Firebase project with a MakerKit application'
---

You should be all set if you have already installed NodeJS on your machine. If not, please install [NodeJS](https://nodejs.org) before continuing.

If any of the following commands do not work, please install the Firebase tools package globally:

```bash
npm i firebase-tools -g
```

Now, run this command to set up your Firebase project:

```bash
firebase init
```

Now, select all the products you would like to set up. If you're not sure yet, I recommend the following ones:
- Emulators
- Cloud Firestore
- Cloud Storage
- Remote Config

{% alert type="info" title="You do not need any changes to start developing your application" %}
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

We need to add this to your `.env.production` file, which will automatically
populate the project configuration file `~/.configuration`.

```bash
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=
```

{% alert type="warning" %}
Next.js bundles the environment variables prefixed with "NEXT_PUBLIC" with
your client-side code. It means that you should use these for non-secret values that need to be exposed.
{% /alert %}

### Auth Domain

To make sure that the authentication works correctly across the majority of browsers, please make sure the `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` is set to your custom domain.

For example, if your app is hosted at `https://app.makerkit.dev`, the `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` should be set to `app.makerkit.dev`.

**This is extremely important for the authentication to work correctly.**

### Getting the service account

You can find the service account by clicking on `Settings->Service Accounts`.
You should copy the text highlighted in the video below:

{% video src="/assets/images/docs/find-service-account.mp4" /%}

Afterward, you need to populate the relative environment variable:

```bash
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

```bash
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
FILE PATH: kits\next-fire\configuration.mdoc

---
status: "published"
title: Define the global configuration of your MakerKit SaaS application with Next.js and Firebase
label: 'Global Configuration'
order: 2
description: 'Learn how to define the global configuration of a MakerKit
application with Next.js and Firebase'
collapsible: true
collapsed: true
---

We store the global configuration of a MakerKit application in `/configuration.ts`.

Within any application file, we can use the path `~/configuration` to
import it from any other file.

{% alert type="info" title="You do not need any changes to start developing your application" %}
You do not need any changes to start developing your application. Feel free
to complete the configuration once you want to deploy the app for the first
time.
{% /alert %}

The configuration has the following structure:

 ```tsx {% title="configuration.ts" %}
export default {
  site: {
    title: '',
    description: '',
    themeColor: '',
    siteUrl: '',
    siteName: '',
    twitterHandle: '',
    language: 'en',
  },
  paths: {
    signIn: '/auth/sign-in',
    signUp: '/auth/sign-up',
    emailLinkSignIn: '/auth/link',
    onboarding: `/onboarding`,
    appHome: '/tasks',
    settings: {
      profile: '/settings/profile',
      authentication: '/settings/profile/authentication',
      email: '/settings/profile/email',
      password: '/settings/profile/password',
    },
    api: {
      checkout: `/api/stripe/checkout`,
      billingPortal: `/api/stripe/portal`,
    },
    searchIndex: `/public/search-index`,
  },
  firebase: {
    apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
    authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
    storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
    appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
    measurementId: process.env.NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID,
  },
  auth: {
    // Enable MFA. You must upgrade to GCP Identity Platform to use it.
    // see: https://cloud.google.com/identity-platform/docs/product-comparison
    enableMultiFactorAuth: true,
    // NB: Enable the providers below in the Firebase Console
    // in your production project
    providers: {
      emailPassword: true,
      phoneNumber: false,
      emailLink: false,
      oAuth: [GoogleAuthProvider],
    },
  },
  appCheckSiteKey: process.env.NEXT_PUBLIC_APPCHECK_KEY,
  theme: Themes.Light,
  features: {
    enableThemeSwitch: true,
    enableAccountDeletion: true,
    enableOrganizationDeletion: true,
  },
  navigation: {
    style: NavigationStyle.TopHeader,
  },
  environment: process.env.NEXT_PUBLIC_VERCEL_ENV ?? 'development',
  emulatorHost: process.env.NEXT_PUBLIC_EMULATOR_HOST,
  emulator: process.env.NEXT_PUBLIC_EMULATOR === 'true',
  production: process.env.NEXT_PUBLIC_NODE_ENV === 'production',
  stripe: {
    products: [
      {
        name: 'Basic',
        description: 'Describe your basic plan',
        plans: [
          {
            price: '$249/year',
            stripePriceId: '<STRIPE_PRICE_ID>',
          }
        ],
      },
      {
        name: 'Pro',
        description: 'Describe your pro plan',
        plans: [
          {
            price: '$249/year',
            stripePriceId: '<STRIPE_PRICE_ID>',
          }
        ],
      },
    ],
  },
};
```

These values are used throughout the application instead of being hardcoded into the codebase.

As you may have noticed, some of these values are taken from Node's
environment. We will use an additional file name `.env`, which is used by
`Next` to read the configuration.

This file will not be committed (at least not
the "secret" variable). So instead, you should define those variables within your
favorite CI/CD.

## Environment Variables

Makerkit provides templates for configuring your environment variables correctly and the **minimum** environment variables to run your local environment.

To push your project to production, **you must fill these variables by creating your Firebase project** and adding the required values.

{% alert type="warning" %}
When you first run your project build, ensure to fill the required environment variables using the ".env.production" environment file.
{% /alert %}

### Development Environment Variables

The development environment variables set your application up for the Firebase Emulators:

```bash
NEXT_PUBLIC_EMULATOR=true

NEXT_PUBLIC_FIREBASE_PROJECT_ID=demo-makerkit
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=demo-makerkit.appspot.com
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=localhost
NEXT_PUBLIC_FIREBASE_EMULATOR_HOST=localhost
NEXT_PUBLIC_FIRESTORE_EMULATOR_PORT=8080
NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_PORT=9099
NEXT_PUBLIC_FIREBASE_STORAGE_EMULATOR_PORT=9199

# Change this with your project's APP ID
NEXT_PUBLIC_FIREBASE_APP_ID=<MAKERKIT_DEV_APP_ID>
# Change this with your project's API KEY
NEXT_PUBLIC_FIREBASE_API_KEY=<MAKERKIT_DEV_API_KEY>

FIRESTORE_EMULATOR_HOST=localhost:8080
FIREBASE_AUTH_EMULATOR_HOST=localhost:9099
FIREBASE_STORAGE_EMULATOR_HOST=localhost:9199
FIREBASE_PUBSUB_EMULATOR_HOST=localhost:8085

SERVICE_ACCOUNT_CLIENT_EMAIL=
SERVICE_ACCOUNT_PRIVATE_KEY=
```

Please remember to update the `NEXT_PUBLIC_FIREBASE_APP_ID` and `NEXT_PUBLIC_FIREBASE_API_KEY` with the ones from your Firebase project ID. The only reason they're predefined is to allow you to quickly start the application.

### Production Environment Variables

When you go to production, create the `.env.production` by copying the content of `.env.production.template`:

```bash
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=
NEXT_PUBLIC_SITE_URL=

SERVICE_ACCOUNT_CLIENT_EMAIL=

## SECRET KEYS ARE BEST ADDED TO YOUR CI
SERVICE_ACCOUNT_PRIVATE_KEY=
SECRET_KEY=
SECRET_APPCHECK_KEY=

NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# Add these in Vercel or .env.local
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# EMAIL
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_SENDER='MakerKit Team <info@makerkit.dev>'
```

All the secret variables should be added from your Vercel or CI console; if you need them locally, you should add them to your `.env.local` file, an environment file that is ignored by Git, and, therefore, suitable for storing sensitive data that is local and isolated to your machine.

When pushing your project to Vercel, the build will pick up the values added to `.env.production`.

### Configuring the Firebase Secret Key environment variable

If you are adding the Firebase secret key as an environment variable (as you
should) from your Vercel console (or other providers), you should make sure to
 enter the key in a valid format. But, again, this can differ between providers.

If using Vercel, add the line breaks to the pasted code.
 To do so, you can use the following regexp to replace `\\n` with `\n`. You
 should be seeing the private key with the correct line breaks (instead of
 `\n`).

Next, you can paste the secret key in the correct format into your Vercel
console.

### MakerKit configuration details

Let's see the configuration in detail:

## Site

In this section, we define some overall details about the website. We use most of these configuration properties in public pages, blog posts, and documentation.

#### Title

This property is the default title of the website. We use it in conjunction with the page's name. For example, the blog's title will be `Blog - { site.name }`.

#### Description

This title is the default description of the website. For blog posts, we override it with each entry's excerpt.

#### Theme Color

We use the property `themeColor` to define the primary color browsers display. For example, in a PWA, it is the background color of the window's header.

#### Site URL

This `siteUrl` property should be your website's URL, including the protocol, such as https://yourwonderfulwebsite.com

#### Twitter Handle

You can use this or your company's Twitter handle to link your blog posts.

## Paths

Some of the paths are repeated many times throughout the codebase. For example: where should the user be redirected to after login?

These paths help you set up the default behavior when the user signs in, out, etc.

Of course, while we could store these in the codebase, it's the quickest way to get started with the boilerplate without changing any line of code.

## Firebase

The `firebase` object is Firebase's configuration which you can retrieve from the Firebase console. You could copy and paste it in, as it's all required.

If you use multiple environments (for example, `staging` and `production`), I recommend creating two `.env` files and starting your application using the correct one.

We will show later how to do set-up multiple environments effectively.

## Plans

Here you can store the plans that your application offers. They should match
the plans that you are going to create in the Stripe Dashboard.


