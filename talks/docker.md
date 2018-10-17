## docker: what's under the hood?

### docker features
- instant start up time
- piggybacks off of features in Linux kernel to perform its magic
- only runs on Linux. If using other platforms, need to install a Linux virtual machine first, and using that to run Docker

### docker virtualization vs other virtualization strategies
- hardware virtualization: QEMU
- paravirtualization: xen hypervisor
- software virtualization / Virtual Machines: VirtualBox / VMWare
- other operating system level virtualization: FreeBSD jails

### technology
- Linux kernel
- LXC containers
- cgroups, kernel namespaces
- union file system
- docker layer caching
- libcontainer
- how it works at execution (docker image creates tmp directory, runs process there)

### faking it: udocker
- https://github.com/indigo-dc/udocker

### tools
- docker-machine / docker engine
- docker client
- docker image vs docker container
- docker registry

### the future: unikernels

### sources
- <https://en.wikipedia.org/wiki/Docker_(software)>
- <https://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-virtual-machine>
- <https://github.com/skilbjo/articles/blob/master/talks/free_bsd_jails.md>
- <https://docs.docker.com/engine/docker-overview/#docker-engine>
- <https://devopscube.com/what-is-docker/>

### images
<img src='../lib/docker/docker-engine.png' width='800' />
<img src='../lib/docker/docker-linux-interface.png' width='800' />
<img src='../lib/docker/linux-containers.png' width='800' />
