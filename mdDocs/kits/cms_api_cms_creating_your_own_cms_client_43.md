-----------------
FILE PATH: kits/next-supabase-turbo/content/cms-api.mdoc

---
status: "published"
label: "CMS API"
title: "Introducing the CMS API that allows you to pull content from your CMS | Next Supabase SaaS Kit"
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
7. `status` - The status of the content items to fetch. It can be one of `published`, `draft`, `pending` or `review`. By default, only `published` content items are fetched.

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
3. `status` (optional): The status of the content item to fetch. It can be one of `published`, `draft`, `pending` or `review`. By default, only `published` content items are fetched.

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
FILE PATH: kits/next-supabase-turbo/content/cms.mdoc

---
status: "published"
title: "Using the CMS interface in Makerkit | Next.js Supabase SaaS Kit"
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
FILE PATH: kits/next-supabase-turbo/content/creating-your-own-cms-client.mdoc

---
status: "published"
label: "Creating your own CMS client | Next Supabase SaaS Kit"
title: "Creating your own CMS client with the CMS API"
description: "Learn how to create your own CMS client using the CMS API in Makerkit (Next Supabase SaaS Kit) to fetch content from your CMS"
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

export class MyCmsClient implements CmsClient {
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
FILE PATH: kits/next-supabase-turbo/content/keystatic.mdoc

---
status: "published"
title: "Using KeyStatic in the Next.js Supabase SaaS Starter Kit"
label: "Keystatic"
description: "KeyStatic is a CMS that stores data in a JSON file or on your Github repository. It's the default CMS implementation in Makerkit."
order: 1
---

To use Keystatic, you need to set the `CMS_CLIENT` environment variable to `keystatic`. This is the default value, so you don't need to do anything.

```bash
CMS_CLIENT=keystatic
```

## Storage kinds
Keystatic can be used in two ways:

1. **Local storage**: the content is loaded directly from the file system.
2. **GitHub storage**: the content is loaded from a Github repository. This is best for collaborative content editing or if using Cloudflare.

## Local storage

Keystatic uses the `local` storage kind by default. It is easy to get started with and requires no additional setup. Perfect for personal projects.

```bash
NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND=local # local, cloud, github
KEYSTATIC_PATH_PREFIX=apps/web
NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH=./content
```

Depending on whether you use the `local` or `github` storage kind, you may need to change the value of `NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH` (see below).

## GitHub storage

You can also use Keystatic Cloud or GitHub as the storage kind as remote storage.

If `KEYSTATIC_STORAGE_KIND` is set to `cloud`, the following environment variables are required:

```bash
NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND=cloud
KEYSTATIC_STORAGE_PROJECT=project-id
```

If `KEYSTATIC_STORAGE_KIND` is set to `github`, the following environment variables are required:

```bash
NEXT_PUBLIC_KEYSTATIC_STORAGE_KIND=github
NEXT_PUBLIC_KEYSTATIC_STORAGE_REPO=makerkit/next-supabase-saas-kit-turbo-demo
KEYSTATIC_PATH_PREFIX=
NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH=./apps/web/content
KEYSTATIC_GITHUB_TOKEN=github_**********************************************
KEYSTATIC_PATH_PREFIX=apps/web
```

Of course, you need to replace the `NEXT_PUBLIC_KEYSTATIC_STORAGE_REPO` and `KEYSTATIC_GITHUB_TOKEN` with your own values.

### GitHub token
The `KEYSTATIC_GITHUB_TOKEN` is a Github token with read permissions to the repository, which you can generate from the Github Developer Settings in your account.

**Note**: the Github token must have permissions to read to the repository (for read-only).

### GitHub mode
GitHub mode requires the installation of a GitHub app for displaying the admin.

Please refer to the [Keystatic documentation](https://keystatic.com/docs/github-model) for more information.

If your content folder is not at `content`, you can set the `NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH` environment variable to the correct path. For example, if your content folder is at `data/content`, you can set the `NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH` environment variable as:

```bash
NEXT_PUBLIC_KEYSTATIC_CONTENT_PATH=apps/web/data/content
```

## Adding the Keystatic admin to your app

To add the Keystatic admin UI to your app, please run the following command:

```bash
turbo gen keystatic
```

Your app will now have a new route at `/keystatic` where you can manage your content.

**Note**: your Github token must have permissions to read to the repository (for read-only).

**Note**: by default, the Keystatic admin is only available in a development environment. For production environments, please add some sort of authentication. For example, use the `isSuperAdmin` function to check if the user is a super admin and allow them to access the Keystatic admin.


-----------------
FILE PATH: kits/next-supabase-turbo/content/wordpress.mdoc

---
status: "published"
title: "Using Wordpress in Makerkit | Next.js Supabase Turbo"
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
FILE PATH: kits/next-supabase-turbo/content.mdoc

---
status: "published"
title: "Content in Next.js Supabase Turbo"
label: "Content"
order: 8
description: "Content in Next.js Supabase Turbo"
collapsible: true
collapsed: true
---

Content in Next.js Supabase Turbo


-----------------
FILE PATH: kits/next-supabase-turbo/customization/adding-shadcn-ui-components.mdoc

---
status: "published"
label: "Adding Shadcn UI components to your application | Next.js Supabase SaaS Kit"
title: "How to add new Shadcn UI components to your Next.js Supabase application"
order: 2
description: "Update your Next.js Supabase application with new Shadcn UI components"
---

Makerkit implements most of the Shadcn UI components - however, if you need to add a new component, you can do so by following the steps below.

1. Use **Manual Installation** steps from the Shadcn UI component documentation.
2. Add the component to the `packages/ui/src/shadcn` directory.
3. Replace the imports with the relative imports.
4. Export the component by adding a new export to the `package.json` file.
5. Import the component directly from the package.

### Exporting the component

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
FILE PATH: kits/next-supabase-turbo/customization/fonts.mdoc

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

The fonts are defined at `apps/web/lib/fonts.ts`:

```tsx
import { Urbanist as HeadingFont, Inter as SansFont } from 'next/font/google';

/**
 * @sans
 * @description Define here the sans font.
 * By default, it uses the Inter font from Google Fonts.
 */
const sans = SansFont({
  subsets: ['latin'],
  variable: '--font-sans',
  fallback: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
  preload: true,
  weight: ['300', '400', '500', '600', '700'],
});

/**
 * @heading
 * @description Define here the heading font.
 * By default, it uses the Urbanist font from Google Fonts.
 */
const heading = HeadingFont({
  subsets: ['latin'],
  variable: '--font-heading',
  fallback: ['system-ui', 'Helvetica Neue', 'Helvetica', 'Arial'],
  preload: true,
  weight: ['500', '700'],
});

// we export these fonts into the root layout
export { sans, heading };
```

To display a different font, please replace the imports from `next/font/google` if the font is there.

## Removing Apple's system as default font on Apple Devices

To set another font instead of Apple's system font, update the Tailwind variables.

Open `apps/web/styles/shadcn-ui.css.css` and replace `-apple-system` with the font you want to use.

-----------------
FILE PATH: kits/next-supabase-turbo/customization/layout-style.mdoc

---
status: "published"
label: "Layout Style"
title: "Update the default layout style of your SaaS | Next.js Supabase SaaS Starter Kit"
order: 4
description: "Learn how to update the default layout style of your Makerkit application"
---

{% sequence title="How to update the default layout style of your Makerkit application" description="Learn how to update the default layout style of your Makerkit application" %}

[Changing the layout style](#changing-the-layout-style)

[Sidebar Layout](#sidebar-layout)

[Header Layout](#header-layout)

{% /sequence %}

By default, Makerkit uses the `sidebar` layout style for both the user and the team workspaces.

## Changing the layout style

You can change the layout style by setting the `NEXT_PUBLIC_USER_NAVIGATION_STYLE` and `NEXT_PUBLIC_TEAM_NAVIGATION_STYLE` environment variables. The default style is `sidebar`.

To set the layout style to `header`, update the environment variables:

```bash
NEXT_PUBLIC_TEAM_NAVIGATION_STYLE=header
NEXT_PUBLIC_USER_NAVIGATION_STYLE=header
```

## Sidebar Layout

The default layout style is `sidebar`:

{% img src="/assets/images/docs/turbo-sidebar-layout.webp" width="2522" height="1910" /%}

You can customize the sidebar layout to set it as expanded or collapsed. By default, it is always set to expanded.

```
NEXT_PUBLIC_HOME_SIDEBAR_COLLAPSED=true
NEXT_PUBLIC_TEAM_SIDEBAR_COLLAPSED=true
```

You can decide the configuration for the each workspace.

## Header Layout

And here is `header` layout:

{% img src="/assets/images/docs/turbo-header-layout.webp" width="3282" height="1918" /%}


-----------------
FILE PATH: kits/next-supabase-turbo/customization/logo.mdoc

---
status: "published"
label: "Updating the Logo"
title: "Updating the default application logo in your Makerkir application"
order: 1
description: "Updating the default application logo in your Makerkit application"
---


Of course - you'd like to have your own logo in your application. To do so, please update the component at `apps/web/components/app-logo.tsx` with your own logo.

You can either use an SVG component, or drop an image, or anything really. As long as it's rendered within this component, it doesn't matter how you do it.

The logo image will be used across the application, including the auth page, the site header, the site footer, and the app sidebar (when team accounts are disabled).


-----------------
FILE PATH: kits/next-supabase-turbo/customization/tailwind-css.mdoc

---
status: "published"
label: "Tailwind CSS"
title: "Updating the Tailwind CSS Shadcn theme | Next.js Supabase SaaS Kit"
order: -1
description: "How to update the Tailwind CSS theme in your Makerkit application"
---

Makerkit uses [Tailwind CSS](https://tailwindcss.com) to style the application.

### Tailwind CSS (v4) Configuration

If you are using Tailwind CSS v4, you can update the configuration in the `apps/web/styles/global.css` file.

This file imports the following Tailwind CSS configuration files:
- `apps/web/styles/global.css`: - the main global styles for the application (imports the other files)
- `apps/web/styles/theme.css` - this is where we store the Shadcn UI theme for Tailwind CSS v4
- `apps/web/styles/shadcn-ui.css` - where you can override the default style, either define your own colors or pick a theme from the [Shadcn UI Themes page](https://ui.shadcn.com/themes).
- `apps/web/styles/markdoc.css` - for markdoc content
- `apps/web/styles/makerkit.css` - for makerkit-specific styles
- `apps/web/styles/theme.utilities.css` - for utility classes

### Tailwind CSS (v3) Configuration

The older default configuration is at `tooling/tailwind/index.ts` - where you can set the default colors, fonts, and more.

Whenever you add a new package, please remember to update the Tailwind configuration file and add the path to the `content` glob

```tsx
  content: [
    '../../packages/ui/src/**/*.tsx',
    '../../packages/billing/gateway/src/**/*.tsx',
    '../../packages/features/auth/src/**/*.tsx',
    '../../packages/features/notifications/src/**/*.tsx',
    '../../packages/features/admin/src/**/*.tsx',
    '../../packages/features/accounts/src/**/*.tsx',
    '../../packages/features/team-accounts/src/**/*.tsx',
    '../../packages/plugins/testimonial/src/**/*.tsx',
    '../../packages/plugins/roadmap/src/**/*.tsx',
    '../../packages/plugins/kanban/src/**/*.tsx',
    // add the path to the content glob
    '!**/node_modules',
  ],
```

### Shadcn UI Theme

Makerkit uses [Shadcn UI](https://ui.shadcn.com) and it defines the theme according to its guidelines.

You can find the default theme inside the application at `apps/web/styles/global.css`.

If you want to override the default style, either define your own colors or pick a theme from the [Shadcn UI Themes page](https://ui.shadcn.com/themes), and copy/paste the theme into this file.

### Using JSX files

If you plan on using JSX files, please update the global paths in the `content` glob in the Tailwind configuration file.

For targeting both TSX and JSX files, update the `content` glob paths `{tsx|jsx}`. For example:

```tsx
  content: [
    '../../packages/ui/src/**/*.{tsx,jsx}',
    '../../packages/billing/gateway/src/**/*.{tsx,jsx}',
    '../../packages/features/auth/src/**/*.{tsx,jsx}',
    '../../packages/features/notifications/src/**/*.{tsx,jsx}',
    '../../packages/features/admin/src/**/*.{tsx,jsx}',
    '../../packages/features/accounts/src/**/*.{tsx,jsx}',
    '../../packages/features/team-accounts/src/**/*.{tsx,jsx}',
    '../../packages/plugins/testimonial/src/**/*.{tsx,jsx}',
    '../../packages/plugins/roadmap/src/**/*.{tsx,jsx}',
    '../../packages/plugins/kanban/src/**/*.{tsx,jsx}',
    '!**/node_modules',
  ],
```

You will need to do the same in `apps/web/tailwind.config.ts`.

-----------------
FILE PATH: kits/next-supabase-turbo/customization/theme.mdoc

---
status: "published"
label: "Updating the theme"
title: "Updating the Shadcn theme in your Makerkit Application | Next.js Supabase SaaS Kit"
order: 0
description: "How to update the theme in your Makerkit application"
---

Makerkit uses [Shadcn UI](https://ui.shadcn.com) and it defines the theme according to its guidelines.

You can find the default theme inside the application at `apps/web/styles/shadcn-ui.css`.

If you want to override the default style, either define your own colors or pick a theme from the [Shadcn Themes page](https://ui.shadcn.com/themes), and copy/paste the theme into this file.

One difference with the Shadcn UI theme (Tailwind CSS v3 as of this writing), is that you need to convert the CSS variables to using the `hsl` function.

For example, the default theme has the following CSS variable:

```css
--color-primary: 229 231 239;
```

To convert it to the `hsl` function, you need to convert the values to HSL and then convert them to hex.

```css
--color-primary: hsl(229 23% 15%);
```

You can use any AI tool for quickly converting the Shadcn UI CSS theme to the `hsl` function.

-----------------
FILE PATH: kits/next-supabase-turbo/customization.mdoc

---
status: "published"
title: "Customization in Next.js Supabase Turbo"
label: "Customization"
order: 3
description: "Customization in Next.js Supabase Turbo"
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
FILE PATH: kits/next-supabase-turbo/data-fetching/captcha-protection.mdoc

---
status: "published"
description: "Learn how to set up captcha protection for your API routes."
title: "Captcha Protection for your API Routes"
label: "Captcha Protection"
order: 7
---

For captcha protection, we use [Cloudflare Turnstile](https://developers.cloudflare.com/turnstile). 

{% sequence title="How to set up captcha protection for your API routes" description="Learn how to set up captcha protection for your API routes" %}

[Setting up the environment variables](#setting-up-the-environment-variables)

[Enabling the captcha protection](#enabling-the-captcha-protection)

[Retrieving the token](#retrieving-the-token)

[Resetting the token](#resetting-the-token)

[Verifying the token](#verifying-the-token)

{% /sequence %}

## Setting up the environment variables

To enable it, you need to set the following environment variables:

```bash
CAPTCHA_SECRET_TOKEN=
NEXT_PUBLIC_CAPTCHA_SITE_KEY=
```

You can find the `CAPTCHA_SECRET_TOKEN` in the Turnstile configuration. The `NEXT_PUBLIC_CAPTCHA_SITE_KEY` is public and safe to share. Instead, the `CAPTCHA_SECRET_TOKEN` should be kept secret.

This guide assumes you have correctly set up your Turnstile configuration. If you haven't, please refer to the [Turnstile documentation](https://developers.cloudflare.com/turnstile).

## Enabling the captcha protection

When you set the token in the environment variables, the kit will automatically protect your API routes with captcha.

**NB**: you also need to set the token in the Supabase Dashboard!

## Retrieving the Token

To retrieve the captcha token, you can use the `useCaptchaToken` function from `@kit/auth/captcha/client`:

```tsx
import { useCaptchaToken } from '@kit/auth/captcha/client';

function MyComponent() {
  const { captchaToken } = useCaptchaToken();

  const handleSubmit = async (e) => {
    e.preventDefault();

    const response = await fetch('/my-endpoint', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-captcha-token': captchaToken,
      },
      body: JSON.stringify({ message: 'Hello, world!' }),
    });
  };

  // your component code
}
```

When using Server Actions, use `enhanceAction` from `@kit/next/actions`:

```tsx
const MySchema = z.object({
  message: z.string(),
  captchaToken: z.string().min(1),
});

export const myServerAction = enhanceAction(async (data) => {
  // your action code
}, {
  captcha: true,
  schema: MySchema
});
```

When calling the server action, you need to pass the captcha:

```tsx
function MyForm() {
  const { captchaToken, resetCaptchaToken } = useCaptchaToken();

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
       const response = await myServerAction({
        message: 'Hello, world!',
        captchaToken,
      });
    } finally {
      // always reset the token!
      resetCaptchaToken();
    }
  };

  // your component code
}
```

A token is valid for one request only. You need to reset the token after each request.

The library will take care of renewing the token automatically when needed - but you need to reset it manually when consuming the token.

## Resetting the Token

The underlying library [React Turnstile](https://github.com/marsidev/react-turnstile) resets the token automatically - but we need to reset it manually when actually consuming the token since the token is only valid for one request. 

To reset the token, please call `resetCaptchaToken` from the `useCaptchaToken` hook - as shown in the example above.

You should be doing so in all situations, whether the request was successful or not.

## Verifying the Token

To verify the captcha manually server-side, please use the following code:

```tsx
import { verifyCaptchaToken } from '@kit/auth/captcha/server';

function assertCaptchaValidity(request: Request) {
  const token = request.headers.get('x-captcha-token');

  await verifyCaptchaToken(token);
}
```

If you use the utilities provided by the kit `enhanceAction` and `enhanceRouteHandler`, you don't need to worry about this, as the kit will automatically verify the captcha token for you (as long as it is passed).


