---
layout: post
title: Vundle 설치
description: Vundle 설치 방법 및 작업내역
category: blog
tags: [Vim, Vundle]
---

[Vundle](https://github.com/VundleVim/Vundle.vim) 은 `Vim` 플러그인 관리 도구이다.

`aaa` 이라는 플러그인을 설치할려고 할때
오리지날 `vim` 시스템은
```
vim/
    syntax/
        aaa.vim
    indent/
        aaa.vim
```
이런식으로 저장되는데

`Vundle`을 이용하면
```
vim/bundle/
    aaa/
        syntax/
            aaa.vim
        indent/
            aaa.vim
```
이렇게 저장된다.

`aaa` 플러그인을 bundle로서 관리하여 해당 플러그인의 모든 파일을 알맞은 디렉토리에 저장하고 관리하기 편하게 도와준다.


# History

## Vundle Clone
```bash
jkpark@cactus:~$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
## vimrc 기본 설정

아래 내용을 `.vimrc` 파일의 제일 윗줄에 추가한다.

설치와 테스트를 위해 `NERD Tree` 플러그인을 설치할 것이다.  
```
"===== Vundle =====  
set nocompatible              " be iMproved, required  
filetype off                  " required  
" set the runtime path to include Vundle and initialize  
set rtp+=~/.vim/bundle/Vundle.vim  
call vundle#begin()  
" alternatively, pass a path where Vundle should install plugins  
"call vundle#begin('~/some/path/here')  
" let Vundle manage Vundle, required  
Plugin 'VundleVim/Vundle.vim'  
" The following are examples of different formats supported.  
" Keep Plugin commands between vundle#begin/end.  
" 여기에  원하는 플러그인 삽입 
" All of your Plugins must be added before the following line  
call vundle#end()            " required  
filetype plugin indent on    " required  
" To ignore plugin indent changes, instead use:  
"filetype plugin in  
"  
" Brief help  
" :PluginList       - lists configured plugins  
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate  
" :PluginSearch foo - searches for foo; append `!` to refresh local cache  
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal  
"  
" see :h vundle for more details or wiki for FAQ  
" Put your non-Plugin stuff after this line
```
  
## 플러그인 검색 및 설치

플러그인 검색 방법은 `.vimrc`파일을 열어 `:BundleSearch` 를 입력하면 플러그인 리스트가 출력된다.  

![](https://1.bp.blogspot.com/-v6P66qxoMDY/WG8y20KSoSI/AAAAAAAAAqw/YvMrawW83886k_nMX01Hbm1hdPrbe5lxACLcB/s700/%25EC%25BA%25A1%25EC%25B2%2598.PNG)

  
원하는 플러그인은 [여기](http://vim-scripts.org/vim/scripts.html) 에서 찾을 수 있다.  
원하는 플러그인을 찾은다음, 해당 이름을 검색하고  
```
> Shift+V 라인 선택  
Y키로 복사  
Ctrl+W ->w 로 창 이동  
P로 붙여넣기
```
하면 아래와 같이 붙여넣기가 된다.  

![](https://2.bp.blogspot.com/-ZAIj_SHJzkY/WG8z0c-YSDI/AAAAAAAAAq4/JDhdP7wVAqIsvXjVxYumU_5duO46_TKEgCLcB/s700/%25EC%25BA%25A1%25EC%25B2%2598.PNG)

## 인스톨
`vim` 을 열고 `:PluginInstall` 을 입력한 후 엔터

.vim/bundle 에서 설치된 플러그인이 존재하는지 확인한다.  
```bash
jkpark@cactus:~/.vim/bundle$ ls -al  
합계 20  
drwxrwxr-x 5 jkpark jkpark 4096  1월  6 14:41 .  
drwxrwxr-x 3 jkpark jkpark 4096  1월  6 14:27 ..  
drwxrwxr-x 8 jkpark jkpark 4096  1월  6 14:27 Vundle.vim  
drwxrwxr-x 9 jkpark jkpark 4096  1월  6 14:41 nerdtree  
```
또한 `vim` 에서 `:NERDTree` 를 입력후 엔터치면 아래 그림과 같이 `NERDTree`가 실행된다.

![](https://3.bp.blogspot.com/--MOg70KpEao/WG8vW3oitXI/AAAAAAAAAqE/Sef4ofWkbAUBhZdzjo5WdX_nj0iMRJyVwCLcB/s700/%25EC%25BA%25A1%25EC%25B2%2598.PNG)
