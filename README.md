   # Venus CLI
   
   Public releases for Venus CLI.
   
   See the [releases page](https://github.com/Zee-Series-AI/venus_cli_releases/releases) for downloads.

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Commands](#commands)
  - [create-app](#create-app)
  - [update-app](#update-app)
  - [list-apps](#list-apps)
  - [configure](#configure)
- [Environment Variables](#environment-variables)
- [Usage Examples](#usage-examples)

## Installation

#### macOS/Linux

Install Venus CLI with a single command:

```bash
curl -fsSL https://github.com/Zee-Series-AI/venus_cli_releases/releases/latest/download/install.sh | bash
```

#### Windows (PowerShell)

```powershell
irm https://github.com/Zee-Series-AI/venus_cli_releases/releases/latest/download/install.ps1 | iex
```

## Quick Start

The CLI works out of the box with your local development environment. No configuration needed!

```bash
# Create a new game
venus create-app --name "My Game" --path "./dist"

# List your games
venus list-apps

# Update an existing game (defaults to patch bump)
venus update-app --id "your-app-id" --path "./dist"
```

## Configuration

Configuration is **optional**. By default, the CLI will automatically connect to the local development environment:
- **Default API Base URL**: `http://localhost:5001/dev-venus-app/us-central1`
- **Default API Access Token**: `dev-secret-key`

### Custom Configuration (Optional)

If you need to connect to a different environment (staging, production, etc.), you can configure the CLI with your Venus API credentials:

```bash
venus configure
```

You'll be prompted to enter:
- **API Base URL**: The Venus API endpoint URL
- **API Access Token**: Your authentication token for the Venus API

Alternatively, you can provide these values directly:

```bash
venus configure --api-base-url "https://your-api-url.com" --api-access-token "your-access-token"
```

The configuration is saved to `~/.venus_cli/config.json`.

## Commands

### create-app

Creates a new HTML5 game application on Venus.

```bash
venus create-app --name "My Awesome Game" --path "/path/to/game/dist"
```

**Options:**
- `--name` (required): The name of your game
- `--path` (required): Path to your game's distribution/build folder

**What it does:**
1. Zips your game distribution folder
2. Uploads the zip file to Venus storage
3. Creates a new app with version 0.0.1
4. Returns the created app ID

### update-app

Updates an existing game with a new version.

```bash
venus update-app --id "your-app-id" --path "/path/to/game/dist" --bump patch
```

**Options:**
- `--id` (required): The app ID to update
- `--path` (required): Path to your updated game's distribution/build folder
- `--bump` (optional, default: `patch`): Version bump type - `major`, `minor`, or `patch`

**Version Bumping:**
- `major`: 1.0.0 → 2.0.0
- `minor`: 1.0.0 → 1.1.0
- `patch`: 1.0.0 → 1.0.1

### list-apps

Lists all your games on Venus.

```bash
venus list-apps
```

**Output includes:**
- App ID
- Game name
- Current version
- Last update timestamp

### configure

Updates the CLI configuration.

```bash
venus configure [--api-base-url "url"] [--api-access-token "token"]
```

## Environment Variables

You can override configuration settings using environment variables:

- `VENUS_API_BASE_URL`: Overrides the configured API base URL
- `VENUS_API_ACCESS_TOKEN`: Overrides the configured API access token

Environment variables take precedence over the config file.

## Usage Examples

### Example 1: Creating a New Game

```bash
# For local development (no configuration needed)
venus create-app --name "Space Invaders HD" --path "./build/web"
# Output: App created with id 'abc123xyz'

# For production environment (configure first)
venus configure --api-base-url "https://api.venus.com" --api-access-token "my-secret-token"
venus create-app --name "Space Invaders HD" --path "./build/web"
```

### Example 2: Updating an Existing Game

```bash
# Update with default patch version bump (1.0.0 → 1.0.1)
venus update-app --id "abc123xyz" --path "./build/web"

# Explicitly specify patch version bump (1.0.0 → 1.0.1)
venus update-app --id "abc123xyz" --path "./build/web" --bump patch

# Update with a minor version bump (1.0.1 → 1.1.0)
venus update-app --id "abc123xyz" --path "./build/web" --bump minor

# Update with a major version bump (1.1.0 → 2.0.0)
venus update-app --id "abc123xyz" --path "./build/web" --bump major
```

### Example 3: Development Workflow

```bash
# List all your games
venus list-apps

# Use environment variables for different environments
export VENUS_API_BASE_URL="https://staging-api.venus.com"
export VENUS_API_ACCESS_TOKEN="staging-token"

# Create app in staging
venus create-app --name "My Game Beta" --path "./dist"

# Switch to production
export VENUS_API_BASE_URL="https://api.venus.com"
export VENUS_API_ACCESS_TOKEN="production-token"

# Create app in production
venus create-app --name "My Game" --path "./dist"
```
