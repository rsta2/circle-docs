Time
~~~~

This section describes the services, which are provided by Circle regarding time. This concerns:

* The current non-consecutive system time (local and UTC) in seconds since 1970-01-01 00:00:00
* The timezone, expressed in minutes +/- from UTC
* The current consecutive system up-time in seconds
* A coarse grained, consecutive system tick counter in 1/100 seconds
* A fine grained, consecutive system tick counter in microseconds
* A possibly greater number of kernel timers, which elapse after a given number of coarse system ticks, resulting in a callback function to be executed
* Up to four periodic timer handlers, called 100 times per second
* Delaying program execution for a number of milli- or microseconds
* One fine grained periodic user timer, executing a callback function in an interval down to microseconds
* Converting time values (seconds since 1970-01-01 00:00:00) into time components (i.e. year, month, day, hour, minute, seconds) or reversed or into a string representation

These services are implemented in the classes ``CTimer``, ``CUserTimer`` and ``CTime``.

.. note::

	"Consecutive" in this context means, that the time does never "jump". For example the current local system time may be updated, while the system is running, from an external time source (e.g. NTP server). This may cause a step back or forward in time. Consecutive time sources ensure, that this does not happen, which is important, e.g. when a program waits for an amount of time to pass, by calculating the difference between the current time and a start time.

CTimer
^^^^^^

This class is the main provider of time services in Circle.

.. code-block:: c++

	#include <circle/timer.h>

.. cpp:class:: CTimer

There is exactly one or no instance of this class in the system. Only relatively simple programs can work without an instance of ``CTimer``. The only static timer functions, which can be called before this instance of ``CTimer`` is created and initialized, are:

.. cpp:function:: static unsigned CTimer::GetClockTicks (void)

	Returns the current value of the fine grained, consecutive system tick counter in microseconds. It does not necessarily start at zero and may overrun after a while. It continues to count from zero then.

.. c:macro:: CLOCKHZ

	Frequency of the fine grained, consecutive system tick counter (1000000).

.. cpp:function:: static void CTimer::SimpleMsDelay (unsigned nMilliSeconds)
.. cpp:function:: static void CTimer::SimpleusDelay (unsigned nMicroSeconds)

	Delay the program execution by the given amount of time.

Initialization
""""""""""""""

.. cpp:function:: CTimer::CTimer (CInterruptSystem *pInterruptSystem)

	Creates the instance of ``CTimer``.

.. cpp:function:: boolean CTimer::Initialize (void)

	Initializes and activates the system timer services. Returns ``TRUE`` on success.

.. note::

	``CTimer::Initialize()`` may generate log messages. Therefore it requires an initialized instance  of the class ``CLogger`` in the system.

	``CTimer::Initialize()`` determines the CPU speed by calibrating a delay loop by default. This can be suppressed with the system option ``NO_CALIBRATE_DELAY`` (e.g. to reduce boot time).

.. cpp:function:: static CTimer *CTimer::Get (void)

	Returns a pointer to the single instance of ``CTimer``.

Local time and UTC
""""""""""""""""""

.. cpp:function:: boolean CTimer::SetTimeZone (int nMinutesDiff)
.. cpp:function:: int CTimer::GetTimeZone (void) const

	Sets or returns the current timezone in minutes difference to UTC.

.. cpp:function:: boolean CTimer::SetTime (unsigned nTime, boolean bLocal = TRUE)

	Sets the current system time in seconds since 1970-01-01 00:00:00. The time is given according to the timezone by default or in UTC, if the parameter ``bLocal`` is FALSE. Returns ``TRUE``, if the time is valid.

.. cpp:function:: unsigned CTimer::GetTime (void) const
.. cpp:function:: unsigned CTimer::GetLocalTime (void) const
.. cpp:function:: boolean CTimer::GetLocalTime (unsigned *pSeconds, unsigned *pMicroSeconds)

	Returns the current local system time in seconds since 1970-01-01 00:00:00. The third variant always returns ``TRUE``.

.. cpp:function:: unsigned CTimer::GetUniversalTime (void) const
.. cpp:function:: boolean CTimer::GetUniversalTime (unsigned *pSeconds, unsigned *pMicroSeconds)

	Returns the current universal system time (UTC) in seconds since 1970-01-01 00:00:00. This value may be invalid, if the time was not set and the timezone difference is greater than zero. The third variant returns ``FALSE`` in this case.

.. cpp:function:: CString *CTimer::GetTimeString (void)

	Returns the current local system time as a string (format ``"[MMM dD ]HH:MM:SS.ss"``). Returns zero, when ``Initialize()`` has not been called yet. The resulting ``CString`` object must be deleted by the caller.

Coarse system tick and up-time
""""""""""""""""""""""""""""""

.. cpp:function:: unsigned CTimer::GetTicks (void) const

	Returns the current value of the coarse grained, consecutive system tick counter in 1/100 seconds units.

.. note::

	``CTimer::GetTicks()`` reads the ticks variable only and returns quickly. ``CTimer::GetClockTicks()`` reads a hardware register (on Raspberry Pi 1 and Zero) or has to do some calculations (in 64-bit mode). Therefore calling ``CTimer::GetTicks()`` does normally cost less CPU cycles. You should use ``CTimer::GetTicks()``, if its precision is sufficient for your purpose, or ``CTimer::GetClockTicks()`` otherwise.

.. c:macro:: HZ

	Frequency of the coarse grained, consecutive system tick counter (100).

.. cpp:function:: unsigned CTimer::GetUptime (void) const

	Returns the system up-time in seconds, since the class ``CTimer`` has been initialized.

Kernel timers
"""""""""""""

.. cpp:function:: TKernelTimerHandle CTimer::StartKernelTimer (unsigned nDelay, TKernelTimerHandler *pHandler, void *pParam = 0, void *pContext = 0)

	Start a kernel timer, which elapses after ``nDelay`` coarse system ticks (100 Hz). Call ``pHandler`` on elapse with the given values of ``pParam`` and ``pContext``. Returns a handle to the started timer. ``TKernelTimerHandler`` has the following prototype:

.. code-block:: c++

	void TKernelTimerHandler (TKernelTimerHandle hTimer, void *pParam, void *pContext);

.. c:macro:: MSEC2HZ(msecs)

	A macro, which converts milliseconds into coarse system ticks.

.. cpp:function:: void CTimer::CancelKernelTimer (TKernelTimerHandle hTimer)

	Cancel (remove) the kernel timer given with the handle ``hTimer``. It will not elapse any more.

Periodic timers
"""""""""""""""

.. cpp:function:: void CTimer::RegisterPeriodicHandler (TPeriodicTimerHandler *pHandler)

	Register a periodic timer handler, which is called ``HZ`` times (100) per second. Up to four handlers are allowed. ``TPeriodicTimerHandler`` has the following prototype:

.. code-block:: c++

	void TPeriodicTimerHandler (void);

Update time handler
"""""""""""""""""""

.. cpp:function:: void CTimer::RegisterUpdateTimeHandler (TUpdateTimeHandler *pHandler)

	Register a handler, which is called when ``SetTime()`` is invoked. This allows the application to apply additional checks, before the new time is set.

.. c:type:: boolean TUpdateTimeHandler (unsigned nNewTime, unsigned nOldTime)

	The handler gets the ``nNewTime`` to be set and the current ``nOldTime`` in seconds since 1970-01-01 00:00:00 UTC, and returns ``TRUE``, if the new time can be set or ``FALSE``, if the time is invalid. The call to ``SetTime()`` is ignored then.

Delay
"""""

.. cpp:function:: void CTimer::MsDelay (unsigned nMilliSeconds)
.. cpp:function:: void CTimer::usDelay (unsigned nMicroSeconds)

	Delay the program execution by the given amount of time. These functions should be used, when an instance of ``CTimer`` is available in the system (i.e. instead of ``SimpleMsDelay()`` and ``SimpleusDelay()``).

CUserTimer
^^^^^^^^^^

This class implements a fine grained, user programmable interrupt timer. It uses the system timer 1 hardware, which must not be used for other purposes in the application then.

.. code-block:: c++

	#include <circle/usertimer.h>

.. cpp:class:: CUserTimer

.. cpp:function:: CUserTimer::CUserTimer (CInterruptSystem *pInterruptSystem, TUserTimerHandler *pHandler, void *pParam = 0, boolean bUseFIQ = FALSE)

	Creates an instance of ``CUserTimer``. Only one is allowed. ``pHandler`` is a pointer to the callback function, which is executed, when the user timer elapses. By default the IRQ is used to trigger the interrupt. ``bUseFIQ`` has to be set to ``TRUE`` to use the FIQ instead (e.g. for high frequencies). ``TUserTimerHandler`` has this prototype:

.. code-block:: c++

	void TUserTimerHandler (CUserTimer *pUserTimer, void *pParam);

.. cpp:function:: boolean CUserTimer::Initialize (void)

	Initializes the user timer. Returns ``TRUE`` on success. Automatically starts the user timer with a delay of 1 hour.

.. cpp:function:: void CUserTimer::Stop (void)

	Stops the user timer. It has to be re-initialized to be used again.

.. cpp:function:: void CUserTimer::Start (unsigned nDelayMicros)

	(Re-)starts the user timer to elapse after the given number of microseconds (> 1). This method must be called from the user timer handler to a set new delay. It can be called on a running user timer to update the delay.

.. c:macro:: USER_CLOCKHZ

	Frequency of the user timer (1000000).

CTime
^^^^^

This class converts the time into different representations.

.. code-block:: c++

	#include <circle/time.h>

.. c:type:: time_t

	Time representation (normally) in seconds since 1970-01-01 00:00:00.

.. cpp:class:: CTime

.. cpp:function:: CTime::CTime (void)

	Creates an instance of ``CTime``.

.. cpp:function:: CTime::CTime (const CTime &rSource)

	Creates an instance of ``CTime`` from a different ``CTime`` object (copy constructor).

.. cpp:function:: void CTime::Set (time_t Time)

	Sets the time to the number seconds since 1970-01-01 00:00:00.

.. cpp:function:: boolean CTime::SetTime (unsigned nHours, unsigned nMinutes, unsigned nSeconds)

	Sets the time from its components hours (0-23), minutes (0-59) and seconds (0-59). Returns ``TRUE`` if the time is valid.

.. cpp:function:: boolean CTime::SetDate (unsigned nMonthDay, unsigned nMonth, unsigned nYear)

	Sets the date from its components month-day (1-31), month (1-12) and year (1970-). Returns ``TRUE`` if the date is valid.

.. cpp:function:: time_t CTime::Get (void) const

	Returns the time in the number seconds since 1970-01-01 00:00:00.

.. cpp:function:: unsigned CTime::GetSeconds (void) const
.. cpp:function:: unsigned CTime::GetMinutes (void) const
.. cpp:function:: unsigned CTime::GetHours (void) const
.. cpp:function:: unsigned CTime::GetMonthDay (void) const
.. cpp:function:: unsigned CTime::GetMonth (void) const
.. cpp:function:: unsigned CTime::GetYear (void) const

	Return the components of the time. See ``SetTime()`` and ``SetDate()`` for the possible value ranges.

.. cpp:function:: unsigned CTime::GetWeekDay (void) const

	Returns the weekday (0-6, Sun-Sat).

.. cpp:function:: const char *CTime::GetString (void)

	Returns a string representation of the time. The format is ``"WWW MMM DD HH:MM:SS YYYY"``, where "WWW" is the weekday.
