+++
title = "The Linux File System Structure"
date = 2020-04-16T21:37:45+02:00
draft = true
author = "Marc Benedi"
authorTwitter = "marcbenedi" #do not include @
cover = "img/the-linux-file-system-structure.png"
tags = ["Linux", ""]
keywords = ["", ""]
description = ""
showFullContent = false
+++

> Cover source: https://i.pinimg.com/564x/d0/04/28/d00428efa0bf27b9edd37eac32dfd2c1.jpg

I know there are thousands of blogs around The Internet that have already explained this. This is only for myself, to see if writing it down helps me remember it better. However, I hope it can be useful to somebody else. I definetely recommend looking at https://jlk.fjfi.cvut.cz/arch/manpages/man/file-hierarchy.7

# /bin/ 
where binaries are stored

points to /usr/bin

# /boot/ 
This folder contains the files requierd for bringing up the system. 


This directory only exists on systems that run on physical or emulated hardware that requires boot loaders. 

## /boot/EFI/

### /boot/EFI/BOOT/BOOTX64.EFI

efi = extensible firmware interface

efi files are boot loader executables.

### /boot/EFI/systemd/systemd-bootx64.efi
systemd-boot is a simple uefi boot manager
## /bot/initramfs-linux.img

## /boot/loader/

### /boot/loader/loader.conf
systemd-boot configuration
### /boot/loader/random-seed

It is one of the sources for initializing the entropy pool during boot. For more information check [this](https://systemd.io/RANDOM_SEEDS/). It explains very clearly how Linux manages random numbers.

### /boot/loader/entries/arch.conf

A boot entry.

## /boot/vmlinuz-linux

The compressed and bootable kernel. It should not be confused with vmlinux, which is the non-compressed and non-bootable kernel form. It is usually an intermediate step to producing vmlinuz. 

# /dev/ 
where physical devices are mounted

Mounted as a "devtmpfs" instance. Managed by kernel and systemd-udevd (device event managing daemon)

## /dev/shm/

Place for POSIX shared memory segments.
POSIX: Portable Operating System Interface is a familiy of standards specified by the IEEE computer society for maintaining compatibility between operating systems. 

# /etc/ 
where configuration files are stored, system wide

Pre-populated with vendor configuration files, but applications should not make assumptions.

## /etc/package

System-specific ocnfiguration for the package

# /home/
user directories
$HOME

## ~/.cache/

persistent user cache data. user programs may place non-essential data in this directory.

### ~/.cache/package/

Persistent cache data of the package. 

## ~/.config/

Application configuration and state. 

### ~/.config/package/

User-specific configuration and state for the package

## ~/.local/

### ~/.local/bin/

Package executable that shall appear in the $PATH. 
Not recommended to place executables that are not useful for invocation from shell. these should be placed in ~/.local/lib/instead. 

### ~/.local/lib/

Static, private vendor data that is compatible with all architectures. 

#### ~/.local/lib/arch-id/

Location for placing public dynamic libraries. 

Public shared libraries of the package

##### ~/.local/lib/arch-id/package/

Private other vendor resources of the package that are architecture-specific.

#### ~/.local/lib/package/

Private, static vendor resources of the package, compatible with any architecture, or any other kind of read-only vendor data.

### ~/.local/share/

Resources shared between multiple packages, such as fonts. 

Additional static vendor files

# /lib/ 
libraries
points to /usr/lib
# /lib64/
points to /usr/lib

# /lost+found/

https://unix.stackexchange.com/questions/18154/what-is-the-purpose-of-the-lostfound-folder-in-linux-and-unix

# /mnt/ 
placeholder for mounting other folders or drived

# /opt/ 
optional software that is not already managed by package manager

# /proc/ 

A virtual kernel file system exposing the process list and other funcionality. Is mostly an API interface with the kernel. 

Processes folder where a lot of system information is represented as files. 

## /proc/sys/

Exposes a number of kernel tubables

# /root/ 
root user folder

mounted outside /home/ to make sure the root user may log in even without /home/ being available and mounted

# /run/
A tmpfs file system for system packages to place runtime data in. 

## /run/log/
Runtime system logs. 

### /run/log/package/

Runtime log data for the package. 

## /run/user/
Contains per-user runtime directories, each usually individually mounted "tmpfs" instances. 

$XDG_RUNTIME_DIR

## /run/package/

Runtime data for the package. Flushed automatically on boot. 

# /sbin/ 
same as /bin but for superuser binaries
points to /usr/bin

cannot be executed by normal user
# /srv/

The place to store general server payload. 

# /sys/

A virtual kernel file system exposing discovered devices and other functionalities. is api interface with kerne. 

# /tmp/ 
temporary files

Usually mounted as "tmpfs" and should not be used for larger files. (User /var/tmp/ for larger files).

Usually flushed at boot-up. 

## tmpfs filesystem
- The filesystem can employ swap space when physical memory pressure demands it
- Only consumes as much memory as it needs to store content
- During remount, the size can be changed. 

# /usr/ 
Files and utilities shared between users

vendor-supplied operating system resources

## /usr/bin/

Package executables that shall appear in the $PATH, compiled for any of th supported architectures compatible with the OS. Not recommended to place binaries taht are not commonly invoked from the shell. 

## /usr/sbin/
points to /usr/bin
extend set of commands to /sbin folder
## /usr/include/

### /usr/include/package/

Public C/C++ APIs of public shared libraries of the package

## /usr/lib/

### /usr/lib/arch-id/

Public shared libraries of the package

#### /usr/lib/arch-id/packages/

Private other vendor resources of the package that are architecture specific and cannot be shared between architectures. 

### /usr/lib/package/

Private static vendor resources of the package, including private binaries and libraries.

## /usr/share/

### /usr/share/doc/

#### /usr/share/factory/etc/

#### /usr/share/factory/var/

# /var/ 
where variable data is kept, usually system logs 

## /var/cache/

### /var/chache/package/

Persistent cache data of the package. 

## /var/lib/

### /var/lib/package/

Persistent private data of the package.

## /var/log/

### /var/log/package/

Persistent log data of the package. 

## /var/spool/

Contains data which is awaiting some kind of later processing

### /var/spool/package/

Persistent spool/queue data of the package.

## /var/tmp/

## /var/run/
Points to /run/

# Extra: Types of Unix files

- Ordinary Files: Contains data, text, or program instructions. A long-format output of ls -l, "-" symbol.
- Directories: Store both special and ordinady files. "d"
- Special Files: real physical device such as printer, terminal I/O. 
    - Character special files: data transferred one char at time "c"
    - BLock special files: data transfered in large fixed-size blocks "b"
- Pipes: acts a temportray file which only exist s to hold data from one command until it is read by another. one-way flow data. "p"
- Sockets: (inter-process communication socket) allows for advanced inter-process communicaon. Used in a client-server. stream of daat. "s"
- Symbolic Links: for referencing some other file. known as soft link. if we delete the soft link, data will still be tehre. if we delete source file, symbolic link will fail. "l"

# References

[1] http://www.linuxandubuntu.com/home/the-linux-file-system-structure-explained

[2] https://jlk.fjfi.cvut.cz/arch/manpages/man/file-hierarchy.7

[3] http://man7.org/linux/man-pages/man5/tmpfs.5.html

[4] https://wiki.archlinux.org/index.php/Tmpfs

[5] https://en.wikipedia.org/wiki/POSIX

[6] https://www.geeksforgeeks.org/unix-file-system/

[7] https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s14.html

[8] https://unix.stackexchange.com/a/18157

[9] https://wiki.archlinux.org/index.php/systemd-boot

[10] https://systemd.io/RANDOM_SEEDS/

[11] http://www.linfo.org/vmlinuz.html