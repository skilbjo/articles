# Operating Systems

## What

A program that allows other programs to run on the computer, but maintains
control of system resources, preventing one program from going rogue. Allows
for abstractions that allows the computer to maximize resource allocation given
many programs normally run at once.

## How it works

### History

Altas Supervisor
CTSS
Multics
Unix
Linux, GNU, BSD

## Programs and Processes

Programs are a written instructions for the computer to execute. A process is a
running invocation of a program.

System call with switch into kernel mode, kernel code will read the cat program
from disk, create a stake/heap.

### Fork / exec (System calls)

## Kernel, Userland, System Call

## Process Execution and Scheduling

Timer interrupt, hardware interrupts

Context switching involves saving program state (values of registers,
intruction pointer, and what is in RAM)

Program A working -> working -> \*timer interrupt\* -> context switch to kernel
code -> do work to save program A state -> do work to load program B state ->
start execution of program B

## Concurrency, Threading, Parallel Programming

Concurrency - illusion of two or more things happening at once, when in reality
only one thing is happening at a time

Threads -

Threads vs Processes

Lightweight threads vs POSIX threads

Parallelism - two things actually happening at once. Many hazards!

## Hierarchical Filesystem

Abstractions of files, directors, links

Files are just random places on the hard drive. They are not physicallly next
to each other. As such, heirarchical filesystem is just a userland abstraction.
Directories are just files, with the file itself being a list of the files in the directory.
Heirarchical file path such as /usr/local/bin/some-file is
1: go to / and open up the directory
2: see where /usr directory is
3: go to /usr and open up the directory file
4: see where ./local directory file is
5: go to /usr/local/ and open up the directory file
6: see if some-file is a file, and if so go to that file location
7: open up /usr/local/some-file

As you can see, the deeper the file tree, the longer it takes the operating
system to find that file

inode

No such newlines

## Virtual Memory

Page size, page table

Program thinks it has all the RAM in the system. It also thinks it has more
memory than is actually in the system (ie 64 bit addressesable RAM - 64 bits is
a very large number, so that addressesable RAM space means far too many
addresses than actually exist with modern hardware)

If two processes share same instructions, page table will reference the same
physical memory (an optimization).

Translation lookaside buffer

## Inter-process Communication

How to pass messages between to processes?

Can use RAM, but that will get tricky very fast. When is RAM ready for writing,
and when is it ready for reading? Many hazards.

Mutex and semaphores

## Virtual Machines and Containers

Hypervisors

Virtual Machines

Containers: not a VM at all! Doesn't even try to be. Just provides process
isolation and can constrain resources (max CPU, max RAM). Needs to be on a
Linux system.
