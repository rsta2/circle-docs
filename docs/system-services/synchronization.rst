.. _synchronization:

Synchronization
~~~~~~~~~~~~~~~

This section discusses the different system execution levels of code inside a Circle application and how they can be synchronized. Furthermore the class ``CSpinLock`` will be introduced, which is the main synchronization object in multi-core environments, but also in single-core environments, because all Circle code should be prepared to run on multiple cores, at least where it is possible.

Execution levels
^^^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/synchronize.h>

The current execution level is determined by the type of interrupt requests, which are enabled (i.e. allowed to occur) or active (i.e. currently handled) at a given time. Circle defines the following execution levels:

==============	======================================	==================
Level [#lv]_	Currently running			Enabled interrupts
==============	======================================	==================
TASK_LEVEL	normal application code or task [#mt]_	IRQ, FIQ
IRQ_LEVEL	IRQ handler or callback [#iq]_		FIQ
FIQ_LEVEL	FIQ handler
==============	======================================	==================

Interrupt requests of the same type (i.e. IRQ or FIQ) cannot be nested. That means, when for example an IRQ handler is running for one device, a triggered IRQ of another device has to wait for the execution of its IRQ handler, until the previous IRQ handler has been completed.

The execution level (e.g. ``TASK_LEVEL``) of the currently running code is returned by the following function:

.. cpp:function:: unsigned CurrentExecutionLevel (void)

The current execution level can be explicitly raised with this function:

.. cpp:function:: void EnterCritical (unsigned nTargetLevel = IRQ_LEVEL)

``EnterCritical()`` can be called with the same as the current execution level or with a higher level, but not with a lower one. Reducing the current execution level is possible with this function:

.. cpp:function:: void LeaveCritical (void)

In summary ``EnterCritical()`` is called to enter a critical code region, which must not be interrupted by an IRQ, or by both IRQ and FIQ, depending on the target level. This critical region will be left with ``LeaveCritical()``. Calls to ``EnterCritical()`` can be nested with the same or increasing target level. Every ``EnterCritical()`` has its corresponding ``LeaveCritical()``.

.. important::

	In a multi-core environment using ``EnterCritical()`` for synchronization (e.g. protecting data structures in a critical region) is not recommended or does not work at all. You should use spin locks (see below) instead. Furthermore, because Circle source code should be able to run in any environment, where possible, it is good practice to use spin locks also for code, which is developed for a single-core environment. If the system option ``ARM_ALLOW_MULTI_CORE`` is disabled, all spin lock operations mutate to calls of ``EnterCritical()`` and ``LeaveCritical()`` automatically.

.. _CSpinLock:

CSpinLock
^^^^^^^^^

The class ``CSpinLock`` implements a spin lock, which is a synchronization object in a multi-core environment. It can be used to protect a data structure, which is shared between multiple cores, from destruction, when multiple cores are trying to access this data structure at the same time. The spin lock serializes the access, so that only one core can write or read the data at a time.

.. code-block:: c++

	#include <circle/spinlock.h>

.. cpp:class:: CSpinLock

In Circle a spin lock is initialized with this constructor:

.. cpp:function:: CSpinLock::CSpinLock (unsigned nTargetLevel = IRQ_LEVEL)

	nTargetLevel is the maximum execution level from which the spin lock is acquired and released.

.. cpp:function:: void CSpinLock::Acquire (void)

	This method tries to acquire the spin lock. It also raises the execution level to the level given to the constructor. If the spin lock is currently acquired by another core, the execution will be stalled, until the spin lock is released by the other core.

.. cpp:function:: void CSpinLock::Release (void)

	Releases the spin lock.

.. important::

	Calls to ``Acquire()`` cannot be nested for the same spin lock. If doing so, the execution will freeze. Multiple spin locks can be acquired in a row, but must be released in the opposite order. Otherwise a system deadlock may occur randomly.

.. rubric:: Footnotes

.. [#lv] These symbols are defined as C macros.

.. [#mt] Tasks are discussed in the section Multi-tasking.

.. [#iq] A number of callback functions in an Circle application (e.g. kernel timer handler) will be called directly from an IRQ handler.

.. _Memory Barriers:

Memory barriers
^^^^^^^^^^^^^^^

Memory barriers are system control CPU instructions, which influence the access to the main memory. They can be important especially in multi-core applications to ensure, that data has been written to or read from memory at a given place in the code.

When a variable is written by one CPU core in a multi-core environment, this is normally recognized by the other CPU cores, but for synchronization purposes barriers may be required, if a write or read operation must be completed at a specific place in code.

.. code-block:: c

	#include <circle/synchronization.h>

Circle defines the following memory barriers:

.. c:macro:: DataSyncBarrier()

	This barrier (also known as `DSB`) ensures, that all memory read and write operations have been completed, at the place where it is inserted in the code. It may be required to insert this barrier, after an application has written data from one CPU core, which will be read from an other CPU core afterwards.

.. c:macro:: DataMemBarrier()

	This barrier (also known as `DMB`) ensures, that all memory read operations have been completed, at the place where it is inserted in the code. It may be required to insert this barrier, before an application will read data, which has been written by an other CPU core before.

The following special barriers are especially used on the Raspberry Pi 1 and Zero. On other Raspberry Pi models they have no function.

.. c:macro:: PeripheralEntry()

	If your code directly accesses memory-mapped hardware registers, you should insert this special barrier before the first access to a specific hardware device.

.. c:macro:: PeripheralExit()

	If your code directly accesses memory-mapped hardware registers, you should insert this special barrier after the last access to a specific hardware device.

.. note::

	Most programs would work without ``PeripheralEntry()`` and ``PeripheralExit()``, but to be sure, it should be used as noted. In a few tests there have been issues on the Raspberry Pi 1, where invalid data was read from hardware registers, without these barriers inserted.

	You do not need to care about this, when you access hardware devices using a Circle device driver class, because this is handled inside the driver.
