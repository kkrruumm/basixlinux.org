title: Basix Wiki

# nemu

The supported QEMU frontend for Basix is [nemu](https://github.com/nemuTUI/nemu).

nemu is an ncurses UI for QEMU and does not require libvirt. Overall, this is reasonably powerful and should serve most needs.

To set this up, install the `nemu` package.

You may need to modprobe `tun`, `virtio_net`, and the KVM module for your particular CPU if you want acceleration, `kvm_intel` for example. Take a look at the [kmodules](/wiki/kmodules) page for instructions on loading these on boot.

You should make sure you have added your user to the `kvm` and `qemu` system groups.

## Bridging with nemu

Sometimes it may be desirable to have a virtual machine act as though it is a machine on your normal LAN. To do this with nemu, make sure you have already created a bridge interface, `br0` for example.

You should also have the `bridge-helper` use flag enabled on qemu, and the `allow br0` line in `/etc/qemu/bridge.conf` uncommented.

Upon setting up your bridge, you can navigate to your VM configuration in nemu and set, for example,

```
-netdev bridge,br=br0,id=lan0 -device virtio-net-pci,netdev=lan0,mac=<MAC ADDRESS>
```

in your "Extra QEMU args," noting the `<MAC ADDRESS>` needs to be replaced with the MAC you'd like to provision to this VM.

To generate a MAC address that follows typical QEMU convention, one may run:

```
hexdump -n3 -e '"52:54:00:" 2/1 "%02x:" 1/1 "%02x\n"' /dev/urandom
```

and replace `<MAC ADDRESS>` with the output.

Make sure you've set "Network interfaces" to 0 on your VM as well.

## UEFI with nemu

After having installed QEMU, you should have a few files in `/usr/share/qemu/`.

Given nemu passes the flash as writable, one should copy their flash image out to their user directory, and pass that into nemu.

To do this, for example:
```
cp /usr/share/qemu/edk2-x86_64-code.fd ~/nemu_vm/testvm/OVMF.fd
```

on a per-VM basis.

After doing so, in your VMs boot settings, remove the "Path to BIOS" value and enter the path to your copied image in "Path to flash"

For example, ``/home/kris/nemu_vm/voidvm/OVMF.fd``.

## Viewing a nemu VM

You may install the `virt-viewer` package, or just the `spice-gtk` package as it includes its own spice viewer.

Assuming one has installed just `spice-gtk`, open up your nemu config (`~/.config/nemu/nemu.config`) in a text editor.

You should find a `spice_bin` option, and you may set this to the location of your viewer binary.

For example, ``spice_bin = /usr/bin/spicy``

nemu will automatically use this when you request a viewer from the UI.

# libvirt

libvirt is in the repository, though is more or less not recommended. Nonetheless, if you would like to use it, install the `libvirt` package and enable the ``libvirtd libvirtd-log virtlogd virtlogd-log`` services.

After installation, standard `virsh` commands will be available to control your virtual machines. You will have to set up your VMs XML yourself.

Follow other distros documentation regarding libvirt usage.
