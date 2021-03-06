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

For a first test create a subdirectory in the *app/* directory and save this program as *main.cpp* there. Furthermore you need the following *Makefile* in the same directory:

.. code-block:: make
	:caption: Makefile

	CIRCLEHOME = ../..

	OBJS	= main.o

	LIBS	= $(CIRCLEHOME)/lib/libcircle.a

	include $(CIRCLEHOME)/Rules.mk

	-include $(DEPS)

Now enter ``make`` in this directory and copy the resulting *kernel\*.img* file to the SD card. When you power on your Raspberry Pi, the green Activity LED should blink ten times. Then the system halts.

The CKernel class
~~~~~~~~~~~~~~~~~

Normally an application is not that simple and we should apply some structure to our program, which can be used for any Circle application. In C++ the means of abstraction is a class and we want to define our application's main class now. In Circle it is usually called ``CKernel``. It is a good practice to separate class definitions from its implementation, so we define the class in the header file *kernel.h*:

.. code-block:: c++
	:caption: kernel.h

	#ifndef _kernel_h
	#define _kernel_h

	//#include <circle/memory.h>
	#include <circle/actled.h>
	#include <circle/types.h>

	enum TShutdownMode
	{
		ShutdownNone,
		ShutdownHalt,
		ShutdownReboot
	};

	class CKernel
	{
	public:
		CKernel (void);
		~CKernel (void);

		boolean Initialize (void);

		TShutdownMode Run (void);

	private:
		//CMemorySystem	m_Memory;	// not needed any more
		CActLED		m_ActLED;
	};

	#endif

You should create a new subdirectory under *app/* and save this file there. Beside the class constructor ``CKernel()`` and destructor ``~CKernel()`` there are the methods ``Initialize()`` and ``Run()``. This implements a three step initialization for the class members, which is common throughout Circle:

1. The constructor ``CKernel()`` does some basic initialization for the class member variables.
2. The method ``Initialize()`` completes the initialization of the class members and returns ``TRUE``, if the initialization was successful.
3. The method ``Run()`` is entered to start the execution of the application. When it returns, the application halts or the system reboots, depending of the returned value of type ``TShutdownMode``. Many applications never return from ``Run()``.

.. note::

	Circle uses the type ``boolean`` with the possible values ``TRUE`` and ``FALSE`` for historical reasons. You can use ``bool``, ``true`` and ``false`` instead, which is equivalent.

.. note::

	Earlier Circle versions required a member of the class ``CMemorySystem`` in ``CKernel``, which initializes and manages the system memory. An instance of ``CMemorySystem`` is created now, before the function ``main()`` is called, so that there is no need to add it to ``CKernel`` any more. For compatibility ``CMemorySystem`` may still be instantiated in ``CKernel``, but this is deprecated.

A possible class implementation for ``CKernel``, with the same function as the "Hello world!" program before, looks as follows:

.. code-block:: c++
	:caption: kernel.cpp

	#include "kernel.h"
	#include <circle/timer.h>

	CKernel::CKernel (void)
	{
	}

	CKernel::~CKernel (void)
	{
	}

	boolean CKernel::Initialize (void)
	{
		return TRUE;
	}

	TShutdownMode CKernel::Run (void)
	{
		for (unsigned i = 1; i <= 10; i++)
		{
			m_ActLED.On ();
			CTimer::SimpleMsDelay (200);

			m_ActLED.Off ();
			CTimer::SimpleMsDelay (500);
		}

		return ShutdownHalt;
	}

The class constructor ``CKernel()`` and destructor ``~CKernel()`` and the method ``Initialize()`` are not really used here, but this will change in real applications. Please note, that the constructor of the member variable ``m_ActLED`` is implicitly called in ``CKernel()``. This call is automatically generated by the compiler.

Now that we have defined and implemented the class ``CKernel``, we still have to provide a ``main()`` function, which implements the three step procedure given above for our class. This can be done as follows:

.. code-block:: c++
	:caption: main.cpp

	#include "kernel.h"
	#include <circle/startup.h>

	int main (void)
	{
		CKernel Kernel;
		if (!Kernel.Initialize ())
		{
			halt ();
			return EXIT_HALT;
		}

		TShutdownMode ShutdownMode = Kernel.Run ();

		switch (ShutdownMode)
		{
		case ShutdownReboot:
			reboot ();
			return EXIT_REBOOT;

		case ShutdownHalt:
		default:
			halt ();
			return EXIT_HALT;
		}
	}

This *main.cpp* file is part of most Circle programs without changes.

.. note::

	Because some destructors used in ``CKernel`` may not be implemented, ``main()`` never really returns, but calls ``halt()`` or ``reboot()`` instead. Because we want to provide a common implementation of *main.cpp* here, we have to accept this little flaw here. In fact with the described ``CKernel`` implementation, it would be possible to return from ``main()``, but this need not be the case in other Circle applications.

Finally we have to add *kernel.o* to the *Makefile* listed above:

.. code-block:: make
	:caption: Makefile

	CIRCLEHOME = ../..

	OBJS	= main.o kernel.o

	LIBS	= $(CIRCLEHOME)/lib/libcircle.a

	include $(CIRCLEHOME)/Rules.mk

	-include $(DEPS)

That's all. Now we have the basic structure of a Circle application and you should be able to build it using ``make``.
