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

---

# Roots

Roots are a way to grant MCP servers access to specific files and folders on your local machine.

## Implementation Details

The MCP SDK doesn't automatically enforce root restrictions - you need to implement this yourself. A typical pattern is to create a helper function like `is_path_allowed()` that:

- Takes a requested file path
- Gets the list of approved roots
- Checks if the requested path falls within one of those roots
- Returns true/false for access permission

You then call this function in any tool that accesses files or directories before performing the actual file operation.

## Key Benefits

- **User-friendly** - Users don't need to provide full file paths
- **Focused search** - Claude only looks in approved directories, making file discovery faster
- **Security** - Prevents accidental access to sensitive files outside approved areas
- **Flexibility** - You can provide roots through tools or inject them directly into prompts
