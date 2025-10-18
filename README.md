# Venus CLI

A command-line interface tool for managing HTML5 games on the Venus platform. This CLI helps developers create, update, and manage their games efficiently.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Commands](#commands)
  - [games create](#games-create)
  - [games push-update](#games-push-update)
  - [games list](#games-list)
  - [configure](#configure)
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

### Manual Installation

If you prefer manual installation, download the latest release for your platform from the [Releases page](https://github.com/Zee-Series-AI/venus_cli_releases/releases):

- **macOS (Apple Silicon)**: `venus-macos-arm64.tar.gz`
- **macOS (Intel)**: `venus-macos-x64.tar.gz`  
- **Linux**: `venus-linux-x64.tar.gz`
- **Windows**: `venus-windows-x64.zip`

**Installation (macOS/Linux):**
```bash
# Extract the archive
tar -xzf venus-macos-arm64.tar.gz  # or your downloaded file

# Make executable (if needed)
chmod +x venus

# Move to PATH (optional)
sudo mv venus /usr/local/bin/

# Verify installation
venus --help
```

**Installation (Windows):**
1. Extract the zip file
2. Add the directory to your PATH or run directly from the folder
3. Run `venus --help` to verify

## Quick Start

The CLI works out of the box with your local development environment. No configuration needed!

```bash
# Create a new game (creates game.config.json automatically)
venus games create --name "My Game" --path "./dist"

# List your games
venus games list

# Update your game (uses game.config.json - no need to specify ID or path!)
venus games push-update
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

With this file, you can run `venus games push-update` without specifying `--id` or `--path`!

### games push-update

Updates an existing game with a new version.

```bash
# Using game.config.json (recommended - created by venus games create)
venus games push-update --bump patch

# Or specify manually
venus games push-update --id "your-app-id" --path "/path/to/game/dist" --bump patch
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
venus games create --name "Space Invaders HD" --path "./build/web"
# Output: App created with id 'abc123xyz'

# For production environment (configure first)
venus configure --api-base-url "https://api.venus.com" --api-access-token "my-secret-token"
venus games create --name "Space Invaders HD" --path "./build/web"
```

### Example 2: Updating an Existing Game

```bash
# If you have game.config.json (created by venus games create):

# Update with default patch version bump (1.0.0 → 1.0.1)
venus games push-update

# Update with a minor version bump (1.0.1 → 1.1.0)
venus games push-update --bump minor

# Update with a major version bump (1.1.0 → 2.0.0)
venus games push-update --bump major

# If you don't have game.config.json, specify manually:

# Update with patch version bump
venus games push-update --id "abc123xyz" --path "./build/web" --bump patch

# Override config file values
venus games push-update --id "different-id" --path "./different/path" --bump minor
```

### Example 3: Typical Development Workflow

```bash
# Initial setup: Create your game
venus games create --name "My Awesome Game" --path "./dist"
# This creates game.config.json automatically

# Make changes to your game...
# Build your game to ./dist

# Push updates - super simple!
venus games push-update
# Uses game.config.json, defaults to patch bump (1.0.0 → 1.0.1)

# Major feature release
venus games push-update --bump minor
# 1.0.1 → 1.1.0

# Breaking changes
venus games push-update --bump major
# 1.1.0 → 2.0.0

# Check all your games
venus games list
```

### Example 4: Multi-Environment Workflow

```bash
# Use environment variables for different environments
export VENUS_API_BASE_URL="https://staging-api.venus.com"
export VENUS_API_ACCESS_TOKEN="staging-token"

# Create app in staging
venus games create --name "My Game Beta" --path "./dist"

# Switch to production
export VENUS_API_BASE_URL="https://api.venus.com"
export VENUS_API_ACCESS_TOKEN="production-token"

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
   - Run `venus configure` to update your credentials
   - Check if your access token has expired
   - Verify the API base URL is correct

5. **Version conflicts**
   - When updating an app, the version must be higher than the current version
   - Use appropriate bump type (major, minor, patch)

### Getting Help

- Check the command help: `venus --help`
- Check games command help: `venus games --help`
- Check specific command help: `venus games create --help`
- Review the error messages carefully - they often contain helpful information

## For Maintainers

### Creating a Release

To create a new release with automatic builds for all platforms:

```bash
# Create and push a version tag
git tag v1.0.0
git push origin v1.0.0
```

The CI/CD pipeline will automatically:
- Build binaries for macOS (ARM64 & x64), Linux (x64), and Windows (x64)
- Create a GitHub release with all platform binaries
- Generate release notes

For detailed instructions, see [.github/RELEASE_GUIDE.md](.github/RELEASE_GUIDE.md).

## License

[Include license information here]

## Contributing

[Include contribution guidelines if applicable]
