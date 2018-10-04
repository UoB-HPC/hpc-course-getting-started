Connecting to BlueCrystal
=========================

Throughout the HPC course, you will use [BlueCrystal](https://www.acrc.bris.ac.uk/), the University of Bristol's supercomputer.
This part of the tutorial shows how to connect to and accomplish basic tasks on BlueCrystal.

## Before you start

You will connect to BlueCrystal from your shell, over SSH. Make sure you have read [the Prerequisites section](0_Prerequisites.md) and have set up your software accordingly.
If you are using a lab machine, everything is already set up.

In order to access BlueCrystal, you need to have an account.
If you don't have an account yet and haven't received any instructions, e.g. via email, on how to obtain one, you can find them on the unit's BlackBoard page.
If you have any issues signing up, please ask a lab helper.

In the sections below, shell commands are prefixed with a `$` symbol.
This is to indicate that you would type them at a shell prompt and _you do not need to type the `$`_.
In the following example, `echo hello` is the command typed into the shell, and `hello` (on the following line) is the output of the command.

```bash
$ echo hello
hello
```

Where a username is required, we either use the placeholder `<username>` or the example `ab12345`.
**Make sure you use _your own_ username in your commands and configuration files**.

BlueCrystal is split into several _phases_, which are system parts that have been added over time to keep up with hardware advances.
The phases are independent and differ slightly in their configuration, so it's important that you don't confuse them.
If you're taking the Introduction to HPC course (COMS30005), the you will be using [BlueCrystal Phase 3](https://www.acrc.bris.ac.uk/acrc/phase3.htm) (BCp3); for Advanced HPC (COMS30006), you will use [BlueCrystal Phase 4](https://www.acrc.bris.ac.uk/acrc/phase4.htm) (BCp4).

## Connecting

BlueCrystal is only accessible from inside the University's network.
If you require access from outside, e.g. from your home network, see [Connecting from outside the University](#connecting-from-outside-the-university).

From a laptop connected to eduroam, a lab machine, or a computer otherwise attached to the University's network, run the following command to connect to BCp3, replacing `<username>` with your UoB username:

```
$ ssh <username>@bluecrystalp3.bris.ac.uk
```

After you type your password, you should get a prompt showing your username and the system's hostname, for example:

```
[ab12345@newblue2 ~]$
```

For BCp4, only the hostname is different:

```
$ ssh <username>@bc4login.acrc.bris.ac.uk
[ab12345@bc4login4 ~]$
```

Note that you get automatically-generated passwords when you sign up for an account.
These are different between Phase 3 and Phase 4, and different again from your University account's password.
If you want to change the provided password, simply run `passwd` and follow the instructions.

### Running graphical programs

If you want to run a program that has a GUI, you need to enable X forwarding:

```
$ ssh -X <username>@bluecrystalp3.bris.ac.uk
```

If you get a `Can't open display error`, then you have not used `-X`.
You will need to log out and log back in using the forwarding option.
If you _can_ start GUI programs but parts of the interface are missing, try logging out and using the trusted forwarding option instead:

```
$ ssh -Y <username>@bluecrystalp3.bris.ac.uk
```

### Passwordless SSH access

You can simplify your login process by setting up authentication using an SSH key instead of your password.
The key is transmitted automatically, saving you from having to type your password for every connection to BlueCrystal.
Note that the content of this section applies to _any remote server running Linux_, not just BlueCrystal.

If you don't already have an SSH key, or if you want to use a new one for BlueCrystal, we will generate one now.
**On your local machine**, go into the `.ssh` directory in your home folder (creating it if it doesn't exist) and follow the instructions to create your key:

```
$ mkdir ~/.ssh # if it doesn't exist
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/andrei/.ssh/id_rsa): uob
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Although `rsa` should be the default type, we specify it for completeness.
We then ask for a 4096 bit key, which should be a good size to use in 2018.
You can give your key file any name when prompted to do so (`uob` is used in this example).

You can also set a password for your key, if you choose to.
If you do, then the key will be encrypted before being stored on your disk and you will have to enter the password to unlock it before it can be used.
If you don't set a password, no interaction will be required, but keep in mind that this means anyone can take your key and use it as if they were you!
There are ways to set up your key so that it only needs to be unlocked once per session, i.e. until you log off, but there are subtle security issues that could come up; see [this SO answer](https://unix.stackexchange.com/a/90869) for an in-depth look at your options, and make sure you understand the consequences of your actions!

Once you have your SSH key, observe that it is made up of two parts, a private key and a public key:

```bash
$ ls uob*
uob uob.pub
```

The private key (`uob`) _never leave your machine_.
The public key (`uob.pub`) needs to be uploaded on any server where you want to use it for authentication.
To do this, first make sure that you have a `.ssh` folder on BlueCrystal, and an `authorized_keys` file inside it with `0644` permissions:

```bash
$ ssh ab12345@bluecrystalp3.bris.ac.uk
[ab12345@newblue2 ~]$ mkdir .ssh
[ab12345@newblue2 ~]$ touch .ssh/authorized_keys
[ab12345@newblue2 ~]$ chmod 644 .ssh/authorized_keys
```

Then, coming back to your local machine, upload your public key to the `.ssh` folder on BlueCrystal (make sure you don't upload your private key by mistake!):

```bash
$ cat .ssh/uob.pub | ssh ab12345@bluecrystalp3.bris.ac.uk 'cat >> .ssh/authorized_keys'
```

Finally, configure SSH to use the key.
To do this, open the `~/.ssh/config` file (creating if it does not exist) and add a configuration for BlueCrystal:

```
Host bcp3
    HostName bluecrystalp3.bris.ac.uk
    User ab12345
    IdentityFile ~/.ssh/uob
```

The `Host` alias (`bcp3` in this case) can be set to anything you want and will be used as an identifier for this connection.
The rest of the parameters should be self-explanatory.
You should now be able to connect easily:

```bash
$ ssh bcp3
[ab12345@newblue2 ~]$
```

**Note**: There are several schools of thought regarding how many SSH keys you should be using (see [this SO discussion](https://security.stackexchange.com/questions/40050/best-practice-separate-ssh-key-per-host-and-user-vs-one-ssh-key-for-all-hos) for some revealing insight).
If you have followed this tutorial to create a key specifically for BlueCrystal (and potentially other UoB services), then you should add the following line at the very top of your `~/.ssh/config` to avoid [your machine offering any other SSH keys you might have](https://superuser.com/a/187790):

```
IdentitiesOnly yes
```

### Connecting from outside the University

If you want to connect to BlueCrystal from outside the University, you first need to somehow connect to the network.

Your first option is to use [the University's VPN](https://www.bris.ac.uk/it-services/advice/homeusers/uobonly/uobvpn/).
Once you set this up and connect to it, your traffic will be tunneled to a server on the University's network.
From there you can access BlueCrystal (and any other University resources) as if you were connected to eduroam.
**WARNING**: While connected to the VPN, _all your traffic will go via the University_, so make sure to disable it when you don't need it.

The other option is to connect to snowy, a server in the CS Department that exists specifically to be accessed from outside.
Once connected, you will be in an environment similar to a lab machine, from where you can connect to BlueCrystal.
To connect to snowy, use your UoB username and password:

```
$ ssh <username>@snowy.cs.bris.ac.uk
```

If you have [set up access using an SSH key](#passwordless-ssh-access), then you can take a step further and automate connecting to snowy and from there to BlueCrystal.
First, set up your SSH key on snowy following the same procedure you used for BlueCrystal and confirm it works, i.e. that you can connect without typing your password.
Then, add the following `ProxyJump` line to your SSH configuration for BlueCrystal (the example assumes that you have used the alias `snowy`):

```
ProxyCommand ssh -q -W %h:%p snowy
```

You can find more details about SSH proxy jumps [on WikiBooks](https://en.wikibooks.org/wiki/OpenSSH%2FCookbook%2FProxies_and_Jump_Hosts#Passing_Through_One_or_More_Gateways_Using_ProxyJump).
In particular, if you have a recent version of OpenSSH, then you can use the `ProxyJump` directive.

If you have followed all the steps so far, your SSH configuration file should contain the following (but using your own username, and potentially BCp4 instead of BCp3):

```
IdentitiesOnly yes

Host snowy
    HostName snowy.cs.bris.ac.uk
    User ab12345
    IdentityFile ~/.ssh/uob

Host bcp3
    HostName bluecrystalp3.bris.ac.uk
    User ab12345
    IdentityFile ~/.ssh/uob
    ProxyCommand ssh -q -W %h:%p snowy
```

If you want to [use GUI applications](#running-graphical-programs) when connecting with a proxy jump, you will need to use `-X` (or `-Y`) for _all_ the proxy connections.
If you use the `ProxyCommand` configuration, just add the flag to the `ssh` command line.

## Transferring files

The easiest way to transfer files to BlueCrystal is through SFTP, for which we will use the `scp` command.
`scp` works just like `cp`, in that its first argument is the source, the second one is the destination, and it will _not_ copy directories recursively unless given the `-r` flag.
In addition, you can specify a remote host for either the source or the destination (or even both, but there are [some subtleties](https://superuser.com/a/686527)) using the colon syntax:

```bash
$ scp bcp3:hello.c test.c #1
$ scp hello.c bcp3:test.c #2
```

In the example above, the first command _downloads_ `hello.c` from BlueCrystal into a local file called `test.c`.
The second command does the reverse process, _uploading_ the local `hello.c` file to a remote file called `test.c`.
Both commands assume you have followed the [SSH key setup above](#passwordless-ssh-access), which we recommend; if you haven't, then you will need to use the fully `username@host` syntax and enter your password:

```bash
$ scp hello.c ab12345@bluecrystalp3.bris.ac.uk:test.c
```

If you want to use a GUI for your file transfers, you can use:

- Your native file explorer on Linux
- [WinSCP](https://winscp.net/eng/index.php) on Windows
- [Cyberduck](https://cyberduck.io/) on Mac

## Editing files remotely

Depending on your editor of choice, you will either be editing files on your own machine and transferring them using, for example, SSHFS, or editing directly on the BlueCrystal.

If you want to edit files locally and transfer, some options are discussed in [_Prerequisites_](0_prerequisites.md#Required-(and-useful)-tools).

If your editor supports editing files remotely, then you can use your _local editor_ to open _files on BlueCrystal_).
For example, you can achieve this using [a VSCode plugin](https://marketplace.visualstudio.com/items?itemName=rafaelmaiolla.remote-vscode) or [the `scp://` scheme in Vim](http://vim.wikia.com/wiki/Editing_remote_files_via_scp_in_vim).

Finally, you can use a terminal editor on BlueCrystal.
Emacs, Vim, and nano are available, among others.
If you decide to use Vim and you haven't used it before, you can greatly improve the default configuration; see [amix's basic vimrc](https://github.com/amix/vimrc/blob/master/vimrcs/basic.vim) for a good starting point or [Will Price's vim configuration](https://github.com/willprice/dotfiles/tree/master/vim/.vim) for a more advanced setup.
Emacs has more sensible defaults, so you may find that it needs less tweaking.
Using nano on anything else than a temporary basis or in emergencies is controversial...

## Further resources

A variety of guides and cheat sheets on using BlueCrystal are available on the ACRC website: <https://www.acrc.bris.ac.uk/acrc/resources.htm>.
