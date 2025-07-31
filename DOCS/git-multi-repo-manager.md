# Git Multi-Repository Manager

A simple yet powerful tool to manage multiple Git repositories from a single command line interface.

## Features

- Commit changes across multiple repositories with one command
- Support for both full paths and project numbers
- Clear visual feedback for each repository
- Easy to configure and extend
- Cross-platform support (Windows/Linux/Mac)

## Setup Instructions

### 1. Basic Setup (Manual)

1. Edit your `~/.bash_aliases` (Linux/Mac) or `%USERPROFILE%\.bash_aliases` (Windows Git Bash):

```bash
# Define your project directories
export PROJECTS=(
    "$HOME/Documents/your/project1"
    "$HOME/Documents/your/project2"
    # Add more projects as needed
)

# Git All - commit changes in all projects
gta() {
    if [ $# -eq 0 ]; then
        echo "Usage: gta \"commit message\" [project1] [project2] ..."
        echo "Available projects:"
        for i in "${!PROJECTS[@]}"; do 
            echo "  $((i+1)). ${PROJECTS[$i]}"
        done
        return 1
    fi

    local message="$1"
    shift
    local projects_to_commit=("$@")
    
    if [ ${#projects_to_commit[@]} -eq 0 ]; then
        projects_to_commit=("${PROJECTS[@]}")
    fi

    for project in "${projects_to_commit[@]}"; do
        if [[ $project =~ ^[0-9]+$ ]]; then
            local idx=$((project-1))
            if [ $idx -ge 0 ] && [ $idx -lt ${#PROJECTS[@]} ]; then
                project="${PROJECTS[$idx]}"
            else
                echo "Invalid project number: $project"
                continue
            fi
        fi

        if [ -d "$project/.git" ]; then
            echo -e "\n\033[1;34mProcessing: $project\033[0m"
            (cd "$project" && git add . && git commit -m "$message" && git push)
        else
            echo -e "\n\033[1;31mNot a git repository or directory doesn't exist: $project\033[0m"
        fi
    done
}
```

2. Reload your shell configuration:
   ```bash
   source ~/.bash_aliases  # or source ~/.bashrc
   ```

### 2. Windows Setup Script

Create a file named `setup_git_manager.bat` in your project root:

```batch
@echo off
setlocal enabledelayedexpansion

:: Create .bash_aliases if it doesn't exist
if not exist "%USERPROFILE%\.bash_aliases" (
    echo Creating .bash_aliases...
    echo # Git Multi-Repository Manager Configuration > "%USERPROFILE%\.bash_aliases"
)

echo Adding Git Multi-Repository Manager configuration...
echo. >> "%USERPROFILE%\.bash_aliases"
echo # Git Multi-Repository Manager Configuration >> "%USERPROFILE%\.bash_aliases"
echo export PROJECTS^(^)='^(' >> "%USERPROFILE%\.bash_aliases"
echo     "%%USERPROFILE%%/Documents/your/project1" >> "%USERPROFILE%\.bash_aliases"
echo     "%%USERPROFILE%%/Documents/your/project2" >> "%USERPROFILE%\.bash_aliases"
echo ^')' >> "%USERPROFILE%\.bash_aliases"
echo. >> "%USERPROFILE%\.bash_aliases"

type "%~dp0git_manager_functions.sh" >> "%USERPROFILE%\.bash_aliases"

echo Configuration complete. Please restart your Git Bash or run: source %%USERPROFILE%%/.bash_aliases
```

### 3. Linux/Mac Setup Script

Create a file named `setup_git_manager.sh` in your project root:

```bash
#!/bin/bash

# Create .bash_aliases if it doesn't exist
touch ~/.bash_aliases

# Check if configuration already exists
if grep -q "Git Multi-Repository Manager" ~/.bash_aliases; then
    echo "Configuration already exists in ~/.bash_aliases"
    exit 1
fi

echo "# Git Multi-Repository Manager Configuration" >> ~/.bash_aliases
echo "export PROJECTS=(" >> ~/.bash_aliases
echo "    \"$HOME/Documents/your/project1\"" >> ~/.bash_aliases
echo "    \"$HOME/Documents/your/project2\"" >> ~/.bash_aliases
echo ")" >> ~/.bash_aliases
echo "" >> ~/.bash_aliases

# Append the function definition
cat "$(dirname "$0")/git_manager_functions.sh" >> ~/.bash_aliases

echo "Configuration complete. Please run: source ~/.bash_aliases"
```

## Usage Examples

1. **List available projects** (shows all configured repositories):
   ```bash
   gta
   ```

2. **Commit changes in all projects**:
   ```bash
   gta "Update all projects with latest changes"
   ```

3. **Commit changes in specific projects by number**:
   ```bash
   gta "Update first two projects" 1 2
   ```

4. **Commit changes in specific projects by path**:
   ```bash
   gta "Update specific projects" /path/to/project1 /path/to/project2
   ```

## Best Practices

1. **Backup your .bash_aliases** before making changes
2. **Test with a dry run** before committing to all repositories
3. **Keep your project list updated** in the `PROJECTS` array
4. **Use meaningful commit messages** that describe changes across all affected repositories

## Troubleshooting

- If you get "command not found", make sure to source your .bash_aliases:
  ```bash
  source ~/.bash_aliases
  ```

- On Windows, ensure Git Bash is using the correct .bash_aliases file

- If you encounter permission issues, make the script executable:
  ```bash
  chmod +x setup_git_manager.sh
  ```

## License

This tool is provided as-is under the MIT License.
