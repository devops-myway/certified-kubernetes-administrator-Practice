

##### https://thoughtbot.com/upcase/videos/onramp-to-vim-surviving-your-first-week

#### Insert
i – switch to insert mode before the cursor
Esc – exit insert mode; switch to command mode
Ctrl + r – redo
u – undo
s – delete a character (and move into insert mode)

##### Delete
dd – cut (delete) entire line
yy – copy (yank) entire line
p – paste after the cursor

#### save and quit: 
If you are currently in insert or append mode, press Esc key.
Press : (colon). The cursor should reappear at the lower left corner of the screen beside a colon prompt

:wq! - document will be saved to the file for all changes
:w – save the file
:wq / :x / ZZ – save and close the file
:q – quit
:q!/ ZQ – quit without saving changes

##### .vimrc
Configuring your .vimrc file lets you use the full power of Vim

#### Settings for vimrc file
Here are some more common setting that enhance the editing experience.
Each line contains a comment above it explaining what it does.

set shiftwidth=4            Set shift width to 4 spaces.
set tabstop=4               Set tab width to 4 columns.
set expandtab               Use space characters instead of tabs.



