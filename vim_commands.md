
VIM Commands

yy: yank lines
p: paste below cursor
P: paste above cursor

fN: jump forward to first 'N'
3fN: jump forward to third 'N'

2j: Down 2 lines 
6l: forward 6 letters

u: undo
ctrl+R: redo

cw: change word
3cw CHange 3 words

w: forward one word
3w: forward 3 words

n: search again
N: search again backward

gg: go to top of file 
G: go to bottom of file

A: move to end of line 
o: open new line below
O: open new line above

:set list                   # turn a boolean value on
:set nolist                 # turn a boolean value off 
:set list?                  # show current value
:set list&                  # reset to default value

:set                        # get default values
:set filetype               # get filetype 
:colorscheme vividchalk     # change color theme 

# window management
ctrl+w s: split window horizontally
ctrl+w v: split window vertically
ctrl+w c: close window 
ctrl+w j: to switch to bottom window
ctrl+w k: to switch to top window
ctrl+w l: to switch to right window
ctrl+w h: to switch to left window

# shifting lines
N>>: shift N lines right 
N<<: shift N lines left 

:options        # to get options available

# set file syntax
:set syntax=apache

# searching with highlight
:set hlsearch
:set incsearch

# show line number
:set number

# Complex pattern
V3jd: Line Visual move 3 lines down and delete it

#========================
# Configuration of VIM
#========================
~./vimrc: Most setting goes here
~/.gvimrc: Graphic Only settings (font, window size)
~/.vim: Plugins, languages spec, color themes
