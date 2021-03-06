---
layout: post
title:  "Git memo"
---
A simple memo to remind me some basic git commands. I will update this list from time to time.

{% highlight ruby %}
# Clone
git clone https://github.com/someone/repo.git

# Init
git init

# Stages all files
git add -A
# Stages new and modified files, without deleted ones
git add .
# Stages modifies and deleted files, without new ones
git add -u

# Commit
git commit -m "Comment"
# See staged changes
git diff --cached

# Check status
git status
# See last commit with changes made
git show
# See last commit without changes made
git log -1

# See differences between local 'master' and remote 'origin' depository
git diff master origin/master
# Push commits from local branch 'master' to remote 'origin'
git push origin master
# Pull changes from remote 'origin' repository
# This is basically a 'git fetch' followed by a 'git merge'
git pull origin master

# When I`ve done some local changes but I need to pull
git stash #save local changes away
git pull
git stash pop #merge local changes

# Create and move to new branch
git checkout -b branch-name
# Come back to master
git checkout master
# Merge branch into master
git merge branch-name

# Check remote repositories
git remote -v
# Add remote repository
git remote add origin https://github.com/someone/repo.git
# Change remote's URL
git remote set-url origin https://github.com/someone/repo.git

# Puch subtree to remote 'heroku'
git subtree push --prefix path/to/subpath heroku master
{% endhighlight %}