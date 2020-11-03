Getting started
---------------

To start using Circle, you need to download the project and a toolchain [#tc]_, configure Circle for your target platform, build the Circle libraries and your application [#ap]_, and install the built binary image (the kernel image) [#ki]_ on a SD card, along with a number of firmware files. In some cases an additional configuration file *config.txt* is needed on the SD card. The following notes require a x86_64 PC running Linux as development host.

Download
~~~~~~~~

The Circle project can be downloaded using *Git* as follows:

.. code-block:: shell

	cd /path/to/your/projects
	git clone https://github.com/rsta2/circle.git

The recommended toolchains for building Circle applications can be downloaded from `here <https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads>`_. Please note that there are different toolchains for 32-bit (AArch32, normally *arm-none-eabi-*) and for 64-bit (AArch64, normally *aarch64-none-elf-*) targets.

Configuration
~~~~~~~~~~~~~

Circle is configured using the file *Config.mk* in the project's root directory. This file has to be created by yourself. A typical 32-bit configuration looks like that:

.. code-block:: make

	PREFIX = /path/to/your/toolchain/bin/arm-none-eabi-
	AARCH = 32
	RASPPI = 3
	DEFINE += -DREALTIME
	FLASHBAUD = 921600

This sets the path and name of your toolchain, the architecture and model of your Raspberry Pi [#pi]_ computer, defines a system option (optional) and sets the baud rate for the serial bootloader (optional).

.. note::

	The configurable system options are described in the file `include/circle/sysconfig.h <https://github.com/rsta2/circle/blob/master/include/circle/sysconfig.h>`_. They can be defined there or in the *Config.mk* file.

A typical 64-bit configuration looks like that:

.. code-block:: make

	PREFIX64 = /path/to/your/toolchain/bin/aarch64-none-elf-
	AARCH = 64
	RASPPI = 3

64-bit operation is possible on the Raspberry Pi 3 and 4 only. The optional settings (see above) have been omitted here, but are possible too.

Building
~~~~~~~~

After configuring Circle, go to the root directory of the Circle project and enter:

.. code-block:: shell

	./makeall clean
	./makeall

By default only the latest sample (with the highest number) is build. The ready build kernel image file should be in its subdirectory of *sample/*. If you want to build another sample after ``./makeall`` go to its subdirectory and do ``make``.


Installation
~~~~~~~~~~~~

Copy the Raspberry Pi firmware (from *boot/* subdirectory, do ``make`` there to get them) files along with the *kernel\*.img* (from *sample/* subdirectory) to a SD card with FAT file system.

The *config32.txt* file, provided in the *boot/* subdirectory, is needed to enable FIQ use in 32-bit mode on the Raspberry Pi 4 and has to be copied to the SD card in this case (rename it to *config.txt*). Furthermore the additional file *armstub7-rpi4.bin* is required on the SD card then. Please see `boot/README <https://github.com/rsta2/circle/blob/master/boot/README>`_ for information on how to build this file.

The *config64.txt* file, provided in the *boot/* directory, is needed to enable 64-bit mode and has to be copied to the SD card in this case (rename it to *config.txt*). FIQ support for 64-bit mode on the Raspberry Pi 4 requires an additional file *armstub8-rpi4.bin* on the SD card. Please see `boot/README <https://github.com/rsta2/circle/blob/master/boot/README>`_ for information on how to build this file.

Put the SD card into your Raspberry Pi and power it on.

.. rubric:: Footnotes

.. [#tc] A toolchain in this context is cross compiler with additional tools and libraries, which runs on a specific platform and builds binaries for another (normally different) platform.
.. [#ap] For a start this can be one of the provided `sample programs <https://github.com/rsta2/circle/blob/master/sample/README>`_.
.. [#ki] Depending on the Raspberry Pi model and the target architecture (32- or 64-bit) a binary image has the filename *kernel.img*, *kernel7.img*, *kernel7l.img*, *kernel8.img* or *kernel8-rpi4.img*.
.. [#pi] For the Raspberry Pi Zero and Zero W the target ``RASPPI = 1`` has to be configured.
