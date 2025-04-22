-----------------
FILE PATH: courses\nextjs-turbo\building-javascript-widget.mdoc

---
title: "Creating a Javascript Widget with Makerkit Next.js Supabase Turbo"
description: "Learn how to create a Javascript widget with Makerkit Next.js Supabase Turbo."
label: "Building a Javascript Widget"
order: 5
---

Hey, welcome to this module on creating a Javascript widget with Makerkit Next.js Supabase Turbo. In this module, we will explore how you can create a Javascript widget that can be embedded on any website.

## A recap of the project

If you have followed the previous modules, you should have a good understanding of how to build a Next.js application with Supabase and Turbo.

Let's summarize what we have done so far:

1. Set up the Makerkit's Next.js Supabase Turbo project
2. Added the DB schema
3. Displayed the Support tickets and messages

To have customers create new tickets and chat with the support team, we need to build a Javascript widget that can be embedded on any website.

## Introduction: Building a Javascript Widget

In this module, we will create a Javascript widget that can be embedded on any website. This widget will allow users to create a new ticket and chat with the support team in real-time.

1. **Customer**: On the customer's side, they will chat with a support team member. To store the conversation, we use the local storage of the user's browser. So, the conversation will be lost if the user clears their browser's local storage. This is a fairly normal behavior for chats with anonymous visitors.
2. **Support Agent**: On the support team's side, they will chat with the customer. The support team will see the conversation in real-time. This is achieved by using the Supabase Realtime feature, which will be done in the next module.

In this module, we will:
1. **Create a new Turborepo package** to manage the widget
2. **Build a new component** that allows customers to submit a new ticket and chat with the support team
3. **Bundle the Component with Rollup** as a Javascript widget
4. **Bundle the Tailwind CSS styles** with PostCSS and inject them into the widget
5. **Embed the widget** on any website that allows visitors to create a new ticket and chat with the support team
6. **Poll the server for new messages** and update the chat in real-time
7. **Create a external-facing API Routes** to handle the new ticket submission and message fetching
8. **Use the OpenAI API** to generate a meaningful title for the ticket based on the conversation

By the end of this lesson, we will have a fully functional Javascript widget that can be embedded on any website.

{% img src="/assets/courses/next-turbo/support-ticket-widget.webp" width="1068" height="1642" /%}

There's a lot to cover, so let's get started!

## High Level Architecture of the Widget

The widget will be a simple chat widget that allows users to create a new ticket and chat with the support team. The widget will be embedded on any website and will be displayed as a bubble on the bottom right corner of the screen.

When a user clicks on the bubble, the widget will open, and the user can start chatting with the support team. The widget will display the messages in real-time, and the user can send a new message by typing in the input field and pressing the send button.

In terms of architecture, the widget will consist of the following components:

1. **SupportTicketWidgetContainer**: This is the main component that will be rendered on the website. It contains the header, messages container, and input form.
2. **WidgetHeader**: This is the header of the widget. It contains the title and close button.
3. **WidgetContainer**: This is the container of the widget. It contains the header, messages container, and input form.
4. **WidgetBubble**: This is the bubble that will be displayed on the website. It contains the message icon and opens the widget when clicked.
5. **WidgetInput**: This is the input form of the widget. It contains the input field and send button.
6. **WidgetMessagesContainer**: This is the container that displays the messages. It contains the messages and author.

We also define two hooks: `useSubmitMessage` and `useFetchTicketMessages`. These hooks are used to submit a new message and fetch messages, respectively.

1. The `useSubmitMessage` hook is used to submit a new message. It sends a POST request to the API endpoint with the message content and ticket ID. If the request is successful, it returns the message object.
2. The `useFetchTicketMessages` hook is used to fetch messages. It sends a GET request to the API endpoint with the ticket ID. If the request is successful, it returns the messages array.

We will build these components and hooks in the next sections.

{% img src="/assets/courses/next-turbo/widget-architecture.webp" width="615" height="497" /%}

## Creating a New Turborepo Package

This is a fairly advanced topic, however, I will try to explain it in a simple way.

As you may have noticed, we have been using a monorepo structure to manage our Next.js, Supabase, and Turbo projects. This structure allows us to manage multiple packages in a single repository.

Since our component needs dependencies (such as Rollup and all its dependencies) and will be fairly decoupled from the main project, it makes sense to create a new package for it. We can still use the UI components from the main project, but the widget will be a standalone package.

To create a new package, we will use the `turbo gen package` command. This command will create a new package in the `packages` directory.

```bash
turbo gen package
```

Please follow the prompts to create a new package. You can name the package `ticket-widget`, and you can skip the dependencies prompt for now.

This will create a new package in the `packages` directory. You can now navigate to the `packages/ticket-widget` directory and start building your widget.

## Creating the New Ticket Widget

This part is going to involve some bundling magic with Rollup and its dependencies: you don't need to know how to do this, and it's not very useful to you unless you want to learn how to deploy Javascript widgets to your website.
Feel free to skip this section if you don't think it applies to you.

Let's begin by creating a new component in the `packages/ticket-widget` directory. This component will be a simple chat widget that allows users to create a new ticket and chat with the support team.

NB: we will write some code without too many dependencies (ex. React Query) to keep it simple and as lightweight as possible (since it will be embedded on other websites).

### Adding the Widget Context

To be able to share state between components, we need to create a context that will store the state of the widget. We use the `Context API`, which is a built-in state management solution in React. This allows us to avoid installing any additional dependencies.

```tsx {% title="packages/ticket-widget/src/components/context.tsx" %}
import { createContext } from 'react';

export const WidgetContext = createContext({
  isOpen: false,
  setIsOpen: (_: boolean) => {
    //
  },
  ticketId: '',
  setTicketId: (_: string) => {
    //
  },
});
```

To make it more explicit and easier to use, we create a hook that will allow us to access the state of the widget.

We call this hook `useWidgetState`, which will return the state of the widget.

```tsx {% title="packages/ticket-widget/src/components/use-widget-state.tsx" %}
import { useContext } from 'react';

import { WidgetContext } from './context';

export function useWidgetState() {
  return useContext(WidgetContext);
}
```

### Creating the Widget Container

Below is the full source code of the `SupportTicketWidgetContainer` component:

```tsx {% title="packages/ticket-widget/src/components/support-ticket-widget-container.tsx" %}
'use client';

import { useCallback, useEffect, useRef, useState } from 'react';

import { MessageCircle, Send, X } from 'lucide-react';

import { If } from '@kit/ui/if';
import {
  Tooltip,
  TooltipContent,
  TooltipProvider,
  TooltipTrigger,
} from '@kit/ui/tooltip';
import { cn } from '@kit/ui/utils';

import { useWidgetState } from '../lib/use-widget-state';

interface Message {
  id: string;
  ticketId: string;
  author: 'customer' | 'support';
  content: string;
  createdAt: string;
}

const API_URL = process.env.API_URL!;

export default function SupportTicketWidgetContainer(props: {
  accountId: string;
}) {
  const state = useWidgetState();
  const scrollingDiv = useRef<HTMLDivElement>(null);

  const { messages, appendMessage } = useFetchTicketMessages({
    ticketId: state.ticketId,
    isOpen: state.isOpen,
  });

  const scrollToBottom = () => {
    scrollingDiv.current?.scrollTo({
      top: scrollingDiv.current.scrollHeight,
    });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  return (
    <>
      <If condition={state.isOpen}>
        <WidgetContainer>
          <div className={'flex h-full flex-1 flex-col'}>
            <WidgetHeader />

            <div
              ref={(div) => {
                scrollingDiv.current = div as HTMLDivElement;
              }}
              className={'flex flex-1 flex-col overflow-y-auto p-4'}
            >
              <WidgetMessagesContainer messages={messages} />
            </div>

            <WidgetInput
              accountId={props.accountId}
              ticketId={state.ticketId}
              onSubmit={(message) => {
                state.setTicketId(message.ticketId);
                appendMessage(message);
              }}
            />
          </div>
        </WidgetContainer>
      </If>

      <WidgetBubble />
    </>
  );
}

function WidgetHeader() {
  const { setIsOpen } = useWidgetState();

  return (
    <div
      className={
        'flex items-center justify-between border-b px-4 py-3 md:rounded-t-xl'
      }
    >
      <div className={'text-foreground flex flex-col'}>
        <span className={'font-semibold'}>Ask Support</span>
      </div>

      <div className={'flex items-center space-x-4'}>
        <TooltipProvider>
          <Tooltip>
            <TooltipTrigger asChild>
              <button
                onClick={() => {
                  setIsOpen(false);
                }}
              >
                <X className={'text-foreground h-4 dark:hover:text-white'} />
              </button>
            </TooltipTrigger>

            <TooltipContent>Close</TooltipContent>
          </Tooltip>
        </TooltipProvider>
      </div>
    </div>
  );
}

function WidgetContainer(props: React.PropsWithChildren) {
  return (
    <div
      className={cn(
        'animate-in fade-in slide-in-from-bottom-24 fixed z-50 duration-200' +
          ' bg-background font-sans md:rounded-lg' +
          ' h-[60vh] w-full md:w-[40vw] xl:w-[26vw]' +
          ' zoom-in-90 bottom-0 border shadow-2xl md:bottom-36 md:right-8',
      )}
    >
      {props.children}
    </div>
  );
}

function WidgetBubble() {
  const { isOpen, setIsOpen } = useWidgetState();

  const className = cn('bottom-8 md:bottom-16 md:right-8', {
    'hidden md:flex': isOpen,
  });

  const iconClassName = 'w-8 h-8 animate-in fade-in zoom-in';

  const Icon = isOpen ? (
    <X className={iconClassName} />
  ) : (
    <MessageCircle className={iconClassName} />
  );

  return (
    <button
      className={cn(
        'animate-out text-primary-foreground bg-primary h-16 w-16 rounded-full' +
          ' animate-in zoom-in slide-in-from-bottom-16 fixed flex items-center justify-center' +
          ' hover:opacity/90 transition-all hover:shadow-xl' +
          ' z-50 duration-500 hover:-translate-y-1 hover:scale-105',
        className,
      )}
      onClick={() => setIsOpen(!isOpen)}
    >
      {Icon}
    </button>
  );
}

function WidgetInput(props: {
  accountId: string;
  ticketId: string | undefined;
  onSubmit: (message: Message) => void;
}) {
  const submitMessage = useSubmitMessage();

  const onSubmit: React.FormEventHandler<HTMLFormElement> = useCallback(
    async (e) => {
      e.preventDefault();

      const form = e.currentTarget;
      const element = form.elements.namedItem('message') as HTMLInputElement;
      const value = element.value.trim();

      if (!value) {
        return;
      }

      element.value = '';

      const message = await submitMessage.mutateAsync({
        accountId: props.accountId,
        ticketId: props.ticketId,
        message: value,
      });

      props.onSubmit(message);
    },
    [props],
  );

  return (
    <form className={'mt-auto'} onSubmit={onSubmit}>
      <div className={'relative flex'}>
        <input
          disabled={submitMessage.loading}
          autoComplete={'off'}
          required
          name={'message'}
          className={
            'text-muted-foreground h-14 p-4' +
            ' w-full rounded-bl-xl rounded-br-xl outline-none' +
            ' resize-none border-t text-sm transition-colors' +
            ' bg-background focus:text-secondary-foreground pr-8'
          }
          placeholder="Type your message..."
        />

        <button
          type={'submit'}
          className={'absolute right-4 top-4 bg-transparent'}
        >
          <Send className={'text-muted-foreground h-6'} />
        </button>
      </div>
    </form>
  );
}

function WidgetMessagesContainer(props: {
  messages: Message[]
}) {
  if (!props.messages.length) {
    return (
      <div className={'text-muted-foreground text-center'}>
        Please send a message to start a conversation
      </div>
    );
  }

  return (
    <div className={'flex flex-col space-y-5'}>
      {props.messages.map((message) => {
        const name = message.author === 'customer' ? 'You' : 'Support';

        let className = 'p-3 flex flex-col space-y-1 border rounded-lg';

        if (message.author === 'customer') {
          className += ' bg-primary/5 border-primary/10';
        } else {
          className += '';
        }

        return (
          <div className={'flex flex-col space-y-2'} key={message.id}>
            <div className={'px-3'}>
              <b className={'text-sm font-semibold'}>{name}</b>
            </div>

            <div className={className}>
              <div className={'block max-w-full break-words text-sm'}>
                {message.content}
              </div>
            </div>
          </div>
        );
      })}
    </div>
  );
}

function useSubmitMessage() {
  const [state, setState] = useState<{
    loading: boolean;
    error: Error | null;
    data: { ticketId: string } | null;
  }>({
    loading: false,
    error: null,
    data: null,
  });

  const mutateAsync = useCallback(
    async (props: {
      accountId: string;
      ticketId: string | undefined;
      message: string;
    }) => {
      setState(state => ({ ...state, loading: true, error: null }));

      try {
        const response = await fetch(API_URL, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            ticketId: props.ticketId ?? undefined,
            accountId: props.accountId,
            message: props.message,
          }),
        });

        if (!response.ok) {
          throw new Error('Failed to submit message');
        }

        const result = (await response.json()) as Message;

        setState({
          loading: false,
          error: null,
          data: result,
        });

        return result;
      } catch (error) {
        setState({
          loading: false,
          error: new Error('Failed to parse response'),
          data: null,
        });

        throw new Error('Failed to parse response');
      }
    },
    [],
  );

  return {
    ...state,
    mutateAsync,
  };
}

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

  const messages = state.messages;

  const lastMessage = messages.reduce<Message | undefined>((acc, curr) => {
    if (!acc) return;

    return acc.createdAt > curr.createdAt ? acc : curr;
  }, messages[0]);

  const lastCreatedAt = lastMessage?.createdAt;

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
      setState(state => ({ ...state, loading: true, error: null }));

      const timestamp = lastCreatedAt ? new Date(lastCreatedAt).toISOString() : '';

      fetch(`${API_URL}/messages?ticketId=${ticketId}&lastCreatedAt=${timestamp}`)
        .then((response) => {
          if (!response.ok) {
            throw new Error('Failed to fetch messages');
          }

          return response.json();
        })
        .then((messages) => {
          const newMessages = messages.filter(
            (message: Message) =>
              !state.messages.some((m) => m.id === message.id),
          );

          setState(state => ({
            loading: false,
            error: null,
            messages: [...state.messages, ...newMessages],
          }));
        })
        .catch((error) => {
          setState(state => ({
            ...state,
            loading: false,
            error,
          }));
        });
    }

    // Fetch messages on mount
    fetchMessages();

    // Fetch messages every 10 seconds
    const interval = setInterval(() => {
      fetchMessages(lastCreatedAt);
    }, 10_000);

    return () => clearInterval(interval);
  }, [ticketId, isOpen, lastCreatedAt]);

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

### Creating a new Ticket

The components is fairly simple, and works as follows:

1. **Load current Ticket ID**: We check if we have an existing ticket ID in Local Storage. If we do, we fetch the messages for that ticket. Otherwise, we send a message without a ticket ID, which the API will handle as a new ticket.
2. **Load Messages**: If we have a ticket ID, we fetch the messages for that ticket ID.

### Create a new Service to handle the logic

We will create a new service to handle the logic for submitting a new message and fetching messages. This service will send requests to the API endpoints to create a new ticket and message.

Services make it simpler to manage logic within Server Actions, and ensure there is a separate layer between the front-facing API and the inner logic.

Let's go on and create the service at `apps/web/app/api/ticket/_lib/server/customer-ticket.service.ts`.

```tsx title="apps/web/app/api/ticket/_lib/server/customer-ticket.service.ts"
import 'server-only';

import { SupabaseClient } from '@supabase/supabase-js';

import { getLogger } from '@kit/shared/logger';

import { Database } from '~/lib/database.types';

export function createCustomerTicketService(client: SupabaseClient<Database>) {
  return new CustomerTicketService(client);
}

class CustomerTicketService {
  constructor(private readonly client: SupabaseClient<Database>) {}

  async createTicket(params: { accountId: string; message: string }) {
    const logger = await getLogger();

    logger.info(params, 'Creating ticket...');

    const ticket = await this.client
      .from('tickets')
      .insert({
        account_id: params.accountId,
        title: 'New ticket',
      })
      .select('id')
      .single();

    if (ticket.error) {
      logger.error({ error: ticket.error }, 'Error creating ticket');

      throw ticket.error;
    }

    // create message
    const { data, error } = await this.client
      .from('messages')
      .insert({
        ticket_id: ticket.data.id,
        content: params.message,
        author: 'customer',
      })
      .select(
        `
          ticketId: ticket_id,
          content,
          author,
          createdAt: created_at
        `,
      )
      .single();

    if (error) {
      logger.error({ error }, 'Error creating message');

      throw error;
    }

    return data;
  }

  async createMessage(params: { ticketId: string; message: string }) {
    const logger = await getLogger();

    logger.info(params, 'Creating message...');

    const { data, error } = await this.client
      .from('messages')
      .insert({
        ticket_id: params.ticketId,
        content: params.message,
        author: 'customer',
      })
      .select(
        `
          ticketId: ticket_id,
          content,
          author,
          createdAt: created_at
        `,
      )
      .single();

    if (error) {
      logger.error(
        { error, ticketId: params.ticketId },
        'Error creating message',
      );

      throw error;
    }

    logger.info(data, 'Message successfully created');

    return data;
  }
}
```

In the service above, we create a new `CustomerTicketService` class that contains two methods: `createTicket` and `createMessage`.

1. **Create Ticket**: The `createTicket` method creates a new ticket and message. It inserts a new ticket into the `tickets` table and a new message into the `messages` table. It returns the new message object.
2. **Create Message**: The `createMessage` method creates a new message for an existing ticket. It inserts a new message into the `messages` table and returns the new message object.

Depending on whether it's the initial message or a follow-up message, we use the `createTicket` or `createMessage` method.

NB: we are currently passing a static title for the ticket: we will use the OpenAI API to generate a more meaningful title at end of this section. The current focus is to understand the flow of the widget and how to build an external-facing API to handle the new ticket submission.

Now that we have the service, let's create the API Handler to handle the new ticket submission.

### Sending a new Message

When a user sends a new message, we send a POST request to the API with the message content and ticket ID. If the request is successful, we append the message to the messages array.

Let's create the API Handler to handle the new ticket submission.

We will create an [API Route Handler](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) in our application that is responsible for handling the new ticket submission. This handler will receive the ticket ID, account ID, and message content, and save the message to the database.

```tsx {% title="apps/web/app/api/ticket/route.ts" %}
import { NextResponse } from 'next/server';

import { literal, z } from 'zod';

import { enhanceRouteHandler } from '@kit/next/routes';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

import { createCustomerTicketService } from './_lib/server/customer-ticket.service';

const NewMessageSchema = z.object({
  message: z.string().min(1).max(5000),
  accountId: z.string().uuid(),
  ticketId: z.string().uuid().or(literal('')),
});

export const OPTIONS = () => {
  return NextResponse.json(
    {},
    {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST',
        'Access-Control-Allow-Headers': '*',
      },
    },
  );
};

export const POST = enhanceRouteHandler(
  async ({ body }) => {
    const client = getSupabaseServerAdminClient();

    const { message, ticketId, accountId } = body;
    const service = createCustomerTicketService(client);

    const newMessage = ticketId
      ? await service.createMessage({
          ticketId,
          message,
        })
      : await service.createTicket({
          accountId,
          message,
        });

    return NextResponse.json(newMessage, {
      headers: {
        'Access-Control-Allow-Origin': '*',
      },
    });
  },
  {
    schema: NewMessageSchema,
    auth: false,
  },
);
```

In the above, we use the utility `enhanceRouteHandler`, which is similar to `enhanceAction` but for API routes. This utility allows us to define a schema for the request body and set the authentication requirement.

In the Route above, we split the code in two branches:

1. **Existing Ticket**: If the ticket ID is provided, we create a new message for the existing ticket.
2. **New Ticket**: If the ticket ID is not provided, we create a new ticket and a new message.

We then return the new message as a JSON response, which will be displayed in the widget.

#### Handle CORS requests in Next.js App Router

As an external-facing API, we need to handle CORS. We do this by setting the `Access-Control-Allow-Origin` header to allow requests from any origin.

We set up an `OPTIONS` HTTP route handler to handle the preflight request. This route handler returns an empty JSON response with the appropriate CORS headers.

We also set the `Access-Control-Allow-Methods` and `Access-Control-Allow-Headers` headers to allow POST requests with any method and any headers.

This is extremely important to have your API accessible from any website **specifically for routes that need to be accessed from the browser**.

### Fetching and displaying the Messages

To fetch the messages, we use the `useFetchTicketMessages` hook. This hook sends a GET request to the API endpoint with the ticket ID and fetches the messages for that ticket.

Let's enrich the service with a method to fetch the messages for a ticket ID.

```tsx {% title="apps/web/app/api/ticket/_lib/server/customer-ticket.service.ts" %}
class CustomerTicketService {
  constructor(private readonly client: SupabaseClient<Database>) {}

  async getTicketMessages(params: {
    ticketId: string;
    lastCreatedAt?: string;
  }) {
    let query = this.client
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
      .eq('ticket_id', params.ticketId)
      .order('created_at', { ascending: true });

    if (params.lastCreatedAt) {
      query = query.gt('created_at', params.lastCreatedAt);
    }

    const { data, error } = await query;

    if (error) {
      throw error;
    }

    return data;
  }

  // other methods...
}
```

In the service above, we create a new `getTicketMessages` method that fetches the messages for a ticket ID. We use the `eq` method to filter the messages by the ticket ID and the `order` method to order the messages by the created date.

Let's define the API Route Handler to handle the messages fetching.

```tsx {% title="apps/web/app/api/ticket/messages/route.ts" %}
import { NextResponse } from 'next/server';

import { z } from 'zod';

import { enhanceRouteHandler } from '@kit/next/routes';
import { getSupabaseServerAdminClient } from '@kit/supabase/server-admin-client';

import { createCustomerTicketService } from '../_lib/server/customer-ticket.service';

const GetTicketMessagesSchema = z.object({
  ticketId: z.string().uuid(),
  lastCreatedAt: z
    .string()
    .or(z.literal(''))
    .transform((value) => {
      if (value === 'undefined') {
        return;
      }

      return value;
    }),
});

export const OPTIONS = () => {
  return new Response(null, {
    status: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET',
      'Access-Control-Allow-Headers': '*',
    },
  });
};

export const GET = enhanceRouteHandler(
  async ({ request }) => {
    const client = getSupabaseServerAdminClient();

    const searchParams = new URL(request.url).searchParams;

    const params = GetTicketMessagesSchema.parse({
      ticketId: searchParams.get('ticketId') ?? '',
      lastCreatedAt: searchParams.get('lastCreatedAt') ?? '',
    });

    const service = createCustomerTicketService(client);
    const messages = await service.getTicketMessages(params);

    return NextResponse.json(messages, {
      headers: {
        'Access-Control-Allow-Origin': '*',
      },
    });
  },
  {
    auth: false,
    schema: undefined,
  },
);
```

Similar to the previous Route Handler, we use the `enhanceRouteHandler` utility to define the schema for the request URL and set the authentication requirement.

In the Route Handler defined above, we fetch the messages for the ticket ID provided in the query string. We then return the messages as a JSON response, which will be displayed in the widget.

In this instance we don't paginate the messages, but it's a good idea to do so if you expect a large number of messages.

For more information about writing API Route Handlers in Next.js, you can check the [official documentation](https://nextjs.org/docs/app/building-your-application/routing/route-handlers).

### Embedding the Widget

Let's continue with building the widget.

Let's create a component wrapper that takes care of setting the Context Provider and the Widget itself.

First, we create a global state manager using React Context. This context will store the state of the widget, such as whether it is open, the ticket ID, and the functions to set the state.

```tsx {% title="packages/ticket-widget/src/lib/context.ts" %}
import { createContext } from 'react';

export const WidgetContext = createContext({
  isOpen: false,
  setIsOpen: (_: boolean) => {
    //
  },
  ticketId: '',
  setTicketId: (_: string) => {
    //
  },
});
```

We then create the `WidgetContainer` component that wraps the `SupportTicketWidgetContainer` component with the `WidgetContext.Provider`.

```tsx {% title="packages/ticket-widget/src/components/index.tsx" %}
'use client';

import { useEffect, useState } from 'react';

import { WidgetContext } from '../lib/context';
import SupportTicketWidgetContainer from './support-ticket-widget-container';

export default function WidgetContainer(props: { accountId: string }) {
  const [mounted, setMounted] = useState(false);
  const [isOpen, setIsOpen] = useState(false);
  const [ticketId, setTicketId] = useState(getTicketIdFromLocalStorage());

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return null;
  }

  const context = {
    isOpen,
    setIsOpen,
    ticketId,
    setTicketId: (ticketId: string) => {
      localStorage.setItem('ticketId', ticketId);
      setTicketId(ticketId);
    },
  };

  return (
    <WidgetContext.Provider value={context}>
      <SupportTicketWidgetContainer accountId={props.accountId} />
    </WidgetContext.Provider>
  );
}

function getTicketIdFromLocalStorage() {
  return localStorage.getItem('ticketId') ?? '';
}
```

Finally, we need the functions to embed the widget on any website by rendering the React component.

Let's create the `index.css` style that will be used to style the widget.

```css {% title="packages/ticket-widget/src/index.css" %}
@import "../../../apps/web/styles/globals.css";

:host {
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI',
    Roboto, 'Helvetica Neue', Arial;
    --background: 0 0% 100%;
    --foreground: 224 71.4% 4.1%;
    --card: 0 0% 100%;
    --card-foreground: 224 71.4% 4.1%;
    --popover: 0 0% 100%;
    --popover-foreground: 224 71.4% 4.1%;
    --primary: 262.1 83.3% 57.8%;
    --primary-foreground: 210 20% 98%;
    --secondary: 220 14.3% 95.9%;
    --secondary-foreground: 220.9 39.3% 11%;
    --muted: 220 14.3% 95.9%;
    --muted-foreground: 220 8.9% 46.1%;
    --accent: 220 14.3% 95.9%;
    --accent-foreground: 220.9 39.3% 11%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 20% 98%;
    --border: 220 13% 91%;
    --input: 220 13% 91%;
    --ring: 262.1 83.3% 57.8%;
    --radius: 0.75rem;
}
```

NB: you can change the variables above with any other value you want.

Let's add a new file to create an Iframe component that will be used to render the widget.

```tsx {% title="packages/ticket-widget/src/components/iframe.tsx" %}
'use client';

import { useState } from 'react';

import { createPortal } from 'react-dom';

import { cn } from '@kit/ui/utils';

export const IFrame: React.FC<
  React.IframeHTMLAttributes<unknown> & {
    setInnerRef?: (ref: HTMLIFrameElement | undefined) => void;
    style?: React.CSSProperties;
    className?: string;
  }
> = ({ children, setInnerRef, style = {}, className }) => {
  const [ref, setRef] = useState<HTMLIFrameElement | null>();
  const doc = ref?.contentWindow?.document as Document;
  const mountNode = doc?.body;

  return (
    <iframe
      className={cn(className)}
      style={{
        all: 'initial',
        position: 'fixed',
        width: '100%',
        height: '100%',
        border: 0,
        zIndex: 1000,
        ...(style ?? {}),
      }}
      ref={(ref) => {
        if (ref) {
          setRef(ref);

          if (setInnerRef) {
            setInnerRef(ref);
          }
        }
      }}
    >
      {mountNode ? createPortal(children, mountNode) : null}
    </iframe>
  );
};
```

Finally, we create the `index.tsx` file that will render the `WidgetContainer` component and inject the CSS styles.

```tsx {% title="packages/ticket-widget/src/index.tsx" %}
import { Suspense } from 'react';

import { hydrateRoot } from 'react-dom/client';

import Widget from './components';
import { IFrame } from './components/iframe';
import styles from './index.css';

const WIDGET_NAME = process.env.WIDGET_NAME;

// initialize the widget
initializeWidget();

function initializeWidget() {
  if (document.readyState !== 'loading') {
    onReady();
  } else {
    document.addEventListener('DOMContentLoaded', onReady);
  }
}

function onReady() {
  try {
    const element = document.createElement('div');
    const accountId = getAccountId();

    const component = (
      <IFrame>
        <Suspense fallback={null}>
          <style suppressHydrationWarning>{styles}</style>
          <Widget accountId={accountId} />
        </Suspense>
      </IFrame>
    );

    hydrateRoot(element, component);

    document.body.appendChild(element);
  } catch (error) {
    console.warn(`Could not initialize Widget`);
    console.warn(error);
  }
}

function getAccountId() {
  const script = getCurrentScript();

  if (!script) {
    throw new Error('Script not found');
  }

  const accountId = script.getAttribute('data-account');

  if (!accountId) {
    throw new Error('Missing data-account-id attribute');
  }

  return accountId;
}

function getCurrentScript() {
  const currentScript = document.currentScript;

  if (!WIDGET_NAME) {
    throw new Error('Missing WIDGET_NAME environment variable');
  }

  if (currentScript?.getAttribute('src')?.includes(WIDGET_NAME)) {
    return currentScript as HTMLScriptElement;
  }

  return Array.from(document.scripts).find((item) => {
    return item.src.includes(WIDGET_NAME);
  });
}
```

The script above will render the `WidgetContainer` component and inject the CSS styles into the shadow DOM.

#### How do we know the Account ID?

The Account ID is passed as a data attribute in the script tag that loads the widget. This allows us to identify the account that the widget belongs to.

```html
<script
  data-account="account-id"
  src="https://widget-url.com/widget.js"
></script>
```

Basically, the clients will include the script tag in their website, and the script will load the widget and pass the account.

### Rollup Configuration

We will use Rollup to bundle the widget into a single Javascript file. This file will contain the React component, CSS styles, and any other assets required by the widget.

A requirement here is to install the rollup dependencies in the `ticket-widget` package.

For simplicity, here is the `package.json` file that you can replace:

```json {% title="packages/ticket-widget/package.json" %}
{
  "name": "@kit/ticket-widget",
  "private": true,
  "version": "0.1.0",
  "exports": {
    ".": "./index.ts"
  },
  "typesVersions": {
    "*": {
      "*": [
        "src/*"
      ]
    }
  },
  "license": "MIT",
  "scripts": {
    "clean": "rm -rf .turbo node_modules",
    "lint": "eslint .",
    "format": "prettier --check \"**/*.{mjs,ts,md,json}\"",
    "typecheck": "tsc --noEmit",
    "build": "rollup -c ./rollup.config.mjs",
    "build:production": "rollup -c ./rollup.config.mjs --environment=production",
    "serve": "npx http-server ./ --cors -p 3333"
  },
  "dependencies": {},
  "devDependencies": {
    "@babel/preset-react": "7.24.7",
    "@babel/preset-typescript": "^7.24.1",
    "@kit/eslint-config": "workspace:*",
    "@kit/prettier-config": "workspace:*",
    "@kit/tsconfig": "workspace:*",
    "@kit/ui": "workspace:*",
    "@rollup/plugin-babel": "^6.0.4",
    "@rollup/plugin-commonjs": "^26.0.1",
    "@rollup/plugin-node-resolve": "^15.2.3",
    "@rollup/plugin-replace": "^5.0.5",
    "@rollup/plugin-terser": "latest",
    "@rollup/plugin-typescript": "^11.1.6",
    "@types/node": "^20.14.6",
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "dotenv": "^16.3.1",
    "eslint": "^8.57.0",
    "lucide-react": "^0.399.0",
    "prettier": "^3.3.2",
    "react": "18.3.1",
    "react-dom": "18.3.1",
    "rollup": "^4.9.6",
    "rollup-plugin-inject-process-env": "latest",
    "rollup-plugin-polyfill-node": "^0.13.0",
    "rollup-plugin-postcss": "latest",
    "rollup-plugin-tsconfig-paths": "^1.5.2",
    "rollup-plugin-visualizer": "latest",
    "tailwindcss": "3.4.4",
    "tailwindcss-animate": "^1.0.7",
    "tslib": "^2.8.1",
    "typescript": "^5.5.2"
  },
  "prettier": "@kit/prettier-config"
}
```

We will also need to create a `rollup.config.mjs` file in the `ticket-widget` package. This file will contain the Rollup configuration to bundle the widget.

```js {% title="packages/ticket-widget/rollup.config.mjs" %}
import babel from '@rollup/plugin-babel';
import commonjs from '@rollup/plugin-commonjs';
import nodeResolve from '@rollup/plugin-node-resolve';
import replace from '@rollup/plugin-replace';
import terser from '@rollup/plugin-terser';
import typescript from '@rollup/plugin-typescript';
import { config } from 'dotenv';
import { parseArgs } from 'node:util';
import injectProcessEnv from 'rollup-plugin-inject-process-env';
import nodePolyfills from 'rollup-plugin-polyfill-node';
import postcss from 'rollup-plugin-postcss';
import tsConfigPaths from 'rollup-plugin-tsconfig-paths';
import { visualizer } from 'rollup-plugin-visualizer';

const args = parseArgs({
  options: {
    environment: {
      type: 'string',
      short: 'e',
      default: 'development',
    },
    configuration: {
      type: 'string',
      short: 'c',
    },
  },
});

const env = args.values.environment;
const production = env === 'production';
let environmentVariablesPath = './.env';

console.log(`Building widget for ${env} environment...`);

if (production) {
  environmentVariablesPath += '.production';
}

const ENV_VARIABLES = config({
  path: environmentVariablesPath,
}).parsed;

const fileName = ENV_VARIABLES.WIDGET_NAME || 'makerdesk-widget.js';

export default {
  input: './src/index.tsx',
  output: {
    file: `dist/${fileName}`,
    format: 'iife',
    sourcemap: false,
    inlineDynamicImports: true,
    globals: {
      'react/jsx-runtime': 'jsxRuntime',
      'react-dom/client': 'ReactDOM',
      react: 'React',
    },
  },
  plugins: [
    tsConfigPaths({
      tsConfigPath: './tsconfig.json',
    }),
    replace({ preventAssignment: true }),
    typescript({
      tsconfig: './tsconfig.json',
    }),
    nodeResolve({
      extensions: ['.tsx', '.ts', '.json', '.js', '.jsx', '.mjs'],
      browser: true,
      dedupe: ['react', 'react-dom'],
    }),
    babel({
      babelHelpers: 'bundled',
      presets: [
        '@babel/preset-typescript',
        [
          '@babel/preset-react',
          {
            runtime: 'automatic',
            targets: '>0.1%, not dead, not op_mini all',
          },
        ],
      ],
      extensions: ['.js', '.jsx', '.ts', '.tsx', '.mjs'],
    }),
    postcss({
      extensions: ['.css'],
      minimize: false,
      extract: false,
      inject: false
    }),
    commonjs(),
    nodePolyfills({
      exclude: ['crypto'],
    }),
    injectProcessEnv(ENV_VARIABLES),
    terser({
      ecma: 2020,
      mangle: { toplevel: true },
      compress: {
        module: true,
        toplevel: true,
        unsafe_arrows: true,
        drop_console: true,
        drop_debugger: true,
      },
      output: { quote_style: 1 },
    }),
    visualizer(),
  ],
};
```

We have to pass certain environment variables to the Rollup configuration. These variables will be used to configure the widget, such as the widget name and CSS URL.

You can build two different configurations:
1. Development: `rollup -c ./rollup.config.mjs`
2. Production: `rollup -c ./rollup.config.mjs --environment=production`

In the case of the production environment, we will use the `.env.production` file to load the environment variables. This file will contain the widget name and CSS URL.

```bash {% title=".env.production" %}
WIDGET_NAME=makerdesk-widget.js
API_URL=https://api-url.com/api/ticket
```

In the development environment, we will use the `.env` file to load the environment variables. This file will contain the widget name and CSS URL.

```bash {% title=".env" %}
WIDGET_NAME=makerdesk-widget.js
API_URL=http://localhost:3000/api/ticket
```

To build the widget, run the following command:

```bash
rollup -c ./rollup.config.mjs
```

This will bundle the widget into a single Javascript file and output it to the `dist` directory. Finally, you can place the widget on any website by including the script tag that loads the widget.

```html
<script
  data-account="account-id"
  src="https://widget-url.com/makerdesk-widget.js"
></script>
```

This script tag will load the widget on the website and pass the account ID to the widget.

## Testing the Widget

Now we can add two commands to the main package.json that we can conveniently use to build and serve the widget.

```json {% title="package.json" %}
{
  "scripts": {
    "widget:build": "pnpm --filter '@kit/ticket-widget' build",
    "widget:serve": "pnpm --filter '@kit/ticket-widget' serve"
  }
}
```

To build the widget, run the following command:

```bash
pnpm run widget:build
```

Instead, to serve the widget, run the following command:

```bash
pnpm run widget:serve
```

The command above will start a server that allows us to test the widget locally. Open the URL outputted in the terminal to see the widget in action.

### Using OpenAI to generate Ticket Titles

With the bulk of work done, we can now focus on improving the user experience by generating meaningful ticket titles using OpenAI.

#### Install the OpenAI SDK

First, we want to install the library `openai`, which allows us to interact with the OpenAI API.

```bash
pnpm add --filter web openai
```

The command above will install the `openai` library in the `web` package. We will use the package in our API route (which is in the `web` package), therefore we only need to install it in the `web` package.

#### Using the OpenAI API

Let's now go back and update the `customer-ticket.service.ts` file to use the OpenAI API to generate a ticket title.

```tsx {% title="apps/web/app/api/ticket/_lib/server/customer-ticket.service.ts" %}
import { OpenAI } from 'openai';

class CustomerTicketService {
  async generateTicketTitle(message: string) {
    try {
      const openAI = new OpenAI();

      const response = await openAI.completions.create({
        model: 'gpt-3.5-turbo',
        prompt: `Generate a short (under 10 words) title for a support ticket based on the following message: "${message}"`,
        max_tokens: 10,
      });

      return response.choices[0]?.text ?? 'New ticket';
    } catch (error) {
      console.warn(`Failed to generate ticket title:`, error);

      return 'New ticket';
    }
  }

  // other methods...
}
```

In the service above, we create a new `generateTicketTitle` method that generates a ticket title using the OpenAI API. We use the `openai` library to interact with the OpenAI API and send a request to the `completions.create` endpoint.

We use `gpt-3.5-turbo` as the model and provide a prompt with the message content, which is fast and cheap, generally suitable for the simple use-case.

Let's now call this function in the `createTicket` method to generate a ticket title.

```tsx {% title="apps/web/app/api/ticket/_lib/server/customer-ticket.service.ts" %}
class CustomerTicketService {
  // other methods...

  async createTicket(params: { accountId: string; message: string }) {
    const logger = await getLogger();

    logger.info(params, 'Creating ticket...');

    const title = await this.generateTicketTitle(params.message);

    const ticket = await this.client
      .from('tickets')
      .insert({
        account_id: params.accountId,
        title,
      })
      .select('id')
      .single();

    if (ticket.error) {
      logger.error({ error: ticket.error }, 'Error creating ticket');

      throw ticket.error;
    }

    // create message
    const { data, error } = await this.client
      .from('messages')
      .insert({
        ticket_id: ticket.data.id,
        content: params.message,
        author: 'customer',
      })
      .select(
        `
          ticketId: ticket_id,
          content,
          author,
          createdAt: created_at
        `,
      )
      .single();

    if (error) {
      logger.error({ error }, 'Error creating message');

      throw error;
    }

    return data;
  }
}
````

To have the API working, you need to set an API Key in the environment variables.

As we've said before, these must be added exclusively to the `.env.local` file, which is never committed to Git.

```bash {% title="apps/web/.env.local" %}
OPENAI_API_KEY=sk-...
```

The OpenAI client will use the API key to authenticate with the OpenAI API and generate the ticket title.

You can now retry submitting a new ticket and see the title generated by OpenAI. You may have to clear the storage or simply run the following in your Console:

```js
localStorage.removeItem('ticketId')
```

Now, refresh the page and submit a new ticket. You should see the title generated by OpenAI.

### Conclusion

In this module, we created a new Turborepo package for the widget and built the widget using Rollup. We embedded the widget on any website and allowed visitors to create a new ticket and chat with the support team. We also polled the server for new messages and updated the chat in real-time. Finally, we created an external-facing API Route to handle the new ticket submission.

In the next module, we add Real-time updates to support ticket in the agent's side using Supabase Realtime. Additionally, we will add polling to the widget to fetch new messages in real-time.

