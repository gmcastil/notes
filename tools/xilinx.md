# Xilinx
That's right, the Xilinx tools get an entire page all to themselves

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

