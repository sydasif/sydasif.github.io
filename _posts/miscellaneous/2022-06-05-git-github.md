---
title:  "Getting Started with Git and GitHub"
date:   2022-06-05 15:02:15 +0500
categories: miscellaneous
tags:
  - git
  - github
---

## What is Git?
Git is a distributed version control system _(VCS) that maintains a history of changes to files for reference and rolls back a change_. It is software for tracking changes in any set of files, usually used for coordinating work among programmers who are collaborating on the same source code, and widely used, released under GNU GPL v2 license.

This blog focuses on:

- How to start using Git and GitHub
- How to perform basic setup
- How to view information and/or changes

Git can:

- track changes in ﬁle
- store multiple versions of the same ﬁle
- cancel changes made
- record who made changes and when.

Git was developed in 2005 to maintain all data, history, file revisions, etc locally. Every user has a copy of their own entire clone repository and allows each node to be a full clone of the entire historical record, not on the main server.

## What is GitHub?

GitHub on the other hand the web-based cloud offering of Git. These services use Git in the background to offer a cloud / online repository to store your code securely so that it’s accessible to everyone, no matter where they are and they can collectivity contribute to the same source code. GitHub also has private repositories, the main use of GitHub is to place code for various projects.

## Installing Git and Initializing Repository

Download [GitHub](https://git-scm.com/downloads) command-line tool, using the command line is better for cross-platform transformation because command lines are the same for Mac-OS, Linux, and Windows.

### Git installation on Ubuntu Linux

```console
sudo apt install git
```

### Configuring Git

To start working with Git you need to specify the user name and e-mail that will be used to synchronize the local repository with your GitHub repository:

```console
git config --global user.name “userName”
git config --global user.email “userEmail@gmail.com”
git config --global core.editor "vim"
```

or use `git config --global --edit` will open _.gitconfig_ file in your text editor.

```vim
  1 [user]
  2         name = “userName”
  3         email = “userEail@gmail.com”
  4 [core]
  5         editor = vim
```

To check your Git setting use the command:

```console
git config --list
```

or

```console
git config --global --list
```

### Initializing Repository

There are two ways to initialize a repository:

- The first one is to clone an existing repository from a server to your local machine.
- The second is to create a new repository on your machine locally.

## Fundamentals of Git

Git has maintains a file systems also known as git structures or trees as below:

- ### Working directory / local workspace

Starting with the working area, the working area is where the files that are not handled by git, it is the place where all the untracked files are kept.

- ### Staging area / index

Staging area has files that are add by `git add` command, going to be part of the next commit.

- ### Local repository

The repository contains all the project and store all committed items.

- ### Remote repository

The remote repository contains your copy of local repo on a server.

## Staging and Committing

Files in Git follow a life cycle as below:

1. _Untracked:_  When you first create a file, Git sees this file but does not perform any operations on it.
2. _Staged:_ when you add a file in Git to track.
3. _Unmodified:_ When a file is committed.
4. _Modified:_ when you made a change in an unmodified file.

## Working with Git & Basic Commands

### File .gitignore

Git has the option to specify which ﬁle or directories to ignore. To do this, you need to create appropriate templates in the .gitignore ﬁle in the repository directory. To make Git ignore, you can add such a file name to .gitignore ﬁle.

### git init

Initialize a new local git repository inside a directory with `git init` or use `git init <name_of_dir>` will create a directory and also initialize a repository.

### git add

Command _git add_ is used to start Git tracking ﬁles. You can specify that you want to track a particular ﬁle.

```console
git add <file_name>
```

Or all ﬁles with the below command:

```console
git add .
```

### git status

check the status of a working directory from git’s perspective.

### git commit

After all necessary ﬁles have been added to a staging area, you can commit changes. Staging is a collection of ﬁles that will be added to the next commit.

```console
git commit -m "your commit message"
```

or use `git commit` to open a text editor for commit message.

### git show

The git show command will help to see details of a particular commit.

```console
git show 1c16945
```

### git rm

delete a git tracked file from a repository

### git mv

rename a git tracked file

## Additional features

### git diﬀ

Command _git diff_ allows you to see the difference between different states in the working area. Command _git diff_ shows what changes have been made since the last commit. If you add changes made to the staging area via _git add_ command and run _git diff_ again, it will show nothing.

Use the `git diff` to see the difference between two commits.

```console
git diff 1c16945..9dcff6
```

To show the difference between staging and last commit, add the parameter:

```console
git diff --staged
```

### Repository History

- The _git log_ command shows a history of the repository in descending order.
- The _git log_ command displays a brief description of a repository
    : it includes the change, date and time, who made the changes, commit message and commit identifier (hash value).
- You can tidy this history with the _git log --oneline_ command, which shows a clean-cut history.

There are many ways to see commit history :

Most basic one: Shows all the commits from end to start, with all the info related to commits.

```console
git log
```

To see some number of the last five commits:

```console
git log -5
```

To see commit from some commit to some other commit: We use the hash value of those commits.

```console
git log af156ff..32509d0
```

To see the stat of the commits:

```console
git log --stat
```

## Check out and Detached Head

Git checkout changes your working directory to match a specific commit and your git's head is in a detached head state, the Git repository is reverted and your work is temporarily lost means going back to the past.

The commands below are used:

```console
git checkout <first 7-bits of commit>
git checkout master
```

### Reset and Revert to Undo

To replace the last commit, _git commit --amend_ replaces/updates the last commit, if anything is present in the staging area will also commit and your previous commit has been replaced.

To reset, _git rest_ removes commits and resets the head. Don’t use _git commit –amend_ and _git reset_, these two commands will create a conflict if you are collaborating with a team in your remote repository.

## Git Branch

To create a new branch:

```console
git branch <branch-name>
```

When checkout to a branch make sure that the working area of the current branch should be clean.

```terminal
git checkout <branch-name>
```

This command is a mix of the above two commands creating and changing to a new branch.

```terminal
git checkout -b <branch-name>
```

Merging is to combine the commit history of two branches in a unified history and merge the branch after some changes with the master branch.

```terminal
git merge <branch-name>
```

To change the name of the branch, HEAD is currently pointing to:

```console
git branch -m newBranchName
```

To change the name of the branch irrespective of HEAD:

```console
git branch -m oldName newName
```

Delete the branch when work is complete.

```terminal
git branch -d <branch-name>
```

List all branches:

```terminal
git branch
```

or

```terminal
git branch -a
```

## GitHub Authentication

### With SSH

In order to start working with GitHub, you need to register on it. It is better to use SSH key authentication to work safely with GitHub.

Generation of a new SSH key (use an e-mail that is linked to GitHub):

```console
ssh-keygen
```

On all questions, just press Enter.

#### Add SSH key to GitHub

To add a key you have to copy it. For example, you can display a key with a cat to copy it:

```console
cat ~/.ssh/id_rsa.pub
```

After copying, go to GitHub, in the upper right-hand corner click on the picture of your pro-ﬁle and select “Settings” in the drop-down list. In list on the left, select “SSH and GPG keys”. Then press “New SSH key” and in, “Title” ﬁeld write key name (for example “my_laptop”) and in ﬁeld “Key” insert the content that was copied from ﬁle ~/. ssh/id_rsa.pub.

To check if everything has been successful, try executing the command ```ssh -T git@Github.com```.

The output should be as follows:

```console
ssh -T git@github.com
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### With Token

Open your browser and go to your GitHub account and generate the token as below:

```console
Settings -->> Developer settings -->> Personal access tokens -->>
Generate new token -->> Add note -->> Set Expiration -->> Add Scope and generate token.
```

Add token to your configuration with the below command:

```terminal
git config credential.helper store
```

or store globally in Git configuration

```terminal
git config --global credential.helper store
```

Copy your token to notepad and use it for authentication on GitHub.

## Working with GitHub

### Creating a GitHub repository

To create a GitHub repository you need to:

- log in to GitHub
- In the upper right corner press plus and select “New repository” to create a new repository
- Name of repository should be entered in a window that appears

You can put “Initialize this repository with a README”. This will create a README.md ﬁle that only
contains repository name.

### Cloning a GitHub repository

To work locally with a repository, it should be cloned. Use git clone command to clone the repository:

```console
git clone <url of repo>
```

As a result, in current directory in which git clone was executed, a directory with name of repository
will appear. Now you have a complete local Git repository where you can work. Typically, sequence of steps will be as follows:

- Before starting, synchronize local content with GitHub using git pull command
- Modifying repository ﬁles
- Adding modified ﬁles to staging with “git add” command
- Commit changes using git commit command
- Transferring local changes to GitHub repository with git push command

When working with tasks at work and at home, it is necessary to pay special attention to ﬁrst and
last step:

- The ﬁrst step is to update local repository
- The last step - load changes to GitHub

### Synchronizing local repository with GitHub

To synchronize a local repository with GitHub repository you need to:

- log in to GitHub
- In the upper right corner press plus and select “New repository” to create a new repository
- Name of repository should be entered in a window that appears

Make sure do not tick “Initialize this repository with a README” or any other file.

### git remote add

Now to push the changes from the local git repository on your machine to a remote git server, you need to tell git the location of that repository. This command doesn’t actually push your code to a remote repository, it only maps the location of the remote.

```console
git remote add origin <url>
```

### Push on GitHub

Command “git push” is used to load all local changes to GitHub:

```console
git push origin master
```

### git remote rm

Remove the remote repository from your git project. All commands are executed inside the repository directory.

```console
git remote remove origin
```

#### External Links

- [FancyGit](https://github.com/diogocavilha/fancy-git)
- [Using Git with VSCode](https://www.youtube.com/watch?v=i_23KUAEtUM&t=168s)
