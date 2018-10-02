Prerequisites
=============

## Assumptions

Throughout this tutorial-style introduction to working with HPC codes, we will assume that the reader:

- Is familiar with the C programming language.
- Is familiar with using a [command line](https://en.wikipedia.org/wiki/Command-line_interface) in a [Linux](https://en.wikipedia.org/wiki/Linux) environment.
Since the supercomputers used for this course are accessed _exclusively_ through a [terminal](https://en.wikipedia.org/wiki/Terminal_emulator), this is _essential_.
This tutorial assumes that the shell used is [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)).
- Can efficiently navigate documentation and reference material, _both_ offline, e.g. [man pages](https://en.wikipedia.org/wiki/Man_page) or language specifications, and online, e.g. discussion forums or mailing lists, in order to _independently_ find answers to questions and solve programming and system usage issues.

## What this tutorial is _not_

While there may be _some_ overlap with the lectures, particularly concerning tools and programming frameworks, it is important to keep in mind that **this tutorial is not a replacement for the lectures**.
It also does not aim to:

- Give a complete course on any of the programming languages and frameworks discussed
- Provide a "walkthrough" for the coursework
- Explain how to write a good report (although this will be covered in class)
- Replace engagement in labs, including any feedback, suggestions, and clarifications
- Deter students from taking novel approaches or in any way limit their use of technologies not discussed in these pages
    - For example, even though this repository shows sample code in C, feel free to write Fortran!
- Substitute for interactive learning, e.g. in lectures, labs, and on the forum

## Required (and useful) tools

To connect to the BlueCrystal supercomputer, you will need an [SSH](https://en.wikipedia.org/wiki/Secure_Shell) client.
If you use any Unix derivative, chances are you have a working `ssh` command for this purpose; if you're on Windows, some of your options are listed [below](#suggestions-for-windows-users).

To write your code, we suggest you use an editor that you can customise to your liking, e.g. indentation or syntax highlighting.
Many editors have out-of-the-box support for the languages you will likely use throughout this course; these include:

- [Visual Studio Code](https://code.visualstudio.com/) and [Atom](https://atom.io/), which are modern open-source text editors with rich plugin support, although they are written in JavaScript.
- [gedit](https://en.wikipedia.org/wiki/Gedit) or [Kate](https://en.wikipedia.org/wiki/Kate_(text_editor)), as your [DE](https://en.wikipedia.org/wiki/Desktop_environment)'s packaged text editor.
- [Vim](https://en.wikipedia.org/wiki/Vim_(text_editor)) and [Emacs](https://en.wikipedia.org/wiki/Emacs), in the terminal.

If you want to be able to compile and run your code locally you will need a compiler supporting your languages and frameworks.
Although this is _not strictly required_, it may ease and speed up debugging and correctness testing.
For C, both [GCC](https://en.wikipedia.org/wiki/GNU_Compiler_Collection) and [LLVM/Clang](https://llvm.org/) are available for virtually all Linux distributions and support OpenMP.
As as student, you can also get access to a copy of [the Intel Compiler](https://software.intel.com/en-us/parallel-studio-xe/choose-download/student-linux-fortran).
To compile MPI programs, you will need to install an MPI implementation, with the most common choices being [OpenMPI](https://www.open-mpi.org/) and [MPICH](http://www.mpich.org/); it does not matter which one you choose, but make sure you are not following the documentation for the other choice!

To transfer files between your machine and the supercomputer, you can use `scp`, which is part of OpenSSH and likely already available on your Linux box. There is also [rsync](https://en.wikipedia.org/wiki/Rsync), which may speed up repeated trasnfers.
However, manually transferring files every time you make a change is cumbersome, so a better alternative is to set up [SSHFS](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh) to mount a remote directory as a virtual drive on your local machine.
Another option is to set up your text editor to automatically sync with a remote folder ([example for VS Code](https://marketplace.visualstudio.com/items?itemName=mkloubert.vscode-remote-workspace)).

We provide the starting code and some of the examples through GitHub repositories.
Although you can obtain the files without using `git`, we _strongly_ encourage you to use version control for your assignment.
Without such a system, you will find yourself saving multiple copies of your files with attempted optimisations, and you risk losing track of which changes stay and which go.
If you are not familiar with Git, a good starting point is [the Atlassian tutorial series](https://www.atlassian.com/git/tutorials)—use it, it may well save you a great deal of wasted effort!

### Suggestions for Windows Users

While we _encourage_ you to use a Linux machine, or at least a [*nix](https://en.wikipedia.org/wiki/Unix-like) environment, this is _not strictly required_.
If you use Windows, you are advised to:

- Set up a more _Linux-like_ environment, in order to avoid many (sometimes subtle!) inter-platform issues.
Avoid using `cmd`, as it lacks features; some better options are (in rough order of "niceness"):
    - Make use of [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/about) to obtain a full Unix shell. This will give you an environment which is virtually indistinguishable from running full Linux for the purpose of this course. [WSLtty](https://github.com/mintty/wsltty) is a very good terminal for use with WSL; [ConEmu](https://conemu.github.io/) is also good, but you may run into text display issues.
    - If you are not on Windows 10 (although arguably you should be!), [Cygwin](https://cygwin.com/) can provide a Linux-like environment that makes many Linux tools available on Windows.
    - Install Linux in a [Virtual Machine](https://en.wikipedia.org/wiki/Virtual_machine). Both [VirtualBox](https://www.virtualbox.org/) and [VMWare](https://www.vmware.com/uk/products/workstation-player/workstation-player-evaluation.html) provide free virtualisation software. If in doubt which Linux distribution to choose, go with [Ubuntu](https://www.ubuntu.com/) (for large community support), [Fedora](https://getfedora.org/) (for a stable an up-to-date distribution), or [Debian](https://www.debian.org/) (for long-term stability).
    - Install [Git for Windows](https://git-scm.com/downloads) and choose the option to use the packaged terminal (mintty). This will have Bash and OpenSSH packaged in alongside git.
    - ConEmu (mentioned above) includes a packaged, but minimal, version of Bash that you can use without either WSL or Cygiwn. This _may_ be enough for your needs. [cmder](http://cmder.net/) is ConEmu with more sane defaults and Bash and OpenSSH built-in.
    - Use [Microsoft's OpenSSH for Windows](https://blogs.msdn.microsoft.com/commandline/2018/01/22/openssh-in-windows-10/) or a stand-alone SSH client ([PuTTY](https://putty.org/) is a reliable and featureful choice) to allow you to connect to the supercomputer remotely. Note that if you choose this option only, _you will not be able to test your code on your own machine_. This likely needs to be paired with a [SFTP](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol) client, such as [WinSCP](https://winscp.net/eng/index.php).
- Read about [issues with line endings](https://help.github.com/articles/dealing-with-line-endings/#platform-windows) and make sure you work around them. Also keep in mind that on Windows [NTFS](https://en.wikipedia.org/wiki/NTFS) is _case-insensitive_ ([_but case-preserving_](https://superuser.com/questions/364057/why-is-ntfs-case-sensitive)!), so a file that looks as if it's named `Makefile` may actually be `MaKEfILe`—take care when (re)naming files.
- Use a text editor designed with programming in mind, and **not** Notepad or—even worse—Word or WordPad. This will help you avoid silly mistakes such as [spaces instead of tabs in Makefiles](https://stackoverflow.com/a/28720186).
[Visual Studio Code](https://code.visualstudio.com/) is a very good (extensible) editor, and [Notepad++](https://notepad-plus-plus.org/) is a more lightweight alternative. There is a related issue with file extensions: if you have those hidden (which is the default in Windows), make sure you editor _doesn't_ automatically add one, e.g. `Makefile.txt`.

Note that the rest of this tutorial assumes a Linux environment, and so will only show command examples for Linux.
If you use Windows, or software that differs from our standard choices, _you_ are responsible for setting it up in such a way that you can follow the tutorial.
As long as you choose reasonably popular software, there should be plenty of support available online, and we will do our best to highlight any issues that we know might impact non-Linux platforms.
