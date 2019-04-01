Bash

Why?
Get to know your shell, which is the command interpreter for many variants of
Unix. This means, Linux[]; \*BSD, and mac OS all use bash as the default login command
interpreter.

History / sh vs bash
Unix / Ken Thompson / Dennis Ritchie
The Unix operating system was first written in 1969 by Ken Thompson and Dennis
Ritchie.[] \(Ken Thompson and Dennis Ritchie are the authors of both Unix and C
language; however, Thompson was more of the Unix developer and Ritchie more the
C language developer\( Thompson, in August 1969, when his wife was on vacation
with his newborn son, wrote the first version of Unix...

> I allocated a week each to the operating system, the shell, the editor and the
> assembler [he told me]... and during the month she was gone, it was totally
> rewritten in a form that looked like an operating system, with tools that were
> sort of known, you know, assembler, editor, and shell -- if not maintaining
> itself, right on the verge of maintaining itself
-- Ken Thompson, describing the origin story of Unix

This original Unix shell is called the Thompson shell. Shell redirection was an
early feature of the shell, and was present for the Version 1 Unix, released on
November 3, 1971. Additionally, some tools in this first release included:
- cal
- cat
- chdir (later shortened to cd)
- chmod / chown
- cp
- date
- df
- du
- ed (the ancestor of vi/vim: the ancestry is: ed (editor)
                                              -> em (editor for "mortals")
                                              -> en
                                              -> ex ("extended" en)
                                              -> vi (visual)
                                              -> vim (vi "improved") )
- ls
- mkdir
- su

Pipes were added in the Unix Version 3, released February 1973.

The Thompson shell was intentially minimalistic, was inadequate for programming
tasks (such as flow control statements such as `if`). In Version 7 Unix
(released 1979), the Thompson shell was replaced by the Bourne shell.

Steven Bourne
Richard Stallman
Current program, written in C

What you need to know (main body of the article)
I assume you are already know your way around bash, so I will skip much of the basic content. However, if you want to brush up, [find good link] is a good place to start

Friendly interactive behavior
Tab once, don’t ring bell on tab
Bash-completion / autocomplete,

Exit codes
A bash script will by default not exit if an error from a program has been
thrown (not the same as a syntax error); it will continue on interpreting. This
is not normally the case with most other programming languages. This can lead
to unexpected behavior in bash scripts. A sane default I use that the top of my
scripts (in addition to the shebang) is:

```
#!/usr/bin/env bash
set -euo pipefail
```

`set -e` (from `man bash`) is: exit immediately if a pipeline (which man
consist of a single shell command... exits with a non-zero status.

`set -u` (from `man bash`) Treat unset variables as an error when performing
parameter expansion.

`set -o pipefail` (from `man bash`) The return value ... is the value of the
last command to exit with a non-zero status, or zero if all commands in the
pipeline exit succesfully.

Bash, and many other programming languages have the concept of exit codes. The
exit code of the prior command is stored in the $? variable. Exit codes 1 - 2,
126 - 165, and 255 all have special meanings, and should only be called if the
command warrants that exit code. Any exit code can be thrown however, as demonstrated by the following:

```
@mbp2:~ $ /usr/local/bin/bash -c 'exit 0'; echo $?
0
@mbp2:~ $ /usr/local/bin/bash -c 'exit 1'; echo $?
1
@mbp2:~ $ /usr/local/bin/bash -c 'exit 12'; echo $?
12
```

Variables



Style guide
- indentation: 2 spaces
- no tabs
- variables, functions, and file names all have descriptive names, unless using
  standard convention (using i in for loop, i being the incrementor)
- use `"$(command substitution)"` instead of `\`backticks`\`
- variables with no variable access in single quotes, variables with variable
  expansion with double quotes. ie `base_url='https://www.codementor.io'` and
  `full_url="${url}/${endpoint}?${query_parameters}"`

Named arguments

```bash
tmp_dir="$(mkdir -p /tmp/email && echo '/tmp/email')"
report=''
distro_list=''
html=''
date_override=''
body_override=''

usage(){
  echo "Usage: email: ${0} [--report <file_path>] [--distro-list <'distro@list.com'>]" 1>&2
  echo "                   [--html] [--date-override <date>] [--body-override <body>]" 1>&2
  echo "       Do not use --html and body override in the same call.                 " 1>&2
  exit 1
}
while [[ $# -gt 0 ]]; do
  case "$1" in
    -r|--report)        report="$2";        shift ;;
    -l|--distro-list)   distro_list="$2";   shift ;;
    -h|--html)          html='y' ;;
    -d|--date-override) date_override="$2"; shift;;
    -b|--body-override) body_override="$2"; shift;;
    *) break ;;
  esac
  shift
done
if [[ -z $report ]] || [[ -z $distro_list ]]; then usage; fi
if [[ ! -z $html ]] && [[ ! -z $body_override ]]; then usage; fi

email(){
  local report="$1"
  local distro_list="$2"
  local html="$3"
  local date_override="$4"
  local body_override="$5"

  if [[ $(whoami) == 'root' ]]; then            # docker (k8s, odroid, pi)
    curl_email "$report" "$distro_list" "$html" "$date_override" "$body_override"
  elif [[ $(whoami) == 'sbx_'* ]]; then         # AWS Lambda
    curl_email "$report" "$distro_list" "$html" "$date_override" "$body_override"
  elif [[ $(whoami) == 'skilbjo' ]]; then       # mac OS
    curl_email "$report" "$distro_list" "$html" "$date_override" "$body_override"
    #mutt_email "$report" "$distro_list" "$html" "$date_override" "$body_override"
  fi
}

email "$report" "$distro_list" "$html" "$date_override" "$body_override"
```


Sourcing files

Aliases (you can inline a function!)

Dotfiles + Github / Dropbox
For syncing your dotfiles across many devices (work computer, home computer, ssh server, demo raspberry pis, girlfriend’s computer, etc)
Execute Commands / Run scripts / Commands

Some great / handy programs
- man: man - format and display the on-line manual pages.
- htop: interactive process viewer / a more modern `top` command
- tree: visual version of `ls` command
- ack: quickly search for file contents of many files
- vim: vim - Vi IMproved, a programmer's text editor
- tmux: tmux -- terminal multiplexer
- bc: evaluate simple mathmatical equations. ie: `sleep "$(echo '60 * 5' | bc)" # sleep for 5 min`
- gzip: compression/decompression tool. compress: `gzip -9 [file]`. decompress: `gzip -d [file].gz`
- watch: run the same command repeatedly in an ncurses window. `watch -n1 'docker ps | grep 'my-container'`



Other shells
- zsh: z shell
- ksh: korn shell
- tcsh: c shell
- fish: a modern shell

[]Linux: Linux is custom so distribution maintainers may craft special versions of Linux that may not use Bash. An example is Alpine Linux, the default distribution meant for Docker containers, which uses almquist shell (a lightweight clone of the Bourne Shell)
[] Linux is also referred to as GNU/Linux
[]Unix itself was spun from a project, Multics [Multiplexed Information and Computing Service],
which itself was spun from a project at MIT called CTSS [Compatible Time-Sharing System] - an
incredible video from 1963 is on youtube describing the reason why an operating system is important
and how it works: https://www.youtube.com/watch?v=Q07PhW5sCEk

References
https://www.tldp.org/LDP/abs/html/exitcodes.html
https://en.wikipedia.org/wiki/Ancient_UNIX
https://en.wikipedia.org/wiki/Research_Unix
http://www.groklaw.net/article.php?story=20050414215646742
https://en.wikipedia.org/wiki/Bourne_shell
https://en.wikipedia.org/wiki/Bash_(Unix_shell)
https://en.wikipedia.org/wiki/Almquist_shell
https://www.reddit.com/r/copypasta/comments/63oudw/gnu_linux/
Source code: http://git.savannah.gnu.org/cgit/bash.git/ (github clone: https://github.com/bminor/bash)
