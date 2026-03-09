---
description: Generate CLAUDE.md for a PowerBuilder project. Analyzes the solution and creates comprehensive project documentation for Claude.
argument-hint: "[solution-path]"
---

# PowerBuilder Project Setup — CLAUDE.md Generator

Generate a comprehensive CLAUDE.md file for a PowerBuilder project.

## Process

1. **Analyze the project** using `pb_get_project_structure`
2. **Discover the inheritance tree** by running `pb_get_inheritance` on key ancestor objects
3. **Identify conventions** by sampling object names across the project
4. **Read the project file** (.pbproj) for build configuration

## Generate CLAUDE.md with these sections:

### Project Overview
- Application name, version
- Number of libraries, objects by type
- Database type and connection info

### Architecture
- Key libraries and their roles
- Inheritance hierarchy (from _ancestor)
- Module organization

### Naming Conventions
- Detected naming patterns (prefixes for windows, objects, datawindows, variables, functions)

### Build & Deploy
- PBAutoBuild250 command with all flags
- PBD copy step
- Launch instructions

### Development Workflow
- How to modify code (reference pb-modify skill)
- How to debug (reference pb-debug skill)
- How to create new objects (reference pb-create skill)

### Key Objects
- List of ancestor objects and their purpose
- Most frequently referenced objects

### MCP Tools Available

List all available MCP tools (pb_list_objects, pb_read_object, pb_search_code, etc.) with brief descriptions.

### PMIX Auto-detection (if PMIX project)

If the project contains PMIX-specific libraries (`_sysxtra`, `_cust2`, `Cust_Empty`), add ALL of the following sections to the generated CLAUDE.md:

```markdown
## Auto-detection nouveau projet
Si `.pmix-client.json` n'existe pas a la racine, invoquer le skill pmix-onboard
avant toute autre action. Ce skill scanne les libraries custom, identifie le client,
indexe le code specifique dans le RAG, et genere un resume du projet.
```

```markdown
## RAG — Base de connaissances PMIX

### Base standard (incluse)
La base RAG standard (`pmix-standard.db`) contient ~17200 chunks couvrant :
- Architecture PMIX, modules, fenetres, DataWindows
- 428 tables, 175 vues, 82 procedures stockees
- 12 flux metier detailles (vente, achat, stock, fabrication, etc.)
- Patterns UI, regles universelles, DDDWs

### Base custom (a initialiser)
La partie custom du RAG indexe le code specifique au client (`Cust_*`, `_sysxtra`).
Elle est initialisee automatiquement par le skill `pmix-onboard` ou manuellement via `pmix_reindex`.

### Outils RAG
- `pmix_search` — recherche hybride FTS5 + semantique
- `pmix_lookup` — lookup direct par nom d'objet/table
- `pmix_correct` — corriger une doc erronee
- `pmix_learn` — ajouter une connaissance apprise
- `pmix_reindex` — re-indexer les docs

### REGLE ABSOLUE : TOUJOURS consulter le RAG AVANT d'agir
Avant toute action sur PMIX (modification, creation, navigation GUI, debug) :
1. Utiliser `pmix_search` ou `pmix_lookup` pour comprendre le contexte
2. Verifier les tables, fenetres, et flux concernes
3. Puis seulement agir en connaissance de cause
Ne JAMAIS deviner ou improviser sans avoir consulte le RAG d'abord.
```

```markdown
## Skills disponibles

### Skills PowerBuilder (generiques)
- **pb-analyze** — Explorer l'architecture, heritage, dependances, flux de donnees
- **pb-modify** — Modifier du code PB (workflow obligatoire en 5 etapes)
- **pb-debug** — Diagnostiquer bugs et comportements inattendus
- **pb-create** — Creer de nouveaux objets PB

### Skills PMIX (specifiques)
- **pmix-navigate** — Repondre a toute question PMIX via le RAG et les docs knowledge
- **pmix-flux** — Documenter et expliquer les processus metier
- **pmix-impact** — Analyser l'impact d'une modification sur les flux
- **pmix-onboard** — Initialiser un nouveau projet PMIX (scan custom, indexation RAG)
```

Write the generated CLAUDE.md to the project root.

## Generate .mcp.json

After generating CLAUDE.md, create or FIX the `.mcp.json` file at the project root.

**IMPORTANT**: If `.mcp.json` already exists but contains placeholders like `<SERVER_JS_PATH>` or `<TOOLKIT_DIR>`, you MUST fix them. A `.mcp.json` with placeholders is NOT complete.

### How to find the server path

1. Read `~/.claude/pb-toolkit-path.txt` — it contains the full path to `server.js`
2. If that file doesn't exist, search for PB-Toolkit installation:
   - Check `C:/Program Files/PB-Toolkit/packages/mcp-server/dist/server.js`
   - Check `C:/Program Files (x86)/PB-Toolkit/packages/mcp-server/dist/server.js`
   - Ask the user where PB-Toolkit is installed

### .mcp.json template

Replace `<SERVER_JS_PATH>` with the path found above, and `<PROJECT_ROOT>` with the current project's absolute path (forward slashes):

```json
{
  "mcpServers": {
    "powerbuilder": {
      "command": "node",
      "args": ["<SERVER_JS_PATH>"],
      "env": {
        "PB_SOLUTION_PATH": "<PROJECT_ROOT>",
        "PB_EXE_PATH": "<PROJECT_ROOT>/pmix/pmix.exe",
        "PB_REFERENCES_DIR": "<TOOLKIT_DIR>/references",
        "PYTHON_EXE": "python"
      }
    }
  }
}
```

Notes:
- `PB_SOLUTION_PATH` MUST point to the current project root (not PMIX reference)
- `PB_EXE_PATH` should point to the project's compiled .exe (adjust if not in pmix/ subdir)
- `PYTHON_EXE` can be left as "python" — the bridge will find it
- All paths must use forward slashes
- After creating `.mcp.json`, tell the user to **restart Claude Code** for the MCP server to connect
