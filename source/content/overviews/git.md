# Git 

## Basics of the Git File System

When we initialize a Git repository, Git creates a `.git` directory in the root of the repository. This directory contains all of the information that Git needs to track the history of the project. The `.git` directory contains the following:

-   **COMMIT_EDITMSG**: A text file that contains the commit message of the last commit.
-   **description**: A text file that contains a description of the repository.
-   **info**: A directory that contains a global exclude file.
-   **packed-refs**: A text file that contains a list of references that were updated by the `git prune` command.
-   **index**: A binary file that contains a sorted list of file names, file modes, and file hashes of the files that Git is currently tracking.
-   **logs**: A directory that contains logs from the `git reflog` command.
-   **branches**: A directory that contains files that store the commit IDs of the last commit of each branch in the repository.
-   **refs**: A directory that contains pointers to commits, called references.
-   **objects**: A directory that contains all of the objects in the repository.
-   **hooks**: A directory that contains scripts that are run before or after certain commands, such as `commit` or `push`.
-   **config**: A configuration file that contains project-specific configuration options for Git.
-   **HEAD**: A pointer to the current branch.

## Creating a local Git repository

To create a local Git repository, we use the `git init` command. This command creates a `.git` directory in the root of the repository. The `.git` directory contains all of the information that Git needs to track the history of the project.

```bash
git init
```

You can use `git init --bare` to create a bare repository. A bare repository is a repository that does not contain a working directory. Bare repositories are typically used as remote repositories.

```bash
git init --bare
```

## Setting up git config

Git uses a configuration file to store information about the repository. The configuration file is located in the `.git` directory. The configuration file contains the following sections:

-   **core**: Contains core settings for the repository.
-   **user**: Contains user-specific settings for the repository.
-   **commit**: Contains commit-specific settings for the repository.
-   **remote**: Contains remote-specific settings for the repository.
-   **branch**: Contains branch-specific settings for the repository.
-   **alias**: Contains aliases for Git commands.

You can use the `git config` command to set configuration options for the repository. The `git config` command takes the following arguments:

-   **--global**: Sets the configuration option globally.
-   **--local**: Sets the configuration option locally.
-   **--system**: Sets the configuration option system-wide.
-   **--unset**: Unsets the configuration option.
-   **--get**: Gets the configuration option.
-   **--get-all**: Gets all values for the configuration option.
-   **--add**: Adds a value to the configuration option.
-   **--replace-all**: Replaces all values for the configuration option.
-   **--list**: Lists all configuration options.
-   **--show-origin**: Shows the origin of the configuration option.
-   **--show-scope**: Shows the scope of the configuration option.
-   **--show-origin**: Shows the origin of the configuration option.

A mandatory task is to set the user name and email address for the repository. You can use the following commands to set the user name and email address for the repository:

```bash
git config --global user.name "YOUR-USERNAME"
git config --global user.email "your-email@email.com"
```
Some people prefer to set the default text editor for Git. You can use the following command to set the default text editor for Git (replace `/usr/bin/vim` with the path to your preferred text editor):

```bash
git config --global core.editor "/usr/bin/vim"
```

Using `git config` will edit a file named .gitconfig in your home directory. You can also edit this file directly. Here is a sample .gitconfig file:

```bash
[user]
    name = YOUR-USERNAME
    email = YOUR-EMAIL-ADDRESS

[core]
    editor = /usr/bin/vim
```

To quickly get an overview of all the settings and where they are coming from, you can use the `git config --list --show-origin` command.

## Managing files in a project

To add files to a project, we use the `git add` command. The `git add` command takes the following arguments:

-   **-A**: Adds all files to the project.
-   **-f**: Adds files that are ignored by Git.
-   **-i**: Adds files interactively.
-   **-n**: Adds files without actually adding them.
-   **-p**: Adds files interactively.
-   **-v**: Adds files verbosely.

You can create a `.keep` file in a directory to add the directory to the project. This is useful for empty directories that you want to add to the project.

```bash
touch src/.keep
git add src/.keep
git commit -m "Add empty src directory"
```

Files can be removed from a project by using the `git rm` command. The `git rm` command takes the following arguments:

-   **-f**: Removes files that are ignored by Git.
-   **-r**: Removes files recursively.
-   **-n**: Removes files without actually removing them.
-   **-cached**: Removes files from the index without removing them from the working tree.

## Using Git Status

You can see the status of the files in the project by using the `git status` command. The `git status` command takes the following arguments:

-   **-s**: Shows the status of the files in the project in short format.
-   **-b**: Shows the status of the files in the project in long format.
-   **-v**: Shows the status of the files in the project in verbose format.

An example of a status message is:

```bash
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.MD

no changes added to commit (use "git add" and/or "git commit -a")
```

## Removing Sensitive Data Using `git filter-repo`

This guide will walk you through the process of removing a sensitive file from your repository's history. We will also add the file to `.gitignore` to prevent accidental re-committing.

:warning: Precautions :warning:

Before running `git filter-repo`, ensure that you have no stashed changes. Stashed changes can't be retrieved after running this command. To unstash the last set of changes, execute:

```bash
git stash show -p | git apply -R
```

### Step 1: Install `git filter-repo`

1.  **Manual Installation**: Visit [git filter-repo](https://github.com/newren/git-filter-repo) for the latest release.
    
2.  **Homebrew Installation**: Run the following command:
    

```bash
brew install git-filter-repo
```

For more installation options, check [INSTALL.md](https://github.com/newren/git-filter-repo/blob/main/INSTALL.md).

### Step 2: Clone the Repository

If you don’t already have a local copy of the repository, clone it:

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
```

### Step 3: Navigate to the Repository

```bash
cd YOUR-REPOSITORY
```

### Step 4: Run `git filter-repo`

Execute the following command, substituting `PATH-TO-YOUR-FILE-WITH-SENSITIVE-DATA` with the specific path to the sensitive file:

```bash
git filter-repo --invert-paths --path PATH-TO-YOUR-FILE-WITH-SENSITIVE-DATA
```

> **Note**: If the file has existed at other paths in the past due to renaming or moving, you must run the command for those paths as well.

### Step 5: Update `.gitignore`

Prevent future commits of the sensitive file:

```bash
echo "YOUR-FILE-WITH-SENSITIVE-DATA" >> .gitignore
git add .gitignore
git commit -m "Add sensitive file to .gitignore"
```

### Step 6: Verify the Changes

Review your repository to ensure the sensitive data has been removed from its history.

### Step 7: Push the Changes to GitHub

After confirming that the repository's history is as you desire, force-push your changes:

```bash
git push origin --force --all
``` 

### Step 8: Update Tags

If you have tags, you'll need to force-push those as well:

```bash
git push origin --force --tags
```
