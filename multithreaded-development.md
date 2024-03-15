# Multithreaded development: Increase your productivity when you have background jobs

As a C++ developer, I sometimes have to wait for builds to complete. There's a well-known joke among C++ developers that when you're waiting for your code to compile, it's the [ultimate legitimate excuse to slack off](https://xkcd.com/303/), but when you have multiple deadlines, slacking off isn't always an option. So if you're like me and you want to get more out of your day, you might be working on two (or more) projects at once, working on a second issue while your code for the first issue is compiling.

Hang on. Doesn't that remind you of something? A CPU scheduling threads, being blocked by IO, perhaps? If we think of ourselves as the CPU and long running tasks as IO operations, then we can borrow many of the same concepts from CPU scheduling for task scheduling!

## You can use environment management tools to reduce the overhead from context switching to different issues

In a typical 'single-threaded' developer experience, especially starting out, you have a singular work environment on a single project. As you gain responsibility for more projects, you may find tools like [venv](https://docs.python.org/3/library/venv.html) for python or the [anaconda package/environment manager](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) useful for loading dependencies which change between projects.

The slower pace of single-threaded development means that your work environment is [a pet, not one of many cattle in a herd](https://www.engineyard.com/blog/pets-vs-cattle/). You can afford to perform tasks which assist in one particular issue, even if they might interfere with another issue: perhaps manually editing a configuration file to reproduce a particular issue.

In multithreaded development, you begin to realize that work environments (including repositories) can't be treated as pets, because support packages and repositories are a _shared resource_ between issues. In an actual computer, shared resources are often physical such as a shared disk drive, and require locks and synchronization to prevent loss of state information.

In multithreaded development, shared repositories and packages, as well as build artifacts, are easier to manage, since files and code are easily copied. However, if any of the project dependencies are pets, they need to be encapsulated (using docker or chroot for example) in order to turn them into copyable cattle. The first among these is the code repository for the project itself; where state is typically managed by git.

In the beginning I would create a separate folder per issue, and clone my project repository for each folder. This worked, but I had to wait for git to pull the entire repository down from the remote each time I opened a new issue. However, having different git folders became quite annoying when working on dependent tickets, as you needed to constantly keep the .gits in sync when rebasing.

Fortunately, git has a feature called `git worktree`, which allows you to keep one centralized .git repository folder and checkout different branches to different folders. I believe this makes you more conscious about your impact on your fellow and future teammate developers (including future you!).

## Switching from polling to interrupt driven completion notification helps you improve your focus on secondary tasks

Another immensely important part of multithreaded development is alerts on build completion - and adding priorities to the alerting system helps immensely. In the CPU / IO analogy, this is the difference between a polling loop and interrupts: instead of constantly checking if a build job is finished, you want the system to notify you actively when the job is finished, so you can fully attend to your new task.

When I started my multithreaded development, I had a manual command that I could attach on the end of long-running processes that would send an alert to [terminal-notifier](https://github.com/julienXX/terminal-notifier), so I could go away and do another task and my mac would let me know when the task was done. This works somewhat well with two threads-of-work, but the issue is that if you have issue 1 and issue 2, issue 1 is more important than issue 2, and you alias your build command to notify you automatically on _any_ build, then issue 2 build completions might annoy you while you're focusing on issue 1. Also mac notifications may automatically disappear if not acknowledged, or be dismissed alongside other slack notifications, and they aren't particularly salient when you're thinking about a separate issue; so I needed a better system.

Instead, I built [stack](https://github.com/acenturyandabit/stack): a notification aggregator that automatically keeps track of long running shells. Stack allows you to prioritize different threads, so that higher priority tasks alert over lower priority tasks, allowing you to stay focused on higher priority work.

## There are two main downsides to multithreaded development: reduced conscientiousness and reduced rest; but these can be actively managed back into the loop

One potential downside to multithreaded development is that it trains you to delegate away thinking and lose focus. Sending a task to the background and letting the compiler chew over it gives a little rush of dopamine, a little 'haha, I am the boss and you are the worker now', which might discourage you from being as conscientious about what you are doing. Conscientiousness is important as it helps you make better design decisions and have a better working understanding of your project, so you can become a better custodian of your project. It arises from idle time if you like diving through code while waiting for your code to build. You can actively put conscientiousness back into your project by setting documentation writing as a task in your system.

Another legitimate issue is that you get less rest. While the whole point of multithreaded development is to get more stuff done, less rest does take a toll on your ability to work in the medium to long run. But hopefully you already know this and are keeping tabs on getting enough rest.

## Data analysis on my multithreaded programming would be pretty interesting.

I think it would be possible to integrate time tracking into this whole system, and perhaps a way to leave notes for myself for each project.

I will also be working on my `stack` idea and add features like supporting long running web worker jobs, such as on databricks. `stack` is a http server, so it's compatible with anything that can send http requests and monitor its own state. [Check it out here!](https://github.com/acenturyandabit/stack)
