title: Basix Wiki

# Introduction

By default, Basix uses the [nitro](https://github.com/leahneukirchen/nitro) init/service manager.

This is supplemented by s6' [s6-log](https://www.skarnet.org/software/s6/s6-log.html) and runits [chpst](https://smarden.org/runit/chpst.8)

All "normal services" are written in the [execline](https://www.skarnet.org/software/execline/) scripting language because these are generally just `exec`s with very little logic.

Core system services that perform more complex operations are not written in execline as that would be far less efficient compared to just spawning a shell.

This wiki page exists to document basic usage of this stack, and you should absolutely consult the pages linked above for in-depth information.

# System Services

Services are kept in `/etc/sv`. Symlinks to said services, when supervision of them is desired, are kept in `/etc/nitro`.

# Usage

Most of your questions are probably answered by nitroctl(1):

```
NITROCTL(1)                 General Commands Manual                NITROCTL(1)

NAME
     nitroctl - control and manage services monitored by nitro

SYNOPSIS
     nitroctl [-t timeout] [-v] command services ...

DESCRIPTION
     nitroctl supports the following commands:

     list             List all services and information about them (default).
     scan, rescan     Rescan the service list from the file system.
     up, u            Request services to be up.
     down, d          Request services to be down.
     start            Request services to be up and wait for it to enter UP.
     stop             Request services to be down and wait for it to enter
                      DOWN.
     restart          Request services to be restarted and wait for it to be
                      UP again.
     fast-start       Request services to be started and wait for it to be in
                      state STARTING.
     fast-stop        Request services to be down and wait for it to be in
                      state SHUTDOWN.
     fast-restart, r  Request services to be restarted and wait for it to be
                      in state RESTART.
     ready            If services are in state STARTING, put them into state
                      UP.
     wait-up          Wait for services to be in state UP.
     wait-down        Wait for services to be in state DOWN or FATAL.
     wait-starting    Wait for services to be in state STARTING or UP or
                      ONESHOT.
     pause, cont, hup, alarm, interrupt, quit, term, kill, 1, 2
                      Send the signal to services.  The first character can be
                      used as a shortcut.
     pidof            Print PIDs of the services.
     check            Exit with status 0 if the services are running (known
                      PID)
     Reboot           Request system reboot.
     Shutdown         Request system shutdown.

     Note that requesting multiple transitions in a single invocation only
     guarantees each service has reached the target state at some point, not
     that all services are in the target state at the same time when nitroctl
     exits.  Requested service state transitions happen concurrently.

     The options are as follows:

     -t timeout       Exit after timeout seconds (as a decimal number) if the
                      target state has not been reached.  By default, the
                      timeout is infinite.

     -v               Verbose mode, print each state transition as it happens.

ENVIRONMENT
     NITRO_SOCK
             The control socket (by default: /var/run/nitro/nitro.sock) where
             nitro(8) listens for commands.

EXIT STATUS
     The nitroctl utility exits 0 on success, and >0 if an error occurs.

SEE ALSO
     nitro(8)

AUTHORS
     Leah Neukirchen <leah@vuxu.org>

Linux 7.2.12_1                 January 23, 2026                 Linux 7.2.12_1
```

## Listing the currently supervised services

When running `nitroctl` with no arguments, a list of currently supervised (symlinked to /etc/nitro) services and their current state is listed. Example:

```
(0) sagittarius ~ > doas nitroctl
doas (kris@sagittarius) password:
UP LOG@acpid (pid 3643) (wstatus 0) 13756s
UP LOG@dhcpcd (pid 3658) (wstatus 0) 13756s
UP LOG@greetd (pid 3640) (wstatus 0) 13756s
UP LOG@local (pid 3656) (wstatus 0) 13756s
UP LOG@mdevd (pid 3303) (wstatus 0) 13760s
UP LOG@mdevd-coldplug (pid 3307) (wstatus 0) 13760s
UP LOG@nftables (pid 3631) (wstatus 0) 13756s
UP LOG@ntpd (pid 3633) (wstatus 0) 13756s
UP LOG@seatd (pid 3637) (wstatus 0) 13756s
UP LOG@smartd (pid 3653) (wstatus 0) 13756s
UP LOG@tty1 (pid 3647) (wstatus 0) 13756s
UP LOG@tty2 (pid 3650) (wstatus 0) 13756s
UP acpid (pid 3644) (wstatus 0) 13756s
UP dhcpcd (pid 3660) (wstatus 0) 13756s
UP getty@tty1 (pid 3648) (wstatus 0) 13756s
UP getty@tty2 (pid 3651) (wstatus 0) 13756s
UP greetd (pid 3699) (wstatus 0) 13754s
ONESHOT local (wstatus 0) 13756s
UP mdevd (pid 3304) (wstatus 0) 13760s
ONESHOT mdevd-coldplug (wstatus 0) 13756s
ONESHOT nftables (wstatus 0) 13756s
UP ntpd (pid 3635) (wstatus 0) 13756s
UP seatd (pid 3639) (wstatus 0) 13756s
UP smartd (pid 3654) (wstatus 0) 13756s
```

## Enabling and starting a service

"Enabling" a service in this context means symlinking it to nitros supervision directory. Without doing this, nitro won't even know the service exists.

To do so, for example:

``ln -s /etc/sv/smartd /etc/nitro``

After doing this, you will need to have nitro rescan for services. You may do this by running `nitroctl rescan`.

## Enabling and starting a parameterized service

Some services may be parameterized, such as ``getty@`` (notice the @).

What this means is that there's one service that takes arguments and may change behavior based on its name, and thus may be used to spawn several services.

To do this, enable the `getty@` service, for example.

You may then symlink *that* to other services, such as `getty@tty1` or `getty@tty2`.

## Disabling and stopping a service

Remove its symlink. For example:

``rm /etc/nitro/smartd``

Run `nitroctl rescan` so nitro will be aware the service is gone.

## Starting or stopping while supervised

Run `nitroctl {start,stop} <service>` to start or stop a service while keeping its supervision symlink.

## Custom services

If a sysadmin wishes to create their own service, they may place it in `/etc/sv`.

A typical service directory may follow this structure, as an incredibly simple oneshot service:

```
(0) sagittarius ~ > tree /etc/sv/nftables/
/etc/sv/nftables/
├── finish
└── setup
```

When a service has a `setup` script but no `run` script, nitro considers it a `oneshot` service.

If you want a long-running service, you should generally have a `run` script here that `exec`s something to replace the caller, whether that be an execline script, shell script, actual binary, whatever.

If you have a service that has dependencies, there is an included `need` wrapper at `/usr/lib/nitro/need` that may be used in a `setup` script.

For example:

```
(0) sagittarius ~ > cat /etc/sv/greetd/setup
#!/bin/execlineb -P
fdmove -c 2 1
/usr/lib/nitro/need seatd
```

Admittedly this script is a slight bit of a hack (what isn't?) and probably isn't strictly necessary, but it improves UX a bit and is likely superior to blind crash looping like you'd do with runit.

Upon creating your custom service, symlink it to `/etc/nitro` and treat it like any other service.

If you need to run something as another user from a service, use [chpst](https://smarden.org/runit/chpst.8).

# Reading logs

By default, you will have a `LOG@` service enabled. This is an all-encompassing default logger service for every other service that doesn't specify its own logging setup.

All services will get their own distinct directory in `/var/log/<service>`. An example of this directory may be:

```
(0) sagittarius ~ > doas ls -al /var/log/greetd
total 40
drwx------   2 root root  151 Oct 10 01:33 .
drwxr-xr-x  18 root root 4096 Oct 10 01:10 ..
-rw-r--r--   1 root root  450 Oct  9 17:53 current
-rw-r--r--   1 root root    0 Sep  7 19:53 lock
-rw-r--r--   1 root root    0 Sep 28 10:55 state
```

These logs are only readable by root by default.

Given s6-log is sane, these are just plaintext logs (cough cough). View the contents however you wish.

# Core system services

There are a handful of services on Basix that are considered "core" services and thus are run by the init system before any of the "normal" services get to run. These are provided by the ``baseinit`` package and are kept in `/usr/lib/nitro/scripts`. If you're curious, you may view these [here](https://github.com/kkrruumm/basix-packages/tree/main/sys/baseinit/files/scripts).

What actually kicks off these scripts would be `/etc/sv/SYS/setup`.

These are written in POSIX shell as they perform relatively complex operations and execline becomes drastically less efficient with more complex scripts.

# I don't want to use nitro!

Sweet! The primary point of Basix is modularity, and will not stand in the way of replacing it with something else on your own system.

Disable the global `services` use flag in `/etc/bpm/bpm.conf` so packages will not install service files, remove or simply do not install the `baseinit` package, and install something else.

If you're using something that doesn't just kick off scripts and supervise the result for services, you will have to write your own services.

There are currently no other init/supervision setups in the core repository, so packages for other inits/service managers should go in your own package repository (setting this up is much easier than it sounds or would be on most other distros!).
