# Minishell

## Overview
Minishell is an executable program written in C that imitates the functionality of a basic shell program. The shell implements `cd` and `exit`
as built-ins, fork/exec for all other commands, and handles signals appropriately.

## Functionality
The program is called from the command line and takes no arguments. It prints the current working directory (cwd) in square brackets followed immediately by a $ and a space as the prompt and then waits for user input. The cwd in the prompt is displayed in blue. Two commands are implemented manually: 'cd' and 'exit'.

**cd**

If 'cd' is called with no arguments or ~, it changes the working directory to the user's home directory. If 'cd' is called with one argument that isn't ~, it will then try to change the working directory to the directory specified in the argument. The command also supports ~ with trailing directory names such as 

    cd ~/Desktop

The minishell 'cd' command also supports changing into a directory whose name contains spaces, as long as the name is enclosed appropriately in double quotes:

    cd "some folder name"
    cd "some"" folder ""name"

Malformed directory names such as

    cd "Desktop
    cd "some" "folder""name"

will not go through and result in an error message.

**exit**

The 'exit' command causes the minishell to terminate and return 'EXIT_SUCCESS'. Using 'exit' is the only way to stop the minishell's execution normally.

### Other Commands

All other commands are handled by utilizing fork and exec. The program forks upon any other command other than the two mentioned above, and the child program will attempt to exec the given command while the parent process waits till the child is finished before returning to the prompt. Commands are assumed to contain no more than 4096 characters and that there will be no more than 2048 tokens on a line (including the trailing NULL to mark the end the vector).

### Error and Signal Handling

Errors for system/function calls are handled accordingly, with the minishell exiting gracefully and displaying a detailed error message, using "strerror(errno)". Examples are as follows but not limited to:

    "Error: Cannot get passwd entry. %s.\n"
    "Error: Cannot change directory to '%s'. %s.\n"
    "Error: Too many arguments to cd.\n"
    "Error: Failed to read from stdin. %s.\n"
    "Error: fork() failed. %s.\n"
    "Error: exec() failed. %s.\n"
    "Error: wait() failed. %s.\n"
    "Error: malloc() failed. %s.\n" 
    "Error: Cannot get current working directory. %s.\n"
    "Error: Cannot register signal handler. %s.\n"

The minishell captures the `SIGINT` signal. Upon doing so, it returns the user to the prompt. Interrupt signals generated in the terminal are delivered to the active process group, which includes both parent and child processes. The child will receive the `SIGINT` and deal with it accordingly. I made sure to use only async-safe functions in my signal handler, where I used a single volatile sig_atomic_t variable
`interrupted` that is set to true inside the signal handler. Then, inside the program's main loop that displays the prompt, reads the input, and executes the command, it doesn't do anything if that iteration of the loop was interrupted by the signal. If read fails, I made sure it wasn't simply interrupted before erroring out of the minishell. Finally, I set `interrupted` back to false before the next iteration of the main loop.

## Sample Execution

```
$ ./minishell

[/home/user/minishell]$ echo HI

HI

[/home/user/minishell]$ cd ..

[/home/user]$ cd minishell

[/home/user/minishell]$ cd /tmp

[/tmp]$ cd ~

[/home/user]$ cd minishell

[/home/user/minishell]$ ls

Makefile minishell minishell.c

[/home/user/minishell]$ pwd

/home/user/minishell

[/home/user/minishell]$ cd

[/home/user]$ pwd

/home/user

[/home/user]$ nocommand

Error: exec() failed. No such file or directory.

[/home/user]$ cd minishell

[/home/user/minishell]$ ^C

[/home/user/minishell]$ sleep 10

^C

[/home/user/minishell]$ exit

$

