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

