-----------------
FILE PATH: courses/nextjs-turbo/data-fetching.mdoc

---
title: "Fetching and Mutating data in Makerkit Next.js Supabase Turbo"
description: "Learn how to fetch and mutate data from Server Components in your Makerkit Next.js Supabase Turbo project."
label: "Fetching and Mutating Data"
order: 4
---

Hi there! 👋 Welcome to this module. In this section, we will be doing the following:

- **Ticket Detail**: Create a new page where support agents can view the details of a support ticket and reply to the ticket.

From a technical perspective, we will be covering the following topics:

1. **Forms**: Building forms with React Hook Form and submitting data to the database.
2. **Server Actions/Mutations**: Writing data to the database using Supabase in Next.js and Server Actions
3. **Data Fetching with React Query**: Fetching data with React Query in Next.js and Supabase.

As you may have noticed, this will be a *huuuge* section, so let's get started! 🚀

## Support Ticket Detail

In the next step, we want to enable support agents to view the details of a support ticket and reply to the ticket. We will be creating a new page where support agents can view the details of a support ticket and reply to the ticket.

This part is going to be a bit more complex, as we will be fetching the support ticket data from the database, displaying the support ticket data, and allowing support agents to reply to the ticket, or delete them as needed.

By the end of this lesson, we will have a page where support agents can view the details of a support ticket and reply to the ticket.

{% img src="/assets/courses/next-turbo/support-ticket-detail.webp" width="3032" height="1664" alt="Support Ticket Detail" /%}

Let's first think through the sort of logic we are facing when the customer agent receives a support ticket:

1. **Customer opens a support ticket**: The customer opens a support ticket from the website widget. The ticket is saved to the database with the status `open`.
2. **Support agent views the support ticket**: The support agent views the support ticket from the support tickets list. The support agent can view the details of the support ticket and reply to the ticket.
3. **Support agent replies to the support ticket**: The support agent replies to the support ticket. The reply is saved to the database and the status of the support ticket is updated to `in_progress` as soon as the support agent replies to the ticket.
4. **Customer replies to the support ticket**: The customer replies to the support ticket. The reply is saved to the database.
5. **Support agent actions the ticket**: At this point, the support agent can either close the ticket or mark it as resolved. The status of the support ticket is updated to `closed` or `resolved` as needed.

#### How do we handle the status of the support ticket?

This sequence of events raises some questions about our data model, such as:

1. How do we handle the status of the support ticket?
2. How do we handle the replies to the support ticket?

We will be revisiting our data model to handle the status of the support ticket and the replies to the support ticket, as we've mentioned in the Database module.

### Updating the Schema to handle the status of the Support Ticket

As we've just mentioned, handling the flow of a support ticket **in a consistent manner** database-wise is very important, and ultimately a requirement for a good user experience.

We will now be updating the schema so that we can handle the status of a ticket using constraints and triggers, which Postgres provides us to enforce data integrity.

#### Handling the Status of the Ticket using Triggers

Next, we add a trigger to handle the status of the ticket. We will be using the `handle_ticket_status_change` function to handle the status of the ticket.

```sql
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
```

The trigger above does the following:
1. We created a new function `handle_ticket_status_change` that handles the status changes of the ticket. The function sets the `closed_by`, `closed_at`, `resolved_by`, and `resolved_at` fields based on the status of the ticket.
2. We created a new trigger `handle_ticket_status_change_trigger` that executes the `handle_ticket_status_change` function before updating the status of the ticket.

By using these triggers, we make sure that the status of the ticket is updated consistently as the status of the ticket is updated.

#### 💡 Learning Point: Triggers and Constraints

In the above, we have used triggers and constraints to enforce data integrity in the database. Triggers and constraints are powerful tools that you can use to enforce data integrity in the database, and it's very likely you'll be employing them in your application to ensure that the data is consistent and accurate.

I'd suggest you dive more into these by yourself, as they are a very important part of database design.

In the next sections, we will:

1. **Create the Support Ticket Detail Page**: A page where support agents can view the details of a support ticket and reply to the ticket.
2. **Fetch the Support Ticket Data**: Fetch the support ticket data from the database and display it on the page.
3. **Reply to the Support Ticket**: Allow support agents to reply to the support ticket.
4. **Manage the Ticket Status**: Allow support agents to close the support ticket, set the ticket as resolved, or assign the ticket to another support agent.

Let's go!

### Extending the Ticket Service to fetch a single ticket

Let's extend the `TicketsService` to fetch a single support ticket from the database. We will be using the `getTicket` method to fetch the support ticket data from the database.

```tsx {% title="apps/web/lib/server/tickets/tickets.service.ts" %}
import { SupabaseClient } from '@supabase/supabase-js';

import { Database, Tables } from '~/lib/database.types';

class TicketsService {
  //...

  async getTicket(params: { ticketId: string; account: string }) {
    const { data, error } = await this.client
      .from('tickets')
      .select<
        string,
        Tables<'tickets'> & {
          account_id: {
            id: string;
            slug: string;
          };
        }
      >('*, account_id !inner (slug, id)')
      .eq('id', params.ticketId)
      .eq('account_id.slug', params.account)
      .single();

    if (error) {
      throw error;
    }

    return data;
  }
}
```

NB: we're being a bit eager by fetching all the data from the ticket. In a real-world application, you would only fetch the data you need, and not all the data. In this case, the rows are small, so it's not a big deal.

Perfect! We have now extended the `TicketsService` to fetch a single support ticket from the database. We can now use the `getTicket` method to fetch the support ticket data and display it on the page.

#### What about a ticket's messages?

Great question! 🤔

We could argue that fetching messages is not **really** important to display as we render the page server side. Messages can be a quite long list (therefore paginated) - and I'm of the opinion we would rather fetch the page and render the initial data (i.e. the details) and only then fetch the messages once we're on the client side.

In this way, we load the page quicker, and lazy load the messages directly from the Supabase client, without needing a server roundtrip.

### Creating the Support Ticket Detail Page and retrieving data

Let's create a new page at `home/[account]/tickets/[ticketId]` to view the details of a support ticket and reply to the ticket. We will be fetching the support ticket data from the database and displaying it on the page.

Let's create a new file `apps/web/app/home/[account]/tickets/[ticketId]/page.tsx` and add the following code:

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/page.tsx" %}
import { use } from 'react';

import { getSupabaseServerClient } from '@kit/supabase/server-client';
import { Badge } from '@kit/ui/badge';
import { If } from '@kit/ui/if';
import { Heading } from '@kit/ui/heading';
import { PageBody } from '@kit/ui/page';

import { createTicketsService } from '~/lib/server/tickets/tickets.service';

import { TicketPriorityBadge } from '../_components/ticket-priority-badge';
import { TicketStatusBadge } from '../_components/ticket-status-badge';

interface TicketDetailPageProps {
  params: Promise<{
    ticketId: string;
    account: string;
  }>;

  searchParams: Promise<{
    page?: string;
  }>;
}

export default function TicketDetailPage({
  params,
  searchParams,
}: TicketDetailPageProps) {
  const client = getSupabaseServerClient();
  const service = createTicketsService(client);

  const { ticketId, account } = use(params);
  const page = Number(use(searchParams).page ?? '1');

  const ticket = use(
    service.getTicket({
      ticketId,
      account,
    }),
  );

  const timeAgo = getTimeAgo(ticket.created_at);

  return (
    <div className={'flex flex-1'}>
      <div className={'flex flex-1 flex-col overflow-y-hidden h-screen space-y-8'}>
        <div className={'p-4 border-b'}>
          <div className={'flex flex-col space-y-2.5'}>
            <Heading className={'font-semibold'} level={5}>
              {ticket.title}
            </Heading>

            <div className={'flex space-x-2.5'}>
              <Badge variant={'outline'}>Created {timeAgo}</Badge>

              <If condition={ticket.customer_email}>
                <Badge variant={'outline'}>by {ticket.customer_email}</Badge>
              </If>

              <TicketStatusBadge status={ticket.status}/>
              <TicketPriorityBadge priority={ticket.priority}/>
            </div>
          </div>
        </div>

        <PageBody>
          {/* Add the ticket messages here */}
        </PageBody>
      </div>

      <div className={'flex flex-col w-[25%] border-l py-4'}>
        <PageBody>
          {/* Add the ticket actions here */}
        </PageBody>
      </div>
    </div>
  );
}

function getTimeAgo(timestamp: string, locale = 'en') {
  let value;

  const diff = (new Date().getTime() - new Date(timestamp).getTime()) / 1000;
  const minutes = Math.floor(diff / 60);
  const hours = Math.floor(minutes / 60);
  const days = Math.floor(hours / 24);
  const months = Math.floor(days / 30);
  const years = Math.floor(months / 12);
  const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });

  if (years > 0) {
    value = rtf.format(0 - years, 'year');
  } else if (months > 0) {
    value = rtf.format(0 - months, 'month');
  } else if (days > 0) {
    value = rtf.format(0 - days, 'day');
  } else if (hours > 0) {
    value = rtf.format(0 - hours, 'hour');
  } else if (minutes > 0) {
    value = rtf.format(0 - minutes, 'minute');
  } else {
    value = rtf.format(0 - diff, 'second');
  }

  return value;
}
```

In the code above, we define the `TicketDetailPage` component that fetches the support ticket data from the database and displays it on the page. We use the `getTicket` method to fetch the support ticket data and display it on the page.

The rest of the code is pretty straightforward, we display the ticket details, and we have a placeholder for the ticket messages and ticket actions.

Now, we will be adding the two large parts that make up this page, which we add as separate components:

1. **Ticket Messages**: A component that fetches the ticket messages from the database and displays them on the page using the `TicketMessagesContainer` component, and lets the user reply to the ticket.
2. **Ticket Details**: A component that allows updating the status of the ticket, replying to the ticket, and deleting the ticket using the `TicketDetailsContainer` component.

Let's start with the `TicketMessagesContainer` component.

### Fetching and displaying the ticket's messages

In this section, we will:

1. Fetch the ticket messages from the database using React Query
2. Display the ticket messages on the page
3. Allow the user to reply to the ticket

Let's create a component `TicketMessagesContainer` at `apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx` and add the following code.

We have two missing pieces to this (which we commented out below):
1. The Server Action `insertTicketMessageAction` which we will add in the next section. We use this to insert a new message into the database.
2. The `MessageFormSchema` which we will add in the next section. We use this to validate the message form using Zod.

Here is the code (you can safely ignore the commented out parts):

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx" %}
'use client';

import { useState, useTransition, useCallback, useRef, useEffect } from 'react';

import { zodResolver } from '@hookform/resolvers/zod';
import { useInfiniteQuery, useQueryClient } from '@tanstack/react-query';
import { useForm } from 'react-hook-form';

import { useSupabase } from '@kit/supabase/hooks/use-supabase';
import { Alert, AlertTitle } from '@kit/ui/alert';
import { Button } from '@kit/ui/button';
import { Input } from '@kit/ui/input';
import { LoadingOverlay } from '@kit/ui/loading-overlay';
import { cn } from '@kit/ui/utils';

import { Tables } from '~/lib/database.types';

// import { MessageFormSchema } from '../_lib/schema/message-form.schema';
// import { insertTicketMessageAction } from '../_lib/server/server-actions';

type Message = Tables<'messages'> & {
  account: {
    email: string;
    name: string;
    picture_url: string;
  };
};

export function TicketMessagesContainer(props: {
  ticketId: string;
  page: number;
}) {
  const [state] = useState<{
    page: number;
  }>({
    page: props.page,
  });

  const scrollingDiv = useRef<HTMLDivElement | null>(null);
  const queryKey = ['ticket-messages', props.ticketId, props.page.toString()];
  const appendMessage = useAppendNewMessage({ queryKey });

  const { status, data } = useFetchTicketMessages({
    ticketId: props.ticketId,
    page: state.page,
    queryKey,
  });

  useEffect(() => {
    if (scrollingDiv.current) {
      scrollingDiv.current.scrollTo({
        top: scrollingDiv.current.scrollHeight,
      });
    }
  }, [data]);

  if (status === 'pending') {
    return (
      <LoadingOverlay fullPage={false}>Loading messages...</LoadingOverlay>
    );
  }

  if (status === 'error') {
    return (
      <Alert variant={'destructive'}>
        <AlertTitle>Error loading messages</AlertTitle>
      </Alert>
    );
  }

  return (
    <div className={'flex flex-1 flex-col relative'}>
      <div
        ref={ref => {
          scrollingDiv.current = ref;
        }}
        className={'flex flex-col gap-4 pb-24 overflow-y-auto h-full w-full absolute'}
      >
        {data.pages.map((page) => {
          return page.map((message) => (
            <TicketMessage key={message.id} message={message} />
          ));
        })}
      </div>

      <SendMessageInput
        ticketId={props.ticketId}
        onMessageSent={appendMessage}
      />
    </div>
  );
}

function SendMessageInput({
  onMessageSent,
  ticketId,
}: {
  ticketId: string;
  onMessageSent: (message: Tables<'messages'>) => void;
}) {
  const [pending, startTransition] = useTransition();

  const form = useForm({
    // resolver: zodResolver(MessageFormSchema),
    defaultValues: {
      message: '',
      ticketId,
    },
  });

  return (
    <form
      className={'sticky bottom-8 mt-auto z-10'}
      onSubmit={form.handleSubmit((data) => {
        startTransition(async () => {
          // const message = await insertTicketMessageAction(data);

          // onMessageSent(message);
        });
      })}
    >
      <Input
        {...form.register('message')}
        disabled={pending}
        placeholder={'Send a reply...'}
        type="text"
        className={'h-16 border pr-36 bg-background focus:shadow-xl'}
      />

      <Button disabled={pending} className={'absolute right-4 top-3.5'}>
        Send
      </Button>
    </form>
  );
}

function TicketMessage(props: { message: Message }) {
  const author = props.message.author;
  const content = props.message.content;
  const account = props.message.account;

  const alignClassname = cn('flex w-full lg:w-6/12', {
    'self-end justify-end': author === 'support',
    'self-start': author === 'customer',
  });

  const className = cn('rounded-lg w-full border flex gap-4 p-2.5', {
    'bg-primary/5 text-primary-900 dark:bg-primary/90': author === 'support',
  });

  const authorName =
    author === 'customer'
      ? 'Customer'
      : account?.name || account?.email || `Support`;

  const date = new Date(props.message.created_at);

  return (
    <div className={alignClassname}>
      <div className={'flex max-w-full w-auto flex-col gap-2'}>
        <div className={'flex flex-col'}>
          <div className={'font-medium text-sm capitalize'}>
            {authorName}
          </div>

          <div>
            <span className={'text-xs text-muted-foreground'}>
              Sent on {date.toLocaleString('en-US')}
            </span>
          </div>
        </div>

        <div className={className}>
          <p className={'inline-block break-words text-sm'}>{content}</p>
        </div>
      </div>
    </div>
  );
}

function useAppendNewMessage(params: { queryKey: string[] }) {
  const queryClient = useQueryClient();
  const { queryKey } = params;

  return useCallback(
    (message: Tables<'messages'>) => {
      queryClient.setQueryData(
        queryKey,
        (data: { pages: Array<Tables<'messages'>[]> }) => {
          // append message to the last page
          const lastPage = [...data.pages[data.pages.length - 1]!, message];

          return {
            ...data,
            // replace the last page
            pages: [...data.pages.slice(0, -1), lastPage],
          };
        },
      );
    },
    [queryClient, queryKey],
  );
}

function useFetchTicketMessages(params: {
  ticketId: string;
  page: number;
  queryKey: string[];
}) {
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
      .order('created_at', { ascending: true })
      .range(startOffset, endOffset);

    if (error) {
      throw error;
    }

    return messages;
  };

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

Let's break down the key parts of this `TicketMessagesContainer` component:

### Fetching Messages with React Query

The component uses React Query's `useInfiniteQuery` hook to fetch ticket messages:

```tsx
const { status, data } = useFetchTicketMessages({
  ticketId: props.ticketId,
  page: state.page,
  queryKey,
});
```

The `useFetchTicketMessages` custom hook encapsulates the logic for fetching messages:

```tsx
function useFetchTicketMessages(params: {
  ticketId: string;
  page: number;
  queryKey: string[];
}) {
  const client = useSupabase();
  const messagesPerPage = 25;

  const queryFn = async () => {
    // ... Supabase query logic ...
  };

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

This setup allows for efficient pagination and caching of messages.

#### Why fetch data with React Query instead of a Server Component?

Good question! 🤔

When deciding between fetching data with a Server Component or React Query, consider the following: does the data need to **absolutely** be rendered as part of the initial page load?

When the answer is yes - use a Server Component to fetch the data and render it. When the answer is no - use React Query and load the data as you need it. In this case, we're fetching messages that are not essential to the initial page load, so React Query is a good choice (although you may disagree with me here).

Another approach could have been:
1. fetching one single page of messages
2. passing it to React Query as initial data

This would have been a good compromise as well, but it's slightly more complex.

### Displaying Messages

The component maps over the fetched messages and renders them:

```tsx
<div className={'flex flex-col gap-4 overflow-y-auto'}>
  {data.pages.map((page) => {
    return page.map((message) => (
      <TicketMessage key={message.id} message={message} />
    ));
  })}
</div>
```

The `TicketMessage` component handles the display of individual messages, including author information and styling based on the message type (customer or support).

### Sending New Messages

The `SendMessageInput` component provides a form for users to send new messages:

```tsx
<SendMessageInput
  ticketId={props.ticketId}
  onMessageSent={appendMessage}
/>
```

This component uses React Hook Form for form management and will use a server action (to be implemented) to send the message:

```tsx
<form
  className={'relative mb-4 mt-auto'}
  onSubmit={form.handleSubmit((data) => {
    startTransition(async () => {
      // const message = await insertTicketMessageAction(data);
      // onMessageSent(message);
    });
  })}
>
  {/* Input and button */}
</form>
```

Minor, but worth noting: we use a ref to scroll to the bottom of the message container when new messages are added:

```tsx
const scrollingDiv = useRef<HTMLDivElement | null>(null);
```

```tsx
useEffect(() => {
  if (scrollingDiv.current) {
    scrollingDiv.current.scrollTo({
      top: scrollingDiv.current.scrollHeight,
    });
  }
}, [data]);
```

This ensures that the user sees new messages as they are sent.

### Updating the UI with New Messages

The `useAppendNewMessage` hook provides a function to update the React Query cache when a new message is sent:

```tsx
function useAppendNewMessage(params: { queryKey: string[] }) {
  const queryClient = useQueryClient();
  const { queryKey } = params;

  return useCallback(
    (message: Tables<'messages'>) => {
      queryClient.setQueryData(
        queryKey,
        (data: { pages: Array<Tables<'messages'>[]> }) => {
          // append message to the last page
          const lastPage = [...data.pages[data.pages.length - 1]!, message];

          return {
            ...data,
            // replace the last page
            pages: [...data.pages.slice(0, -1), lastPage],
          };
        },
      );
    },
    [queryClient, queryKey],
  );
}
```

This ensures that new messages appear immediately in the UI without requiring a full refetch.

To complete this feature, we will need to:

1. Implement the `insertTicketMessageAction` server action
2. Create the `MessageFormSchema` for form validation
3. Uncomment the related code in the `SendMessageInput` component

This setup provides a solid foundation for a real-time messaging interface within our support ticket system, which we will build upon in the next lessons.

#### Adding the `TicketMessagesContainer` component to the page

Let's go back to the page and import this component:

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/page.tsx" %}
import { TicketMessagesContainer } from './_components/ticket-messages-container';
```

And add the component to the page in the left-hand side section:

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/page.tsx" %}
<PageBody>
  <TicketMessagesContainer ticketId={ticket.id} page={page} />
</PageBody>
```

Nice! You should now be able to display the ticket messages on the page. The `TicketMessagesContainer` component fetches the ticket messages from the database and displays them on the page. The `SendMessageInput` component allows the user to reply to the ticket (not working yet).

Let's analyze some of the meaty parts of the code above.

##### Fetching the Ticket Messages

We use the `useFetchTicketMessages` hook to fetch the ticket messages from the database. The `useFetchTicketMessages` hook fetches the ticket messages from the database using the `useInfiniteQuery` hook from React Query.

```tsx
const { data: messages, error } = await client
  .from('messages')
  .select<
    string,
    Message
  >('*, account: author_account_id (email, name, picture_url)')
  .eq('ticket_id', ticketId)
  .order('created_at', { ascending: false })
  .range(startOffset, endOffset);
```

In the above we use the Supabase Client (directly from the browser) and we select the messages from the `messages` table. We select the `account` field from the `author_account_id` field to get the account details of the author of the message. We filter the messages based on the `ticket_id` field and order the messages by the `created_at` field in descending order.

In addition, we make a join to the `account` table to get the account details of the author of the message:

```
account: author_account_id (email, name, picture_url)'
```

This literally means:
1. Get the `author_account_id` field from the message
2. Expand the `author_account_id` field to get the `email`, `name`, and `picture_url` fields from the `account` table.
3. Name the returned object `account` (so we can access it as `message.account`)
4. The `account` object will have the `email`, `name`, and `picture_url` fields.

##### Paginating the Ticket Messages

```tsx
const [state, setState] = useState<{
  page: number;
}>({ page: props.page });

const queryKey = ['ticket-messages', props.ticketId, props.page.toString()];
const queryClient = useQueryClient();

const { status, data } = useFetchTicketMessages({
  ticketId: props.ticketId,
  page: state.page,
  queryKey,
});
```

The `queryKey` is used to cache the data in the React Query cache. We use the `useQueryClient` hook to access the query client and update the query data when a new message is sent.

Once the user changes page (as we paginate a possibly long list), React Query will fetch the next page of messages. The navigation buttons are missing at this point, but we will get back here.

##### Updating the cache when a new message is sent

To instantly update the UI when a new message is sent, we update the cache using the `queryClient.setQueryData` method when a new message is sent.

We use the hook `useAppendNewMessage` to update the cache when a new message is sent:

```tsx
const appendMessage = useAppendNewMessage({ queryKey });

<SendMessageInput
  ticketId={props.ticketId}
  onMessageSent={appendMessage}
/>
```

### Introducing Server Actions

Now that we can display the messages on page, we want to add the Server Action `insertTicketMessage` to insert a new message into the database.

Let's talk about Server Actions a little bit.

#### What are Server Actions?

**Server Actions are special Javascript Functions that React turns into API endpoints**. They are used to perform server-side logic that can't be done on the client side, such as interacting with the database, sending emails, etc.

The special thing about it is that we just call them like normal functions! But behind the scenes, they are actually making an API request to the server. The amount of code we cut down by using Server Actions is huge, and it's a very powerful feature of React.

#### Creating a Server Action
You can create a Server Action in two ways:

1. **Individual Server Action**: You can create a single Server Action in a file by adding `use server` at the top of the function. This only works if it's defined in a Server Component.
2. **Server Actions file**: by adding `use server` at the top of the file, all the **exported functions** in the file will be turned into Server Actions, i.e. endpoints that can be called from the client. This is the only way to call a Server Action from a client component.

#### Misconceptions about Server Actions

There are a few misconceptions about Server Actions that I'd like to clear up:

1. **Server Actions and Server Components are not the same**: Server Actions are functions that are turned into POST API endpoints, while Server Components are React components that are rendered exclusively server-side and stream serializable content to the client
2. **You don't need a Server Action to call a function in a Server Component**: a function in a Server Component is just it, a function. As long as it's not passed to a Client Component, you can call any functions you wish - without it being a Server Action. You only need a Server Action if these are called in response to user input (such as a form submission).

```tsx
// ✅ a server action that can be called from a server component
// ❌ a server action that cannot be called from a client component
function myServerAction() {
  'use server';

  return 'Hello from the server!';
}
```

If you need to call a Server Action from a client component, you should opt for the second option, creating a Server Actions file.

```tsx
'use server';

// ✅ a server action that can be called from a client component
export function myServerAction() {
  return 'Hello from the server!';
}

// just a function
function hiThere() {
  return 'Hi there!';
}
```

I personally add server actions using the following convention: I create a folder named `_lib/server` and a file named `server-actions.ts` in the page where they're used.

#### Creating the "`insertTicketMessageAction`" Server Action

Makerkit provides the `enhanceAction` utility to greatly improve the DX of creating Server Actions. It provides a lot of features, such as:

1. automatically authenticating the user
2. automatically validating the input
3. automatically handling errors
4. automatically handling captcha protection

The usage is very simple: when creating a server action, wrap the action in `enhanceAction` and provide the action function and the options.

```tsx
'use server';

export const insertTicketMessageAction = enhanceAction(
  async (data, user) => {
    // your logic here
  },
  {
    auth: true,
    schema: MessageFormSchema,
  },
);
```

In the above code, we create a new Server Action `insertTicketMessageAction` that inserts a new message into the database. The Server Action is wrapped in the `enhanceAction` function and we provide the action function and the options:
1. `auth: true` - This option requires the user to be authenticated to call the Server Action. It is `true` by default.
2. `schema: MessageFormSchema` - This option validates the input data using the `MessageFormSchema` schema.
3. Also, it will automatically handle and report errors (Sentry or Baselime), and can also validate the user with a captcha. Please check the documentation for more information.

Validation is done through Zod, for which we need to provide a Zod Schema. Yes, I'm talking about the `MessageFormSchema` we commented out in the previous section.

I like to add schemas into separate files, since we reuse them for the Server Action and the Form. Let's create a new file `apps/web/app/home/[account]/tickets/[ticketId]/_lib/schema/message-form.schema.ts` and add the following code:

```tsx {% title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/schema/message-form.schema.ts" %}
import { object, string } from 'zod';

export const MessageFormSchema = object({
  message: string().min(1).max(5000),
  ticketId: string().uuid(),
});
```

We can now create the `insertTicketMessageAction` Server Action. Let's create a new file `apps/web/app/home/[account]/tickets/[ticketId]/_lib/server/server-actions.ts` and add the following code.

NB: noticed the `use server` at the top of the file? That's how we add a Server Action in Next.js. All the functions exported from a file that declares `use server` will be turned into Server Actions, and will be accessible API endpoints to your application. This means you have to be particularly careful with what you expose.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/server/server-actions.ts"
'use server';

import { enhanceAction } from '@kit/next/actions';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

import { MessageFormSchema } from '../schema/message-form.schema';

export const insertTicketMessageAction = enhanceAction(
  async (data, user) => {
    const client = getSupabaseServerClient();

    const response = await client
      .from('messages')
      .insert({
        content: data.message,
        ticket_id: data.ticketId,
        author_account_id: user.id,
        author: 'support',
      })
      .select('*, account: author_account_id (email, picture_url, name)')
      .single();

    if (response.error) {
      throw new Error(response.error.message);
    }

    return response.data;
  },
  {
    auth: true,
    schema: MessageFormSchema,
  },
);
```

We use the Server Action `insertTicketMessageAction` to insert a new message into the database. The `insertTicketMessage` Server Action inserts a new message into the database and returns the message data.

Simple right? We can now use the `insertTicketMessageAction` Server Action to insert a new message into the database.

💡As you may have noticed, we added the server actions in a folder named `_lib/server`: this is a convention that we use in Makerkit to denote when a file is exclusively for server-side code. This is a good practice to follow, as it makes it clear that the file is not meant to be imported in the client-side code.

### Replying to a Support Ticket

Now, we will comment back in the `MessageFormSchema` and the `insertTicketMessageAction` function in the `TicketMessagesContainer` component.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx"
import { MessageFormSchema } from '../_lib/schema/message-form.schema';
import { insertTicketMessageAction } from '../_lib/server/server-actions';
```

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx"
<form
  className={'relative mb-4 mt-auto'}
  onSubmit={form.handleSubmit((data) => {
    startTransition(async () => {
      const message = await insertTicketMessageAction(data);

      onMessageSent(message);
    });
  })}
>
```

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-messages-container.tsx"
const form = useForm({
  resolver: zodResolver(MessageFormSchema),
  defaultValues: {
    message: '',
    ticketId,
  },
});
```

You can now reply to a ticket, and the page will **automatically** update with the new message as we update the cache with the new message.

Spectacular! 🎉

### Adding the Ticket Details Component

In this section, we will:
1. Add the `TicketDetailsContainer` component to allow updating the status of the ticket and other details
2. Allow the support agent to update the status of the ticket, assignee and priority of the ticket

Let's create a new component `TicketDetailsContainer` at `apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-details-container.tsx` and add the following code:

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-details-container.tsx"
'use client';

import { useTransition } from 'react';

import { useForm } from 'react-hook-form';

import { useTeamAccountWorkspace } from '@kit/team-accounts/hooks/use-team-account-workspace';
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
} from '@kit/ui/form';
import { Heading } from '@kit/ui/heading';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@kit/ui/select';

import { Tables } from '~/lib/database.types';

export function TicketDetailsContainer({
  ticket,
}: {
  ticket: Tables<'tickets'> & {
    account_id: {
      slug: string;
    };
  };
}) {
  const { account } = useTeamAccountWorkspace();
  const permissions = account.permissions;

  // check if the user can update the ticket
  const canUpdateTicket = permissions.includes('tickets.update');

  return (
    <div className={'flex h-screen flex-1 flex-col space-y-8'}>
      <div>
        <Heading level={4}>Ticket details</Heading>

        <Heading level={6} className={'text-muted-foreground'}>
          Manage the status, assignee, and priority of the ticket.
        </Heading>
      </div>

      <div className={'flex flex-1 flex-col space-y-4'}>
        {
          // Add the ticket details form here
        }
      </div>
    </div>
  );
}
```

#### Account Workspace

When you are within the account workspace, e.g. in a page inside the `/home/[account]` layout - you have access to the account workspace.

This is data fetched at layout level from a Server Component and passed to the client, which is extremely useful in a number of situations, such as:

1. **Account Details**: getting the current account ID
2. **Permissions**: getting the list of permissions the user has in the current team
3. **Subscription Status**: getting the subscription status of the account

It includes information needed to bootstrap the page, such as the account, the team, the user, etc. [For a more detailed explanation, check the documentation](https://makerkit.dev/docs/next-supabase-turbo/account-workspace-api).

Of course, you have a wealth of options to retrieve data using the [various API Makerkit offers](/docs/next-supabase-turbo/account-api), but it's very important to know about the account workspace.

Please note that the account workspace is only available in the account workspace layout, and not in the global layout.

#### Checking Permissions to Update a Ticket

We are using the data from the account workspace to **retrieve the user's permissions within the current team and check if the user can update the ticket**. If not, we disable the Select.

**NB - this is only a UX feature**: the real permissions check should be done server-side - in our case, it's done by the RLS using the `public.has_permission` function.

You should always perform checks server-side when it comes to permissions - while relying on these client-side checks just for UX.

#### Adding the TicketDetailsContainer component to the page

Now, let's add the component to the page and pass the ticket data to the component.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/page.tsx"
import { TicketDetailsContainer } from './_components/ticket-details-container';
```

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/page.tsx"
<div className={'flex h-full w-[25%] border-l py-4'}>
  <PageBody>
    <TicketDetailsContainer ticket={ticket} />
  </PageBody>
</div>
```

Now, we will add the forms to update the status, assignee, and priority of the ticket.

Here we will learn about:

1. **Shadcn UI**: Using the Shadcn UI components to create forms
2. **Forms**;Using React Hook Form to manage form state
3. **Server Actions**: Using Server Actions in response to form submissions to update the ticket data

#### Changing the status of a Support Ticket

We will use a Select which will auto-save on change - which is a nice UX touch, rather than rely on a submit button. We will use the `update_ticket_status` function to update the status of the ticket.

Before we get to the UI, I want to add the relative Zod Schema, and the Server Action to update the status of the ticket.

##### Zod Schema for the Ticket Status Form

Let's create a new file `apps/web/app/home/[account]/tickets/[ticketId]/_lib/schema/update-ticket-status.schema.ts` and add the following code:

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/schema/update-ticket-status.schema.ts"
import { z } from 'zod';

export const UpdateTicketStatusSchema = z.object({
  ticketId: z.string().uuid(),
  status: z.enum(['open', 'closed', 'resolved', 'in_progress']),
});
```

We will use this schema to validate the form data before updating the status of the ticket and to validate it server-side in the Server Action.

##### Adding the Server Action to update the status of the ticket

Now, let's add the Server Action to update the status of the ticket.

We use the Supabase client using the `update` method to update the status of the ticket in the database. We use the `eq` method to filter the ticket based on the `id` field and update the status of the ticket.

Let's add the server action `updateTicketStatusAction` to update the status of the ticket:

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/server/server-actions.ts"
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerClient } from '@kit/supabase/server-client'

import { UpdateTicketStatusSchema } from '../schema/update-ticket-status.schema';

export const updateTicketStatusAction = enhanceAction(
  async (data) => {
    const logger = await getLogger();
    const client = getSupabaseServerClient();

    logger.info({ data }, 'Updating ticket status...');

    const response = await client
      .from('tickets')
      .update({
        status: data.status,
      })
      .eq('id', data.ticketId)
      .single();

    if (response.error) {
      logger.error(
        { error: response.error.message },
        'Error updating ticket status',
      );

      throw new Error(response.error.message);
    }

    return response.data;
  },
  {
    auth: true,
    schema: UpdateTicketStatusSchema,
  },
);
```

Of course, we reuse the `enhanceAction` utility to create the Server Action. We use the `UpdateTicketStatusSchema` to validate the form data before updating the status of the ticket.

#### Logging - your new best friend

We use the `getLogger` utility to log the data and errors when updating the status of the ticket. We log the data before updating the status of the ticket and log the error if there is an error updating the status of the ticket.

The logger is your best friend when it comes to debugging and monitoring your application. It's a powerful tool that allows you to log data, errors, and other information to the console, and it's very easy to use.

My recommendation is to **log as much as possible**, especially when you're developing a new feature or debugging an issue. It will help you understand what's happening in your application and make it easier to find and fix bugs.

When you have issues, **logs are the first thing you should check**. They will give you a lot of information about what's happening in your application and help you understand what's going wrong.

This is especially important if you are reporting a bug to Makerkit - logs are the first thing I will ask for - because they are what will help me understand what's going wrong.

##### How to use logging effectively

Logging well means:

1. providing an explanation of what you are doing **before** the action
2. logging the data you are using (avoid logging sensitive data)
3. logging the result of the action
4. logging any errors that occur

This will give you a clear picture of what's happening in your application and help you understand what's going wrong.

Generally speaking you'll add logging to your Server Actions and API handlers.

##### Building the Ticket Status Form

We can now build the Ticket Status Form. Let's add the form to the `TicketDetailsContainer` component.

To do so, we use the following:
1. **useForm**: The `useForm` hook from React Hook Form to manage the form state. It's a bit overkill here, but it's necessary to display the `FormField` component.
2. **Select**: The `Select` component from Shadcn UI to create a dropdown to select the status of the ticket.
3. **updateTicketStatusAction**: The `updateTicketStatusAction` Server Action to update the status of the ticket.
4. **useTransition**: Allows us to get the `pending` state while the Server Action is pending. We disable the form while the Server Action is pending to give feedback to the user.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-details-container.tsx"
'use client';

// all the imports previously added ...

import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
} from '@kit/ui/form';
import { Heading } from '@kit/ui/heading';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@kit/ui/select';

import { Tables } from '~/lib/database.types';

import { UpdateTicketStatusSchema } from '../_lib/schema/update-ticket-status.schema';
import { updateTicketStatusAction } from '../_lib/server/server-actions';

// ... rest of the code

function StatusSelect(props: {
  status: Tables<'tickets'>['status'];
  ticketId: string;
  disabled: boolean;
}) {
  const form = useForm({
    resolver: zodResolver(UpdateTicketStatusSchema),
    defaultValues: {
      status: props.status,
      ticketId: props.ticketId,
    },
  });

  const [pending, startTransition] = useTransition();

  return (
    <Form {...form}>
      <FormField
        render={({ field }) => {
          return (
            <FormItem>
              <FormLabel>Status</FormLabel>

              <FormControl>
                <Select
                  value={form.getValues('status')}
                  disabled={pending || props.disabled}
                  onValueChange={(value) => {
                    form.setValue(
                      field.name,
                      value as Tables<'tickets'>['status'],
                      {
                        shouldValidate: true,
                      },
                    );

                    void form.handleSubmit((value) => {
                      startTransition(async () => {
                        await updateTicketStatusAction(value);
                      });
                    })();
                  }}
                >
                  <SelectTrigger>
                    <SelectValue placeholder={'Status'} />
                  </SelectTrigger>

                  <SelectContent>
                    <SelectItem value={'open'}>Open</SelectItem>
                    <SelectItem value={'closed'}>Closed</SelectItem>
                    <SelectItem value={'in_progress'}>In Progress</SelectItem>
                    <SelectItem value={'resolved'}>Resolved</SelectItem>
                  </SelectContent>
                </Select>
              </FormControl>

              <FormDescription>
                The status of the ticket determines its current state.
              </FormDescription>
            </FormItem>
          );
        }}
        name={'status'}
      />
    </Form>
  );
}
```

In the code above, we define the `StatusSelect` component that allows the user to update the status of the ticket. We use the `Select` component from Shadcn UI to create a dropdown to select the status of the ticket. We use the `updateTicketStatusAction` Server Action to update the status of the ticket.

When the user selects a new status, we call the `updateTicketStatusAction` Server Action to update the status of the ticket. We use the `useTransition` hook to get the `pending` state while the Server Action is pending. We disable the form while the Server Action is pending to give feedback to the user.

Now, let's add the `StatusSelect` component to the `TicketDetailsContainer` component.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-details-container.tsx"
<div className={'flex flex-1 flex-col space-y-4'}>
  <StatusSelect
    disabled={!canUpdateTicket}
    status={ticket.status}
    ticketId={ticket.id}
  />
</div>
```

And you should now be able to update the status of the ticket on the page!

##### Problem! The UI is not updating! 😱

You may have noticed that the UI is not updating when you change the status of the ticket. This is because we are not updating the cache when the status of the ticket is updated.

Whenever we call a Server Action, we want the data fetched from a Server Component to revalidate. In our case, we want the UI to display the new status of the ticket when the status of the ticket is updated.

To do so, we will be using the utility `revalidatePath` from Next.js to revalidate the data fetched from the Server Component when the status of the ticket is updated.

Let's update the server action to include the `revalidatePath` utility:

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/server/server-actions.ts"
import { revalidatePath } from 'next/cache';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerClient } from '@kit/supabase/server-client'

export const updateTicketStatusAction = enhanceAction(
  async (data) => {
    const logger = await getLogger();
    const client = getSupabaseServerClient();

    logger.info({ data }, 'Updating ticket status...');

    const response = await client
      .from('tickets')
      .update({
        status: data.status,
      })
      .eq('id', data.ticketId)
      .single();

    if (response.error) {
      logger.error(
        { error: response.error.message },
        'Error updating ticket status',
      );

      throw new Error(response.error.message);
    }

    revalidatePath(`/home/[account]/tickets/[ticket]`, 'page');

    return response.data;
  },
  {
    auth: true,
    schema: UpdateTicketStatusSchema,
  },
);
```

#### Changing the priority and assignee of a Support Ticket

Since we will be doing very similar things, we will go through this quickly - we will be adding a `PrioritySelect` and an `AssigneeSelect` to the `TicketDetailsContainer` component.

Let's add the Server Actions and the Zod Schemas for the priority and assignee of the ticket.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/schema/update-ticket-priority.schema.ts"
import { z } from 'zod';

export const UpdateTicketPrioritySchema = z.object({
  ticketId: z.string().uuid(),
  priority: z.enum(['low', 'medium', 'high']),
});
```

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_lib/schema/update-ticket-assignee.schema.ts"
import { z } from 'zod';

export const UpdateTicketAssigneeSchema = z.object({
  ticketId: z.string().uuid(),
  assigneeId: z.string().uuid(),
});
```

Now let's add the Server Actions to update the priority and assignee of the ticket.

```tsx
import { revalidatePath } from 'next/cache';
import { getLogger } from '@kit/shared/logger';
import { getSupabaseServerClient } from '@kit/supabase/server-client';

export const updateTicketPriorityAction = enhanceAction(
  async (data) => {
    const logger = await getLogger();
    const client = getSupabaseServerClient();

    logger.info({ data }, 'Updating ticket priority...');

    const response = await client
      .from('tickets')
      .update({
        priority: data.priority,
      })
      .eq('id', data.ticketId)
      .single();

    if (response.error) {
      logger.error(
        { error: response.error.message },
        'Error updating ticket priority',
      );

      throw new Error(response.error.message);
    }

    revalidatePath(`/home/[account]/tickets/[ticket]`, 'page');

    return response.data;
  },
  {
    auth: true,
    schema: UpdateTicketPrioritySchema,
  },
);

export const updateTicketAssigneeAction = enhanceAction(
  async (data) => {
    const client = getSupabaseServerClient();
    const logger = await getLogger();

    logger.info({ data }, 'Updating ticket assignee...');

    const response = await client
      .from('tickets')
      .update({
        assigned_to: data.assigneeId,
      })
      .eq('id', data.ticketId)
      .single();

    if (response.error) {
      logger.error(
        { error: response.error.message },
        'Error updating ticket assignee',
      );

      throw new Error(response.error.message);
    }

    revalidatePath(`/home/[account]/tickets/[ticket]`, 'page');

    return response.data;
  },
  {
    auth: true,
    schema: UpdateTicketAssigneeSchema,
  },
);
```

Below, we add the `PrioritySelect` and `AssigneeSelect` components to the `TicketDetailsContainer` component.

```tsx title="apps/web/app/home/[account]/tickets/[ticketId]/_components/ticket-details-container.tsx"
// all the imports previously added ...

import { UpdateTicketAssigneeSchema } from '../_lib/schema/update-ticket-assignee.schema';
import { UpdateTicketPrioritySchema } from '../_lib/schema/update-ticket-priority.schema';
import { UpdateTicketStatusSchema } from '../_lib/schema/update-ticket-status.schema';
import {
  updateTicketAssigneeAction,
  updateTicketPriorityAction,
  updateTicketStatusAction,
} from '../_lib/server/server-actions';

function AssigneeSelect(props: {
  assignee: Tables<'tickets'>['assigned_to'];
  ticketId: string;
  accountSlug: string;
  disabled: boolean;
}) {
  const [pending, startTransition] = useTransition();

  const form = useForm({
    resolver: zodResolver(UpdateTicketAssigneeSchema),
    defaultValues: {
      assigneeId: props.assignee!,
      ticketId: props.ticketId,
    },
  });

  return (
    <Form {...form}>
      <FormField
        render={({ field }) => {
          return (
            <FormItem>
              <FormLabel>Assignee</FormLabel>

              <FormControl>
                <Select
                  disabled={pending || props.disabled}
                  value={field.value}
                  onValueChange={(value) => {
                    form.setValue(field.name, value, {
                      shouldValidate: true,
                    });

                    void form.handleSubmit((value) => {
                      startTransition(async () => {
                        await updateTicketAssigneeAction(value);
                      });
                    })();
                  }}
                >
                  <SelectTrigger>
                    <SelectValue placeholder={'Choose an Assignee'} />
                  </SelectTrigger>

                  <SelectContent>

                  </SelectContent>
                </Select>
              </FormControl>

              <FormDescription>
                The person responsible for resolving the ticket.
              </FormDescription>
            </FormItem>
          );
        }}
        name={'assigneeId'}
      />
    </Form>
  );
}

function PrioritySelect(props: {
  priority: Tables<'tickets'>['priority'];
  ticketId: string;
  disabled: boolean;
}) {
  const form = useForm({
    resolver: zodResolver(UpdateTicketPrioritySchema),
    defaultValues: {
      priority: props.priority,
      ticketId: props.ticketId,
    },
  });

  const [pending, startTransition] = useTransition();

  return (
    <Form {...form}>
      <FormField
        render={({ field }) => {
          return (
            <FormItem>
              <FormLabel>Priority</FormLabel>

              <FormControl>
                <Select
                  value={form.getValues('priority')}
                  disabled={pending || props.disabled}
                  onValueChange={(value) => {
                    form.setValue(
                      field.name,
                      value as Tables<'tickets'>['priority'],
                      { shouldValidate: true },
                    );

                    void form.handleSubmit((value) => {
                      startTransition(async () => {
                        await updateTicketPriorityAction(value);
                      });
                    })();
                  }}
                >
                  <SelectTrigger>
                    <SelectValue placeholder={'Choose Priority'} />
                  </SelectTrigger>

                  <SelectContent>
                    <SelectItem value={'low'}>Low</SelectItem>
                    <SelectItem value={'medium'}>Medium</SelectItem>
                    <SelectItem value={'high'}>High</SelectItem>
                  </SelectContent>
                </Select>
              </FormControl>

              <FormDescription>
                The priority of the ticket determines how quickly it should be
                resolved.
              </FormDescription>
            </FormItem>
          );
        }}
        name={'priority'}
      />
    </Form>
  );
}
```

Bonus: adding the assignees list will be done at the bottom of this module, as it's not that important.

## Conclusion

In this section, we learned how to set up routing, fetch data, and write data in your Makerkit Next.js Supabase Turbo project. We built the following:

1. **Support Tickets List**: A page for team accounts where support agents can view new support tickets.
2. **Ticket Detail**: A page where support agents can view the details of a support ticket and reply to the ticket.

We also learn the following Next.js concepts:

1. Server Components and Client Components
2. The anatomy of a Next.js Page
3. Server Actions
4. Basics of Routing in Next.js

---

In the next section, we will be building the Javascript widget that gets embedded into a customer's website to create a new support ticket. We will be building the following:

1. **Creating a New Ticket**: A widget to create a new support ticket from any website page. This lets customers create a new support ticket from any page on the website.
2. **Real-time Updates**: We will be learning how to stream real-time updates in your Makerkit Next.js Supabase Turbo project.
3. **Building a Javascript Widget**: We will be learning how to create a Javascript widget with Makerkit Next.js Supabase Turbo.

We will get back to this page to add Real-time Updates - but first, we want customers to be able to create a new ticket from their website and send messages from there.

Hopefully you're excited to learn more! 🚀

## (Bonus) Adding the Assignees List

If you care enough about learning about adding the assignees list, let's go ahead and add it.

### Fetching the List of members of an account

We will be using the `useFetchMembers` hook to fetch the list of members of an account. The `useFetchMembers` hook is a custom hook that fetches the list of members of an account using the `get_account_members` RPC, which is a function built-in to Makerkit that fetches the list of members of an account.

```tsx
function useFetchMembers(accountSlug: string) {
  const supabase = useSupabase();
  const queryKey = ['accounts_memberships', accountSlug];

  const queryFn = async () => {
    const { data, error } = await supabase.rpc('get_account_members', {
      account_slug: accountSlug,
    });

    if (error) {
      throw error;
    }

    return data;
  };

  return useQuery({
    queryKey,
    queryFn,
  });
}
```

Now that we have the `useFetchMembers` hook, we can use it to fetch the list of members of an account in the `AssigneeSelect` component.

```tsx
// all the code previously added ...

function AssigneeSelect(props: {
  assignee: Tables<'tickets'>['assigned_to'];
  ticketId: string;
  accountSlug: string;
  disabled: boolean;
}) {
  const [pending, startTransition] = useTransition();

  const form = useForm({
    resolver: zodResolver(UpdateTicketAssigneeSchema),
    defaultValues: {
      assigneeId: props.assignee!,
      ticketId: props.ticketId,
    },
  });

  const membersQuery = useFetchMembers(props.accountSlug);
  const members = membersQuery.data ?? [];

  return (
    <Form {...form}>
      <FormField
        render={({ field }) => {
          return (
            <FormItem>
              <FormLabel>Assignee</FormLabel>

              <FormControl>
                <Select
                  value={form.getValues('assigneeId')}
                  disabled={pending || props.disabled || membersQuery.isPending}
                  onValueChange={(value) => {
                    form.setValue(field.name, value, {
                      shouldValidate: true,
                    });

                    void form.handleSubmit((value) => {
                      startTransition(async () => {
                        await updateTicketAssigneeAction(value);
                      });
                    })();
                  }}
                >
                  <SelectTrigger>
                    <SelectValue placeholder={'Choose an Assignee'} />
                  </SelectTrigger>

                  <SelectContent>
                    {members.map((member) => {
                      return (
                        <SelectItem key={member.id} value={member.id}>
                          {member.name || member.email}
                        </SelectItem>
                      );
                    })}
                  </SelectContent>
                </Select>
              </FormControl>

              <FormDescription>
                The person responsible for resolving the ticket.
              </FormDescription>
            </FormItem>
          );
        }}
        name={'assigneeId'}
      />
    </Form>
  );
}
```

We only need to pass the correct data to the `AssigneeSelect` component in the `TicketDetailsContainer` component.

```tsx
<AssigneeSelect
  disabled={!canUpdateTicket}
  assignee={ticket.assigned_to}
  ticketId={ticket.id}
  accountSlug={ticket.account_id.slug}
/>
```

And that's it! You can now assign the ticket to a member of the account.


