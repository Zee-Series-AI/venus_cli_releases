# Venus CLI

A command-line interface tool for managing HTML5 games on the Venus platform. This CLI helps developers create, update, and manage their games efficiently.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Commands](#commands)
  - [games create](#games-create)
  - [games update](#games-update)
  - [games list](#games-list)
  - [games configure](#games-configure)
  - [env use](#env-use)
  - [env list](#env-list)
  - [env configure](#env-configure)
  - [update](#update)
- [Environment Variables](#environment-variables)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

## Installation

### Quick Install (Recommended)

#### macOS/Linux

Install Venus CLI with a single command:

```bash
curl -fsSL https://github.com/Zee-Series-AI/venus_cli_releases/releases/latest/download/install.sh | bash
```

The installer will:
- Automatically detect your OS and architecture
- Download the correct binary
- Install it to `/usr/local/bin`
- Make it executable and ready to use

#### Windows (PowerShell)

```powershell
irm https://github.com/Zee-Series-AI/venus_cli_releases/releases/latest/download/install.ps1 | iex
```

The installer will:
- Automatically detect your architecture
- Download the correct binary
- Install it to `%LOCALAPPDATA%\Programs\Venus`
- Optionally add the installation directory to your PATH

## Quick Start

The CLI works out of the box with your local development environment. No configuration needed!

```bash
# Create a new game (creates game.config.json automatically)
venus games create --name "My Game" --path "./dist"

# List your games
venus games list

# Update your game (uses game.config.json - no need to specify ID or path!)
venus games update
```

## Configuration

Configuration is **optional**. By default, the CLI will automatically connect to the local development environment:
- **Default API Base URL**: `http://localhost:5001/dev-venus-app/us-central1`
- **Default API Access Token**: `dev-secret-key`

### Custom Configuration (Optional)

If you need to connect to a different environment (staging, production, etc.), you can configure the CLI with your Venus API credentials.

#### Configuring Environments

The CLI supports multiple named environments (dev, staging, production, etc.):

```bash
# Configure the default 'dev' environment
venus env configure dev --base-url "http://localhost:5001/dev-venus-app/us-central1" --access-token "dev-secret-key"

# Configure a staging environment
venus env configure staging --base-url "https://staging-api.venus.com" --access-token "your-staging-token"

# Configure a production environment
venus env configure prod --base-url "https://api.venus.com" --access-token "your-production-token"
```

**Options:**
- Environment name (argument): The name of the environment (e.g., `dev`, `staging`, `prod`)
- `--base-url`: The Venus API endpoint URL
- `--access-token`: Your authentication token for the Venus API

The configuration is saved to `~/.venus_cli/config.json`.

#### Switching Between Environments

```bash
# Switch to staging environment
venus env use staging

# Switch to production environment
venus env use prod
```

#### Listing Environments

```bash
# See all configured environments
venus env list
```

## Commands

### games create

Creates a new HTML5 game application on Venus.

```bash
venus games create --name "My Awesome Game" --path "/path/to/game/dist"
```

**Options:**
- `--name` (required): The name of your game
- `--path` (required): Path to your game's distribution/build folder

**What it does:**
1. Zips your game distribution folder
2. Uploads the zip file to Venus storage
3. Creates a new app with version 0.0.1
4. Returns the created app ID
5. Creates a `game.config.json` file in your current directory with the game ID and path

**About game.config.json:**

This file stores your game's configuration and makes future updates easier:
```json
{
  "gameId": "your-app-id",
  "relativePathToDistFolder": "./dist"
}
```

With this file, you can run `venus games update` without specifying `--id` or `--path`!

### games update

Pushes a new game update to Venus with a new version.

```bash
# Using game.config.json (recommended - created by venus games create)
venus games update --bump patch

# Or specify manually
venus games update --id "your-app-id" --path "/path/to/game/dist" --bump patch
```

**Options:**
- `--id` (optional): The app ID to update. If not provided, reads from `game.config.json`
- `--path` (optional): Path to your updated game's distribution/build folder. If not provided, reads from `game.config.json`
- `--bump` (optional, default: `patch`): Version bump type - `major`, `minor`, or `patch`

**Version Bumping:**
- `major`: 1.0.0 → 2.0.0
- `minor`: 1.0.0 → 1.1.0
- `patch`: 1.0.0 → 1.0.1

**Note:** If you have a `game.config.json` file in your current directory (created by `venus games create`), the `--id` and `--path` options are optional. The command will use the values from the config file unless you explicitly override them.

### games list

Lists all your games on Venus.

```bash
venus games list
```

**Output includes:**
- App ID
- Game name
- Current version
- Last update timestamp

### games configure

Configure your game.config.json for ease of deployment.

```bash
venus games configure --id "your-app-id" --path "./dist"
```

**Options:**
- `--id` (required): The app ID of your game
- `--path` (required): Relative path to your game's distribution/build folder

**What it does:**
Creates or updates the `game.config.json` file in your current directory, making future deployments easier by storing your game ID and path.

### env use

Switch to a different environment.

```bash
venus env use staging
```

**Argument:**
- Environment name: The name of the environment to use (e.g., `dev`, `staging`, `prod`)

### env list

List all configured environments.

```bash
venus env list
```

Shows all environments you've configured, along with their base URLs and which one is currently active.

### env configure

Configure a specific environment or update the currently active environment.

```bash
venus env configure dev --base-url "http://localhost:5001/dev-venus-app/us-central1" --access-token "dev-secret-key"
venus env configure staging --base-url "https://staging-api.venus.com" --access-token "staging-token"
```

**Argument:**
- Environment name: The name of the environment to configure (e.g., `dev`, `staging`, `prod`)

**Options:**
- `--base-url`: The Venus API endpoint URL
- `--access-token`: Your authentication token for the Venus API

### update

Update the Venus CLI to the latest version.

```bash
venus update
```

**What it does:**
1. Checks your current installed CLI version
2. Fetches the latest version available from GitHub releases
3. If a newer version is available, automatically downloads and installs it
4. If you're already on the latest version, informs you that no update is needed

**How it works:**
- On Windows: Downloads and runs the PowerShell installer script
- On macOS/Linux: Downloads and runs the bash installer script
- The update process is automatic and will replace your current installation

**Example output:**
```
Current version: 1.0.0
Latest version: 1.1.0
Updating via bash...
✅ Update completed successfully.
```

or if already up to date:

```
Current version: 1.1.0
Latest version: 1.1.0
You are already up to date.
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
venus games create --name "Space Invaders HD" --path "./build/web"
# Output: App created with id 'abc123xyz'

# For production environment (configure and switch first)
venus env configure prod --base-url "https://api.venus.com" --access-token "my-secret-token"
venus env use prod
venus games create --name "Space Invaders HD" --path "./build/web"
```

### Example 2: Updating an Existing Game

```bash
# If you have game.config.json (created by venus games create):

# Update with default patch version bump (1.0.0 → 1.0.1)
venus games update

# Update with a minor version bump (1.0.1 → 1.1.0)
venus games update --bump minor

# Update with a major version bump (1.1.0 → 2.0.0)
venus games update --bump major

# If you don't have game.config.json, specify manually:

# Update with patch version bump
venus games update --id "abc123xyz" --path "./build/web" --bump patch

# Override config file values
venus games update --id "different-id" --path "./different/path" --bump minor
```

### Example 3: Typical Development Workflow

```bash
# Initial setup: Create your game
venus games create --name "My Awesome Game" --path "./dist"
# This creates game.config.json automatically

# Make changes to your game...
# Build your game to ./dist

# Push updates - super simple!
venus games update
# Uses game.config.json, defaults to patch bump (1.0.0 → 1.0.1)

# Major feature release
venus games update --bump minor
# 1.0.1 → 1.1.0

# Breaking changes
venus games update --bump major
# 1.1.0 → 2.0.0

# Check all your games
venus games list
```

### Example 4: Multi-Environment Workflow

```bash
# Configure staging environment
venus env configure staging --base-url "https://staging-api.venus.com" --access-token "staging-token"

# Configure production environment
venus env configure prod --base-url "https://api.venus.com" --access-token "production-token"

# List all environments
venus env list

# Use staging environment
venus env use staging

# Create app in staging
venus games create --name "My Game Beta" --path "./dist"

# Switch to production
venus env use prod

# Create app in production
venus games create --name "My Game" --path "./dist"
```

## Troubleshooting

### Common Issues

1. **"Failed to upload file" error**
   - Check your internet connection
   - Verify your API access token is valid
   - Ensure the game distribution folder exists and is not empty

2. **"Game dist folder does not exist" error**
   - Verify the path to your game's build folder is correct
   - Ensure you're using the full path or correct relative path
   - Check that the path in `game.config.json` is correct if using auto-detection

3. **"Unable to load game config" error**
   - Make sure you're running the command from the directory containing `game.config.json`
   - Or provide `--id` and `--path` options manually
   - Verify the `game.config.json` file is valid JSON

4. **Authentication errors**
   - Run `venus env configure <env-name>` to update your credentials
   - Check if your access token has expired
   - Verify the API base URL is correct
   - Ensure you're using the correct environment with `venus env use <env-name>`

5. **Version conflicts**
   - When updating an app, the version must be higher than the current version
   - Use appropriate bump type (major, minor, patch)

### Getting Help

- Check the command help: `venus --help`
- Check games command help: `venus games --help`
- Check env command help: `venus env --help`
- Check specific command help: `venus games create --help` or `venus env configure --help`
- Review the error messages carefully - they often contain helpful information

## License

[Include license information here]

## Contributing

[Include contribution guidelines if applicable]
