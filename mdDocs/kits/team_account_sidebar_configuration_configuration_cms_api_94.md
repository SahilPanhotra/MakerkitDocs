-----------------
FILE PATH: kits/remix-supabase-turbo/configuration/team-account-sidebar-configuration.mdoc

---
status: "published"
label: "Team Account Navigation Configuration"
title: "Setting your team account navigation configuration | Remix Supabase SaaS Kit"
description: "Learn how to setup the team account navigation of your Remix Supabase application"
order: 6
---

The team account navigation is set at `apps/web/config/team-account-navigation.config.tsx`. We use this configuration to define the menu of the team account. By default, it has four routes: dashboard, settings, members, and billing (if enabled).

We define it in one place so we can build different views at once (for example, the mobile menu).

**Please update this file to add more routes to the sidebar.**

 ```tsx {% title="apps/web/config/team-account-navigation.config.tsx" %}
const getRoutes = (account: string) => [
  {
    label: 'common:dashboardTabLabel',
    path: pathsConfig.app.accountHome.replace('[account]', account),
    Icon: <LayoutDashboard className={iconClasses} />,
    end: true,
  },
  {
    label: 'common:settingsTabLabel',
    collapsible: false,
    children: [
      {
        label: 'common:settingsTabLabel',
        path: createPath(pathsConfig.app.accountSettings, account),
        Icon: <Settings className={iconClasses} />,
      },
      {
        label: 'common:accountMembers',
        path: createPath(pathsConfig.app.accountMembers, account),
        Icon: <Users className={iconClasses} />,
      },
      featureFlagsConfig.enableTeamAccountBilling
        ? {
            label: 'common:billingTabLabel',
            path: createPath(pathsConfig.app.accountBilling, account),
            Icon: <CreditCard className={iconClasses} />,
          }
        : undefined,
    ].filter(Boolean),
  },
];

export function getTeamAccountSidebarConfig(account: string) {
  return SidebarConfigSchema.parse({
    routes: getRoutes(account),
    style: import.meta.env.VITE_TEAM_NAVIGATION_STYLE,
  });
}

function createPath(path: string, account: string) {
  return path.replace('[account]', account);
}
```

You can choose the style of the navigation by setting the `VITE_TEAM_NAVIGATION_STYLE` environment variable. The default style is `sidebar`.

```bash
VITE_TEAM_NAVIGATION_STYLE=sidebar
```

Alternatively, you can set the style to `header`:

```bash
VITE_TEAM_NAVIGATION_STYLE=header
```

-----------------
FILE PATH: kits/remix-supabase-turbo/configuration.mdoc

---
status: "published"
title: "Configuration in Remix Supabase Turbo"
label: "Configuration"
order: 2
description: "Configuration in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Makerkit uses a configuration-first approach, making it easy to customize your application without diving into the code. The configuration layer acts as a bridge between your environment variables and the application, ensuring type-safety and validation.

### How Configuration Works in Makerkit

Configuration in Makerkit follows a simple pattern:

1. Environment variables define the settings
2. Configuration files validate and parse these variables
3. The application consumes the validated configuration

This approach ensures your application never starts with invalid configuration, preventing runtime errors that could affect your users.

## Key Configuration Areas

### 📱 Application Configuration
The core settings of your application like name, URL, and metadata. This configuration defines how your application presents itself to users and search engines.

[Learn more about Application Configuration →](configuration/application-configuration)

### 🔐 Authentication Configuration
Control which authentication methods are available to your users - from password-based auth to magic links and OAuth providers.

[Learn more about Authentication Configuration →](configuration/authentication-configuration)

### 🚩 Feature Flags Configuration
Toggle features on/off across your application. Perfect for rolling out new features or maintaining different versions of your product.

[Learn more about Feature Flags →](configuration/feature-flags-configuration)

### 🛣️ Paths Configuration
Define the routing structure of your application. This configuration helps maintain consistent navigation across your app.

[Learn more about Paths Configuration →](configuration/paths-configuration)

### 📱 Navigation Configuration
Customize the navigation structure for both personal and team accounts. Define what appears in your sidebars and headers.

- [Personal Account Navigation →](configuration/personal-account-sidebar-configuration)
- [Team Account Navigation →](configuration/team-account-sidebar-configuration)

## Environment Variables

All configuration ultimately stems from environment variables. These variables let you maintain different settings across environments without changing code.

[Learn more about Environment Variables →](configuration/environment-variables)

## Production Configuration

When deploying to production, you'll need to ensure all your configuration is properly set up. We've created a comprehensive guide to help you get everything right.

[Learn more about Production Configuration →](going-to-production/checklist)

-----------------
FILE PATH: kits/remix-supabase-turbo/content/cms-api.mdoc

---
status: "published"
label: "CMS API"
title: "Introducing the CMS API that allows you to pull content from your CMS | Remix Supabase SaaS Kit"
description: "Introducing the CMS API that allows you to pull content from your CMS"
order: 3
---

To fetch content from your CMS, you can use the CMS API. The CMS API is an interface that allows you to pull content from your CMS and display it on your website. This is useful if you want to display dynamic content on your website that is stored in your CMS.

### Creating a CMS client

To create a CMS client, you can use the `createCmsClient` function. This function returns a client that you can use to fetch content from your CMS.

```tsx
import { createCmsClient } from '@kit/cms';

const client = await createCmsClient();
```

The implementation depends on the CMS you are using. By default, the CMS client uses the `keystatic` CMS. If you are using a different CMS, you can specify the CMS client to use by setting the `CMS_CLIENT` environment variable.

```
CMS_CLIENT=keystatic
```

### Fetching content items

To fetch content items from your CMS, you can use the `getContentItems` function.

```tsx
// import the createCmsClient function
import { createCmsClient } from '@kit/cms';

// create a client
const client = await createCmsClient();

// Fetch content items
const { items, count } = await client.getContentItems({
  collection: 'posts',
});
```

The `getContentItems` function takes an object with the following properties:
1. `collection` - The collection to fetch content items from.
2. `limit` - The number of content items to fetch.
3. `offset` - The offset to start fetching content items from.
4. `language` - The language to fetch content items in.
5. `sortBy` - The field to sort content items by.
6. `sortDirection` - The direction to sort content items in.

```tsx
import { createCmsClient } from '@kit/cms';

const getContentItems = cache(
  async (language: string | undefined, limit: number, offset: number) => {
    const client = await createCmsClient();

    return client.getContentItems({
      collection: 'posts',
      limit,
      offset,
      language,
      sortBy: 'publishedAt',
      sortDirection: 'desc',
    });
  },
);
```

NB: how these values are used is entirely dependent on the CMS you are using. For example, Wordpress will only support `posts` or `pages` as collections.

Additionally: how the language filtering is implemented is also dependent on the CMS you are using.

### Fetching a single content item

To fetch a single content item from your CMS, you can use the `getContentItemBySlug` function.

```tsx
import { createCmsClient } from '@kit/cms';

const client = await createCmsClient();

// Fetch a single content item
const item = await client.getContentItemBySlug({
  slug: 'hello-world',
  collection: 'posts'
});
```

The `getContentItemBySlug` function takes an object with the following properties:

1. `slug` - The slug of the content item to fetch.
2. `collection` - The collection to fetch the content item from.

#### Rendering pages using content from your CMS

You can use the `getContentItemBySlug` function to fetch content from your CMS and render pages using that content.

For example, if you want to store your Terms and Conditions in your CMS, you can fetch the content item for the Terms and Conditions page and render the page using that content.

{% alert type="info" title="Create the 'pages' collection in your CMS" %}
This example assumes that you have created the 'pages' collection in your CMS. By default, the kit includes the `posts` and `documents` collections.
{% alert /%}

```tsx
import { createCmsClient } from '@kit/cms';

async function TermsAndConditionsPage() {
  const client = await createCmsClient();

  const { content, title } = await client.getContentItemBySlug({
    slug: 'terms-and-conditions',
    collection: 'pages',
  });

  return (
    <div>
      <h1>{title}</h1>
      <div>{content}</div>
    </div>
  );
}
```

-----------------
FILE PATH: kits/remix-supabase-turbo/content/cms.mdoc

---
status: "published"
title: "Using the CMS interface in Makerkit | Remix Supabase SaaS Kit"
label: "CMS"
description: "The CMS library in Makerkit abstracts the implementation from where you store your data. It provides a simple API to interact with your data, and it's easy to extend and customize."
order: 0
---

Makerkit implements a CMS interface that abstracts the implementation from where you store your data. It provides a simple API to interact with your data, and it's easy to extend and customize.

By default, Makerkit ships with two CMS implementations:

1. [Keystatic](https://keystatic.com) - a CMS that stores data in a JSON file or on your Github repository
2. [Wordpress](https://wordpress.org) - needs no introduction

You can also create your own CMS implementation by extending the `CMS` class.

By default, we use Keystatic using `local` mode (eg. storing data in a JSON file). You can change the mode to `github` to store data in your Github repository or `cloud` to use their cloud service.

Local mode is the easiest way to get started since you need no setup. However, assuming you want to use edge-rendering, you'll need to switch to a remote mode (eg. Github) or a remote-only CMS (eg. Wordpress).


-----------------
FILE PATH: kits/remix-supabase-turbo/content/creating-your-own-cms-client.mdoc

---
status: "published"
label: "Creating your own CMS client | Remix Supabase SaaS Kit"
title: "Creating your own CMS client with the CMS API"
description: "Learn how to create your own CMS client using the CMS API in Makerkit (Remix Supabase SaaS Kit) to fetch content from your CMS"
order: 4
---

Makerkit may not be able to accomodate everyone's preferences when it comes to CMSs. After all - there are too many to support! But that's okay - you can create your own CMS client using the CMS API in Makerkit.

To create your own CMS API, you need to implement the following methods:

```tsx
export abstract class CmsClient {
  /**
   * Retrieves content items based on the provided options.
   * @param options - Options for filtering and pagination.
   * @returns A promise that resolves to an array of content items.
   */
  abstract getContentItems(options?: Cms.GetContentItemsOptions): Promise<{
    total: number;
    items: Cms.ContentItem[];
  }>;

  /**
   * Retrieves a content item by its ID and type.
   * @returns A promise that resolves to the content item, or undefined if not found.
   */
  abstract getContentItemBySlug(params: {
    slug: string;
    collection: string;
  }): Promise<Cms.ContentItem | undefined>;

  /**
   * Retrieves categories based on the provided options.
   * @param options - Options for filtering and pagination.
   * @returns A promise that resolves to an array of categories.
   */
  abstract getCategories(
    options?: Cms.GetCategoriesOptions,
  ): Promise<Cms.Category[]>;

  /**
   * Retrieves a category by its slug.
   * @param slug - The slug of the category.
   * @returns A promise that resolves to the category, or undefined if not found.
   */
  abstract getCategoryBySlug(slug: string): Promise<Cms.Category | undefined>;

  /**
   * Retrieves tags based on the provided options.
   * @param options - Options for filtering and pagination.
   * @returns A promise that resolves to an array of tags.
   */
  abstract getTags(options?: Cms.GetTagsOptions): Promise<Cms.Tag[]>;

  /**
   * Retrieves a tag by its slug.
   * @param slug - The slug of the tag.
   * @returns A promise that resolves to the tag, or undefined if not found.
   */
  abstract getTagBySlug(slug: string): Promise<Cms.Tag | undefined>;
}
```

For a detailed view of the CMS interface, please refer to the API at `packages/cms/core/src/cms-client.ts`.

For example, let' assume you have an HTTP API to a private repository you can access at `https://my-cms-api.com`. You can create a CMS client that interacts with this API as follows:

```tsx
import { CmsClient } from '@kit/cms';

export class MyCmsClient extends CmsClient {
  async getContentItems(options) {
    const response = await fetch('https://my-cms-api.com/content', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(options),
    });

    const { total, items } = await response.json();

    return { total, items };
  }

  async getContentItemBySlug({ slug, collection }) {
    const response = await fetch(
      `https://my-cms-api.com/content/${collection}/${slug}`,
    );

    if (response.status === 404) {
      return undefined;
    }

    return response.json();
  }

  async getCategories(options) {
    const response = await fetch('https://my-cms-api.com/categories', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(options),
    });

    return response.json();
  }

  async getCategoryBySlug(slug) {
    const response = await fetch(`https://my-cms-api.com/categories/${slug}`);

    if (response.status === 404) {
      return undefined;
    }

    return response.json();
  }

  async getTags(options) {
    const response = await fetch('https://my-cms-api.com/tags', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(options),
    });

    return response.json();
  }

  async getTagBySlug(slug) {
    const response = await fetch(`https://my-cms-api.com/tags/${slug}`);

    if (response.status === 404) {
      return undefined;
    }

    return response.json();
  }
}
```

Of course, you can do the same using SDKs of your preferred CMS client, such as Payload CMS, Strapi, Sanity, and so on.


-----------------
FILE PATH: kits/remix-supabase-turbo/content/keystatic.mdoc

---
status: "published"
title: "Using KeyStatic with the Remix Supabase SaaS Starter Kit"
label: "Keystatic"
description: "KeyStatic is a CMS that stores data in a JSON file or on your Github repository. It's the default CMS implementation in Makerkit."
order: 1
---

This implementation is used when the host app's environment variable is set as:

```bash
CMS_CLIENT=keystatic
```

Additionally, the following environment variables may be required:

```bash
VITE_KEYSTATIC_STORAGE_KIND=local # local, cloud, github
VITE_KEYSTATIC_CONTENT_PATH=content # apps/web/content
KEYSTATIC_PATH_PREFIX=apps/web
```

You can also use Keystatic Cloud or GitHub as the storage kind as remote storage.

If `VITE_KEYSTATIC_STORAGE_KIND` is set to `cloud`, the following environment variables are required:

```bash
KEYSTATIC_STORAGE_KIND=cloud
KEYSTATIC_STORAGE_PROJECT=project-id
```

If `KEYSTATIC_STORAGE_KIND` is set to `github`, the following environment variables are required:

```bash
VITE_KEYSTATIC_STORAGE_KIND=github
VITE_KEYSTATIC_STORAGE_REPO=makerkit/remix-supabase-saas-kit-turbo-demo
KEYSTATIC_GITHUB_TOKEN=github_*****************************************************
KEYSTATIC_PATH_PREFIX=apps/web
VITE_KEYSTATIC_CONTENT_PATH=./content
```

Of course, you need to replace the `VITE_KEYSTATIC_STORAGE_KIND` and `KEYSTATIC_GITHUB_TOKEN` with your own values.

GitHub mode requires the installation of a GitHub app for displaying the admin.

Please refer to the [Keystatic documentation](https://keystatic.com/docs/github-model) for more information.

If your content folder is not at `content`, you can set the `VITE_KEYSTATIC_CONTENT_PATH` environment variable to the correct path. For example, if your content folder is at `data/content`, you can set the `VITE_KEYSTATIC_CONTENT_PATH` environment variable as:

```bash
VITE_KEYSTATIC_CONTENT_PATH=data/content
```

**Note**: your Github token must have permissions to read to the repository (for read-only).


-----------------
FILE PATH: kits/remix-supabase-turbo/content/wordpress.mdoc

---
status: "published"
title: "Using Wordpress in Makerkit | Remix Supabase Kit"
label: "Wordpress"
description: "Wordpress is a popular CMS that you can use with Makerkit. Learn how to set it up and use it in your project."
order: 2
---

The Wordpress implementation is used when the host app's environment variable is set as:

```bash
CMS_CLIENT=wordpress
```

Additionally, please set the following environment variables:

```bash
WORDPRESS_API_URL=http://localhost:8080
```

For development purposes, we ship a Docker container that runs a Wordpress instance. To start the container, run:

```bash
docker-compose up
```

or

```bash
pnpm run start
```

from this package's root directory.

The credentials for the Wordpress instance are:

```bash
WORDPRESS_DB_HOST=db
WORDPRESS_DB_USER=wordpress
WORDPRESS_DB_PASSWORD=wordpress
WORDPRESS_DB_NAME=wordpress
```

You will be asked to set up the Wordpress instance when you visit `http://localhost:8080` for the first time.

## Note for Wordpress REST API

To make the REST API in your Wordpress instance work, please change the permalink structure to `/%post%/` from the Wordpress admin panel.

## Blog

To include Blog Posts from Wordpress - please create a **post** with category named `blog` and add posts to it.

## Documentation

To include Documentation from Wordpress - please create a **page** with category named `documentation` and add pages to it.

This **involves enabling categories for pages. To do this, add the following code to your theme's `functions.php` file:

```php
function add_categories_to_pages() {
    register_taxonomy_for_object_type('category', 'page');
}
add_action('init', 'add_categories_to_pages');
```

Please refer to `wp-content/themes/twentytwentyfour/functions.php` for an example of a theme that includes this code.

## Language Filtering

To make the language filtering work, please add a tag with the language name to the post. For example, if you have a post in English, add the tag `en` to it, and if you have a post in German, add the tag `de` to it.

-----------------
FILE PATH: kits/remix-supabase-turbo/content.mdoc

---
status: "published"
title: "Content in Remix Supabase Turbo"
label: "Content"
order: 8
description: "Content in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Content in Remix Supabase Turbo


-----------------
FILE PATH: kits/remix-supabase-turbo/customization/adding-shadcn-ui-components.mdoc

---
status: "published"

label: "Adding Shadcn UI components"
title: "How to add new Shadcn UI components to your Remix Supabase application"
order: 2
description: "Update your Remix Supabase application with new Shadcn UI components"
---


To install a Shadcn UI component, you can use the following command:

```bash
npx shadcn@latest add <component> -c ./packages/ui
```

For example, to install the `Button` component, you can use the following command:

```bash
npx shadcn@latest add button -c ./packages/ui
```

We pass the `--path` flag to specify the path where the component should be installed. You may need to adjust the path based on your project structure.

### Update the imports

**NB**: you may need to update the imports to the `cn` utility function to use the relative imports because it somehow breaks. Please do that.

### Export the component

To achieve optimal tree-shaking, we export each component individually using the `exports` field in the `package.json` file. This allows you to import the component directly from the package.

Once the component has been created, you can export by adding a new export to the `package.json` file.

We assume that the component is located at `src/shadcn/avatar.tsx`. To export the component, you can append a new export field to the `exports` map inside the `package.json` file:

```json
{
  "exports": {
    "./avatar": "./src/shadcn/avatar.tsx"
  }
}
```

Now you can import it directly from the package:

```tsx
import { Avatar } from '@kit/ui/avatar';
```

**NB**: this is an example, you need to adjust the component name based on the component you are exporting.

-----------------
FILE PATH: kits/remix-supabase-turbo/customization/fonts.mdoc

---
status: "published"

label: "Updating Fonts"
title: "Update the default fonts of your SaaS"
order: 3
description: "Learn how to update the fonts of your Makerkit application"
---


By default, Makerkit uses two fonts:

1. `font-sans`: using Apple's default font on Apple devices, or `Inter` on others
2. `font-heading`: uses `Urbanist` from Google Fonts

The fonts are defined at `apps/web/styles/global.css`:

```css
@import url('//fonts.googleapis.com/css2?family=Inter:wght@300;400;500;700;800&family=Urbanist:wght@500;700&display=swap&display=swap');

--font-sans: -apple-system, Inter, Helvetica, Arial, sans-serif;
--font-heading: Urbanist, var(--font-sans);
```

Please change them to your preferred fonts.

-----------------
FILE PATH: kits/remix-supabase-turbo/customization/layout-style.mdoc

---
status: "published"
label: "Layout Style"
title: "Update the default layout style of your SaaS | Remix Supabase SaaS Starter Kit"
order: 4
description: "Learn how to update the default layout style of your Makerkit application"
---

By default, Makerkit uses the `sidebar` layout style for both the user and the team workspaces.

You can change the layout style by setting the `VITE_USER_NAVIGATION_STYLE` and `VITE_TEAM_NAVIGATION_STYLE` environment variables. The default style is `sidebar`.

To set the layout style to `header`, update the environment variables:

```bash
VITE_TEAM_NAVIGATION_STYLE=header
VITE_USER_NAVIGATION_STYLE=header
```

The default layout style is `sidebar`:

{% img src="/assets/images/docs/turbo-sidebar-layout.webp" width="2522" height="1910" /%}

And here is `header` layout:

{% img src="/assets/images/docs/turbo-header-layout.webp" width="3282" height="1918" /%}


-----------------
FILE PATH: kits/remix-supabase-turbo/customization/logo.mdoc

---
status: "published"

label: "Updating the Logo"
title: "Updating the default application logo in your Makerkir application"
order: 1
description: "Updating the default application logo in your Makerkit application"
---


Of course - you'd like to have your own logo in your application. To do so, please update the component at `apps/web/components/app-logo.tsx`.

You can either use an SVG component, or drop an image. As long as it's rendered within this component, it doesn't matter how you do it.

The logo image will be used across the application, including the auth page, the site header, the site footer, and the app sidebar (when team accounts are disabled).

-----------------
FILE PATH: kits/remix-supabase-turbo/customization/theme.mdoc

---
status: "published"
label: "Updating the theme"
title: "Updating the Shadcn theme in your Makerkit Application | Remix Supabase SaaS Kit"
order: 0
description: "How to update the theme in your  Remix Supabase Makerkit application"
---

Makerkit uses [Shadcn UI](https://ui.shadcn.com) and it defines the theme according to its guidelines.

You can find the default theme inside the application at `apps/web/styles/global.css`.

If you want to override the default style, either define your own colors or pick a theme from the [Shadcn Themes page](https://ui.shadcn.com/themes), and copy/paste the theme into this file.

-----------------
FILE PATH: kits/remix-supabase-turbo/customization.mdoc

---
status: "published"
title: "Customization in Remix Supabase Turbo"
label: "Customization"
order: 3
description: "Customization in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Welcome to the customization guide!

One of Makerkit's greatest strengths is its flexibility - you can customize nearly every aspect of your application to match your brand and requirements. Let's explore the key areas you can personalize.

## Core Customization Areas

### Environment Variables
The foundation of customization in Makerkit starts with environment variables. These control everything from your app's name and description to feature flags and authentication methods. Most customizations can be made without touching the source code - just update your environment variables and you're good to go!

### Branding and Design
Make your app uniquely yours with:
- Custom colors and themes using Shadcn UI
- Your own logo and favicons
- Personalized fonts (using any Google Font or system font)
- Light and dark mode customization

### Layout and Navigation
Choose how your app looks and feels:
- Switch between sidebar and header navigation layouts
- Customize navigation items for both user and team workspaces
- Add or remove menu entries based on your needs

### Feature Configuration
Enable or disable various features to match your business needs:
- Authentication methods (password, magic link, OAuth)
- Team accounts and permissions
- Billing capabilities
- Notifications system

### Content Management
Manage your marketing content and documentation using either:
- Keystatic (default) - perfect for Git-based content management
- WordPress - ideal for traditional CMS workflows

## Where to Start?

1. Begin with the [Environment Variables](customization/environment-variables) to set up your basic configuration
2. Move on to [Theme Customization](customization/theme) to match your brand colors
3. Update your [Logo](customization/logo) and [Fonts](/docs/next-supabase-turbo/fonts)
4. Configure your [Layout Style](customization/layout-style)

## A Note on Customization

While Makerkit is highly customizable, we've worked hard to make these customizations manageable and maintainable. Most changes can be made through configuration rather than code changes, making it easier to update to newer versions of Makerkit in the future.

Remember - you don't need to customize everything at once! Start with the essentials (branding, layout) and progressively enhance your application as needed.


-----------------
FILE PATH: kits/remix-supabase-turbo/data-fetching.mdoc

---
status: "published"
title: "Data Fetching in Remix Supabase Turbo"
label: "Data Fetching"
order: 5
description: "Data Fetching in Remix Supabase Turbo"
collapsible: true
collapsed: true
---

Here's a draft of the Data Fetching pillar page for Makerkit:

# Data Fetching in Makerkit

Makerkit provides a robust approach to fetching and managing data in your Remix Supabase application. Whether you're building server components, handling real-time updates, or managing client-side state, Makerkit offers patterns and utilities to make data fetching clean and efficient.

## Data Loading Patterns

Common patterns for loading data include:

- Server Component data fetching for initial page loads
- React Query for client-side data management
- Server Actions for mutations
- CSRF and Captcha protection for API Routes

## Navigation

In the section below, you'll find detailed information on each of these patterns:

- [Supabase Clients](data-fetching/supabase-clients)
- [Server Components](data-fetching/server-components)
- [Server Actions](data-fetching/server-actions)
- [Route Handlers](data-fetching/route-handlers)
- [CSRF Protection](data-fetching/csrf-protection)
- [Captcha Protection](data-fetching/captcha-protection)

-----------------
FILE PATH: kits/remix-supabase-turbo/development/adding-turborepo-app.mdoc

---
status: "published"
label: "Adding a Turborepo application"
title: "Add a new Turborepo application to your Makerkit application | Remix Supabase"
description: "Learn how to add a new Turborepo application to your Makerkit application"
order: 12
---


This is an **advanced topic** - you should only follow these instructions if you are placing a new app within your monorepo and want to keep pulling updates from the Makerkit repository.

In some ways - creating a new repository may be the easiest way to manage your application. However, if you want to keep your application within the monorepo and pull updates from the Makerkit repository, you can follow these instructions.

---


To pull updates into a separate application outside of `web` - we can use `git subtree`.

Basically, we will create a subtree at `apps/web` and create a new remote branch for the subtree. When we create a new application, we will pull the subtree into the new application. This allows us to keep it in sync with the `apps/web` folder.

### 1. Create a subtree

First, we need to create a subtree for the `apps/web` folder. We will create a new branch called `web` and create a subtree for the `apps/web` folder. We will create a branch named `web-branch` and create a subtree for the `apps/web` folder.

```bash
git subtree split --prefix=apps/web --branch web-branch
```

### 2. Create a new application

Now, we can create a new application in the `apps` folder. For example, let's create a new application called `api`.

Let's say we want to create a new app `pdf-chat` at `apps/pdf-chat` with the same structure as the `apps/web` folder (which acts as the template for all new apps).

```bash
git subtree add --prefix=apps/pdf-chat origin web-branch --squash
```

You should now be able to see the `apps/pdf-chat` folder with the contents of the `apps/web` folder.

### 3. Update the new application

When you want to update the new application, follow these steps:

#### Pull updates from the Makerkit repository

The command below will update all the changes from the Makerkit repository:

```bash
git pull upstream main
```

#### Push the web-branch updates

After you have pulled the updates from the Makerkit repository, you can split the branch again and push the updates to the `web-branch`:

```bash
git subtree split --prefix=apps/web --branch web-branch
```

Now, you can push the updates to the `web-branch`:

```bash
git push origin web-branch
```

#### Pull the updates to the new application

Now, you can pull the updates to the new application:

```bash
git subtree pull --prefix=apps/pdf-chat origin web-branch --squash
```


