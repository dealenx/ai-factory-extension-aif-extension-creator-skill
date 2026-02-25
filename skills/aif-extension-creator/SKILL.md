---
name: aif-extension-creator
description: Interactively creates a new AI Factory extension with selected components (skills, commands, injections, MCP servers, agents, replacements).
---

# AI Factory Extension Creator

Create a new AI Factory extension interactively. The skill walks the user through component selection, collects details for each component, then generates the full extension directory structure.

## Invocation

```
/aif-extension-creator
/aif-extension-creator my-extension-name
```

If a name is provided as an argument, skip the name prompt and use it directly.

---

## Step 1: Gather Basic Information

Ask the user each of the following questions **one at a time**. Wait for the answer before proceeding.

### 1.1 Extension Name

Ask: **What is the name of your extension?**

- Skip this prompt if a name was passed as an argument.
- Validate the name: allowed characters are letters, digits, `_`, `-`, `.`, `@`, `/`. The name must NOT contain `..` or be an absolute path.
- If invalid, explain the rules and ask again.
- Good examples: `aif-ext-my-tool`, `my-company/aif-logging`, `aif-ext-prettier`.

### 1.2 Description

Ask: **Provide a short description of your extension.**

A one-liner, e.g. "Adds Prettier formatting to the implement workflow".

### 1.3 Version

Ask: **What version should the extension start at?** (default: `0.1.0`)

Accept a SemVer string or use the default.

### 1.4 Output Path

Ask: **Where should the extension be generated?**

- Default: `.ai-factory/local_extensions/<extension_name>`
- The user can provide any relative or absolute path.
- If the directory already exists and contains files, warn the user and ask for confirmation before overwriting.

### 1.5 Components to Include

Ask: **Which components should the extension include?** (select all that apply)

Present these options:

1. **Skills** — custom slash-command skills (SKILL.md files)
2. **Commands** — custom CLI commands (JavaScript ESM modules)
3. **Injections** — content injected into existing skills (append/prepend)
4. **MCP Servers** — Model Context Protocol server configurations
5. **Agents** — new agent definitions
6. **Skill Replacements** — replace built-in skills with custom versions

The user can select one or more. At minimum, suggest including **Skills** as the most common use case.

### 1.6 GitHub Actions CI (optional)

Ask: **Do you want to generate a GitHub Actions CI workflow for testing the extension?** (yes/no, default: no)

---

## Step 2: Collect Component Details

For each component the user selected in Step 1.5, ask follow-up questions to gather the details needed for generation. Process components in the order listed below.

### 2.1 Skills Details

For each skill the user wants to create:

Ask:
1. **Skill name** — e.g. `my-custom-skill` (used as directory name under `skills/`)
2. **Skill description** — one-liner for the YAML frontmatter
3. **Brief purpose** — what should this skill do? (used to generate initial SKILL.md body content)

Ask: **Do you want to add another skill?** Repeat until the user says no.

### 2.2 Commands Details

For each command:

Ask:
1. **Command name** — e.g. `hello` (used as both the CLI command and filename `commands/<name>.js`)
2. **Command description** — shown in CLI help
3. **Options** — any CLI options? e.g. `--name <name>` with default value (optional, can be none)

Ask: **Do you want to add another command?** Repeat until the user says no.

### 2.3 Injections Details

For each injection:

Ask:
1. **Target skill** — which existing skill to inject into, e.g. `aif-implement`, `aif-commit`, `aif-plan`
2. **Position** — `append` (add to end) or `prepend` (add after YAML frontmatter)
3. **Content summary** — what should the injected content say? (used to generate the injection markdown file)

The injection filename will be derived as `<target>-<position>.md` (e.g. `implement-append.md`).

Ask: **Do you want to add another injection?** Repeat until the user says no.

### 2.4 MCP Servers Details

For each MCP server:

Ask:
1. **Server key** — unique identifier, e.g. `my-server`
2. **Command** — the command to run, e.g. `npx`
3. **Args** — command arguments as a list, e.g. `["-y", "@my-org/my-mcp-server"]`
4. **Environment variables** — any env vars needed? e.g. `MY_API_KEY` (will use `${MY_API_KEY}` placeholder)
5. **Instruction** — optional message shown to the user after installation, e.g. "Set MY_API_KEY environment variable"

Ask: **Do you want to add another MCP server?** Repeat until the user says no.

### 2.5 Agents Details

For each agent:

Ask:
1. **Agent ID** — unique identifier, e.g. `my-agent`
2. **Display name** — human-readable name, e.g. `My Agent`
3. **Config directory** — e.g. `.my-agent`
4. **Skills directory** — e.g. `.my-agent/skills`
5. **Settings file** — path to MCP settings file, or `null`
6. **Supports MCP?** — yes/no
7. **Skills CLI agent** — value for `--agent` flag, or `null`

Ask: **Do you want to add another agent?** Repeat until the user says no.

### 2.6 Skill Replacements Details

For each replacement:

Ask:
1. **Extension skill path** — which skill from this extension replaces a built-in? Must be one of the skills defined in Step 2.1 (format: `skills/<skill-name>`)
2. **Base skill name** — the built-in skill name being replaced, e.g. `aif-commit`, `aif-implement`

If the user hasn't created any skills in Step 2.1, warn them that replacements require at least one skill and ask if they want to add a skill first.

Ask: **Do you want to add another replacement?** Repeat until the user says no.

---

## Step 3: Generate the Extension

After all information is collected, generate the extension directory structure. Create ALL files using the templates below.

### Target directory structure

```
<output_path>/
├── extension.json
├── package.json
├── README.md
├── skills/                  # Only if Skills were selected
│   └── <skill-name>/
│       └── SKILL.md
├── commands/                # Only if Commands were selected
│   └── <command-name>.js
├── injections/              # Only if Injections were selected
│   └── <injection-file>.md
├── mcp/                     # Only if MCP Servers were selected
│   └── <server-key>.json
└── .github/                 # Only if CI was selected
    └── workflows/
        └── test.yml
```

---

### 3.1 Generate `extension.json`

This is the extension manifest. Only include fields for components the user selected.

```json
{
  "name": "<extension_name>",
  "version": "<version>",
  "description": "<description>",
  "commands": [
    {
      "name": "<command_name>",
      "description": "<command_description>",
      "module": "./commands/<command_name>.js"
    }
  ],
  "injections": [
    {
      "target": "<target_skill>",
      "position": "<append|prepend>",
      "file": "./injections/<injection_file>.md"
    }
  ],
  "skills": [
    "skills/<skill_name>"
  ],
  "replaces": {
    "skills/<extension_skill_path>": "<base_skill_name>"
  },
  "mcpServers": [
    {
      "key": "<server_key>",
      "template": "./mcp/<server_key>.json",
      "instruction": "<instruction_text>"
    }
  ],
  "agents": [
    {
      "id": "<agent_id>",
      "displayName": "<display_name>",
      "configDir": "<config_dir>",
      "skillsDir": "<skills_dir>",
      "settingsFile": "<settings_file_or_null>",
      "supportsMcp": <true|false>,
      "skillsCliAgent": "<cli_agent_or_null>"
    }
  ]
}
```

**Rules:**
- `name` and `version` are always required.
- Omit `commands` array entirely if no commands were selected.
- Omit `injections` array entirely if no injections were selected.
- Omit `skills` array entirely if no skills were selected.
- Omit `replaces` object entirely if no replacements were selected.
- Omit `mcpServers` array entirely if no MCP servers were selected.
- Omit `agents` array entirely if no agents were selected.
- `description` can always be included.

### 3.2 Generate `package.json`

Always generate a minimal package.json:

```json
{
  "type": "module"
}
```

### 3.3 Generate Skill Files

For each skill, create `skills/<skill_name>/SKILL.md`:

```markdown
---
name: <skill_name>
description: <skill_description>
---

# <Skill Name (title case)>

<Brief purpose provided by the user, expanded into a starter instruction set.>

## Usage

```
/<skill_name>
```

## Instructions

<!-- Add your skill instructions here -->

```

Write a meaningful starter body based on the user's description of the skill's purpose. Do NOT leave it as just a placeholder — include at least a basic workflow outline.

### 3.4 Generate Command Files

For each command, create `commands/<command_name>.js`:

```javascript
// commands/<command_name>.js

/**
 * <command_description>
 */
export function register(program) {
  program
    .command('<command_name>')
    .description('<command_description>')
    // Add options here:
    // .option('--name <name>', 'Name to greet', 'World')
    .action((opts) => {
      // TODO: Implement command logic
      console.log('<command_name> executed');
    });
}
```

If the user specified options, include them as `.option()` calls with proper defaults.

### 3.5 Generate Injection Files

For each injection, create `injections/<injection_file>.md`:

```markdown
## <Title derived from target and purpose> (from <extension_name>)

<Content based on the user's content summary. Write meaningful instructional content, not just a placeholder.>
```

The filename is derived as: `<target_skill_name_without_aif_prefix>-<position>.md`
For example: target `aif-implement`, position `append` → `implement-append.md`.

### 3.6 Generate MCP Server Template Files

For each MCP server, create `mcp/<server_key>.json`:

```json
{
  "command": "<command>",
  "args": [<args_array>],
  "env": {
    "<ENV_VAR>": "${<ENV_VAR>}"
  }
}
```

If no environment variables were specified, omit the `env` field.

### 3.7 Generate README.md

Always generate a README with the following structure:

```markdown
# <extension_name>

<description>

## Installation

```bash
# From local directory
ai-factory extension add ./<relative_path_to_extension>

# From git repository (if published)
# ai-factory extension add https://github.com/<user>/<extension_name>.git
```

## Components

<List each component type that was included, with a brief description of each item.>

### Skills
- `/<skill_name>` — <skill_description>

### Commands
- `ai-factory <command_name>` — <command_description>

### Injections
- Injects into `/<target_skill>` (<position>) — <brief description>

### MCP Servers
- `<server_key>` — <instruction>

### Agents
- `<agent_id>` (<display_name>) — <brief description>

## Removal

```bash
ai-factory extension remove <extension_name>
```

## License

This extension is provided as-is.
```

Only include sections for component types that were actually selected. Omit sections for components not included.

### 3.8 Generate GitHub Actions CI Workflow (if selected)

Create `.github/workflows/test.yml`:

```yaml
name: Test Extension

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install AI Factory
        run: |
          git clone https://github.com/lee-to/ai-factory.git /tmp/ai-factory
          cd /tmp/ai-factory
          npm install
          npm link

      - name: Create test project
        run: |
          mkdir -p /tmp/test-project
          cd /tmp/test-project
          cat > .ai-factory.json << 'CONF'
          {
            "version": "2.2.0",
            "agents": [
              {
                "id": "opencode",
                "skillsDir": ".opencode/skills",
                "installedSkills": [],
                "mcp": {}
              }
            ],
            "extensions": []
          }
          CONF

      - name: Install extension
        run: |
          cd /tmp/test-project
          ai-factory extension add ${{ github.workspace }}

      - name: Verify extension is registered
        run: |
          cd /tmp/test-project
          ai-factory extension list | grep "<extension_name>"

      - name: Verify extension files exist
        run: |
          test -f /tmp/test-project/.ai-factory/extensions/<extension_name>/extension.json
```

Replace `<extension_name>` with the actual extension name.

---

## Step 4: Summary

After generating all files, present a summary to the user:

### Format

```
Extension "<extension_name>" created successfully!

Location: <full_output_path>

Generated files:
  - extension.json (manifest)
  - package.json
  - README.md
  - skills/<skill_name>/SKILL.md          (for each skill)
  - commands/<command_name>.js             (for each command)
  - injections/<injection_file>.md         (for each injection)
  - mcp/<server_key>.json                  (for each MCP server)
  - .github/workflows/test.yml            (if CI selected)

To install this extension in your project:
  ai-factory extension add ./<relative_path>

To install from a git repo (after pushing):
  ai-factory extension add https://github.com/<user>/<extension_name>.git
```

---

## Important Rules

1. **Always create the output directory** — use `mkdir -p` equivalent to create all parent directories.
2. **Never overwrite without asking** — if the target directory exists and has files, ask for confirmation.
3. **Validate the extension name** — reject names with `..`, absolute paths, or characters outside `[a-zA-Z0-9_\-\.@/]`.
4. **Generate only selected components** — do not create directories or files for components the user did not select.
5. **Write meaningful content** — do not leave files as empty stubs. Use the user's descriptions to write starter content that is actually useful.
6. **Follow AI Factory conventions** — the generated extension must be installable via `ai-factory extension add` without modifications.
7. **Use ESM** — all JavaScript files must use ES module syntax (`export function`, not `module.exports`).
8. **JSON formatting** — use 2-space indentation for all JSON files.
9. **Markdown formatting** — use standard markdown with YAML frontmatter for SKILL.md files.
10. **Default output path** — if the user accepts the default, generate into `.ai-factory/local_extensions/<extension_name>`.

## Reference: extension.json Field Validation

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | Yes | Unique name. Chars: `[a-zA-Z0-9_\-\.@/]`, no `..`, no absolute paths. |
| `version` | `string` | Yes | SemVer recommended. |
| `description` | `string` | No | Human-readable description. |
| `commands` | `array` | No | CLI commands. Each: `{ name, description, module }`. |
| `injections` | `array` | No | Skill injections. Each: `{ target, position, file }`. |
| `skills` | `array` | No | Relative paths to skill directories. |
| `replaces` | `object` | No | Maps extension skill path to base skill name. |
| `mcpServers` | `array` | No | MCP configs. Each: `{ key, template, instruction }`. |
| `agents` | `array` | No | Agent definitions. Each: `{ id, displayName, configDir, skillsDir, settingsFile, supportsMcp, skillsCliAgent }`. |
