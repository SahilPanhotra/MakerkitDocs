-----------------
FILE PATH: kits/next-fire/architecture.mdoc

---
status: "published"
title: Architecture and Folder Structure for your Next.js SaaS application
label: Architecture and Folder Structure
order: 3
description: 'Learn how MakerKit allows you to build a Next.js application
with a scalable and production-grade architecture and folder structure'
collapsible: true
collapsed: true
---

MakerKit's primary goals are:

- **Solid Foundations** - providing you with the necessary code to get up and
running
with a Saas with minimal configuration
- **Extendable and Adaptable** - the ability to change and extend the codebase as efficiently as possible

While it's relatively simple to provide a functioning codebase, it's not always easy to make it:
- **Simple** - a clean codebase quickly understandable by anyone
- **Easy to change** - a codebase that can be easily changed to a
particular domain

Once you start your project, you can begin unpicking and replacing the existing code to adapt it to your application's domain. At the same time, MakerKit may have pushed an update that fixed a bug or added a new cool feature.

The two projects will diverge and inevitably end up in a conflicting state.

Thankfully, Git makes it easy to merge conflicts. But it's still a hassle.

### Framework Architecture

To reduce the number of possible conflicts, MakerKit ships with a particular folder structure:

```
- src
  - lib
  - components
  - pages
    - api
  - core
```

These folders represent four separate layers. MakerKit is an Onion:

{% img width="2103" height="1221" src="/assets/images/docs/architecture.png" alt="MakerKit's architecture" /%}

### Core
Let's talk about the lower level: the `core`. This layer is a collection of building blocks, utilities, and APIs that make up MakerKit.

This part of the boilerplate is configurable and makes no assumptions about your domain.

You are still welcome to change this code to suit your needs, but you can expect most updates from upstream to change the code in this directory.

### Lib and Components

The mid-level is in two separate folders (to avoid excessive nesting):

- `lib`: here is where we write most domain-related business logic (hooks, contexts, queries, mutations, etc.)
- `components`: here is where we write the domain-related components (ex. Profile Page, Create Project form, etc.)

MakerKit comes with some business logic (it would be nearly impossible otherwise to build a genuinely functioning SaaS). You should customize the existing business logic for your application, and it is likely where you will add most of your code.

This layer can still be updated by the `upstream` repository, but you should not expect too many changes unless it's additions for new features or bug fixes.

### Pages

Finally, the `pages` layer is the outer layer of the application.

It's entirely dependent on your application, and you should not expect updates from upstream unless for fixing blatantly buggy code.

## Import Flow

{% img width="1433" height="218" src="/assets/images/docs/nodejs-structure-imports-wrong.png" /%}

We use `eslint` to check that imports are flowing correctly through your application: if this feds you up, you can remove them from the `eslint` configuration at `/.eslintrc.json`;



-----------------
FILE PATH: kits/next-fire/authentication/api-guards.mdoc

---
status: "published"

title: Protect your Next.js application's API with Guards
label: API Guards
order: 9
description: 'MakerKit uses API Guards allow to protect your Next.js API endpoints from unauthorized users, bad input, etc.'
---

API Guards are similar to Page Guards, except they belong to the server and
will protect our API endpoints.

As an example, let's take MakerKit's implementation for handling user
invites.

We want to check that:
- the user is authenticated
- the endpoint method being called is correct

```tsx
import { withAuthedUser } from '~/core/middleware/with-authed-user';

export default function inviteHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const handler = withPipe(
    withMethodsGuard(SUPPORTED_METHODS),
    withAuthedUser,
    inviteMembersToOrganizationHandler
  );

  return withExceptionFilter(req, res)(handler);
}
```

The `withPipe` is a simple utility that can chain API handlers that respect the interface of a Next.js handler. The above handler can be written as follows, by passing in the parameters `req` and `res` explicitly:

```tsx
export default withPipe(
  (req, res) => withMethodsGuard(SUPPORTED_METHODS)(req, res),
  (req, res) => withAuthedUser(req, res),
  (req, res) => inviteMembersToOrganizationHandler(req, res)
);
```

The final `inviteMembersToOrganizationHandler` is the one that gets called at the end and is responsible for returning data to the client.

## Makerkit Default API Guards

As you can see above, we're using various guards: `withMethodsGuard` and
`withAuthedUser`.

These two guards will make sure that the **request will get rejected if the
user isn't authenticated** or if the user **submitted a request using a
disallowed HTTP method**.

You can also simplify the above if you only want to check if the user is
signed-in:

```tsx
export default function myAPIHandler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  await withAuthedUser(req, res);

  // do something with res
}
```

### withAuthedUser

The `withAuthedUser` API guard will reject the API request when the user is not authenticated.

### withMethodsGuard

The `withMethodsGuard` API guard will reject the API request when the method is not among the ones used.

For example, the code below will never execute the handler `myApiHandler` if the method is not `GET`:

```tsx
export default withPipe(
  withMethodsGuard(['GET']),
  myApiHandler
);
```

### withAdmin

The `withAdmin` API guard will ensure the Firebase Admin is initialized before continuing with the rest of the handlers. This is useful before accessing any of the Firebase Admin packages.


-----------------
FILE PATH: kits/next-fire/authentication/app-check.mdoc

---
status: "published"

title: Protect your Next.js application from abuse using Firebase AppCheck
label: Prevent abuse with AppCheck
order: 10
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

Save the public key and add it to your Next.js `.env` file:

```
NEXT_PUBLIC_APPCHECK_KEY=<YOUR_KEY>
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
FILE PATH: kits/next-fire/authentication/auth-flow.mdoc

---
status: "published"

title: Firebase and Next.js authentication flow
label: Auth Flow
order: 2
description: 'Learn how MakerKit handles the authentication flow with Firebase Auth and Next.js'
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

When the user is redirected to a protected page, the user's session authentication state is checked in the `getServerSideProps` function.

### Step 4: User signs out

The client SDK starts a sign-out request when the user decides to sign out.

When Firestore Auth destroys the current user session, the event is intercepted by a listener and then calls the API endpoint to destroy the session cookie.

Here is an illustration of the flows:

{% img src="/assets/images/docs/auth-flow.png" width="2344" height="1166" /%}

## Keeping the client and server sessions in sync

Because the Firebase Session cookies expire after 14 days, we need to keep the client SDK session in sync when the session cookie expires. 

To do so, when the user signs in, we also store a cookie with the expiration date. When the client SDK detects that the server-side session cookie expired, the user gets automatically signed out.

-----------------
FILE PATH: kits/next-fire/authentication/auth-hooks.mdoc

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
FILE PATH: kits/next-fire/authentication/auth-overview.mdoc

---
status: "published"
title: Setting up Firebase and Next.js authentication with MakerKit
label: Auth Overview
order: 0
description: 'MakerKit uses Firebase Authentication to allow access to your Next.js application using oAuth providers and email/password.'
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

{% alert type="warning" title="Remember to enable authentication methods" %}
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

While it sounds great, there is a downside: by server-side rendering the page (SSR), we lose the possibility to statically generate the page (SSG), which is faster.

#### Should you use SSR or SSG?

Ultimately, it comes down to what your needs and preferences are. We recommend:

- using SSG for static pages, such as the home page, your blog, and the
documentation pages
- using SSR for dynamic pages, such as your application's code (the
 pages behind authentication)

The above is the default configuration of the Makerkit's codebase, so if you're OK with that, simply keep using the same patterns as the base application.

-----------------
FILE PATH: kits/next-fire/authentication/auth-ssr.mdoc

---
status: "published"

title: Using SSR with Firebase and Next.js authentication
label: Auth SSR
order: 7
description: 'Learn how to use SSR with Firebase Authentication and
Next.js'
---

MakerKit uses SSR throughout the website. If you choose to use SSR, you can persist the user's authentication on every page.

Whether SSR on every page is something your website may need or not is up to your needs, but MakerKit makes this process very easy.

Let's assume the following is your Next.js Index page, and we want to show the user's Profile avatar if they signed in. Therefore, you will need to fetch the user's data from the `getServerSideProps` function and return it as `props` to the current page.

The MakerKit default template does it for you, but you need to remember to add the following snippet to every page you want the authentication persisted.

```tsx
import { withUserProps } from '~/app/props/with-user-props';

function Index({ user }) {
    // display user in your components
}

export function getServerSideProps(ctx: GetServerSidePropsContext) {
  return withUserProps(ctx);
}
```

MakerKit also provides a `UserSessionContext` context, which you can inject with `useContext` in any of your components. Assuming you have a component `AvatarComponent` in which you want to display your user's avatar:

```tsx
function ProfileAvatar() {
    const { user } = useContext(UserSessionContext);

    return (
        <img src={user.profileURL} alt='Profile Photo' />
    )
}
```

Alternatively, you can use the hook `useUserSession`:

```tsx
function ProfileAvatar() {
    const {auth} = useUserSession();

    return (
        <img src={auth.profileURL} alt='Profile Photo' />
    )
}
```


-----------------
FILE PATH: kits/next-fire/authentication/authentication-troubleshooting.mdoc

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

--BEGIN PRIVATE KEY---
--
***************************
---
--END PRIVATE KEY---

```

It's much longer than the above, but you get the idea.

The key needs to be added to your environment variables settings. Please use your CI to add this key since it's a secret key:

```
SERVICE_ACCOUNT_PRIVATE_KEY=
```

-----------------
FILE PATH: kits/next-fire/authentication/email-link-authentication.mdoc

---
status: "published"

title: Set up Email Link Authentication with Next.js and Firebase
label: Email Link Authentication
order: 4
description: 'Learn how to setup Email Link Authentication with Firebase Auth in
your MakerKit application'
---

Makerkit provides a component and a route to sign your users in using an Email Link.

Thanks to Email Links Authentication, users can enter their email and then sign in by clicking on the link they received to their email.

{% alert type="warning" title="Remember to activate Email Links" %}
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
FILE PATH: kits/next-fire/authentication/email-verification.mdoc

---
status: "published"

title: Requiring Email verification
label: Requiring Email verification
order: 6
description: 'Learn how to enforce email verification for your users with Firebase.'
---

By default, Firebase does not enforce email verification. This means that users can sign up with an email address and password, and then immediately start using your app. However, you may want to require that users verify their email address before they can use your app.

To do so, Makerkit can check that the user has verified their email address before allowing them to use your app from the server side. How? When loading the server side session, we check that the user's property `emailVerified` is `true`. If not, the user is redirected to the sign in page, and signed out.

By default, **this feature is disabled**. To enable it, you need to set the environment variable `NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION` to `true`.

## Enabling requiring email verification

Simply set the following environment variable to `true`:

```bash
NEXT_PUBLIC_REQUIRE_EMAIL_VERIFICATION=true
```

That's pretty much it!

NB: invited users are not required to verify their email address, since they are already verified by having received the invitation by email.

-----------------
FILE PATH: kits/next-fire/authentication/multi-factor-authentication.mdoc

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
FILE PATH: kits/next-fire/authentication/page-guards.mdoc

---
status: "published"

title: Protect your Next.js application's pages with Guards
label: Page Guards
order: 8
description: 'Page Guards allow you to protect certain routes from
unauthorized access. This docs explain how MakerKit makes the process of
protecting your Next.js pages easy and quick.'
---

MakerKit provides various utilities that allow you to restrict access to
certain pages or API according to factors such as sign-in state, role,
membership plan, etc.

## Protecting Next.js Pages

To protect against access to certain pages, we can use the function
`withAppProps`. This function is a server-side props handler that will take
care of checking the following conditions:

- if the user is signed in - otherwise will be redirected back to the sign
in page (this is customizable)
- if the user is onboarded - otherwise will be redirected back to the
onboarding
- if the page they are trying to access requires a particular membership plan

Assuming you want to protect the `Dashboard` page, it will be as easy as:

```tsx
import { withAppProps } from '~/lib/props/with-app-props';

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  return await withAppProps(ctx);
}
```

You will likely want to add additional logic or additional props
to be sent to the page. In this case, you can partially evaluate
`withAppProps` and extend the props with other information:

```tsx
import { withAppProps } from '~/lib/props/with-app-props';

export async function getServerSideProps(
  ctx: GetServerSidePropsContext
) {
  const { props: appProps } =
    await withAppProps(ctx);

  const data = await getDataFromDb();

  return {
    props: {
      ...appProps,
      data,
    }
  };
}
```
This handler takes the following parameters:

```tsx
const DEFAULT_OPTIONS = {
  redirectPath: configuration.paths.signIn,
  locale: 'en',
  localeNamespaces: <string[]>[],
  requirePlans: <string[]>[],
}
```

- `redirectPath`: by default, we redirect the user to the signIn page when
they're not authenticated
- `redirectPath`: by default, we use the `en` locale
- `localeNamespaces`: additional namespaces you want to add to the route
- `requirePlans`: a list of Stripe price IDs that you may want to require to
access this page. If you require more complex logic, it's best to handle it
separately

## Makerkit's default page guards

### withAppProps

The page guard `withAppProps` is ideal for internal pages of your application, i.e. the pages that require that the user is authenticated and that have an active account; i.e. they went past the onboarding:

This page guard will:
- ensure the user is authenticated
- ensure the user created an account after the onboarding flow
- ensure the user belongs to an organization

### withAuthProps

The page guard `withAuthProps` will disallow users from entering authentication routes (sign-in, sign-up, password-reset) while they are authenticated:

This page guard will:
- ensure the user is not authenticated before granting access to pages that **require the user not to be authenticated**

### withTranslationProps

The page guard `withTranslationProps` is used by default by the other guards and is used to load the translations before rendering the page. Use this as the default guard for static pages that do not have any authentication requirement:

This page guard will:
- ensure the translations are loaded onto the page being rendered

-----------------
FILE PATH: kits/next-fire/authentication/setting-up-firebase-auth.mdoc

---
status: "published"

title: Setting up Firebase Auth with Next.js
label: Setting up Firebase Auth
order: 1
description: 'Learn how to set up Firebase Auth in your MakerKit application'
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


