Libraries
~~~~~~~~~

This appendix lists the libraries, which are provided by the Circle project.

Base libraries
^^^^^^^^^^^^^^

The base libraries will be built using ``./makeall`` from Circle's project root.

======================	==============================================	=================
Library lib/...		Description					Depends on lib
======================	==============================================	=================
libcircle.a		Basic system services and drivers
usb/libusb.a		USB host controller and class drivers		circle, input, fs
input/libinput.a	Generic input device services			circle
fs/libfs.a		Basic file system services (partition manager)	circle
fs/fat/libfatfs.a	FAT file system driver [#fs]_			circle, fs
sched/libsched.a	Cooperative multi-tasking support		circle
net/libnet.a		TCP/IP networking				circle, sched
======================	==============================================	=================

Add-on libraries
^^^^^^^^^^^^^^^^

Add-on libraries will be built using ``make`` from the target directory. This appendix lists only a subset of the available add-on libraries. All provided add-on modules are listed `here <https://github.com/rsta2/circle/blob/master/addon/README>`_.

==============================	=========================================
Library addon/...		Description
==============================	=========================================
SDCard/libsdcard.a		EMMC and SDHOST SD card drivers
fatfs/libfatfs.a		`FatFs file system module`_ [#fs]_
Properties/libproperties.a	Property file (.ini) support
linux/liblinuxemu.a		Linux kernel driver and pthread emulation
vc4/vchiq/libvchiq.a		VCHIQ interface driver
vc4/sound/libvchiqsound.a	VCHIQ (HDMI) sound driver
ugui/libugui.a			`uGUI graphics library`_
lvgl/liblvgl.a			`LVGL graphics library`_
==============================	=========================================

.. _FatFs file system module: http://elm-chan.org/fsw/ff/00index_e.html
.. _uGUI graphics library: http://embeddedlightning.com/ugui
.. _LVGL graphics library: https://lvgl.io

These libraries provide accelerated graphics support for the Raspberry Pi 1-3 and Zero (32-bit only) in *addon/vc4/interface/*:

* bcm_host/libbcm_host.a
* khronos/libkhrn_client.a
* vmcs_host/libvmcs_host.a
* vcos/libvcos.a

.. rubric:: Footnotes

.. [#fs] The file system support in the base libraries is restricted (no subdirectories, short file names). The FatFs file system module in *addon/fatfs/* provides full function support.
