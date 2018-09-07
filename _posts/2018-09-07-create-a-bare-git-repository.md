---
layout: post 
title: "Create a bare git repository"
tags: ["git", "howto"]
description: "How to create a bare git repo"
lang: en
---

Differences between a normal git repository and a bare git repository is :
- in the normal git repository you have a .git folder inside the repository containing all relevant data and all other files build your working copy
- in a bare git repository, there is no working copy and the folder (let's call it repo.git) contains the actual repository data

Normally the normal git repository cann't be used as the remote repository, so you need to convert it into a bare git repository.

Here is the steps:
```sh
cd repo
mv .git ../repo.git # rename just for clarity
cd ..
rm -rf repo 
cd repo.git
git config --bool core.bare true
```

Sometimes you also need to change the url of the remote repository:
```sh
git remote set-url origin git://new.url.here
```

{% include JB/setup %}
