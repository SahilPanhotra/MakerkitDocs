-----------------
FILE PATH: kits\remix-fire\authentication\app-check.mdoc

---
status: "published"

title: Protect your Remix application from abuse using Firebase AppCheck
label: Prevent abuse with AppCheck
order: 9
description: 'Learn how to set up Firebase AppCheck in your MakerKit application to protect your application from abuse'
---


App Check is a Firebase service that helps you to protect your web app from
bots, spammy users, and general abuse detected by Google's Recaptcha.

The Makerkit SaaS starter is already configured to secure your application with Firebase AppCheck, but you will need to set it up in the Firebase Console first.

### Registering your website for Recaptcha v3

[To register your website for Recaptcha v3, you need to complete the linked form](https://www.google.com/recaptcha/admin/create). After entering the details, you'll be provided with a `Site Key`, which we use to initialize the Firebase AppCheck service.

{% img src="/assets/images/posts/create-recaptchha-key.webp" width="1494" height="878" /%}

NB: when registering the website, create a **Recaptcha v3** key, not v2.

After submitting the form, you will receive two keys:

1. a public key (on the client side)
2. a private key (to be used on the server side)

Save the public key and add it to your Remix `.env` file:

```
APPCHECK_KEY=<YOUR_KEY>
```

### Enabling Firebase App Check from the console

First, we need to enable Firebase App Check from the Firebase Console to register our application using the keys we have generated in the previous step.

{% img src="/assets/images/posts/register-app-check.webp" width="2850" height="902" /%}

Above, you should have added the secret key that we have generated. If it all went well, you have successfully enabled app-check for your application 🎉!

#### Enforcing Firestore to use App Check

To enforce Firestore to always validate requests using AppCheck, you must enable it from the Console. From where you are, click on the `API` tab, then click on `Firebase Firestore`. You will then be presented with the popup as in the image below:

{% img src="/assets/images/posts/enforce-appcheck-firestore.webp" width="1752" height="1340" /%}

Continue and enforce Firestore to use AppCheck.

### Testing AppCheck

AppCheck will not work in emulator mode, so you have two options to ensure it's working as intended:

- run a production build locally
- or simply deploy your applications


-----------------
FILE PATH: kits\remix-fire\authentication\auth-flow.mdoc

---
status: "published"

title: Firebase and Remix authentication flow
label: Auth Flow
order: 2
description: 'Learn how MakerKit handles the authentication flow with
Firebase Auth and Remix'
---


If you use SSR, the flow is slightly different from what you may be
used.

Typically, Firebase allows signing users in on the client-side. If you use SSR,
we need a way to authenticate the user on the server-side.

To do so, we use Firebase Auth's session cookies: we create the cookie when
the user signs in and then destroy it when the user signs out.

### Step 1: User signs in using the client SDK

The user can sign in (or up) using the Firebase Auth client SDK from the client side.

### Step 2: Creating a Session Cookie using an API endpoint

After being authenticated (signing in or up), the application requests to create and store a session cookie using an API endpoint.

The cookie is stored as an HTTP-only cookie and will be sent to the server,
so we can authenticate users right when we render the page.

When needed, we can perform the necessary redirects and security checks server-side.

### Step 3: User gets redirected to Application Home Page

The application renders a session-aware page if you have opted to use SSR on every page. Otherwise, the Firebase SDK should reflect the current session once the page has been rendered.

When the user is redirected to a protected page, the user's session authentication state is checked in the `loader` function of the `__app` layout.

### Step 4: User signs out

The client SDK starts a sign-out request when the user decides to sign out.

When Firestore Auth destroys the current user session, the event is intercepted by a listener and then calls the API endpoint to destroy the session cookie.

Here is an illustration of the flows:

{% img src="/assets/images/docs/auth-flow.png" width="2344" height="1166" /%}

## Keeping the client and server sessions in sync

Because the Firebase Session cookies expire after 14 days, we need to keep the client SDK session in sync when the session cookie expires.

To do so, when the user signs in, we also store a cookie with the expiration date. When the client SDK detects that the server-side session cookie expired, the user gets automatically signed out.


-----------------
FILE PATH: kits\remix-fire\authentication\auth-hooks.mdoc

---
status: "published"

title: Authentication React Hooks
label: Custom React Hooks
order: 10
description: 'Reference for the authentication React hooks'
---


Makerkit provides a set of React hooks to help you manage authentication in your React application. These hooks are an addition to the ones provided by `reactfire`.

## useUserSession

This hook returns the current user session. It contains the user's auth data from Firebase, and its data record from Firestore. Basically, it wraps the `UserSessionContext` value.

```tsx
import { useUserSession } from "~/core/hooks/use-user-session";

const userSession = useUserSession();
```

## useUserId

This hook returns the current user's ID.

```tsx
import { useUserId } from "~/core/hooks/use-user-id";

const userId = useUserId();
```

## useCreateServerSideSession

This hook returns a function that can be used to create a server-side session. It is useful for creating a session when a user logs in, or when a user is created.

You shouldn't be needing this unless you're customizing the authentication flow.

```tsx
import { useCreateServerSideSession } from "~/core/hooks/use-create-server-side-session";

const [sessionRequest, { loading, error }] = useCreateServerSideSession();
```

-----------------
FILE PATH: kits\remix-fire\authentication\auth-overview.mdoc

---
status: "published"

title: Setting up Firebase and Remix authentication with MakerKit
label: Auth Overview
order: 0
description: 'MakerKit uses Firebase Authentication to allow access to your Remix application using oAuth providers and email/password.'
---


MakerKit uses Firebase to manage authentication within your application.

By default, every kit comes with the following built-in authentication methods:
- **Email/Password** - we added, by default, the traditional way of signing in
- **Third Party Providers** - we also added by default Google Auth sign-in
- **Email Links**
- **Phone Number**

You're free to add (or remove) any of the methods supported by Firebase's
Authentication: we will see how.

This documentation will help you with the following:
 - **Setup** - setting up your Firebase project
 - **SSR** - use SSR to persist your users' authentication, adding new
providers
 - **Customization** - an overview of how MakerKit works so that you can adapt
it to your own application's needs

## Configuration

The way you want your users to authenticate can be driven via configuration.

If you open the global configuration at `src/configuration.ts`, you'll find
the `auth` object:

 ```tsx {% title="configuration.ts" %}
auth: {
  enableMultiFactorAuth: true,
  providers: {
    emailPassword: true,
    phoneNumber: false,
    emailLink: false,
    oAuth: [GoogleAuthProvider],
  },
},
```

As you can see, the `providers` object can be configured to only display the
auth methods we want to use.

1. For example, by setting both `phoneNumber` and `emailLink` to `true`, the
authentication pages will display the `Email Link` authentication
 and the `Phone Number` authentication forms.
2. Instead, by setting `emailPassword` to `false`, we will remove the
`email/password` form from the authentication and user profile pages.

Additionally, we can choose to add one or multiple oAuth providers by adding
the Firebase provider to the `oAuth` array. For example, we could also add
Facebook, Twitter, and GitHub:

```tsx
import { FacebookAuthProvider, TwitterAuthProvider, GitHubAuthProvider } from
'firebase/auth';

oAuth: [
  GoogleAuthProvider,
  FacebookAuthProvider,
  TwitterAuthProvider,
  GitHubAuthProvider
],
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

{% alert type="warning" %}
Remember that you will always need to enable the authentication methods
  you want to use from the Firebase Console once you deploy your application
  to production
{% /alert %}

## SSR

Using SSR, we set up MakerKit to persist the user's session on all the website's pages.

SSR allows for seamless integration between the pages of your website. For example, your pricing page could prompt users to upgrade to the new plan rather than what it shows to non-subscribers of your service.

Many websites use persistent sessions in different ways:

- personalized content
- pre-filled forms
- and a lot more

-----------------
FILE PATH: kits\remix-fire\authentication\authentication-troubleshooting.mdoc

---
status: "published"

title: Authentication Troubleshooting
label: Troubleshooting
order: 11
description: 'Common issues with authentication'
---


Here are some common issues with authentication and how to fix them.

## I keep getting signed out after signing in

**Possible cause**: You are using the wrong Firebase private key.

This issue is usually caused by a misconfigured Firebase private key. Make sure you are using the correct private key and that it is not expired.

The Firebase private key is downloaded from the JSON Service Account file after you create a new service account in the Firebase console

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

It's much longer than the above, but you get the idea.

The key needs to be added to your environment variables settings. Please use your CI to add this key since it's a secret key:

```
SERVICE_ACCOUNT_PRIVATE_KEY=
```

-----------------
FILE PATH: kits\remix-fire\authentication\email-link-authentication.mdoc

---
status: "published"

title: Set up Email Link Authentication with Remix and Firebase
label: Email Link Authentication
order: 4
description: 'Learn how to setup Email Link Authentication with Firebase Auth in
your MakerKit application'
---


Makerkit provides a component and a route to sign your users in using an Email Link.

Thanks to Email Links Authentication, users can enter their email and then sign in by clicking on the link they received to their email.

{% alert type="warning" %}
Remember, when using this authentication method in production, ensure you manually activate it from the Firebase Console first.
{% /alert %}

Navigate to the `configuration.ts` file, and let's switch the
`email/password` method with `email link`:

 ```tsx {% title="configuration.ts" %}
{
  auth: {
    providers: {
      emailPassword: false, // we switch this to false
      phoneNumber: false,
      emailLink: true, // we switch this to true
      oAuth: [GoogleAuthProvider, FacebookAuthProvider],
    },
  }
}
```

That's it! The result should be similar to the image below:

{% img src="/assets/images/posts/email-link-authentication.webp" width="1128" height="1206" /%}

Ok, cool, **but can you use both?**

Yes, of course! With that said, you may need to tweak the
UI so that the result will look nice.


-----------------
FILE PATH: kits\remix-fire\authentication\email-verification.mdoc

---
status: "published"

title: Requiring Email verification
label: Requiring Email verification
order: 6
description: 'Learn how to enforce email verification for your users with Firebase.'
---


By default, Firebase does not enforce email verification. This means that users can sign up with an email address and password, and then immediately start using your app. However, you may want to require that users verify their email address before they can use your app.

To do so, Makerkit can check that the user has verified their email address before allowing them to use your app from the server side. How? When loading the server side session, we check that the user's property `emailVerified` is `true`. If not, the user is redirected to the sign in page, and signed out.

By default, **this feature is disabled**. To enable it, you need to set the environment variable `REQUIRE_EMAIL_VERIFICATION` to `true`.

## Enabling requiring email verification

Simply set the following environment variable to `true`:

```bash
REQUIRE_EMAIL_VERIFICATION=true
```

That's pretty much it!

NB: invited users are not required to verify their email address, since they are already verified by having received the invitation by email.

-----------------
FILE PATH: kits\remix-fire\authentication\multi-factor-authentication.mdoc

---
status: "published"

title: Set up Multi-Factor Authentication
label: Multi-Factor Authentication
order: 5
description: 'Learn how to setup Multi Factor Authentication with Firebase Auth in your MakerKit application'
---


Multi-Factor Authentication allows users to add an additional layer of protection when logging in to a website, which is ideal for services that tend to be more sensitive or where privacy is paramount. 

At the time of writing, Firebase Auth supports only SMS MFA.

## Enabling Multi-Factor Authentication

Enabling MFA in your Makerkit application requires two steps:

1. You need to upgrade to [Google Cloud Identity Platform](https://cloud.google.com/identity-platform/docs/product-comparison) from the Firebase Console, as it is needed to support MFA
2. Flipping the variable `auth.enableMultiFactorAuth` to `true` in the configuration file, as it is disabled by default

 ```tsx {% title="configuration.ts" %}
auth: {
  // flip this to "true"
  enableMultiFactorAuth: true,
}
```

After enabling MFA, users can set it up from the `Authentication` page at `/settings/authentication`.

{% video src="/assets/images/posts/enabling-multi-factor-authentication.mp4" /%}

-----------------
FILE PATH: kits\remix-fire\authentication\setting-up-firebase-auth.mdoc

---
status: "published"

title: Setting up Firebase Auth with Remix
label: Setting up Firebase Auth
order: 1
description: 'Learn how to set up Firebase Auth in your Remix
application'
---


If you have completed the previous steps, you should have already created a Firebase Project and initialized the repository.

At this point, we can start setting up the authentication providers from the Firebase admin console.

While using the emulator, this step is not needed; feel free to skip this part in the beginning.

## Getting Started with Firebase Auth

From the Firebase console, navigate to the `Authentication` menu item using
the left-hand side panel.

{% img width="1838" height="798" src="/assets/images/docs/auth-setup.webp" alt="Firebase Auth Getting Started" /%}

After clicking on "Getting Started" you should see all the available providers that you can choose:

{% video src="/assets/images/docs/activate-sign-up.mp4" /%}

Please activate all the ones you're planning on using.

We have already configured MakerKit to work with:

 - Email/Password authentication
 - Google oAuth
 - Facebook oAuth (remember to create a FB app before going to production)
 - Email Links
 - Phone Number


-----------------
FILE PATH: kits\remix-fire\authentication\third-party-auth-providers.mdoc

---
status: "published"

title: Set up Third party authentication providers with Remix and Firebase
label: Third-Party Providers
order: 3
description: 'Learn how to setup Third-Party oAuth providers with Firebase
Auth in your Remix MakerKit application'
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

{% alert type="warning" %}
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
FILE PATH: kits\remix-fire\configuration\configuration.mdoc

---
status: "published"

title: Define the global configuration of your MakerKit SaaS application with Remix and Firebase
label: 'Global Configuration'
order: 0
description: 'Learn how to define the global configuration of a MakerKit
application with Remix and Firebase'
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
      checkout: `/resources/stripe/checkout`,
      billingPortal: `/resources/stripe/portal`,
    },
    searchIndex: `/public/search-index`,
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
  appCheckSiteKey: process.env.APPCHECK_KEY,
  navigation: {
    style: NavigationStyle.TopHeader,
  },
  environment: process.env.VERCEL_ENV ?? 'development',
  emulatorHost: process.env.EMULATOR_HOST,
  emulator: process.env.EMULATOR === 'true',
  production: process.env.NODE_ENV === 'production',
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

```
EMULATOR=true

GCLOUD_PROJECT=demo-makerkit
FIREBASE_PROJECT_ID=demo-makerkit
FIREBASE_STORAGE_BUCKET=demo-makerkit.appspot.com
SITE_URL=http://localhost:3000
FIREBASE_AUTH_DOMAIN=localhost
FIREBASE_EMULATOR_HOST=localhost
FIRESTORE_EMULATOR_PORT=8080
FIREBASE_AUTH_EMULATOR_PORT=9099
FIREBASE_STORAGE_EMULATOR_PORT=9199

# Change this with your project's APP ID
FIREBASE_APP_ID=<MAKERKIT_DEV_APP_ID>
# Change this with your project's API KEY
FIREBASE_API_KEY=<MAKERKIT_DEV_API_KEY>

FIRESTORE_EMULATOR_HOST=localhost:8080
FIREBASE_AUTH_EMULATOR_HOST=localhost:9099
FIREBASE_STORAGE_EMULATOR_HOST=localhost:9199
FIREBASE_PUBSUB_EMULATOR_HOST=localhost:8085

SERVICE_ACCOUNT_CLIENT_EMAIL=
SERVICE_ACCOUNT_PRIVATE_KEY=
```

Please remember to update the `FIREBASE_APP_ID` and `FIREBASE_API_KEY` with the ones from your Firebase project ID. The only reason they're predefined is to allow you to quickly start the application.

### Production Environment Variables

When you go to production, create the `.env.production` by copying the content of `.env.production.template`:

```
FIREBASE_API_KEY=
FIREBASE_PROJECT_ID=
FIREBASE_STORAGE_BUCKET=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_APP_ID=
FIREBASE_MEASUREMENT_ID=
SITE_URL=

SERVICE_ACCOUNT_CLIENT_EMAIL=

## SECRET KEYS ARE BEST ADDED TO YOUR CI
SERVICE_ACCOUNT_PRIVATE_KEY=
SECRET_KEY=
SECRET_APPCHECK_KEY=

# Add these in Vercel or .env.local
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
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


-----------------
FILE PATH: kits\remix-fire\configuration\firebase-functions.mdoc

---
status: "published"

title: Setting up Firebase Functions for your MakerKit application
label: Setting up Firebase Functions
order: 2
description: 'Learn how to set up Firebase Functions in your Makerkit SaaS project'
---


{% alert type="info" %}
Setting Firebase Functions is optional. You only need to set up Firebase Functions if you need certain features, such as Firestore Triggers, Storage Triggers, Authentication Blocking Functions, etc.
{% /alert %}


While you may want to use Remix's own API functions for your application's API,
there are many scenarios where you may be required to use Firebase Functions, for example:

1. You want to implement [Authentication Blocking Functions](/blog/tutorials/blocking-authentication-firebase-auth)
2. You need to use Firebase Storage Triggers
3. You need to use Firebase Firestore Triggers

Therefore, it's essential to know how to set up Firebase Functions in a Remix environment, such as the Makerkit SaaS template.

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

