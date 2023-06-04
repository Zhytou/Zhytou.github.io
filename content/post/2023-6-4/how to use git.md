---
title: "Git Workflow&Tips"
date: 2023-06-04T15:31:06+08:00
draft: false
---

Git is a version control system that will help you keep track of the changes and history of your project throughout its lifestyle.

## Basical Concepts

The diagram below most closely represents the flow of a typical group project using git.

![git workflow](https://res.cloudinary.com/practicaldev/image/fetch/s--M_fHUEqA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/128hsgntnsu9bww0y8sz.png)

**git branch vs git tag**：

Both tags and branches point to a commit, they are thus aliases for a specific hash and will save you time by not requiring to type in a hash.

The difference between tags and branches are that a branch always points to the top of a development line and will change when a new commit is pushed whereas a tag will not change. Thus tags are more useful to "tag" a specific version and the tag will then always stay on that version and usually not be changed.

**git add vs git stash**：

Stash will move your modified files into a stack. So, later in the same or in another branch, you will be able to bring them back and see those modifications in your project.

Stage is the step before to make a commit, you add modified files to "Staged files" to create your next commit.

## Usage

### Simple example at a glance

**initialize a local git repo and bind it with several remote repo**:

``` bash
# create an empty git repo
git init

# add a remote repo and get the existed files from the remote repo
git remote add public https://github.com/xxx/project.git
git fecth public
git merge public/master

# first commit
git add .
git commit -m "first commit"

# add your personal remote repo and set the upstream 
git remote add origin https://github.com/Zhytou/project.git
git push -u origin master
```

**use .gitignore to omit some files**:

``` bash
# change .git/info/exclude to make .gitignore work
cat .gitignore > .git/info/exclude
```

**some basic and useful git commands**:

``` bash
# check the files state(untracked or unstaged)
git status

# change to another branch
# if the branch does not already exist, it will be created.
git checkout [branch name]

# add a new branch
git checkout -b [new-branch-name] | git branch [new-branch-name]

# delete a branch
git branch -D [branch-name]

# show commit history
git log

# show differences between two files
git diff [file name]
```

### Advanced git tips

**stash temporary changes**:

``` bash
# move your modified files into a stack
git stash
# pop out your modified files into working directory
# the first command applies the most recent stash to your working directory and then removes it from the stash list, while the second one applies the most recent stash to your working directory, but does not remove it from the stash list. 
git stash pop | git stash apply
```

**revert commits**:

``` bash
# Hard delete unpublished commits
git reset --hard [commit number]
```

**set user name and email**:

``` bash
# make sure the email same as your github registered email, it's important to count your contribution
git config --global user.name "Zhytou"
git config --global user.email "zhoytou@gmail.com"
```

**use ssh instead of http**:

``` bash
# generate key
ssh-keygen -t rsa -C "zhoytou@gmail.com"

# show the public key and add it to your personal account
cat id_rsa.pub

# show remote url and change it into ssh
git remote -v
git remote set-url origin git@github.com:Zhytou/project.git

# start ssh-agent(if you are using windows terminal, then skip this step)
eval `ssh-agent -s`

# add the private key
ssh-add ~/.ssh/id_rsa

# test 
ssh -T git@github.com

# see something like "Hi Zhytou! You've successfully authenticated, but GitHub does not provide shell access", then you're ready to go
```

### How to write a good git commit message

**introduction to conventional commits**:

AngularJS commit message format, also known as conventional commits follows a specific structure:

``` txt
<type>(<scope>): subject

<body>

<footer>
```

**label**:

- The type field indicates the type of commits and must be one of the following: feat/fix/docs/style/refactor/test/chore
- The scope field is optional and indicates the scope of the commit, such as the module or file name.
- The subject field is a brief description of the commit, no more than 50 characters, and is used to display in Git log.
- The body field (optional) is a detailed description of the commit, used to explain the reason and impact of the commit and can contain multiple lines of text.
- The footer field (optional) is used to close an issue or add metadata, etc.

**example**:

``` txt
feat(users): add login feature

This commit adds a login feature to the users module. Users can now create accounts and log in to the application using their email address and password. This feature required changes to the user model and the addition of several new routes and templates.

Closes #123
```

## References

- [conneting to github with ssh](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [git revert handbook](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging#_undoing_merges)
