.. _System log:

System log
~~~~~~~~~~

Circle uses a system log facility throughout the system to report status information from other system facilities or devices to the user. It is recommended to use this log facility from application code too and this is implemented in all Circle sample programs. While an application may write information messages directly to a device, the log facility provides additional services and is a standard tool for collecting status information in Circle. The system log is implemented by the class ``CLogger``.

CLogger
^^^^^^^

.. code-block:: c++

	#include <circle/logger.h>

.. cpp:class:: CLogger

There is exactly one or no instance of ``CLogger`` in the system. Only relatively simple programs can work without an instance of ``CLogger``.

Initialization
""""""""""""""

.. cpp:function:: CLogger::CLogger (unsigned nLogLevel, CTimer *pTimer = 0)

	Creates the instance of ``CLogger``. ``nLogLevel`` (0-4) determines, which log messages are included in the system log. Only messages with a log level smaller or equal to ``nLogLevel`` are considered. ``pTimer`` is a pointer to the system timer object. The time is not logged, if ``pTimer`` is zero. The following log levels are defined:

======	==============	===============================================================
Level	Severity	Description
======	==============	===============================================================
0	LogPanic	Halt the system after processing this message
1	LogError	Severe error in this component, system may continue to work
2	LogWarning	Non-severe problem, component continues to work
3	LogNotice	Informative message, which is interesting for the system user
4	LogDebug	Message, which is only interesting for debugging this component
======	==============	===============================================================

.. cpp:function:: boolean CLogger::Initialize (CDevice *pTarget)

	Initializes the system log facility. Returns ``TRUE`` on success. ``pTarget`` is a pointer to the device, to which the log messages will be written.

.. cpp:function:: void CLogger::SetNewTarget (CDevice *pTarget)

	Sets the target for the log messages to a new device.

.. cpp:function:: static CLogger *CLogger::Get (void)

	Returns a pointer to the only instance of ``CLogger``.

Write the log
"""""""""""""

.. cpp:function:: void CLogger::Write (const char *pSource, TLogSeverity Severity, const char *pMessage, ...)

	Writes a message from the module ``pSource`` with ``Severity`` (see table above) to the log. The message can be composed using format specifiers as supported by :ref:`CString`::Format().

.. cpp:function:: void CLogger::WriteV (const char *pSource, TLogSeverity Severity, const char *pMessage, va_list Args)

	Same function as ``Write()``, but the message parameters are given as ``va_list``.

Read the log
""""""""""""

``CLogger`` has a 16K buffer, which saves the written log messages. If this buffer is full, old messages will be overwritten.

.. cpp:function:: int CLogger::Read (void *pBuffer, unsigned nCount)

	Reads and deletes maximal ``nCount`` characters from the log buffer. The read characters will be returned in ``pBuffer``. Returns the number of characters actually read.

.. cpp:function:: boolean CLogger::ReadEvent (TLogSeverity *pSeverity, char *pSource, char *pMessage, time_t *pTime, unsigned *pHundredthTime, int *pTimeZone)

	Returns the next log event (message) from a log event queue with maximal 50 entries or ``FALSE``, if the queue is empty. The buffers at ``pSource`` and ``pMessage`` must have the sizes ``LOG_MAX_SOURCE`` and ``LOG_MAX_MESSAGE``. This queue is normally used by the class ``CSysLogDaemon``, which sends log messages to a syslog server.

Log event notification
""""""""""""""""""""""

.. cpp:function:: void CLogger::RegisterEventNotificationHandler (TLogEventNotificationHandler *pHandler)

	Registers a callback function, which is executed, when a log event (message) arrives. This is normally used by the class ``CSysLogDaemon``, which sends log messages to a syslog server. ``TLogEventNotificationHandler`` has the following prototype:

.. code-block:: c++

	void TLogEventNotificationHandler (void);

.. cpp:function:: void CLogger::RegisterPanicHandler (TLogPanicHandler *pHandler)

	Registers a callback function, which is executed, before a system halt, which is triggered by a log message with severity ``LogPanic``. This is normally used by the class ``CSysLogDaemon``, which sends log messages to a syslog server. If ``CSysLogDaemon`` is not in the system, ``RegisterPanicHandler()`` can be used for other application purposes. ``TLogPanicHandler`` has to following prototype:

.. code-block:: c++

	void TLogPanicHandler (void);