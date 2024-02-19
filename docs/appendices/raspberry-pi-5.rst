Raspberry Pi 5
~~~~~~~~~~~~~~

This appendix gives information about the Raspberry Pi 5 support in Circle.

Currently is only a subset of features supported on the Raspberry Pi 5. See the main project `README <https://github.com/rsta2/circle/blob/master/README.md#features>`_ for details!

The Cortex-A76 CPU of the Raspberry Pi 5 supports only AArch64 in the mode, which is used by Circle (EL1), so AArch32 is and will not be supported.

Installing
^^^^^^^^^^

The resulting kernel image file has the name *kernel_2712.img*. Additionally the following files are required on the SD card:

* *bcm2712-rpi-5-b.dtb* (can be downloaded in *boot/*)
* *config.txt* (copy *boot/config64.txt* and rename it)

RP1 southbridge
^^^^^^^^^^^^^^^

The Raspberry Pi 5 features the RP1 southbridge, which provides many of the available peripherals. The RP1 peripherals are accessible automatically on entry into ``main()`` in Circle applications. No specific action is necessary for this purpose.
