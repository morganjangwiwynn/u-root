# u-root

[![Build Status](https://circleci.com/gh/u-root/u-root/tree/master.png?style=shield&circle-token=8d9396e32f76f82bf4257b60b414743e57734244)](https://circleci.com/gh/u-root/u-root/tree/master)
[![Go Report Card](https://goreportcard.com/badge/github.com/u-root/u-root)](https://goreportcard.com/report/github.com/u-root/u-root)
[![GoDoc](https://godoc.org/github.com/u-root/u-root?status.svg)](https://godoc.org/github.com/u-root/u-root)
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://github.com/u-root/u-root/blob/master/LICENSE)

# Description

u-root contains simple Go versions of many standard Linux tools, similar to
busybox. It's a pure Go userland!

u-root stands for "universal root". It can create an initramfs in two different
modes:

*   source mode: Go toolchain binaries + simple shell + Go source for tools to
    be compiled on the fly by the shell.

    When you try to run a command that is not built, it is compiled first and
    stored in tmpfs. From that point on, when you run the command, you get the
    one in tmpfs. Don't worry: the Go compiler is pretty fast.

*   bb mode: One busybox-like binary comprising all the Go tools you ask to
    include.

    In this mode, u-root copies and rewrites the source of the tools you asked
    to include to be able to compile everything into one busybox-like binary.

# Contributing

For information about contributing, including how we sign off commits, please
see CONTRIBUTING.md

# Usage

Make sure your Go version is 1.12. Make sure your `GOPATH` is set up correctly.

Download and install u-root:

```shell
go get github.com/u-root/u-root
```

You can now use the u-root command to build an initramfs. Here are some
examples:

```shell
# Build a bb-mode cpio initramfs of all the Go cmds in ./cmds/...
u-root -build=bb

# Generate a cpio archive named initramfs.cpio.
u-root -format=cpio -build=source -o initramfs.cpio

# Generate a bb-mode archive with only these given commands.
u-root -format=cpio -build=bb ./cmds/core/{init,ls,ip,dhclient,wget,cat}
```

`-format=cpio` and `-build=source` are the default flag values. The default set
of packages included is all packages in
`github.com/u-root/u-root/cmds/core/...`.

In addition to using paths to specify Go source packages to include, you may
also use Go package import paths (e.g. `golang.org/x/tools/imports`) to include
commands. Only the `main` package and its dependencies in those source
directories will be included. For example:

You can build the initramfs built by u-root into the kernel via the
`CONFIG_INITRAMFS_SOURCE` config variable or you can load it separately via an
option in for example Grub or the QEMU command line or coreboot config variable.

A good way to test the initramfs generated by u-root is with qemu:

```shell
qemu-system-x86_64 -nographic -kernel path/to/kernel -initrd /tmp/initramfs.linux_amd64.cpio
```

Note that you do not have to build a special kernel on your own, it is
sufficient to use an existing one. Usually you can find one in `/boot`.

> NOTE: you can compress the initramfs but for xz compression, the kernel has
> some restrictions on the compression options and it is suggested to align the
> file to 512 byte boundaries `shell xz --check=crc32 -9 --lzma2=dict=1MiB
> --stdout /tmp/initramfs.linux_amd64.cpio \ | dd conv=sync bs=512
> of=/tmp/initramfs.linux_amd64.cpio.xz`

You may also include additional files in the initramfs using the `-files` flag.
If you add binaries with `-files` are listed, their ldd dependencies will be
included as well. As example for Debian, you want to add two kernel modules for
testing, executing your currently booted kernel:

> NOTE: these files will be placed in the `$HOME` dir in the initramfs.

```shell
u-root -files "$HOME/hello.ko $HOME/hello2.ko"
qemu-system-x86_64 -kernel /boot/vmlinuz-$(uname -r) -initrd /tmp/initramfs.linux_amd64.cpio
```

To specify the location in the initramfs, use `<sourcefile>:<destinationfile>`.
For example:

```shell
u-root -files "root-fs/usr/bin/runc:usr/bin/run"
```

## Getting Packages of TinyCore

Using the `tcz` command included in u-root, you can install tinycore linux
packages for things you want.

You can use QEMU NAT to allow you to fetch packages. Let's suppose, for example,
you want bash. Once u-root is running, you can do this:

```shell
% tcz bash
```

The tcz command computes and fetches all dependencies. If you can't get to
tinycorelinux.net, or you want package fetching to be faster, you can run your
own server for tinycore packages.

You can do this to get a local server using the u-root srvfiles command:

```shell
% srvfiles -p 80 -d path-to-local-tinycore-packages
```

Of course you have to fetch all those packages first somehow :-)

## Build an Embeddable U-root

You can build this environment into a kernel as an initramfs, and further embed
that into firmware as a coreboot payload.

In the kernel and coreboot case, you need to configure ethernet. We have a
`dhclient` command that works for both ipv4 and ipv6. Since v6 does not yet work
that well for most people, a typical invocation looks like this:

```shell
% dhclient -ipv4 -ipv6=false
```

Or, on newer linux kernels (> 4.x) boot with ip=dhcp in the command line,
assuming your kernel is configured to work that way.

## Updating Dependencies

```shell
# The latest released version of dep is required:
curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
dep ensure
```

# Hardware

If you want to see u-root on real hardware, this
[board](https://www.pcengines.ch/apu2.htm) is a good start.

# Contributions

See [CONTRIBUTING.md](CONTRIBUTING.md)

Improving existing commands (e.g., additional currently unsupported flags) is
very welcome. In this case it is not even required to build an initramfs, just
enter the `cmds/` directory and start coding. A list of commands that are on the
roadmap can be found [here](roadmap.md).

# systemboot

[![Build Status](https://travis-ci.org/systemboot/systemboot.svg?branch=master)](https://travis-ci.org/systemboot/systemboot)
[![codecov](https://codecov.io/gh/systemboot/systemboot/branch/master/graph/badge.svg)](https://codecov.io/gh/systemboot/systemboot)
[![Go Report Card](https://goreportcard.com/badge/github.com/systemboot/systemboot)](https://goreportcard.com/report/github.com/systemboot/systemboot)

SystemBoot is a distribution for LinuxBoot to create a system firmware +
bootloader. It is based on [u-root](https://github.com/u-root/u-root). The
provided programs are:

*   `netboot`: a network boot client that uses DHCP and HTTP to get a boot
    program based on Linux, and uses kexec to run it
*   `localboot`: a tool that finds bootable kernel configurations on the local
    disks and boots them
*   `uinit`: a wrapper around `netboot` and `localboot` that just mimicks a
    BIOS/UEFI BDS behaviour, by looping between network booting and local
    booting. The name `uinit` is necessary to be picked up as boot program by
    u-root.

This work is similar to the `pxeboot` and `boot` commands that are already part
of u-root, but approach and implementation are slightly different. Thanks to
Chris Koch and Jean-Marie Verdun for pioneering in this area.

This project started as a personal experiment under
github.com/insomniacslk/systemboot but it is now an effort of a broader
community and graduated to a real project for system firmwares.

The next sections go into further details.

## netboot

The `netboot` client has the duty of configuring the network, downloading a boot
program, and kexec'ing it. Optionally, the network configuration can be obtained
via SLAAC and the boot program URL can be overridden to use a known endpoint.

In its DHCP-mode operation, `netboot` does the following: * bring up the
selected network interface (`eth0` by default) * make a DHCPv6 transaction
asking for network configuration, DNS, and a boot file URL * extract network and
DNS configuration from the DHCP reply and configure the interface * extract the
boot file URL from the DHCP reply and download it. The only supported scheme at
the moment is HTTP. No TFTP, sorry, it's 2018 (but I accept pull requests) *
kexec the downloaded boot program

There is an additional mode that uses SLAAC and a known endpoint, that can be
enabled with `-skip-dhcp`, `-netboot-url`, and a working SLAAC configuration.

## localboot

The `localboot` program looks for bootable kernels on attached storage and tries
to boot them in order, until one succeeds. In the future it will support a
configurable boot order, but for that I need
[Google VPD](https://chromium.googlesource.com/chromiumos/platform/vpd/)
support, which will come soon.

In the current mode, `localboot` does the following: * look for all the locally
attached block devices * try to mount them with all the available file systems *
look for a GRUB configuration on each mounted partition * look for valid kernel
configurations in each GRUB config * try to boot (via kexec) each valid
kernel/ramfs combination found above

In the future I will also support VPD, which will be used as a substitute for
EFI variables, in this specific case to hold the boot order of the various boot
entries.

## uinit

The `uinit` program just wraps `netboot` and `localboot` in a forever-loop
logic, just like your BIOS/UEFI would do. At the moment it just loops between
netboot and localboot in this order, but I plan to make this more flexible and
configurable.

## How to build systemboot

*   Install a recent version of Go, we recommend 1.10 or later
*   make sure that your PATH points appropriately to wherever Go stores the
    go-get'ed executables
*   Then build it with the `u-root` ramfs builder using the following commands:

```
go get -u github.com/u-root/u-root
go get -u github.com/systemboot/systemboot/{uinit,localboot,netboot}
u-root -build=bb core github.com/systemboot/systemboot/{uinit,localboot,netboot}
```

The initramfs will be located in `/tmp/initramfs_${platform}_${arch}.cpio`.

More detailed information about the build process for a full LinuxBoot firmware
image using u-root/systemboot and coreboot can be found in the
[LinuxBoot book](https://github.com/linuxboot/book) chapter 11,
[LinuxBoot using coreboot, u-root and systemboot](https://github.com/linuxboot/book/blob/master/11.coreboot.u-root.systemboot/README.md).

## Example: LinuxBoot with coreboot

One of the ways to create a LinuxBoot system firmware is by using
[coreboot](https://coreboot.org) do the basic silicon and DRAM initialization,
and then run Linux as payload, with u-root and systemboot as initramfs. See the
following diagram:

![LinuxBoot and coreboot](resources/LinuxBoot.png) (images from coreboot.org and
wikipedia.org, diagram generated with draw.io)

## Build and run as a fully open source bootloader in Qemu

Systemboot is one of the parts of a bigger picture: running Linux as firmware.
We call this [LinuxBoot](https://linuxboot.org), and it can be achieved in
various ways. One of these is by combining [coreboot](https://coreboot.org),
[Linux](https://kernel.org), [u-root](https://u-root.tk) and `systemboot`. Check
out the instructions on the
[LinuxBoot using coreboot, u-root and systemboot](https://github.com/linuxboot/book/tree/master/11.coreboot.u-root.systemboot)
chapter of the [LinuxBoot Book](https://github.com/linuxboot/book).

## TODO

*   verified and measured boot
*   a proper GRUB config parser
*   backwards compatibility with BIOS-style partitions
