# MCP Chat with File System Access

MCP Chat is a command-line interface application that enables interactive chat capabilities with AI models through the Anthropic API. The application supports file system operations with controlled access to specified directories, video conversion capabilities, and extensible tool integrations via the MCP (Model Control Protocol) architecture.

> This repository is part of the Model Context Protocol Advanced course and is used as a hands-on playground for learning about Roots.

## Prerequisites

- Python 3.10+
- Anthropic API Key
- FFmpeg (for video conversion features)

## Setup

_You must have FFmpeg already installed to convert a video file_. To install FFmpeg on MacOS run:

```
brew install ffmpeg
```

### Step 1: Configure the environment variables

1. Copy the `.env.example` file to create a new `.env` file:

```bash
cp .env.example .env
```

2. Edit the `.env` file and set your environment variables:

```
CLAUDE_MODEL="claude-sonnet-4-0"  # Or your preferred Claude model
ANTHROPIC_API_KEY=""  # Enter your Anthropic API secret key
```

### Step 2: Install dependencies

#### Setup with uv

[uv](https://github.com/astral-sh/uv) is a fast Python package installer and resolver.

1. Install uv, if not already installed:

```bash
pip install uv
```

2. Install dependencies:

```bash
uv sync
```

3. Run the project

When running the project, you must specify one or more root directories that the MCP server will have access to. Only files and directories within these roots can be accessed by the server.

```bash
uv run main.py <root1> [root2] [root3] ...
```

Examples:

```bash
# Single directory
uv run main.py /path/to/videos

# Multiple directories
uv run main.py /home/user/videos /mnt/storage/media ~/Documents

# Current directory
uv run main.py .
```

## Features

### File System Access

The server can only access files and directories within the specified root paths. This provides security by limiting file system access to approved locations.

### Available Tools

- **list_roots**: List all accessible root directories
- **read_dir**: Read contents of a directory (must be within a root)
- **convert_video**: Convert MP4 videos to other formats (avi, mov, webm, mkv, gif)

### Video Conversion

The video conversion tool uses FFmpeg to convert MP4 files to various formats:

- Standard video formats: AVI, MOV, WebM, MKV
- GIF conversion with optimized settings
- Medium quality preset for balanced file size and quality

### Defining roots

This program is set up to accept a list of CLI arguments, which are interpreted as paths that the user wants to allow access to.

**_main.py_**
`root_paths = sys.argv[1:]`

That list of paths is provided to the MCPClient down on lines 42.

### Creating root objects

According to the MCP spec, all roots should have a URI that begins with file://.

The function `_create_roots()` takes the list of paths of that the user provided and turns them into Root objects.

### Roots callback

The client doesn't immediately provide the list of roots to the server. Instead, the server can make a request to the client at some future point in time. We make a callback that will be executed when the server requests the roots. The callback needs to return the list of roots inside of a `ListRootsResult` object.

This callback is passed into the `ClientSession` down on line 58 of `mcp_client.py`.

### Using the roots

On to the server. The server will use the roots in two scenarios:

1. Whenever a tool attempts to access a file or folder
2. When a LLM (like Claude) needs to resolve a file or folder to a full path. Think of when a user says 'read the todos.txt file' - Claude needs to figure out where the text file is, and might do so by looking at the list of roots

To handle the second case, we can either define a tool that lists out the roots or inject them directly in a prompt.

### Accessing the roots

Roots are accessed by calling `ctx.session.list_roots()`.

This sends a message back to the client, which causes it to run the root-listing callback.

### Authorizing access

The MCP SDK does not attempt to limit what files or folders your tools attempt to read.

A function like `is_path_allowed`, will decide whether a path is accessible by comparing it to the list of roots.

## Use it throughout your tools to ensure the requested path is accessible

# Roots

Roots are a way to grant MCP servers access to specific files and folders on your local machine.

## Implementation Details

The MCP SDK doesn't automatically enforce root restrictions - you need to implement this yourself. A typical pattern is to create a helper function like `is_path_allowed()` that:

- Takes a requested file path
- Gets the list of approved roots
- Checks if the requested path falls within one of those roots
- Returns true/false for access permission

```python
async def is_path_allowed(requested_path: Path, ctx: Context) -> bool:
    roots_result = await ctx.session.list_roots()
    client_roots = roots_result.roots

    if not requested_path.exists():
        return False

    if requested_path.is_file():
        requested_path = requested_path.parent

    for root in client_roots:
        root_path = file_url_to_path(root.uri)
        try:
            requested_path.relative_to(root_path)
            return True
        except ValueError:
            continue

    return False
```

You then call this function in any tool that accesses files or directories before performing the actual file operation.

> Ideally, a user will dictate which files/folders can be accessed by the MCP server.

## Key Benefits

- **User-friendly** - Users don't need to provide full file paths
- **Focused search** - Claude only looks in approved directories, making file discovery faster
- **Security** - Prevents accidental access to sensitive files outside approved areas
- **Flexibility** - You can provide roots through tools or inject them directly into prompts
