# Git Multi-Repository Manager - Advanced Usage

**Last Updated**: 2025-07-31  
**Location**: `~/.bash_aliases`

## Overview

This document details the advanced usage of the Git Multi-Repository Manager, a bash function that allows you to manage multiple git repositories from a single command line interface.

## Features

- **Status Check**: View git status across all repositories
- **Batch Committing**: Commit changes to multiple repositories with one command
- **Flexible Project Selection**: Use project numbers or paths
- **Error Handling**: Graceful handling of non-git directories

## Setup

### 1. Configuration

Edit your `~/.bash_aliases` file to include the following:

```bash
# Define your project directories
export PROJECTS=(
    "$HOME/Documents/SITES/LILAC/L317/app/public/wp-content/plugins/simple-teacher-dashboard"
    "$HOME/Documents/SITES/LILAC/L317/app/public/wp-content/plugins/school-manager-lite"
    "$HOME/Documents/SITES/LILAC/L317/app/public/wp-content/plugins/lilac-quiz-sidebar"
    "$HOME/Documents/SITES/LILAC/L317/app/public/wp-content/themes/hello-theme-child-master"
)

# Git All - manage multiple repositories
gta() {
    # Handle status command
    if [ "$1" = "status" ]; then
        shift
        for project in "${PROJECTS[@]}"; do
            if [ -d "$project/.git" ]; then
                echo -e "\n\033[1;34mRepository: $project\033[0m"
                (cd "$project" && git status -sb)
                echo "----------------------------------------"
            else
                echo -e "\n\033[1;31mNot a git repository: $project\033[0m"
            fi
        done
        return 0
    fi
    
    # Handle commit command (original functionality)
    if [ $# -eq 0 ]; then
        echo "Usage:"
        echo "  gta status"
        echo "  gta \"commit message\" [project1] [project2] ..."
        echo "\nAvailable projects:"
        for i in "${!PROJECTS[@]}"; do 
            echo "  $((i+1)). ${PROJECTS[$i]}"
        done
        return 1
    fi

    local message="$1"
    shift
    local projects_to_commit=("$@")
    
    # If no specific projects provided, use all projects
    if [ ${#projects_to_commit[@]} -eq 0 ]; then
        projects_to_commit=("${PROJECTS[@]}")
    fi

    for project in "${projects_to_commit[@]}"; do
        # Check if project is a number (index)
        if [[ $project =~ ^[0-9]+$ ]]; then
            local idx=$((project-1))
            if [ $idx -ge 0 ] && [ $idx -lt ${#PROJECTS[@]} ]; then
                project="${PROJECTS[$idx]}"
            else
                echo "Invalid project number: $project"
                continue
            fi
        fi

        # Check if directory exists and is a git repo
        if [ -d "$project/.git" ]; then
            echo -e "\n\033[1;34mProcessing: $project\033[0m"
            (cd "$project" && git add . && git commit -m "$message" && git push)
        else
            echo -e "\n\033[1;31mNot a git repository or directory doesn't exist: $project\033[0m"
        fi
    done
}
```

### 2. Apply Changes

After updating `.bash_aliases`, apply the changes by running:

```bash
source ~/.bash_aliases
```

## Usage Examples

### 1. Check Status of All Repositories

```bash
gta status
```

### 2. Commit Changes to All Repositories

```bash
gta "Update all repositories with latest changes"
```

### 3. Commit Changes to Specific Repositories by Number

```bash
gta "Update specific repositories" 1 3
```

### 4. Commit Changes to Specific Repositories by Path

```bash
gta "Update specific repositories" /path/to/repo1 /path/to/repo2
```

## Best Practices

1. **Regular Status Checks**: Use `gta status` frequently to monitor changes across repositories
2. **Focused Commits**: Commit related changes together with a clear message
3. **Selective Updates**: Only update repositories that need changes
4. **Backup**: Keep regular backups of your `.bash_aliases` file

## Troubleshooting

### Common Issues

1. **Command not found**
   - Ensure `.bash_aliases` is sourced in your `.bashrc` or `.bash_profile`
   - Run `source ~/.bash_aliases` after making changes

2. **Permission denied**
   - Ensure you have execute permissions on the script
   - Run `chmod +x ~/.bash_aliases` if needed

3. **Repository not found**
   - Verify the paths in the PROJECTS array are correct
   - Ensure the repositories exist and are git repositories

## Version History

### v1.1 (2025-07-31)
- Added `gta status` command
- Improved error handling
- Enhanced documentation

### v1.0 (Initial Release)
- Basic multi-repo commit functionality
- Project selection by number or path
- Basic error handling
