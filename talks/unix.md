# UNIX

<img src='../lib/k&r-pdp11.jpg' width=500>

## What is it

Unix (trademarked as UNIX) is a family of multitasking, multiuser computer operating
systems that derive from the original AT&T Unix, developed starting in the 1970s
at the Bell Labs research center by Ken Thompson, Dennis Ritchie.

Initially intended for use inside the Bell System, AT&T licensed Unix to outside
parties from the late 1970s, leading to a variety of both academic and commercial
variants of Unix from vendors such as the University of California, Berkeley (BSD),
Microsoft (Xenix), IBM (AIX) and Sun Microsystems (Solaris). AT&T finally sold its
rights in Unix to Novell in the early 1990s, which then sold its Unix business to
the Santa Cruz Operation (SCO) in 1995, but the UNIX trademark passed to the
industry standards consortium The Open Group, which allows the use of the mark for
certified operating systems compliant with the Single UNIX Specification (SUS).
Among these is Apple's macOS, which is the Unix version with the largest installed
base as of 2014.

Many Unix-like operating systems have arisen over the years, of which Linux is the
most popular, having displaced SUS-certified Unix on many server platforms since
its inception in the early 1990s. Android, the most widely used mobile operating
system in the world, is in turn based on Linux.

## Name
Munics -> Multiplexed Information and Computing Service)
Unics -> UNiplexed Information and Computing Service)

## History and background

<img src='../lib/ibm704.jpg' width=700>

### Operating Systems
Why were operating systems needed?

The earliest computers were mainframes that lacked any form of operating system.
Each user had sole use of the machine for a scheduled period of time and would
arrive at the computer with program and data, often on punched paper cards and
magnetic or paper tape. The program would be loaded into the machine, and the
machine would be set to work until the program completed or crashed. Programs
could generally be debugged via a control panel using dials, toggle switches and
panel lights.

Symbolic languages, assemblers, and compilers were developed for
programmers to translate symbolic program-code into machine code that previously
would have been hand-encoded. Later machines came with libraries of support code
on punched cards or magnetic tape, which would be linked to the user's program
to assist in operations such as input and output. This was the genesis of the
modern-day operating system; however, machines still ran a single job at a time.
At Cambridge University in England the job queue was at one time a washing line
from which tapes were hung with different colored clothes-pegs to indicate
job-priority.

As machines became more powerful the time to run programs diminished, and the time
to hand off the equipment to the next user became large by comparison. Accounting
for and paying for machine usage moved on from checking the wall clock to automatic
logging by the computer. Run queues evolved from a literal queue of people at the
door, to a heap of media on a jobs-waiting table, or batches of punch-cards stacked
one on top of the other in the reader, until the machine itself was able to select
and sequence which magnetic tape drives processed which tapes.

IBM (manufacturer of most computers) was slow to introduce operating systems:
General Motors produced General Motors OS in 1955 and GM-NAA I/O in 1956 for use
on its own IBM computers; and in 1962 Burroughs Corporation released MCP and General
Electric introduced GECOS, in both cases for use by their customers.

The first operating systems for IBM computers were written by IBM customers
who did not wish to have their very expensive machines sitting idle while operators
set up jobs manually, and so they wanted a mechanism for maintaining a queue of jobs.

Through the 1950s, many major features were pioneered in the field of operating
systems, including batch processing, input/output interrupt, buffering, multitasking,
spooling, runtime libraries, link-loading, and programs for sorting records in
files.  These features were included or not included in application software at
the option of application programmers, rather than in a separate operating system
used by all applications.

OS/360 (an IBM OS from the mid 60s) pioneered the concept that the operating system
keeps track of all of the system resources that are used, including program and
data space allocation in main memory and file space in secondary storage, and file
locking during update.  When the process is terminated for any reason, all of these
resources are re-claimed by the operating system.


### CTSS (Time sharing)

> "Much of my work has come from being lazy. I didn't like writing programs, and
> so, when I was working on the IBM 701, writing programs for computing missile
> trajectories, I started work on a programming system to make it easier to write
> programs."
 -- John Backus

John Backus (creator of first high-level language- FORTRAN, Backus-Naur Form) said
in the 1954 summer session at MIT that "By time-sharing, a big computer could be
used as several small ones; there would need to be a reading station for each user".
However, the computers (IBM 704, were not powerful enough to implement. In June
1959, Christopher Strachey published a paper "Time Sharing in
Large Fast Computers" at the UNESCO Information Processing Conference in Paris,
where he envisaged a programmer debugging a program at a console (like a teletype)
connected to the computer, while another program was running in the computer at
the same time. Debugging programs was an important problem at that time, because
with batch processing, it then often took a day from submitting a changed code,
to getting the results. John McCarthy wrote a memo about that at MIT, after which
a preliminary study committee and a working committee were established at MIT, to
develop time-sharing. The committees envisaged many users using the computer at
the same time, decided the details of implementing such system at MIT, and started
the development of the system.

### Bell Labs
The French government awarded Alexander Graham Bell $10,000 for the invention of the
telephone; which Bell used to fund the Volta Laboratory. In 1925, the engineering
department of the American Telephone & Telegraph company and Western Electric Laboratories
consolidated to form a seperate entity; ownership of which was shared by AT&T and Western
Electric.

8 Nobel Prizes have been awarded to people for work completed at Bell Labs. Noteably,
in 1947 the transistor was invented my John Bardeen, Walter Houser Brattain, and William
Bradford Shockley. Ken Thompson and Dennis Ritchie were awarded in in the early 80s
for their work on operating system theory and for developing Unix.

### Unix family tree
Dates are when development work first started; not release date.

Pre history (tools for calculation, harnessing electric logic, processing one program):
- (2300BC, 600BC, 500BC, 200BC) Abacus -> Mesopotamian, Persian, Greek, Chinese
- (1642) Pascal's calulator
- (1786,1823) Difference Machine & Difference Engine -> J.H. Muller (engineer in Hessian army), Charles Babage
- (1835) Analytical Engine -> Charles Babage & Ada Lovelace (gears, cogs, wheels)
- (1907) Vacuum tube
- (1940) Plugboards (vacuum tubes replacing mechanical relays)
- (1947) Transistor
- (1958) Integrated circuit

(batch processing -> multiprogramming -> time sharing)
- (1957) Atlas Supervisor & BESYS (Bell Operating System)
- (1961) Compatible Time-Sharing System (CTSS)
- (1964) Multics
- (1969) UNIX -> (1978) *BSD -> (1987) Mach -> (2000) Darwin
- (1985) Plan 9
- (1987) Minix
- (1991) Linux
- (1996) Inferno

<img src='../lib/unix-history-simple.png' width=700>

<img src='../lib/unix-history-super-simple.png' width=700>

## Philosophy
The Unix philosophy, originated by Ken Thompson, is a set of cultural norms and
philosophical approaches to minimalist, modular software development.  Unix developers
were important in bringing the concepts of modularity and reusability into software
engineering practice, spawning a "software tools" movement.

### The Bell System Technical Journal (1978)
> Make each program do one thing well. To do a new job, build afresh rather than
> complicate old programs by adding new "features".

> Expect the output of every program to become the input to another, as yet unknown,
> program. Don't clutter output with extraneous information. Avoid stringently columnar
> or binary input formats. Don't insist on interactive input.

> Design and build software, even operating systems, to be tried early, ideally within
> weeks. Don't hesitate to throw away the clumsy parts and rebuild them.

> Use tools in preference to unskilled help to lighten a programming task, even if
> you have to detour to build the tools and expect to throw some of them out after
> you've finished using them.

## Mike Gancarz (author of X windowing system): The UNIX Philosophy
> Small is beautiful.
> Make each program do one thing well.
> Build a prototype as soon as possible.
> Choose portability over efficiency.
> Store data in flat text files.
> Use software leverage to your advantage.
> Use shell scripts to increase leverage and portability.
> Avoid captive user interfaces.
> Make every program a filter.

## Ancient UNIX vs Research UNIX vs Commercial UNIX

## Legacy
### Plan 9
Under development from the mid 80s til mid 90s, Plan 9 was the Bell Labs successor
to UNIX, meant to clean up many of UNIX's loose threads, such as "everything is
a file" and inter process communication.

### Inferno
Under computing threat from Sun, with the Java processor, Java OS, Java Virtual
Machine (Write Once Run Anywhere), and Java language, Bell Labs discontinued Plan 9
to start on a competing computing platform. Similar to Plan 9 but with two noteable
features: a virtual machine called Dis, and a new programming language called
Limbo.

### Minix
Microkernel. Slow to adapt new features. Meant as a teaching system. Strictly
implements POSIX.

### Linux
Monolithic kernel. Fast to adapt new features.

## References
- https://www.youtube.com/watch?v=XvDZLjaCJuw
