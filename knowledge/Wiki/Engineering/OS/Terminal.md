
# Session

A terminal session refers to an instance of a terminal emulator application running on a computer, allowing the user to interact with the operating system through a command-line interface (CLI). In simpler terms, it's the time you spend working within a terminal window or console, issuing commands to the computer's operating system.

The scope of the exported variable is limited to the current session. Once you close the terminal window or end the session in another way (such as logging out), the exported variables are no longer available to subsequent sessions unless they are explicitly set again

# Session starting
1. **Shell Initialization**: When you start a terminal session, the terminal emulator (such as Terminal in macOS) launches a shell process. The default shell on macOS is usually Bash, but other shells like Zsh or Fish can also be used. The shell process is responsible for interpreting your commands and interacting with the operating system.
2. **Shell Configuration Files**: Upon startup, the shell reads configuration files to set up the environment. These files can include system-wide configuration files (like `/etc/profile`) and user-specific configuration files (like `~/.bash_profile`, `~/.bashrc`, `~/.zshrc`, etc.). These files can contain commands to set environment variables, define aliases, configure the prompt, and perform other initialization tasks.
3. **Environment Variable Setup**: The shell reads and processes environment variables specified in the configuration files. These variables affect the behavior of the shell and the programs it runs. Common examples include `PATH` (which specifies the directories where executable programs are located), `HOME` (which points to the user's home directory), and `USER` (which contains the username of the current user).
4. **Interactive Shell Prompt**: After completing the initialization steps, the shell presents you with an interactive prompt, indicating that it's ready to accept commands. This prompt typically includes information like the username, hostname, current directory, and a symbol indicating readiness to accept input (commonly `$` for regular users and `#` for root or superuser).
5. **User Interaction**: You can now interact with the shell by typing commands and pressing Enter. The shell interprets your commands, executes them, and displays the results.
6. **Shell Termination**: When you exit the terminal session (by typing `exit` or pressing Ctrl+D), the shell process terminates, and any resources associated with it are released.

# Sudo prefix
