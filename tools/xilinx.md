# Xilinx
That's right, the Xilinx tools get an entire page all to themselves

# Installation
Installation is generally a pain because there does not seem to much recourse if
the installation hangs for any reasons.  Network outages, low bandwidth, VPN
issues, etc. can all cause the installer to hang and if it does, you generally
need to restart again.

I seem to have been doing this a lot lately, so we will go through the steps for
manually installing via SSH here. These steps are taken directly from the Vivado
Release Notes (UG973).

First, extract the installer (the web installer works). For this example, we'll
be using the 2019.1 

```bash
$ ./Xilinx_Vivado_SDK_Web_2019.1_0524_1430_Lin64.bin --keep --noexec --target unpacked
Creating directory unpacked/
Verifying archive integrity... All good.
Uncompressing Xilinx Installer........................
```
That directory should contain the `xsetup` script that installs the bloated
corpus that is the Vivado Design Suite. First, the installer
```bash
$ sudo xsetup -b ConfigGen
```
Select Vivado HL Design Edition from the text menu. This should generate a
configuration file in the root user home directory (usually someplace like
`/root/.Xilinx/install_config.txt`) which can be edited to determine
installation options, notably the installation directory which defaults to
something that virtually no one with any sense would want to use.  Change that
and then run the installation script again with these options
```bash
$ sudo ./xsetup --agree XilinxEULA,3rdPartyEULA,WebTalkTerms --batch Install --config /root/.Xilinx/install_config.txt
```
As with all things Xilinx, it varies from version to version of the tool, so
your mileage might vary - the licensing agreements provided as command line
arguments during the installation are known to change from version of the tool
to version.  At any rate, the installation should kick off using the arguments
provided in the installation configuration file.  Note the location of the log
file (again, usually someplace silly like `/root/.Xilinx/xinstall/xinstall_<timstamp>.log`).

If all goes according to plan, you should find the Vivado tools installed in
their respective location. If you're doing this via SSH, I highly recommend
backgrounding the process and then tailing the log file from a different
terminal.

# SDK
## 2019.1
Back when the land was green and things sort of made sense.  When trying to load
Xilinx SDK, an error of the following nature will be printed to the console
```
Launching SDK with command /opt/Xilinx/SDK/2019.1/eclipse/lnx64.o/eclipse -data /home/castillo/git-repos/microzed/proj/initial/initial.sdk -vmargs -Dcom.xilinx.sdk.args.hwspec=/home/castillo/git-repos/microzed/proj/initial/initial.sdk/uzed_wrapper.hdf -Xms64m -Xmx4G -Dorg.eclipse.swt.internal.gtk.cairoGraphics=false &
log4j:WARN No appenders could be found for logger (org.eclipse.jgit.util.FS).
log4j:WARN Please initialize the log4j system properly.
```
This occasionally causes Eclipse to fail when loading.  I haven't been able to
figure out exactly why it does so, but it appears that in the SDK directory
there is a `.metadata` directory that may contain a `.lock` file of some sort.
This may be left behind if SDK crashes or if the VM that it is in is reset
improperly.  It appears that deleting this file fixes the problem, but I haven't
extensively tested it yet.

### Adding existing files
This is so much more difficult than it needs to be. First, don't use their
stupid 'Hello world' application as a starter.  Start from a blank empty
application, and then start adding source code from there, because it's much
easier to have faith that the tools are actually doing something sensible than
if you rely on it to modify their existing crap code generator.

With an empty project, start by adding the source files by right clicking the
`/src` directory in the Project Explorer window.  If these are added as
filesystem links, you'll notice that they appear in the src directory as regular
files, but in the actual filesystem, there are no files in that directory. This
is because Eclipse apparently uses some sort of virtual filesystem that it
manages with workspaces and things of that nature (probably to make it easier to
support multiple operating systems like Windows that don't have a concept of
symlnks).  Ok, fair enough.

Now, to add additional files, like my own libraries without having to manually
copy files, create my own symlinks, etc. Do this by importing a file and then,
under the advanced settings, telling the tool to create virtual filesystem links
instead.  For header files, these need to be added to the include path by
selecting project Properties -> C/C++ General Settings -> Paths and Symbols ->
Includes.  When the smoke has cleared, you should have somethign like this:

```bash
$ tree
.
├── Debug
│   ├── makefile
│   ├── objects.mk
│   ├── sources.mk
│   ├── src
│   │   ├── hello.d
│   │   ├── hello.o
│   │   ├── ps7_dbg.d
│   │   ├── ps7_dbg.o
│   │   └── subdir.mk
│   ├── testing.elf
│   ├── testing.elf.size
│   └── Xilinx.spec
└── src
    ├── lscript.ld
    ├── README.txt
    └── Xilinx.spec
```
where we have a project names `testing` that contains C source files `hello.c`
and a `ps7_dbg.c` which are not in the source directory (because they're
elsewhere on the filesystem) yet the project can still find them.  Same with the
header files.

# Vivado
## 2022.1
Even with board files installed in the proper location, Vivado may be unwilling
to allow a user to select the desired board when creating a new project (the
'Next' button is greyed out) and progress beyond the 'Default Part' selection
menu.  The individual board still needs to be selected in the GUI and the
'Download' button clicked to install part specifics from the XHUB board store.
The following Tcl command
```tcl
xhub::install [xhub::get_xitems digilentinc.com:xilinx_board_store:arty-z7-20:1.1]
```
gets run behind the scenes and satisfies whatever it is that the tool seems to
be missing.  Depending upon board files is truly nuts, but with the
proliferation of MPSoC boards out there, it's not going to get better anytime
soon.

