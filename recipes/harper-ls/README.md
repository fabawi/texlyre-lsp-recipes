# Harper LS (Grammar & Spelling)

Harper LS (harper-ls) is an offline, privacy-first grammar and spelling checker
exposed as a Language Server. It works well for prose-heavy formats such as
TeX, LaTeX, Typst, BibTeX, Markdown, and plain text.

TeXlyre connects to Harper LS via WebSocket using lsp-ws-proxy.

---

## 1. Install the server

Harper LS is distributed via Cargo and requires the Rust toolchain.

Install harper-ls:

```bash
cargo install harper-ls --locked
````

---

## 2. Install the WebSocket proxy

TeXlyre requires a WebSocket transport. lsp-ws-proxy bridges WebSocket to stdio.

Install lsp-ws-proxy:

```bash
cargo install lsp-ws-proxy --locked
```

---

## 3. Run the server

Start the proxy and launch Harper LS over stdio:

```bash
lsp-ws-proxy -l 127.0.0.1:7000 -- harper-ls --stdio
```

* The proxy listens on ws://localhost:7000
* Keep this process running while using TeXlyre

---

## 4. TeXlyre configuration

 You can use **more than one LSP at the same time**. To do this, simply **add another config block below this one in the same list**. 
 Keep everything inside the square brackets `[` `]` and separate each  `{` `}` block with a comma.
 
Paste the following JSON into Settings ⚙️ → LSP → Generic LSP → LSP Configurations.

This configuration maps all supported file types to the markdown language ID so
Harper LS treats them consistently as prose.

```json
[
  {
    "id": "harper-ls",
    "name": "Harper LS",
    "enabled": true,
    "fileExtensions": ["tex", "latex", "typ", "bib", "md", "txt"],
    "languageIdMap": {
      "tex": "markdown",
      "latex": "markdown",
      "typ": "markdown",
      "bib": "markdown",
      "md": "markdown",
      "txt": "markdown"
    },
    "transportConfig": {
      "type": "websocket",
      "url": "ws://localhost:7000",
      "contentLength": false
    },
    "clientConfig": "{\"rootUri\":\"file:///\",\"workspaceFolders\":[]}"
  }
]
```

---

## Notes

* If you change the proxy port, update `transportConfig.url` accordingly.
* Harper LS provides diagnostics and code actions such as ignoring rules or
  adding words to a dictionary (currently, adding new words to the dictionary is functional but **NOT stable** in TeXlyre).

