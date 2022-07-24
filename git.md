# Git

A version control tool:

* command line tool that can keep track of changes made to a file.
* synchronize code between different people.
* Test changes to code without losing the original.
* Revert back to old versions of cod.

GitHub: the host of the repositories

To copy an existing repository from github:

    git clone <url>

A commit is when you save the file so that git starts keeping track of it

`git add <filename>` tells git which files that we want to start keeping track of

`git commit -m "message"` tells git to save a snapshot of all files added with a commit message of changes made.

All the above are changes to the local repositories

`git status` gives the status of what's going on
`git push` to github

`git commit -am "message"` do git add and commit at same time


`git pull` take changes from github and pull most recent changes down


## Merge Conflicts:

    a = 1   
    <<<<<< HEAD
    b = 2   your changes
    ======
    b = 0   remote changes
    >>>>>>
    c = 3
    d = 4
    e = 5

Look at my version and conflicting version to pick what needs to say

`git log` gives description of each commit

`git reset` revert repository to older state

`git reset --hard <commit>` reset everything to a previous state based on commit hash

`git reset --hard origin/master` reset everything to the state on github

## Branching

main branch has the stable code, use feature branches to work on different features.

Head points to the branch that you are currently focused on

Can merge the branches after completed

`git branch` tells the branches in the repository and which branch i'm on

`git checkout -b <branch name>` to go to a new branch
`git checkout <branc name>` to go to existing branch

`git merge <other branch name>` merge branch with current branch

## Forking 

Copying the code so you can make changes and push to it

then make a pull request


## GitHub Pages 

allows you to create html page 

    create new repository 
    username.github.io

    settings and enable github pages and launch
