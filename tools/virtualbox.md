# Guest Additions
The vboxusers group does not appear to get created as part of the installation,
so need to create it with `groupadd` and then add the guest user as a member.

Note that these notes are largely for instances where you are running a Linux
guest from a Mac OS or Windows host operating system of some sort.

## Dependancies
On Linux guests, installing the guest additions requires a few packages prior to
trying to install them.  There are scores of people on the internet that are
confused about guest additions, are incapable of reading the VirtualBox
documentation, or some combination of the two.  At least on Debian, the
following need to be installed
```bash
$ apt-get install build-essential dkms linux-headers-$(uname -r)
```
Some distributions may come with the kernel headers already installed, but these
are generally what are required to build the kernel modules that are required.

## Installation
Note to folks reading this - there is a ton of horrible information on the
internet about installing guest additions, probably because no one is capable of
reading user documentation, reference manuals, etc. which clearly describe how
to do it.  The only reason I include it here is because I hadn't done it in a
while and figured I would add some notes for anyone that came across my notes.
Also, as it turns out, I ran into a snag that took a few minutes to straighten
out (Debian by default apparently mounted my guest additions media in such a
fashion I could not actually run the installer script).

To install the guest additions, insert the guest additions CD using the
`Devices` menu. Depending upon the way your system is set up, this may be
trivial or require some more hacking.  For me, at least with this variant of
Debian 11 and the way it installed out of the box, it required a bit of work.
First, by default the user created during the installation process is not a
member of the `sudo` group and is no in `/etc/sudoers`. Using `visudo` you will
need to add them.  Log out and then log back in.

With that out of the way, after inserting the guest additions CD, you should see
a newly mounted filesystem somewhere (if you don't know about the `findmnt`
command then you're in luck). In my case, it was mounted with the `noexec` flag
set so if that is the case for you, then you will need to remount the filesystem
with something like
```bash
$ sudo mount -o remount,exec,ro /media/cdrom0
```
Run `findmnt` again and you should a) see that the `noexec` flag is gone and b)
be able to run the guest additions installer script from the media. There are
probably folks that suggest copying everything to a location that you actually
*can* execute commands from, but that is silly. At any rate, run the guest
installation script, which should run to completion (it will fail if you do not
have the build tools installed). You will need to restart the guest and once you
do, you should see something like
```bash
$ sudo dmesg | grep -i guest
[    0.053644] kvm-guest: PV spinlocks enabled
[    1.431431] [drm]   Guest Backed Resources.
[    2.026002] vboxguest: loading out-of-tree module taints kernel.
[    2.026127] vboxguest: module verification failed: signature and/or required key missing - tainting kernel
[    2.031182] vboxguest: Successfully loaded version 7.0.6 r155176
[    2.031192] vboxguest: misc device minor 61, IRQ 20, I/O port d040, MMIO at 00000000f0400000 (size 0x400000)
[    2.031192] vboxguest: Successfully loaded version 7.0.6 r155176 (interface 0x00010004)
[    2.429703] 15:31:26.012717 main     Executable: /opt/VBoxGuestAdditions-7.0.6/sbin/VBoxService
[    2.431335] 15:31:26.014344 main     vbglR3GuestCtrlDetectPeekGetCancelSupport: Supported (#1)
```
Things like shared directories, full screen resolution, etc. should all work
once configured, although some things depend upon graphics acceleration to
actually work properly.

# Networking
For VM guests that use the VirtualBox NAT engine, the following are potentially
useful modifications, particularly when attempting to resolve domain names that
are only known by the host via a VPN.  I'm sure it is more complicated than what
I've just described, but for a scenario with a host using a VPN that
resolves names like `internal-only.foo.com` the following commands will set the
host DNS resolver as the DNS proxy of that VM:
```bash
$ VBoxManage list vms
```
This will identify the names and UUID being managed by the guest, and then
```bash
$ VBoxManage modifyvm <UUID> --natdnshostresolver1 on
```
Note that these commands are run from the host (e.g., PowerShell on Windows).
For more information, see [this section](https://www.virtualbox.org/manual/ch09.html#changenat) of the VirtualBox documentation.  Also, this is an example of features exposed by the VirtualBox virtualization engine that are not available in the GUI.


# Debugging
A common occurrence after upgrading the VirtualBox host application is that some
VMs will be inaccessible due to mismatches between the UUID of the inserted
Guest Additions medium.  This results in an error of the sort like
```
Cannot register the DVD image
'/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso'
<UUID> because a CD/DVD image
'/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso'
with UUID <UUID> already exists.
```
This question gets asked a lot on the internet and the oft supplied suggestion
is to open the XML file for the virtual machine and start hacking on it.  The
more appropriate way is to use the Media Manager to remove and release the
image.  This will render the VM accessible again.  There is no doubt a better
way to do this using the `vboxmanage` tools, but I haven't had a chance to
figure that out yet.
