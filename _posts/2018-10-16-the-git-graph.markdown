---
layout: post
title: The Git Graph and Merge vs Rebase
date: 2018-10-16 12:00:00 +0300
description: Visualizing Git as a Graph
img: git/git_merge_log.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Git, Graph Theory]
---

The Git version control system abstracts code changes in a repository by storing those changes inside of a **Directed Acyclic Graph**.  While this abstraction might sound complex, it is actually an elegant way to visualize the history of a project and it gives us powerful tools to manipulate that history.  Once a Git users starts visualizing the graph they are building while running the Git commands, they are able to manipulate this history of the code in complex ways. Let’s break down what a **Directed Acyclic Graph** is and how it is used by Git to build out the history.

A **Graph** is a visual representation of objects, also called vertices, and relationships between those objects also known as edges. In Git the vertices represent a change in the code and the edges represent the order in which those changes happened. Here is a simple graph which 5 vertices, labeled v1 through v5 and 5 edges between those vertices:

![Graph]({{site.baseurl}}/assets/img/git/graph.png)

A **Directed Graph** is a graph where the edges have a relationship that requires a direction. For example, if we were graphing out a family tree we would use a directed graph to show a parent-child relationship. In the Git world the direction shows that one commit is 'on top' of another commit, meaning that code that changed in commit 2 happened after the code change in commit 1. With this logic in place Git is able to store the diff of the code change between two commits instead of the whole code code base on each commit. Here is an example graph as a **Directed Graph**.

![Directed Graph]({{site.baseurl}}/assets/img/git/graph_with_direction.png)

A **Directed Acyclic Graph** is a graph whose direction never makes a circle. Our above example is not **Acyclic** because you are able travel in a cycle from v2 → v3 → v5 → v4 → v2. Git's graph needs to be acyclic because Git is representing the history of a code base, if we had any circle in the graph that would mean there are two was to get to one point in time, we would have a Back to the Future situation on our hands. So Git doesn't allow this and all of the Git's Graphs are Acyclic. To fix the above graph we need to change the direction of one of the edges causing this cycle, lets change edge between v2 and v4.

![Directed Acyclic Graph]({{site.baseurl}}/assets/img/git/graph_that_is_acyclic.png)

And that's it! We now know what a **Directed Acyclic Graph** is, woot! Let's try and use that knowledge to run a few git commands to reproduce different graphs.

## Merging Two Branches

Lets start off by trying to reproduce the above graph. Here are the steps we need to follow:
1. Initializing a git project
2. Creating commits v1 and v2 on the current branch (default will be master)
3. Create a new branch for commit v3
4. Jumping back to our intial branch and create commit v4
5. Then bringing it all back together with a **merge** commit v5

Here are the commands to accomplish this:

```
git init
echo "First File Content" >> first_file.txt
git add first_file.txt
git commit -m 'v1'
echo "second master commit" >> first_file.txt
git add first_file.txt
git commit -m 'v2'
git checkout -b new_branch_for_v3
echo "on new_branch_for_v3 branch" >> second_file.txt
git add second_file.txt
git commit -m 'v3'
git checkout master
echo "3rd master commit" >> first_file.txt
git add first_file.txt
git commit -m 'v4'
git merge new_branch_for_v3 -m 'v5'
```

We can look at what we have done with the `git log --graph --oneline --decorate` command.

![Git Merge]({{site.baseurl}}/assets/img/git/git_merge_log.png)

And there we have it! We reproduce our sample graph using Git commands. A few interesting takeaways about this image are 1) each commit has a `sha` or a unique set of numbers and letters and 2) we can see that `HEAD` is equal to master, meaning that our currently checked out branch and master at the same spot. I really like the `'git log --graph --oneline --decorate'` command because it helps me visually understand the history of the repository so I have a [git alias](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases) for it to `git lol`.

## Rebase Two Branches

Now that we understand the basics of the graph that is being drawn we can ask interesting questions like: What will happen to our graph if instead of **merging** in the last step we were to **rebase** the `'new_branch_for_v3'` branch?

```
git init
echo "First File Content" >> first_file.txt
git add first_file.txt
git commit -m 'v1'
echo "second master commit" >> first_file.txt
git add first_file.txt
git commit -m 'v2'
git checkout -b new_branch_for_v3
echo "on new_branch_for_v3 branch" >> second_file.txt
git add second_file.txt
git commit -m 'v3'
git checkout master
echo "3rd master commit" >> first_file.txt
git add first_file.txt
git commit -m 'v4'
git rebase new_branch_for_v3
```

This time lets look at the `git log` before and after we run the rebase.

![Git Rebase]({{site.baseurl}}/assets/img/git/git_rebase_log.png)

There are a few interesting things about the above image:
1. We didn't end up with 5 commits we only have v1 through v4. Rebasing doesn't make an extra `merge` commit.
2. The graph never branches out. After having rebased, the history of the extra branch is gone.
3. The `sha` for commit v4 has changed, since v4 is now based off of v3 instead of v2 the content of that commit had to change so effectively it is a new commit with a new `sha`.

We can visualize the rebase process by drawing out the graphs that made this happen.

Here is the graph before we run the `git rebase` command:

![Graph Rebase 1]({{site.baseurl}}/assets/img/git/graph_rebase_1.png)

Here is the rebase in action:

![Graph Rebase 2]({{site.baseurl}}/assets/img/git/graph_rebase_2.png)

Here is what we are left with once everything is over.

![Graph Rebase 3]({{site.baseurl}}/assets/img/git/graph_rebase_3.png)

## Merge vs Rebase

Now that we analyzed the graphs generated by the two different Git commands we can come up with some key differences between them. Merge keeps the branching in history and records when a branch diverged and when it was merged back in. Merge never changes history it only adds new commits to the graph. Rebase, on the other had, re-writes history as if we never branched. Rewriting history requires the graph to be rewound and played forward making new `sha`s along the way. This means that if you push changes that have been rebased this push will impact anyone else that is working on that branch since they will have the wrong version of history. Because of these differences most team, generally, don't allow rebased changes to be pushed to their main line branches but they will encourage rebases to happen on 'feature' branches. Doing so allows the Git history to be a clean set of merges for different features, while the extra clutter is rebased away.
