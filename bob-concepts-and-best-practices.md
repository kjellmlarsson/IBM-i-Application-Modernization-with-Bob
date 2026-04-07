# Bob Concepts and Best Practices for IBM i Modernization

## Essential Concepts

| Concept | Description | Location/Usage |
|---------|-------------|----------------|
| **Modes** | Specialized AI assistants for different tasks (Code, Plan, Ask, IBM i Developer, etc.) | Switch via UI or `/mode <slug>` command |
| **Skills** | Reusable automation workflows activated by name | Currently none available by default - create custom as needed |
| **AGENTS.md** | Project-specific guidance files for Bob | Created by `/init` command |
| **`.bobignore`** | Exclude files from Bob's context | Root directory (like `.gitignore`) |
| **`.bob/rules-*/`** | Mode-specific instruction files | Created by `/init` in project root |
| **MCP Servers** | External tool integrations (databases, APIs, etc.) | Configure in Bob settings |
| **`/init` Command** | Analyzes codebase and creates AGENTS.md files | Run at project start |

---

## `/init` Command

### Purpose
Analyzes your codebase to create `AGENTS.md` files that help Bob understand project structure, conventions, and commands.

### When to Run
- **At project start** - Before first interactions with Bob
- **After major changes** - Structural refactoring, new conventions
- **Team onboarding** - Help new developers understand project

### What It Does
1. Scans project files and configuration
2. Extracts build/test/lint commands
3. Documents code style and conventions
4. Creates mode-specific guidance files

### Output Files
```
project-root/
├── AGENTS.md                           # General project guidance
└── .bob/
    ├── rules-code/AGENTS.md           # Code mode specifics
    ├── rules-advance/AGENTS.md        # Advanced mode specifics
    ├── rules-ask/AGENTS.md            # Documentation context
    └── rules-plan/AGENTS.md           # Architecture constraints
```

---

## Bob Interfaces: When to Use What

### Agentic Chat UI vs. Inline Assistant

| Interface | Best For | Use When | Example Tasks |
|-----------|----------|----------|---------------|
| **Agentic Chat UI** | Complex, multi-step tasks | Need planning, analysis, or coordination across files | • Modernizing entire programs<br>• Impact analysis<br>• Architecture design<br>• Multi-file refactoring |
| **Inline Assistant** | Quick, focused edits | Working in a single file with clear intent | • Fix syntax error<br>• Add a procedure<br>• Rename variables<br>• Format code block |

### Agentic Chat UI - Autonomous Task Execution

The **Chat UI** provides Bob with full autonomy to:
- Read and analyze multiple files
- Execute commands and run tests
- Create, modify, and delete files
- Switch between modes as needed
- Plan and execute multi-step workflows

### Inline Assistant - Direct Code Editing

The **Inline Assistant** (Cmd/Ctrl+I in editor) provides:
- Quick edits within the current file
- Immediate code generation at cursor
- Focused refactoring
- Limited to current file context

**When to use:**
```
✅ "Add error handling to this procedure"
✅ "Convert this fixed-format code block to free format"
✅ "Add comments explaining this calculation"
✅ "Rename this variable to follow naming conventions"
```

### Choosing the Right Interface

**Use Agentic Chat when:**
- Task requires reading/analyzing multiple files
- Need to understand project structure
- Requires running commands or tests
- Involves creating/modifying multiple files
- Need impact analysis or planning

**Use Inline Assistant when:**
- Working within a single file
- Making localized changes
- Quick fixes or additions
- Clear, specific edit at cursor position

---

## Best Practices for Large IBM i Projects

**Bob automatically handles:**
- Large file chunking (no manual intervention needed)
- Incremental diff application for large programs

### 1. Modernization Workflow

1. **Document** (IBM i Developer mode)
   - Understand existing business logic
   - Identify dependencies

2. **Analyze Impact** (Impact Analysis mode)
   - Check program/file dependencies
   - Assess change scope

3. **Convert Incrementally** (IBM i Modernization mode)
   - Fixed→free format conversion
   - RLA→SQL refactoring
   - One program/module at a time

4. **Test** (IBM i Test mode)
   - Create RPGUnit tests
   - Validate conversions

5. **Automate** (IBM i DevOps mode)
   - Set up build pipelines
   - Configure CI/CD

### 2. Useful Settings

Access via Bob Settings UI:

- **Max file read size** - Adjust for very large RPG programs
- **MCP server connections** - Connect to IBM i systems, databases
- **Mode-specific rules** - Customize behavior per mode
- **File restrictions** - Control which files modes can edit

### 3. Common Pitfalls to Avoid

❌ **Don't:** Ask Bob to read entire application or 10,000-line programs by default
✅ **Do:** When possible, specify program names, or line ranges for the sections you need

❌ **Don't:** Mix multiple tasks in one prompt  
✅ **Do:** Break complex tasks into steps, use appropriate modes

❌ **Don't:** Ignore `/init` command
✅ **Do:** Run `/init` to help Bob understand your project

---

### Code Review and Findings

Bob provides automated code review capabilities:

**Review Commands:**
- `/review` - Analyze uncommitted changes for issues
- `/review --issue-coverage` - Verify changes align with GitHub issue

**Review Categories:**
- **Maintainability** - Code quality, naming, DRY violations, modularity
- **Security** - Vulnerabilities, hardcoded credentials, input validation
- **Performance** - Inefficient algorithms, memory/resource leaks
- **Functionality** - Logic errors, edge cases, error handling
- **Style** - Formatting inconsistencies, documentation gaps

**Findings Panel:**
- Categorized issues with severity levels (Critical/High/Medium/Low)
- Click to navigate to issue location
- Track resolution status

### GitHub Integration

**Pull Request Commands:**
- `/pr-description` - Generate PR description from git diff
- `/create-pr` - Create pull request directly from Bob

**Issue Analysis:**
- Fetch GitHub issues by URL or number
- Analyze if local changes address issue requirements
- Example: `/review https://github.com/owner/repo/issues/123 --issue-coverage`

### MCP (Model Context Protocol) Servers

Extend Bob with external tool integrations via MCP protocol:

**Example MCP Servers:**
- **bob-rag-server** - Search library documentation (React, Python, IBM i, etc.)
- **Custom servers** - Build your own integrations (Python, Node.js)

**Configuration:** Add MCP servers in Bob Settings → MCP Servers

### Custom Modes

Create specialized modes for team-specific workflows:

**Location:** `.bob/modes/` directory in project root

**Mode Structure:**
```
.bob/modes/
└── rules-custom-mode/
    ├── 1_workflow.xml          # Task execution flow
    ├── 2_best_practices.xml    # Coding standards
    └── 3_examples.xml          # Usage examples
```

**Use Cases:**
- Company-specific coding standards
- Industry compliance requirements (HIPAA, PCI-DSS, etc.)
- Custom modernization patterns
- Team-specific workflows

**Activation:** Use `/mode custom-mode` or select from mode dropdown

---

## Quick Start Checklist

- [ ] Run `/init` to create AGENTS.md files
- [ ] Review generated `.bob/rules-*/AGENTS.md` files
- [ ] Add IBM i-specific conventions to AGENTS.md
- [ ] Create `.bobignore` for temporary files and logs
- [ ] Choose appropriate IBM i mode for your task
- [ ] Use Agentic Chat for multi-file tasks, Inline Assistant for quick edits
- [ ] Run impact analysis before major refactoring
- [ ] Create RPGUnit tests for converted code
- [ ] Set up CI/CD with IBM i DevOps mode

---

## Additional Resources

- **IBM Bob Documentation**: https://bob.ibm.com/docs/
