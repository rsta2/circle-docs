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

Display support
^^^^^^^^^^^^^^^

The firmware support for frame buffer device(s) is not as comfortable on the Raspberry Pi 5 as on earlier models. Because Circle relies on this firmware support, there are limitations, when using HDMI displays (e.g. no configuration in *config.txt*, cannot set display resolution from application) and DSI displays (e.g. the Official 7" touchscreen) do not work at all.

Fan support
^^^^^^^^^^^

If you want to use the Case Fan or Active Cooler with the Raspberry Pi 5 and the class ``CCPUThrottle``, you have to add the option ``gpiofanpin=45`` to the file `cmdline.txt <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt>`_.

Serial devices
^^^^^^^^^^^^^^

The serial bootloader is supported via the dedicated UART connector. Please note that the UART connector is the serial device ``ttyS11`` (``nDevice = 10``), which is the default on the Raspberry Pi 5 for applications, which do not explicitly select a serial device number like all sample programs. This can be changed with the system option ``SERIAL_DEVICE_DEFAULT``. The serial device at GPIO14/15 has the device number 0 (``ttyS1``).

Sound devices
^^^^^^^^^^^^^

The only option to generate sound is currently via USB streaming.
