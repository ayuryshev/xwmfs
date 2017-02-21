INTRODUCTION
============

This is xwmfs (X window manager file system), a userspace filesystem based on
fuse that allows interaction with an EWMH compliant X11 window manager
via files.

Some of its features are:

- newly appearing and disappearing windows in the X server are recognized and
  the file system is updated in an event based manner
- new values for properties of window manager and windows will be reflected in
  the file system in an event based manner
- properties of windows and window manager can be changed via writing to files
  in the file system
- some X operations are accessible via control files in the file system

Thef file system can be used for easily implementing scripts that operate on
the window manager and windows (for example arranging them in a specific
order, moving them around etc).

For build and installation instructions see INSTALL.

EWMH is a specification of how a window manager makes information about the
current state of itself and the windows it manages available to other
programs. Not all window managers support all features of this specification
and not all support all features correctly. The following window managers have
been basically tested:

- fluxbox
- i3 (does not support the desktop property per window)
- compiz
- xfce (window names can't be changed)
- enlightenment (window names can't be changed, many global wm attributes are
  not supported)
- iceWM (window names can't be changed)
- kde
- gnome

xwmfs is currently in a beta release status. Thus it is far from feature
complete, not well or broadly tested and thus not necessarily very stable.

USAGE
=====

xwmfs consists of a single executable named 'xwmfs'. Currently no configuration
files of any kind are used. The xwmfs program behaves like a standard fuse
program. This means a lot of standard command line options are provided that
influence the behaviour of the file system in general and are not specific to
the xwmfs software.

To mount the file system with default options you simply run xwmfs like this:

	xwmfs [-f] <mount-location>

The switch -f causes the program to run in the foreground, because by default
it will daemonize into the background. Running it in the foreground can be
helpful for testing and for getting log messages.

The <mount-location> can be any suitable location in the file system where the
xwmfs file system will be mounted on. The calling user needs to have write
permissions for the mount-location as is the case with any regular mount of
file systems.

Because the xwmfs acts as an X11 client on the X server it requires to have a
valid DISPLAY environment variable set and the authority to access the X11
display. Otherwise the xwmfs program will print an error and exit immediately.

To unmount the xwmfs file system the command

	fusermount -u <mount-location>

can be used.

FILE SYSTEM STRUCTURE
=====================

Once mounted the xwmfs file system presents the following hierarchy:

<pre>
-windows: A subdirectory containing further directories that represent all
 |        windows managed by the window manager on the current display.
 |
 |--------> <ID>: A directory representing a single window managed by the
 |       |        window manager with the given ID. The ID is unique per window
 |       |        and can also be used to match information from other tools
 |       |        like 'xprop'
 |       |--------> id: A virtual file that again contains the <ID> of the
 |       |              window it represents
 |       |--------> name: A virtual file that contains the user visible name or
 |       |                title of the window. You can also write to this file
 |       |                for giving the window a new name.
 |       |--------> desktop: The number of the desktop the window is currently
 |       |                   located on. Counting starts at zero. You can also
 |       |                   write a new desktop number to this file to move
 |       |                   the window to another desktop.
 |       |--------> pid: The PID of the process that owns the window
 |       |--------> command: Can be used to send a command to the window. When
 |       |                   read then a list of valid commands is returned.
 |       |                   Currently "destroy" or "delete" to force or ask
 |       |                   the window to be closed.
 |
-wm: A directory containing global state information about the
 |   window manager
 |
 |--------> name: Contains the name of the running window manager.
 |--------> class: May contain an identifier that defines the kind of
 |                 window manager that is running
 |--------> active_desktop: Contains the number of the currently active
 |                          desktop, if virtual desktops are available
 |                          and configured. Counting starts at zero.
 |--------> number_of_desktops: Contains the number of virtual desktops
 |                              current configured, if supported
 |--------> show_desktop_mode: Returns a 0/1 value whether currently the
 |                             "show desktop mode" is active. This means
 |                             that all windows are hidden from view and
 |                             desktop icons and background are shown
 |                             instead
 |--------> pid: Contains the PID of the running window manager process

Should the window manager not support some of the properties like
"show_desktop_mode" then a value of -1 is contained in the file if the file
represents an integer value or the value "N/A" if the file represents a string
value.
</pre>

CURRENT STATE OF DEVELOPMENT
============================

xwmfs conforms to the EWMH and ICCCM specifications regarding the name, type
and meaning of X window properties used for representing the state of window
managers.

Not all window managers implement these specifications. Some window
managers only implement part of them. Some don't exactly conform to them. Thus
xwmfs would require to adapt to different types and versions of window managers
to work around some of these limitations.

This is not currently the case. This means if anything unexcepted happens then
xwmfs might behave strangely or simply not be able to provide all features.

xwmfs is able to detect changes to windows (windows appearing and disappearing
or changing properties) in an event based way. This feature has not been
thoroughly tested yet. The details of the X client protocol regarding this are
not too well documented and a lot of research and experimentation was
necessary to get things as far as they are now.

A good source of information about how things can be done is the 'wmctrl'
program that allows to query and alter some state of X11 windows.

TIPS AND TRICKS
===============

- You can get the window ID of the current window in most terminal emulators
  from the $WINDOWID environment variable

FUTURE DIRECTIONS
=================

The feature set I initially intended for xwmfs is roughly the following:

- high stability in all situations
- coverage of all major and popular window managers, if necessary using a set
  of proprietary adaptions to the code
- lots of more contextual information in the file system like:
	* position and size of windows
	* visibility status of windows (raised/lowered)
	* possibility to resize, move, lower/raise windows and influence
	  other properties of them
	* possibility to get and change names of virtual desktops as
	  applicable
	* possibility to wait for events on specific windows or the window
	  manager in the file system