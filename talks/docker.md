## docker: what's under the hood?

Docker has taken its place in developers' workflow. Many articles exist on what
Docker is and how to start using it. I am to explain here the details of how
the technology is working.

### docker features
One of the most important features in a Docker container versus a virtual
machine is Docker's instant startup time. Within miliseconds, the Docker
container and application can be started; as opposed to waiting for a virtual
machine to boot.  Docker piggybacks off of features in the Linux kernel, some
first developed by Google, to perfom its magic. Because of this reliance on the
Linux kernel, it is important to note that Docker *only* runs on Linux. For
instance, if you develop on an Apple computer (Darwin kernel - based on BSD,
not Linux), you need to install and run a lightweight Linux virtual machine
before being able to use Docker.

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
- Linux kernel
- cgroups, kernel namespaces
- union file system
- docker layer caching
- libcontainer

The C library that provides the underlying Docker functionality is called
libcontainer, (which ships with Linux?).  For each execution, each container
gets its own root filesystem (`/tmp/docker/[uuid]/`), and the underlying host
does not let the process leave that directory (called a `chroot jail`).

Docker containers also have network isolation, geting separate virtual
interfaces and IP addressing between containers.

Docker containers also have resource isolation, like CPU and memory limits.
This is provided to each Docker container using `cgroups`, a feature already
provided in the Linux kernel.

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

### images
<img src='../lib/docker/fakechroot.png' width='800' />
<img src='../lib/docker/unikernel.png' width='800' />
<img src='../lib/docker/docker-engine.png' width='800' />
<img src='../lib/docker/docker-linux-interface.png' width='800' />
<img src='../lib/docker/linux-containers.png' width='800' />
