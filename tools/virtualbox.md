# Guest Additions
The vboxusers group does not appear to get created as part of the installation,
so need to create it with `groupadd` and then add the guest user as a member.

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



