# Git

[TOC]

## Introduction

> *Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.*
>
> [Git (git-scm.com)](https://git-scm.com/)

Git is a distribute version control system designed to efficiently manage and track changes to files and directories.

## Architecture of Git

<img src="assets/Git Diagram.svg" alt="Git Diagram" style="zoom: 33%;" />

* **Remote Repository**: A remote repository is a separate copy of the repository that exists on a remote server. It allows multiple developers to collaborate on the same project by sharing changes with each other. Developers can push their local commits to the remote repository and fetch or pull updates from it.
* **Local Repository**: The repository is the central component of Git and contains the complete history and metadata of the project. It is usually divided into two main parts:
  * Object Database: The object database stores all the data and content of the project's files. It uses a content-addressable storage model, where each file and version is identified by a unique SHA-1 hash. Git objects include blobs (file contents), trees (directory structures), commits (snapshots of the project at a specific point in time), and tags (named references to specific commits).
  * Reference Database: The reference database holds references to specific commits within the object database. It includes branch references, which are pointers to specific commits on a branch, and tags, which are human-readable names assigned to specific commits.

* **Index (Staging Area)**: is an intermediate area where changes to the project's files are prepared to be committed.
* **Working Directory**: is the directory on a developer's local machine where they make modifications to the project's files. It represents the current state of the codebase and includes all the files and directories.

## Command
|Command|Meaning|
|:---:|:---|
|`init`| initializes a new Git repository in the current directory.|
|`clone`|Create a copy of a remote repository on your local machine.|
|`add`|Adds changes or new files to the staging area for the next commit.|
|`commit`|Records the changes mode to the repository and creates a new commit.|
|`push`|Upload local commits to a remote repository.|
|`pull`|Fetches changes from a remote repository and merges them into the current branch.|
|`rm`|Removes files from the working directory and stages the removal.|
|`diff`|shows the differences between the working directory and the staging area or previous commit|

## Branch & Gitflow Workflow

Branches are independent lines of development within the repository. They allow developers to work on different features or bug fixes simultaneously. Each branch has its own commit history and can be merged with other branches to incorporate changes.

* Long-term branch
  * `Master`
  * `develop`: the development branch.
* Temporal branch
  * `release`
  * `feature`: new requirement branch.
  * `bugfix`: bug repairment branch.
  * `hotfix`: urgent bug repairment branch.
  * `patch`: The bug repairment batch in the test process. pull from the `feature`.

<img src="assets/git-flow-nvie.png" alt="img" style="zoom: 25%;" />

> [Gitflow Workflow | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
>
> [Read Git Flow | Leanpub](https://leanpub.com/git-flow/read)

### Main & Develop branches

<img src="assets/01 How it works.svg" alt="01 How it works" style="width: 400px;" />

Instead of a single `main` (`master`) branch, this workflow uses two branches to record the history of the project. The `main` branch stores the official release history, and the `develop` branch serves as an integration branch for features. It's also convenient to tag all commits in the `main` branch with a version number.

```shell
git branch develop
git push -u origin develop
```

### Feature branches

<img src="assets/02 Feature branches-1689320359046-12.svg" alt="02 Feature branches" style="width: 400px;" />

Each new feature should reside in its own branch, which can be pushed to the central repository for backup/collaboration. But, instead of branching off of main, feature branches use develop as their parent branch. When a feature is complete, it gets merged back into develop. Features should never interact directly with main.

```shell
# Creating a feature branch
git checkout develop
git checkout -b feature_branch

# Finishing a feature branch
git checkout develop
git merge feature_branch
git branch -d feature_branch
```

### Release branches

<img src="assets/03 Release branches.svg" alt="03 Release branches" style="width: 400px;" />

Once develop has acquired enough features for a release (or a predetermined release date is approaching), you fork a release branch off of develop. Creating this branch starts the next release cycle, so no new features can be added after this point—only bug fixes, documentation generation, and other release-oriented tasks should go in this branch. Once it's ready to ship, the release branch gets merged into main and tagged with a version number. In addition, it should be merged back into develop, which may have progressed since the release was initiated.

```shell
# Creating a release branch
git checkout develop
git checkout -b release/0.1.0

# Finishing a release branch
git checkout main
git merge release/0.1.0
```

### Hotfix branches

<img src="assets/04 Hotfix branches.svg" alt="04 Hotfix branches" style="width: 400px;" />

Maintenance or `“hotfix”` branches are used to quickly patch production releases. `Hotfix` branches are a lot like `release` branches and `feature` branches except they're based on `main` instead of `develop`. This is the only branch that should fork directly off of `main`. As soon as the fix is complete, it should be merged into both `main` and `develop` (or the current `release` branch), and `main` should be tagged with an updated version number.

```shell
# Creating a hotfix branch
git checkout main
git checkout -b hotfix_branch

# Finishing a hotfix branch
git checkout main
git merge hotfix_branch
git checkout develop
git merge hotfix_branch
git branch -D hotfix_branch
```

## Conflict Resolution

- **Identify the conflict & Open the conflicting file(s)**: When you attempt to merge or rebase branches and encounter a conflict, Git will notify you of the conflicting files. Use a text editor or an integrated development environment (IDE) to open the files with conflicts. In the file, Git will mark the conflicting sections with special markers, such as `<<<<<<<`, `=======`, and `>>>>>>>`.

  ```
  <<<<<<< HEAD
  // Code from current branch
  =======
  // Code from merged branch
  >>>>>>> feature_branch
  ```

- **Understand & Resolve the conflict**: Analyze the conflicting sections and understand the changes made in each branch. Edit the conflicting file(s) to manually choose the desired changes. You can modify the code to include the changes from both branches or discard some changes altogether.

  - **Remove conflict markers**: Remove the conflict markers (`<<<<<<<`, `=======`, and `>>>>>>>`) and any unnecessary code added by Git during the conflict resolution process.
  - **Choose desired changes**: Modify the code to select the changes you want to keep. This may involve merging or modifying the conflicting sections to combine the changes or discarding one set of changes entirely.

- **Add the resolved file(s) & Commit the changes**: Once you've finished resolving conflicts, use the `git add` to stage the resolved file(s) for the next commit. Use `git commit` to create a new commit that includes the conflict resolution changes. It's helpful to provide a meaningful commit message that describes the conflict resolution.

  ```shell
  git add <冲突文件1> <冲突文件2> ...
  git commit
  ```

- **Repeat if necessary**: If there are additional conflicts in other files, go back to step 2 and resolve them until all conflicts are resolved.
- **Complete the merge or rebase & Push or continue with your workflow**: Once all conflicts are resolved, continue the merge or rebase operation using `git merge --continue` or `git rebase --continue`. After resolving the conflicts and completing the merge or rebase, you can ```git push``` the changes to the remote repository or continue with your workflow as desired. 

