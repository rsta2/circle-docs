Getting started
---------------

To start using Circle, you need to download the project and a toolchain [#tc]_, configure Circle for your target platform, build the Circle libraries and your application [#ap]_, and install the built binary image (the kernel image) [#ki]_ on a SD card, along with a number of firmware files. In some cases an additional configuration file *config.txt* is needed on the SD card. The following notes require a x86_64 PC running Linux as development host. The file `doc/windows-build.txt <https://github.com/rsta2/circle/blob/master/doc/windows-build.txt>`_ describes, how Windows can be used instead.

Download
~~~~~~~~

The Circle project can be downloaded using *Git* as follows:

.. code-block:: shell

	cd /path/to/your/projects
	git clone https://github.com/rsta2/circle.git

The recommended toolchains for building Circle applications can be downloaded from `here <https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads>`_. Please note that there are different toolchains for 32-bit (AArch32, normally *arm-none-eabi-*) and for 64-bit (AArch64, normally *aarch64-none-elf-*) targets.

Configuration
~~~~~~~~~~~~~

Circle is configured using the file *Config.mk* in the project's root directory. This file can be created using the ``configure`` script, which provides these options:

.. code-block:: none

	-r <number>, --raspberrypi <number>
	                   Raspberry Pi model number (1, 2, 3, 4, default: 1)
	-p <string>, --prefix <string>
	                   Prefix of the toolchain commands (default: arm-none-eabi-)
	--multicore        Allow multi-core applications
	--realtime         Enable real time mode to improve IRQ latency
	--keymap <country> Set default USB keymap (DE, ES, FR, IT, UK, US)
	--qemu             Build for running under QEMU
	-d <option>, --define <option>
	                   Define additional system option
	--c++17            Use C++17 standard for compiling (default C++14)
	-f, --force        Overwrite existing Config.mk file
	-h, --help         Show usage message

If you want to configure Circle for a Raspberry Pi 3 with the default toolchain prefix ``arm-none-eabi-``, with the toolchain path in the system ``PATH`` variable, from Circle's project root enter simply:

.. code-block:: shell

	./configure -r 3

The file *Config.mk* can also be created by yourself. A typical 32-bit configuration looks like this:

.. code-block:: make

	PREFIX = /path/to/your/toolchain/bin/arm-none-eabi-
	AARCH = 32
	RASPPI = 3

This sets the path and name of your toolchain, and the architecture and model of your Raspberry Pi [#pi]_ computer.

.. note::

	The configurable system options, described in the file `include/circle/sysconfig.h <https://github.com/rsta2/circle/blob/master/include/circle/sysconfig.h>`_, can be defined there or in the *Config.mk* file, like that:

	``DEFINE += -DOPTION_NAME``

	System options, which are enabled by default, can be disabled with:

	``DEFINE += -DNO_OPTION_NAME``

A typical 64-bit configuration looks like that:

.. code-block:: make

	PREFIX64 = /path/to/your/toolchain/bin/aarch64-none-elf-
	AARCH = 64
	RASPPI = 3

64-bit operation is possible on the Raspberry Pi 3, 4 and Zero 2 only.

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

It is now always recommended to copy the file *config32.txt* (for 32-bit operation, AArch32) or *config64.txt* (for 64-bit operation, AArch64) from the *boot/* subdirectory to the SD card and to rename it to *config.txt* there.

If you want to use the FIQ on a Raspberry Pi 4, you need an additional Circle-specific ARM stub file (*armstub7-rpi4.bin* for 32-bit operation or *armstub8-rpi4.bin* for 64-bit operation), which will be loaded by the firmware. This ARM stub can be built in the *boot/* subdirectory. Please see `boot/README <https://github.com/rsta2/circle/blob/master/boot/README>`_ for information on how to build these files.

Put the SD card into your Raspberry Pi and power it on.

.. rubric:: Footnotes

.. [#tc] A toolchain in this context is cross compiler with additional tools and libraries, which runs on a specific platform and builds binaries for another (normally different) platform.
.. [#ap] For a start this can be one of the provided `sample programs <https://github.com/rsta2/circle/blob/master/sample/README>`_.
.. [#ki] Depending on the Raspberry Pi model and the target architecture (32- or 64-bit) a binary image has the filename *kernel.img*, *kernel7.img*, *kernel7l.img*, *kernel8.img* or *kernel8-rpi4.img*.
.. [#pi] For the Raspberry Pi Zero and Zero W the target ``RASPPI = 1`` has to be configured. The Raspberry Pi Zero 2 W requires the target ``RASPPI = 3``.
