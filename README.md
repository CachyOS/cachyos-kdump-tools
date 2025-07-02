# cachyos-kdump-tools

## Description

This is a set of tools needed to automatically create and save a dump together
with dmesg log in case of a kernel panic in your CachyOS system, which is
especially useful when debugging random system freezes.

## Setup

This tool was specially made for beginners, so its usage consists of four
steps:

1) Install cachyos-kdump-tools package from our repository: ``sudo pacman -S
cachyos-kdump-tools``

2) Add kernel parameter ``crashkernel=256M`` to your bootloader configuration
manually or with ``sudo kdump setup`` command.

3) Enable kdump service: ``sudo systemctl enable kdump.service``

4) Restart system to apply the paramater.

## How it works

This works on top of the kexec-tools, which perform loading of a "fallback"
kernel in case of panic. It is necessary because all I/O operations performed
by the primary kernel cannot be performed safely after an error occurs. To load a
fallback kernel, you must first reserve area in your system's memory using the
``crashkernel`` kernel parameter.

Once kernel panic occurs and fallback kernel is booted, your system enters the
emergency stage, in which kdump-collected service is started, extracting the
dump and log, and then saving them in ``/var/crash`` directory.

Your system should reboot automatically at this point. Note that writing dump
on the disk takes time, so don't be surprised if nothing happens for first 1-2
minutes after kernel panic.

After rebooting, you can send resulting dump to developers, or debug it
yourself using the [crash](https://crash-utility.github.io/) or
[drgn](https://github.com/osandov/drgn) utilities. Note that when debugging
yourself, you will also need to have debugging symbols for kernel image and
modules used, otherwise this makes the dump useless. In CachyOS we distribute
unstripped kernel image and modules with ``linux-cachyos-dbg`` package.

> [!CAUTION]
> The dump contains the full memory content after the kernel panic
> occurred. It may contain privacy-sensitive data, so it is not recommended to
> share it with third parties. It is recommended to trust dumps only to
> developers.
>
> At the same time, dmesg log can be provided safely to form an idea of
> the issue.

## Limitations

This tool will not help if your file system has been damaged or if panic occurs
early in the system boot process. Although the first case is partially handled
by forcing fsck to be used during boot, this does not provide any guarantees.
