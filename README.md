# Venus CLI

A command-line interface tool for managing HTML5 games on the Venus platform. This CLI helps developers create, update, and manage their games efficiently.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Commands](#commands)
  - [create-game](#create-game)
  - [update-game](#update-game)
  - [publish-game](#publish-game)
  - [list-games](#list-games)
  - [configure-game](#configure-game)
  - [use-env](#use-env)
  - [list-envs](#list-envs)
  - [configure-env](#configure-env)
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

### Publishing NEW game to Venus

```bash
# 1. Create a new game (creates game.config.json automatically) Skip this step if you alread have a game uploaded to venus
venus create-game --name "My Game" --path "./dist"

# 2. Create new versions as you develop (uses game.config.json)
venus update-game --bump patch

# 3. Publish your game to the selected environment
venus publish-game

# List your games
venus list-games
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
venus configure-env dev --base-url "http://localhost:5001/dev-venus-app/us-central1" --access-token "dev-secret-key"

# Configure a staging environment
venus configure-env staging --base-url "https://staging-api.venus.com" --access-token "your-staging-token"

# Configure a production environment
venus configure-env prod --base-url "https://api.venus.com" --access-token "your-production-token"
```

**Options:**
- Environment name (argument): The name of the environment (e.g., `dev`, `staging`, `prod`)
- `--base-url`: The Venus API endpoint URL
- `--access-token`: Your authentication token for the Venus API

The configuration is saved to `~/.venus_cli/config.json`.

#### Switching Between Environments

```bash
# Switch to staging environment
venus use-env staging

# Switch to production environment
venus use-env prod
```

#### Listing Environments

```bash
# See all configured environments
venus list-envs
```

## Commands

### create-game

Creates a new HTML5 game application on Venus.

```bash
venus create-game --name "My Awesome Game" --path "/path/to/game/dist"
```

**Options:**
- `--name` (required): The name of your game
- `--path` (required): Path to your game's distribution/build folder

**What it does:**
1. Zips your game distribution folder
2. Uploads the zip file to Venus storage
3. Creates a new game with an initial version (0.0.1)
4. Returns the created game ID
5. Creates a `game.config.json` file in your current directory with the game ID and path

**About game.config.json:**

This file stores your game's configuration and makes future updates easier:
```json
{
  "gameId": "your-game-id",
  "relativePathToDistFolder": "./dist"
}
```

With this file, you can run `venus update-game` and `venus publish-game` without specifying the game ID!

**Note:** After creating a game, you'll need to use `publish-game` to make it visible in a specific environment.

### update-game

Creates a new version of your game with updated content.

```bash
# Using game.config.json (recommended - created by venus create-game)
venus update-game --bump patch

# Or specify manually
venus update-game --id "your-game-id" --path "/path/to/game/dist" --bump patch
```

**Options:**
- `--id` (optional): The game ID to update. If not provided, reads from `game.config.json`
- `--path` (optional): Path to your updated game's distribution/build folder. If not provided, reads from `game.config.json`
- `--bump` (optional, default: `patch`): Version bump type - `major`, `minor`, or `patch`

**Version Bumping:**
- `major`: 1.0.0 → 2.0.0 (breaking changes)
- `minor`: 1.0.0 → 1.1.0 (new features)
- `patch`: 1.0.0 → 1.0.1 (bug fixes)

**What it does:**
1. Zips your game distribution folder
2. Uploads the new version to Venus storage
3. Creates a new version entry for your game
4. Does NOT automatically publish the version - use `publish-game` for that

**Note:** If you have a `game.config.json` file in your current directory (created by `venus create-game`), the `--id` and `--path` options are optional. The command will use the values from the config file unless you explicitly override them.

### publish-game

Makes your game visible in a specific environment by publishing the latest version.

```bash
# Publish to a custom environment (dev, staging, etc.)
venus publish-game dev

# Publish to production
venus publish-game prod
```

**Arguments:**
- `environment` (required): The environment name to publish to (e.g., `dev`, `staging`, `prod`)

**What it does:**
1. Fetches your game information from `game.config.json`
2. Gets the latest version of your game
3. Publishes that version to the specified environment
4. Returns a OneLink URL for accessing the game

**Important:**
- Uses `game.config.json` to determine which game to publish
- For custom environments (non-prod), you must have configured that environment using `configure-env`
- Publishing to `prod` uses the production publishing flow
- Publishing to other environments pushes the game to that specific environment's backend

**Example workflow:**
```bash
# Create a new version with your latest changes
venus update-game --bump patch

# Publish to dev environment for testing
venus publish-game dev

# Once tested, publish to production
venus publish-game prod
```

### list-games

Lists all your games on Venus.

```bash
venus list-games
```

**Output includes:**
- App ID
- Game name
- Current version
- Last update timestamp

### configure-game

Configure your game.config.json for ease of deployment.

```bash
venus configure-game --id "your-app-id" --path "./dist"
```

**Options:**
- `--id` (required): The app ID of your game
- `--path` (required): Relative path to your game's distribution/build folder

**What it does:**
Creates or updates the `game.config.json` file in your current directory, making future deployments easier by storing your game ID and path.

### use-env

Switch to a different environment.

```bash
venus use-env staging
```

**Argument:**
- Environment name: The name of the environment to use (e.g., `dev`, `staging`, `prod`)

### list-envs

List all configured environments.

```bash
venus list-envs
```

Shows all environments you've configured, along with their base URLs and which one is currently active.

### configure-env

Configure a specific environment or update the currently active environment.

```bash
venus configure-env dev --base-url "http://localhost:5001/dev-venus-app/us-central1" --access-token "dev-secret-key"
venus configure-env staging --base-url "https://staging-api.venus.com" --access-token "staging-token"
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

### Example 1: Creating and Publishing a New Game

```bash
# Step 1: Create your game
venus create-game --name "Space Invaders HD" --path "./build/web"
# Output: Game created with id 'abc123xyz'
# Creates game.config.json automatically

# Step 2: Publish to dev environment for testing
venus publish-game dev
# Output: Published game with OneLink

# Step 3: Once tested, publish to production
venus publish-game prod
```

### Example 2: Updating an Existing Game

```bash
# If you have game.config.json (created by venus create-game):

# Create a new version with patch bump (1.0.0 → 1.0.1)
venus update-game --bump patch

# Publish the new version to dev for testing
venus publish-game dev

# After testing, publish to production
venus publish-game prod

# Create a new version with a minor bump (1.0.1 → 1.1.0)
venus update-game --bump minor
venus publish-game prod

# Create a new version with a major bump (1.1.0 → 2.0.0)
venus update-game --bump major
venus publish-game prod

# If you don't have game.config.json, specify manually:
venus update-game --id "abc123xyz" --path "./build/web" --bump patch
```

### Example 3: Typical Development Workflow

```bash
# Initial setup: Create your game
venus create-game --name "My Awesome Game" --path "./dist"
# This creates game.config.json automatically

# Publish to dev for initial testing
venus publish-game dev

# Make changes to your game...
# Build your game to ./dist

# Create a new version - super simple!
venus update-game --bump patch
# Uses game.config.json, defaults to patch bump (1.0.0 → 1.0.1)

# Publish to dev for testing
venus publish-game dev

# After testing, publish to production
venus publish-game prod

# Major feature release
venus update-game --bump minor  # 1.0.1 → 1.1.0
venus publish-game dev          # Test first
venus publish-game prod         # Then go live

# Breaking changes
venus update-game --bump major  # 1.1.0 → 2.0.0
venus publish-game dev          # Test first
venus publish-game prod         # Then go live

# Check all your games
venus list-games
```

### Example 4: Multi-Environment Workflow

```bash
# Configure your environments
venus configure-env dev --base-url "https://dev-api.venus.com" --access-token "dev-token"
venus configure-env staging --base-url "https://staging-api.venus.com" --access-token "staging-token"
venus configure-env prod --base-url "https://api.venus.com" --access-token "production-token"

# List all environments
venus list-envs

# Create your game
venus create-game --name "My Game" --path "./dist"

# Publish to different environments for testing
venus publish-game dev       # Development testing
venus publish-game staging   # Staging/QA testing
venus publish-game prod      # Production release

# Update and publish to all environments
venus update-game --bump minor
venus publish-game dev
venus publish-game staging
venus publish-game prod
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
   - Or provide `--id` and `--path` options manually (for `update-game`)
   - Verify the `game.config.json` file is valid JSON
   - For `publish-game`, ensure `game.config.json` exists in the current directory

4. **"Game not found" or "Game has no version" error**
   - Ensure you've created the game using `venus create-game` first
   - Verify the game ID in `game.config.json` is correct
   - Make sure you've created at least one version using `venus update-game` before publishing

5. **"Env not found" error**
   - The environment you're trying to publish to hasn't been configured
   - Run `venus configure-env <env-name>` to set up the environment
   - Use `venus list-envs` to see all configured environments
   - Note: `prod` doesn't need to be configured, but other environments do

6. **Authentication errors**
   - Run `venus configure-env <env-name>` to update your credentials
   - Check if your access token has expired
   - Verify the API base URL is correct
   - Ensure you're using the correct environment with `venus use-env <env-name>`

7. **Version conflicts**
   - When updating a game, the version must be higher than the current version
   - Use appropriate bump type (major, minor, patch)

### Getting Help

- Check the command help: `venus --help`
- Check specific command help: `venus create-game --help`, `venus update-game --help`, `venus publish-game --help`, `venus configure-env --help`, etc.
- Review the error messages carefully - they often contain helpful information

## License

[Include license information here]

## Contributing

[Include contribution guidelines if applicable]
