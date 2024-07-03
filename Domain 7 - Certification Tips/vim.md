

##### https://thoughtbot.com/upcase/videos/onramp-to-vim-surviving-your-first-week

##### Normal Mode

navigate the shell with:
- J : Downward arrow botton
- K : Top arrow botton
- L : Right arrow botton
- H : Left arrow botton

#### Insert Mode
- i : switch to insert mode before the cursor
- Esc : exit insert mode; switch to command mode
- Ctrl + r : redo
- u : undo
- s : delete a character (and move into insert mode)

##### Efficient Commands to use to make wor easier

- W - Word
- e - end of
- b - back
- $ -  end of line
- 0 - beginning

##### Delete a line in vim
Press ESC to go to Normal mode.
Place the cursor on the line you need to delete or copy.


- dd – cut (delete) entire line - Press dd. This will delete the current line.
- yy – copy (yank) entire line - Press yy and move the new line and paste
- p – paste after the cursor - to paste

#### save and quit: 
If you are currently in insert or append mode, press Esc key.
Press : (colon). The cursor should reappear at the lower left corner of the screen beside a colon prompt

- :wq! - document will be saved to the file for all changes
- :w – save the file
- :wq / :x / ZZ – save and close the file
- :q – quit
- :q!/ ZQ – quit without saving changes

##### .vimrc
Configuring your .vimrc file lets you use the full power of Vim

#### Settings for vimrc file
Here are some more common setting that enhance the editing experience.
Each line contains a comment above it explaining what it does.

- set shiftwidth=4            Set shift width to 4 spaces.
- set tabstop=4               Set tab width to 4 columns.
- set expandtab               Use space characters instead of tabs.

