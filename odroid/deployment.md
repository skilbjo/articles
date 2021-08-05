# home datacenter: code deployment w/ arch linux + odroid
by skilbjo

## Background
DevOps is hard. Large softwares (mesos, kubernetes), teams (most tech organizations have a
devops team), and companies (CoreOS) exist to support developers getting their apps out
the door and running. However, what's a good solution for a lone developer, or a small home
network? How can we use code as infrastructure?

I use a macOS laptop for development, but for long-lived applications or apps to run at night,
I need a remote always-on environment, as my laptop will be offline or on the train with me
commuting. It makes sense to not use the development machine as an evironment for deployment.

The odroid XU-4 is an ideal computer for a remote deployment environment, as it's cheap,
flexible, and has great specs, and can run linux.

The idea of this article is to explain how to keep deployment code in your project
repository, and automate deployments and execution.

## Arch Linux
Arch Linux is a free and open-source linux distribution first released in 2002. It
focuses on elegance, code correctness, minimalism and simplicity, and expects the user
to make some effort to understand the system's operation. Arch Linux notably uses
a rolling release model, so all that is needed to obtain the latest system software
is a regular system update.

Arch Linux can be difficult to pick up, as it uses different tools than a debian
distro. The package manager is called via `pacman` rather than `apt-get`. There is a
popular add-on package manager called `yaourt`. Many common tools or services aren't
installed by default.

Arch Linux is mainly for x86 processors, but a project called Arch Linux ARM has a ARM
distribution of Arch Linux for ARMv7 and ARMv8 AArch64 architectures. Hardkernel (manufacturer
of odroid) is actually a sponsor of the Arch Linux ARM project.

## Network Setup
You'll want to give your device a reserved DHCP LAN IP address, and ideally a hostname
that gets propogated over your network through your router's DNS server. That way, on
our development/local environment, we can use a hostname to always resolve to the
remote/deployment environment.

For example, on my network I reserve `192.168.2.49` to my odroid's MAC address. I also
set up a DNS entry that maps that IP address to "odroid". Using a custom router firmware
like tomato USB or DD-WRT makes this extremely easy (as those firmwares turn your
router into a small linux computer with a slick GUI webapp), but implementing that is
outside the scope of this article.

If going across subnets, make sure to port forward an external port that maps to the
ssh port of the odroid device, as git runs over ssh.

## Project Setup
Logically, you'll want to standardize your deploy workflow. That will make working
across projects extremely easy, and remove a lot of the mental context switching
you use when working across projects.

We'll create a folder for housing all of our files related to deployments. Place any
executable files in `deploy/bin`, and any cron files in `deploy/tasks` (more on
that later).

[in local environment, in project directory]

    mkdir -p deploy/bin
    mkdir -p deploy/tasks
    cd deploy/bin && touch run-job && chmod u+x run-job && cd -
    cd deploy/tasks && touch crontab

You could also standardize where you place your source code. That way, it becomes very
easy for another party how your project is organized, and what is source code and what is not.

[in local environment, in project directory]

    mkdir src
    cd src && (place soure code here, ie python: core.py, clojure: core.clj, nodejs: app.js)

## Abstracting the entrypoint of your app into a shell script
Starting an app can be confusing with all the various commands to run in various languages.
For example, `java -jar [my-jar].jar`, or python `python my-app.py`, and your app also might
require various arguments. This should all be simplified in a single file and abstracted in a
file, `deploy/bin/run-job`:

[in local environment, project directory]

    #!/bin/sh
    set -e

    CMD="src/duck"

    exec $CMD $@

## Creating the cron file
Arch Linux doesn't come with a cron daemon or client by default. Install it with
`sudo pacman -Syu cronie`. Using cron, you can run commands at specified time
intervals using special cron syntax. This is normally stored in a user's crontab file,
which you access with `crontab -e`. However, this is too manual, and we want to use code
as infrastructure. Cron also has some helpful subdirectories in `/etc/cron.*`,
like `/etc/cron.daily`, `/etc/cron.hourly`, etc, and if we place files here,
they will be run at those intervals specified.

Now, review our file from `deploy/tasks/crontab` that we'll place in `/etc/cron.d`
(this is done automatically with our `post-receive` script):

    ### variables ##
      SHELL=/bin/bash
      PATH=:/bin:/usr/bin:/usr/local/bin:/usr/sbin:/usr/local/sbin
      MAILTO=[your-email-address]@gmail.com
      cmd="deploy/bin/run-job"
      app_dir="/home/skilbjo/deploy/app/duckdns"

    ## jobs ##
    5 * * * * skilbjo cd "$app_dir" ; $cmd >/dev/null

Here is an overview of a simple project structure (the only project executable is a
single shell script in `src/`:

    @mbp:duckdns $ tree
    .
    ├── README.md
    ├── deploy
    │   ├── bin
    │   │   ├── post-receive
    │   │   └── run-job
    │   └── tasks
    │       └── crontab
    └── src
        └── duck

    4 directories, 5 files

## Git
### Local environment
First we'll want to add a remote url in our project on our local environment.

[in local environment, in project directory]

    git remote add odroid ssh://odroid/~/deploy/git/duckdns.git

Note depending on your network topology, this url might need to be adjusted.
If you can't assign hostnames, then the git url would look like:
`ssh://192.168.2.49/~/deploy/git/duckdns.git` where `192.168.2.49` is your device's LAN IP address.

If you have a different user on your odroid environment than your development environment then the
url would look like: `ssh://skilbjo@odroid/~/deploy/git/duckdns.git` where `skilbjo` is your
username.

If your remote server is on a different subnet and you are portfowarding, your url would
look like: `ssh://192.168.1.2:2222/~/deploy/git/duckdns.git` where `2222` is your external port.

### Remote environment (odroid)
In the home directory of your remote environment, create a folder called `~/deploy` with
two subfolders, `~/deploy/app` and `~/deploy/git`.

The `~/deploy/git` subdirectories will be the endpoints of our pushes, and with a hook,
they will then execute some deployment commands in the `~/deploy/app` subdirectories.

[in remote environment, in home directory]
    mkdir -p ~/deploy/app
    mkdir -p ~/deploy/git
    mkdir -p ~/deploy/git/duckdns.git
    mkdir -p ~/deploy/app/duckdns

Now, in `~/deploy/git/duckdns.git/hooks`, create an executable file called `post-receive`,
which will be trigger with every push to the endpoint.

[in remote environment, in home directory]

    cd ~/deploy/app/git/duckdns.git/hooks
    touch post-receive && chmod u+x post-receive
    vim post-receive

Fill the executable with the following:

[in remote environment, in `~/deploy/git/duckdns.git/hooks` directory]

    #!/usr/bin/env bash
    set -eou pipefail

    user=$(whoami)
    dir="/home/${user}/deploy/app"
    app=$(basename $(pwd) | sed -e 's/.git//')
    deploy_dir="$dir/$app"
    cron_dir="/etc/cron.d"

    GIT_WORK_TREE="$deploy_dir" git checkout -f

    cd "$deploy_dir"

    ## build steps here ##
    case "$user" in
      (skilbjo) sudo cp deploy/tasks/crontab "$cron_dir/$app" ;;
    esac

    ## you can also do project-specific build steps in this section, like install
    ## dependencies, ## (ie npm install), compile source code (ie lein uberjar),
    ## as well as if a long-lived app, run commands as well (ie java -jar my_jar.jar)

    echo "all done"

    exit 0

## Deployment
Now we're ready. Our local environment is set up to hit the deployment server's endpoint,
our remote environment is set up to receive the notification and check out the source code,
run any build steps, and place a job in the system's cron directory for launching.

Deploy with:

[in local environment, in project directory]

    git push odroid

Additionally, to see how this has been implemented in a sample project, see:
https://github.com/skilbjo/duckdns

## Next steps
Some features that can be added to the flow above are mutliple environments (either with
multiple odroids, or with a single odroid treating it as a utility server. This can be done
with subdirectories under `~/deploy` such as `~/deploy/staging/app/my_app` or
`~/deploy/production/app/my_app`.

Additionally, you could add a continuous integration service (like CircleCI) that would
run a test suite from every push to github, and if successful, build a docker image.
You could then have have a file in the remote environment that would checkout an image
from a docker repository and run that image at a specified interval. This is what many of
the distributed DevOps softwares do (mesos, kubernetes), but much more feature rich
than bare-bones bash and linux :)

## References
- https://en.wikipedia.org/wiki/Arch_Linux
- https://archlinuxarm.org
- http://blog.fxndev.com/raspberry-pi-nodejs-project-setup/ (this was the article I read
three years ago that inspired the idea of my article)
