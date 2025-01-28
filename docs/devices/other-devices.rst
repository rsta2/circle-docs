Other devices
~~~~~~~~~~~~~

This section covers some device driver classes, which do not belong to other groups of devices. These classes have their own interface and are not derived from the class :cpp:class:`CDevice`.

CBcmRandomNumberGenerator
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmrandom.h>

.. cpp:class:: CBcmRandomNumberGenerator

	This class is a driver for the built-in hardware random number generator.

.. cpp:function:: u32 CBcmRandomNumberGenerator::GetNumber (void)

	Returns a 32-bit random number.

.. note::

	Generating a random number takes a short while. For generating a large number of random numbers, you should use a polynomial random number generator, and seed it using this hardware random number generator.

CBcmWatchdog
^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmwatchdog.h>

.. cpp:class:: CBcmWatchdog

	This class is a driver for the built-in watchdog device. It can be used to automatically restart a Raspberry Pi computer after program failure, or to restart it immediately from a specific partition.

.. cpp:function:: void CBcmWatchdog::Start (unsigned nTimeoutSeconds = MaxTimeoutSeconds)

	Starts the watchdog, to elapse after ``nTimeoutSeconds`` seconds. The system restarts after this timeout, if the watchdog is not re-triggered before.

.. cpp:var:: const unsigned CBcmWatchdog::MaxTimeoutSeconds = 15

	Is the maximum timeout in seconds.

.. cpp:function:: void CBcmWatchdog::Stop (void)

	Stops the watchdog. It will not elapse any more.

.. cpp:function:: void CBcmWatchdog::Restart (unsigned nPartition = PartitionDefault)

	Immediately restarts the system from the SD card partition with the number ``nPartition``, with these special values:

.. cpp:var:: const unsigned CBcmWatchdog::PartitionDefault = 0
.. cpp:var:: const unsigned CBcmWatchdog::PartitionHalt = 63

	``PartitionHalt`` halts the system, instead of restarting it.

.. cpp:function:: boolean CBcmWatchdog::IsRunning (void) const

	Returns ``TRUE``, if the watchdog is currently running.

.. cpp:function:: unsigned CBcmWatchdog::GetTimeLeft (void) const

	Returns the number of seconds left, until a restart will triggered.
