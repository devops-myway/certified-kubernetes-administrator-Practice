##### New Mac
https://stackoverflow.com/questions/29955500/code-is-not-working-in-on-the-command-line-for-visual-studio-code-on-os-x-ma

- new macbook dont come preinstalled with: .bashrc or .zshrc files and you should use
- ls -l -A, create the files with touch .bashrc or touch .zshrc
- Add this to your ~/.bash_profile, or to ~/.zshrc if you are running macOS v10.15 (Catalina) or later. (To add, cd ~ and vim ~/.bash_profile in terminal)

``````sh
cd ~
ls -l -A
vi .bashrc
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
Save and close using :wq
source ~/.bashrc
open iterm and code . # open vscode

``````
``````sh


``````
