# Git tricks 2: Using `git worktree` to reduce the size of your `.git` folders across multiple issues

When a project starts out, you typically have a single instance on a single server. This instance and the underlying server is a pet: if it breaks, you have to fix it for the project to go on.

However, when things scale up, having a single point of failure is an unacceptable risk. One way to mitigate this risk is to turn your single pet instance/server into a herd of cattle instances/servers.

While this is common devops knowledge, we can apply it to our development efforts as well! If we have multiple workfronts, then the classic way of managing these is to create one branch per workfront, and we can swap between workfronts by changing branches.

```bash
git checkout -b workfront-1
git checkout main
```

This is ok, assuming your code doesn't produce any build time artefacts. If you're working in large C++ projects, that assumption is quite untrue - you will have build artefacts (object files) which will vary between workfronts and should not be committed to source control.

One simple way to do this is to create a workspace folder for each workfront, and clone the repository to each of the folders.

```bash
$ tree
.
├── workfront-1
│   ├── .git
│   └── some_files
└── workfront-2
    ├── .git
    └── some_files
```

This is fine, but having two separate .git directories has two main disadvantages: Firstly, if the repository is huge, then cloning it each time wastes bandwidth, disk space and time; and secondly, your work might produce patches and stashes that you want to live on in your local .git.

In the ideal case, we want a single .git to store all our commits, while having multiple directories with a workfront each. Fortunately, this functionality exists in [git-worktree](https://git-scm.com/docs/git-worktree).

## Git worktree 101

First, clone your repository as usual. Then, instead of `git checkout -b issue-name`, use `git worktree add -b ticket-name /path/to/workspace/ticket-name/repository_name`. This will create the folder `/path/to/workspace/repository_name` with your new worktree in it.

What's in this worktree? Instead of `.git` being a directory, we can `cat .git` and we will discover a regular text file with:

```txt
gitdir: /path/to/workspace/ticket-name/repository_name
```

## The principles behind the advantages of using `git worktree`

When you're working on different projects, you might use tools like `venv` or `conda` to setup different python environments for different projects, to prevent a project's dependencies from leaking into the global scope. Git worktree does a similar thing by making sure temporary files don't leak into your disk's global scope.

Creating multiple worktrees means you only have one `.git` folder per repository, rather than one per repository per issue, increasing your [scalability (i.e cost per marginal issue)](https://brooker.co.za/blog/2024/01/18/scalability.html). This means commits persist between issues, allowing you to apply temporary commits useful in one place to another without using `git patch`; and you don't have to worry about losing temporary commits when you're cleaning up after an issue. One persistent `.git` folder also means you spend less time running `git clone`. If you work on issues in parallel, this can also reduce your disk usage.

## But that command is so tedious! Can't we do better?

Sure! Let's say we're happy to  put all of our tickets in a folder called `/path/to/workspace` (as above). Then we can make a git alias:

```bash
git config --global alias.wktr-add '!git fetch origin; git reset --hard origin/master; git worktree add /path/to/workspace/$1/$(basename $(git rev-parse --show-toplevel)) -b'

# Then:
cd /path/to/repository
git wktr-add issue1
cd /path/to/workspace/issue1/repository
# and get to work!
```

Let's break this down, shall we?

1. First, `git fetch origin` and `git reset --hard origin/master` make sure that you're on the latest master branch before you do any checking out. You can omit these if you want to save some time, or if your origin's master branch is updated less frequently than your local branches get checked out, since you're saving so much time.
2. Next, add a worktree in an issue folder in your workspace folder, with your repository name (which we get using `git rev-parse --show-toplevel`).

This is much better, since you have to specify only one thing: the issue name :)

## Even better: multi-repository drifting

But what if issues frequently require multiple repositories? Then, every time we want to create an issue, we have to `cd` to multiple repository folders and clone them all to our issue space.

Well, we can put all our repositories in /path/to/repositories, and create a bash alias for that too:

```bash
alias wktr-add='f(){ cd /path/to/repositories/$1; git wktr-add $2; }; f'

# Usage:
wktr-add issue1 repositoryA
wktr-add issue1 repositoryB
```

An extra benefit of this is that you don't have to `cd` into the repository you're trying to clone because the function's `cd` does it for you, but only inside the function scope.

## One last thing

It might be useful to `git checkout parked-branch` after you `git clone` new repositories into your /path/to/repositories so that you don't have to worry when you are trying to quickly checkout to master when fixing some issue or another.

Of course, add this to your `~/.bashrc` if you find this is useful.
