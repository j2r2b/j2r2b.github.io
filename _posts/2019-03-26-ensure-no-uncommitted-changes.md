---
title: "Ensure no uncommitted changes in a git repository"
---

## Ensure no uncommitted changes in a git repository

This bash script can be used to check if there are uncommitted changes in a git repository or not.

If the working directory is dirty both commands `git diff` and `git status` are also called in order to print the modified content in the console.

```bash
#!/bin/bash

if [ -n "$(git status --porcelain)" ]; then
    echo "There are uncommitted changes in working tree after execution of the build"
    echo "Perform git diff"
    git --no-pager diff
    echo "Perform git status"
    git status
    echo "Please run the build locally and commit changes"
    exit 1
else
    echo "Git working tree is clean"
fi
```

If you have a build that generates or modifies files and those derived resources are checked in your git repository (this is a project/team decision), then it is great to ensure that with each commit the generated code stays up-to-date. 
Using a script like the one presented in this article is a good idea.
The CI server will perform the check for you.