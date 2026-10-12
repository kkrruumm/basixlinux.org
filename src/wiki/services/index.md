title: Basix Wiki

# Introduction

By default, Basix currently uses the [s6](https://skarnet.org/software/s6/) init/supervision suite.

This includes:
  - [s6](https://skarnet.org/software/s6/)
  - [s6-rc](https://skarnet.org/software/s6-rc/)
  - [s6-linux-init](https://skarnet.org/software/s6-linux-init/)
  - [s6-frontend](https://skarnet.org/software/s6-frontend/)

In addition, Basix also defaults to skarnets [execline](https://skarnet.org/software/execline/) scripting language for typical services (though, core services are still POSIX shell, bear with me)

In general, everything from skarnet is abnormally high quality ~~albeit baiscally impossible to interact with in the case of s6 without its frontend~~, incredibly modular, small, efficient, and powerful.

All Basix documentation will assume the usage of s6-frontend.

This page isn't meant to be an all-encompassing guide to using s6, but rather just enough to get a user going. Those who want more information should visit the [skarnet website](https://skarnet.org/) and read around, or, read the manpages shipped with s6-* packages on Basix.

# System services

When a package ships a service, its "source" directory, that being the directory that contains its `run` script and otherwise, it's placed in `/usr/share/s6-rc/sources`. This is not meant to be directly interacted with on Basix, but rather by service-providing packages. You may reference packages in the repository that provide services of some form, such as [openntpd](https://github.com/kkrruumm/basix-packages/tree/main/openntpd) for this.

For services that the sysadmin may create, they are to be placed in `/etc/s6-rc/sources` (more on that later).

## Usage

s6 is not particularly similar in usage command wise to something along the lines of runit. s6 is largely (entirely? dont quote me) free of polling, and `/sbin/init` isn't a static thing.

What this means for the sysadmin is that when an modifying a system service, adding a new service, whatever it may be- the "database" has to be rescanned, recompiled, and optionally applied (committed, basically) so it will stick between boots.

For commands, largely, when using s6-frontend (for all that is good please do), the help output covers usage pretty well:

```
(0) sagittarius ~ > s6 --help
Usage: s6 [ global_options... ] command subcommand [ subcommand_options... ] [ args... ]

For details about a command and its options, try: s6 command help (e.g.: s6 live help)

Commands:
  help		this message
  version	print the version of this package
  system	boot and shutdown the system
  live		start and stop services and their dependencies
  process	control individual long-running processes
  set		manipulate sets of services (offline, to later install them live)
  repository	manipulate the service repository itself

Shortcuts:
  enable	s6 set enable
  disable	s6 set disable
  start		s6 live start
  stop		s6 live stop
  kill		s6 process kill
  apply		s6 set apply

Global options:
  -h		--help			equivalent to "s6 help"
  -u		--user			override config values with XDG environment

		--color=yes|no|auto	display fancy or boring-but-parsable text.
					auto is the default: fancy if in a terminal.

 These options override the values set in s6-frontend.conf:
  -v V		--verbosity=V		verbosity (0 is terse, 1 is normal/default, more is verbose)
  -s SCANDIR	--scandir=SCANDIR	where s6-svscan is running
  -l LIVEDIR	--livedir=LIVEDIR	s6-rc live directory
  -r REPODIR	--repodir=REPODIR	s6-rc service set repository
  -c BOOTDB	--bootdb=BOOTDB		live database the system is booting on
		--stmpdir=STMPDIR	root-only temporary directory
		--storelist=LIST	list of service stores
		--fdholder-user=USER	the s6-fdholderd user embedded in service databases
```

### Listing available services

When running `s6 set list`, a list of services in general will be listed.

When running `s6 set status`, a list of services in general will be listed but will also include their current state. Example:

```
(0) sagittarius ~ > s6 set status
sysctl/always
swap/always
random-seed/always
mount-fs/always
loopback/always
hostname/always
fsck/always
devices/always
getty-tty2/active
getty-tty1/active
mdevd-coldplug/active
mdevd/active
run-user-dir/active
nftables/active
virtlogd-log/active
virtlogd/active
seatd-log/active
seatd/active
ntpd-log/active
ntpd/active
libvirtd-log/active
libvirtd/active
greetd-log/active
greetd/active
dbus-log/active
dbus/active
acpid-log/active
acpid/active
zram/active
hidepid/active
gpu-downclock/active
cpu-governor/active
bridge/active
sshd-log/usable
sshd/usable
snooze-15m-log/usable
snooze-15m/usable
snooze-daily-log/usable
snooze-daily/usable
snooze-hourly-log/usable
snooze-hourly/usable
snooze-monthly-log/usable
snooze-monthly/usable
snooze-weekly-log/usable
snooze-weekly/usable
dhcpcd-log/usable
dhcpcd/usable
```

In this case, we can see that various services here are marked "always," meaning that they are marked with a `flag-essential` file. What this means is that they are considered fundamental services, and will *always* be run.

Other services here are tagged "active," which means enabled but not strictly required.

At the bottom of my list of services, there are a handful marked "usable," meaning they exist to the current database but are not enabled.

### Enabling and starting a service

If you wish to just start a service, this may be done by running `s6 start dhcpcd dhcpcd-log`, for example.

If you wish to enable and start a service while having it persist across reboots, you may run `s6 enable dhcpcd dhcpd-log && s6 start dhcpcd dhcpcd-log && s6 apply`.

Note the existence of the attached `-log` service on certain things, if a service has a `producer-for` and `consumer-for` setup both will be required. More information on that below.

### Disabling and stopping a service

Almost to be expected, you can run: ``s6 disable dhcpcd dhcpcd-log && s6 stop dhcpcd dhcpcd-log``. If you want this to persist across reboots, don't forget to ``s6 apply``.

## Custom services

If an admin wishes to create their own service, they may place it in `/etc/s6-rc/sources`.

A typical service directory here may follow this structure, as an incredibly simple oneshot service:

```
/etc/s6-rc/sources/zram/
├── dependencies.d
│   └── mdevd-coldplug
├── type
└── up
```

In this case, `dependencies.d/mdevd-coldplug` is just an empty file telling s6 that `mdevd-coldplug` should come up before this is run.

`up` contains our actual service script, in my case:

```
if { zramctl /dev/zram0 --algorithm lz4 --size 24G }
if { mkswap /dev/zram0 }
swapon -d -p 100 /dev/zram0
```

..and `type` contains the type of service this is. Typically, this is one of `longrun` or `oneshot`. If this were a `longrun` service, the executable would be titled `run` as opposed to `up`. In this case, this file only contains the text "oneshot"

Of course, you may also have a `down` script, if needed or wanted.

After creating your custom service, you should run ``s6 repository sync`` so that s6 will pick up your new service. Upon doing this, it should appear in your set listings, and you may enable and treat this like any other service.

If you wish to attach a logger to your service, create a second ``zram-log`` service that is the ``consumer-for`` zram, and zram is the ``producer-for`` the logger. To do this, just create those files and place the service names in the corresponding ``*-for`` files.

The logger service should, in addition, contain a ``notification-fd`` that generally contains ``3``, as a readiness check.

## Reading logs

By default, every service that may produce logs has an attached ``<service>-log`` service that spawns an attached s6-log process.

All of these services will get their own distinct directory in ``/var/log/<service>``. An example of this directory may be:

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

Given s6-log is reasonably sane, these are just plaintext logs. View them however you wish.

There is also a more generalized log directory at ``/run/uncaught-logs/``. Generally, this just contains logs from the boot/init process. This log is also only readable to root.


# Core system services

There are a handful of services on Basix that are considered "core" services which are run by the init system before any "normal services" get to run. These are provided by the ``baseinit`` package, and are stored in ``/usr/lib/s6-rc/scripts/``. 

An example of one of these scripts is the `devices` script, which is pretty much always required for anything else at all to be functional.

# I don't want to use s6!

Remove its packages and the ``baseinit`` package, disable the global `services` use flag, and use something else! [nitro](https://github.com/leahneukirchen/nitro) is one of my favorites, personally.

This will, however, mean you must write your own services, including core ones.
