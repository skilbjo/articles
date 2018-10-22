## docker: what's under the hood?

Docker has taken its place in developers' workflow. Many articles exist on what
Docker is and how to start using it. I am to explain here the details of how
the technology works.

### docker features
One of the most important features in a Docker container versus a virtual
machine is Docker's instant startup time. Within miliseconds, the Docker
container and application can be started; as opposed to waiting for a virtual
machine to boot. Docker does this by not actually using a virtual machine to
simulate a virtual machine. Docker piggybacks off of features in the Linux
kernel, some first developed by Google, to perfom its magic. Because of this
reliance on the Linux kernel, it is important to note that Docker *only* runs
on Linux. For instance, if you develop on an Apple computer (Darwin kernel -
based on BSD, not Linux), you need to install and run a lightweight Linux
virtual machine before being able to use Docker.

### docker virtualization vs other virtualization strategies
Let's take a step back. Operating systems are software that run on hardware.
An operating system was orignally called a "supervisor", which is a fantastic
analogy what the operating system is doing. Think of the operating system as a
referee or parent, watching over the other programs. The operating system lets other
programs run, and coordinates the scheduling and execution of those programs.
Remember, back in the old days, computers could only do one thing at a time.
Actually, that's still true (a single processor can only do one thing at a time
- let's save pipelining, multiple cores, and hyperthreading for another
article), and it's the operating system that is context switching back and
forth very quickly that gives the appearance of multiple programs running at
the same time.

<img src='../lib/docker/kernel.png' width='800' />

#### hardware virtualization: QEMU
QEMU (quick emulator) is software that performs hardware virtualization. QEMU
emulates CPUs through dynamic binary translation and provides device models;
letting it run on unmodified host operating systems. Downsides: it's very slow,
instructions need to be mapped from one instruction set to another (translating
the x86 `JCXZ` [Jump if CX register is zero] would need to be mapped into many
different ARM instructions).

#### paravirtualization: xen hypervisor

#### software virtualization / Virtual Machines: VirtualBox / VMWare

#### other operating system level virtualization: FreeBSD jails

### technology
Once you are familiar with the underlying technologies, Docker seems less
mystical.  In fact, Docker only works with the Linux kernel, it's not really a
virtual machine as much as it is simulating various Linux distributions,
environments or installs. (Did you know that, if you wanted to, you could take
an Ubuntu distribution and install/remove software until it looks and feels
like another distribution, such as Gentoo? It's all just using the Linux kernel
- various distros are just different userland / package managers on top of the
Linux kernel!).

#### cgropus & namespaces
The backbone of the Docker technology are cgroups and kernel namespaces. With
cgroups, the Linux operating system can easily manage and monitor resource
allocation for a given process. With namespaces, you can control the maximum
amount of resources a prcoess gets. This lets the Docker engine only give out
50% of the computer's memory, 50% of processors, 50% of network, etc. to a
running Docker container.

Docker containers also have network isolation, geting separate virtual
interfaces and IP addressing between containers.

Docker containers also have resource isolation, like CPU and memory limits.
This is provided to each Docker container using `cgroups`, a feature already
provided in the Linux kernel.

#### union file system
Docker uses union filesystem to create and layer Docker images. This means all
images are done to a base image, and then actions added to that base image (ie:
`RUN apt install curl` to create a new image. Under the hood, it allows files
and directories of seperate file systems (known as branches), to be
transparently overlaid, forming a single coherent file system. When branch
contents have the same directory, the contents are merged and seen together in
a single merged directory, and if branches contain the same file, a branch
priority is specified, with the priority branch's file succeeding.

Docker, when building an image, will save these branches in its cache. A
benefit is, if Docker detects the command to create a branch (or layer) already
exist in its cache, it will re-use the branch and not run the command (a cache
hit). This is known as docker layer caching.

#### libcontainer
The C library that provides the underlying Docker functionality is called
libcontainer.  For each execution, each container gets its own root filesystem
(`/tmp/docker/[uuid]/`), and the underlying host does not let the process leave
that directory (called a `chroot jail` - a quick rundown: one of the variables
that a process gets when started by the operating system is current working
directory - a `chroot jail` is where the operating system prevents that started
process from accessing a parent directory, such as `../`).

- how it works at execution (docker image creates tmp directory, runs process there)

### faking it: udocker
One of the drawbacks of Docker is that the Docker engine requires root privileges
to run containers. However, an open source project, `udocker`, provides the same
basic functionality the Docker engine does without root privileges.

It works by creating a `chroot`-like environment over the extracted container, and
using various implementation strategies, to mimic chroot execution with just
user-level privileges.

- https://github.com/indigo-dc/udocker

### tools
There are several tools to using docker. The Docker client is `docker`, which
most of your commands will be run using. There is also `docker-machine`, which
is a wrapper on OS X to help set up the lightweight Linux virtual machine to
execute to `docker` commands inside of.

A Docker image is an artifact. It is created by running the `docker build`
command on a given Dockerfile (the script used to create a docker image). A
Docker container is a run-time instance of a Docker file. An analogy might be:
Docker image is to a `.jar` file as a Docker container is to a running process
of that `.jar`.

Artifacts (Docker images) can be stored on private or public repositories
called registries. Some common registries are Docker Hub, quay.io, and AWS ECR.

### the future: unikernels

### sources
- <https://en.wikipedia.org/wiki/Docker_(software)>
- <https://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-virtual-machine>
- <https://github.com/skilbjo/articles/blob/master/talks/free_bsd_jails.md>
- <https://docs.docker.com/engine/docker-overview/#docker-engine>
- <https://devopscube.com/what-is-docker/>
- <https://github.com/moby/moby> (Docker source code)
- <https://books.google.com/books?id=4xQKBAAAQBAJ>
- <https://github.com/docker/engine/blob/master/daemon/daemon.go>
- <https://wxdublin.gitbooks.io/docker-code-walk/content/daemon.html>
- <https://www.itworld.com/article/2698646/virtualization/containers-bring-a-skinny-new-world-of-virtualization-to-linux.html>

### images
<img src='../lib/docker/fakechroot.png' width='800' />
<img src='../lib/docker/unikernel.png' width='800' />
<img src='../lib/docker/docker-engine.png' width='800' />
<img src='../lib/docker/docker-linux-interface.png' width='800' />
<img src='../lib/docker/linux-containers.png' width='800' />
