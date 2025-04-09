# tmux-sessionizer

A lightning-fast tmux session manager that simplifies project workflows with automatic environment configuration.

## Overview

`tmux-sessionizer` is a powerful Bash utility designed to streamline tmux workflow by:

- Quickly switching between projects
- Automatically creating consistent session layouts
- Providing project-specific environment configuration
- Eliminating repetitive setup tasks

The tool consists of two components:

1. `tmux-sessionizer` - The main script for creating and switching sessions
2. `.tmux-sessionizer` - A project-specific configuration script

## Prerequisites

- tmux 3.0+
- Bash 4.0+ or Zsh
- fzf (for interactive directory selection)
- fd (optional, for faster directory scanning)

## Installation

### 1. Install tmux-sessionizer

```bash
# Create bin directory if it doesn't exist
mkdir -p ~/.local/bin

# Download the script
curl -o ~/.local/bin/tmux-sessionizer https://raw.githubusercontent.com/thenameiswiiwin/tmux-sessionizer/main/tmux-sessionizer

# Make it executable
chmod +x ~/.local/bin/tmux-sessionizer

# Ensure ~/.local/bin is in your PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc  # or ~/.bashrc
```

### 2. Install .tmux-sessionizer

```bash
# Download the configuration script
curl -o ~/.tmux-sessionizer https://raw.githubusercontent.com/thenameiswiiwin/dev-env/develop/.tmux-sessionizer

# Make it executable
chmod +x ~/.tmux-sessionizer
```

### 3. Set up keyboard shortcut

Add to your `.zshrc` or `.bashrc`:

```bash
# Add Ctrl+f shortcut for tmux-sessionizer
if command -v tmux-sessionizer &>/dev/null; then
    bindkey -s "^f" "tmux-sessionizer\n"  # Zsh
    # OR
    # bind -x '"\C-f":"tmux-sessionizer"'  # Bash
fi
```

## Basic Usage

### Starting a Session

- **Interactive**: Press `Ctrl+f` or run `tmux-sessionizer` with no arguments
- **Direct path**: `tmux-sessionizer ~/dev/myproject`

### During a Session

- **Switch projects**: Press `prefix + f` (typically `Ctrl+a f`) within tmux
- **Create windows**: Standard tmux commands after session is created

## Session Creation Process

1. **Directory Selection**: Choose a project directory
2. **Session Creation**: Creates tmux session named after directory
3. **Layout Configuration**: Automatically sets up project-specific windows
4. **Environment Hydration**: Sources `.tmux-sessionizer` for custom setup

## Project-Specific Configuration

Create a `.tmux-sessionizer` file in your project root for custom setup:

```bash
#!/usr/bin/env bash

# Project-specific setup goes here
echo "Setting up custom project environment..."

# Add commands to run in the session
# Example: Starting development server in a window named "server"
if tmux has-session -t "$(tmux display-message -p '#S'):server" 2>/dev/null; then
    tmux send-keys -t "$(tmux display-message -p '#S'):server" "npm run dev" C-m
fi
```

## Project Type Detection

The `.tmux-sessionizer` script automatically detects project types and creates appropriate window layouts:

| Project Type | Detection Method | Windows Created           |
| ------------ | ---------------- | ------------------------- |
| Node.js      | package.json     | edit, shell, server, test |
| Go           | go.mod           | edit, shell, run, test    |
| Rust         | Cargo.toml       | edit, shell, build, test  |
| PHP          | composer.json    | edit, shell, server, test |
| Python       | requirements.txt | edit, shell, run, test    |
| Generic      | (fallback)       | edit, shell               |

## Advanced Features

### Window Management

The `.tmux-sessionizer` script provides several helper functions:

- **window_exists**: Check if a window exists in a session
- **create_window_if_missing**: Create a window if it doesn't exist
- **send_command**: Send a command to a specific window

### Custom Project Layouts

You can create project-specific layouts by modifying the `.tmux-sessionizer` file:

```bash
#!/usr/bin/env bash

# Skip for home directory
if [[ "$PWD" == "$HOME" ]]; then
  return 0
fi

# Create a custom layout
session_name=$(tmux display-message -p '#S')

# Create windows
tmux new-window -t "$session_name" -n "database"
tmux new-window -t "$session_name" -n "logs"

# Set up commands
tmux send-keys -t "$session_name:database" "docker-compose up -d db" C-m
tmux send-keys -t "$session_name:logs" "tail -f logs/development.log" C-m

# Return to edit window
tmux select-window -t "$session_name:edit"
```

### Theme Integration

The `.tmux-sessionizer` script respects your tmux theme settings (dark/light mode). To toggle themes, press `prefix + T`.

## Troubleshooting

### Common Issues

**Issue**: tmux-sessionizer not found
**Solution**: Ensure ~/.local/bin is in your PATH and the script is executable

**Issue**: Session not created
**Solution**: Check for error messages during session creation

**Issue**: Windows not being created
**Solution**: Verify that tmux is properly installed and the session name is valid

**Issue**: Custom configuration not being loaded
**Solution**: Check that `.tmux-sessionizer` is executable and contains valid Bash syntax

## Performance Optimization

For faster operation:

1. Install `fd` for faster directory scanning
2. Minimize file operations in your `.tmux-sessionizer` file
3. Use efficient window creation patterns
4. Avoid unnecessary commands in the script

## Customization Examples

### Custom Search Paths

Modify the `tmux-sessionizer` script to search different directories:

```bash
# Find in custom locations
selected=$(find ~/work ~/clients ~/experiments -mindepth 1 -maxdepth 1 -type d | fzf)
```

### Project Types

Add support for additional project types in `.tmux-sessionizer`:

```bash
# Laravel project
if [[ -f "$git_root/artisan" ]]; then
  echo "Setting up Laravel project environment"
  setup_project_windows "$session_name" "$git_root" "shell" "artisan" "routes" "test"
fi
```

## Tips & Tricks

1. **Fast Project Switching**: Use `Ctrl+f` outside tmux or `prefix + f` inside
2. **Quick Command Execution**: Use tmux send-keys for automating common tasks
3. **Multiple Monitors**: Create separate sessions for different displays
4. **Session Persistence**: tmux sessions remain after disconnect; reattach later

## Credits

- Original concept by ThePrimeagen
