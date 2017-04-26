# The continuing saga of my /boot partition

## Problem

On boot, cannot type the password into (graphical) prompt in order to decrypt the laptop's disk and boot into the full desktop.

Cause: `/boot` fills up and newer kernels are either not installed or are installed buggily.

Workaround: reboot to grub, Ctrl-Alt-Del works, and select a different, working, kernel.
So far, I've discovered that the oldest one (installed with original distro, presumably) works.

## Observations

Most of the bulk space is taken up by `initrd.img` files

## What doesn't work

Suggested by Jez Cope on 2017-04-20:

    sudo update-initramfs -k all -u
    
Note to previous: This does appear to successfully do what is intended:
the `initrd.img*` files are rebuilt (they change sizes a little bit; hope it doesn't matter).

Suggested by Will Furnass on 2017-04-26:

Move the (suspect) `initrd.img` file away and create it fresh with

    update-initramfs -c -k 3.19.whatever
    
(you have to specify the full filename suffix, eg `3.19.0-65-generic`)


## tips, tricks, and diagnostics

List kernels.

    sudo dpkg -l "linux-image*"
    
(see `man dkpg-query` to interpret the output)

List packages owning each file in `/boot/`:

    dpkg -S /boot/*

Nots to previous: the `initrd.img*` files and the `vmlinux*.efi.signed` files are not owned by any package.
(On my current system) The only packages are
`memtest86+` and a selection of different versions of the Linux kernel: `linux-image-*-generic`

The `initrd.img*` files are `cpio` archives. to list their contents:

    cpio -t < initrd.img.whatever
    
On my current system, those files are far too big for the `cpio` archive that the above command lists
(the `cpio` archive is 30 blocks or 15KB).
Following the `cpio` archive in the file is a `gzip` file:

    { cpio -t > /dev/null ; file - ; } < initrd.img-3.19.0-28-generic

of:

    { cpio -t > /dev/null ; zcat | file - ; } < initrd.img-3.19.0-28-generic

a `cpio` file! (it's `cpio` files all the way down).

Note to previous: After `cpio` reads the archive from the file, the file offset is left in place,
at exactly the right point to read the gzip archive.

So to read the file listing of the inner `cpio` archive:

    { cpio -t > /dev/null ; zcat ; } < initrd.img-3.19.0-28-generic  | cpio -t
    
