---
layout: post
title: Collection of Ubuntu problems and solutions
published: true
---


During my time spent with Ubuntu, I've encountered many problems. Writting them down in my blog can be a good way to back them up so next time I don't have to take the trouble to look for answers everywhere.

## Gnome-terminal Disapperance.

I encountered this problem from nowhere, I don't know what caused it. Suddenly I can't find the default gnome-terminal from the Application or by Control+Alt+T. All I got was Xterm. I found answers [here][1]. The solution is like this:

        sudo apt-get remove gnome-terminal
        sudo apt-get install gnome-terminal

## Ubuntu Desktop doesn't load.

I also don't know what caused this one. My side bar and top bar just disapper, and I can't use Control+Alt+T to open the terminal either. I found solution [here][2]. The solution is like this:

1.  Entering the text mode by Ctrl + Alt + F1
2.  Run commands below:

        sudo apt-get install gnome-panel
        sudo mv ~/.Xauthority ~/.Xauthority.backup
3.  Reboot and select gnome on login. Selecting gnome on login means clicking on the little ubuntu logo next to your user name when you log in. As shown by the picture below:

    ![gnome-log-in][3]
4.  Then open a terminal by Ctrl + Alt + T
5.  Run commands below:

        sudo apt-get install unity-tweak-tool
        unity-tweak-tool --reset-unity
6.  Reboot and log in again using unity(also by clicking the ubuntu logo next to your user name when you log in) this time and the bars are back!

## How to install Sublime Text3 on Ubuntu
Answers found [here][4]

        sudo add-apt-repository ppa:webupd8team/sublime-text-3
        sudo apt-get update
        sudo apt-get install sublime-text

[1]: http://askubuntu.com/questions/356842/ubuntu-default-terminal-missing-on-13-04
[2]: http://askubuntu.com/questions/476930/ubuntu-desktop-does-not-load
[3]: https://raw.githubusercontent.com/sunshineatnoon/sunshineatnoon.github.io/master/images/gnome-log-in.png
[4]: http://askubuntu.com/questions/172698/how-do-i-install-sublime-text-2-3




