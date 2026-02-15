# VS Code Language Servers (vscode-langservers-extracted)

`vscode-langservers-extracted` is a bundle of Language Servers extracted from VS Code.
It provides IDE features such as diagnostics, completion, hover, formatting, and (where supported) linting for several common web/config languages.

TeXlyre connects to these servers via WebSocket using `lsp-ws-proxy`.

---

## 1. Install the server

The VS Code language servers are distributed via npm.

Install `vscode-langservers-extracted` globally:

```bash
npm install -g vscode-langservers-extracted
```

This installs the following language server executables:

* `vscode-css-language-server`
* `vscode-html-language-server`
* `vscode-json-language-server`
* `vscode-markdown-language-server`

---

## 2. Install the WebSocket proxy

TeXlyre requires a WebSocket transport. `lsp-ws-proxy` bridges WebSocket to stdio.

Install `lsp-ws-proxy`:

```bash
cargo install lsp-ws-proxy --locked
```

---

## 3. Run the server

Each language server should be launched behind its own `lsp-ws-proxy` instance (one port per server).

### JSON / JSONC

```bash
lsp-ws-proxy -l 127.0.0.1:7010 -- vscode-json-language-server --stdio
```

### HTML

```bash
lsp-ws-proxy -l 127.0.0.1:7011 -- vscode-html-language-server --stdio
```

### CSS / SCSS / LESS

```bash
lsp-ws-proxy -l 127.0.0.1:7012 -- vscode-css-language-server --stdio
```

### Markdown

```bash
lsp-ws-proxy -l 127.0.0.1:7013 -- vscode-markdown-language-server --stdio
```

* Each proxy listens on `ws://localhost:<port>`
* Keep these processes running while using TeXlyre

---

## 4. TeXlyre configuration

You can use **more than one LSP at the same time**. To do this, simply **add another config block below** in the **same list**.
Keep everything inside the square brackets `[` `]` and separate each `{` `}` block with a comma.

Paste the following JSON into Settings ⚙️ → LSP → Generic LSP → LSP Configurations.

### JSON / JSONC configuration

```json
[
  {
    "id": "vscode-json-ls",
    "name": "VS Code JSON Language Server",
    "enabled": true,
    "fileExtensions": ["json", "jsonc"],
    "languageIdMap": {
      "json": "json",
      "jsonc": "jsonc"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7010",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

### HTML configuration

```json
[
  {
    "id": "vscode-html-ls",
    "name": "VS Code HTML Language Server",
    "enabled": true,
    "fileExtensions": ["html", "htm"],
    "languageIdMap": {
      "html": "html",
      "htm": "html"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7011",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

### CSS / SCSS / LESS configuration

```json
[
  {
    "id": "vscode-css-ls",
    "name": "VS Code CSS Language Server",
    "enabled": true,
    "fileExtensions": ["css", "scss", "less"],
    "languageIdMap": {
      "css": "css",
      "scss": "scss",
      "less": "less"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7012",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

### Markdown configuration

```json
[
  {
    "id": "vscode-markdown-ls",
    "name": "VS Code Markdown Language Server",
    "enabled": true,
    "fileExtensions": ["md", "markdown"],
    "languageIdMap": {
      "md": "markdown",
      "markdown": "markdown"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7013",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

### Example: multiple configs in one list

If you want JSON + HTML + CSS + Markdown at the same time, combine them like this:

```json
[
  {
    "id": "vscode-json-ls",
    "name": "VS Code JSON Language Server",
    "enabled": true,
    "fileExtensions": ["json", "jsonc"],
    "languageIdMap": { "json": "json", "jsonc": "jsonc" },
    "transportConfig": { "type": "websocket", "url": "ws://localhost:7010", "contentLength": false },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  },
  {
    "id": "vscode-html-ls",
    "name": "VS Code HTML Language Server",
    "enabled": true,
    "fileExtensions": ["html", "htm"],
    "languageIdMap": { "html": "html", "htm": "html" },
    "transportConfig": { "type": "websocket", "url": "ws://localhost:7011", "contentLength": false },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  },
  {
    "id": "vscode-css-ls",
    "name": "VS Code CSS Language Server",
    "enabled": true,
    "fileExtensions": ["css", "scss", "less"],
    "languageIdMap": { "css": "css", "scss": "scss", "less": "less" },
    "transportConfig": { "type": "websocket", "url": "ws://localhost:7012", "contentLength": false },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  },
  {
    "id": "vscode-markdown-ls",
    "name": "VS Code Markdown Language Server",
    "enabled": true,
    "fileExtensions": ["md", "markdown"],
    "languageIdMap": { "md": "markdown", "markdown": "markdown" },
    "transportConfig": { "type": "websocket", "url": "ws://localhost:7013", "contentLength": false },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

---

## Notes

* If you change a proxy port, update the matching `transportConfig.url`.
* These language servers support different feature sets:

  * **JSON**: validation, schemas, completion, hover, formatting
  * **HTML**: validation, completion, hover, formatting
  * **CSS/SCSS/LESS**: validation, completion, hover, formatting
  * **Markdown**: formatting and document features (feature support depends on client capabilities)
* Some actions like `sort JSON` are still not fully compatible with TeXlyre
