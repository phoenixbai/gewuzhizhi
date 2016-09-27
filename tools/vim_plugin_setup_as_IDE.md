# VIM+Plugins As IDE



Recently, I am reading the source code of project xgboost, which is mainly written in c++ and provides python interfaces as well. So, I need both c++ and python IDE at the same time and switching between two different IDEs just for reading through a single logic is tiresome. So I decided to choose the highway, that is to use omnipotent VIM as my IDE for this project, or more to come.

To use VIM as IDE, here is a must plugin list as a minimal requirement (will continue to update later):

- Project/Filesystem Browsing: NERDTree
- Buffer/File Browsing: Command-T
- Code Browsing: Tagbar
- Writing Code:
- Vim functionality:
- Compilation:
- IDE integration: for python - jedi-vim
- Source Control Integration:
- Debugging:



BTW, pathogen is a great tool for managing plugins. Least effort is needed from user perspective.



## NERDTree

- [Nerd Tree: A File Explorer](http://www.34m0.com/2011/04/nerd-tree-file-explorer-with-mac-vim.html)
- [NerdTree tabs: This plugin aims at making NERDTree feel like a true panel](https://recordnotfound.com/vim-nerdtree-tabs-jistr-17287)
```
cd ~/.vim/bundle
git clone git://github.com/scrooloose/nerdtree.git

" Give a shortcut key to NERD Tree (fn+F2)
map <F2> :NERDTreeToggle<CR>

cd ~/.vim/bundle
git clone https://github.com/jistr/vim-nerdtree-tabs.git

map <F3> <plug>NERDTreeTabsToggle<CR>
map <Leader>f <plug>NERDTreeTabsFind<CR>

:helptags
```
- Use the natural vim navigation keys hjkl to navigate the files, or just the arrow keys
  * o open the file in a new buffer or open/close directory.
  * t open the file in a new tab.
  * i open the file in a new horizontal split.
  * s open the file in a new vertical split.
  * p go to parent directory.
  * r refresh the current directory.
- open project with nerdTree: vim ~/Dev/path/to/project
- All window commands start with ctrl+w:
  * ctrl+ww cycle though all windows
  * crtl+wh takes you left a window
  * crtl+wj takes you down a window
  * crtl+wk takes you up a window
  * crtl+wl takes you right a window
- Let's open some tabs
  * t open new tab
  * T open new tab while staying in current tab
  * gt cycle though all tabs
  * gT cycle though all tabs (moves to the left)
- :help NERD_tree.txt



## Command-T

- [command-t documentation](https://github.com/wincent/command-t/blob/master/doc/command-t.txt)
   * The <Leader> key is mapped to \ by default. So if you have a map of <Leader>t, you can execute it by default with \+t
- command-t usage:
   * <Leader>t or :CommandT bring up the file window
   * The following mappings are active when the prompt has focus:
     * <BS>        delete the character to the left of the cursor
     * <Del>       delete the character at the cursor
     * <Left>      move the cursor one character to the left
     * <C-h>       move the cursor one character to the left
     * <Right>     move the cursor one character to the right
     * <C-l>       move the cursor one character to the right
     * <C-a>       move the cursor to the start (left)
     * <C-e>       move the cursor to the end (right)
     * <C-u>       clear the contents of the prompt  <C-u> mean ctrl+u
     * <Tab>       change focus to the file listing
   * The following mappings are active when the file listing has focus:
     * <Tab>       change focus to the prompt
   * The following mappings are active when either the prompt or the file listing has focus:
     * <CR>        open the selected file
     * <C-CR>      open the selected file in a new split window
     * <C-s>       open the selected file in a new split window
     * <C-v>       open the selected file in a new vertical split window
     * <C-t>       open the selected file in a new tab
     * <C-j>       select next file in the file listing
     * <C-n>       select next file in the file listing
     * <Down>      select next file in the file listing
     * <C-k>       select previous file in the file listing
     * <C-p>       select previous file in the file listing
     * <Up>        select previous file in the file listing
     * <C-f>       flush the cache (see |:CommandTFlush| for details)
     * <C-q>       place the current matches in the quickfix window
     * <C-c>       cancel (dismisses file listing)
   * The following is also available on terminals which support it:
     * <Esc>       cancel (dismisses file listing)
- command-t-commands:
   * |:CommandT|		map to <Leader>t
   * |:CommandTBuffer|   map to <Leader>b
   * |:CommandTCommand|
   * |:CommandTHelp|
   * |:CommandTHistory|
   * |:CommandTLine|
   * |:CommandTMRU|
   * |:CommandTJump|   map to <Leader>j
   * |:CommandTSearch|
   * |:CommandTTag|
   * |:CommandTFlush|
   * |:CommandTLoad|



## TagBar

- [tagbar](https://github.com/majutsushi/tagbar)
- :help tarbar.txt
- key mappings:
  * nmap <F8> :TagbarToggle<CR>
  * Folding: o to fold/unfold
  * <F1>/?        Display key mapping help. Map option: tagbar_map_help
  * <CR>/<Enter>  Jump to the tag under the cursor. Map option: tagbar_map_jump
  * p             Jump to the tag under the cursor, but stay in the Tagbar window. Map option: tagbar_map_preview
  * P             Open the tag in a |preview-window|. Map option: tagbar_map_previewwin
  * <C-N>         Go to the next top-level tag. Map option: tagbar_map_nexttag
  * <C-P>         Go to the previous top-level tag. Map option: tagbar_map_prevtag
  * <Space>       Display the prototype of the current tag. Map option: tagbar_map_showproto
  * v             Hide tags that are declared non-public.Map option: tagbar_map_hidenonpublic
  * +/zo          Open the fold under the cursor. Map option: tagbar_map_openfold
  * -/zc          Close the fold under the cursor. Map option: tagbar_map_closefold
  * o/za          Toggle the fold under the cursor. Map option: tagbar_map_togglefold
  * \*/zR          Open all folds by setting foldlevel to 99. Map option: tagbar_map_openallfolds,
  * =/zM          Close all folds by setting foldlevel to 0. Map option: tagbar_map_closeallfolds
  * zj            Go to the start of the next fold, like the standard Vim |zj|. Map option: tagbar_map_nextfold
  * zk            Go to the end of the previous fold, like the standard Vim |zk|. Map option: tagbar_map_prevfold
  * s             Toggle sort order between name and file order. Map option: tagbar_map_togglesort
  * c             Toggle the |g:tagbar_autoclose| option. Map option: tagbar_map_toggleautoclose
  * x             Toggle zooming the window. Map option: tagbar_map_zoomwin
  * q             Close the Tagbar window. Map option: tagbar_map_close

- COMMANDS:
  * :TagbarOpen [f/j/c]  - :TagbarOpen fj
  * :TagbarClose
  * :TagbarToggle
  * :Tagbar
  * :TagbarOpenAutoClose
  * :TagbarTogglePause
  * :TagbarSetFoldlevel[!] {number}
  * :TagbarShowTag
  * :TagbarCurrentTag
  * :TagbarGetTypeConfig {filetype}
  * :TagbarDebug [logfile]
  * :TagbarDebugEnd



## Jedi

I spent two days (must have) in total, trying to install jedi plugin to vim 7.3 but failed. The vim could not import/recognize jedi, always throwing errors like No Module named jedi.

Out of desperation, I created [an issue on jedi-vim project](https://github.com/davidhalter/jedi-vim/issues/616), looking for a solution. 

The main reason system-default vim could not recognize jedi plugin is due to the python vim compiled against originally. There are so many python versions installed on my mac OS X that, I failed to find the right version that used in vim. Even if I did, there are many so files that failed to be invoked through dependency. The detailed information are pasted in the issue page. I also tried neovim, does not work, and then the latest vim-8, still not working. 

I googled for a while and come to the conclusion that, it is practically impossible to use anaconda python in vim installation. And the only way to solve the issue is to install vim through homebrew channel. Try figuring out the real reason behind the error is a little waste of time for me at this moment, so I decided to let it slide and maybe come back to it later, when I am more experienced and familiar with these than now. 

Here are the commands used to [install latest vim-8 through homebrew](http://www.prioritized.net/blog/upgrading-vim-on-os-x), not the macvim:

```shell
brew install mercurial
brew install vim
brew link --overwrite vim  #to use latest vim
```
I also encountered with permission issue when doing brew link vim. As of writing, homebrew requires the contents of /usr/local to be chown'd to current username. 

```shell
sudo chown -R `whoami` /usr/local
brew link --overwrite vim
```

Now, to install jedi:

```shell
cd ~/.vim/bundle/ && git clone https://github.com/davidhalter/jedi-vim.git
```

- :help jedi-vim
- key mappings:
  * let g:jedi#goto_command = "<leader>d"
  * let g:jedi#goto_assignments_command = "<leader>g"
  * let g:jedi#goto_definitions_command = ""
  * let g:jedi#documentation_command = "K"
  * let g:jedi#usages_command = "<leader>n"
  * let g:jedi#completions_command = "<C-Space>"
  * let g:jedi#rename_command = "<leader>r"

Finally, jedi works as expected. And if you use jedi as plugin only, then no need to install it through pip as python module.

Jedi is great, with its source code navigation capability, I can totally get rid of IDE now. A big thanks & hugs to @davidhalter.

p.s.

As a reference or note for myself, here is the way to compile and install vim from source:

```shell
make distclean
./configure --enable-pythoninterp=yes --enable-rubyinterp=yes --enable-luainterp=yes --enable-perlinterp=yes --with-python-config-dir=/Users/phoenix/programs/anaconda/lib/python2.7/config
make
sudo make install
```

- [vim source code](https://github.com/vim/vim)
- Not sure the difference between yes/no/dynamic for enable-xxxinterp options. 



So far, the plugins that I need are all in place and ready for VIM-only-mode now.

