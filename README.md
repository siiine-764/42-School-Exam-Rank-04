# Exam Question

This exam has 1 question, microshell:

- [Microshell.c](https://github.com/pasqualerossi/42-School-Exam-Rank-04/blob/main/microshell.c)
- [Microshell.h](https://github.com/pasqualerossi/42-School-Exam-Rank-04/blob/main/microshell.h)

if you can make this code shorter, but readable, let me know!

<br>

## Excepted Files

- microshell.c

- microshell.h

## Subject Text

Allowed functions: 

> malloc, free, write, close, fork, waitpid, signal, kill, exit, chdir, execve, dup, dup2, pipe, strcmp, strncmp


## The Program
Write a program that will behave like executing a shell command

- The command line to execute will be the arguments of this program

- Executable's path will be absolute or relative but your program must not build a path (from the PATH variable for example)

- You must implement "|" and ";" like in bash
	- we will never try a "|" immediately followed or preceded by nothing or "|" or ";"

- Your program must implement the built-in command cd only with a path as argument (no '-' or without parameters)
	- if cd has the wrong number of argument your program should print in STDERR "error: cd: bad arguments" followed by a '\n'
	- if cd failed your program should print in STDERR "error: cd: cannot change directory to path_to_change" followed by a '\n' with path_to_change replaced by the argument to cd
	- a cd command will never be immediately followed or preceded by a "|"

- You don't need to manage any type of wildcards (*, ~ etc...)

- You don't need to manage environment variables ($BLA ...)

- If a system call, except execve and chdir, returns an error your program should immediatly print "error: fatal" in STDERR followed by a '\n' and the program should exit

- If execve failed you should print "error: cannot execute executable_that_failed" in STDERR followed by a '\n' with executable_that_failed replaced with the path of the failed executable (It should be the first argument of execve)

- Your program should be able to manage more than hundreds of "|" even if we limit the number of "open files" to less than 30.

## Example

for example this should work:
```
$>./microshell /bin/ls "|" /usr/bin/grep microshell ";" /bin/echo i love my microshell
microshell
i love my microshell
$>

>./microshell 
```

## Hints
- Don't forget to pass the environment variable to execve
- Do not leak file descriptors!

## Exam Practice Tool

Practice the exam just like you would in the real exam - https://github.com/JCluzet/42_EXAM

# Code Commented
```c
#include "microshell.h"

int ft_print(char *message, char *av)
{
    int i = 0;
    while (message[i])
    {
        write(2, &message[i], 1);
        i++;
    }
    if (av)
    {
        i = 0;
        while (av[i])
        {
            write(2, &av[i], 1);
            i++;
        }
    }
    write(2, "\n", 1);
    return (1);
}

void ft_cd(char **av, int j)
{
    if (j != 2)
        ft_print("error: cd: bad arguments", 0);
    else if (chdir(av[1]) == -1)
        ft_print("error: cd: cannot change ", av[1]);
}

void ft_exec(char **av, char **env, int j)
{
    int fd[2];
    int has_pipe = (av[j]) && (strcmp(av[j], "|") == 0);
    if ((has_pipe) && (pipe(fd) == -1))
    {
        ft_print("error: fatal", 0);
        return;
    }
    int status;
    int pid = fork();
    if (pid == 0)
    {
        av[j] = 0;
        if ((has_pipe) && (dup2(fd[1], 1) == -1 || close(fd[0]) == -1 || close(fd[1]) == -1))
        {
            ft_print("error: fatal", 0);
            return;
        }
        execve(*av, av, env);
        ft_print("error: cannot execute ", av[0]);
        return ;
    }
    waitpid(pid, &status, 0);
    if ((has_pipe) && (dup2(fd[0], 0) == -1 || close(fd[0]) == -1 || close(fd[1]) == -1))
    {
        ft_print("error: fatal", 0);
    }
}
int main(int ac, char **av, char **env)
{
    if (ac > 1)
    {
        int i = 1;
        int j = 0;
        while (ac > i)
        {
            j = 0;
            while(av[i + j] && (strcmp(av[i + j], "|") && strcmp(av[i + j], ";")))
                j++;
            if (strcmp(av[i], "cd") == 0)
                ft_cd(&av[i], j);
            else if (j)
                ft_exec(&av[i], env, j);
            i += j + 1;
        }
    }
}
```
