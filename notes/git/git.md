# Git (What i know)

> **Remember: I haven't explain all the commands that are basics cause i have learned git many times and i know basic commands how it work also this is the final time i will be learing properly with writing the notes**

> skipped: **add,commit,log**

> Git have lots of command and most used are called **porcelain** commands may be i heard some where

--- 

## What to do After installation 

### First thing to do is to setup the config file 

- We can setup config file in **two** different ways 
    - with **command** 
    - with **config file**  **`.gitconfig`**

- **Setup Config using Commands**

```bash
    git config set --global user.name "ronish"
    git config set --global user.email "ronish@email.com"
    git config --unset --global user.email //used to unset or remove the config 
    git config --add project.colaborators "ram"
```
- there are lots of other configs but this two is require to work with git.
- other command i will use are init.defaultbranch,core.editor
- here i used `--global` to save config in .gitconfig file , but u can do it **without global** but one thing **keep in mind if you use inside the git repo then it will change the config for only the repo which you are inside**.

- **Setup Config using config file**
- so the config file have a format and its easy to remember too **did you notice like we used `user.name, init.defaultbranch` the first word like user,init are the main keys and another one is subkeys.**
- **Format**
```bash 
[main_key]
    subkey=value
    subkey=value

[main_key]
    subkey=value

eg: 
[user]
    name=ronish
```

> What i prefer is use **command** way to setup configs but i will be using config file way cause without pain in ass things doesnt feel good :)

--- 

## Git Internals 
- Git stores everything inside the **`.git`** folder which is hidden
- Every file tracked by Git is represented internally as objects (blobs for content, trees for structure and commits/tags for history)

### See content of git objects
- We can't use normal cat command to see what inside the object of git its in encrypted form
- Git provide command **`git cat-file <object_hash>`** to see the actual content stored inside the hash

#### Important Things To Remember
- Git store file in different type like **blob, commit and tree** 
- **Blob** type holds actual content of the file. **Its created when we do git add**
- **Tree** type holds snapshot of a folder, but with structured metadata, not just “extra info”.
- **Commit** type file holds the commiter name, email, date and commit-message.

--- 

## Branches

### How to rename branch 
- use **`-m`** to rename the branch 
`git branch -m oldname newname`


--- 

## Commit Logs
- We can use **git log** to see commit logs its normal but we can add extra flags on log to get more from log.
- Usually i use **`--oneline, --parents, --decorate and --graph`**  the most
- The **graph** is great to visulize, **oneline** will show less content easy to quick view, **parents** will shows if the commit have any other parents **great for seeing merge commit**, **decorate** will shows info like **tags**

---

## Git Rebase
- Its hard to understand this rebase but after understanding i feel more comfortable using it than commit 
- So it does three things 
- First it look for the **closest ancestor** between the **target branch** and the **current branch** you are on 
- Then it will save the new commits after the closest ancestor of the current branch 
- Then it will move the pointer of the current branch to the latest commit of the traget branch  

### Visually 
```bash
                  (Target Branch)
                  main
                   ↓
A --- B --- C --- D
       \
        E --- F
              ↑
              feature (Current Branch)

 B = Closest Ancestor (the last point they shared)
 E, F = unique commits to be "saved" and moved


// After rebase

                               feature
                                 ↓
A --- B --- C --- D --- E' --- F'
                  ↑
                 main
Now "Closest Ancestor" becomes D.

```

--- 

## Git Reset
- Its most benificial things in git 
- So it also have two modes **soft and hard** 
- **`--soft`** will undo the commit but the changes will remain there
-- **`--hard`** will do same thing as soft but the changes will be gone.

> **Note** hard will remove all commit and it will be gone forever we can say but reflog can get it back with pain in ass.
