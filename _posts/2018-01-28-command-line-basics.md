---
layout: post
title:  "Command line basics"
date:   2018-01-28
---

The command line is a place where you input commands that your computer will execute. These can be simple commands like
creating a folder, but also much more complicated ones. On Linux and Mac, most people use the Terminal application to
enter commands, while on Windows I suggest you use PowerShell.

If you forget what a command does or how exactly to use it (which happens to me all the time) you can look it up on
[tldr.ostera.io](https://tldr.ostera.io), or if you want more detail (more than you need 99% of the time) you can
type `man <command>`, for example `man ls` if you want to know everything about `ls`.

We will stick with the following commands. Note that I use `<argument>` to denote an *argument*, i.e. a path or a name
of a file or command that applies to your case. You shouldn't actually type `<` and `>`.

- `pwd` -- print working directory.
- `ls` -- list working directory.
- `cd <path>` -- change directory.
  * `cd Code` -- switches to folder named "Code".
  * `cd ..` -- switches to the parent of the current directory.
  * `cd` -- (no arguments) switches to your home folder
- `cp <file> <destination>` -- copies file.
  * `cp poem.txt copied_poem.txt`
- `mv <file> <destination>` -- moves file.
  * `mv poem.txt ../poem.txt` -- move `poem.txt` to the parent of current folder.
  * `mv poem.txt another_name.txt` -- rename `poem.txt` to `another_name.txt`.
- `rm <file>` -- removes file.
- `mkdir <dirname>` -- creates directory.
- `rmdir <dirname>` -- removes directory.
- `man <command>` -- shows manual page of command.
- `clear` -- clears terminal window.
- `python <program>` -- our star: calls the Python interpreter to execute a program.
  * `python hello_world.py`
  * `python` -- (no arguments) opens interactive Python interpreter.

Two *very* useful tricks:

- press the up and down arrows to cycle through commands you've entered previously.
- press Tab to auto-complete a command you're currently typing.
