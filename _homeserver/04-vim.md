---
layout: post
title: 4장. 파일 편집기 - VIM
category: homeserver
lang: en
---

I've been using [Vundle](https://github.com/VundleVim/Vundle.Vim) to manage my vim plugins for past years and I also wrote about [Vundle](install-vundle). Vundle is still good but I'm just wondering which is good for managing vim plugins in now a days. See [this compared post](https://www.slant.co/topics/1224/versus/~vim-plug_vs_vundle_vs_pathogen). as you can see, many people recommand to use `Vim-plug`, so I aim to install [Vim-plug](https://github.com/junegunn/vim-plug) in my new machine.

`Vim-plug` is a free, open source, very fast, minimalist vim plugin manager. 

# Setting up

Download plug.vim and put it in the "autoload" directory.

```
$ curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

# Installing plugins

Open `~/.vimrc` and declare the list of plugins you want to use in your Vim configuration file. 

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

![](/images/posts/install-ubuntu1804/vim01.png)

## My .vimrc

<script src="https://gist.github.com/jkpark/da218ce74c8be8d21617353245e7df95.js"></script>
