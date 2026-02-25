# ai-factory-extension-aif-extension-creator-skill

An [AI Factory](https://github.com/lee-to/ai-factory) extension that provides an interactive skill for scaffolding new AI Factory extensions.

The `/aif-extension-creator` skill walks you through component selection (skills, commands, injections, MCP servers, agents, replacements), collects details, and generates a ready-to-install extension directory.

## Installation

```bash
# From a local directory
ai-factory extension add ./ai-factory-extension-aif-extension-creator-skill

# From git
ai-factory extension add https://github.com/dealenx/ai-factory-extension-aif-extension-creator-skill.git
```

## Usage

Invoke the skill in your AI agent:

```
/aif-extension-creator
```

Or pass the extension name directly:

```
/aif-extension-creator my-extension
```

The skill will interactively ask you for:

1. Extension name, description, version
2. Output path (default: `.ai-factory/local_extensions/<name>`)
3. Which components to include (skills, commands, injections, MCP servers, agents, replacements)
4. Details for each selected component
5. Whether to generate a GitHub Actions CI workflow

### Example

```
/aif-extension-creator make extension with MCP:

- name: aif-ext-shadcn-mcp
- description: Adds shadcn MCP server to AI Factory
- components: MCP Servers
- MCP server key: shadcn
- MCP server config:
  {
    "command": "npx",
    "args": ["shadcn@latest", "mcp"]
  }
```

Or more simple (You need quiz to enter your extension's data):

```
/aif-extension-creator make extension with:

{
  "command": "npx",
  "args": ["shadcn@latest", "mcp"]
}
```

This generates:

```
.ai-factory/local_extensions/aif-ext-shadcn-mcp/
├── extension.json
├── package.json
├── README.md
└── mcp/
    └── shadcn.json
```

With `extension.json`:

```json
{
  "name": "aif-ext-shadcn-mcp",
  "version": "0.1.0",
  "description": "Adds shadcn MCP server to AI Factory",
  "mcpServers": [
    {
      "key": "shadcn",
      "template": "./mcp/shadcn.json",
      "instruction": "shadcn MCP server for UI component generation"
    }
  ]
}
```

And `mcp/shadcn.json`:

```json
{
  "command": "npx",
  "args": ["shadcn@latest", "mcp"]
}
```

Then install it:

```bash
ai-factory extension add ./.ai-factory/local_extensions/aif-ext-shadcn-mcp
```

## Project Structure

```
ai-factory-extension-aif-extension-creator-skill/
├── skills/
│   └── aif-extension-creator/
│       └── SKILL.md             # The extension creator skill
├── extension.json               # Extension manifest
├── package.json
├── LICENSE
└── README.md
```

## Managing the Extension

```bash
# List installed extensions
ai-factory extension list

# Remove the extension
ai-factory extension remove ai-factory-extension-aif-extension-creator-skill
```

## Documentation

- [AI Factory Extensions Guide](https://github.com/lee-to/ai-factory/blob/2.x/docs/extensions.md) — full reference for extension manifest fields, MCP servers, injections, commands, skills, and agents.

## License

[MIT](LICENSE)
