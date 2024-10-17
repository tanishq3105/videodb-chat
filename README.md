[![NPM version][npm-shield]][npm-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![Website][website-shield]][website-url]
[![Discord][discord-shield]][discord-url]

<br />

<p align="center">
  <a href="https://videodb.io/">
    <img src="https://codaio.imgix.net/docs/_s5lUnUCIU/blobs/bl-RgjcFrrJjj/d3cbc44f8584ecd42f2a97d981a144dce6a66d83ddd5864f723b7808c7d1dfbc25034f2f25e1b2188e78f78f37bcb79d3c34ca937cbb08ca8b3da1526c29da9a897ab38eb39d084fd715028b7cc60eb595c68ecfa6fa0bb125ec2b09da65664a4f172c2f" alt="Logo" width="300" height="">
  </a>

  <h3 align="center">VideoDB Chat</h3>

  <p align="center">
    Chat UI Components for <a href="https://github.com/video-db/spielberg">  Spielberg</a>
    <br />
    <a href="https://stackblitz.com/edit/vitejs-vite-zyq2no?file=src%2FApp.vue"><strong>View Demo »</strong></a>
    <br />
    <br />
    <a href="https://github.com/video-db/videodb-chat/issues">Report Bug</a>
    ·
    <a href="https://github.com/video-db/videodb-chat/issues">Request Feature</a>
  </p>
</p>

# 💬 VideoDB Chat

These are Chat UI components to use with [Spielberg](https://github.com/video-db/spielberg).

# 🚀 Quickstart

### Installation

```
npm install @videodb/chat-vue
```

### Usage

Import the necessary components and styles.

If you are using [Spielberg](https://github.com/video-db/spielberg), make sure it's backend is running and the socket url is correctly passed in `chat-hook-config`

```html
<script setup>
  import { ChatInterface } from "@videodb/chat-vue";
  import "@videodb/chat-vue/dist/style.css";

  const socketUrl = "http://127.0.0.1:8000/chat";
  const httpUrl = "http://127.0.0.1:8000";
</script>

<template>
  <div>
    <ChatInterface
      :chat-hook-config="{
        socketUrl: socketUrl,
        httpUrl: httpUrl,
        debug: true,
      }"
    />
  </div>
</template>
```

# 📖 Application Flow

The `ChatInterface` component is composed of two primary sub-components:

- `<ChatMessageContainer/>`
- `<ChatMessageInput/>`

### `<ChatMessageContainer/>`

This component displays both past and current conversations within a session. Conversations are stored in the `conversations` variable, a reactive object exported by the chat hook. This variable updates in real-time as the conversation progresses.

Each conversation consists of input and output messages, which are rendered as `<ChatMessage/>` components. Output messages can contain various content types such as `text`, `video`, or `image`. These are rendered by their respective message handlers.

> ℹ️ **Note:** To add support for additional content types, please refer to the [Custom Message Handler](#custom-message-handler) section.

### `<ChatMessageInput/>`

When a user sends a message, this component calls the `addMessage()` function, which in turn invokes the `addMessage` function of the chat hook.

### Default Chat Hook

The default chat hook is `videoDBChatHook`, which integrates with [Spielberg](https://github.com/video-db/spielberg).

> ℹ️ **Note:** To configure your own chat hook, please refer to the [Custom ChatHook](#custom-chathook) section.

# 🧑‍💻 Additional Components

_This package includes other UI components that enhance the chat experience_

### `<Sidebar/>`

This component facilitates navigation between different sessions and collections. It can be used to switch between various conversations or collections.

> You can customize the branding of the sidebar by passing a `sidebarConfig` prop to `<ChatInterface/>`.

### `<DefaultScreen/>`

This component displays the default screen when there are no conversations in the chat. It showcases the agents and action cards.

> You can configure the `DefaultScreen` by passing a `defaultScreenConfig` prop to `<ChatInterface/>`.

### `<CollectionView/>` and `<VideoView/>`

These components are used to display collection and video views, helping users better understand the context of the conversation.

# 🧑‍💻 Advanced Usage

### 🔧 Custom Message Handler

---

_Custom message handlers allow you to register specialized UI components for various content types. This is particularly useful when adding new agents that require UI elements beyond the currently supported types._

The `ChatInterface` component exposes a method `registerMessageHandler` accessible via `ref`, enabling you to register custom message handlers. This function accepts two arguments:

- `contentType`: _String_  
  The name of the agent for which the message handler is registered. The handler will be used for content types where message's content has a content type that matches `contentType`.

- `handler`: _Component_  
  The UI component to be rendered for the message type.

**The handler component will receive the following props:**

- `content`: _Object_  
  The content object of matched content type.

- `isLastConv`: _Boolean_  
  Indicates if the message is the last conversation.

**Checkout these resources to understand better:**

- [View default message handlers](https://github.com/video-db/videodb-chat/blob/main/src/components/message-handlers/)
- [View custom message handler example on CodeSandbox](https://stackblitz.com/edit/vitejs-vite-qnka6j?file=src%2FApp.vue)

### 🔧 Custom ChatHook

---

The Custom ChatHook is an advanced feature of this package that allows you to:

- Connect to your own backend, bypassing or configuring Spielberg's default backend.
- Develop custom logic for agent interactions.
- Control conversation state and manage side effects.

To use a custom hook, pass a function to the `customChatHook` prop. This function should return an object with the following properties:

- `conversations`: _Object_ (reactive)  
  See the [Conversations](#conversations) section for more details.

- `addMessage`: _Function_  
  Adds a message to the conversation. This function is called when the user clicks the **Send** button

**Checkout these resources to understand better:**

- [View default chat hook](https://github.com/video-db/videodb-chat/blob/main/src/components/hooks/useVideoDBAgent.js)
- [View custom chat hook example on CodeSandbox](https://stackblitz.com/edit/vitejs-vite-knrrbv?file=src%2FApp.vue)

# 📡 Interface

## ChatInterface

### Props

The ChatInterface component accepts the following props:

- `chatInputPlaceholder`: 
  - default: "Ask Speilberg"
  - Customizes the placeholder text for the chat input field.

- `size(string)`: 
  - default: full  
  - Determines the size of the chat interface. Options are `full` or `embedded`.
      Full takes up the entire width of the screen.
      Embedded takes up space of the parent container.

- `customChatHook(Function)`: 
  - default: [videoDBChatHook](https://github.com/video-db/videodb-chat/blob/main/src/components/hooks/useVideoDBAgent.js)
  - Allows for a custom hook to handle chat functionality.

- `chatHookConfig(object)`: 
  - Configures the chat hook. For the default chat hook, it includes:
  - default
    ```js
      socketUrl: "http://127.0.0.1:8000/chat",
      httpUrl: "http://127.0.0.1:8000",
      debug: false,
    ```

- `sidebarConfig(string)`: 
  - Customizes the sidebar.  
  - default:
    ```js
    {
          icon: SpielbergIcon,
          links: [
            {
              href: "https://docs.videodb.io",
              text: "Documentation",
            },
          ],
          primaryLink: {
            href: "https://console.videodb.io",
            text: "VideoDB Console",
            icon: VideoDBLogo,
          },
    }
    ```

- `defaultScreenConfig(Object)`: 
  - default: a list of action cards with default queries  
  - Customizes the default screen.

### Exposed Variables

#### State Variables

- `conversations`: Object  

#### Methods

- `addMessage(message)`:   
    Adds a message to the conversation.
- `registerMessageHandler(contentType, handler)`:   
  Registers a custom message handler for a specific content type.



[npm-shield]: https://img.shields.io/npm/v/@videodb/chat-vue?style=for-the-badge
[npm-url]: https://www.npmjs.com/package/@videodb/chat-vue
[discord-shield]: https://img.shields.io/badge/dynamic/json?style=for-the-badge&url=https://discord.com/api/invites/py9P639jGz?with_counts=true&query=$.approximate_member_count&logo=discord&logoColor=blue&color=green&label=discord
[discord-url]: https://discord.com/invite/py9P639jGz
[stars-shield]: https://img.shields.io/github/stars/video-db/videodb-chat.svg?style=for-the-badge
[stars-url]: https://github.com/video-db/videodb-chat/stargazers
[issues-shield]: https://img.shields.io/github/issues/video-db/videodb-chat.svg?style=for-the-badge
[issues-url]: https://github.com/video-db/videodb-chat/issues
[website-shield]: https://img.shields.io/website?url=https%3A%2F%2Fvideodb.io%2F&style=for-the-badge&label=videodb.io
[website-url]: https://videodb.io/
