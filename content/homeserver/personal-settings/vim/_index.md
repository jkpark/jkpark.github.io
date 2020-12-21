---
title: Vim
description: 
date: 2020-12-08T12:46:15+09:00
draft: false
weight: 0
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

I've been using [Vundle](https://github.com/VundleVim/Vundle.Vim) to manage my vim plugins for past years. Vundle is still good but I'm just wondering which is good for managing vim plugins in now a days. See [this compared post](https://www.slant.co/topics/1224/versus/~vim-plug_vs_vundle_vs_pathogen). as you can see, many people recommand to use `Vim-plug`, so I aim to install [Vim-plug](https://github.com/junegunn/vim-plug) in my new machine.

`Vim-plug` is a free, open source, very fast, minimalist vim plugin manager. 

# Setting up

Download plug.vim and put it in the "autoload" directory.

```
$ curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

# Installing plugins

Open `~/.vimrc` and declare the list of plugins you want to use in your Vim configuration file. 

1. Begin the section with call plug#begin()
2. List the plugins with Plug commands
3. call plug#end() to update &runtimepath and initialize plugin system
    - Automatically executes filetype plugin indent on and syntax enable. You can revert the settings after the call. e.g. filetype indent off, syntax off, etc.

```
" Plugins will be downloaded under the specified directory.
call plug#begin('~/.vim/plugged')

" Declare the list of plugins.

"Vim color scheme
Plug 'junegunn/seoul256.vim'

" List ends here. Plugins become visible to Vim after this call.
call plug#end()
```

Reload vimrc(`:source ~/.vimrc`) and `:PlugInstall` to install plugins.

## My .vimrc

https://github.com/jkpark/dotfiles/blob/master/vimrc