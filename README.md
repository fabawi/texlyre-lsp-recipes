# TeXlyre LSP Recipes

This repository contains **Language Server Protocol (LSP) recipes** for use with TeXlyre’s
**Settings → LSP → Generic LSP** feature.

Each recipe in the `recipes` directory provides:
- A brief description of the LSP tool
- Installation instructions
- How to run the server (including WebSocket proxies if required)
- A ready-to-paste JSON configuration for TeXlyre

TeXlyre’s LSP support is built on top of CodeMirror’s LSP client, enabling common editor
features depending on server capabilities:
- Diagnostics / linting
- Code actions
- Hover information
- Auto-completion

---

## How Generic LSP configs work in TeXlyre

TeXlyre expects an **array of LSP configurations**. Each configuration object has the
following important fields:

- `id`  
  A unique identifier for the LSP configuration.

- `name`  
  A human-readable name shown in the UI.

- `enabled`  
  Whether TeXlyre should attempt to connect to this server.

- `fileExtensions`  
  File extensions that should activate this LSP.

- `languageIdMap` (optional)  
  Maps file extensions to the LSP `languageId`.  
  This is useful when an LSP expects a specific language (for example, treating TeX,
  Typst, and BibTeX as `markdown` for prose-focused servers).

- `transportConfig`  
  Defines how TeXlyre communicates with the server.
  - `type`: Currently most commonly `websocket`
  - `url`: WebSocket address TeXlyre connects to
  - `contentLength` (optional): Disable if the server or proxy does not use
    `Content-Length` framing

- `clientConfig`  
  A JSON **string** passed to the LSP client during initialization.
  A minimal setup usually includes `rootUri` and `workspaceFolders`.

---

## Minimal example

```json
[
  {
    "id": "example-lsp",
    "name": "Example LSP",
    "enabled": true,
    "fileExtensions": ["tex"],
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7000"
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

## Contributing

When contributing a new recipe:
* Create a new folder under `recipes/`
* Add a README.md
* Keep the README structured as:
  1. What the LSP does
  2. How to install it
  3. How to run it
  4. TeXlyre JSON configuration
