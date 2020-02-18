---
layout: default
title: Linux IPC and pipes
<!--license: https://github.com/DavidKendall/blinky/blob/master/LICENSE-->
---
* TOC
{: toc}

## Introduction

This lab has a number of elements:

1. Practice with C structs, arrays and pointers
2. Linux IPC with signals
3. Linux IPC with pipes

## C structs, arrays and pointers

### Introduction
You need to understand how to use C structs, arrays and pointers in order to develop a strong understanding of operating systems. 
You should already have a good C text book. There is also some useful material online. You can also have a look at lecture slides of week 2.

<!--
See the [module page](http://hesabu.net/kf4005/).
-->

### What to do

1. Read Section 3 Complex Data Types in [Essential C](http://cslibrary.stanford.edu/101/EssentialC.pdf).
2. Create a program called `c_structs_and_pointers.c` from the example below.

     ```c
     #include <stdio.h>

     struct fraction {
     int numerator;
     int denominator;
     };

     void printFraction(struct fraction f) {
     printf("%i/%i", f.numerator, f.denominator);
     }

     int main () {
     struct fraction f1;
     struct fraction f2;

     f1.numerator = 21;
     f1.denominator = 3;

     f2 = f1;

     printf("f1 = ");
     printFraction(f1);
     printf("\n");
     printf("f2 = ");
     printFraction(f2);
     printf("\n");
     }
     ```
     {: .code}
Read through it carefully. Compile and run the program. 

<!--
Ask your lab tutor
about any aspects that you don't understand.  
In the exercises that follow,
make sure that you compile and run the program after every modification. Don't
leave compiling and running to the end.
-->

Exercises

1. Edit the program to add two new variables of type `struct fraction`, called `f3` and `f4`. Add assignment statements to
set the numerator of `f3` to 24 and its denominator to 6. Do the same for `f4` making its numerator and denominator 48 and 8,
respectively. Add statements to print out the details of `f3` and `f4`, similarly to the way that `f1` and `f2` are printed
in the example.
2. Add two new integer variables called `num` and `denom` to the program. Write assignments to make the value of `num` equal
to the numerator of `f3` and the value of `denom` equal to the denominator of `f4`. Add print statements to print out the
values of `num` and `denom`.

<!--
5. Declare an array `a` of 5 variables of type `struct fraction`. Write a function `initFractionArray` to initialise the numerators and 
denominators of all elements in the array to the value 1 (use a loop). Call `initFractionArray` from your `main` function
to initialise `a`. Now add code to your main function to set the values
of the numerator and denominator of the third element in the array to the same values as f2, and the values of the 
numerator and denominator of the fourth element in the array to 6 and 12, respectively. Add code to print out the values
of all elements of `a`.
6. Declare a variable `p1` to be a pointer to a `struct fraction`. Write an assignment statement to make `p1` point to the
fraction `f1`. Use the `*` notation with `p1` to assign the value 5 to the numerator of the fraction that it points to.
Use the `->` notation with `p1` to assign the value 25 to the denominator. Write statements to print the values of
the fraction `f1` and the fraction pointed to by `p1`. Using a single assignment statement, assign the value of the fraction pointed to by `p1` to the 
fraction `f3`. Print out the value of `f3`. 
7. Write an assignment statement to make `p1` point to the third element of the array `a`. Now assign the value of `f4` to the fraction
pointed to by `p1`. Print out the values of all the variables in your program. Make sure that you can explain the results that you see.
-->





## Linux IPC with signals

### Introduction

A very basic form of inter-process communication in Unix is the use of *signals*. Signals are 
described in [this lecture](http://hesabu.net/kf4005/assets/ra/B05.pdf). 

### Using signals in a C program

The program below gives a simple example of the use of a signal in a C program.
The `alarm()` function causes the `SIGALRM` signal to be generated at the end
of a specified period of time, e.g. `alarm(2)` generates the signal after 2
seconds. The `pause()` function simply waits until *any* signal is generated.
The program shows how to install your own handler for the `SIGALRM` signal. You
can use this approach to install a handler for any signal.

`struct sigaction` is a C struct that includes the fields
* `sa_handler` - this should be set to the function that you want to act as the
  handler for the signal. In the example below, the handler function is
  `catchAlarm()`. Notice that the type of this function shows that it takes a
  single integer argument and returns a void result, i.e. `void catchAlarm(int
  sig)`. Every signal handler function that you write must have a type like
  this.
* `sa_mask` - this is a value that determines which other signals will be
  *blocked* while your signal handler function is executed. If `act` is a
  `struct sigaction` then calling `sigfillset(&act.sa_mask)` ensures that *all*
  signals will be blocked.
* `sa_flags` is a field that controls some aspects of the signal handling. For
  the moment, it is fine for you to accept the default behaviour by setting the
  value of this field to 0.

   ```c
   #include <signal.h>
   #include <unistd.h>
   #include <stdlib.h>
   #include <stdio.h>

   #define TIMEOUT_SECS 2

   int tick = 0;

   void give_up(char *msg) {perror(msg); exit(1);}
   void catchAlarm(int ignored) {tick += 1;}

   int main() {

     struct sigaction act;

     act.sa_handler = catchAlarm;
     if (sigfillset(&act.sa_mask) < 0) {
       give_up("sigfillset");
     }
     act.sa_flags = 0;

     if (sigaction(SIGALRM, &act, 0) < 0) {
       give_up("sigaction");
     }

     do {
       alarm(TIMEOUT_SECS);
       pause();
       printf("Tick %i\n", tick);
     } while (1);
   }
   ```
   {: .code}

Run this program and have a look at what happens.

<!--
1. Create a program called `ticker.c` using the program above. Compile and
   build the program. Run it and observe its behaviour. Make sure that you
   understand the program.  Ask your lab tutor about any aspects that you're
   not clear about.

2. Add code to this program to install your own handler for the `SIGQUIT`
   signal. Your handler should just print the message `"SIGQUIT handler
   called"` and allow the program to continue execution. Build and test your
   program.
-->

### Using signals in a shell script

It is also possible to handle signals in a shell script. The `trap` command can
be used to install your own signal handler. It is good practice to write a
shell function to act as the handler. See the program below for examples of how
this is done.

   ```sh
   #!/bin/bash

   SIG_handler() {
     echo "This is a signal handler for SIGQUIT and SIGTERM"
     exit 0
   }

   SIGINT_handler() {
     echo ""
     echo "This is the SIGINT handler"
     read -p "Press ENTER ..."
   }

   EXIT_handler() {
     echo "This is the EXIT handler"
     exit 0
   }

   trap SIG_handler SIGQUIT SIGTERM
   trap SIGINT_handler SIGINT
   trap EXIT_handler EXIT

   while true
   do
     echo "Hello"
     sleep 1
   done
   ```
   {: .code}

1. Create a shell script called `trap_demo` that contains the code above. Make
   the the script executable. Run it and observe its behaviour. 

2. Try pressing Control-C and Control-\ while your script is running. What
   happens? Why?

<!--
3. Run the script again. Discover the process id your running script. In
   another terminal, use the `kill` command to send the `TERM` signal to the
   process. What happens? Why?
-->

## Linux IPC with pipes

### Introduction

<!--
You have already seen many examples of the use of pipes. What you are aiming to
do here is to clarify your understanding of what the shell does when you enter
a command that contains the pipe symbol (`|`).  You will also look at a simple
example of using a *named pipe*.
-->

You will look at examples of the use of *pipes* and *named pipe* next.

### Simple pipes


     $ ls -l | wc -l


The pipe is indicated using the (`|`) symbol and causes the standard
output from the first command to be piped to the standard input of
the second command.


<!--
1. Create a shell script that will print `Hello` 30 times at 1 second
   intervals. Call this script `hello30`.  Make the script executable.

2. Execute the script `hello30` and pipe its output into the command `wc -l`.
   What behaviour do you expect from this pipeline?

3. Discover the process id of your terminal. Use `echo $$` to do this. Run the
   pipeline again.

4. While your pipeline is running, in another terminal window, use the `ps`
   command to identify which processes have been created to run the pipeline.
   Explain what you observe.
-->

### Named pipes

In the shell we can use the *mkfifo* command

$ mkfifo mypipe

$ ls -l mypipe

$ cat /etc/init.d/apport >mypipe

In a different terminal, execute

$ cat <mypipe

and you’ll see the output from the command that’s running in the first terminal.






<!--
Read the section on named pipes in the [lecture](http://hesabu.net/kf4005/assets/ra/B05.pdf)

1. Implement the example from the lecture and observe the behaviour.

1. Write a C program that writes "Hello named pipe world" to `stdout`. Write a second C program that reads
   a string from `stdin`. Open a terminal window. Create a named pipe. Run your first program with output 
   redirected to the named pipe. Open a second terminal window. Run the second program with input redirected
   from the named pipe. Observe what happens. Use the `ls -l` command to examine the named pipe.

1. Repeat the previous exercise but this time using shell scripts.
-->