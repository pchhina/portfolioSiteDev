+++
date = "2017-12-26T21:40:42-06:00"
draft = false
title = "Just Enough... git"
tags = ["git"]
categories = ["Tutorials", "Tools"]

+++
This is just a list of git commands I am currently using. There are tons of tutorials out there as well as quite a few cheat sheets. While those are mostly comprehensive resources, I tend to get distracted and try to learn all commands at once without using them in real projects. That doesn't help much.

**Initialize a local git repository**
{{<highlight git>}}
git init
{{</highlight>}}

**Cloning an existing project from github**
{{<highlight git>}}
git clone <project url here>
{{</highlight>}}

**Status Check**
{{<highlight git>}}
git status
{{</highlight>}}

**Create a branch and check it out**
{{<highlight git>}}
git checkout -b <branch>
{{</highlight>}}

**List all branches and active branch**
{{<highlight git>}}
git branch
{{</highlight>}}

**Stage specific files, all files, all modified files**
{{<highlight git>}}
git add <files>
git add --all
git add -u
{{</highlight>}}

**Unstage a file**
{{<highlight git>}}
git reset HEAD <file-name>
{{</highlight>}}

**Commit changes**
{{<highlight git>}}
git commit -m "commit message"
{{</highlight>}}

**Check log**
{{<highlight git>}}
git log
{{</highlight>}}

**Merge branch into master**
{{<highlight git>}}
git merge <branch>
{{</highlight>}}

**Delete a branch**
{{<highlight git>}}
git branch -d <branch>
{{</highlight>}}

**Push changes to remote(master)**
{{<highlight git>}}
git push origin master
{{</highlight>}}

With a new local git that has never been linked to remote, first create a new Github repo. Then:
{{<highlight git>}}
git remote add origin <remote repo URL>
git remote -v #just for verification
git push origin master
{{</highlight>}}

### Care to learn more?
Please see the [official git documentation](https://git-scm.com/doc).
