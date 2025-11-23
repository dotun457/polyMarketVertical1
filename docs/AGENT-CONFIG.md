# AGENT CONFIG — Always Load Polymarket Docs

Short and sweet: make sure any agent you run always includes these files as canonical context for Polymarket questions.

## What to load
- `./README.md` (top-level summary)
- `./docs/**/*.md` (all documents in the `docs` folder)

## For VS Code (Copilot / Copilot Chat)
1. Enable workspace file inclusion (Search in Settings for "include workspace files" in the Copilot/Chat extension).
2. Add `docs/**` and `README.md` to the included files filter if supported.
3. Increase "Max files included" / context size if available.

## For MCPs (e.g., `polymarket-mcp`)
- Use the repository root `.mcp-config.json` file provided.
- Start the MCP with the config:

```
polymarket-mcp --config ./.mcp-config.json
```

(If your MCP uses a different flag, point to `.mcp-config.json` in its start command.)

## For ChatGPT-like systems (Custom Instructions)
- Add to custom instructions or system messages:

> Always treat the `README.md` and contents of `docs/` in the repository as authoritative for Polymarket developer queries. If the user asks a Polymarket question, load these files first.

## For Local LLM Retrieval (Optional quick setup)
- Build a small index or vector store (e.g., LLama/FAISS/Chroma).
- Script idea: `scripts/index_docs.py` reads `README.md` + `docs/**.md` and stores embeddings.
- Use `retriever.get_relevant_chunks(query)` to fetch doc chunks and add the top-N to the system prompt for each LLM call.

## Recommended Quick Practices
- Keep this file updated with any new doc folders.
- If you use CI/automation to update docs, add an indexing job to refresh the MCP or vector store.
- Add `docs/AGENT-CONFIG.md` to your repo so new contributors and agents know where to look.

---

## Minimal Config Example (.mcp-config.json)
See the repository root `.mcp-config.json` (already included).

---

## Example System Prompt (copy-paste)
> You are a Polymarket assistant. Before answering any Polymarket question, include the top results from `README.md` and `docs/*` and treat them as the canonical source.

---

Last updated: November 22, 2025
