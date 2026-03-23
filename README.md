# Drafts MCP Server

A [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server that enables AI assistants to interact with the [Drafts](https://getdrafts.com) app on macOS through AppleScript.

For additional discussion of uses, see [the Drafts forum](https://forums.getdrafts.com/t/drafts-mcp-server-for-ai-integration/16507)

## Features

- 📝 **Draft Management**: Create, read, update, and search drafts
- 🏷️ **Tags**: Add and manage tags on drafts
- 📂 **Workspaces**: List and query drafts from specific workspaces
- ⚡ **Actions**: Run Drafts actions programmatically
- 🔍 **Search**: Full-text search across all drafts
- 🚩 **Flags & Archive**: Flag, archive, or trash drafts

## CLI Tool

In addition to the MCP server, this package includes a standalone `drafts` CLI for interacting with Drafts directly from the terminal.

### CLI Installation

```bash
# Install globally from npm
npm install -g @agiletortoise/drafts-mcp-server

# Or link locally for development
git clone https://github.com/agiletortoise/drafts-mcp-server.git
cd drafts-mcp-server
npm install
npm run build
npm link
```

After installation, the `drafts` command is available globally.

### CLI Usage

Output is human-readable by default. Add `--json` for machine-readable JSON output.

```bash
# Workspaces
drafts workspace list
drafts workspace current
drafts workspace open "Work"
drafts workspace drafts "Work" --folder inbox

# Tags
drafts tag list
drafts tag get "work"

# List and search drafts
drafts list
drafts list --folder inbox --tag work --flagged
drafts list --created-after 2025-01-01 --modified-before 2025-06-30
drafts search "meeting notes"

# Single draft operations
drafts current
drafts get <uuid>
drafts create "Hello world" --tag notes --tag ideas --flagged
drafts update <uuid> "Updated content"
drafts open <uuid>

# Organize drafts
drafts flag <uuid>
drafts flag <uuid> --unflag
drafts archive <uuid>
drafts inbox <uuid>
drafts trash <uuid>
drafts add-tags <uuid> important urgent

# Actions
drafts action list
drafts action run <draft-uuid> "Send to Email"

# JSON output for scripting
drafts --json list --folder inbox
drafts --json search "budget"
```

## Requirements

- macOS (AppleScript is macOS-only)
- [Drafts](https://getdrafts.com) app v50.0.3 or greater installed
- Node.js 18 or higher

## Installation

### Quick Start (After Publishing to npm)

You will need to have Node installed on your Mac so make the `npx` command available. If you do not already have Node installed, you can do so with [Homebrew](https://brew.sh) using:

```bash
brew install node
```

Once published, use with `npx` - no installation needed:

```bash
npx @agiletortoise/drafts-mcp-server
```

### Global Installation

```bash
npm install -g @agiletortoise/drafts-mcp-server
```

### Local Development/Testing

Before publishing, test locally:

```bash
# Clone or extract the package
cd drafts-mcp-server

# Install dependencies
npm install

# Build
npm run build

# Test with MCP Inspector
npm run inspector

# Or run directly
node dist/index.js
```

### Configuration for Claude Desktop

The easiest way to use this MCP with the Claude Desktop app by installing the MCPB version. To do so:

* Download the [drafts-mcp-server.mcpb](https://github.com/agiletortoise/drafts-mcp-server/blob/main/drafts-mcp-server.mcpb?raw=true) from the project to your Mac.
* Open Claude Desktop
* Navigate to `Settings > Extensions`
* Drag and drop the drafts-mcp-server.mcpb file from the Finder into this window to install.
* Follow the prompts to complete installation.

We are likely to submit this project to the Claude extension directory soon to make it available directly in the app.

#### Advanced Claude Desktop Configuration

If you prefer to install locally, or run via `npm`, these instructions are for you...

You will need to have Node installed on your Mac so make the `npx` command available. If you do not already have Node installed, you can do so with [Homebrew](https://brew.sh) using:

```bash
brew install node
```

Add the below to the Claude Desktop confguration file (`~/Library/Application Support/Claude/claude_desktop_config.json`), and relaunch Claude.

```json
{
  "mcpServers": {
    "drafts": {
      "command": "npx",
      "args": ["@agiletortoise/drafts-mcp-server"]
    }
  }
}
```

**For local development/testing** (before publishing), use:

```json
{
  "mcpServers": {
    "drafts": {
      "command": "node",
      "args": ["/absolute/path/to/drafts-mcp-server/dist/index.js"]
    }
  }
}
```

Or if globally installed:

```json
{
  "mcpServers": {
    "drafts": {
      "command": "drafts-mcp-server"
    }
  }
}
```

### Configuration for Cursor

**After publishing to npm**, add to your Cursor MCP settings (`.cursor/mcp.json` in your project or global settings):

```json
{
  "mcpServers": {
    "drafts": {
      "command": "npx",
      "args": ["@agiletortoise/drafts-mcp-server"]
    }
  }
}
```

**For local development/testing**, use:

```json
{
  "mcpServers": {
    "drafts": {
      "command": "node",
      "args": ["/absolute/path/to/drafts-mcp-server/dist/index.js"]
    }
  }
}
```

### Configuration for Claude Code

Claude Code (the CLI tool) can be configured using the `/mcp` command or by editing the settings file directly.

**Using the CLI:**

```bash
claude mcp add drafts -- npx @agiletortoise/drafts-mcp-server
```

**For local development/testing**, use:

```bash
claude mcp add drafts -- node /absolute/path/to/drafts-mcp-server/dist/index.js
```

After adding, restart Claude Code or start a new session for the MCP server to be available.

## Permissions

The first time the server runs, macOS will ask for permissions:

1. **System Preferences** > **Security & Privacy** > **Privacy** > **Automation**
2. Allow the MCP host (e.g., Claude Desktop, Claude Code, Cursor) to control **Drafts**

## Available Tools

### Workspace Management

#### `drafts_list_workspaces`
List all workspaces in Drafts.

```typescript
// No parameters required
```

### Draft Operations

#### `drafts_get_drafts`
Get drafts with flexible filtering by content, folder, tag, flagged status, and dates.

```typescript
{
  query?: string;                // Optional: Query string to filter drafts
  folder?: inbox, archive, trash // Optional: Limit to one folder
  tag: string                    // Optional: Limit by assigned tag
}
```

#### `drafts_create_draft`
Create a new draft with content and optional tags.

```typescript
{
  content: string;        // Required: Draft content
  tags?: string[];        // Optional: Array of tag names
  flagged?: boolean;      // Optional: Flag the draft
}
```

#### `drafts_get_draft`
Retrieve a specific draft by UUID.

```typescript
{
  uuid: string;           // Required: UUID of the draft
}
```

#### `drafts_update_draft`
Update the content of an existing draft.

```typescript
{
  uuid: string;           // Required: UUID of the draft
  content: string;        // Required: New content
}
```

#### `drafts_search`
Search for drafts across all workspaces.

```typescript
{
  query: string;          // Required: Search query
}
```

### Tags

#### `drafts_add_tags`
Add tags to an existing draft.

```typescript
{
  uuid: string;           // Required: UUID of the draft
  tags: string[];         // Required: Array of tag names to add
}
```

### Actions

#### `drafts_list_actions`
List all available Drafts actions.

```typescript
// No parameters required
```

#### `drafts_run_action`
Run a Drafts action on a specific draft.

```typescript
{
  draftUuid: string;      // Required: UUID of the draft
  actionName: string;     // Required: Name of the action to run
}
```

### Draft Status

#### `drafts_flag`
Flag or unflag a draft.

```typescript
{
  uuid: string;           // Required: UUID of the draft
  flagged: boolean;       // Required: true to flag, false to unflag
}
```

#### `drafts_archive`
Archive a draft.

```typescript
{
  uuid: string;           // Required: UUID of the draft to archive
}
```

#### `drafts_trash`
Move a draft to trash.

```typescript
{
  uuid: string;           // Required: UUID of the draft to trash
}
```

## Example Usage

Here are some example prompts you can use with Claude or other AI assistants:

### Basic Operations

```
"Show me all my workspaces in Drafts"

"Get all drafts from my 'Work' workspace"

"Create a new draft with the content 'Meeting notes for Q1 planning'"

"Search for drafts containing 'budget'"
```

### Advanced Operations

```
"Create a draft with content 'Todo: Review PR #123' and tag it with 'work' and 'urgent'"

"Find the draft about the marketing campaign and run the 'Send to Email' action on it"

"Flag all drafts in my Inbox workspace that contain 'follow up'"

"Archive all drafts tagged 'completed'"
```

### Workflow Integration

```
"Get all drafts from my 'Daily Notes' workspace from the last week, 
 then create a summary draft tagged 'weekly-review'"

"Search for all drafts tagged 'meeting-notes', extract action items, 
 and create a new draft with all the action items combined"
```

## Development

### Setup

```bash
# Clone the repository
git clone https://github.com/agiletortoise/drafts-mcp-server.git
cd drafts-mcp-server

# Install dependencies
npm install

# Build
npm run build

# Test with MCP Inspector
npm run inspector
```

### Project Structure

```
drafts-mcp-server/
├── src/
│   ├── index.ts           # Main MCP server implementation
│   ├── cli.ts             # Standalone CLI tool
│   ├── drafts.ts          # Drafts-specific operations
│   └── applescript.ts     # AppleScript execution utilities
├── dist/                  # Compiled JavaScript (generated)
├── package.json
├── tsconfig.json
└── README.md
```

### Testing

Use the [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) to test the server:

```bash
npm run inspector
```

This opens a web interface where you can test each tool interactively.

## Troubleshooting

### "Drafts got an error: Can't get workspace..."

Make sure you've spelled the workspace name exactly as it appears in Drafts. Workspace names are case-sensitive.

### Permission Errors

1. Check **System Preferences** > **Security & Privacy** > **Privacy** > **Automation**
2. Ensure your MCP client (Claude Desktop, Cursor, etc.) has permission to control Drafts
3. You may need to restart the client after granting permissions

### Script Execution Errors

1. Ensure Drafts is installed and has been launched at least once
2. Try running a simple AppleScript manually to verify permissions:
   ```bash
   osascript -e 'tell application "Drafts" to get name of first workspace'
   ```

### No Workspaces/Drafts Returned

This might indicate that AppleScript dictionary access isn't working. Check that:
- You're using the latest version of Drafts
- Your AppleScript dictionary is properly exposed in Drafts

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

MIT License - see the [LICENSE](LICENSE) file for details.

## Related Projects

- [Model Context Protocol](https://modelcontextprotocol.io) - The protocol specification
- [Drafts](https://getdrafts.com) - Where text starts
- [Claude Desktop](https://claude.ai/download) - AI assistant with MCP support

## Support

- **Issues**: [GitHub Issues](https://github.com/agiletortoise/drafts-mcp-server/issues)
- **Drafts Support**: [Drafts Community](https://forums.getdrafts.com)
- **MCP Documentation**: [MCP Docs](https://modelcontextprotocol.io/docs)
