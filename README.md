# Minimal QEMU Environment

## Summary

This repository contains an (almost) stand-aloneenvironment for
running qemu as well as some scripts for creating and managing the
image and a script to run qemu. This environemnt is targetted at
testing RDMA using SoftRoCE but can be used for many other things.

There is a minimalist kernel config in this repo as well that can be used as
a starting point for building a suitable kernel. It should be used with a
[SoftRoCE kernel](https://github.com/SoftRoCE/rxe-dev). A kernel patch is
included in this repo to allow using rxe compiled into the kernel (not as a
module). A sample bzImage, compiled from this kernel is included in this repo.

To run qemu using this image from the command should be:

```
./run-host1 <path_to_bzimage>
```

You can run
```
./run-host1 -h
```
to get the command line arguments supported.

This script will automatically create a snapshot image so you can revert
to the original image by deleting images/host1.qcow2.

A second host that is connected to the first host can be launched by
using the run-host2 script in the same fashion.

## QEMU Executable

This repo does not include the QEMU executable. You can specific a
path to the exe you want to run using the -q option. This is useful if
you are using a fork (or your own branch) of QEMU that has support for
specific hardware (e.g. LightNVM SSDs). Some useful links include the
[upstream](http://git.qemu-project.org/qemu.git) QEMU repo, Keith
Busch's [NVMe](git://git.infradead.org/users/kbusch/qemu-nvme.git)
repo and Matias Bjorling's
[LightNVM](https://github.com/OpenChannelSSD/qemu-nvme) fork.

Note you might want to track all three of these and install them all
since certain things are only supported in certain forks. With luck,
over time, support for all things will end up upstream ;-).

Note that by default KVM support is turned on. Use the -k switch to
turn this off (and suffer the wrath of slowness).

## Image Features

The runqemu script will automatically boot queitly and login as root
giving a shell over stdio (which is managable but can have some rough
edges). When the user logs out it will automatically powerdown the
machin and exit qemu. An SSH login is also available, while running,
forwarded to localhost port 3324. The root password is 'awhisten'.

The host's /home file is also passthrough mounted to the guests /home
directory so test scripts, etc can be stored and run directly from the
users home directory on the host.

QEmu's gdb feature is turned on so you may debug the kernel using
gdb. Note that you run the first command below from a shell prompt and
the second from within gdb. Note you should run these two command
before invoking QEMU.

```
(shell) gdb vmlinux
(gdb)   target remote :1234
```

## RDMA Features

The images come with infinband tools installed as well as librxe. On
boot it adds eth1 (which is connected between the two hosts) as a
SoftRoCE intefrace. Perftools and other tests can then be run between
the hosts.

## Scripts Folder

There are a few scripts in the scripts sub-folder that can be used to
re-generate the jessie-clean.qcow2 image. To regenerate this file or
create a new base image use the following steps:

   1. sudo modprobe nbd max_part=8 - this makes sure that Network
   Block Device kernel module is loaded. We need this for step 2, 3
   and 4.

   2. ./create <image name> - this creates the bare qcow2 image,
   partitions the image and then uses deboostrap to setup the image as
   a chroot with a basic Debian Jessie install.

   3. ./setup <image name> - this configures the Debian Jessie
   install. Setups up networking, machine name and some other key
   attributes. Note you can change the machine name and root password
   by editing this script.

   4. ./shrink <image name> - This script shrinks the image file as
   much as possible by doing zerofree and then running qemu-img
   convert.

We are happy to consider PRs for other -clean images as long as they
utilize the Large File Storage (LFS) feature or are pulled in via curl
or wget.

## Large File Storage

This repo utilizes git Large File Storage (lfs) in order to avoid
having to host the large jessie-clean.qcow2 image inside the repo. See
[here](https://git-lfs.github.com/) for more information.
