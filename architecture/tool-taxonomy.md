# Tool Taxonomy

Claude Code has 16 built-in tools, each with specific purposes, usage instructions, and behavioural constraints encoded in the system prompt. This document provides the full taxonomy.

---

## The Four Categories

```
CODEBASE INTELLIGENCE    →    MODIFICATION    →    EXECUTION
(understand first)            (change second)       (verify third)

Read, Glob, Grep, LS    Write, Edit,          Bash
                        MultiEdit,
                        NotebookEdit,
                        NotebookRead

WEB                     TASK AND AGENT
WebSearch, WebFetch     TodoWrite,
                        TaskCreate/Get/Update/List,
                        Task
```

---

## Category 1: Codebase Intelligence

### Read
**Purpose:** Read the full contents of a file.  
**When to use:** Before any edit. Before running commands that depend on file content.  
**Key behavioural instruction from prompt:** Read files before editing them to understand current state.  
**Example:**
```
Read("src/middleware/auth.js")
```

### Glob
**Purpose:** Find files matching a pattern.  
**When to use:** When you need to find all files of a type, or find files matching a naming pattern.  
**Key behavioural note:** Returns file paths, not contents. Follow with Read for content.  
**Example:**
```
Glob("src/**/*.test.ts")
Glob("**/*.env*")
```

### Grep
**Purpose:** Search file contents using regex.  
**When to use:** Finding all usages of a function, locating configuration values, finding all imports of a module.  
**Key behavioural note:** Returns matching lines with line numbers. Use to pinpoint targets for Edit.  
**Example:**
```
Grep("getUserById", include="*.ts")
Grep("JWT_SECRET", include="**/*.{ts,js,env*}")
```

### LS
**Purpose:** List directory contents.  
**When to use:** Mapping project structure, verifying a file was created, checking what's in a directory.  
**Example:**
```
LS("src/")
LS(".")
```

---

## Category 2: Modification

### Write
**Purpose:** Create a new file with specified content.  
**Key behavioural instruction from prompt:** Check if file exists before writing (use Read or Glob first). Do not overwrite existing files with Write — use Edit instead.  
**Example:**
```
Write("src/utils/jwt.js", content="...")
```

### Edit
**Purpose:** Edit an existing file using targeted find-replace operations.  
**Key behavioural instruction from prompt:** Read the file first. Target the minimum change. Verify surrounding context to avoid editing the wrong lines.  
**Example:**
```
Edit("src/routes/auth.js",
  old_string="// TODO: implement login",
  new_string="return handler.login(req, res)")
```

### MultiEdit
**Purpose:** Multiple targeted edits in a single file, in one operation.  
**When to use:** When multiple sections of the same file need changes (e.g., adding an import AND adding a function). More efficient than sequential Edit calls.  
**Example:**
```
MultiEdit("src/app.js", [
  {old: "// imports", new: "const jwt = require('./utils/jwt');\n// imports"},
  {old: "// middleware", new: "app.use('/api', jwtMiddleware);\n// middleware"}
])
```

### NotebookRead
**Purpose:** Read Jupyter notebook cells.  
**When to use:** Projects that include `.ipynb` files (data science, ML engineering codebases).

### NotebookEdit
**Purpose:** Edit Jupyter notebook cells.  
**When to use:** Modifying analysis notebooks or ML training scripts in Jupyter format.

---

## Category 3: Execution

### Bash
**Purpose:** Execute shell commands.  
**When to use:** Running tests, linters, type checkers, git commands, package installation, file system operations, reading CI logs.  
**Key behavioural instructions from prompt:**
- Do not use interactive flags (`-i`, `-p`, editor-opening commands)
- Read output before deciding next action
- For destructive operations, confirm intent first
- Use HEREDOC for multi-line input to bash commands

**Example:**
```bash
# Run tests
Bash("npm test 2>&1")

# Check TypeScript
Bash("npx tsc --noEmit 2>&1")

# Git commit
Bash('git commit -m "$(cat <<\'EOF\'\nfeat: add JWT auth\nEOF\n)"')
```

---

## Category 4: Web

### WebSearch
**Purpose:** Search the web for information.  
**When to use:** Finding library documentation, checking if a package exists, looking up an API specification, verifying a technique.  
**Key behavioural note:** Use for reference research, not for continuous browsing. Prefer official docs over community sources for API specifications.

### WebFetch
**Purpose:** Fetch a specific URL.  
**When to use:** Reading API documentation, fetching an example from a known URL, reading a specification document.  
**Example:**
```
WebFetch("https://docs.stripe.com/api/payment_intents/create")
```

---

## Category 5: Task and Agent

### TodoWrite
**Purpose:** Simple in-session task list (legacy, pre-January 2025).  
**Current status:** Present, but superseded by TaskCreate system for complex tasks.  
**When to use:** Quick in-session tracking of minor items (still useful for simple sub-tasks).

### TaskCreate / TaskGet / TaskUpdate / TaskList
**Purpose:** Full persistent task management with dependency graphs.  
**See:** [`sdlc/02-task-management.md`](../sdlc/02-task-management.md) for complete documentation.

### Task
**Purpose:** Spawn a sub-agent with a specified prompt and toolset.  
**This is the most architecturally significant tool** — it enables the entire multi-agent system.  
**Key parameters:**
- `prompt`: The sub-agent's task description
- `tools`: Restricted toolset for the sub-agent
- `model`: Override model (e.g., Haiku for cheap tasks, Opus for complex)

**Example:**
```
Task(
  prompt="Read all files in src/auth/ and identify all places where user credentials are handled. Return a list of file paths and line numbers.",
  tools=["Read", "Grep", "LS"],
  model="haiku"
)
```

---

## The Read-Before-Write Principle

The most important cross-tool behavioural instruction in the system prompt: **read before writing.**

This is stated explicitly for `Write` and `Edit`, and implicitly encoded in the recommended workflow:
1. `Glob` to find relevant files
2. `Read` to understand current state
3. `Edit` or `Write` to make targeted changes
4. `Bash` to verify (run tests/linter)

This principle prevents the most common agentic coding failure: writing code that conflicts with existing patterns, duplicates existing functionality, or breaks file structures that weren't visible at planning time.

---

## MCP Server Extension

The 16 built-in tools are a floor. Model Context Protocol (MCP) servers add additional tools that Claude Code treats as native. Common MCP extensions:

- **Database tools** — introspect schema, run queries, check relationships
- **CI/CD tools** — trigger pipelines, read build logs, manage deployments
- **Internal APIs** — connect to proprietary internal systems
- **Custom search** — search internal documentation or knowledge bases

MCP servers are configured in Claude Code's settings file and appear as additional tools in the tool list. The same usage instructions and trust model apply.
