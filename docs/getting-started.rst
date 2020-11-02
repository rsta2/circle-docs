Getting started
---------------

Download
~~~~~~~~

The Circle project can be downloaded using *Git* as follows:

.. code-block:: shell

	cd /path/to/your/projects
	git clone https://github.com/rsta2/circle.git

Configuration
~~~~~~~~~~~~~

Circle is configured using the file *Config.mk* in the project's root directory. A typical configuration looks like that:

.. code-block:: make

	PREFIX = /path/to/your/toolchain/bin/arm-none-eabi-
	AARCH = 32
	RASPPI = 3
	DEFINE += -DREALTIME
	FLASHBAUD = 921600

This sets the path and name of your toolchain, the architecture and model of your Raspberry Pi computer, defines a system option (optional) and sets the baud rate for the serial bootloader.

.. note::

	The configurable system options are described in the file `include/circle/sysconfig.h <https://github.com/rsta2/circle/blob/master/include/circle/sysconfig.h>`_. They can be defined there or in the *Config.mk* file.

Installation
~~~~~~~~~~~~

Building and installation are described in the project's main `README <https://github.com/rsta2/circle/blob/master/README.md#building>`_ file.

One of the provided sample programs can be built as follows:

.. code-block:: shell

	cd sample/04-timer
	make

Hello world!
~~~~~~~~~~~~

A simple Circle program looks like that:

.. code-block:: c++
	:caption: main.cpp

	#include <circle/startup.h>
	#include <circle/actled.h>
	#include <circle/timer.h>

	int main (void)
	{
		CActLED ActLED;

		for (unsigned i = 1; i <= 10; i++)
		{
			m_ActLED.On ();
			CTimer::SimpleMsDelay (200);

			m_ActLED.Off ();
			CTimer::SimpleMsDelay (500);
		}

		return EXIT_HALT;
	}

For a first test, create a sub-directory in the *app/* directory and save this program as *main.cpp* there. Furthermore you need the following *Makefile* in the same directory:

.. code-block:: make
	:caption: Makefile

	CIRCLEHOME = ../..

	OBJS	= main.o

	LIBS	= $(CIRCLEHOME)/lib/libcircle.a

	include $(CIRCLEHOME)/Rules.mk

	-include $(DEPS)

Now enter ``make`` in this directory and copy the resulting *kernel\*.img* file to the SD card. When you power on your Raspberry Pi, the green Activity LED should blink ten times. Then the system halts.
