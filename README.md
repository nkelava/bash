# Bash

A small Unix shell written in C, built as a university project. It reads a line,
splits it into arguments, and then runs either a built-in command or an external
program (`fork` + `execvp` + `waitpid`). The goal is to show how a shell works
inside, not to replace a real one.

The core loop is based on Stephen Brennan's tutorial
[Write a Shell in C][1]. This project adds more built-in commands, path expansion
for `~`, `.` and `..`, and a colored prompt that shows the current directory.

## Features

- An interactive prompt that shows the current working directory in yellow.
- Runs other programs found on your `PATH` (see [Limitations](#limitations)).
- Expands `~`, `~/`, `./`, `../` and plain relative names into absolute paths
  before a command runs.
- 15 built-in commands:

| Command | What it does |
| --- | --- |
| `cd <dir>` | Change the current directory |
| `help` | List the built-in commands |
| `exit` | Quit the shell |
| `whoami` | Print the current user (from `$USER`) |
| `programmer` | Print the project's author |
| `home`, `~` | Print the home directory path |
| `pwd` | Print the current directory |
| `echo <text...>` | Print the arguments |
| `time` | Print the current date and time |
| `ls [dir]` | List a directory, hiding dotfiles (defaults to the home directory) |
| `touch <file...>` | Create one or more empty files (truncates existing ones) |
| `cat <file...>` | Print the contents of one or more files |
| `cp <src...> <dest>` | Append one or more source files to `dest` |
| `mv <src> <dest>` | Move or rename a file; refuses to overwrite an existing `dest` |

Built-ins take priority over programs with the same name, so `ls`, `cat`, `cp`,
`mv`, `touch`, `pwd`, `echo` and `time` always run the built-in versions.

## Prerequisites

- A POSIX system (Linux or macOS; on Windows use WSL).
- A C compiler such as `gcc` or `clang`.

There are no external libraries and no build system; it uses only the C standard
library and POSIX headers.

## Installation

```sh
git clone https://github.com/nkelava/bash.git
cd bash
gcc -o bash src/*.c
```

All `.c` files in `src/` need to be compiled together. Compiling only
`src/main.c` fails at link time.

## Usage

Start the shell:

```sh
./bash
```

You get a prompt showing the current directory. Type a command and press Enter:

```text
/home/nikola/bash> pwd
/home/nikola/bash
/home/nikola/bash> whoami
User: nikola
/home/nikola/bash> cd ~/projects
/home/nikola/projects> ls .
bash
notes
/home/nikola/projects> time
Current time: Tue Oct 27 21:23:57 2026
/home/nikola/projects> exit
```

Type `help` to see the list of built-ins and `exit` to leave.

## Configuration

There are no config files or command-line flags. The shell reads two
environment variables:

| Variable | Used for |
| --- | --- |
| `HOME` | Expanding `~` and as the default directory for `ls` |
| `USER` | Output of `whoami` |

`HOME` must be set, or the shell crashes on startup.

## Project structure

```text
.
├── src/
│   ├── main.c            # Entry point; starts the read-eval loop
│   ├── logic.c/.h        # Prompt, line reading, tokenizing, dispatch, fork/exec
│   ├── builtins.c/.h     # Built-in command table and implementations
│   ├── path_handling.c/.h # Expands ~, ., .. and relative names into absolute paths
│   └── arguments.c/.h    # Helpers for counting and freeing argument arrays
└── README.md
```

To add a built-in, add its name to `builtin_str`, a description to
`builtin_info` and a function pointer to `builtin_func` in `src/builtins.c`,
keeping all three arrays in the same order.

## Limitations

This is a learning project and has rough edges:

- Commands must fit on one line, and arguments are split on whitespace only.
- No quoting, escaping, pipes, redirection, globbing or background jobs.
- The shell appends the home directory path as an extra hidden argument to every
  command, so external programs receive one more argument than you typed
  (`uname -s`, for example, fails with "extra operand").
- Path expansion is applied to every argument, so `echo hello` prints an
  absolute path rather than `hello`.
- `touch` can crash the shell with a heap corruption error, caused by an
  off-by-one buffer size in `src/path_handling.c`.
- The descriptions printed by `help` are in Croatian and are shifted by one line
  relative to the command names.

## Credits

The basic shell loop (read, parse, fork, exec, wait) follows
[Write a Shell in C][1] by Stephen Brennan. Additional built-ins and path
handling by Nikola Kelava.

[1]: https://brennan.io/2015/01/16/write-a-shell-in-c/
