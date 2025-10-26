# Venus CLI

A command-line interface tool for managing HTML5 games on the Venus platform. This CLI helps developers create, update, and manage their games efficiently.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Commands](#commands)
  - [login](#login)
  - [create-game](#create-game)
  - [update-game](#update-game)
  - [publish-game](#publish-game)
  - [update-and-publish-game](#update-and-publish-game)
  - [list-games](#list-games)
  - [configure-game](#configure-game)
  - [use-env](#use-env)
  - [list-envs](#list-envs)
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
# 1. Login to Venus (required for authentication)
venus login

# 2. Create a new game (creates game.config.json automatically)
venus create-game --name "My Game" --path "./dist"

# 3. Publish your game to the selected environment
venus publish-game --env prod

# Or create new versions and publish
venus update-game --bump patch
venus publish-game --env prod

# List your games
venus list-games
```

## Configuration

### Authentication

Before using the CLI, you need to authenticate with your Venus account:

```bash
venus login
```

This will open a browser window for you to sign in with your Venus credentials. Your session will be saved locally and automatically refreshed when needed.

**Login Options:**
- `--force`: Force a new login even if you're already authenticated

### Environments

The CLI comes with pre-configured environments:
- **local**: For local development (`http://localhost:5001`)
- **dev**: Development environment
- **prod**: Production environment (default)

The default environment is `prod`. You can switch between environments or check which one is active.

#### Switching Between Environments

```bash
# Switch to dev environment
venus use-env dev

# Switch to production environment
venus use-env prod

# Switch to local environment
venus use-env local
```

#### Listing Environments

```bash
# See all available environments and which is active
venus list-envs
```

## Commands

### login

Authenticate with your Venus account.

```bash
venus login
```

**Options:**
- `--force`: Force a new login even if you're already authenticated

**What it does:**
1. Opens a browser window for authentication
2. Saves your session locally in `~/.venus_cli/`
3. Automatically refreshes your session when it expires

**Note:** You need to login before using commands like `create-game`, `update-game`, `publish-game`, and `list-games`.

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
# Publish to dev environment
venus publish-game --env dev

# Publish to production
venus publish-game --env prod

# Skip confirmation prompt
venus publish-game --env prod --yes
```

**Options:**
- `--env`: The environment name to publish to (e.g., `local`, `dev`, `prod`). If not specified, uses the active environment.
- `--yes`: Skip the confirmation prompt
- `--id`: Game ID to publish (optional, reads from `game.config.json` if not provided)

**What it does:**
1. Fetches your game information from `game.config.json` (or uses the `--id` option)
2. Gets the latest version of your game
3. Asks for confirmation (unless `--yes` is used)
4. Publishes that version to the specified environment
5. Returns a OneLink URL for accessing the game

**Important:**
- Uses `game.config.json` to determine which game to publish (if `--id` is not provided)
- If `--env` is not specified, uses the currently active environment (check with `venus list-envs`)
- You must be logged in to publish games

**Example workflow:**
```bash
# Create a new version with your latest changes
venus update-game --bump patch

# Publish to dev environment for testing
venus publish-game --env dev

# Once tested, publish to production
venus publish-game --env prod --yes
```

### update-and-publish-game

A convenience command that updates your game to a new version and immediately publishes it.

```bash
# Update with patch bump and publish to dev
venus update-and-publish-game --bump patch --env dev

# Update with minor bump and publish to prod (with confirmation skip)
venus update-and-publish-game --bump minor --env prod --yes

# Use defaults from game.config.json
venus update-and-publish-game
```

**Options:**
- `--id`: Game ID to update (optional, reads from `game.config.json` if not provided)
- `--path`: Path to game distribution folder (optional, reads from `game.config.json` if not provided)
- `--bump`: Version bump type - `major`, `minor`, or `patch` (default: `patch`)
- `--env`: Environment to publish to (default: active environment)
- `--yes`: Skip confirmation prompt

**What it does:**
1. Runs `update-game` with the specified options
2. If update succeeds, runs `publish-game` with the specified environment
3. Returns the OneLink URL for the published game

This is equivalent to running:
```bash
venus update-game --bump patch
venus publish-game --env dev
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
venus use-env dev
```

**Argument:**
- Environment name: The name of the environment to use (e.g., `local`, `dev`, `prod`)

### list-envs

List all available environments.

```bash
venus list-envs
```

Shows all pre-configured environments (local, dev, prod) and which one is currently active.

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
- Automatically detects your operating system (Windows, macOS, or Linux) and architecture (x64 or ARM64)
- Downloads the appropriate binary package for your platform
- Replaces your current installation with the new version
- Creates a backup of the old executable (.bak file)
- The update process is automatic and seamless

**Note:** The CLI also automatically checks for updates when you run any command and will notify you if a new version is available.

**Example output:**
```
Current version: 1.0.0
Latest version: 1.1.0
✅ Version 1.1.0 has been installed.
```

or if already up to date:

```
Current version: 1.1.0
Latest version: 1.1.0
You are already up to date.
```

## Environment Variables

Currently, the CLI does not use environment variables for configuration. All settings are managed through the `venus login` and `venus use-env` commands.

## Usage Examples

### Example 1: Creating and Publishing a New Game

```bash
# Step 1: Login to Venus
venus login
# Opens browser for authentication

# Step 2: Create your game
venus create-game --name "Space Invaders HD" --path "./build/web"
# Output: Game created with id 'abc123xyz'
# Creates game.config.json automatically

# Step 3: Publish to dev environment for testing
venus publish-game --env dev
# Output: Published game with OneLink

# Step 4: Once tested, publish to production
venus publish-game --env prod
```

### Example 2: Updating an Existing Game

```bash
# If you have game.config.json (created by venus create-game):

# Create a new version with patch bump (1.0.0 → 1.0.1)
venus update-game --bump patch

# Publish the new version to dev for testing
venus publish-game --env dev

# After testing, publish to production
venus publish-game --env prod

# Create a new version with a minor bump (1.0.1 → 1.1.0)
venus update-game --bump minor
venus publish-game --env prod

# Create a new version with a major bump (1.1.0 → 2.0.0)
venus update-game --bump major
venus publish-game --env prod

# If you don't have game.config.json, specify manually:
venus update-game --id "abc123xyz" --path "./build/web" --bump patch
venus publish-game --id "abc123xyz" --env prod
```

### Example 3: Typical Development Workflow

```bash
# Initial setup: Login
venus login

# Create your game
venus create-game --name "My Awesome Game" --path "./dist"
# This creates game.config.json automatically

# Publish to dev for initial testing
venus publish-game --env dev

# Make changes to your game...
# Build your game to ./dist

# Create a new version - super simple!
venus update-game --bump patch
# Uses game.config.json, defaults to patch bump (1.0.0 → 1.0.1)

# Publish to dev for testing
venus publish-game --env dev

# After testing, publish to production
venus publish-game --env prod

# Major feature release
venus update-game --bump minor        # 1.0.1 → 1.1.0
venus publish-game --env dev          # Test first
venus publish-game --env prod         # Then go live

# Breaking changes
venus update-game --bump major        # 1.1.0 → 2.0.0
venus publish-game --env dev          # Test first
venus publish-game --env prod         # Then go live

# Check all your games
venus list-games
```

### Example 4: Multi-Environment Workflow

```bash
# Login first
venus login

# List all available environments
venus list-envs

# Create your game
venus create-game --name "My Game" --path "./dist"

# Publish to different environments for testing
venus publish-game --env dev        # Development testing
venus publish-game --env prod       # Production release

# Update and publish to all environments
venus update-game --bump minor
venus publish-game --env dev --yes
venus publish-game --env prod --yes

# Or use the combined command for quick iterations
venus update-and-publish-game --bump patch --env dev --yes
```

### Example 5: Using the Combined Update and Publish Command

```bash
# Quick iteration: update and publish in one command
venus update-and-publish-game --bump patch --env dev

# With confirmation skip for automated workflows
venus update-and-publish-game --bump minor --env prod --yes

# Use defaults from game.config.json and active environment
venus use-env dev
venus update-and-publish-game
```

## Troubleshooting

### Common Issues

1. **"Session expired" or authentication errors**
   - Run `venus login` to authenticate
   - If already logged in, try `venus login --force` to force a new login session
   - Your session is automatically refreshed, but if you encounter issues, re-login

2. **"Failed to upload file" error**
   - Check your internet connection
   - Ensure you're logged in with `venus login`
   - Verify the game distribution folder exists and is not empty

3. **"Game dist folder does not exist" error**
   - Verify the path to your game's build folder is correct
   - Ensure you're using the full path or correct relative path
   - Check that the path in `game.config.json` is correct if using auto-detection

4. **"Unable to load game config" error**
   - Make sure you're running the command from the directory containing `game.config.json`
   - Or provide `--id` and `--path` options manually (for `update-game`)
   - Verify the `game.config.json` file is valid JSON
   - For `publish-game`, ensure `game.config.json` exists in the current directory

5. **"Game not found" or "Game has no version" error**
   - Ensure you've created the game using `venus create-game` first
   - Verify the game ID in `game.config.json` is correct
   - Make sure you've created at least one version using `venus update-game` before publishing

6. **"Env not found" error**
   - The environment you're trying to use doesn't exist
   - Use `venus list-envs` to see all available environments (local, dev, prod)
   - Make sure you're using the correct environment name with `--env` option

7. **Version conflicts**
   - When updating a game, the version must be higher than the current version
   - Use appropriate bump type (major, minor, patch)

8. **Permission issues on macOS/Linux**
   - If you encounter permission errors during installation, make sure the installation directory is writable
   - You may need to use `sudo` for installation to `/usr/local/bin`

### Getting Help

- Check the command help: `venus --help`
- Check specific command help: `venus login --help`, `venus create-game --help`, `venus update-game --help`, `venus publish-game --help`, etc.
- Review the error messages carefully - they often contain helpful information
- Make sure you're on the latest version by running `venus update`

## License

[Include license information here]

## Contributing

[Include contribution guidelines if applicable]
