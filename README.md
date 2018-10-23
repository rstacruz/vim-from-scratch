<p align='center'>
<br><img src='https://user-images.githubusercontent.com/74385/46930533-c84de080-d078-11e8-8b8a-24945201be94.png' width='192'><br>
</p>

<h1 align='center'>Vim from scratch</h1>

<p align='center'>
<em>Rico's guide to setting up Vim for<br> everyday development</em>
</p>

<br>

This guide will walk you through setting up a practical config that will work on [Vim], [Neovim], [Macvim], and any other Vim implementation out there.

#### Getting started

- [Install Vim and Neovim](#install)
- [Back up your existing config](#backup)
- [Create ~/.vim](#vimpath)
- [Create your .vimrc](#vimrc)
- [Set up symlinks](#symlinks)

#### Customizations

- [Add vim-plug](#vim-plug)
- [Set up plugins](#plugins)
- [Set up additional options](#options)
- [Set up key bindings](#keys)
- [Set up the leader key](#leader)

#### Interoperability

- [With GUI Vim apps](#gui)
- [Between Vim and Neovim](#vim-and-neovim)
- [With Oni](#oni)

#### Moving forward

- [Commit your config](#more)
- [Share your config](#more)
- [Learn more about Vim](#more)
- [Look at other configs](#more)

## Install Vim and Neovim <a id='install'></a>

> (Skip this step if you've already installed Vim.)

There are many ways to acquire Vim. I suggest using [Neovim], a fork of Vim with extra features--but regular [Vim] would work just fine.

- **Vim on Linux** <br> Most distributions come with `vim` and `neovim` packages. Some distributions have different versions available. When in doubt, pick the `vim-gnome` or `vim-gtk3` or `gvim` package.

  ```bash
  sudo pacman -S gvim         # Arch Linux
  sudo apt install vim-gnome  # Ubuntu
  ```

- **Neovim on Linux** <br> If your distro ships with python-neovim, add it in too.

  ```bash
  sudo pacman -S neovim python-neovim
  ```

- **Neovim on MacOS** <br> The `neovim` package is available in [Homebrew].

  ```bash
  brew install neovim
  # (todo: add more notes on python integration etc)
  ```

- **Vim on MacOS** <br> I recommend using [Macvim] with installed via [Homebrew] with `--override-system-vim`. This gets you a more updated version of Vim than if you used the `vim` package. You'll also get a GUI app, which can be nice.

  ```bash
  brew install macvim --with-cscope --with-lua --override-system-vim --with-luajit --with-python3
  ```

## Back up your existing Vim config <a id='backup'></a>

> (Skip this step if you're setting up a fresh installation of Vim.)

Want to try out this guide, but you already have Vim set up? You can rename them for now, and restore it later on.

```bash
mv ~/.vimrc ~/.vimrc~
mv ~/.vim ~/.vim~
mv ~/.config/nvim ~/.config/nvim~
```

## Create your ~/.vim <a id='vimpath'></a>

The first obvious step is to create your config folder. Vim expects this in `~/.vim`, and Neovim expects it in `~/.config/nvim`. Since our goal is to make a Vim config that'll work everywhere, I suggest keeping it in _~/.vim_ and symlinking it as needed.

```sh
mkdir -p ~/.vim
cd ~/.vim

# Version it using Git
git init
git commit -m "Initial commit" --allow-empty
```

## Create your init.vim (aka .vimrc) <a id='vimrc'></a>

Vim looks for your config in `~/.vimrc`, and Neovim looks for it in `~/.config/nvim/init.vim`. Let's create the file as `~/.vim/init.vim`, which we will symlink to the proper locations later.

```bash
cd ~/.vim
touch init.vim
```

## Set up symlinks <a id='symlinks'></a>

My preferred method is to create a `Makefile` which will set up symlinks as necessary. In `~/.vim`, create a file called `Makefile` and add this in:

```bash
# Makefile
pwd := $(shell pwd -LP)

link:
  @if [ ! . -ef ~/.vim ]; then ln -nfs "${pwd}" ~/.vim; fi
  @if [ ! . -ef ~/.config/nvim ]; then ln -nfs "${pwd}" ~/.config/nvim; fi
  @ln -nfs "${pwd}/init.vim" ~/.vimrc
```

After creating it, just run `make link`. This should finally make your config available in both `~/.config/nvim/init.vim` and `~/.vimrc`.

```bash
# Before doing this, make sure you don't have ~/.vimrc (careful!)
rm ~/.vimrc

# Set up symlinks
cd ~/.vim
make link
```

## Install vim-plug <a id='vim-plug'></a>

[**vim-plug**][vim-plug] is the plugin manager I can recommend the most. It's ridiculously fast, and supports a lot of great features. This command will download `plug.vim` into your Vim config path:

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
  https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

Edit your config file by doing `vim ~/.vim/init.vim`. Add the following:

```vim
set nocompatible
let g:mapleader="<space>"

call plug#begin('~/.vim/vendor')

if !has('nvim') && !exists('g:gui_oni') | Plug 'tpope/vim-sensible' | endif
Plug 'rstacruz/vim-opinion'

call plug#end()
```

Save it, restart Vim, then call _PlugInstall_.

```vim
" Save the file and exit vim
:wq

" Start vim again, then install the plugins
:PlugInstall
```

> See: [vim-plug usage](https://github.com/junegunn/vim-plug#usage) _(github.com)_

## Install plugins <a id='plugins'></a>

The config above will install 2 plugins. Both are optional, but I recommend them:

- [**vim-sensible**](https://github.com/tpope/vim-sensible) enables some good "sensible" defaults, such as turning on syntax highlighting. This is superfluous in some vim forks like Neovim so I suggest to conditionally load it only when needed.

  ```vim
  if !has('nvim') && !exists('g:gui_oni') | Plug 'tpope/vim-sensible' | endif
  ```

- [**vim-opinion**](https://github.com/rstacruz/vim-opinion) enables some good "opinionated" defaults that I prefer (I'm the author of this plugin!). This has some settings that I think will do well for most setups, such as incremental search and so on.

  ```vim
  Plug 'rstacruz/vim-opinion'
  ```

### More plugins

Here are some more that I can recommend to almost every developer:

- [**fzf**](https://github.com/junegunn/fzf) is a very fast file picker. I recommend this over alternatives like ctrlp.vim.

  ```vim
  Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
  Plug 'junegunn/fzf.vim'
  ```

- [**ale**](https://github.com/w0rp/ale) verifies your files for syntax errors.

  ```vim
  Plug 'w0rp/ale'
  ```

- [**vim-sleuth**](https://github.com/tpope/vim-sleuth) auto-detects if files use space or tabs, and how many spaces each file should have.

  ```vim
  Plug 'tpope/vim-sleuth'
  ```

- [**vim-polyglot**](https://github.com/sheerun/vim-polyglot) adds automatic language support for every language that Vim can support through 3rd party plugins.

  ```vim
  Plug 'sheerun/vim-polyglot'
  ```

## Set up additional options <a id='options'></a>

Our config so far has _vim-sensible_ and _vim-opinion_, which has some great defaults. You may want to add more settings. Instead of dumping them into `~/.vimrc`, I suggest adding them to your [after-directory] instead. This will keep your config file as clean as possible.

```bash
mkdir -p ~/.vim/after/plugin
vim ~/.vim/after/plugin/options.vim
```

Here are some stuff you can add. All of these are optional.

```vim
" Enable 256-color by default in the terminal
if !has('gui_running') | set t_Co=256 | endif

" Hide line numbers by default
set nonumber

" Wildignore
set wig+=vendor,log,logs
```

> See: [Keep your vimrc clean](http://vim.wikia.com/wiki/Keep_your_vimrc_file_clean) _(vim.wikia.com)_, [~/.vim/after](http://learnvimscriptthehardway.stevelosh.com/chapters/42.html#vimafter) \_(learnvimscriptthehardway.stevelosh.com)\_

[after-directory]: http://learnvimscriptthehardway.stevelosh.com/chapters/42.html#vimafter

## Set up additional key bindings <a id='keys'></a>

I suggest keeping most (all?) of your key bindings in one file in your _after-directory_. I prefer to keep them in `~/.vim/after/plugin/key_bindings.vim`. This way, you can

```bash
vim ~/.vim/after/plugin/key_bindings.vim
```

```vim
" ctrl-s to save
nnoremap <C-s> :w<CR>

" ctrl-p to open a file via fzf
if exists(':FZF')
  nnoremap <C-p> :FZF!<cr>
endif

" SPC-f-e-d to edit your config file
nnoremap <leader>fed :cd ~/.vim;<CR>:e ~/.vim/init.vim<CR>
" SPC-f-e-k to edit your kepmap file
nnoremap <leader>fek :cd ~/.vim;<CR>:e ~/.vim/after/plugin/key_bindings.vim<CR>
" SPC-f-e-o to edit your options file
nnoremap <leader>feo :cd ~/.vim;<CR>:e ~/.vim/after/plugin/options.vim<CR>
```

The `leader` keymaps at the end can be triggered with the _Spacebar_ as the leader key. For instance, the first one is `SPACE` `f` `e` `d`. These are inspired by Spacemacs.

## Change your leader key <a id='leader'></a>

The default `init.vim` above has a `g:mapleader` setting of spacebar. This is a great default that a lot of people use! I personally prefer the `,` key as a Dvorak user, but this is totally up to you. Common leader keys are `<space>`, `<cr>`, `<bs>`, `-` and `,`.

```vim
" In your ~/.vim/init.vim
let g:mapleader=","
```

> See: [Leaders](http://learnvimscriptthehardway.stevelosh.com/chapters/06.html) _(learnvimscriptthehardway.stevelosh.com)_

## Interoperability with GUI Vim apps <a id='gui'></a>

There are many Vim GUI apps available today. Some popular ones include [Macvim], [VimR], vim-gtk and more are probably coming out everyday.

There are some settings you might only want to use on GUI. You can use `if has('gui_running')` to conditionally only apply settings when running in a GUI.

Like most settings, I suggest placing them in the after-directory, eg, `~/.vim/after/plugin/theme.vim`. Here's an example that sets fonts for GUIs:

```vim
" ~/.vim/after/plugin/theme.vim

if has('gui_running')
  " Settings for when running in a GUI
  set transparency=0
  set guifont=Iosevka\ Medium:h16 linespace=-1
  set guioptions+=gme " gray menu items, menu bar, gui tabs
  set antialias
  color ir_black+
else
  " Settings for when running in the console
  color base16
endif
```

## Interoperability between Vim and Neovim <a id='vim-and-neovim'></a>

> TODO: talk about `has('nvim')`, config paths, etc

## Interoperability with Oni <a id='oni'></a>

> TODO: talk about `exists('g:gui_oni')`

## More to come! <a id='more'></a>

This guide is a work in progress, more stuff soon! But at this point you should have a working Vim config. Commit it, and share it!

Here are some more resources to look at:

- [mhinz/vim-galore](https://github.com/mhinz/vim-galore#readme) has a lot of tips on learning Vim.

- [devhints.io/vim](http://devhints.io/vim) is a quick reference on Vim.

- [Learn vimscript the hard way](http://learnvimscriptthehardway.stevelosh.com/) is a free book on Vim scripting.

- [vim-galore plugins](https://github.com/mhinz/vim-galore/blob/master/PLUGINS.md) is a curated list of common Vim plugins.

> Icon from [Thenounproject.com](https://thenounproject.com/search/?q=code&i=995778)

[homebrew]: https://brew.sh/
[macvim]: http://macvim-dev.github.io/macvim/
[neovim]: https://neovim.io/
[vim]: https://www.vim.org/
[vim-plug]: https://github.com/junegunn/vim-plug
[vim-sensible]: https://github.com/tpope/vim-sensible
[vim-opinion]: https://github.com/tpope/vim-sensible
[vimr]: http://vimr.org/
