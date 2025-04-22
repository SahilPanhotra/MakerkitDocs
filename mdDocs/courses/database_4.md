-----------------
FILE PATH: courses\nextjs-turbo\database.mdoc

---
title: "Setting up your Supabase Database - Schema and Migrations"
description: "Learn how to set up your Supabase database, run migrations, seed data, and more."
label: "Database Schema and Migrations"
order: 2
---

Hi there! 👋 Welcome to this module on setting up your database for your Makerkit Next.js Supabase Turbo project. In this module, **we will explore how you can set up your database, run migrations, seed data, and more**.

Your Database schema is the blueprint of your database. It defines the structure of your database, including tables, columns, and relationships. Migrations are scripts that help you manage changes to your database schema over time.

Understanding the DB is crucial for building an application - and this module will help you understand how to set up your database for your Makerkit Next.js Supabase Turbo project.

For our project, we want to create a schema that helps our customers manage support tickets sent to their support team - and by the end of this module, we will have our initial* database schema and hopefully a deeper understanding of how to work with the database in Makerkit.

*We will be adding more tables and relationships as we progress through the course.

## Supabase Postgres Database

Supabase uses PostgreSQL as the underlying database for its services. PostgreSQL is a powerful, open-source relational database management system that is known for its reliability, scalability, and extensibility. It is widely used in production environments for its robust features and performance.

If you're new to Postgres or Supabase, [please read the official documentation](https://supabase.com/docs/guides/database/overview) to get a better understanding of how Supabase uses Postgres as the database for its services.

This course will need you to have a basic understanding of SQL and Postgres - so if you're new to these technologies, I recommend you take some time to learn the basics before proceeding with this course.

## Supabase Database - Local and Remote

When you run Supabase locally, it will run the migrations and seed the DB with the initial data. However, when you deploy your app, you need to run the migrations and seed the DB on the server to seed the remote instance with the initial data.

### Why use the local DB for development?

I'm often asked why we need to use the local DB for development when we have a remote DB. It's a valid question, and I'll answer it here.

Why should you use the local Supabase Database for development?

1. **Speed**: The local DB is faster than the remote DB, which helps you iterate faster.
2. **Cost**: Running the DB locally is free, while the remote DB may incur costs.
3. **Isolation**: The local DB is isolated from the remote DB, which helps you test changes without affecting the production data.
4. **Easy to reset**: You can easily reset the local DB to the initial state by running the migrations and seeding the DB.

While you're in a development phase, you'll need to iterate over and over again with your DB schema. **You won't get it right the first time, you won't get it right the second time**. You iterate and iterate as you develop your application.

This is why it's important to have a local DB that you can easily reset to the initial state to test your changes.

## An Introduction to the Makerkit Database Schema

The crucial entity of the Makerkit schema is the `public.accounts` table - which holds information about the accounts in our system.

An account can **either be related to a user account or to a team account** - and it's **the central entity that ties everything together in our system**, since everything will be related to and owned by an account.

Here's a brief overview of the tables in our database:

1. `public.accounts` - Holds information about the accounts in our system.
2. `public.accounts_memberships` - Holds information about the memberships of accounts.
3. `public.roles` - Holds information about the roles in our system.
4. `public.role_permissions` - Holds information about the permissions of roles.
5. `public.invitations` - Holds information about the invitations sent to users to join a team.
6. `public.billing_customers` - Holds information about the billing customers in our system and are linked to the relative customer ID in the billing provider.
7. `public.subscriptions` - Holds information about recurring subscriptions in our system.
8. `public.subscription_items` - Holds information about the items in a subscription. This is useful for tracking the items in a subscription and their type (per-seat, metered, or flat)
9. `public.orders` - Holds information about one-off orders in our system. This table is used when offering one-off purchases to customers.
10. `public.order_items` - Holds information about the items in an order.
11. `public.notifications` - Holds information about the notifications sent to users in our system.

### Supabase Auth Tables

In addition to these, you need to take into account the `auth.users` table that is created by Supabase Auth.

This table holds information about the users in our system and is used for authentication purposes. **You won't generally interact with this table directly**, but it's important to know that it exists and that you can use the API to interact with it.

In the next sections, we will explore how to set up your database, run migrations, seed data, and more.

### The flow of working with the database in Makerkit

Generally speaking, I follow these steps when working with the database in Makerkit:

1. **Create a new migration**: I create a new migration file that contains the schema changes I want to make to the database.
2. **Run the migrations**: I run the migrations to apply the changes to the database. This will reset the DB to the initial state and apply the changes in the migration files.
3. **Typegen**: I run the typegen script to generate the types for the database schema. This will generate TypeScript types for the tables and columns in the database.

With the schema added to the DB - and the Supabase client up to date with the generated types - we have everything we need to start building our application.

{% img src="/assets/courses/next-turbo/supabase-database-flow.webp" width="258" height="573" /%}

## Setting up your Database

To set up your database, you need to run the migrations that create the tables and relationships in your database.

Migrations are scripts that help you manage changes to your database schema over time. They are version-controlled and can be run in sequence to bring your database to the desired state.

To add a schema for your new project, we create a new migration with your desired schema. To create a new migration, run the following command:

```bash
pnpm --filter web supabase migration new support-schema
```

This command will create a new migration file in the `apps/web/supabase/migrations` folder. We can now open the migration file and add the schema for our project.

**This is where you arm yourself with pen and paper, and start designing your database schema**. You need to think about the tables you need, the columns in each table, and the relationships between the tables. This is the blueprint of your database - and it's crucial to get it right.

### Thinking about the schema

When designing your schema, you need to think about the entities in your system and how they relate to each other.

For our support system, we need to think about the following entities:

1. **Tickets**: The ticket submitted by users to the support team. The "owner" of the ticket is the account that received the ticket, which means they can read it, update it, and delete it.
2. **Messages**: The messages added to the ticket by the support team. The "owner" of the message is the account that owns the ticket itself, which means they can read it, update it, and delete it.

#### Homework - what's your schema?

Before we move on, I want you to take a moment to think about the schema for your project. What are the entities in your system? How do they relate to each other? What are the columns in each table? Take a moment to sketch out the schema on paper - and we'll use this as a reference when we create the migration.

Bonus points if:

- **Constraints**: You have added the required constraints to the schema.
- **RLS**: You have added the required RLS policies to the schema.

Super bonus points 🚀:
- **Indexes**: You have added the required indexes to the schema.

Super duper bonus points 🚀🚀:
- **Tests**: You have added the required pgTap tests to the schema.

This should take approximately 30 minutes. If you don't have a clear idea of the schema yet, that's okay - let's proceed with my own schema, and you can come back to this later to improve it, edit it, or add more to it.

Done? Great! Let's move on to adding the schema to the migration.

## Diagramming the schema

Below is a diagram of the schema for our support system. This diagram shows the tables in our database and the relationships between them.

```
+---------------+       +---------------+
|    tickets    |       |   messages    |
+---------------+       +---------------+
| id            |1     *| id            |
| account_id    |<------| ticket_id     |
| title         |       | author        |
| category      |       | author_account_id
| assigned_to   |       | content       |
| priority      |       | attachment_url|
| status        |       | created_at    |
| customer_email|       | updated_at    |
| resolution    |       +---------------+
| created_at    |
| updated_at    |
+---------------+
       ^
       |
       |
       |
+---------------+
|   accounts    |
+---------------+
| id            |
| name          |
| ...           |
+---------------+

+------------------+
| storage.buckets  |
+------------------+
| id: 'attachments'|
| name:'attachments'
| public: false    |
+------------------+
```

In the diagram above, we have the `tickets` table that holds information about the tickets submitted by users to the support team. The `messages` table holds information about the messages added to the ticket by the support team. The `accounts` table holds information about the accounts in our system.

1. The `tickets` table is related to the `accounts` table through the `account_id` column.
2. The `messages` table is related to the `tickets` table through the `ticket_id` column.
3. The `messages` table is also related to the `accounts` table through the `author_account_id` column.

This is a simple schema that captures the essence of our support system. You can expand on this schema by adding more tables and relationships as needed.

{% alert type="warning" title="Make sure your own tables have a different name!" %}
Some of the Makerkit plugins, or some of your tables, may have the same name as the ones below. Make sure to change the name of your own tables to avoid conflicts.
{% /alert %}

## Adding the schema to the migration

Now that you have a clear idea of the schema you want to create, you can add it to the migration file.

### Adding the `tickets` table

Let's begin adding the first table, `tickets`, to the migration file. Here's how you can add the `tickets` table to the migration file:

```sql
-- public.ticket_status: enum type for ticket status
create type public.ticket_status as enum ('open', 'closed', 'resolved', 'in_progress');

-- public.ticket_priority: enum type for ticket priority
create type public.ticket_priority as enum ('low', 'medium', 'high');

/*
* Table: public.tickets
-- table for the support tickets
*/
create table if not exists public.tickets (
  id uuid primary key default gen_random_uuid(),
  account_id uuid not null references public.accounts(id) on delete cascade,
  title varchar(255) not null,
  category varchar(100) not null default 'general',
  assigned_to uuid references public.accounts(id) on delete set null,
  priority public.ticket_priority not null default 'medium',
  status public.ticket_status not null default 'open',
  customer_email varchar(255),
  resolution text,
  resolved_at timestamptz,
  resolved_by uuid references public.accounts(id) on delete set null,
  closed_at timestamptz,
  closed_by uuid references public.accounts(id) on delete set null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

-- Indexes
create index ix_tickets_account_id on public.tickets(account_id);
```

This migration creates the `tickets` table with the required columns. Here's a brief overview of the columns in the `tickets` table:

1. `id`: The unique identifier of the ticket.
2. `account_id`: The account that owns the ticket. It is linked to the `public.accounts` table using a foreign key constraint. It will cascade the deletion of the account if the account is deleted.
3. `status`: The status of the ticket. We use an enum type to store the status of the ticket. The status can be one of `open`, `closed`, `resolved`, or `in_progress`.
4. `priority`: The priority of the ticket. We use an enum type to store the priority of the ticket. The priority can be one of `low`, `medium`, or `high`.
5. `title`: The title of the ticket. We use `varchar(255)` to store the title of the ticket, so it can be up to 255 characters long. Feel free to adjust the length based on your requirements.
6. `category`: The category of the ticket. We use `varchar(100)` to store the category of the ticket, so it can be up to 100 characters long. Feel free to adjust the length based on your requirements.
7. `assigned_to`: The account that the ticket is assigned to. It is linked to the `public.accounts` table using a foreign key constraint. It will set the assigned account to `null` if the account is deleted.
8. `customer_email`: The email of the customer who submitted the ticket. We use `varchar(255)` to store the email of the customer, so it can be up to 255 characters long. Feel free to adjust the length based on your requirements.
9. `resolution`: The resolution of the ticket. We use `text` to store the resolution of the ticket, so it can be of any length. Feel free to adjust the length based on your requirements.
10. `created_at`: The timestamp when the ticket was created. We use `timestamptz` to store the timestamp with timezone information.
11. `updated_at`: The timestamp when the ticket was last updated. We use `timestamptz` to store the timestamp with timezone information.

### Securing the `tickets` table with RLS

**One very important thing to understand when using Supabase is that by default, every user can read and write to every table in the database**.

This is not what we want in our application - **we want to restrict access to the tables based on the account that owns the data**.

To secure the `tickets` table, we need to add a Row-Level Security (RLS) policy to the table. This policy will restrict access to the table based on the account that owns the ticket.

In addition - we want to generally revoke all the permissions, and individually grant the permissions to the roles that need them.

#### Revoking all permissions

To revoke all permissions from the `public.tickets` table, we can use the following command:

```sql
revoke all on public.tickets from public, service_role;
```

This command revokes all permissions from the `public.tickets` table for the `public` and `service_role` roles. This means that no one can read, write, or modify the `public.tickets` table until we grant the permissions to the roles that need them.

We can now grant granular permissions to the roles that need them, `authenticated` and `admin`.

#### Why revoke permissions from `anon` and `authenticated`?

Generally speaking, I am not a fan of letting anonymous users read or write to the DB directly.

Instead of opening up a table to anonymous users - you can lock down the table with RLS, and control the requests through an API Route Handler, which ensures you have more control over the data that is being read or written to the DB by accounts that are not authenticated.

#### Granting permissions to the `authenticated` role

We want `authenticated` users to be able to read, update and delete the tickets they own. To grant these permissions to the `authenticated` role, we can use the following command:

```sql
grant select, insert, update, delete on public.tickets to authenticated;
```

Additionally, we want the service role to be able to insert tickets on behalf of anonymous accounts (i.e. accounts that are not part of the system) - so we grant the insert permission to the `service_role`:

```sql
grant select, insert on public.tickets to service_role;
```

Generally speaking, non-authenticated users have no access to the `tickets` table, and authenticated users can only read, update, and delete the tickets they own.

The `Service Role` role can insert tickets on behalf of non-authenticated users behind our Server Side API - which allows to have a layer of control over the data that is being inserted into the DB. You can then control access to the API layer using a captcha, rate limiting, or other mechanisms to prevent abuse.

#### Row Level Security on the `tickets` table

To secure the `tickets` table with Row-Level Security (RLS), we can use the following command:

```sql
alter table public.tickets enable row level security;
```

This command enables Row-Level Security (RLS) on the `public.tickets` table. This means that the RLS policy we define will be applied to the table, and only the rows that match the policy will be returned when querying the table.

By default, RLS will deny access to all rows in the table until we define the policy.

Important concepts to understand about RLS:
1. **Selection** won't return errors, but will filter out rows that the user doesn't have access to. This means that if a user doesn't have access to a row, it will be as if the row doesn't exist for that user.
2. **Insertion** will return an error if the user doesn't have access to the row they are trying to insert.
3. **Update** and **Deletion** will return an error if the user doesn't have access to the row they are trying to update or delete. However, an update will not return an error if the user doesn't have access to the row they are trying to update, but the row will not be updated. That's similar to how the `select` operation works.

##### Introducing RLS Policies

Row Level Security is a powerful feature in PostgreSQL that allows you to control access to rows in a table based on the user executing a query. It's like having a personal bouncer for each row in your database, checking if a user is allowed to see or modify that specific row.

```sql
create policy select_tickets  <-- name of the policy
  on public.tickets           <-- on the public.tickets table, we want to apply the policy
  for select                  <-- for the select operation
  to authenticated            <-- to only the "authenticated" role
  using (
    public.has_role_on_account(account_id) <-- using the has_role_on_account function
  );
```

Let's break this down:

1. `create policy select_tickets`: This names your policy. Think of it as a label for this security rule.
2. `on public.tickets`: This specifies which table the policy applies to. In this case, it's the 'tickets' table in the 'public' schema.
3. `for select`: This indicates the policy applies to SELECT operations (reading data). You can have different policies for INSERT, UPDATE, and DELETE.
4. `to authenticated`: This specifies which database roles this policy applies to. Here, it's for any authenticated user.
using `(public.has_role_on_account(account_id))`: This is the actual security check. It calls a function `has_role_on_account` to determine if the current user has permission to see this ticket.

In plain English, this policy says: "For authenticated users, when they try to read from the tickets table, only show them tickets where they have a role on the associated account."

##### Permissive vs Restrictive Policies

PostgreSQL supports two types of policies: permissive and restrictive.

###### Permissive Policies (default)

These are like bouncers who say "You can come in if you meet any of these criteria."

1. If multiple permissive policies exist, they are combined with OR logic.
2. The policy we showed above is a permissive policy.

###### Restrictive Policies:

These are like extra-strict bouncers who say "You can only come in if you meet all of these criteria."

1. If restrictive policies exist, they are combined with AND logic.
2. To create a restrictive policy, you'd add the USING clause:

```sql
create policy select_tickets
  on public.tickets
  as restrictive
  for select
  to authenticated
  using (
    public.has_role_on_account(account_id)
  );
```

In practice:

1. If you have only permissive policies, a row is visible if it passes any policy.
2. If you have both types, a row is visible if it passes any permissive policy AND all restrictive policies.

##### Why RLS is Awesome

1. **Security at the Data Layer**: It moves security from your application code into the database, making it harder to accidentally expose data.
2. **Simplifies Application Code**: You don't need to add defensive code to check if a user has permission to see a row in application code - since it's enforced at the database level.
3. **Granular Control**: You can create very specific rules about who can see or modify which rows.

For a more complete documentation on RLS, you can check the [Supabase documentation](https://supabase.com/docs/guides/database/postgres/row-level-security).

##### Defining the RLS policy to read the tickets

Our goal is to let team accounts that owns the ticket to read the tickets. To achieve this, we can define the following RLS policy:

```sql
create policy select_tickets  <-- name of the policy
  on public.tickets           <-- on the public.tickets table, we want to apply the policy
  for select                  <-- for the select operation
  to authenticated            <-- to only the "authenticated" role
  using (
    public.has_role_on_account(account_id) <-- using the has_role_on_account function
  );
```

This policy allows the `authenticated` role to read the tickets they own. The `has_role_on_account` function is a custom function that checks if the authenticated user has a role on the account that owns the ticket using the `account_id` property on the ticket record.

If the user has a role on the account, they can read the ticket. If they don't have a role on the account, they won't be able to read the ticket.

**With the policy in place, only the authenticated users who have a role on the account that owns the ticket will be able to read the ticket**. This ensures that only the users who have the necessary permissions can read the tickets.

However, teams should also be able to update or delete the tickets they own. To achieve this, we can define the following RLS policy:

```sql
create policy delete_tickets
  on public.tickets
  for delete
  to authenticated
  using (
    public.has_role_on_account(account_id)
  );

create policy update_tickets
  on public.tickets
  for update
  to authenticated
  using (
    public.has_role_on_account(account_id)
  )
  with check (
    public.has_role_on_account(account_id)
  );
```

This policy allows the `authenticated` role to update and delete the tickets they own. The `has_role_on_account` function is used to check if the authenticated user has a role on the account that owns the ticket. If the user has a role on the account, they can update or delete the ticket. If they don't have a role on the account, they won't be able to update or delete the ticket.

##### Using vs With Check - understanding the difference in RLS

When you update a record, the `using` clause is used to check if the user has the required permissions to update the record. The `with check` clause is used to check if the user has the required permissions to update the record after the update operation is completed. This ensures that the user has the required permissions to update the record before and after the update operation. So you need to add both.

##### The "has_role_on_account" function

The `has_role_on_account` function is a custom function that checks if the authenticated user has a role on the account that owns the ticket. This function is used in the RLS policies to determine if the user has the required permissions to read, update, or delete the ticket.

You can use this in most functions that rely on a user being part of an account.

##### Using the account_id property against the authenticated user

However, if an entity is owned by a user (not a team), you will use the user ID against the Account ID property to check if the user has the required permissions to read, update, or delete the entity.

**It is not the case here (!)** but it's good to know if you were to implement a system where users can own entities directly.

```sql
create policy select_tickets
  on public.tickets
  for select
  to authenticated
  using (
    (select auth.uid()) == account_id
  );
```

In the above example, we are checking if the authenticated user ID is equal to the account ID. If they are equal, the user can read the ticket. If they are not equal, the user won't be able to read the ticket.

## Scenario: We want to allow the `owner` role to read, update, and delete all tickets

Ha! You thought we were done, didn't you? 😄

We have a requirement change from the Product owner (me!) - we want to allow the `owner` role to read, update, and delete all tickets in the system. This is a common requirement in most systems - the owner should be able to read, update, and delete all the entities in the system. In fact, we don't want just about any member of the team to mess with the tickets - only the owner should be able to do that.

She's the boss, after all! 👩‍

### Granting permissions to the `owner` role

Since we want the `owner` role to be able to read, update, and delete all the tickets in the system, we have to

1. add new permissions to the `public.app_permissions` enum
2. add the permissions to the `public.role_permissions` table for the `owner` role

Let's insert the following new permissions to the `public.app_permissions` enum:

1. `tickets.update` - allows the role that has this permission to update the tickets
2. `tickets.delete` - allows the role that has this permission to delete the tickets

We can insert the new permissions using the following SQL commands (please do it at the top of the migration file):

```sql
-- insert new permissions
alter type public.app_permissions add value 'tickets.update';
alter type public.app_permissions add value 'tickets.delete';
commit;
```

Now that we have added the new permissions to the `public.app_permissions` enum, we can grant the permissions to the `owner` role. We can use the following SQL commands to grant the permissions to the `owner` role:

```sql
-- grant permissions to the owner role
insert into public.role_permissions(
  role,
  permission)
values
  ('owner', 'tickets.update'),
  ('owner', 'tickets.delete');
```

This will grant the `owner` role the permissions to read, update, and delete the tickets in the system. The `owner` role will now be able to read, update, and delete all the tickets in the system.

We leave the `read` permission as is, as we want every member of the team to be able to read the tickets. However, we want only the `owner` role to be able to update and delete the tickets.

### Updating the RLS policies with the new permissions

Back to the drawing board!

We need to update the RLS policy to allow the `owner` role to read, update, and delete all the tickets in the system. This means we will rewrite the `update_tickets` and `delete_tickets` policies to include the `owner` role.

To do so, we leverage the `public.has_permission` function to check if the role has the required permission to update or delete the ticket. The `public.has_permission` function checks if the role has the required permission to perform the operation on the ticket.

```sql
create policy delete_tickets
  on public.tickets
  for delete
  to authenticated
  using (
    public.has_permission((select auth.uid()), account_id, 'tickets.delete'::app_permissions)
  );

create policy update_tickets
  on public.tickets
  for update
  to authenticated
  using (
    public.has_permission((select auth.uid()), account_id, 'tickets.update'::app_permissions)
  )
  with check (
    public.has_permission((select auth.uid()), account_id, 'tickets.update'::app_permissions)
  );
```

This policy allows the `owner` role to update and delete all the tickets in the system. The `has_permission` function is used to check if the role has the required permission to update or delete the ticket. If the role has the required permission, they can update or delete the ticket. If the role doesn't have the required permission, they won't be able to update or delete the ticket.

Awesome! We have now secured the `tickets` table with Row-Level Security (RLS) and granted the required permissions to the `authenticated` and `owner` roles. The `authenticated` role can read, update, and delete the tickets they own, while the `owner` role can read, update, and delete all the tickets in the system.

---

You have done a lot, and you deserve to see the results of your hard work. Go ahead, and run the migrations to apply the changes to the database. In the next section, we will explore how to run the migrations and seed the database with the initial data.

Run the following command to run the migrations:

```bash
pnpm run supabase:web:reset
```

Now, you have the schema for the `tickets` table, and the RLS policies in place to secure the table.

Let's run the typegen script to generate the types for the database schema:

```bash
pnpm run supabase:web:typegen
```

This will generate TypeScript types for the tables and columns in the database. You can now use these types in your application to interact with the database.

We will re-run the migrations and seed the database with the initial data as we add the schema for the `messages` and `attachments` tables in the next sections.

## Adding the `messages` table

Now that we have added the schema for the `tickets` table, we can add the schema for the `messages` table. The `messages` table holds information about the messages added to the tickets by the support team. The messages are linked to the tickets they belong to, and the account that owns the ticket can read, update, and delete the messages.

Here's how you can add the `messages` table to the migration file:

```sql
create type public.message_author as enum ('support', 'customer');

/*
* Table: public.messages
*/
create table if not exists public.messages (
  id uuid primary key default gen_random_uuid(),
  ticket_id uuid not null references public.tickets(id) on delete cascade,
  author public.message_author not null,
  author_account_id uuid references public.accounts(id) on delete set null,
  content varchar(5000) not null,
  attachment_url varchar(500),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index ix_messages_ticket_id on public.messages(ticket_id);
```

This migration creates the `messages` table with the required columns. Here's a brief overview of the columns in the `messages` table:

1. `id`: The unique identifier of the message.
2. `ticket_id`: The ticket that the message belongs to. It is linked to the `public.tickets` table using a foreign key constraint. It will cascade the deletion of the ticket if the ticket is deleted.
3. `author`: The author of the message. We use an enum type to store the author of the message. The author can be one of `support` or `customer`.
4. `author_account_id`: The account that the author belongs to. It is linked to the `public.accounts` table using a foreign key constraint. It will set the author account to `null` if the account is deleted.
5. `content`: The content of the message. We use `varchar(5000)` to store the content of the message, so it can be up to 5000 characters long. Feel free to adjust the length based on your requirements.
6. `attachment_url`: The URL of the attachment added to the message that points to Supabase Storage. We use `varchar(500)` to store the URL of the attachment, so it can be up to 500 characters long. Feel free to adjust the length based on your requirements.
7. `created_at`: The timestamp when the message was created. We use `timestamptz` to store the timestamp with timezone information.
8. `updated_at`: The timestamp when the message was last updated. We use `timestamptz` to store the timestamp with timezone information.

### Securing the `messages` table with RLS

We want to secure the `messages` table with Row-Level Security (RLS) to restrict access to the table based on the account that owns the message. We can use the same approach we used for the `tickets` table to secure the `messages` table.

We can revoke all permissions from the `public.messages` table and grant granular permissions to the roles that need them. We can also define RLS policies to read, update, and delete the messages based on the account that owns the message.

```sql
-- revoke all permissions from the messages table
revoke all on public.messages from public, service_role;
```

This command revokes all permissions from the `public.messages` table for the `public` and `service_role` roles. This means that no one can read, write, or modify the `public.messages` table until we grant the permissions to the roles that need them.

We can now grant granular permissions to the roles that need them, such as `authenticated`:

```sql
-- grant permissions to the authenticated role
grant select, insert, update on public.messages to authenticated;
grant select, insert on public.messages to service_role;
```

Perfect. Now, we also need to secure the `messages` table with Row-Level Security (RLS) to restrict access to the table based on the account that owns the message. We can use the same approach we used for the `tickets` table to secure the `messages` table.

```sql
-- enable row level security on the messages table
alter table public.messages enable row level security;
```

This command enables Row-Level Security (RLS) on the `public.messages` table. This means that the RLS policy we define will be applied to the table, and only the rows that match the policy will be returned when querying the table. By default, RLS will deny access to all rows in the table until we define the policy.

To simplify the process of defining the RLS policies, we can create a function that checks if the authenticated user has a role on the account that owns the message. This function can be used in the RLS policies to determine if the user has the required permissions to read, update, or delete the message.

```sql
create or replace function public.has_role_on_ticket_account(ticket_id uuid)
  returns boolean
  set search_path = ''
  as $$
  begin
    return exists (
      select 1
      from public.tickets ticket
      where ticket.id = ticket_id
      and public.has_role_on_account(ticket.account_id)
    );
  end;
  $$ language plpgsql stable;

grant execute on function public.has_role_on_ticket_account(uuid) to authenticated;
```

This function checks if the authenticated user has a role on the account that owns the ticket. It returns `true` if the user has a role on the account, and `false` if the user doesn't have a role on the account. The function is used in the RLS policies to determine if the user has the required permissions to read, update, or delete the message.

We can now define the RLS policy to read the messages:

```sql
-- define the RLS policy to read the messages
create policy select_messages
  on public.messages
  for select
  to authenticated
  using (
    public.has_role_on_ticket_account(ticket_id)
  );
```

This policy allows the `authenticated` role to read the messages they own. The `has_role_on_account` function is used to check if the authenticated user has a role on the account that owns the message. If the user has a role on the account, they can read the message. If they don't have a role on the account, they won't be able to read the message.

With the policy in place, only the authenticated users who have a role on the account that owns the message will be able to read the message. This ensures that only the users who have the necessary permissions can read the messages.

The last policy we need is to insert messages for the users of a team:

```sql
-- define the RLS policy to insert the messages
create policy insert_messages
  on public.messages
  for insert
  to authenticated
  with check (
    public.has_role_on_ticket_account(ticket_id)
  );
```

We have now secured the `messages` table with Row-Level Security (RLS) and granted the required permissions to the `authenticated` role. The `authenticated` role can read, update, and delete the messages they own.

We have disallowed updating and deleting messages for now, as we want to keep the messages immutable. However, you can add the policies to update and delete messages if you need to.

## Adding the `attachments` storage bucket

The `attachments` storage bucket holds the attachments added to the messages by the support team. The attachments are linked to the messages they belong to, and the account that owns the message can read, update, and delete the attachments.

Here's how you can add the `attachments` storage bucket to the migration file:

```sql
-- Storage bucket for attachments
insert into
  storage.buckets (id, name, PUBLIC)
values
  ('attachments', 'attachments', false);
```

This migration creates the `attachments` storage bucket. The bucket is used to store the attachments added to the messages by the support team. The attachments are linked to the messages they belong to, and the account that owns the message can read, update, and delete the attachments. It is a private bucket, which means it requires authentication to access the attachments.

### Securing the `attachments` storage bucket

We want to secure the `attachments` storage bucket to restrict access to the attachments based on the account that owns the message. To store an attachment, we can use the relative message ID as the path to the attachment in the storage bucket. This allows us to check if the user has the required permissions to read, update, or delete the attachment based on the account that owns the message.

```sql
create or replace function public.can_read_message (message_id uuid)
  returns boolean
  set search_path = ''
  as $$
  begin
    return exists (
      select 1
      from public.messages message
      where message.id = message_id
      and public.has_role_on_ticket_account(message.ticket_id)
    );
  end;
  $$ language plpgsql stable;

grant execute on function public.can_read_message(uuid) to authenticated;

-- RLS policies for storage
create policy message_attachments
  on storage.objects
  for select
  to authenticated using (
    bucket_id = 'attachments'
    and public.can_read_message(
      kit.get_storage_filename_as_uuid (name)
    )
);
```

When it comes to writing RLS policies for the Storage buckets, we need to do things a bit differently.

In fact, we need to check **if the user has the required permissions to read the attachment based on the account that owns the message**.

To achieve this, we can create a custom function `public.can_read_message` that checks if the authenticated user has a role on the account that owns the message. This function is used in the RLS policy to determine if the user has the required permissions to read the attachment.

Let's break down the function and the RLS policy in the context of a Storage bucket:

1. The `can_read_message` function **checks if the authenticated user has a role on the account** that owns the message. It returns `true` if the user has a role on the account, and `false` if the user doesn't have a role on the account. The function is used in the RLS policy to determine if the user has the required permissions to read the attachment.
2. We **extract the message ID from the attachment name** using the `kit.get_storage_filename_as_uuid` function. This function extracts the message ID from the attachment name and returns it as a UUID.
3. We verify that the **user has the required permissions to read the attachment based on the account that owns the message**. If the user has the required permissions, they can read the attachment. If they don't have the required permissions, they won't be able to read the attachment.

With the function and the RLS policy in place, only the authenticated users who have a role on the account that owns the message will be able to read the attachment. This ensures that only the users who have the necessary permissions can read the attachments.

## Re-running the migrations and seeding the database

Congratulations! You have added the schema for the `messages` table and the `attachments` storage bucket. You have secured the `messages` table with Row-Level Security (RLS) and granted the required permissions to the `authenticated` role.

Now, you can run the migrations to apply the changes to the database:

```bash
pnpm run supabase:web:reset
```

This will reset the database to the initial state and apply the changes in the migration files. You can now run the typegen script to generate the types for the database schema:

```bash
pnpm run supabase:web:typegen
```

This will generate TypeScript types for the tables and columns in the database. You can now use these types in your application to interact with the database.

You have now set up your database, run the migrations, and seeded the database with the initial data. You can now start building your application using the database schema you have created. Bravo! 🎉

## The full migration file

Here's the full migration file with the schema for the `tickets`, `messages`, and `attachments` tables:

```sql
-- insert new permissions
alter type public.app_permissions add value 'tickets.update';
alter type public.app_permissions add value 'tickets.delete';
commit;

-- grant permissions to the owner role
insert into public.role_permissions(
  role,
  permission)
values
  ('owner', 'tickets.update'),
  ('owner', 'tickets.delete');

--  public.message_author: enum type for message author
create type public.message_author as enum ('support', 'customer');

-- public.ticket_status: enum type for ticket status
create type public.ticket_status as enum ('open', 'closed', 'resolved', 'in_progress');

-- public.ticket_priority: enum type for ticket priority
create type public.ticket_priority as enum ('low', 'medium', 'high');

/*
* Table: public.tickets
-- table for the support tickets
*/
create table if not exists public.tickets (
  id uuid primary key default gen_random_uuid(),
  account_id uuid not null references public.accounts(id) on delete cascade,
  title varchar(255) not null,
  category varchar(100) not null default 'general',
  assigned_to uuid references public.accounts(id) on delete set null,
  priority public.ticket_priority not null default 'medium',
  status public.ticket_status not null default 'open',
  customer_email varchar(255),
  resolution text,
  resolved_at timestamptz,
  resolved_by uuid references public.accounts(id) on delete set null,
  closed_at timestamptz,
  closed_by uuid references public.accounts(id) on delete set null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

-- revoke permissions on public.tickets
revoke all on public.tickets from public, service_role;

-- grant required permissions on public.tickets
grant select, insert, update, delete on public.tickets to authenticated;
grant select, insert on public.tickets to service_role;

-- Indexes
create index ix_tickets_account_id on public.tickets(account_id);

-- RLS
alter table public.tickets enable row level security;

-- SELECT(public.tickets)
create policy select_tickets
  on public.tickets
  for select
  to authenticated
  using (
    public.has_role_on_account(account_id)
  );

-- DELETE(public.tickets)
create policy delete_tickets
  on public.tickets
  for delete
  to authenticated
  using (
    public.has_permission((select auth.uid()), account_id, 'tickets.delete'::app_permissions)
  );

 -- UPDATE(public.tickets)
create policy update_tickets
  on public.tickets
  for update
  to authenticated
  using (
    public.has_permission((select auth.uid()), account_id, 'tickets.update'::app_permissions)
  )
  with check (
    public.has_permission((select auth.uid()), account_id, 'tickets.update'::app_permissions)
  );

/*
* Table: public.messages
*/
create table if not exists public.messages (
  id uuid primary key default gen_random_uuid(),
  ticket_id uuid not null references public.tickets(id) on delete cascade,
  author public.message_author not null,
  author_account_id uuid references public.accounts(id) on delete set null,
  content varchar(5000) not null,
  attachment_url varchar(500),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

-- Indexes
create index ix_messages_ticket_id on public.messages(ticket_id);

-- revoke all permissions from the messages table
revoke all on public.messages from public, service_role;

-- grant permissions to the authenticated role
grant select, insert, update, delete on public.messages to authenticated;

grant select, insert on public.messages to service_role;

-- RLS
alter table public.messages enable row level security;

-- Function: public.has_role_on_ticket_account
-- Description: Check if the authenticated user has a role on the account of the ticket
create or replace function public.has_role_on_ticket_account(ticket_id uuid)
  returns boolean
  set search_path = ''
  as $$
  begin
    return exists (
      select 1
      from public.tickets ticket
      where ticket.id = ticket_id
      and public.has_role_on_account(ticket.account_id)
    );
  end;
  $$ language plpgsql stable;

grant execute on function public.has_role_on_ticket_account(uuid) to authenticated;

-- SELECT(public.messages)
create policy select_messages
  on public.messages
  for select
  to authenticated
  using (
    public.has_role_on_ticket_account(ticket_id)
  );

-- UPDATE(public.messages)
create policy update_messages
  on public.messages
  for update
  to authenticated
  using (
    public.has_role_on_ticket_account(ticket_id)
  )
  with check (
    public.has_role_on_ticket_account(ticket_id)
  );

-- DELETE(public.messages)
create policy delete_messages
  on public.messages
  for delete
  to authenticated
  using (
    public.has_role_on_ticket_account(ticket_id)
  );

-- INSERT(public.messages)
create policy insert_messages
  on public.messages
  for insert
  to authenticated
  with check (
    public.has_role_on_ticket_account(ticket_id)
  );

-- public.plans: table for subscription plans
create table if not exists public.plans (
  variant_id varchar(255) primary key,
  name varchar(255) not null,
  max_tickets int not null
);

-- revoke all permissions from the plans table
revoke all on public.plans from public, service_role;

-- grant permissions to the authenticated role
grant select on public.plans to authenticated, service_role;

-- RLS
alter table public.plans enable row level security;

-- SELECT(public.plans)
create policy select_plans
  on public.plans
  for select
  to authenticated
  using (true);

/*
* Bucket: attachments
*/
insert into
  storage.buckets (id, name, PUBLIC)
values
  ('attachments', 'attachments', false);

-- Function public.can_read_message
-- Description: Check if the authenticated user can read the message
create or replace function public.can_read_message (message_id uuid)
  returns boolean
  set search_path = ''
  as $$
  begin
    return exists (
      select 1
      from public.messages message
      where message.id = message_id
      and public.has_role_on_ticket_account(message.ticket_id)
    );
  end;
  $$ language plpgsql stable;

grant execute on function public.can_read_message(uuid) to authenticated;

-- RLS policies for storage
create policy message_attachments
  on storage.objects
  for select
  to authenticated using (
    bucket_id = 'attachments'
    and public.can_read_message(
      kit.get_storage_filename_as_uuid (name)
    )
);

-- Function to handle status changes
create or replace function kit.handle_ticket_status_change()
returns trigger
set search_path = ''
as $$
begin
  if new.status = 'closed' and old.status != 'closed' then
    new.closed_by := auth.uid();
    new.closed_at := now();
    new.resolved_by := null;
    new.resolved_at := null;
  elsif new.status = 'resolved' and old.status != 'resolved' then
    new.resolved_by := auth.uid();
    new.resolved_at := now();
    new.closed_by := null;
    new.closed_at := null;
  elsif new.status = 'open' and old.status != 'open' then
    new.closed_by := null;
    new.closed_at := null;
    new.resolved_by := null;
    new.resolved_at := null;
  elsif new.status = 'in_progress' and old.status != 'in_progress' then
    new.closed_by := null;
    new.closed_at := null;
    new.resolved_by := null;
    new.resolved_at := null;
  end if;
  return new;
end;
$$ language plpgsql;

-- Trigger to handle status changes
create trigger handle_ticket_status_change_trigger
before update of status on public.tickets
for each row
when (old.status is distinct from new.status)
execute function kit.handle_ticket_status_change();

create
or replace function public.get_subscription_details (target_account_id uuid) returns table (
  variant_id varchar,
  period_starts_at timestamptz,
  period_ends_at timestamptz
)
set search_path = ''
as $$
begin
  -- select the subscription details for the target account
  return query select
        item.variant_id,
        subscription.period_starts_at,
        subscription.period_ends_at
  from
        public.subscription_items as item
  join
        public.subscriptions as subscription
  on
        subscription.id = item.subscription_id
  where
        subscription.account_id = target_account_id
  and   subscription.active = true
  and
        item.type = 'flat';
end;
$$ language plpgsql;

grant execute on function public.get_subscription_details(uuid) to authenticated, service_role;

create
or replace function public.check_ticket_limit () returns trigger
set search_path = ''
as $$
declare
  subscription record;
  ticket_count int;
  max_tickets int;
begin
  -- get the subscription details for the account
  select *
    into subscription
    from public.get_subscription_details(NEW.account_id);

  -- is the user on a free plan?
  if subscription is null then
    select count(*)
      into ticket_count
      from public.tickets
      where account_id = NEW.account_id and
      created_at >= now() - interval '30 days';

    -- check if the user has exceeded the limit
    if ticket_count >= 50 then
      raise exception 'You have reached the maximum number of tickets allowed for your plan';
    end if;

    -- allow the user to create the ticket
    return NEW;
  end if;

  -- get the max tickets allowed for the plan
  select max_tickets
    into max_tickets
    from public.plans
    where variant_id = subscription.variant_id;

  -- Unlimited tickets for the plan, so allow the user to create the ticket
  if max_tickets = -1 then
    return NEW;
  end if;

  -- check the number of tickets created during the billing period
  select count(*)
    into ticket_count
    from public.tickets
    where account_id = NEW.account_id and
    created_at >= subscription.period_starts_at and
    created_at <= subscription.period_ends_at;

  if ticket_count >= max_tickets then
    raise exception 'You have reached the maximum number of tickets allowed for your plan';
  end if;

  return NEW;
end;
$$ language plpgsql;

create or replace trigger check_ticket_limit
before insert on public.tickets
for each row
execute function public.check_ticket_limit ();

create or replace function public.get_remaining_tickets(target_account_id uuid)
returns int
set search_path = ''
as $$
declare
  subscription record;
  ticket_count int;
  max_tickets int;
begin
  select *
    into subscription
    from public.get_subscription_details(target_account_id);

  if subscription is null then
    select count(*)
      into ticket_count
      from public.tickets
      where public.tickets.account_id = target_account_id and
      created_at >= now() - interval '30 days';

    return 50 - ticket_count;
  end if;

  select max_tickets
    into max_tickets
    from public.plans
    where variant_id = subscription.variant_id;

  -- Unlimited tickets
  if max_tickets = -1 then
    return -1;
  end if;

  select count(*)
    into ticket_count
    from public.tickets
    where public.tickets.account_id = target_account_id and
    created_at >= subscription.period_starts_at and
    created_at <= subscription.period_ends_at;

  return max_tickets - ticket_count;
end;
$$ language plpgsql;

grant execute on function public.get_remaining_tickets(uuid) to authenticated, service_role;
```

## Seed the database with initial data

Now that you have set up the database schema and secured the tables with Row-Level Security (RLS), you can seed the database with the initial data.

Seeding the database allows you to populate the tables with sample data that you can use to test your application.

To add seed data into your Supabase database, you can update the `seed.sql` file in the `apps/web/supabase` directory, which already contain some sample data for the `accounts` table.

The seed is re-run every time you run the `supabase:web:reset` command orwhen you start the Supabase Docker container for the first time or when no backups are found.

Here's how you can seed the `tickets` table with sample data:

```sql title="apps/web/supabase/seed.sql"
-- Insert sample tickets
INSERT INTO public.tickets (account_id, status, title, priority, category, resolution, customer_email, updated_at)
VALUES
  ('5deaa894-2094-4da3-b4fd-1fada0809d1c', 'in_progress', 'Cannot access account', 'high', 'Login Issues', NULL, 'john.doe@example.com', NOW() + INTERVAL '1 day'),
  ('5deaa894-2094-4da3-b4fd-1fada0809d1c', 'open', 'Billing discrepancy', 'medium', 'Billing', NULL, 'jane.smith@example.com', NOW() + INTERVAL '2 days'),
  ('5deaa894-2094-4da3-b4fd-1fada0809d1c', 'open', 'Feature request: Dark mode', 'low', 'Feature Request', NULL, 'alex.johnson@example.com', NOW() + INTERVAL '5 days'),
  ('5deaa894-2094-4da3-b4fd-1fada0809d1c', 'resolved', 'App crashes on startup', 'medium', 'Bug', 'Updated app to latest version', 'sarah.lee@example.com', NOW() - INTERVAL '1 hour');

-- Now, let's add some messages for these tickets
WITH ticket_ids AS (
  SELECT id, title FROM public.tickets WHERE title IN (
    'Cannot access account',
    'Billing discrepancy',
    'Feature request: Dark mode',
    'App crashes on startup'
  )
)
INSERT INTO public.messages (ticket_id, author, content, attachment_url)
SELECT
  t.id,
  m.author,
  m.content,
  m.attachment_url
FROM ticket_ids t
JOIN (
  VALUES
    ('Cannot access account', 'customer'::public.message_author, 'I cannot log into my account. It says my password is incorrect, but I''m sure it''s right.', NULL),
    ('Cannot access account', 'support'::public.message_author, 'I''m sorry to hear that. Let''s try resetting your password. I''ve sent a password reset link to your email.', NULL),
    ('Billing discrepancy', 'customer'::public.message_author, 'My last invoice seems to be higher than usual. Can you please check?', NULL),
    ('Billing discrepancy', 'support'::public.message_author, 'Certainly, I''ve looked into your account and noticed that you upgraded your plan last month. This explains the increase. Please confirm if you remember making this change.', NULL),
    ('Feature request: Dark mode', 'customer'::public.message_author, 'It would be great if you could add a dark mode to the app. It''s hard on the eyes at night.', NULL),
    ('App crashes on startup', 'customer'::public.message_author, 'Every time I try to open the app, it crashes immediately. I''m using an iPhone 12.', 'https://example.com/attachments/crash_log.txt'),
    ('App crashes on startup', 'support'::public.message_author, 'Thank you for reporting this and providing the crash log. We''ve identified the issue and it''s been fixed in our latest update. Please update your app and let us know if the problem persists.', NULL),
    ('App crashes on startup', 'customer'::public.message_author, 'I''ve updated the app and it''s working perfectly now. Thank you!', NULL)
) AS m(title, author, content, attachment_url) ON t.title = m.title;
```

## We're not really done yet!

This is a lot of work, and you have done an amazing job so far following all this! 🚀

However, **there are a few more things that we need to add before the DB is complete**, mostly to do with validation and proper updates to ensure Database consistency, and for billing. It doesn't make complete sense to add these now; once we think through the logic a bit more, the requirements will be clearer.

We will get back to the DB and update the schema accordingly once we have a better understanding of the requirements.

For the time being, let's move on to the next section and start building the API that will interact with the database.

## Some good practices to keep in mind with Supabase

Below are some of the best practices to keep in mind when working with Supabase and PostgreSQL:

1. **Use Row-Level Security (RLS) to secure your data** - RLS allows you to control access to your data at the row level, ensuring that only the users who have the required permissions can access the data.
2. **Only grant the necessary permissions to the roles** - this is a good practice to follow to ensure that only the roles that need access to the data have the required permissions.
3. **Use custom functions to enforce business logic** - custom functions can be used to enforce business logic in the database, ensuring that the data is consistent and accurate. They're also required when needing transactions, which is something you will want to ensure data consistency.
4. **Disable anon access by default** - it's a good practice to disable anonymous access to your database by default and only allow authenticated users to access the data. This ensures that only authorized users can access the data.
5. **Use constraints to enforce data integrity** - constraints can be used to enforce data integrity in the database, ensuring that the data is consistent and accurate. You can use constraints like unique constraints, foreign key constraints, and check constraints to enforce data integrity.
6. **Use constraints for size checks** - you can use constraints to enforce size checks on the data, ensuring that the data is within the allowed size limits. Since clients can write directly to the Database, this is a paramount practice. For example, use `varchar` instead of `text` to ensure the data is within a certain size, or use `check` constraints to ensure the data is within a certain range.
7. **Use transactions for complex operations** - transactions can be used to group multiple operations into a single unit of work, ensuring that all the operations are completed successfully or none of them are. This is especially important when you have multiple operations that need to be completed together to ensure data consistency.
8. **Use indexes for performance** - indexes can be used to improve the performance of queries by allowing the database to quickly locate the data. You can use indexes on columns that are frequently used in queries to improve the query performance.
9. **Test with pgTap** - pgTap is a powerful testing framework for PostgreSQL that allows you to write tests for your database schema and ensure that it is working correctly. You can write tests for your database schema using pgTap to ensure that the schema is working as expected.
10. **Use Supabase Studio impersonation** - Supabase Studio allows you to impersonate a user to test the DB as that user. This can be useful for testing the application with different user roles and permissions, such as checking if a user can read a certain row, or if a user can update a certain column.
11. **Use Supabase's Linter** - Supabase has a built-in that will advise you on best practices and potential issues with your SQL queries. This can be very helpful in ensuring that your queries are optimized and secure.

If you think of any other best practices, please let me know!

## Conclusion

In this guide, we have learned how to design a database schema for a customer support ticketing system using Supabase and PostgreSQL. We have covered the following topics:
1. **Understanding the requirements** - we have discussed the requirements of the system and identified the entities and relationships that need to be modeled in the database.
2. **Designing the database schema** - we have designed the database schema for the system, including the tables, columns, relationships, and constraints.
3. **Creating the database schema** - we have created the database schema using SQL and added some sample data to the tables.
4. **Best practices** - we have discussed some best practices to keep in mind when working with Supabase and PostgreSQL.

See you in the next section, where we will start building the API that will interact with the database! 🚀


