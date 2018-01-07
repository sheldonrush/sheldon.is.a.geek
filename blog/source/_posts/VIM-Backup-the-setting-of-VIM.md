title: "[VIM] Backup the setting of VIM"
date: 2015-05-05 08:53:27
categories:
- vim
tags:
- vim
- CTag
- Cscope
toc: true
---
# Purpose

最近感覺電腦快往生了,所以現在未雨愁繆，先將

自己常用的vim setting作一下備份，順便將ctag和cscope

常用的script作一下記錄，假如有人有興趣也可以拿去發揚光大.

## .vimrc 

```
" set up pathogen, https://github.com/tpope/vim-pathogen
filetype on " without this vim emits a zero exit status, later, because of :ft off
filetype off
call pathogen#infect()
filetype plugin indent on
" don't bother with vi compatibility
set nocompatible

" enable syntax highlighting
syntax enable

" auto come back the last file location
 if has("autocmd")
     autocmd BufRead *.txt set tw=78
         autocmd BufReadPost *
             \ if line("'\"") > 0 && line ("'\"") <= line("$") |
    \   exe "normal g'\"" |
    \ endif
endif


let g:session_autosave = 'yes'

set autoindent
set autoread                                                 " reload files when changed on disk, i.e. via `git checkout`
set backspace=2                                              " Fix broken backspace in some setups
set backupcopy=yes                                           " see :help crontab
set clipboard=unnamed                                        " yank and paste with the system clipboard
set directory-=.                                             " don't store swapfiles in the current directory
set encoding=utf-8
set expandtab                                                " expand tabs to spaces
set ignorecase                                               " case-insensitive search
set incsearch                                                " search as you type
set laststatus=2                                             " always show statusline
set list                                                     " show trailing whitespace
set listchars=tab:▸\ ,trail:▫
set number                                                   " show line numbers
set ruler                                                    " show where you are
set scrolloff=3                                              " show context above/below cursorline
set shiftwidth=2                                             " normal mode indentation commands use 2 spaces
set showcmd
set smartcase                                                " case-sensitive search if any caps
set softtabstop=2                                            " insert mode tab and backspace use 2 spaces
set tabstop=8                                                " actual tabs occupy 8 characters
set wildignore=log/**,node_modules/**,target/**,tmp/**,*.rbc
set wildmenu                                                 " show a navigable menu for tab completion
set wildmode=longest,list,full

set modifiable
set write
set hlsearch
set ic
set clipboard=unnamedplus
" Enable basic mouse behavior such as resizing buffers.
set mouse=a
if exists('$TMUX')  " Support resizing in tmux
  set ttymouse=xterm2
endif

" keyboard shortcuts
let mapleader = ','
map <C-h> <C-w>h
map <C-j> <C-w>j
map <C-k> <C-w>k
map <C-l> <C-w>l
map <leader>l :Align
nmap <leader>a :Ack 
nmap <leader>b :CtrlPBuffer<CR>
nmap <leader>d :NERDTreeToggle<CR>
nmap <leader>f :NERDTreeFind<CR>
nmap <leader>t :CtrlP<CR>
nmap <leader>s :cs find c <C-R>=expand("<cword>")<CR><CR>
nmap <leader>e :cs find t <C-R>=expand("<cword>")<CR><CR>
nmap <leader>T :CtrlPClearCache<CR>:CtrlP<CR>
nmap <leader>] :TagbarToggle<CR>
nmap <leader><space> :call whitespace#strip_trailing()<CR>
nmap <leader>g :GitGutterToggle<CR>
nmap <leader>c <Plug>Kwbd
map <silent> <leader>V :source ~/.vimrc<CR>:filetype detect<CR>:exe ":echo 'vimrc reloaded'"<CR>

" NERDTree seeting
let NERDTreeShowHidden=1


"将:cs find
"c等Cscope查找命令映射为<C-_>c等快捷键（按法是先按Ctrl+Shift+-,然后很快再按下c）
"nmap <C-_>s :cs find s <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
"nmap <C-_>g :cs find g <C-R>=expand("<cword>")<CR><CR>
"nmap <C-_>d :cs find d <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
"nmap <C-_>c :cs find c <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
"nmap <C-_>t :cs find t <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
"nmap <C-_>e :cs find e <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
"nmap <C-_>f :cs find f <C-R>=expand("<cfile>")<CR><CR>
"nmap <C-_>i :cs find i <C-R>=expand("<cfile>")<CR><CR>:copen<CR><CR>


" plugin settings
let g:ctrlp_match_window = 'order:ttb,max:20'
let g:NERDSpaceDelims=1
let g:gitgutter_enabled = 0

" Use The Silver Searcher https://github.com/ggreer/the_silver_searcher
if executable('ag')
  let g:ackprg = 'ag --nogroup --column'

  " Use Ag over Grep
  set grepprg=ag\ --nogroup\ --nocolor

  " Use ag in CtrlP for listing files. Lightning fast and respects .gitignore
  let g:ctrlp_user_command = 'ag %s -l --nocolor -g ""'
endif

" fdoc is yaml
autocmd BufRead,BufNewFile *.fdoc set filetype=yaml
" md is markdown
autocmd BufRead,BufNewFile *.md set filetype=markdown
" extra rails.vim help
autocmd User Rails silent! Rnavcommand decorator      app/decorators            -glob=**/* -suffix=_decorator.rb
autocmd User Rails silent! Rnavcommand observer       app/observers             -glob=**/* -suffix=_observer.rb
autocmd User Rails silent! Rnavcommand feature        features                  -glob=**/* -suffix=.feature
autocmd User Rails silent! Rnavcommand job            app/jobs                  -glob=**/* -suffix=_job.rb
autocmd User Rails silent! Rnavcommand mediator       app/mediators             -glob=**/* -suffix=_mediator.rb
autocmd User Rails silent! Rnavcommand stepdefinition features/step_definitions -glob=**/* -suffix=_steps.rb
" automatically rebalance windows on vim resize
autocmd VimResized * :wincmd =

" Fix Cursor in TMUX
if exists('$TMUX')
  let &t_SI = "\<Esc>Ptmux;\<Esc>\<Esc>]50;CursorShape=1\x7\<Esc>\\"
  let &t_EI = "\<Esc>Ptmux;\<Esc>\<Esc>]50;CursorShape=0\x7\<Esc>\\"
else
  let &t_SI = "\<Esc>]50;CursorShape=1\x7"
  let &t_EI = "\<Esc>]50;CursorShape=0\x7"
endif

" Go crazy!
if filereadable(expand("~/.vimrc.local"))
  " In your .vimrc.local, you might like:
  "
  " set autowrite
  " set nocursorline
  " set nowritebackup
  " set whichwrap+=<,>,h,l,[,] " Wrap arrow keys between lines
  "
  " autocmd! bufwritepost .vimrc source ~/.vimrc
  " noremap! jj <ESC>
  source ~/.vimrc.local
endif
```
## .vimrc.local

```
"set nocursorline " don't highlight current line
set cursorline

" keyboard shortcuts
inoremap jj <ESC>

" gui settings
if (&t_Co == 256 || has('gui_running'))
  if ($TERM_PROGRAM == 'iTerm.app')
    colorscheme solarized
  else
    colorscheme desert
  endif
endif
"""
" Cscope settings
if has("cscope")
  let i = 1
  let path_depth = 20
    while i < path_depth
      if filereadable("cscope.out")
        let db = getcwd() . "/cscope.out"
        echo db
        let $CSCOPE_DB = db
        cs add $CSCOPE_DB
        let i = 20
      else
        cd ..
      let i += 1
      endif
    endwhile
endif
```

## sr

```
#!/bin/sh
FILE_LIST=file_list
temp_str=NULL

for i in "$@"
do
  if [ -n "$i" ]; then
    temp_str=$i
    if [ "$1" = "$i" ]; then
      echo "First Source code directory :" "$1"
      find $i -name "*.h" -o -name "*.c" -o -name "*.cpp" -o -name "*.mk" -o -name "*config" -o -name "Makefile" -o -name "*.inl" -o -name "*.py" > ./$FILE_LIST
    else
      echo "the other source code directory :" "$i"
      find $i -name "*.h" -o -name "*.c" -o -name "*.cpp" -o -name "*.mk" -o -name "*config" -o -name "Makefile" -o -name "*.inl" -o -name "*.py" >> ./$FILE_LIST
    fi
  else
    echo "please keyin the project path"
  fi
done

if [ "$#" -ne 0 ]; then
  echo "create the cscope tag and ctags"
  cscope -bkq -i ./$FILE_LIST
  ctags -R -L ./$FILE_LIST
fi
```
