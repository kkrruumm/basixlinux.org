title: Basix Wiki

# Loading kernel modules temporarily

Run `modprobe <module_name>` as root, same as any other system.

# Loading kernel modules on boot

Navigate to the `/etc/modules-load.d/` directory.

Here, you may create a `.conf` file to modprobe a specific kernel module during boot. Simply put the name of the kernel module you'd like to have load on boot in this file.

For example,

```
(1) sagittarius ~ > cat /etc/modules-load.d/zram.conf 
zram
(0) sagittarius ~ >
```

You may have as many of these as you want/need.
