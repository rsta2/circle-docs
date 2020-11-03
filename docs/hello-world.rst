Hello world!
------------

Now we want to start developing our first Circle program. It may look like this:

.. code-block:: c++
	:caption: main.cpp

	#include <circle/startup.h>	// for EXIT_HALT
	#include <circle/actled.h>
	#include <circle/timer.h>

	int main (void)
	{
		CActLED ActLED;

		for (unsigned i = 1; i <= 10; i++)
		{
			ActLED.On ();
			CTimer::SimpleMsDelay (200);

			ActLED.Off ();
			CTimer::SimpleMsDelay (500);
		}

		return EXIT_HALT;
	}

The program should be self-explanatory. ``CTimer::SimpleMsDelay()`` is a static delay function, which can be used, when there is no instance of the class ``CTimer`` in the system.

For a first test, create a subdirectory in the *app/* directory and save this program as *main.cpp* there. Furthermore you need the following *Makefile* in the same directory:

.. code-block:: make
	:caption: Makefile

	CIRCLEHOME = ../..

	OBJS	= main.o

	LIBS	= $(CIRCLEHOME)/lib/libcircle.a

	include $(CIRCLEHOME)/Rules.mk

	-include $(DEPS)

Now enter ``make`` in this directory and copy the resulting *kernel\*.img* file to the SD card. When you power on your Raspberry Pi, the green Activity LED should blink ten times. Then the system halts.
