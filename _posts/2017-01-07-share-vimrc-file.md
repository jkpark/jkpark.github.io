---
layout: post
title: .vimrc 설정파일 공유
description: 
category: blog
tags: [Vim, Vundle]
---

[vim, vimrc 백업 파일 다운로드](https://drive.google.com/file/d/0B1Sz-lqyugC4b0ZTcXBKajR0dW8/view)
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
" NERD Tree
Plugin 'scrooloose/nerdtree'
" NERD Tree Git Plugin
Plugin 'Xuyuanp/nerdtree-git-plugin'
" All of your Plugins must be added before the following line
call vundle#end()            " required
"filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
"
"

"===== Basic Variables =====
"syntax on                                      " Enables Vim to show parts
                                                " of the text in another font or color
colorscheme vibrantink
"set lines=50 columns=120                       " 창 크기 설정
set fencs=utf-8,euc-kr,cp949,cp932,big5,latin1,urs-2le,shift-jis,euc-jp
"set fileencoding=utf-8
set ls=2                                        " 작업표시줄 항상 표시
set nu                                          " display line numbers on the left
set ts=4                                        " tab space
set sw=4                                        " Shift Width
set ruler                                       " display the cursor position on the last line of the screen
set nobackup                                    " no backup
set backspace=eol,start,indent                  " 백스페이스 사용하면 이전줄과 연결됨
set autoindent                                  " automatically indent
set cindent                                     " automatically indent in C files.
set smartindent                                 " smart indent
set ic                                          " 검색 시 대소문자 구별 안함
set nows                                        " 검색 시 파일 끝에서 처음으로 되돌리기 안함
set hls                                         " 검색어 강조
"===== 버퍼간 파일 이동 =====
map ,1 :b!1<cr>
map ,2 :b!2<cr>
map ,3 :b!3<cr>
map ,4 :b!4<cr>
map ,5 :b!5<cr>
map ,6 :b!6<cr>
map ,7 :b!7<cr>
map ,8 :b!8<cr>
map ,9 :b!9<cr>
map ,0 :b!0<cr>
map ,w :bw<cr>                          " remove current bufferfile

"======== KEY MAPPING ========
map <f1> K
map <f2> <c-w>w
map <f3> gg=G
"map <f4> :cn<cr>                        " :cw -&gt; window showing the locaion list for the current window.
"map <f5> :!./%&lt;
"map <f6> :!clear<cr>:w<cr>:make<cr>
"map <f7> :!clear<cr>:w<cr>:!gcc -W -Wall % -o %&lt;<cr>
"au BufWinEnter *.java
"\ map <f5> :!java %&lt;
"au BufWinEnter *.java,*.cpp,*.c
"\ map <f7> I//<esc>
"au BufWinEnter *.java,*.cpp,*.c
"\ map <f8> ^xx
"map <f9> :!ctags -R<cr>
"au BufWinEnter *.cpp
"\ map <f9> :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q .<cr>
"map <f10> <c->                         " 태그 검색
"map <f11> <c-t>                         " 태그 복귀
map <f12>1 :NERDTree<cr>
"map <f12>2 :TlistToggle<cr>
map <f12>3 gg=G
map ,m :new Makefile<cr>:/PROGS<cr>ww
map <pageup> <c-b>
map <pagedown> <c-f>
"저장
"map <c-s> :w<cr>
"복사
"map <c-c> :'a,'b w! ~/tmp/tmp<cr>
"붙여넣기
"map <c-p> :r ~/tmp/tmp<cr>
```