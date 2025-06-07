-----------------
FILE PATH: courses/nextjs-turbo/realtime-updates.mdoc

---
title: "Realtime Updates in Makerkit Next.js Supabase Turbo project"
description: "Learn how to stream real-time updates in your Makerkit Next.js Supabase Turbo project."
label: "Streaming Realtime updates"
order: 6
---

👋 Hey, welcome to this module on Realtime Updates in Makerkit Next.js Supabase Turbo project. In this module, **we will explore how you can stream real-time updates** in your Makerkit Next.js Supabase Turbo project.

## Introduction to Realtime Updates with Supabase

In today's modern SaaS, users expect instant, dynamic experiences from web applications. This is where realtime updates come into play, transforming static web pages into living, breathing interfaces that respond to changes as they happen.

Realtime functionality has become a crucial feature for many SaaS applications and thankfully Supabase makes it easy to implement.

Supabase Realtime enables:

1. **Improved User Experience**: Users receive updates instantly without needing to refresh the page, creating a more engaging and responsive application.
2. **Enhanced Collaboration**: Multiple users can work together seamlessly, seeing each other's changes in real-time.
3. **Timely Notifications**: Important updates or alerts can be pushed to users immediately, ensuring they always have the most current information.
4. **Competitive Advantage**: Applications with realtime features often feel more modern and sophisticated, potentially giving them an edge in the market.

## Use Case: Realtime Support Ticket System

In our support ticket system, realtime updates will allow support agents to see new tickets and messages as they arrive, without needing to manually refresh the page. This leads to faster response times and a more efficient support process.

### Supabase Realtime: An Overview

Supabase, our backend platform of choice, provides powerful realtime capabilities out of the box. Supabase Realtime is built on top of Phoenix Channels and can handle millions of simultaneous connections.
Key features of Supabase Realtime include:

1. **Database Changes**: Subscribe to INSERT, UPDATE, and DELETE operations on your Supabase tables.
2. **Presence**: Track which users are online and what they're doing.
3. **Broadcast**: Send and receive messages to and from connected clients.

Supabase Realtime uses WebSockets to maintain a persistent connection between the client and the server, allowing for bidirectional communication. This means that not only can the server push updates to the client, but the client can also send messages back to the server in real-time.

### What We'll Build

In this module, we'll enhance our support ticket system with realtime capabilities.

Specifically, we'll:

1. Implement realtime updates for new tickets and messages.
2. Create a live chat feature for real-time communication between support agents and customers.

By the end of this module, you'll have a solid understanding of how to implement realtime features in your Makerkit project using Supabase Realtime.

## Setting up Supabase Realtime

Before we start implementing realtime features in our Makerkit project, we need to ensure that Supabase is properly configured for realtime functionality. Luckily, Supabase makes this process straightforward.

### Enabling Realtime for the Messages Table

While Supabase allows realtime functionality for all tables by default, it's a good practice to be selective about which tables you want to enable for realtime updates. This helps optimize performance and reduce unnecessary data transfer.

To do so, you have two options:

1. **Enable Realtime in your Schema**: You can enable realtime for all tables in your schema by running the following SQL command:
2. **Enable Realtime in the Dashboard**: Alternatively, you can enable realtime for specific tables in the Supabase Dashboard.

Since we need this to work locally, it makes sense for us to update our schema and enable realtime for the messages table.

```sql
alter publication supabase_realtime add table messages;
```

**Action**: please add the above to our migration file with our schema. If you don't want to reset your Database, you can run the command above directly from Supabase Studio, or you can enable it from the Dashboard itself.

You can now see that we have enabled realtime for the messages table (and on the notifications table, which Makerkit does by default).

### Adding a Realtime subscription with Supabase Realtime

Now that we have enabled realtime for the messages table, we can start listening for changes in our Next.js application.

To do this, we need to **create a new channel in our Supabase client and subscribe to the changes in the messages table**. We can then update our UI whenever a new message is inserted into the table.

Here's how we can set up a realtime subscription in our Next.js application:

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx" %}
useEffect(() => {
  const channel = client.channel(`messages-channel-${ticketId}`);

  const subscription = channel
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        filter: `ticket_id=eq.${ticketId}`,
        table: 'messages',
      },
      (payload) => {
        const message = payload.new as Tables<'messages'>;

        if (message.author === 'customer') {
          appendMessage(message);
        }
      },
    )
    .subscribe();

  return () => {
    void subscription.unsubscribe();
  };
}, [client, ticketId, appendMessage]);
```

Let's explore the code snippet above:

1. We create a new channel using the `client.channel` method, passing in a unique channel name based on the ticket ID.
2. We then subscribe to the `postgres_changes` event on the `messages` table, filtering by the `ticket_id` field to only receive messages related to the current ticket.
3. When a new message is inserted into the table, the `payload` object contains the new message data. We extract this data and append it to the existing messages using the `appendMessage` function.
4. Finally, we return a cleanup function that unsubscribes from the channel when the component is unmounted.

{% img src="/assets/courses/next-turbo/realtime-architecture.webp" width="840" height="612" /%}

#### Why do we use `useEffect` to set up the subscription?

Because we want to ensure that the subscription is created when the component mounts and removed when the component unmounts. This is a common pattern for managing side effects in React components and helps prevent memory leaks and other issues.

If you added the subscription to the render function, it would execute every time the component re-renders, **leading to multiple subscriptions and potential performance issues**.

#### Updating the UI with the new message

Here's the full code snippet for the `useFetchTicketMessages` hook that fetches messages and sets up the realtime subscription:

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx" %}
function useFetchTicketMessages(params: {
  ticketId: string;
  page: number;
  queryKey: string[];
}) {
  const appendMessage = useAppendNewMessage({ queryKey: params.queryKey });
  const client = useSupabase();

  const { ticketId, page } = params;
  const messagesPerPage = 25;

  const queryFn = async () => {
    const startOffset = (page - 1) * messagesPerPage;
    const endOffset = startOffset + messagesPerPage;

    const { data: messages, error } = await client
      .from('messages')
      .select<
        string,
        Message
      >('*, account: author_account_id (email, name, picture_url)')
      .eq('ticket_id', ticketId)
      .order('created_at', { ascending: false })
      .range(startOffset, endOffset);

    if (error) {
      throw error;
    }

    return messages;
  };

  useEffect(() => {
    const channel = client.channel(`messages-channel-${ticketId}`);

    const subscription = channel
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          filter: `ticket_id=eq.${ticketId}`,
          table: 'messages',
        },
        (payload) => {
          const message = payload.new as Tables<'messages'>;

          if (message.author === 'customer') {
            appendMessage(message);
          }
        },
      )
      .subscribe();

    return () => {
      void subscription.unsubscribe();
    };
  }, [client, ticketId, appendMessage]);

  return useInfiniteQuery({
    queryKey: params.queryKey,
    queryFn,
    initialPageParam: page,
    getNextPageParam: (lastPage, _, lastPageParam) => {
      if (lastPage.length === 0) {
        return;
      }

      return lastPageParam + 1;
    },
    getPreviousPageParam: (_, __, firstPageParam) => {
      if (firstPageParam <= 1) {
        return;
      }

      return firstPageParam - 1;
    },
  });
}
```

You can now try to send messages in your application and see them appear in real-time without needing to refresh the page. This is a powerful feature that enhances the user experience and makes your application feel more dynamic and responsive.

However, the customer's side is not yet implemented. We will implement this in the next section.

## Polling updates from the Ticket Widget

In the previous section, we implemented the support agent's side of the realtime updates. Now we will implement the customer's side, where they can see new messages from the support agent in real-time.

We won't be using Supabase Realtime for the customer's side, as it's more complex to set up and maintain and we generally don't want to place the SDK client onto the customer's website.

Instead, **we'll use a polling mechanism** to fetch new messages from the server at regular intervals.

### Implementing Polling in the Ticket Widget

Polling is a simple mechanism where the client sends a request to the server at regular intervals to check for new data. In our case, we'll fetch new messages from the server every few seconds to update the chat widget in real-time.

While polling isn't as efficient as using WebSockets for real-time updates, it's a good alternative for scenarios where setting up WebSockets is not feasible or necessary.

However, if you're planning on working on a more complex implementation, you could explore different ways to implement real-time updates for the customer's side based on your requirements: for example, a WebSocket connected to a  server that streams messages from Supabase Realtime.

For the sake of simplicity, we'll use an interval. We will fetch new messages every 10 seconds based on the last message's timestamp. This way, we can ensure that we only fetch new messages and avoid unnecessary data transfer.

Let's go on and update our `useFetchTicketMessages` hook to include polling in our widget `packages/ticket-widget/src/components/support-ticket-widget-container.tsx`:

```tsx title="packages/ticket-widget/src/components/support-ticket-widget-container.tsx"
function useFetchTicketMessages({
  ticketId,
  isOpen,
}: {
  ticketId: string | undefined;
  isOpen: boolean;
}) {
  const [state, setState] = useState<{
    loading: boolean;
    error: Error | null;
    messages: Message[];
  }>({
    loading: true,
    error: null,
    messages: [],
  });

  useEffect(() => {
    if (!ticketId || !isOpen) {
      return setState((state) => {
        return {
          ...state,
          loading: false,
          error: null,
        };
      });
    }

    function fetchMessages(lastCreatedAt?: string) {
      setState({ loading: true, error: null, messages: [] });

      fetch(`${API_URL}/messages?ticketId=${ticketId}&lastCreatedAt=${lastCreatedAt}`)
        .then((response) => {
          if (!response.ok) {
            throw new Error('Failed to fetch messages');
          }

          return response.json();
        })
        .then((messages) => {
          setState({
            loading: false,
            error: null,
            messages: messages as Message[],
          });
        })
        .catch((error) => {
          setState({
            loading: false,
            error,
            messages: [],
          });
        });
    }

    // Fetch messages on mount
    fetchMessages();

    // Fetch messages every 10 seconds
    const interval = setInterval(() => {
      const lastMessage = state.messages[state.messages.length - 1];
      const lastCreatedAt = lastMessage?.createdAt;

      fetchMessages(lastCreatedAt);
    }, 10_000);

    return () => clearInterval(interval);
  }, [ticketId, isOpen, state.messages]);

  return {
    ...state,
    appendMessage: (message: Message) => {
      setState((state) => ({
        ...state,
        messages: [...state.messages, message],
      }));
    },
  };
}
```

As you may have noticed, we did the following:

1. Added a `fetchMessages` function that fetches messages from the server based on the last message's timestamp.
2. Called `fetchMessages` on mount to fetch the initial set of messages.
3. Set up an interval that fetches new messages every 10 seconds based on the last message's timestamp.
4. Cleared the interval when the component is unmounted to prevent memory leaks.
5. When a new message is detected, we append it to the existing messages using the `appendMessage` function.

There is one missing piece to this: updating the API handler to fetch messages based on the last message's timestamp. Let's update the API handler to include this functionality:

First, we want to update the Zod schema to include the `lastCreatedAt` field:

```ts title="apps/web/app/api/ticket/messages/route.ts"
const GetTicketMessagesSchema = z.object({
  ticketId: z.string().uuid(),
  lastCreatedAt: z.string().or(z.literal('')).transform((value) => {
    if (value === 'undefined') {
      return;
    }

    return value;
  })
});
```

Next, we want to update the handler to fetch messages based on the `lastCreatedAt` field:

```ts title="apps/web/app/api/ticket/messages/route.ts"
const searchParams = new URL(request.url).searchParams;

const { ticketId, lastCreatedAt } = GetTicketMessagesSchema.parse({
  ticketId: searchParams.get('ticketId') ?? '',
  lastCreatedAt: searchParams.get('lastCreatedAt') ?? '',
});

let query = client
  .from('messages')
  .select(
    `
    id,
    ticketId: ticket_id,
    content,
    author,
    createdAt: created_at
    `,
  )
  .eq('ticket_id', ticketId)
  .order('created_at', { ascending: true });

if (lastCreatedAt) {
  query = query.gt('created_at', lastCreatedAt);
}

const { data, error } = await query;
```

We used the `gt` (greater than) filter to fetch messages that were created after the `lastCreatedAt` timestamp. This ensures that we only fetch new messages and avoid duplicates.

---

This implementation ensures that the customer's side of the chat widget is updated in real-time without needing to refresh the page. Customers can see new messages from the support agent as they arrive, creating a more engaging and interactive experience.

## Conclusion

In this module, we explored the importance of realtime updates in modern web applications and how Supabase Realtime can enhance the user experience. We implemented realtime features in our support ticket system, allowing support agents and customers to communicate in real-time without needing to refresh the page.

By setting up Supabase Realtime and implementing polling for the customer's side, we created a dynamic and responsive chat widget that enhances the support experience for both agents and customers.


-----------------
FILE PATH: courses/nextjs-turbo/server-components.mdoc

---
title: "Fetching data with Server Components in Makerkit Next.js Supabase Turbo"
description: "Learn how to fetch data from Server Components in your Makerkit Next.js Supabase Turbo project."
label: "Server Components"
order: 3
---

Hi there! 👋 Welcome to this module.

**We will learn how to set up routing, build components, fetch data, and write data** in your Makerkit Next.js Supabase Turbo project. In short, everything you need to know to build new features in your Makerkit Next.js Supabase Turbo project.

In the previous section, we learned and set up the project Database, and set up the data model. In this section, we will use the data model to fetch data and write data to the database.

In this section, we will be doing the following: **Creating a new page for team accounts where support agents can view new support tickets**.

We will be building and learning the following in this section:

1. **Routing**: Setting up routing in Next.js, and creating new pages. We focus a little bit on Next.js Layouts and Pages, so you can follow along.
2. **Querying Supabase**: Fetching data from the database using Supabase in Next.js.
3. **Server Components**: Rendering data using React Server Components in Next.js
4. **Client Components**: How and why we switch to Client Components in Next.js when we need to.

As you may have noticed, this will be a *huuuge* section, so let's get started! 🚀

## Anatomy of a Next.js Page

Before we get into building the features, let's take a moment to understand the anatomy of a Next.js Page.

By default - Next.js uses Server Components for rendering every component, unless we opt-out of it.

Server Components are components rendered **exclusively on the server**, unlike Client Components that are rendered both **on the server and the client** (although the name is a bit misleading).

Generally speaking:
1. **Server Components** render the skeleton of the page, and pass the data down to Client Components. Imagine this as render-only subset of React, where its main objective is to render the page with data fetched at runtime. They're only rendered on the server, and never hydrated on the client.
2. **Client Components** add interactivity to the page, and **are rendered both on the server and on the client**. We can use hooks such as useState, useEffect, and others in Client Components. Do they render only on the client? No! They render on the server too, but they are hydrated on the client.

If every component is a Server Component, then you turn it into a Client Component by opting out of it. This is done by using the `use client` directive at the top of the file.

```tsx
'use client';
// This component is now a Client Component
```

{% img src="/assets/courses/next-turbo/nextjs-page-anatomy.webp" width="965" height="658" /%}

In the image above, we can see the anatomy of a Next.js Page using Server Components and Client Components.

Image the following file structure:

```
- home
  - layout.tsx
  - page.tsx
```

We could "think" of it as the following JSX structure:

```jsx
<Layout>
  <Page />
</Layout>
```

At every boundary between layouts and pages, Next.js will automatically switch to Server Components, until you opt out of a component, where all its tree will be Client Components.

Imagine the following layout:

```tsx
export default function MyLayout(props: React.PropsWithChildren) {
  return (
     <MyClientComponent>
        {props.children}
     </MyClientComponent>
  );
}
```

We can "think" of it as the following JSX structure:

```tsx
<Layout> <--- Server Component
  <MyClientComponent> <--- Client Component
    <Page /> <--- Server Component
  </MyClientComponent>
</Layout>
```

As you can see, the `Layout` component is a Server Component, and the `MyClientComponent` is a Client Component. However, the `Page` component is a Server Component, and will render server components until we opt out of it.

{% img src="/assets/courses/next-turbo/nextjs-page-data-fetching.webp" width="619" height="429" /%}

In the image above, we can see how the Server Component is responsible for fetching the data, and passing it down to the Client Component and another Server Component.

Both Server Components and Client Components will render the data, but some components **need to be client components** if you attach an event handler (onClick), or if you need React Hooks, or anything that is purely interactive.

We have now covered the basics of Server Components and Client Components in Next.js. For more information, you can refer to the [Next.js documentation](https://nextjs.org/docs/app/building-your-application/rendering/server-components).

## Understanding where to place Routes in Makerkit

SaaS applications usually falls into two categories: B2B and B2C. Makerkit is flexible to adapt to both, but in this case, we're really building a pure B2B SaaS application.

As such - we want to place our routes in the `/home/[account]` layout - i.e. the team account layout. In fact - this application is meant to be used by teams of users, unlike a common B2C application where it's just one user.

As I said - Makerkit supports both, but in this case, we're focusing on B2B.

### Adding the Team Accounts List to the User home page

When customers log in to the application, they will be redirected to the `/home` route. This is the user home page, and depending on your application, you may want to display a list of team accounts the user is a member of.

To add this list, you will import the `HomeAccountsList` component:

```tsx {% title="apps/web/app/home/(user)/page.tsx" %}
import { HomeAccountsList } from './_components/home-accounts-list';
```

```tsx {% title="apps/web/app/home/(user)/page.tsx" %}
<PageBody>
  <HomeAccountsList />
</PageBody>
```

This is the ideal scenario in B2B SaaS applications, where users can be part of multiple team accounts. The `HomeAccountsList` component will display a list of team accounts the user is a member of.

However, if you're doing B2C - you may want to display different data here, as it's the user's home page.

{% img src="/assets/courses/next-turbo/home-accounts-list.webp" width="1640" height="810" /%}

## Listing Support Tickets

Our first task is to create a page where support agents can view new support tickets. We will be fetching the data from the database and displaying it on the page.

We already have our Database setup (if not, please refer to the previous section) - but we now need the:

1. **Pages**: the pages where we will display the support tickets.
2. **Components**: the components that will display the support tickets data.
3. **Database**: the queries that will fetch the support tickets data from the database.

### Adding a menu Entry for Support Tickets

To create a menu entry to the Account Layout, we need to modify the team account navigation configuration.

In the above, we add the `Support Tickets` menu entry to the team account navigation configuration. We point to the `/home/[account]/tickets` route, which we will create in the next step.

```tsx {% title="apps/web/config/team-account-navigation.config.tsx" %} {1,12-17}
import { CreditCard, LayoutDashboard, Settings, Users, MessageCircle } from 'lucide-react';

//..

const getRoutes = (account: string) => [
  {
    label: 'common:routes.application',
    children: [
      {
        label: 'common:routes.dashboard',
        path: pathsConfig.app.accountHome.replace('[account]', account),
        Icon: <LayoutDashboard className={iconClasses} />,
        end: true,
      },
      {
        label: 'Support Tickets',
        collapsible: false,
        path: `/home/${account}/tickets`,
        Icon: <MessageCircle className={iconClasses} />,
      },
    ]
  },
  {
    label: 'common:routes.settings',
    collapsible: false,
    children: [
      {
        label: 'common:routes.settings',
        path: createPath(pathsConfig.app.accountSettings, account),
        Icon: <Settings className={iconClasses} />,
      },
      {
        label: 'common:routes.members',
        path: createPath(pathsConfig.app.accountMembers, account),
        Icon: <Users className={iconClasses} />,
      },
      featureFlagsConfig.enableTeamAccountBilling
        ? {
            label: 'common:routes.billing',
            path: createPath(pathsConfig.app.accountBilling, account),
            Icon: <CreditCard className={iconClasses} />,
          }
        : undefined,
    ].filter(Boolean),
  },
];
```

### Creating the Support Tickets Page and retrieving data

This is meant as a B2B SaaS - and as such we will be adding our routes in the `/home/[account]` layout.

Let's begin by adding a new route at `/home/[account]/tickets` by adding a new file `apps/web/home/[account]/tickets/page.tsx`.

NB: the kit uses i18n throughout the application, but for simplicity, **we will not be adding i18n to the course** and will be adding English text directly into the components.

```tsx {% title="apps/web/home/[account]/tickets/page.tsx" %}
import { PageHeader, PageBody } from '@kit/ui/page';

export default function TicketsPage() {
  return (
      <>
        <PageHeader
          title={'Support Tickets'}
          description={'Here is the list of the support tickets from your customers'}
        />

        <PageBody>
          {/* Add the support tickets list here */}
        </PageBody>
      </>
  );
}
```

We have added a quick page layout with a title and description. Please log in to the default team account and then navigate to the `/home/[account]/tickets` route to see the page to see the page.

Yay, we have a new page! 🎉 Let's now fetch the support tickets data from the database and display it on the page.

{% img src="/assets/courses/next-turbo/empty-tickets-page.webp" width="1780" height="834" /%}

### Fetching Support Tickets Data

To display the support tickets data on the page, we need to fetch the data from the database. As we use Supabase, we will be using the Supabase client to fetch the data.

#### Quick introduction to Supabase Clients and Next.js

Depending on whether you are running your code in the browser or on the server, you will need to use different clients to interact with Supabase.

### Using the Supabase client in the browser

To import the Supabase client in a browser environment,  you can use the `useSupabase` hook:

```tsx
'use client';

import { useSupabase } from '@kit/supabase/hooks/use-supabase';

function MyComponent() {
  const supabase = useSupabase();

  // Use the supabase client here

  return <div>My Component</div>;
}
```

### Using the Supabase client in a Server environment

To import the Supabase client in a server environment, you can use the `getSupabaseServerClient` function, and you can do the same across all server environments like Server Actions, Route Handlers, and Server Components:

```tsx
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export async function myServerAction() {
  const supabase = getSupabaseServerClient();

  const { data, error } = await supabase.from('users').select('*')

  return {
    success: true,
  };
}
```

To use the Server Admin, e.g. an admin client with elevated privileges, you can use the `getSupabaseServerAdminClient` function:

```tsx
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

export async function myServerAction() {
  const supabase = getSupabaseServerAdminClient();

  const { data, error } = await supabase.from('users').select('*')

  return {
    success: true,
  };
}
```

NB: The `getSupabaseServerAdminClient` function should only be used in server environments, as it requires the `SUPABASE_SERVICE_ROLE_KEY` environment variable to be set. Additionally, it should only be used in very exceptional cases, as it has elevated privileges. In most cases, please use the `getSupabaseServerClient` function.



#### Creating a Service to fetch Support Tickets

Next.js offers a lot of freedom in how you structure your application, so we need to self-discipline ourselves to create a consistent set of patterns for a cohesive and maintainable codebase.

In my quest to make this as simple but also as maintainable as possible, I tend to create `services` that encapsulate the logic for fetching data from the database. These receive a `SupabaseClient` instance and provide the methods needed to interact with the Supabase DB.

In the core kit - **I do this using Turborepo packages**. However, for the sake of simplicity, we will instead use the `apps/web/lib` folder for adding shared/core logic that can be used across the application.

My services usually follow the following structure:

```tsx
export function createService(client: SupabaseClient) {
  return new MyService(client);
}

class MyService {
  constructor(client: SupabaseClient) {}
}
```

If you don't like this pattern, you can use any pattern you like. The goal is to have a consistent pattern across the application, whichever this may be.

Let's create a new file `apps/web/lib/server/tickets/tickets.service.ts` and add the following code:

```tsx {% title="apps/web/lib/server/tickets/tickets.service.ts" %}
import { SupabaseClient } from '@supabase/supabase-js';

import { Database } from '~/lib/database.types';

export function createTicketsService(client: SupabaseClient<Database>) {
  return new TicketsService(client);
}

class TicketsService {
  constructor(private readonly client: SupabaseClient<Database>) {}

  async getTickets(params: {
    accountSlug: string;
    page: number;
    limit?: number;
    query?: string;
  }) {
    const limit = params.limit ?? 10;
    const startOffset = (params.page - 1) * limit;
    const endOffset = startOffset + limit;

    let query = this.client
      .from('tickets')
      .select('*, account_id !inner (slug)', {
        count: 'exact',
      })
      .eq('account_id.slug', params.accountSlug)
      .order('created_at', { ascending: false })
      .range(startOffset, endOffset);

    if (params.query) {
      query = query.textSearch('title', `"${params.query}"`);
    }

    const { data, error, count } = await query;

    if (error) {
      throw error;
    }

    return {
      data: data ?? [],
      count: count ?? 0,
      pageSize: limit,
      page: params.page,
      pageCount: Math.ceil((count ?? 0) / limit),
    };
  }
}
```

Let's break down the code above:

1. We create a `createTicketsService` function that receives a `SupabaseClient` instance and returns a new `TicketsService` instance.
2. The `TicketsService` class has a `getTickets` method that fetches the support tickets data from the database. It receives the `accountSlug`, `page`, `limit`, and `query` as parameters.
3. We use the `client.from('tickets')` method to fetch the support tickets data from the `tickets` table in the database. We use the `eq` method to filter the support tickets by the `accountSlug`.
4. We use the `range` method to paginate the support tickets data. We calculate the `startOffset` and `endOffset` based on the `page` and `limit` parameters.
5. We use the `textSearch` method to search the support tickets data by the `title` field if the `query` parameter is provided.
6. We return a set of data that includes the support tickets data, the total count of support tickets, the number of support tickets per page, the current page, and the total number of pages. This is very useful for our Table component to display the data and for pagination.

A couple of things you should note:

1. We are expanding the data returned ```'*, account_id !inner (slug)'``` to include the joined table `accounts` to get the account slug. This is done so we can filter the tickets by the account slug without having to do a separate query for the account ID.
2. We are using the `exact` count method to get the exact count of the support tickets data. This is useful for pagination.

Let's now use the `TicketsService` to fetch the support tickets data and display it on the page.

```tsx {% title="apps/web/home/[account]/tickets/page.tsx" %}
import { use } from 'react';

import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { DataTable } from '@kit/ui/enhanced-data-table';
import { PageBody, PageHeader } from '@kit/ui/page';

import { createTicketsService } from '~/lib/server/tickets/tickets.service';

interface TicketsPageProps {
  params: Promise<{
    account: string;
  }>;

  searchParams: Promise<{
    page?: string;
    query?: string;
  }>;
}

export default function TicketsPage(props: TicketsPageProps) {
  const client = getSupabaseServerClient();
  const service = createTicketsService(client);

  const { account } = use(props.params);
  const { page: pageParam, query = '' } = use(props.searchParams);
  const page = Number(pageParam ?? '1');

  const { data, pageSize, pageCount } = use(
    service.getTickets({
      accountSlug: account,
      page,
      query,
    }),
  );

  return (
    <>
      <PageHeader
        title={'Support Tickets'}
        description={
          'Here is the list of the support tickets from your customers'
        }
      />

      <PageBody>
        <DataTable
          pageIndex={page - 1}
          pageCount={pageCount}
          pageSize={pageSize}
          data={data}
          columns={[]}
        />
      </PageBody>
    </>
  );
}
```

Let's take a quick look at the code above.

- First, **we define the interface of the component props**. At this time, it's still a manual process to define the props, but in the future, Next.js may do it for us:

```tsx
interface TicketsPageProps {
  params: {
    account: string;
  };

  searchParams: {
    page?: string;
    query?: string;
  };
}
```

The interface above defines the props that the component will receive. We have two parameters:

1. **searchParams**: we have a URL with a query parameter `page` and `query` that we can use to paginate and search the support tickets data.
2. **params**:  we have the `account` parameter that we can use to filter the support tickets data by the account slug in our path: `/home/[account]/tickets`.

When you have a Layout or a Page Server component, you will receive the `params` and `searchParams` as props.

```tsx
export default function TicketsPage(
  { params, searchParams }: TicketsPageProps
) {
  //...
}
```

NB: pages are always **exported as default** in Next.js.

Then, we use the `getSupabaseServerClient` function to get the Supabase client instance. We use the `createTicketsService` function to create a new `TicketsService` instance.

```tsx {% title="apps/web/home/[account]/tickets/page.tsx" %} {8-13}
export default function TicketsPage(props: TicketsPageProps) {
  const client = getSupabaseServerClient();
  const service = createTicketsService(client);

  const page = Number(props.searchParams.page ?? '1');
  const query = props.searchParams.query ?? '';

  const { data, pageSize, pageCount } = use(
    service.getTickets({
      accountSlug: props.params.account,
      page,
      query,
    })
  );
  //...
```

We use the `service.getTickets` to fetch the tickets using the parameters provided, and we use the `use` hook to "syncify" the data fetching process (as you can see, we don't use `async/await`). However, you could switch to `async/await` and replace `use`.

Finally, we display the support tickets data using the `DataTable` component. We pass the `data`, `pageSize`, `pageCount`, and other props to the `DataTable` component, which will render accordingly.

Wait a second... 🤔

**Small issue here**: the `columns` prop is empty, so the DataTable will not render any data. Let's fix this by creating a new component that will wrap the `DataTable` component and define the columns for the support tickets data.

### Displaying the Support Tickets Data

We will be using the `DataTable` component from the `@kit/ui/enhanced-data-table` package to display the support tickets data. The `DataTable` component is a powerful and flexible component that allows you to display data in a table format.

This component wraps the vanilla `@kit/ui/enhanced-data-table` from Shadcn UI with more functionality related to pagination.

In the steps above, we have mentioned that the `columns` prop is empty. Here we define the React Table props that define how
the columns are rendered. However, we have a small issue: we need to pass functions down to the `DataTable` component, which is a client component!
Unfortunately, we can only pass data that can be serialized, and as you can imagine, functions cannot be serialized.

So - here's a good learning point: **if you need to pass functions down to a client component, you will need to wrap the component into a client component and define the functions there**.

Therefore, the solution is to create a separate client component named `TicketsDataTable` that will wrap the `DataTable` component and define the columns and other functions there. Easy, let's do it!

We also want two reusable components for displaying the status and priority of the support tickets. We will be using the `Badge` component from the Shadcn UI kit to display the status and priority of the support tickets.

```tsx title="apps/web/home/[account]/tickets/_components/ticket-status-badge.tsx"
import { Badge } from '@kit/ui/badge';

import { Tables } from '~/lib/database.types';

export function TicketStatusBadge({
  status,
}: {
  status: Tables<'tickets'>['status'];
}) {
  switch (status) {
    case 'open':
      return <Badge variant={'warning'}>Open</Badge>;

    case 'closed':
      return <Badge variant={'secondary'}>Closed</Badge>;

    case 'resolved':
      return <Badge variant={'success'}>Resolved</Badge>;

    case 'in_progress':
      return <Badge variant={'info'}>In Progress</Badge>;
  }
}
```

And the priority badge:

```tsx title="apps/web/home/[account]/tickets/_components/ticket-priority-badge.tsx"
import { Badge } from '@kit/ui/badge';

import { Tables } from '~/lib/database.types';

export function TicketPriorityBadge({
  priority,
}: {
  priority: Tables<'tickets'>['priority'];
}) {
  switch (priority) {
    case 'low':
      return <Badge variant={'outline'}> Low </Badge>;

    case 'medium':
      return <Badge variant={'warning'}> Medium </Badge>;

    case 'high':
      return <Badge variant={'destructive'}> High </Badge>;
  }
}
```

Let's create a new file `apps/web/home/[account]/tickets/_components/tickets-data-table.tsx` and add the following code:

```tsx title="apps/web/home/[account]/tickets/_components/tickets-data-table.tsx"
'use client';

import Link from 'next/link';

import { ColumnDef } from '@tanstack/react-table';

import { Button } from '@kit/ui/button';
import { DataTable } from '@kit/ui/enhanced-data-table';

import { Tables } from '~/lib/database.types';

import { TicketPriorityBadge } from './ticket-priority-badge';
import { TicketStatusBadge } from './ticket-status-badge';

type Ticket = Tables<'tickets'>;

export function TicketsDataTable(props: {
  data: Ticket[];
  pageSize: number;
  pageIndex: number;
  pageCount: number;
}) {
  return <DataTable {...props} columns={getColumns()} />;
}

function getColumns(): ColumnDef<Ticket>[] {
  return [
    {
      header: 'Title',
      cell({ row }) {
        const ticket = row.original;

        return (
          <Link className={'hover:underline'} href={`tickets/${ticket.id}`}>
            {ticket.title}
          </Link>
        );
      },
    },
    {
      header: 'Status',
      cell({ row }) {
        const ticket = row.original;

        return <TicketStatusBadge status={ticket.status} />;
      },
    },
    {
      header: 'Priority',
      cell({ row }) {
        const ticket = row.original;

        return <TicketPriorityBadge priority={ticket.priority} />;
      },
    },
    {
      header: 'Created At',
      cell({ row }) {
        const ticket = row.original;
        const date = new Date(ticket.created_at);

        return getDateString(date);
      },
    },
    {
      header: 'Updated At',
      cell({ row }) {
        const ticket = row.original;
        const date = new Date(ticket.updated_at);

        return getDateString(date);
      },
    },
    {
      header: '',
      id: 'actions',
      cell({ row }) {
        return (
          <div className={'flex justify-end'}>
            <Button asChild variant={'outline'}>
              <Link href={`tickets/${row.original.id}`}>View Issue</Link>
            </Button>
          </div>
        );
      },
    },
  ];
}

function getDateString(date: Date) {
  const day = date.toLocaleDateString();
  const time = date.toLocaleTimeString();

  return `${day} at ${time}`;
}
```

In this component, we define the `TicketsDataTable` component that wraps the `DataTable` component and defines the columns for the support tickets data. We define the `getColumns` function that returns an array of columns for the support tickets data.

We use a variety of components built into the kit, such as `Badge`, `Button` (from Shadcn UI) to display the support tickets data. We also define the `getDateString` function to format the date string.

We can now go back to the `apps/web/home/[account]/tickets/page.tsx` file and import the `TicketsDataTable` component to display the support tickets data.

```tsx title="apps/web/home/[account]/tickets/page.tsx" {8,46-51}
import { use } from 'react';

import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { PageBody, PageHeader } from '@kit/ui/page';

import { createTicketsService } from '~/lib/server/tickets/tickets.service';

import { TicketsDataTable } from './_components/tickets-data-table';

interface TicketsPageProps {
  params: {
    account: string;
  };

  searchParams: {
    page?: string;
    query?: string;
  };
}

export default function TicketsPage(props: TicketsPageProps) {
  const client = getSupabaseServerClient();
  const service = createTicketsService(client);

  const page = Number(props.searchParams.page ?? '1');
  const query = props.searchParams.query ?? '';

  const { data, pageSize, pageCount } = use(
    service.getTickets({
      accountSlug: props.params.account,
      page,
      query,
    }),
  );

  return (
    <>
      <PageHeader
        title={'Support Tickets'}
        description={
          'Here is the list of the support tickets from your customers'
        }
      />

      <PageBody>
        <TicketsDataTable
          pageIndex={page - 1}
          pageCount={pageCount}
          pageSize={pageSize}
          data={data}
        />
      </PageBody>
    </>
  );
}
```

We have now imported the `TicketsDataTable` component and passed the `data`, `pageSize`, `pageCount`, and other props to the `TicketsDataTable` component. The `TicketsDataTable` component will render the support tickets data in a table format.

The page will now fetch the data from the DB and display it in a table format. You can now navigate to the `/home/[account]/tickets` route to see the support tickets data.

{% img src="/assets/courses/next-turbo/tickets-page.webp" width="3000" height="1276" /%}

## Conclusion

In this section, we learned how to set up routing, fetch data, and write data in your Makerkit Next.js Supabase Turbo project. We built the following:

- **Support Tickets List**: A page for team accounts where support agents can view new support tickets.

We also learned the following Next.js concepts:

1. Server Components and Client Components
2. The anatomy of a Next.js Page
3. Basics of Routing in Next.js

In the next section, we want to deep dive into the following:

1. Displaying a Support Ticket's detail by routing to a new page.
2. Submitting messages to the support ticket.
3. Updating the state of the support ticket.

We will also learn the following:

1. Fetching data with React Query
2. Writing data to the database with Server Actions
3. Using React Hook Form to build forms

Cool! Let's move on to the next section. 🚀


