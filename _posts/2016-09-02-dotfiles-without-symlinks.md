---
layout: post
title: Simple way to have a dotfiles repo, with no symlinking
subtitle: No faffing about
bigimg: /img/header_tech_2.jpg
---

Hands down, this is the best way of how found for managing dotfiles, without
needing to symlink to a local repo somewhere.

Instead we use a `--bare` git repository and set up a simple alias.

## Setup
```
git init --bare $HOME/.dotfiles
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
echo "alias dotfiles config='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" >> $HOME/.zshrc
```

* note: I am a zshrc user, if your using bash, then echo the alias to .bashrc

## Github

Create a github repo, named 'dotfiles'

## Set origin for github repo
```
dotfiles remote add origin git@github.com:<username>/dotfiles.git
```

## Usage
```
dotfiles status
dotfiles add .config/i3/config
dotfiles commit -m 'Adding i3 Config'
dotfiles push
```

## Replication

Should you then want to set up your dotfiles on another machine:

```
git clone --separate-git-dir=$HOME/.dotfiles https://github.com/<username>/dotfiles.git dotfiles-tmp
rsync --recursive --verbose --exclude '.git' dotfiles-tmp/ $HOME/
rm --recursive dotfiles-tmp
```
