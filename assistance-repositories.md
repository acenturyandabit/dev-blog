# Using `git worktree` to manage peer dependencies

When I started at my current place, I was tasked with writing integration tests for a C++ project, which would be based off a second repository, which held configuration files for a number of other projects including my own.

To do this, I decided to make a script that would make a copy of the production configuration, edited the config so that it pointed at local resources instead, and then run the C++ app. I put my script in the /scripts folder of my C++ project, and suddenly this created a peer dependency on the production configuration repository. I automated the process of cloning the peer repository to a .gitignored folder under the /scripts folder, so that it could be run smoothly on my code reviewer's machine with a single line invocation.

```txt
.
└── projectA
    ├── .gitignore
    └── scripts
        └── configs
```

But this felt strange: Why was the peer dependency sitting under the main repository, if it was a _peer_ and not a _child_? The peer dependency was a test time dependency, not a runtime or even a build time dependency; and since it held configuration files, it was not only a peer to my specific project, but also to almost every other project in the firm.

Having an issue depend on multiple repositories is a very common scenario. One naive way of addressing this problem is to decide that one repository should be the parent, and that less salient repositores can live as children underneath the main repository as submodules. However, when you switch between branches of one repository, your other repository won't follow along; and if you're [actively trying to develop multiple branches at a time](./multithreaded-development.md), then this is a frequent problem.

## Using a parent repository

Having peer repositories in submodules is also a design issue: peers shouldn't be children of each other. For my first attempt at fixing this, I elevated the integration test into a parent directory, and had a /repos directory where I put both the project repository and its peer. The /repos folder was gitignored and I could put my integration script in the parent directory; which itself became an 'assist' repository. The integration script treats both the other repositories as children, since it needs to know about them.

```txt
~
└── projectA-assist
    ├── config
    └── projectA
```

While this was better, another problem soon came up when I started working on two issues at a time: switching between branches of project A doesn't automatically switch between branches of the configuration repository. A solution to this would be to manage the state of the child repositories in the parent repository, but then the parent repository is given two responsibilies: to hold support scripts, and also to manage the state of the child repositories, which breaks the single responsibility principle and adds complexity. Additionally, my C++ project had many gitignored build artefacts that were at best expensive to recompile, and at worst actually stateful; which meant trying to automate switching branches was incredibly time consuming. Ultimately, having two codependent repositories made them feel like pets, rather than cattle.

## Using `git worktree` and making everything peers

To reduce the state switching involved in switching branches for an issue, we can instead use `git worktree`. This puts every branch in its own folder, so multiple branches can coexist. Since `git worktree` has a single main .git folder for multiple repositories, we can't have a parent assist repository clone and manage the branch state of child branches. Instead, the assist repository should be a peer of the project and configuration repositories. The 'assist' repository can reach into its peers by looking into sibling directories, and its own state may depend on the particular issue you are targeting as well.

So my final home directory looks like this:
```txt
~
├── repo_pool
│   ├── configs
│   ├── projectA
│   └── projectA-assist
└── workspaces
    ├── issue1
    │   ├── configs         // worktree
    │   ├── projectA        // worktree
    │   └── projectA-assist // and so on 
    └── issue2
        ├── configs
        ├── projectA
        └── projectA-assist
```