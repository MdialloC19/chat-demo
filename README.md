# chat-demo

English | [简体中文](README.zh-CN.md)

A static chat demo powered by the DeepSeek API. No application backend is required: the browser connects directly to the configured API service.

You can [try the app online](https://deepseek-ai.github.io/chat-demo/) or download the archive from [Releases](https://github.com/deepseek-ai/deepseek-chat-demo/releases), extract it, and double-click `index.html` to use it locally without installation or a build. Enter your API key in Settings before your first chat.

## Getting started

Install Node.js 22.12+ and pnpm, then run:

```sh
git clone https://github.com/deepseek-ai/chat-demo.git
cd chat-demo
pnpm install
pnpm dev
```

Open http://localhost:8090.

To build the static site:

```sh
pnpm build
```

The output is written to `dist/`. Deploy it to a static web server or open `dist/index.html` directly. Opening the file directly requires browser support for storage on `file://` pages and an API service that permits their cross-origin requests.

## Features

- Multiple conversations with automatic titles, renaming, and deletion; edit prompts, regenerate replies, and switch between message branches.
- Streaming replies, deep thinking, smart search, link reading, and source citations; stop generation or continue eligible replies.
- Upload, paste, or drag and drop attachments; use voice input and read replies aloud. A live audio spectrum appears in the first row of the input area while recording on mobile.
- Markdown, syntax highlighting, KaTeX math, and Mermaid diagrams.
- Simplified Chinese and English, light and dark themes, mobile layouts, location settings, and chat history import and export.

## Data and storage

Your API key and settings are stored in the browser's localStorage. Conversations and original attachments are stored in IndexedDB. Local data is isolated by browser profile and site origin. Sending messages, uploading attachments, and using speech features sends the relevant content to the configured API service.

History import and export in Settings use ZIP archives. After confirmation and successful validation, importing replaces all local conversations and attachments. Exports include conversations and locally available original images, but exclude API keys, settings, and original non-image attachments. See the [history import/export documentation (Chinese)](src/services/chat/history/README.md) for the archive format and preservation details.

Settings → Data → Delete all chats clears all local conversations, original attachments, and unsent drafts after confirmation. It stops ongoing replies and returns to a new chat. Your API key and settings are preserved; files already uploaded to the API service are not deleted.

## Development

| Command       | Description                  |
| ------------- | ---------------------------- |
| `pnpm dev`    | Start the development server |
| `pnpm build`  | Build into `dist/`           |
| `pnpm type`   | Run TypeScript type checks   |
| `pnpm lint`   | Lint with oxlint             |
| `pnpm format` | Format with oxfmt            |

[rspack.config.ts](rspack.config.ts) uses `https://api.deepseek.com` by default. Use `createConfig({ apiOrigin, devServer })` to customize the API origin and development server proxy.

Chat uses `deepseek-flash` by default through Anthropic-compatible Messages / Files APIs. Speech features use ASR / TTS endpoints. Requests include extensions for search, link reading, titles, and `latest_reminder`, so the configured service must support the corresponding capabilities. Supported attachment formats are determined by server-side validation.

```text
src/
  app/                 Application layout, themes, and global styles
  components/          Chat, settings, Markdown, and other UI components
  i18n/                English and Chinese translations
  services/
    api/               API requests, SSE, and protocol types
    chat/              Conversations, message branches, storage, and history import/export
    markdown/          Markdown parsing, syntax highlighting, and diagrams
    speech/            Audio recording and read-aloud playback
  utils/               Shared utilities
```

## Roadmap: local RAG and protected mode

A planned privacy layer, per conversation, so that sensitive documents stay on the device and only selected, masked excerpts reach the API. Two independent settings:

- **RAG: automatic (default) or off.** When automatic, the app uses indexed documents if the conversation has any; otherwise it sends the question directly, with nothing to enable.
- **Protected mode: on or off.** It masks identifiers (emails, phones, names from a user-managed list) before anything is sent, and restores them locally in the streamed reply.

| Protected | RAG | Route | What reaches the API |
|---|---|---|---|
| off | auto, no documents | `direct` | the conversation as-is (current behavior) |
| off | auto, with documents | `rag` | question + retrieved excerpts, in clear |
| on | auto, no documents | `protected` | the conversation, identifiers masked |
| on | auto, with documents | `protected+rag` | question + excerpts, identifiers masked |

The documents, embeddings index, identifier mapping table, and original files never leave the browser. This reduces exposure; it does not make data anonymous. Only a local generation model (e.g. Ollama) provides a "nothing leaves" guarantee.

Planned milestones: (0) a single `prepareRequest()` pipeline in `src/services/privacy/` and `src/services/rag/` that every outgoing request goes through; (1) masking, stream de-anonymization, fail-closed behavior and persistence, no RAG yet; (2) local indexing (pdf.js, WASM embeddings, IndexedDB vectors) and citations; (3) quality: send preview, hybrid search, optional NER, browser compatibility.

## License

[MIT](LICENSE). The build copies the license to `dist/LICENSE`; include it when distributing the app. Third-party dependencies are subject to their own licenses.
