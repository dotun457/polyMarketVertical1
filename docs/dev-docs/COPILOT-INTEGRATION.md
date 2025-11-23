# Copilot Integration Guide  
### Ensuring GitHub Copilot & Copilot Chat Always Use Polymarket Docs

This guide explains how to configure GitHub Copilot Chat (and other workspace LLM agents) to always load the **Polymarket developer documentation** stored in this repo whenever you ask a question related to Polymarket, prediction markets, or CLOB/RTDS/Gamma workflows.

This ensures:
- **Accurate coding assistance**
- **Reduced hallucinations**
- **Reference-driven explanations**
- **Consistent dev workflow**

---

# 🔧 1. What Copilot Should Always Load

Copilot Chat should treat the following files as **canonical references**:

### A. Root project summary
- `./README.md`

### B. All internal documentation
- `./docs/**/*.md`

### C. Polymarket developer docs
Located in:
```
docs/
```

These include references for:
- CLOB  
- RTDS  
- Gamma  
- Rewards  
- Neg-risk  
- Subgraph  
- Resolution  
… and more.

---

# ⚙️ 2. VS Code Setup Instructions (Copilot Chat)

### ✔ Enable workspace file context
Open VS Code → Settings → search:
- `Copilot Chat: Allow File Context`
- `Copilot Chat: Workspace Context`

Turn **both ON**.

---

### ✔ Increase the number of files Copilot can load
Setting:
- `Copilot Chat: Max Context Files`

Set to **1000** (or maximum allowed).

---

### ✔ Add file include patterns
If supported by your Copilot version, add:
```
README.md
docs/**
```

---

### ✔ Use slash commands to force-include context
In Copilot Chat:
```
#file:docs/
#file:README.md
```

Or use the chat interface to explicitly reference files when asking questions.

---

# 🧩 3. Using Copilot with MCPs (Model Context Protocol)

If using MCP tools (like `polymarket-mcp`):

### Use the existing `.mcp-config.json`
Example:

```bash
polymarket-mcp --config ./.mcp-config.json