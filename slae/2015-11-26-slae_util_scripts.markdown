---
title: SLAE Utility Scripts
author: Reuben Sammut
---

This is a short post about the scripts I created to help me in shellcode development.

###genasm.sh

Let’s start with the most basic of scripts. This scripts generates a .asm file with the given name. The .asm file will contain boilerplate code for an assembly source file. Once created, the script also launches the vim text editor to start editing the source file.

To generate sample.asm use

~~~~~{.shell}
  $  genasm.sh sample
~~~~~

###compile.sh

This script is a slightly modified version of the compile script given in the course. There are two modifications from the original version of the shell script. The first modification is that the script expects a .asm file. Expecting a .asm file is a personal preference. For me this has the advantage of tab completion which hurries the process of compilation. The second modification is that the script will halt if any of the compilation steps fail.

To compile sample.asm use

~~~~~{.shell}
  $  compile.sh sample.asm
~~~~~

###genshell.sh

This script is a personal implementation of the command line fu described in class to extract the shellcode from a binary. It also splits the shellcode into 80 character lines for better management. Finally, instead of displaying the shellcode on the screen, it writes the shellcode to a file with the extension .shell. The reason for creating the .shell file will become clear as I explain the compile\_shell.sh script.

To generate sample.shell from the sample executable use

~~~~~~{.shell}
  $  genshell.sh sample
~~~~~~

###compile\_shell.sh

The final shell script I use is compile\_shell.sh. This script is used to grab the shellcode generated from genshell.sh and compile it with the standard C template used to test shellcodes. The script expects a .shell file, it generates a temporary C file with the C shellcode template file and in place of the shellcode, it puts a `#include` of the .shell given in the script. Once created, the file is compiled with the `-fno-stack-protector` and `-z execstack` flags and the C temporary file is removed.

To compile sample.shell as the executable sample\_shell use

~~~~~{.shell}
  $  compile_shell.sh sample.shell
~~~~~

#reverse.hs

Sometimes, in shellcode development, you need to push some string on to the stack. This involves the arduous task of converting the string to its hexadecimal counterpart, reversing the hexadecimal strings, checking lengths and padding to avoid null bytes. For this task I created a simple Haskell program to do this for me. This program takes a string with an optional parameter. Should no extra parameter be given, the string is prepended with the ‘/’ character (as most of the times, you would need to push a path on the stack) until the length is a multiple of 4. The extra parameter could be one of two. In case the extra parameter is –trim, then the exact same process as above is done. However, in the end, the $esp is adjusted to ignore the prepended characters. The extra parameter could also be –trim-safe. When using this parameter, no characters are prepended to the string. Instead, when the length is not a multiple of 4, the last few bytes are pushed as follows:

- 1 byte remaining – use push 0xNN
- 2 bytes remaining – use push word 0xNNNN
- 3 bytes remaining – use push 0xNN; push word 0xNNNN

In this way, null bytes can be avoided without adding padding characters. All other characters are pushed as normal: push 0xNNNNNNNN

To compile reverse.hs use

~~~~~{.shell}
  $  ghc --make reverse.hs
~~~~~

To use the reverse program use one of the following

~~~~~{.shell}
  $  reverse "Hello world"
  $  reverse --trim "Hello world"
  $  reverse --trim-safe "Hello world"
~~~~~

These scripts can be found on [my GitHub account](http://github.com/reubensammut/SLAE32/tree/master/Scripts).
