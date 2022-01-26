---
title: "Git Commands"
tags: ""
---

# Usefull Commands

- Reset repository
```
git reset --hard
```
- Checkout all **submodules**
```
git submodule update --init --recursive
```
- Reset all **submodules**
```
git submodule foreach --recursive git reset --hard
```
- Clean new untracked files
```
git clean -fd
```
- Clean local branchs that doesn't exist remotelly
```
git remote prune origin
```
- Complete hard reset
```
lfs-reset-hard = !git rm --cached -r . && git reset --hard && git rm .gitattributes && git reset . && git checkout .
```

:::tip
You can concatenate commands inside your .gitconfig (on `C:\Users\USER_NAME\.gitconfig` ) file under **alias** tag:

```
[alias]
  lfs-reset = !git lfs uninstall && git reset --hard && git lfs install && git lfs pull
  lfs-reset-hard = !git rm --cached -r . && git reset --hard && git rm .gitattributes && git reset . && git checkout .
```

Then just call `git lfs-reset-hard`
:::
