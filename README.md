# TeXlyre LSP Recipes

This repository contains **Language Server Protocol (LSP) recipes** for use with the
<SettingsPath path="External Tools > Generic LSP > LSP Recipes" /> setting in TeXlyre.

Each recipe in the `recipes` directory provides:

* A brief description of the LSP tool
* Installation instructions
* How to run the server (including WebSocket proxies if required)
* A ready-to-paste JSON configuration for TeXlyre

TeXlyre’s LSP client is capability-aware. Editor features are enabled when the connected
language server advertises the corresponding capability.

## Supported LSP Features

| Group                        | Feature                           | LSP method / capability            | Support                  |
| ---------------------------- | --------------------------------- | ---------------------------------- | ------------------------ |
| **Connection & workspace**   | Initialization                    | `initialize` / `initialized`       | ✅                        |
|                              | Root URI                          | `rootUri`                          | ✅                        |
|                              | Workspace folders                 | `workspaceFolders`                 | ✅                        |
|                              | Initialization options            | `initializationOptions`            | ✅                        |
|                              | Workspace configuration           | `workspace/configuration`          | ✅                        |
| **Document synchronization** | Open document                     | `textDocument/didOpen`             | ✅                        |
|                              | Full document changes             | `textDocument/didChange`           | ✅                        |
|                              | Incremental document changes      | `textDocument/didChange`           | ✅                        |
|                              | Save notification                 | `textDocument/didSave`             | ✅ Respects `includeText` |
|                              | Close document                    | `textDocument/didClose`            | ✅                        |
| **Completion & assistance**  | Auto-completion                   | `textDocument/completion`          | ✅                        |
|                              | Completion trigger context        | `CompletionContext`                | ✅                        |
|                              | Incomplete completion lists       | `isIncomplete`                     | ✅                        |
|                              | Completion text edits             | `TextEdit`                         | ✅                        |
|                              | Insert/replace completion edits   | `InsertReplaceEdit`                | ✅                        |
|                              | Additional completion edits       | `additionalTextEdits`              | ✅                        |
|                              | Completion snippets               | `insertTextFormat: Snippet`        | ✅                        |
|                              | Completion documentation          | `documentation`                    | ✅ Markdown / plaintext   |
|                              | Hover information                 | `textDocument/hover`               | ✅ Markdown / plaintext   |
|                              | Signature help                    | `textDocument/signatureHelp`       | ✅ Basic support          |
| **Navigation & structure**   | Go to declaration                 | `textDocument/declaration`         | ✅                        |
|                              | Go to definition                  | `textDocument/definition`          | ✅                        |
|                              | Go to type definition             | `textDocument/typeDefinition`      | ✅                        |
|                              | Go to implementation              | `textDocument/implementation`      | ✅                        |
|                              | Location responses                | `Location` / `Location[]`          | ✅                        |
|                              | Location links                    | `LocationLink`                     | ✅                        |
|                              | Same-file navigation              | Navigation result                  | ✅                        |
|                              | Cross-file navigation             | Navigation result                  | ✅                        |
|                              | Document symbols                  | `textDocument/documentSymbol`      | ✅                        |
|                              | Hierarchical document symbols     | `DocumentSymbol[]`                 | ✅                        |
|                              | Legacy document symbols           | `SymbolInformation[]`              | ✅                        |
|                              | LSP-powered outline               | Document symbols                   | ✅                        |
| **Diagnostics & actions**    | Diagnostics / linting             | `textDocument/publishDiagnostics`  | ✅                        |
|                              | Document highlights               | `textDocument/documentHighlight`   | ✅                        |
|                              | Code actions / quick fixes        | `textDocument/codeAction`          | ✅                        |
|                              | Code action commands              | `workspace/executeCommand`         | ✅                        |
|                              | Workspace edits from code actions | `WorkspaceEdit`                    | ◐ Current file           |
|                              | Server-requested workspace edits  | `workspace/applyEdit`              | ◐ Current file           |
| **Semantic highlighting**    | Semantic tokens                   | `textDocument/semanticTokens/full` | ✅ Full document          |
|                              | Semantic-token refresh            | `workspace/semanticTokens/refresh` | ✅                        |
| **Server messages**          | Log messages                      | `window/logMessage`                | ✅                        |
|                              | Show messages                     | `window/showMessage`               | ✅ Logged                 |
|                              | Message requests                  | `window/showMessageRequest`        | ◐ Non-interactive        |

Support depends on the capabilities of the individual language server. 
Some more general LSP operations, including references, rename, formatting,
`textDocument/willSave`, `textDocument/willSaveWaitUntil`, and
`workspace/didChangeWatchedFiles`, are not currently exposed by TeXlyre.

---

## How Generic LSP configs work in TeXlyre

TeXlyre expects an **array of LSP configurations**. Each configuration object has the
following important fields:

* `id`
  A unique identifier for the LSP configuration.

* `name`
  A human-readable name shown in the UI.

* `enabled`
  Whether TeXlyre should attempt to connect to this server.

* `fileExtensions`
  File extensions that should activate this LSP.

* `languageIdMap` (optional)
  Maps file extensions to the LSP `languageId`.
  This is useful when an LSP expects a specific language (for example, treating TeX,
  Typst, and BibTeX as `markdown` for prose-focused servers).

* `transportConfig`
  Defines how TeXlyre communicates with the server.

  * `type`: Currently most commonly `websocket`
  * `url`: WebSocket address TeXlyre connects to
  * `contentLength` (optional): Disable if the server or proxy does not use
    `Content-Length` framing

* `clientConfig`
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

For creating installable recipes without having to manually install the server natively 
and copy configurations to TeXlyre, contribute to [Chelys recipes](https://github.com/TeXlyre/chelys-recipes)
