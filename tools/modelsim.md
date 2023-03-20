Installing
----------
Installing the starter version of ModelSim from Mentor Graphics is relatively
painful, if you are on anything resembling a modern platform (i.e., 64-bit).
That said, it isn't difficult once you understand what needs to be done.

Before we get started, assume that the `vsim` executable is in someplace like
`/opt/intelFPGA/16.1/modelsim_ase/bin/vsim` and note that all the ModelSim tools
in that directory are essentially just symlinks to the `vco` script which is in
the ModelSim directory (for simplicity, I will refer to that `modelsim_ase`
directory as the ModelSim root from now on).  Further, note that the `vco`
script is itself just making calls into one of several other places.  More on
that later - having just installed the tool, the central problem is the
following:
```console
$ /opt/intelFPGA/16.1/modelsim_ase/bin/vsim 
Error: cannot find "/opt/intelFPGA/16.1/modelsim_ase/bin/../linux_rh60/vsim"
```
Well, that's helpful.  Now let's get to fixing that.

The following instructions work on Debian 11. The gist of what needs to happen
is the following:
- A hard-coded symlink needs to be fixed for the `vco` script to not look in
  a non-existent RedHat directory (that's the immediate problem seen in that
  rather helpful error message that ModelSim yielded).
- A large number of 32-bit libraries need to be installed on the operating
  system (if you're not using a virtual machine for this, you really should)
- Freetype fonts need (at least on Ubuntu 18 and 20 and elsewhere on the
  internet) to be rebuilt and installed somewhere the linker can see and the
  `LD_LIBRARY_PATH` environment variable hacked in the `vco` script so that
  they can be found at runtime. This isn't strictly necessary, but plenty of
  other instructions on the internet attempt to do this in some fashion, so
  in case someone else stumbles across this, we will do the same.

The first thing to do is to add support for 32-bit libraries (this is all
assuming i386 and x86-64 - if you're on something else, then you're on your
own).

```console
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo dpkg --add-architecture i386
$ sudo apt install \
	build-essential
$ sudo apt install \
	gcc-multilib \
	g++-multilib
$ sudo apt install \
	lib32z1 \
	lib32stdc++6 \
	lib32gcc-s1 \
	libxt6:i386 \
	libxtst6:i386
	expat:i386 \
     	fontconfig:i386 \
	libfreetype6:i386 \
	libexpat1:i386 \
	libc6:i386 \
	libgtk-3-0:i386 \
	libcanberra0:i386 \
	libice6:i386 \
	libsm6:i386     \
	libncurses5:i386
	zlib1g:i386 \
	libx11-6:i386 \
	libxau6:i386     \
	libxdmcp6:i386 \
	libxext6:i386 \
	libxft2:i386 \
	libxrender1:i386upgrade
```
There are some resources on the internet that seem to indicate that `libpng` is
required at some level, but that may depend upon whatever is already installed.
It's worth being mindful at this point that the actual executable that is in
need of service here is the `vish` binary, which is at something like
`/opt/intelFPGA/16.1/modelsim_ase/linuxaloem/vish`.  Note the following output
from the dynamic linker:

```console
$ ldd $(find . -type f -iname 'vish')
	linux-gate.so.1 (0xf7f56000)
	libpthread.so.0 => /lib/i386-linux-gnu/libpthread.so.0 (0xf7f15000)
	libdl.so.2 => /lib/i386-linux-gnu/libdl.so.2 (0xf7f0f000)
	libm.so.6 => /lib/i386-linux-gnu/libm.so.6 (0xf7e0b000)
	libX11.so.6 => /lib/i386-linux-gnu/libX11.so.6 (0xf7cb9000)
	libXext.so.6 => /lib/i386-linux-gnu/libXext.so.6 (0xf7ca3000)
	libXft.so.2 => /lib/i386-linux-gnu/libXft.so.2 (0xf7c8a000)
	libXrender.so.1 => /lib/i386-linux-gnu/libXrender.so.1 (0xf7c7e000)
	libfontconfig.so.1 => /lib/i386-linux-gnu/libfontconfig.so.1 (0xf7c31000)
	librt.so.1 => /lib/i386-linux-gnu/librt.so.1 (0xf7c25000)
	libncurses.so.5 => /lib/i386-linux-gnu/libncurses.so.5 (0xf7bfd000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7a12000)
	/lib/ld-linux.so.2 (0xf7f58000)
	libxcb.so.1 => /lib/i386-linux-gnu/libxcb.so.1 (0xf79e4000)
	libfreetype.so.6 => /lib/i386-linux-gnu/libfreetype.so.6 (0xf791d000)
	libexpat.so.1 => /lib/i386-linux-gnu/libexpat.so.1 (0xf78ef000)
	libuuid.so.1 => /lib/i386-linux-gnu/libuuid.so.1 (0xf78e5000)
	libtinfo.so.5 => /lib/i386-linux-gnu/libtinfo.so.5 (0xf78bf000)
	libXau.so.6 => /lib/i386-linux-gnu/libXau.so.6 (0xf78ba000)
	libXdmcp.so.6 => /lib/i386-linux-gnu/libXdmcp.so.6 (0xf78b3000)
	libpng16.so.16 => /lib/i386-linux-gnu/libpng16.so.16 (0xf7873000)
	libz.so.1 => /lib/i386-linux-gnu/libz.so.1 (0xf7855000)
	libbrotlidec.so.1 => /lib/i386-linux-gnu/libbrotlidec.so.1 (0xf7845000)
	libbsd.so.0 => /lib/i386-linux-gnu/libbsd.so.0 (0xf782d000)
	libbrotlicommon.so.1 => /lib/i386-linux-gnu/libbrotlicommon.so.1 (0xf780a000)
	libmd.so.0 => /lib/i386-linux-gnu/libmd.so.0 (0xf77fb000)
```
This deals with the 32-bit library problem.  Now let's solve the issue that is
generating that error message.  It's caused by the fact that the script assumes
some RedHat related symlinks exist for kernel versions past 3.x and that is most
definitely not us.  There is another reference to the `linux_rh60` string
elsewhere, but for now we're going to tell the `vco` script to run the version
in the `linux` directory and hope for the best.
```console
$ sudo cp -v /opt/intelFPGA/16.1/modelsim_ase/vco \
		/opt/intelFPGA/16.1/modelsim_ase/vco.original
$ sudo sed -i 's/linux_rh60/linux/g' /opt/intelFPGA/16.1/modelsim/vco
$ diff /opt/intelFPGA/16.1/modelsim_ase/vco /opt/intelFPGA/16.1/modelsim_ase/vco.original 
210c210
<           *)                vco="linux" ;;
---
>           *)                vco="linux_rh60" ;;
```
If your diff doesn't look like this, you've probably done something wrong.

Now when we run it, we get the following:
```console
$ MTI_VCO_MODE=32 /opt/intelFPGA/16.1/modelsim_ase/bin/vsim
Error in startup script: 
Initialization problem, exiting.

Initialization problem, exiting.

    while executing
"InitializeINIFile quietly"
    invoked from within
"ncFyP12 -+"
    (file "/mtitcl/vsim/vsim" line 1)
** Fatal: Read failure in vlm process (0,0)
```
Note that we have specified 32-bit libraries instead, since that is all that
this version of the tool supports.  This is entirely speculative, but the
internet has spoken and concluded that the lingering problem is that a different
flavor of [Freetype](https://sourceforge.net/projects/freetype/files/freetype2/2.4.12/)
library needs to be built and installed.

```console
$ wget https://sourceforge.net/projects/freetype/files/freetype2/2.4.12/freetype-2.4.12.tar.bz2
$ tar -xvjf freetype-2.4.12.tar.bz2
$ cd freetype-2.4.12
$ ./configure --build=i686-pc-linux-gnu "CFLAGS=-m32" "CXXFLAGS=-m32" "LDFLAGS=-m32"
$ make clean
$ make
```
Assuming that all worked, create a `lib32` directory in the ModelSim root
```console
$ sudo mkdir -v /opt/intelFPGA/16.1/modelsim_ase/lib32
```
And then from the Freetype source tree, install the newly created libraries into
that location
```console
$ sudo cp -v objs/.libs/libfreetype.so* /opt/intelFPGA/16.1/modelsim_ase/lib32/
'objs/.libs/libfreetype.so' -> '/opt/intelFPGA/16.1/modelsim_ase/lib32/libfreetype.so'
'objs/.libs/libfreetype.so.6' -> '/opt/intelFPGA/16.1/modelsim_ase/lib32/libfreetype.so.6'
'objs/.libs/libfreetype.so.6.10.1' -> '/opt/intelFPGA/16.1/modelsim_ase/lib32/libfreetype.so.6.10.1'
```
This creates the necessary library, but doesn't let the tool find them at
runtime.  Before continuing, we can check to make sure that everything is
installed by running something like this:
```console
$ LD_LIBRARY_PATH=/opt/intelFPGA/16.1/modelsim_ase/lib32 MTI_VCO_MODE=32 /opt/intelFPGA/16.1/modelsim_ase/bin/vsim 
```
You should see a message like *Reading pref.tcl* and a GUI window should pop up
with a welcome message.  Unless


