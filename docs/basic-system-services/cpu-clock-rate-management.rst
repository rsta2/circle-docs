CPU clock rate management
~~~~~~~~~~~~~~~~~~~~~~~~~

Most Raspberry Pi models require a CPU clock rate management by the bare metal application to reach the maximum performance. This management continuously measures the current temperature of the CPU (actually the SoC) and regulates the clock rate of the ARM CPU, so that it is decreased, when the temperature is getting too high.

The absolute maximum of the allowed CPU temperature is 85 degrees Celsius. The firmware automatically ensures, that this limit is not exceeded. If the temperature comes near to this value, the firmware shows a warning icon in the upper right corner of the screen. Please read the `Frequency management and thermal control <https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#frequency-management-and-thermal-control>`_ documentation page to get more information on this.

The different Raspberry Pi models allow different maximum CPU clock rates and the the frequency of the ARM CPU, which is set after boot, is also different:

==============	======================	======================	======================
Raspberry Pi	Maximum CPU clock rate	Boot CPU clock rate	Remarks
==============	======================	======================	======================
1		700 MHz			700 MHz			No management required
2		900 MHz			600 MHz
Zero		1000 MHz		700 MHz
Zero 2		1000 MHz		600 MHz
3 Model B	1200 MHz		600 MHz
3 Model A+/B+	1400 MHz		600 MHz
4		1500 MHz		600 MHz
400		1800 MHz		600 MHz			Head sink included
==============	======================	======================	======================

Circle uses the class ``CCPUThrottle`` to implement a CPU clock rate management, which is described below. The sample program `26-cpustress` demonstrates its usage.

.. important::

	After boot the CPU clock rate of the ARM CPU is not set to the allowed maximum on most Raspberry Pi models. Without further action, the bare metal application will not operate with maximum performance.

	If you need the maximum performance at any time in your application and cannot handle, when the CPU is clocked down, you may need a head sink and/or fan installed.

	``CCPUThrottle`` should not be used together with code doing I2C or SPI transfers. Because clock rate changes to the CPU clock may also effect the CORE clock, this could result in changing transfer speeds.

CCPUThrottle
^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/cputhrottle.h>

.. cpp:class:: CCPUThrottle

.. cpp:function:: CCPUThrottle::CCPUThrottle (TCPUSpeed InitialSpeed = CPUSpeedUnknown)

	Creates the class ``CCPUThrottle``. ``InitialSpeed`` is the CPU speed to be set initially, with these possible values:

	* CPUSpeedLow
	* CPUSpeedMaximum
	* CPUSpeedUnknown

	If ``CPUSpeedUnknown`` is selected as initial speed and the parameter ``fast=true`` is set in the file `cmdline.txt <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt>`_, the resulting setting will be ``CPUSpeedMaximum``, or ``CPUSpeedLow`` if not set.

.. cpp:function:: static CCPUThrottle *CCPUThrottle::Get (void)

	Returns a pointer to the only ``CCPUThrottle`` object in the system (if any).

.. cpp:function:: boolean CCPUThrottle::IsDynamic (void) const

	Returns if CPU clock rate change is supported. Other Methods can be called in any case, but may be nop's or return invalid values, if ``IsDynamic()`` returns ``FALSE``.

.. cpp:function:: unsigned CCPUThrottle::GetClockRate (void) const

	Returns the current CPU clock rate in Hz or zero on failure.

.. cpp:function:: unsigned CCPUThrottle::GetMinClockRate (void) const

	Returns the minimum CPU clock rate in Hz.

.. cpp:function:: unsigned CCPUThrottle::GetMaxClockRate (void) const

	Returns the maximum CPU clock rate in Hz.

.. cpp:function:: unsigned CCPUThrottle::GetTemperature (void) const

	Returns the current CPU (SoC) temperature in degrees Celsius or zero on failure.

.. cpp:function:: unsigned CCPUThrottle::GetMaxTemperature (void) const

	Returns the maximum CPU (SoC) temperature in degrees Celsius.

.. cpp:function:: TCPUSpeed CCPUThrottle::SetSpeed (TCPUSpeed Speed, boolean bWait = TRUE)

	Sets the CPU speed. ``Speed`` selects the speed to be set and overwrites the initial value. Possible values are:

	* CPUSpeedLow
	* CPUSpeedMaximum

	``bWait`` must be ``TRUE`` to wait for new clock rate to settle before return. Returns the previous setting or ``CPUSpeedUnknown`` on error.

.. cpp:function:: boolean CCPUThrottle::SetOnTemperature (void)

	Sets the CPU speed depending on current SoC temperature. Call this repeatedly all 2 to 5 seconds to hold the temperature down! Throttles the CPU down when the SoC temperature reaches 60 degrees Celsius Returns ``TRUE`` if the operation was successful.

.. note::

	The default temperature limit of 60 degrees Celsius may be too small for continuous operation with maximum performance. The limit can be increased with the parameter ``socmaxtemp`` in the file `cmdline.txt <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt>`_.

.. cpp:function:: boolean CCPUThrottle::Update (void)

	Same function as ``SetOnTemperature()``, but can be called as often as you want, without checking the calling interval. Additionally checks for system throttled conditions, if a system throttled handler has been registered with ``RegisterSystemThrottledHandler()``. Returns ``TRUE`` if the operation was successful.

.. important::

	You have to repeatedly call ``SetOnTemperature()`` or ``Update()``, if you use this class!

.. cpp:function:: void CCPUThrottle::RegisterSystemThrottledHandler (unsigned StateMask, TSystemThrottledHandler *pHandler, void *pParam = 0)

	Registers the callback ``pHandler``, which is invoked from ``Update()``, when a system throttled condition occurs, which is given in ``StateMask``. ``pParam`` is any user parameter to be handed over to the callback function. ``StateMask`` can be composed from these bit masks by or'ing them together:

	* SystemStateUnderVoltageOccurred
	* SystemStateFrequencyCappingOccurred
	* SystemStateThrottlingOccurred
	* SystemStateSoftTempLimitOccurred

.. cpp:function:: void CCPUThrottle::DumpStatus (boolean bAll = TRUE)

	Dumps some information on the current CPU status to the :ref:`System log`. Set ``bAll`` to ``TRUE`` to dump all information. Only the current clock rate and temperature will be dumped otherwise.
