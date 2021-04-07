Debugging support
~~~~~~~~~~~~~~~~~

Circle provides a number of classes, functions and macros, which support the debugging of applications. This section describes the tools, which can be included in a program itself. Debugging a Circle application with an external debugger is described in `doc/debug-jtag.txt <https://github.com/rsta2/circle/blob/master/doc/debug-jtag.txt>`_ and `doc/debug.txt <https://github.com/rsta2/circle/blob/master/doc/debug.txt>`_ in the Circle repository.

.. note::

	Beside the tools, which are described here, you can also use the :ref:`System log` to write debug messages to the screen or serial interface.

Assertions
^^^^^^^^^^

Assertions are a common technique to insert runtime checks into the code. This is frequently used in the Circle libraries itself and is also recommended for application code. Assertions will be included in a checked build of Circle only and are ignored, when the macro symbol ``NDEBUG`` is defined.

.. code-block:: c

	#include <assert.h>

.. c:macro:: assert(expr)

	Inserts a runtime check into the code. ``expr`` must be a true boolean expression, otherwise the system is halted with an "Assertion failed" message, which contains the filename and the source code line of the failed assertion, and with a stack trace.

.. c:macro:: ASSERT_STATIC(expr)

	This is a static assertion, which will be evaluated at build time. It will be placed outside of a function, e.g. to check the size of a structure definition. The compiler generates an error message, if the expression ``expr`` is false.

Functions
^^^^^^^^^

The following functions are only available, when ``NDEBUG`` is not defined.

.. code-block:: c

	#include <circle/debug.h>

.. c:function:: void debug_hexdump (const void *pStart, unsigned nBytes, const char *pSource = 0)

	Writes a hexdump of ``nBytes``, starting at ``pStart`` to the :ref:`System log`. ``pSource`` is used as prefix of the log messages ("debug" if omitted).

.. c:function:: void debug_stacktrace (const uintptr *pStackPtr, const char *pSource = 0)

	Writes a stack trace to the :ref:`System log`. This function tests 64 numbers starting at ``pStackPtr``, if they point into the program code and logs them in this case.

.. c:function:: void debug_click (unsigned nMask = DEBUG_CLICK_ALL)

	This function can be used to debug events, which occur frequently, so that writing a log message would destroy the timing of the system. The function generates an audio click, which can be heard via the headphone jack of the Raspberry Pi. Frequent events generate a tone, very frequent events may generate a frequency, which is not hear-able. ``nMask`` can be ``DEBUG_CLICK_LEFT``, ``DEBUG_CLICK_RIGHT`` or ``DEBUG_CLICK_ALL`` and selects the audio channel to be used. On some Raspberry Pi models these channels may be swapped.

.. note::

	The macro ``DEBUG_CLICK`` must be defined, when you want to use ``debug_click()``. The PWM audio device cannot be used in this case.

CExceptionHandler
^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/exceptionhandler.h>

.. cpp:class:: CExceptionHandler

	This class handles abort exceptions, which occur on different program errors. The exception handler displays a stack trace and logs some important register values. An instance of this class should be added to each more complex program, which includes a ``CLogger`` instance too. Usually it will be added as a member to ``CKernel``. This class does not have methods, which can be called from application code.

CTracer
^^^^^^^

.. code-block:: cpp

	#include <circle/tracer.h>

.. cpp:class:: CTracer

	This class can be used to trace the program execution, without changing the timing too much. The class maintains a ring buffer, which is filled with trace events and dumped later, when the execution of the critical program parts has been completed.

.. cpp:function:: CTracer::CTracer (unsigned nDepth, boolean bStopIfFull)

	Creates an instance of this class. ``nDepth`` is the size of the ring buffer in number of events. If ``bStopIfFull`` is ``TRUE``, the tracing stops automatically, when the ring buffer is full. Otherwise a new event overwrites the oldest event.

.. cpp:function:: void CTracer::Start (void)

	Starts the tracing and the tracing clock. Arriving events will be written to the ring buffer now.

.. cpp:function:: void CTracer::Stop (void)

	Stops the tracing. If an event arrives afterwards, it is ignored.

.. cpp:function:: void CTracer::Event (unsigned nID, unsigned nParam1 = 0, unsigned nParam2 = 0, unsigned nParam3 = 0, unsigned nParam4 = 0)

	Sends an event to the tracer. Insert this into your program code, where something important happens to catch an issue. ``nID`` is any number, except 0, which is the stop event. ``nParamN`` is any parameter of the event. This method is not reentrant. You have to use a spin lock, if ``Event()`` may be called concurrently.

.. cpp:function:: void CTracer::Dump (void)

	Writes the entire tracing buffer to the :ref:`System log`. If the tracing was not stopped before, it is stopped automatically before the dump.

.. cpp:function:: static CTracer *CTracer::Get (void)

	Returns a pointer to the ``CTracer`` object.

CLatencyTester
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/latencytester.h>

.. cpp:class:: CLatencyTester

	This class can be used to measure the IRQ latency of the running code. The class continuously triggers an IRQ and measures the delay between the time, the IRQ was triggered and the time, the IRQ handler is called. This delay can be important for real-time applications. This is demonstrated in the sample program `40-irqlatency`.

.. note::

	The class ``CLatencyTester`` blocks the system timer 1, which is used by the class ``CUserTimer`` too. You can use only one of both classes at a time.

.. cpp:function:: CLatencyTester::CLatencyTester (CInterruptSystem *pInterruptSystem)

	Creates a ``CLatencyTester`` object. ``pInterruptSystem`` is a pointer to the interrupt system object.

.. cpp:function:: void CLatencyTester::Start (unsigned nSampleRateHZ)

	Starts the measurement. ``nSampleRateHZ`` is the sample rate in Hz.

.. cpp:function:: void CLatencyTester::Stop (void)

	Stops the measurement.

.. cpp:function:: unsigned CLatencyTester::GetMin (void) const

	Returns the minimum IRQ latency in microseconds. Can be called, while the test is running.

.. cpp:function:: unsigned CLatencyTester::GetMax (void) const

	Returns the maximum IRQ latency in microseconds. Usually this is the most interesting value. Can be called, while the test is running.

.. cpp:function:: unsigned CLatencyTester::GetAvg (void)

	Returns the average IRQ latency in microseconds. Can be called, while the test is running. Please note that the accumulated IRQ latency may overrun after some time. This method will return 0xFFFFFFFFU then.

.. cpp:function:: void CLatencyTester::Dump (void)

	Writes the results to the :ref:`System log`.
