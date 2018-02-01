# free-bsd jails

## BSD
Berkeley Software Distribution, Berkeley Unix, or BSD, is a Unix-like operating
system, probably the closest surviving relative of original UNIX. It shared the
initial codebase and design with the original AT&T Unix operating system.

The original BSD (386BSD, called Jolix) was forked in 1993 into came derivative
operating systems: FreeBSD and NetBSD. OpenBSD was then forked from NetBSD in 1995,
and DragonFlyBSD forked from FreeBSD in 2003.

Note: Linux and the various distributions (Arch Linux, Alpine Linux, Debian) all
share the same kernel. Kernel land and userland are worked on independently in
releases (ie. Linux 3.4 + Alpine Linux 3.5 & Linux 3.4 & Arch Linux LATEST would
look & feel different to the user even though they share the same kernel), and
the distros rely on the kernel independently of the userland. With BSDs, the
kernel and userland are often in the same version control system; there is no
shared BSD kernel FreeBSD and NetBSD, as they use their own kernel. However,
occassionally developers will release userland operating systems on top of a \*BSD
kernel, such as Debian GNU/kNetBSD on top of the NetBSD kernel, or Orbis OS
{runs the Playstation 4} on top of the FreeBSD kernel.

Note: Apple's iOS and macOS along with the Darwin kernel include a large amount
of code derived from FreeBSD.

<img src='../lib/unix-history-simple.png' width=700>

See more in my talk about [Unix](./unix.md).

## Jails
### Why?
> Since system administration is a difficult task, many tools have been developed
> to make life easier for the administrator. These tools often enhance the way
> systems are installed, configured, and maintained. One of the tools which can
> be used to enhance the security of a FreeBSD system is jails. Jails have been
> available since FreeBSD 4.X and continue to be enhanced in their usefulness,
> performance, reliability, and security.

Jails are userland software to run on top of chroot(2) system call, in the
`#include <unistd.h>` library.

```
CHROOT(2)     FreeBSD System Calls Manual        CHROOT(2)

DESCRIPTION
     The dirname argument is the address of the pathname of a directory, ter-
     minated by an ASCII NUL.  The chroot() system call causes dirname to
     become the root directory, that is, the starting point for path searches
     of pathnames beginning with `/'.

     In order for a directory to become the root directory a process must have
     execute (search) access for that directory.

     It should be noted that chroot() has no effect on the process's current
     directory.

     This call is restricted to the super-user.

     Depending on the setting of the `kern.chroot_allow_open_directories'
     sysctl variable, open filedescriptors which reference directories will
     make the chroot() fail as follows:

     If `kern.chroot_allow_open_directories' is set to zero, chroot() will
     always fail with EPERM if there are any directories open.

     If `kern.chroot_allow_open_directories' is set to one (the default),
     chroot() will fail with EPERM if there are any directories open and the
     process is already subject to the chroot() system call.

     Any other value for `kern.chroot_allow_open_directories' will bypass the
     check for open directories
```

Why virtualize? Primarily, so the app will run, damnit. "Write once, run anywhere"
concept. As long as the developer packages the app initially in a virtualization
software, the contract is the app will work.

## Other virtualization technologies
### Full virtualization
#### qemu

### Hardware-assisted virtualization
The idea here is to trap execution calls and send them to the virtualization system.

Hardware-assisted virtualization first appeared in an IMB System/370 in 1972!
If you're on an Intel / AMD x86 CPU purchased in the past 10 years, you have it!
Intel added this technology in 2005.

### Paravirtualization
Paravirtualization is a technique with a
#### xen hypervisor

### Virtual Machines
#### VirtualBox

#### Vagrant
A software that creates
Commonly uses other technologies in the ecosystem such as Chef, Ansible, and Puppet.

### Operating-system-level
#### LXC containers

#### Docker
Uses features in the Linux kernel (libcontainer, cgroups, and kernel namespaces)
{Note: this is also why, in order to use Docker on macOS, you need to install
a Linux VM such as boot2docker or others}

```
@C02NN3NBG3QT:talks $ uname -a
Darwin C02NN3NBG3QT 16.7.0 Darwin Kernel Version 16.7.0: Thu Jun 15 17:36:27 PDT 2017; root:xnu-3789.70.16~2/RELEASE_X86_64 x86_64

@C02NN3NBG3QT:~ $ docker-machine ssh default
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.12.0, build HEAD : e030bab - Fri Jul 29 00:29:14 UTC 2016
Docker version 1.12.0, build 8eab29e

docker@default:~$ uname -a
Linux default 4.4.16-boot2docker #1 SMP Fri Jul 29 00:13:24 UTC 2016 x86_64 GNU/Linux
docker@default:~$
```

```
@C02NN3NBG3QT:talks $ cowsay Thanks for learning about FreeBSD jails!
 ___________________________________
/ Thanks for learning about FreeBSD \
\ jails!                            /
 -----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

## References
- https://www.freebsd.org/doc/handbook/jails.html
- http://www.bsdnow.tv/tutorials/jails
- https://www.freebsd.org/cgi/man.cgi?query=chroot&sektion=2&manpath=freebsd-release-ports
- https://en.wikipedia.org/wiki/Hardware-assisted_virtualization
- https://github.com/skilbjo/articles/blob/master/talks/unix.md
