# TMUX Sessionizer

A powerful, intelligent tmux session manager that streamlines your development workflow by quickly creating, switching between, and managing project-specific tmux sessions.

## Features

- **Intelligent Project Detection**: Automatically detects project types (Node.js, Go, Rust, PHP, Python, etc.) and sets up appropriate window layouts
- **Smart Session Switching**: Seamlessly switch between or create new sessions
- **Custom Session Templating**: Support for project-specific initialization through `.tmux-sessionizer` files
- **Fuzzy Directory Finding**: Quickly navigate your project directories with `fzf` integration
- **Project-Specific Layouts**: Different window arrangements based on the detected project type
- **Configurable Search Paths**: Easily customize which directories to include in your search
- **Cross-Platform Compatibility**: Works on macOS, Linux, and WSL environments
- **Performance Optimized**: Uses faster tools like `fd` when available
- **Comprehensive Logging**: Clear, color-coded status messages

## Installation

### Requirements

- tmux (2.1+)
- fzf (for directory selection)
- bash (4.0+)
- fd (optional, for faster directory finding)

### Installation Steps

1. Download the script:

```bash
curl -fsSL https://raw.githubusercontent.com/username/tmux-sessionizer/main/tmux-sessionizer -o /tmp/tmux-sessionizer
```

2. Install it to a location in your `PATH`:

```bash
# Create directory if it doesn't exist
mkdir -p ~/.local/bin

# Move and make executable
mv /tmp/tmux-sessionizer ~/.local/bin/
chmod +x ~/.local/bin/tmux-sessionizer
```

3. Make sure `~/.local/bin` is in your PATH:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
# Or for zsh users
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

## Usage

### Basic Usage

Launch tmux-sessionizer without arguments to select a project directory using fzf:

```bash
tmux-sessionizer
```

This will:

1. Search your project directories
2. Display a fuzzy-finder for selection
3. Create or switch to a tmux session named after the selected directory

### Direct Session Creation

Specify a directory path to immediately create/switch to that session:

```bash
tmux-sessionizer ~/dev/my-awesome-project
```

### Keyboard Shortcut (Recommended)

Add a keyboard shortcut to your shell configuration for quick access:

For Bash (add to `~/.bashrc`):

```bash
bind -x '"\C-f":"tmux-sessionizer"'
```

For Zsh (add to `~/.zshrc`):

```bash
bindkey -s '^f' 'tmux-sessionizer\n'
```

Now press `Ctrl+f` anytime to launch tmux-sessionizer.

## Configuration

### Custom Search Directories

Create a configuration file at `~/.tmux-sessionizer-config`:

```bash
# Define custom search directories
SEARCH_DIRS=(
  "$HOME/dev"
  "$HOME/work"
  "$HOME/personal"
  "$HOME/clients"
)

# Customize search depth (default is 3)
SEARCH_DEPTH=4
```

### Project-Specific Initialization

Create a `.tmux-sessionizer` file in your project directory with custom commands:

```bash
#!/usr/bin/env bash
# Example .tmux-sessionizer file for a web project

# Rename the current window
tmux rename-window -t 1 "editor"

# Create additional windows with specific layouts
tmux new-window -t "$session_name" -n "server"
tmux send-keys -t "$session_name:server" "cd $PWD && npm run dev" C-m

tmux new-window -t "$session_name" -n "test"
tmux send-keys -t "$session_name:test" "cd $PWD && npm run test:watch" C-m

# Create a window with split panes
tmux new-window -t "$session_name" -n "logs"
tmux split-window -v
tmux send-keys -t "$session_name:logs.0" "cd $PWD && tail -f logs/app.log" C-m
tmux send-keys -t "$session_name:logs.1" "cd $PWD && tail -f logs/error.log" C-m

# Return to editor window
tmux select-window -t "$session_name:editor"
tmux send-keys "vim ." C-m
```

### Global Default Initialization

If no project-specific file exists, the script will use `~/.tmux-sessionizer` as a fallback.

## Default Project Layouts

The script automatically detects project types and sets up appropriate window layouts:

| Project Type | Windows Created                   |
| ------------ | --------------------------------- |
| Node.js      | `edit`, `shell`, `server`, `test` |
| Go           | `edit`, `shell`, `run`, `test`    |
| Rust         | `edit`, `shell`, `build`, `test`  |
| PHP          | `edit`, `shell`, `server`, `test` |
| Python       | `edit`, `shell`, `run`, `test`    |
| Generic      | `edit`, `shell`                   |

## Troubleshooting

### Command Not Found

If you get `command not found: tmux-sessionizer`, make sure:

1. The script is executable: `chmod +x ~/.local/bin/tmux-sessionizer`
2. The location is in your PATH: `echo $PATH | grep -q "$HOME/.local/bin" && echo "In PATH" || echo "Not in PATH"`

### No Sessions Created

If sessions aren't being created correctly:

1. Check tmux is installed: `tmux -V`
2. Verify fzf is installed: `fzf --version`
3. Run with more debug output: `TMUX_SESSIONIZER_DEBUG=1 tmux-sessionizer`
