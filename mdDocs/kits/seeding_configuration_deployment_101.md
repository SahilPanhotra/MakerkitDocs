-----------------
FILE PATH: kits/supamode/configuration/seeding.mdoc

---
status: "published"
title: "Creating Custom Permission Seeds for Supamode"
label: "Custom Permission Seeds"
order: 1
description: "Understanding how to create a seed for your Supamode application"
---

In this comprehensive guide, we'll explore how to create tailored permission structures that perfectly match your organization's unique access control requirements.

## Key Learning Objectives:

- Master the Supamode seed generation architecture
- Build custom permission hierarchies from scratch
- Implement business-specific access control patterns
- Deploy scalable, maintainable permission structures

# Understanding Supamode's Seed Architecture 🏗️

Before diving into custom implementations, let's understand how Supamode's seed system works under the hood.

The default seed schema is a simplified version of the Makerkit SaaS Starter
Kit. Therefore, the default seed file is based off of that.

The recommendation is that instead you modify the default seed with mock
data created based off your own schema.

## The Seed Generator Pattern

Supamode uses a **builder pattern** to construct permission structures programmatically:

```typescript
// Core Architecture
const app = new SupamodeSeedGenerator();

// 1. Create entities
const role = Role.create({ app, id: 'custom_role', config: {...} });
const permission = Permission.create({ app, id: 'custom_perm', config: {...} });

// 2. Establish relationships
role.addPermission(permission);

// 3. Generate SQL
const sql = app.generateSql();
```

**Key Benefits:**
- **Type Safety** - TypeScript ensures correct configuration
- **Relationship Validation** - Prevents orphaned permissions
- **SQL Generation** - Automatically creates deployment scripts
- **Reusability** - Seed templates can be shared across projects

## Seed File Structure

Every seed file follows this consistent pattern:

```typescript
// 1. Import the generator classes
import { Account, Permission, PermissionGroup, Role, SupamodeSeedGenerator } from '../generator';

// 2. Initialize the generator
const app = new SupamodeSeedGenerator();

// 3. Define your structure (accounts, roles, permissions, groups)
// ... your custom definitions ...

// 4. Export the configured generator
export default app;
```

---

# Building Your First Custom Seed 🚀

Let's create a practical example: a **content management system** with editorial workflows.

## Step 1: Define Your Business Requirements

**Scenario:** Online magazine with the following roles:
- **Editor-in-Chief** - Full editorial control
- **Senior Editor** - Manages content and junior staff
- **Staff Writer** - Creates and edits own content
- **Freelancer** - Limited content creation
- **Subscriber** - Read-only access to published content

## Step 2: Create the Seed File

1. Create `packages/schema/src/templates/editorial-seed.ts`
2. Add the following code to define roles, permissions, and groups:

```typescript
import {
  Account,
  Permission,
  PermissionGroup,
  Role,
  SupamodeSeedGenerator,
} from '../generator';

const app = new SupamodeSeedGenerator();

// ========================================
// SECTION 1: DEFINE ROLES
// ========================================

const editorInChiefRole = Role.create({
  app,
  id: 'editor_in_chief',
  config: {
    name: 'Editor-in-Chief',
    description: 'Full editorial control and staff management',
    priority: 95, // High authority
    metadata: {
      department: 'Editorial',
      can_publish: true
    }
  },
});

const seniorEditorRole = Role.create({
  app,
  id: 'senior_editor',
  config: {
    name: 'Senior Editor',
    description: 'Content oversight and team coordination',
    priority: 80,
    metadata: {
      department: 'Editorial',
      can_assign_stories: true
    }
  },
});

const staffWriterRole = Role.create({
  app,
  id: 'staff_writer',
  config: {
    name: 'Staff Writer',
    description: 'Full-time content creator',
    priority: 60,
    metadata: {
      employment_type: 'full_time'
    }
  },
});

const freelancerRole = Role.create({
  app,
  id: 'freelancer',
  config: {
    name: 'Freelancer',
    description: 'Contract-based content contributor',
    priority: 40,
    metadata: {
      employment_type: 'contract'
    }
  },
});

const subscriberRole = Role.create({
  app,
  id: 'subscriber',
  config: {
    name: 'Subscriber',
    description: 'Registered reader with premium access',
    priority: 20
  },
});
```

## Step 3: Create Business-Specific Permissions

Now, let's define the permissions that align with our editorial workflow.

```typescript
// ========================================
// SECTION 2: CONTENT PERMISSIONS
// ========================================

// Article Management
const createArticles = Permission.createDataPermission({
  app,
  id: 'create_articles',
  name: 'Create Articles',
  description: 'Can create new article drafts',
  scope: 'table',
  schema_name: 'public',
  table_name: 'articles',
  action: 'insert'
});

const editOwnArticles = Permission.createDataPermission({
  app,
  id: 'edit_own_articles',
  name: 'Edit Own Articles',
  description: 'Can edit articles they authored',
  scope: 'table',
  schema_name: 'public',
  table_name: 'articles',
  action: 'update',
  conditions: { author_id: '$CURRENT_USER_ID' } // Row-level security
});

const editAllArticles = Permission.createDataPermission({
  app,
  id: 'edit_all_articles',
  name: 'Edit All Articles',
  description: 'Can edit any article regardless of author',
  scope: 'table',
  schema_name: 'public',
  table_name: 'articles',
  action: 'update'
});

const publishArticles = Permission.createDataPermission({
  app,
  id: 'publish_articles',
  name: 'Publish Articles',
  description: 'Can change article status to published',
  scope: 'column',
  schema_name: 'public',
  table_name: 'articles',
  column_name: 'status',
  action: 'update',
  conditions: { status: { $in: ['draft', 'review', 'published'] } }
});

// User Management Permissions
const manageEditorialStaff = Permission.createSystemPermission({
  app,
  id: 'manage_editorial_staff',
  resource: 'account',
  action: 'update',
  name: 'Manage Editorial Staff',
  description: 'Can update staff member accounts and assignments'
});

const viewAnalytics = Permission.createDataPermission({
  app,
  id: 'view_analytics',
  name: 'View Analytics',
  description: 'Access to content performance metrics',
  scope: 'table',
  schema_name: 'public',
  table_name: 'article_analytics',
  action: 'select'
});
```

## Step 4: Organize with Permission Groups

Permission groups help us manage related permissions together, making it easier to assign them to roles.

```typescript
// ========================================
// SECTION 3: PERMISSION GROUPS
// ========================================

const editorialManagementGroup = PermissionGroup.create({
  app,
  id: 'editorial_management',
  config: {
    name: 'Editorial Management',
    description: 'Full editorial oversight and staff management',
  },
});

editorialManagementGroup.addPermissions([
  createArticles,
  editAllArticles,
  publishArticles,
  manageEditorialStaff,
  viewAnalytics
]);

const contentCreationGroup = PermissionGroup.create({
  app,
  id: 'content_creation',
  config: {
    name: 'Content Creation',
    description: 'Core content creation and editing capabilities',
  },
});

contentCreationGroup.addPermissions([
  createArticles,
  editOwnArticles,
  viewAnalytics
]);

const readOnlyGroup = PermissionGroup.create({
  app,
  id: 'subscriber_access',
  config: {
    name: 'Subscriber Access',
    description: 'Read access to published content',
  },
});

// Only published articles for subscribers
const readPublishedArticles = Permission.createDataPermission({
  app,
  id: 'read_published_articles',
  name: 'Read Published Articles',
  description: 'Can view published articles only',
  scope: 'table',
  schema_name: 'public',
  table_name: 'articles',
  action: 'select',
  conditions: { status: 'published' }
});

readOnlyGroup.addPermissions([readPublishedArticles]);
```

## Step 5: Assign Groups to Roles

Now that we have our roles, permissions, and groups defined, we can assign them to the appropriate roles.

```typescript
// ========================================
// SECTION 4: ROLE ASSIGNMENTS
// ========================================

// Editor-in-Chief gets full editorial management
editorInChiefRole.addPermissionGroup({ group: editorialManagementGroup });

// Senior Editors get content creation + some management permissions
seniorEditorRole.addPermissionGroup({ group: contentCreationGroup });
seniorEditorRole.addPermission(editAllArticles); // Can edit any article
seniorEditorRole.addPermission(publishArticles); // Can publish

// Staff Writers get core content creation
staffWriterRole.addPermissionGroup({ group: contentCreationGroup });

// Freelancers get limited content creation (no analytics)
freelancerRole.addPermission(createArticles);
freelancerRole.addPermission(editOwnArticles);

// Subscribers get read-only access
subscriberRole.addPermissionGroup({ group: readOnlyGroup });
```

## Step 6: Create Test Accounts

We can now create test accounts for our test users. We can either reference existing accounts by passing their IDs, or create new ones directly in the seed file.

- if we pass an `ID` - we expect the user to exist in the database
- if the `ID` does not exist, we will create a new account with the provided metadata.

```typescript
// ========================================
// SECTION 5: TEST ACCOUNTS
// ========================================

const editorAccount = Account.create({
  app,
  id: '550e8400-e29b-41d4-a716-446655440001',
  config: {
    metadata: {
      display_name: 'Sarah Johnson',
      email: 'sarah.johnson@magazine.com',
      department: 'Editorial',
      hire_date: '2023-01-15'
    },
  },
});

const writerAccount = Account.create({
  app,
  id: '550e8400-e29b-41d4-a716-446655440002',
  config: {
    metadata: {
      display_name: 'Mike Chen',
      email: 'mike.chen@magazine.com',
      department: 'Editorial',
      specialization: 'Technology'
    },
  },
});

const freelancerAccount = Account.create({
  app,
  id: '550e8400-e29b-41d4-a716-446655440003',
  config: {
    metadata: {
      display_name: 'Alex Rivera',
      email: 'alex.rivera@freelance.com',
      contract_type: 'per_article'
    },
  },
});

// Assign roles to accounts
editorAccount.assignRole(editorInChiefRole);
writerAccount.assignRole(staffWriterRole);
freelancerAccount.assignRole(freelancerRole);

export default app;
```

---

# Advanced Permission Patterns 🎯

Now let's explore sophisticated patterns for complex business requirements.

## Time-Based Access Control

Perfect for temporary permissions or seasonal access:

```typescript
// Temporary elevated access for special projects
const specialProjectRole = Role.create({
  app,
  id: 'special_project_lead',
  config: {
    name: 'Special Project Lead',
    description: 'Temporary leadership role for Q4 campaign',
    priority: 85,
    valid_from: new Date('2025-10-01'),
    valid_until: new Date('2025-12-31'), // Expires automatically
    metadata: {}
  },
});
```

## Conditional Permissions with Row-Level Security

Implement sophisticated access patterns:

```typescript
// Writers can only edit articles in 'draft' or 'revision' status
const editDraftArticles = Permission.createDataPermission({
  app,
  id: 'edit_draft_articles',
  name: 'Edit Draft Articles',
  description: 'Can edit articles that are not yet published',
  scope: 'table',
  schema_name: 'public',
  table_name: 'articles',
  action: 'update',
  conditions: {
    author_id: '$CURRENT_USER_ID',
    status: { $in: ['draft', 'revision'] },
    created_at: { $gte: '$CURRENT_DATE - INTERVAL 30 days' } // Recent articles only
  }
});
```

## Permission Overrides for Exceptional Cases

Handle special circumstances gracefully:

```typescript
// Grant temporary admin access to a writer for migration
writerAccount.grantPermission({
  permission: editAllArticles,
  valid_until: new Date('2025-03-31'),
  metadata: {
    reason: 'Database migration assistance',
    approved_by: 'editor_in_chief',
    ticket_id: 'MAINT-2025-001'
  }
});

// Revoke specific permission (emergency access control)
freelancerAccount.denyPermission({
  permission: createArticles,
  valid_until: new Date('2025-02-15'),
  metadata: {
    reason: 'Contract dispute - suspend access pending resolution',
    approved_by: 'legal_department'
  }
});
```

---

# Business-Specific Seed Examples 📊

## E-Commerce Platform Seed

```typescript
// Roles for online store management
const storeManagerRole = Role.create({
  app,
  id: 'store_manager',
  config: {
    name: 'Store Manager',
    description: 'Full store operations management',
    priority: 90
  }
});

// Inventory permissions
const manageInventory = Permission.createDataPermission({
  app,
  id: 'manage_inventory',
  name: 'Manage Inventory',
  description: 'Full inventory control including stock levels',
  scope: 'table',
  schema_name: 'public',
  table_name: 'products',
  action: '*'
});

// Customer service permissions
const viewCustomerOrders = Permission.createDataPermission({
  app,
  id: 'view_customer_orders',
  name: 'View Customer Orders',
  description: 'Access customer order history for support',
  scope: 'table',
  schema_name: 'public',
  table_name: 'orders',
  action: 'select',
  conditions: {
    status: { $in: ['pending', 'processing', 'shipped'] }
  }
});
```

## Healthcare System Seed

```typescript
// HIPAA-compliant role structure
const doctorRole = Role.create({
  app,
  id: 'doctor',
  config: {
    name: 'Doctor',
    description: 'Licensed physician with patient care access',
    priority: 85,
    metadata: {
      license_required: true,
      hipaa_certified: true
    }
  }
});

// Restricted patient data access
const viewPatientRecords = Permission.createDataPermission({
  app,
  id: 'view_patient_records',
  name: 'View Patient Records',
  description: 'Access to assigned patient medical records',
  scope: 'table',
  schema_name: 'medical',
  table_name: 'patient_records',
  action: 'select',
  conditions: {
    // Only patients assigned to this doctor
    assigned_doctor_id: '$CURRENT_USER_ID',
    // Only active cases
    case_status: 'active'
  }
});
```

## SaaS Multi-Tenant Seed

```typescript
// Tenant-isolated permissions
const tenantAdminRole = Role.create({
  app,
  id: 'tenant_admin',
  config: {
    name: 'Tenant Administrator',
    description: 'Admin access within tenant boundary',
    priority: 80
  }
});

const manageTenantData = Permission.createDataPermission({
  app,
  id: 'manage_tenant_data',
  name: 'Manage Tenant Data',
  description: 'Full access to tenant-specific data',
  scope: 'table',
  schema_name: 'public',
  table_name: '*',
  action: '*',
  conditions: {
    // Tenant isolation
    tenant_id: '$CURRENT_USER_TENANT_ID'
  }
});
```

---

# Generating and Deploying Your Custom Seed 🚀

## Step 1: Generate Your Seed

```bash
# Generate your custom seed
pnpm --filter '@kit/schema' schema:gen-seed -- --seed editorial --output out/editorial-permissions.sql
```

The command will:
1. Execute the `schema:gen-seed` command from the `@kit/schema` package.
2. Use the `editorial` seed template defined in `packages/schema/src/templates/editorial-seed.ts`.
3. Output the generated SQL to `out/editorial-permissions.sql`.

## Step 2: Create Migration

In the next step, we will create a migration to apply this seed to your Supabase database.

```bash
# Create Supabase migration
supabase migration new editorial_permissions_setup
```

Now copy the generated SQL file name into the newly created migration file.

```bash
# Add your generated SQL
cp packages/schema/out/editorial-permissions.sql supabase/migrations/[timestamp]_editorial_permissions_setup.sql
```

NB: Replace `[timestamp]` with the actual timestamp of your migration file. This is printed after you run the `migration new` command.

## Step 4: Deploy to Database

```bash
# Deploy to your Supabase project
supabase db push

# Verify deployment
supabase migration list
```

---

# Best Practices for Custom Seeds 📋

## 1. Planning Your Permission Structure

**Start with User Stories:**
```markdown
As a [role], I need to [action] so that [business value]

Examples:
- As a Staff Writer, I need to create article drafts so that I can contribute content
- As a Senior Editor, I need to publish articles so that content goes live
- As a Freelancer, I need to edit my drafts so that I can refine my work
```

**Map Business Processes:**
- Who creates content?
- Who approves it?
- Who can publish it?
- What are the approval workflows?

## 2. Naming Conventions

**Consistent Naming Patterns:**
```typescript
// Role naming: [Level]_[Function]
'senior_editor', 'staff_writer', 'content_manager'

// Permission naming: [Action]_[Resource]
'create_articles', 'publish_content', 'manage_users'

// Group naming: [Domain]_[Scope]
'editorial_management', 'content_creation', 'user_administration'
```

## 3. Security Considerations

**Principle of Least Privilege:**
```typescript
// ❌ Avoid: Too broad permissions
const badPermission = Permission.createDataPermission({
  name: 'Manage Everything',
  action: '*',
  table_name: '*' // Too permissive!
});

// ✅ Good: Specific, scoped permissions
const goodPermission = Permission.createDataPermission({
  name: 'Edit Own Articles',
  action: 'update',
  table_name: 'articles',
  conditions: { author_id: '$CURRENT_USER_ID' }
});
```

**Always Include Time Bounds for Elevated Access:**
```typescript
// Temporary elevated permissions should always expire
const tempAdminAccess = {
  valid_until: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
  metadata: { reason: 'Emergency system maintenance' }
};
```

# Next Steps: Advanced Customization 🎯

## Dynamic Permission Assignment

Create permissions that adapt to business logic:

```typescript
// Example: Seasonal permissions for holiday content
const createHolidayContent = Permission.createDataPermission({
  app,
  id: 'create_holiday_content',
  name: 'Create Holiday Content',
  description: 'Can create holiday-themed articles during Q4',
  scope: 'table',
  schema_name: 'public',
  table_name: 'articles',
  action: 'insert',
  conditions: {
    category: 'holiday',
    // Only during Q4
    created_at: {
      $gte: '2025-10-01',
      $lte: '2025-12-31'
    }
  }
});
```

Remember: A well-designed permission seed is the foundation of a secure, scalable application. Take time to model your organization's actual workflows and access patterns before implementing!

-----------------
FILE PATH: kits/supamode/configuration.mdoc

---
status: "published"
title: "Configuration"
label: "Configuration"
order: 1
description: "Configure Supamode to best fir your application business logic"
collapsed: false
collapsible: false
---



-----------------
FILE PATH: kits/supamode/deployment.mdoc

---
status: "published"
title: "Deployment"
label: "Deploy to Production"
order: 3
description: "Understanding how to deploy Supamode to production"
collapsed: false
collapsible: false
---



-----------------
FILE PATH: kits/supamode/development.mdoc

---
status: "published"
title: "Development"
label: "Develop and Extend"
order: 4
description: "Understanding how to extend Supamode"
collapsed: false
collapsible: false
---



-----------------
FILE PATH: kits/supamode/features.mdoc

---
status: "published"
title: "Features"
label: "Features"
order: 2
description: "Understanding the features of Supamode"
collapsed: false
collapsible: false
---



-----------------
FILE PATH: kits/supamode/installation/clone-repository.mdoc

---
status: "published"
title: "Clone the Supamode Repository"
label: "Clone the Repository"
order: 3
description: "Clone the Supamode repository to your local machine."
---

## 0. Prerequisites

- Node.js 18.x or later
- Docker
- Pnpm
- Supabase account (optional for local development)

## Verify your Git username

To verify you have access, you need to check that your local Git username is the same as set up in the Makerkit's Github organization.

Please run the following command to check your Git username:

```bash
git config user.username
```

If the output is not your Github username, or does not match the username registered in Makerkit's Github organization, you can set it using the following command:

```bash
git config --global user.username "Your Github Username"
```

Ex. if your Github username is `johndoe`, you can set it using the following command:

```bash
git config --global user.username "johndoe"
```

This is important to ensure you can run the repository.

## Cloning the Repository

Clone this repository with the command:

```bash
git clone git@github.com:makerkit/supamode
```

NB: If your SSH key isn't set - then use the HTTPS.

```bash
git clone https://github.com/makerkit/supamode
```

NB: please switch to HTTPS for ALL commands if you are not using SSH, not just the clone command.

**Windows Users**: place the repository near the root of your drive

Some Windows users hit errors loading certain modules due to very long paths. To avoid this, try placing the repository near the root of your drive.

Avoid using OneDrive, as it can cause issues with Node.js.

Now, remove the original `origin`:

```bash
git remote rm origin
```

Add upstream pointing to this repository so you can pull updates

```bash
git remote add upstream git@github.com:makerkit/remix-supabase-saas-kit-turbo
```

Once you have your own repository, do the same but use `origin` instead of `upstream`

To pull updates (please do this daily with your morning coffee):

```bash
git pull upstream main
```

This will keep your repository up to date.

### 0.1. Install Pnpm

```bash
# Install pnpm
npm i -g pnpm
```

## 1. Setup dependencies

```bash
# Install dependencies
pnpm i
```

## 2. Post-merge Hooks

It's very useful to run the following command after pulling updates from the upstream repository:

```bash
pnpm i
```

This ensures that any new dependencies are installed and the project is up to date. We can run this command automatically after pulling updates by setting up a post-merge hook.

Create a new file named `post-merge` in the `.git/hooks` directory:

```bash
touch .git/hooks/post-merge
```

Add the following content to the `post-merge` file:

```bash
#!/bin/bash

pnpm i
```

Make the `post-merge` file executable:

```bash
chmod +x .git/hooks/post-merge
```

Now, every time you pull updates from the upstream repository, the `pnpm i` command will run automatically to ensure your project is up to date. This ensures you're always working with the latest changes and dependencies and avoid errors that may arise from outdated dependencies.


-----------------
FILE PATH: kits/supamode/installation/common-commands.mdoc

---
status: "published"
title: 'Common commands you need to know for Supamode'
label: 'Common Commands'
order: 5
description: 'Learn about the common commands you need to know to work with Supamode'
---

Here are some common commands you need to know to work with the Supamode.

NB: You don't need these commands to kickstart your project - you just need to know they exist so you can use them when you need them.

## Installing dependencies

To install the dependencies, run the following command:

```bash
pnpm i
```

This command will install all the dependencies for the project.

## Starting the development server

```bash
pnpm run dev
```

## Running the Supabase CLI commands

Supabase is installed in the `apps/app` folder. To run commands with the Supabase CLI, you can use the command:

```
pnpm --filter app supabase <command>
```

For example, if the documentation in Supabase recommends a command such as:

```
supabase db link
```

You will use:

```
pnpm --filter app supabase db link
```

This command will start the development server for the web application.

## Starting Supabase

To start Supabase, run the following command:

```bash
pnpm run supabase:web:start
```

This command will start the Supabase web server.

## Resetting Supabase

Resetting the Database is needed when you update the schema or need to start fresh.

If you need to reset the Supabase database, run the following command:

```bash
pnpm run supabase:web:reset
```

This will reset the Supabase database.

## Generate Supabase types

Generating the types is required when you update the Supabase schema. In this way, the client can have the latest types.

To generate the Supabase types, run the following command:

```bash
pnpm run supabase:web:typegen
```

This will generate the Supabase types for the project. This should be done every time you update the Supabase schema.

## Running tests

To run the tests, run the following command:

```bash
pnpm run test
```

This will run the tests for the project.

## Cleaning the project

To clean the project, run the following command:

```bash
pnpm run clean:workspaces
pnpm run clean
```

Then - reinstall the dependencies:

```bash
pnpm i
```

## Type-checking the project

To type-check the project, run the following command:

```bash
pnpm run typecheck
```

## Linting the project

To lint the project, run the following command:

```bash
pnpm run lint:fix
```

This will lint the project using ESLint.

## Formatting the project

To format the project, run the following command:

```bash
pnpm run format:fix
```

This will format the project using Prettier.


-----------------
FILE PATH: kits/supamode/installation/introduction.mdoc

---
status: "published"
title: "Introduction to Supamode"
label: "Introduction"
order: 1
description: "Introducing Supamode, a Super Admin for Supabase apps"
---

Supamode is a self-hosted Super Admin for Supabase apps, designed to help you manage your DB data as easily as with a CMS. It empowers non-technical users to navigate the DB without needing to know any SQL jargon or database structure.

Supamode is built with the following key features:
- **User-friendly interface**: A clean and intuitive UI that makes it easy to manage your data.
- **Role-based access control**: Fine-grained permissions to ensure that users can only access the data they are allowed to see.
- **Customizable views**: Create custom views for your data, making it easier to work with complex datasets.
- **Data Explorer**: A powerful tool to explore and visualize your data, making it easier to understand and analyze.
- **Users Explorer**: Manage users authenticated with your app.
- **CMS-like interface**: Create and manage your data using a familiar interface.
- **Extensible**: Add your own custom views and features by modifying the code. The API provided by Supamode allows you to extend the functionality of the app to suit your needs.

-----------------
FILE PATH: kits/supamode/installation/running-project.mdoc

---
status: "published"
title: "Running Supamode"
label: "Running Supamode"
order: 4
description: "Learn how to run the Supamode project on your local machine."
---

To run the project - there are some commands you need to run.

1. Start the development server (web application + API)
2. Start Supabase (ensure Docker is running)
3. Start Stripe (optional, if you want to test the billing system)

## 1. Start the development server

```bash
# Start the development server
pnpm dev
```

This command will run the web application.

To get started right away, use the credentials below:

- **Email**: `root@supamode.com`
- **Password**: `testingpassword`

This command will start two servers:

1. **React SPA**: At localhost:5173, the Vite Server running the React Router web application.
2. **Hono API**: At localhost:3000, the Hono API server.

Vite is configured to proxy all `/api` requests to the Hono API server, so you can make API calls from the React application without any additional configuration.

## 2. Start Supabase

Run the following command to start Supabase (or use your IDE to run the command in the package.json)

```bash
pnpm run supabase:web:start
```

This command will start the Supabase web server. Please make sure no other Supabase instance is running on your machine, as it will conflict with the one started by this command.

-----------------
FILE PATH: kits/supamode/installation/technical-details.mdoc

---
status: "published"
title: "Technical Details of Supamode"
label: "Technical Details"
order: 2
description: "A detailed look at the technical details of Supamode"
---

The Supamode app is built as a [Turborepo](https://turbo.build) monorepo, built on top of modern, battle-tested and popular technologies.

The application is an SPA using React, backed by a Hono API.

1. **SPA**: React Router 7 as client-side router
2. **API**: Hono as Backend API
3. **UI**: Shadcn UI for the UI components and Tailwind CSS for styling
4. **ORM**: Drizzle for interacting with Supabase
5. **Libraries**: React Query, Zod, Lucide React, Lexical, CodeMirror and other popular libraries


-----------------
FILE PATH: kits/supamode/installation/updating-codebase.mdoc

---
status: "published"
title: "Updating Supamode"
label: "Updating the Codebase"
order: 6
description: "Learn how to update your Supamode app to the latest version."
---

To update the codebase, you will pull the latest changes from the GitHub repository and merge them into your project. This will ensure that you have the latest features and bug fixes.

If you have followed the instructions in the previous sections, you should have a Git repository set up for your project, with an `upstream` remote pointing to the original repository.

To update your project, you will first fetch the latest changes from the `upstream` remote, and then merge them into your project.

Here are the steps to update your project.

### Update the `upstream` remote

First, you need to fetch the latest changes from the `upstream` remote. To do this, run the following command:

```bash
git pull upstream main
```

Please run `pnpm i` in case there are any changes in the dependencies.

### Resolve any conflicts

If there are any conflicts during the merge, you will need to resolve them manually. Git will show you the files with conflicts, and you can edit them to resolve the conflicts.

#### Conflicts in the lock file "pnpm-lock.yaml"

If you have conflicts in the `pnpm-lock.yaml` file, accept any of the two changes (don't do it manually), then run:

```
pnpm i
```

The lock file will now reflect both your changes and the changes from the `upstream` repository.

#### Conflicts in the DB types "database.types.ts"

Since your types will differ from the ones in the `upstream` repository, you need to rebuild them.

To do so, reset the DB:

```bash
npm run supabase:web:reset
```

Then, run the following command to regenerate the types:

```bash
npm run supabase:web:typegen
```

Now the types will reflect the changes from the `upstream` repository and your project.



-----------------
FILE PATH: kits/supamode/installation.mdoc

---
status: "published"
title: "Installation"
label: "Installation"
order: 0
description: "Get started with Supamode. Install the application locally and start making it your own!"
collapsed: false
collapsible: false
---



