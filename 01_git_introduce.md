---
title: Introduce yourself to Git
output: html_document
---

**You only have to set this up once per machine.**

### TL;DR

``` sh
git config --global user.name 'Ralph Yorke'
git config --global user.email 'ralph.yorke@zoology.ubc.ca'
git config --global --list
```

In words ...

From RStudio, go to *Tools > Shell* and tell `git` your name and **GitHub email** by typing (use your own name and email):

  * `git config --global user.name 'Ralph Yorke'`
  
    - This does __NOT__ have to be your GitHub username, although it can be. Another good option is your actual first name and last name. Your commits will be labelled with this name, so this should be informative to potential collaborators.

  * `git config --global user.email 'ralph.yorke@zoology.ubc.ca'`
  
    - This __must__ be the email that you used to sign up for GitHub.

  * `git config --global core.editor 'nano'`


These commands return nothing. You can check that `git` understood what you typed by looking at the output of `git config --global --list`.

Resources:

* [GitHub's advice](https://help.github.com/articles/set-up-git).