# free-bsd jails

## BSD
<img src='../lib/unix-history-simple.png' width=700 />

<img src='../lib/unix-timeline.png' width=700 />

See more in my talk about [Unix](./unix.md).

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

<img src='../lib/os_wars.png' width=700 />

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

<img src='../lib/freebsd.jpg' width=700 />

Why virtualize? Primarily, so the app will run, damnit. "Write once, run anywhere"
concept. As long as the developer packages the app initially in a virtualization
software, the contract is the app will work.


## jails demo

```bash
Last login: Mon Feb 12 01:03:32 2018 from 10.8.0.10
FreeBSD 11.1-RELEASE-p1 (GENERIC) #0: Wed Aug  9 11:55:48 UTC 2017

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.

 * keychain 2.8.3 ~ http://www.funtoo.org
 * Found existing ssh-agent: 924
 * Known ssh key: /home/skilbjo/.ssh/id_rsa

By pressing "Scroll Lock" you can use the arrow keys to scroll backward
through the console output.  Press "Scroll Lock" again to turn it off.
@udoo:~ $ which ezjail
@udoo:~ $ cd /usr/ports/sysutils/ezjail/
@udoo:ezjail $ make install clean
===>   ezjail-3.4.2 depends on file: /usr/local/sbin/pkg - found
=> ezjail-3.4.2.tar.bz2 doesn't seem to exist in /usr/ports/distfiles/.
=> Attempting to fetch http://erdgeist.org/arts/software/ezjail/ezjail-3.4.2.tar.bz2
ezjail-3.4.2.tar.bz2                          100% of   37 kB  110 kBps 00m01s
===> Fetching all distfiles required by ezjail-3.4.2 for building
===>  Extracting for ezjail-3.4.2
=> SHA256 Checksum OK for ezjail-3.4.2.tar.bz2.
===>  Patching for ezjail-3.4.2
===>  Configuring for ezjail-3.4.2
===>  Building for ezjail-3.4.2
===>  Staging for ezjail-3.4.2
===>   Generating temporary packing list
mkdir -p /usr/ports/sysutils/ezjail/work/stage/usr/local/etc/ezjail/ /usr/ports/sysutils/ezjail/work/stage/usr/local/man/man5/ /usr/ports/sysutils/ezjail/work/stage/usr/local/man/man7 /usr/ports/sysutils/ezjail/work/stage/usr/local/man/man8 /usr/ports/sysutils/ezjail/work/stage/usr/local/etc/rc.d/ /usr/ports/sysutils/ezjail/work/stage/usr/local/bin/ /usr/ports/sysutils/ezjail/work/stage/usr/local/share/examples/ezjail /usr/ports/sysutils/ezjail/work/stage/usr/local/share/zsh/site-functions
cp -R examples/example /usr/ports/sysutils/ezjail/work/stage/usr/local/share/examples/ezjail/
cp -R examples/nullmailer-example /usr/ports/sysutils/ezjail/work/stage/usr/local/share/examples/ezjail/
cp -R share/zsh/site-functions/ /usr/ports/sysutils/ezjail/work/stage/usr/local/share/zsh/site-functions/
sed s:EZJAIL_PREFIX:/usr/local: ezjail.conf.sample > /usr/ports/sysutils/ezjail/work/stage/usr/local/etc/ezjail.conf.sample
sed s:EZJAIL_PREFIX:/usr/local: ezjail.sh > /usr/ports/sysutils/ezjail/work/stage/usr/local/etc/rc.d/ezjail
sed s:EZJAIL_PREFIX:/usr/local: ezjail-admin > /usr/ports/sysutils/ezjail/work/stage/usr/local/bin/ezjail-admin
sed s:EZJAIL_PREFIX:/usr/local: man8/ezjail-admin.8 > /usr/ports/sysutils/ezjail/work/stage/usr/local/man/man8/ezjail-admin.8
sed s:EZJAIL_PREFIX:/usr/local: man5/ezjail.conf.5 > /usr/ports/sysutils/ezjail/work/stage/usr/local/man/man5/ezjail.conf.5
sed s:EZJAIL_PREFIX:/usr/local: man7/ezjail.7 > /usr/ports/sysutils/ezjail/work/stage/usr/local/man/man7/ezjail.7
chmod 755 /usr/ports/sysutils/ezjail/work/stage/usr/local/etc/rc.d/ezjail /usr/ports/sysutils/ezjail/work/stage/usr/local/bin/ezjail-admin
chmod 0440 /usr/ports/sysutils/ezjail/work/stage/usr/local/share/examples/ezjail/example/usr/local/etc/sudoers
====> Compressing man pages (compress-man)
===>  Installing for ezjail-3.4.2
===>  Checking if ezjail already installed
===>   Registering installation for ezjail-3.4.2
Installing ezjail-3.4.2...
===>  Cleaning for ezjail-3.4.2

@udoo:~ $ docker-machine create --driver "virtualbox" --virtualbox-hostonly-cidr "192.168.99.1/24" default
@udoo:~ $ docker-machine stop default
@udoo:~ $ docker-machine start default
@udoo:~ $ docker-machine regenerate-certs default
@udoo:ezjail $ which ezjail-admin
/usr/local/bin/ezjail-admin
@udoo:~ $ sudo ezjail-admin install -sp
@udoo:~ $ ifconfig
re0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=8209b<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING,VLAN_HWCSUM,WOL_MAGIC,LINKSTATE>
        ether 00:00:00:00:00:00
        hwaddr 00:00:00:00:00:00
        inet 192.168.2.49 netmask 0xffffff00 broadcast 192.168.2.255
        nd6 options=29<PERFORMNUD,IFDISABLED,AUTO_LINKLOCAL>
        media: Ethernet autoselect (1000baseT <full-duplex>)
        status: active
vboxnet0: flags=8943<UP,BROADCAST,RUNNING,PROMISC,SIMPLEX,MULTICAST> metric 0 mtu 1500
        ether 0a:00:27:00:00:00
        hwaddr 0a:00:27:00:00:00
        inet 192.168.99.1 netmask 0xffffff00 broadcast 192.168.99.255
        nd6 options=29<PERFORMNUD,IFDISABLED,AUTO_LINKLOCAL>
        media: Ethernet autoselect
        status: active
@udoo:~ $ sudo service netif cloneup
@udoo:~ $ sudo cd /usr/src; sudo make buildworld
@udoo:~ $ sudo ezjail-admin update -i -p
@udoo:~ $ sudo echo 'ezjail_enable="YES"' >> /etc/rc.conf
@udoo:~ $ sudo ezjail-admin create foo 192.168.100.50
# move files into the jail
@udoo:~ $ sudo cp /etc/resolv.conf /usr/jails/foo/etc/
# install software packages into the jail
@udoo:~ $ sudo pkg -j foo install vim curl
@udoo:~ $ sudo service ezjail start
@udoo:~ $ jls
   JID  IP Address      Hostname                      Path
     1  192.168.100.50  foo                           /usr/jails/foo

@udoo:jails $ cd /usr/jails/foo/
@udoo:foo $ ls
COPYRIGHT       bin             dev             lib             media           net             rescue          sbin            tmp             var
basejail        boot            etc             libexec         mnt             proc            root            sys             usr

@udoo:~ $ sudo ezjail-admin console foo
Last login: Tue Feb 20 02:50:37 on pts/7
FreeBSD 11.1-RELEASE-p1 (GENERIC) #0: Wed Aug  9 11:55:48 UTC 2017

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.
root@foo:~ # cat /etc/resolv.conf
# Generated by resolvconf
nameserver 192.168.2.1

root@foo:~ #
@udoo:~ $ sudo ezjail-admin stop foo
@udoo:~ $ sudo ezjail-admin archive foo
```

## Other virtualization technologies
### Full virtualization
#### qemu
QEMU (quick emulator) is software that performs hardware virtualization. QEMU
emulates CPUs through dynamic binary translation and provides device models;
letting it run on unmodified host operating systems.

### Hardware-assisted virtualization
Operating systems run in privileged mode to access drivers. Linux kernel has
special instructions that are meant to run in that privileged mode. If run as
an application, the instruction will error.  The idea is to trap execution
calls and send them to the virtualization system. This is very slow - goal is
have most calls run natively, and only trap/VMM a small subset of calls.

Hardware-assisted virtualization first appeared in an IMB System/370 in 1972!
If you're on an Intel / AMD x86 CPU purchased in the past 10 years, you have
it! Intel added this technology in 2005.

<img src='../lib/privileged_instruction.png' width='800' />

<img src='../lib/trap.png' width='800' />

<img src='../lib/emulate.png' width='800' />

### Paravirtualization
Paravirtualization is a technique with where you try to run software natively
as much as possible; reduce overhead.

#### xen hypervisor
Microkernel design, interacts with guests via dom0 over a "hypercall" (vs
system call) using an ABI (application binary interface) (vs API).

### Virtual Machines
#### VirtualBox
Hypervisor for x86 computers, developed originally by Innotek; acquired by Sun
Microsystems; acquired by Oracle.

#### Vagrant
A software that creates software development environments for VirtualBox; ie
packages, port forwarding, VM distro. Sits on top of VirtualBox, acts as a
wrapper, automates the configuration of virtual environments described by Chef
and Puppet.

Commonly uses other technologies in the ecosystem such
as Chef, Ansible, and Puppet.

Puppet: declarative language to describe system configuration.

```ruby
user { 'john':
  ensure => present,
  uid    => '1000',
  shell  => '/bin/bash',
  home   => '/var/tmp'
}
```

Chef: Ruby DSL for writing system configuration.

```ruby
service 'apache' do
  action [ :enable, :start ]
  retries 3
end
```

Salt: Python-based configuration management software.

<img src='../lib/salt.png' width='600' />

Ansible: software that automates provisioning, configuration management.

```
- hosts: webserver

  vars_files:
  - group_vars/all

  tasks:
  - name: Launch webserver instances
    ec2: >
     access_key="{{ ec2_access_key }}"
     secret_key="{{ ec2_secret_key }}"
     keypair="{{ ec2_keypair }}"
     group="{{ ec2_security_group }}"
     type="{{ ec2_instance_type }}"
     image="{{ ec2_image }}"
     region="{{ ec2_region }}"
     instance_tags="{'ansible_group':'{{ ec2_tag_webservers }}', 'type':'{{ ec2_instance_type }}', 'group':'{{ ec2_security_group }}', 'Name':'demo_''{{ tower_user_name }}'}"
     count="{{ ec2_instance_count }}"
    register: ec2
```

### Operating-system-level
#### LXC / Linux containers
Instant start up time. Similar to jails, no operating system to boot up; just
resource isolation.

#### Docker
Instant start up time. Uses features in the Linux kernel (libcontainer,
cgroups, and kernel namespaces) {Note: this is also why, in order to use Docker
on macOS, you need to install a Linux VM such as boot2docker or others}.

#### FreeBSD capscium

<img src='../lib/bsd_wars.png' width=700 />

```
CAPSICUM(4)            FreeBSD Kernel Interfaces Manual            CAPSICUM(4)

NAME
     Capsicum - lightweight OS capability and sandbox framework

SYNOPSIS
     options CAPABILITY_MODE
     options CAPABILITIES

DESCRIPTION
     Capsicum is a lightweight OS capability and sandbox framework
     implementing a hybrid capability system model.  Capsicum can be used for
     application and library compartmentalisation, the decomposition of larger
     bodies of software into isolated (sandboxed) components in order to
     implement security policies and limit the impact of software
     vulnerabilities.

     Capsicum provides two core kernel primitives:

     capability mode
             A process mode, entered by invoking cap_enter(2), in which access
             to global OS namespaces (such as the file system and PID
             namespaces) is restricted; only explicitly delegated rights,
             referenced by memory mappings or file descriptors, may be used.
             Once set, the flag is inherited by future children processes, and
             may not be cleared.

     capabilities
             Limit operations that can be called on file descriptors.  For
             example, a file descriptor returned by open(2) may be refined
             using cap_rights_limit(2) so that only read(2) and write(2) can
             be called, but not fchmod(2).  The complete list of the
             capability rights can be found in the rights(4) manual page.
```

#### OpenBSD pledge()
A syscall that annotates what a program can do; the kernel will kill the
process if the program exceeds its scope.

If a program needs real-only access to a file, the pledge guards are:

```c
if (pledge(“stdio rpath”, NULL) == -1)
  err(1, “pledge”);
```


```
PLEDGE(2)              System Calls Manual                           PLEDGE(2)

NAME
     pledge — restrict system operations

SYNOPSIS
     #include <unistd.h>
     int
     pledge(const char *promises, const char *execpromises);

DESCRIPTION
     The current process is forced into a restricted-service operating mode. A
     few subsets are available, roughly described as computation, memory
     management, read-write operations on file descriptors, opening of files,
     networking. In general, these modes were selected by studying the
     operation of many programs using libc and other such interfaces, and
     setting promises or execpromises.

     Use of pledge() in an application will require at least some study and
     understanding of the interfaces called. Subsequent calls to pledge() can
     reduce the abilities further, but abilities can never be regained.

     A process which attempts a restricted operation is killed with an
     uncatchable SIGABRT, delivering a core file if possible. A process
     currently running with pledge has state ‘p’ in ps(1) output; a process
     that was terminated due to a pledge violation is accounted by lastcomm(1)
     with the ‘P’ flag.

     A promises value of "" restricts the process to the _exit(2) system call.
     This can be used for pure computation operating on memory shared with
     another process.

     Passing NULL to promises or execpromises specifies to not change the
     current value.

     Some system calls, when allowed, have restrictions applied to them:

     pledge() Can only reduce permissions for promises and execpromises.
```

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
@udoo:~ $ uname -a
FreeBSD udoo 11.1-RELEASE-p1 FreeBSD 11.1-RELEASE-p1 #0: Wed Aug  9 11:55:48 UTC 2017     root@amd64-builder.daemonology.net:/usr/obj/usr/src/sys/GENERIC  amd64
@udoo:~ $ docker run -it --rm quay.io/skilbjo/markets-etl:x86 bash
bash-4.3# uname -a
Linux 2a8a3c5bbf21 4.4.115-boot2docker #1 SMP Thu Feb 8 17:36:45 UTC 2018 x86_64 GNU/Linux
bash-4.3# exit
@udoo:~ $ docker-machine ssh default
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
Boot2Docker version 18.02.0-ce, build HEAD : 99245f4 - Thu Feb  8 17:43:39 UTC 2018
Docker version 18.02.0-ce, build fc4de44
docker@default:~$ uname -a
Linux default 4.4.115-boot2docker #1 SMP Thu Feb 8 17:36:45 UTC 2018 x86_64 GNU/Linux
```

## Questions?

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
- https://www.freebsd.org/doc/handbook/jails-ezjail.html
