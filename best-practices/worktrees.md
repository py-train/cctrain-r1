# Worktrees


## Git Worktrees

A Git worktree allows you to have multiple working directories attached to a single Git repository.
Normally, Git only lets you check out one branch at a time in your project folder. If you need to switch tasks, you have to either commit unfinished work or run git stash. With worktrees, you can check out separate branches into completely distinct folders on your computer at the exact same time, all sharing the same underlying .git history.


## 🧱 Why Use Git Worktrees? (The Core Benefits)

* Context Switching Without Stashing: If you are deep in a feature branch with hundreds of uncommitted changes and an urgent hotfix comes in, you don't need to stash or commit your messy code. You just spin up a hotfix worktree in a side folder, fix the bug, and delete it.
* Simultaneous Testing & Running: You can run and test your main production branch in one terminal window while actively writing code on a feature branch in another terminal window.
* No Redundant Cloning: Instead of cloning the entire repository a second time (which wastes disk space and requires separate network fetches), worktrees share the primary .git folder.


## 🕹️ The Essential Commands

Here is the entire lifecycle of managing worktrees natively in Git:

### 1. Create a New Worktree
To create a new folder and check out a branch into it:

`git worktree add ../my-feature-folder -b feature-branch`


* `../my-feature-folder`: The path where the new folder will be created (it's best to put it outside your main repository directory so you don't nest them).
* `-b feature-branch`: Creates and checks out a brand new branch inside that folder.

### 2. View Active Worktrees
To see all the folders currently linked to your repository and which branches they are on:

`git worktree list`

### 3. Working Inside the Worktree
Simply navigate into the new folder using your terminal or open it in your IDE (like VS Code):

`cd ../my-feature-folder`

From here, you can stage, commit, push, and pull just like normal. Git automatically knows it is linked to your main repository.

### 4. Clean Up and Delete
Once you are done, your code is committed/pushed, and you want to get rid of the extra directory:

### From your main repository folder:
`git worktree remove ../my-feature-folder`

This safely deletes the folder and tells Git to stop tracking that worktree. Your Git branch remains safe in your history.


## ⚠️ Rules and Gotchas to Keep in Mind

* No Duplicate Branches: Git strictly forbids checking out the same exact branch in two different worktrees at the same time. If a branch is active in Worktree A, you cannot check it out in Worktree B.
* Isolated Dependencies: Because a worktree is a physically separate folder, build artifacts and local configuration files (like node_modules/, .env files, or Python virtual environments) are not copied over automatically. You will need to re-run your package installs (e.g., npm install) inside the new worktree folder.

# Merging Worktrees

To merge the changes made inside a Claude Code worktree back into your main branch, you need to treat the worktree's branch just like any standard Git branch. Because Claude creates the worktree folder inside .claude/worktrees/<branch-name>, you can perform the merge either from your main project directory or via a Pull Request.

Here is the step-by-step process:

## Step 1: Ensure Claude is Finished

Make sure the Claude session operating in that worktree has completed its task and you have exited the interactive prompt (Ctrl+D or typing exit). This ensures Claude isn't actively writing files while you try to move or merge them.

## Step 2: Identify the Branch Name
If you didn't explicitly name the branch when running Claude, find its name by running this command from your main repository directory:

`git worktree list`

This will display a list of active worktrees and their associated branches (e.g., .claude/worktrees/claude-feature-xyz).


## Step 3: Choose Your Merging Method## Method A: Merge Locally via Terminal (Recommended for quick local workflows)

   1. Navigate back to your main project root (not inside the .claude/worktrees/ directory).
   2. Switch to your main branch and pull the latest changes:
   
   `git checkout main`

   `git pull origin main`
   
   3. Merge the Claude worktree branch:
   
   `git merge <branch-name-from-step-2>`
   
   4. Resolve conflicts (if any) in your standard editor, commit, and push.

## Method B: Push and Open a Pull Request (Recommended for team collaboration)

   1. Push the worktree branch to your remote repository (GitHub/GitLab):
   
   `git push origin <branch-name-from-step-2>`
   
   2. Open a Pull Request (PR) on your git platform from that branch into main.
   3. Review the code, run your CI/CD pipelines, and hit Merge.

## Step 4: Clean Up the Worktree

Once the changes are safely merged into the main branch, you should delete the temporary worktree to free up disk space and keep your Git environment clean.
Run the following commands from your main repository directory:

### 1. Remove the worktree directory and registration
`git worktree remove .claude/worktrees/<branch-name>`

### 2. Delete the local branch (since it's already merged)
`git branch -d <branch-name>`

(Note: If Git throws an error saying the worktree is locked or contains uncommitted modifications you want to discard, append the --force flag: 
`git worktree remove --force .claude/worktrees/<branch-name>)`


# Worktrees in Claude Code

Claude Code includes native support for Git Worktrees. This feature allows you to run multiple independent AI coding sessions in parallel within the same repository without file edits colliding or having to manually stash and switch branches.  
When you spin up a worktree, Claude creates a new branch and sets up a separate checkout directory under .claude/worktrees/.  


## 🚀 Quick Start Commands
To launch Claude natively in an isolated worktree, run the command from the root of your Git repository:

* Auto-generated branch name:

`claude -w`

(Alternatively: `claude --worktree`)
* Specific branch name:

`claude -w feature-payments`

* Non-interactive / Autonomous run:

`claude -p "Implement the authentication logic" -w feature-auth`

(Runs the task in the background without needing terminal interaction).  

## ⚙️ How it Handles the Environment

* Isolating File Edits: Each worktree lives in its own folder inside .claude/worktrees/. Changes made by Claude in Terminal A will not affect what you or another Claude session are doing in Terminal B.  
* Environment Variables & Dependencies: Since a worktree behaves like a fresh checkout, you will need to re-initialize your development environment (e.g., run npm install or pip install).  
* The .worktreeinclude File: To automatically carry over uncommitted or gitignored files (like your .env configuration file) into every new worktree, create a file named .worktreeinclude at your root directory and list the files you want copied over.  

## 🛠️ Handling Session Cleanup
When you exit an interactive session, Claude Code will typically prompt you asking whether you want to keep or remove the worktree.  

* If you ran a non-interactive task (-p), it won't clean itself up automatically.
* If you need to manually remove a stale or locked worktree left behind, run:

`git worktree unlock .claude/worktrees/<branch-name>`

`git worktree remove .claude/worktrees/<branch-name>`

(Note: You can check active worktrees at any time using git worktree list).  

