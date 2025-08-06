.. _Multitasking:

Multitasking
~~~~~~~~~~~~

Circle provides an optional cooperative non-preemptive scheduler, which allows to solve programming problems based on the process concept. Because in Circle there is only one flat address space with a one-to-one physical-to-virtual address mapping a process in Circle is similar to a thread. In Circle the name "task" is used instead.

Because the scheduler is optional, a Circle application can work without it. The scheduler was introduced to implement TCP/IP networking support, which required many threads of execution at the same time to be implemented even on the one-core Raspberry Pi models. Later porting the VCHIQ driver and HDMI sound support, the accelerated graphics support (Raspberry Pi 1-3 and Zero only) and Wireless LAN support had similar requirements. It can be useful to use the scheduler also for modeling complex user problems in Circle.

The scheduler library provides the following classes:

======================	===================================================
Class			Function
======================	===================================================
CScheduler		Cooperative non-preemptive scheduler
CTask			Representation of a thread of execution, a task
CMutex			Mutual exclusion (critical sections) across tasks
CSemaphore		Implements the well-known semaphore concept
CSynchronizationEvent	Synchronizes the execution of task(s) with an event
======================	===================================================

.. important::

	In a multi-core environment (see :ref:`Multi-core support`) all tasks and the scheduler run on CPU core 0.

CScheduler
^^^^^^^^^^

This class implements a cooperative non-preemptive scheduler, which controls which task runs at a time. Because the scheduler is non-preemptive, a running task has to explicitly release the CPU by sleeping, waiting for a synchronization object (mutex, semaphore, synchronization event) or by calling ``CScheduler::Get()->Yield()`` after a short time, so that the other tasks are able to run. This relatively simple scheduler implements the round-robin policy without task priorities (and without much overhead).

.. code-block:: cpp

	#include <circle/sched/scheduler.h>

.. cpp:class:: CScheduler

.. cpp:function:: static boolean CScheduler::IsActive (void)

	Returns ``TRUE`` if the scheduler is available in the system. The scheduler is optional in Circle.

.. cpp:function:: static CScheduler *CScheduler::Get (void)

	Returns a pointer to the only scheduler object in the system. It must not be called, if the scheduler is not available.

.. cpp:function:: CTask *CScheduler::GetCurrentTask (void)

	Returns a pointer to the ``CTask`` object of the currently running task.

.. cpp:function:: CTask *CScheduler::GetTask (const char *pTaskName)

	Returns a pointer to the ``CTask`` object of the task with the name ``pTaskName`` or 0, if the task was not found.

.. cpp:function:: boolean CScheduler::IsValidTask (CTask *pTask)

	Returns ``TRUE``, if ``pTask`` is referencing a CTask object of a currently known task.

.. cpp:function:: void CScheduler::Yield (void)

	Switch to the next task. The currently running task releases the CPU and the next task in round-robin order, which is not blocked, gets control.

.. important::

	A task should call this from time to time, if it does longer calculations.

.. cpp:function:: void CScheduler::Sleep (unsigned nSeconds)

	The current task pauses execution for ``nSeconds`` seconds. The next ready task gets control.

.. cpp:function:: void CScheduler::MsSleep (unsigned nMilliSeconds)

	The current task pauses execution for ``nMilliSeconds`` milliseconds. The next ready task gets control.

.. cpp:function:: void CScheduler::usSleep (unsigned nMicroSeconds)

	The current task pauses execution for ``nMicroSeconds`` microseconds. The next ready task gets control.

.. cpp:function:: void CScheduler::SuspendNewTasks (void)

	Causes all new tasks to be created in a suspended state. You can achieve the same, if you set the parameter ``bCreateSuspended`` to ``TRUE``, when calling ``new`` for a task. Nested calls to ``SuspendNewTasks()`` and ``ResumeNewTasks()`` are allowed.

.. cpp:function:: void CScheduler::ResumeNewTasks (void)

	Stops causing new tasks to be created in a suspended state and starts any tasks that were created suspended. Nested calls to ``SuspendNewTasks()`` and ``ResumeNewTasks()`` are allowed.

.. cpp:function:: boolean CScheduler::EnumerateTasks (boolean (*pCallback) (CTask *pTask, const char *pName, TTaskState State, TTaskFlags Flags, void *pParam), void *pParam)

	Enumerates all existing tasks. Invokes ``pCallback`` for each task. ``pParam`` is a user defined pointer, that will back passed to the callback. Returns ``FALSE``, if the enumeration was canceled by the callback returning ``FALSE``.

.. c:enum:: TTaskState

	* TaskStateNew
	* TaskStateReady
	* TaskStateBlocked
	* TaskStateBlockedWithTimeout
	* TaskStateSleeping
	* TaskStateTerminated

.. c:enum:: TTaskFlags

	* TaskFlagNone
	* TaskFlagRunning
	* TaskFlagSuspended

	These flag values can be or'ed together.

.. cpp:function:: void CScheduler::ListTasks (CDevice *pTarget)

	Writes a task listing to the device ``pTarget``.

.. cpp:function:: void CScheduler::RegisterTaskSwitchHandler (TSchedulerTaskHandler *pHandler)

	``pHandler`` is called on each task switch. This method is normally used by the Linux kernel driver and Pthreads emulation. The handler is called with a pointer to the ``CTask`` object of the task, which gets control now. The prototype of the handler is:

.. code-block:: c

	void TSchedulerTaskHandler (CTask *pTask);

.. cpp:function:: void CScheduler::RegisterTaskTerminationHandler (TSchedulerTaskHandler *pHandler)

	``pHandler`` is called, when a task terminates. This method is normally used by the Linux kernel driver and Pthreads emulation. The handler is called with a pointer to the ``CTask`` object of the task, which terminates. See ``RegisterTaskSwitchHandler()`` for the prototype of the handler.

CTask
^^^^^

Derive this class, define the ``Run()`` method to implement your own task and call ``new`` on it to start it.

.. code-block:: cpp

	#include <circle/sched/task.h>

.. cpp:class:: CTask

.. cpp:function:: CTask::CTask (unsigned nStackSize = TASK_STACK_SIZE, boolean bCreateSuspended = FALSE)

	Creates a task. ``nStackSize`` is the stack size for this task. By default a new task is immediately ready to run and its ``Run()`` method can be called. If you have to do more initialization, before the task can run, set ``bCreateSuspended`` to ``TRUE``. The task has to be started explicitly by calling ``Start()`` on it then.

.. cpp:function:: virtual void CTask::Run (void)

	Override this method to define the entry point for your own task. The task is automatically terminated, when ``Run()`` returns.

.. cpp:function:: void CTask::Start (void)

	Starts a task, that was created with ``bCreateSuspended = TRUE`` or restarts it after ``Suspend()``.

.. cpp:function:: void CTask::Suspend (void)

	Suspends a task from running, until ``Resume()`` is called for this task.

.. cpp:function:: void CTask::Resume (void)

	Alternative method to (re-)start a suspended task.

.. cpp:function:: boolean CTask::IsSuspended (void) const

	Returns ``TRUE``, if the task is currently suspended from running.

.. cpp:function:: void CTask::Terminate (void)

	Terminates the execution of the task. This method can only be called by the task itself. The task terminates on return from ``Run()`` too.

.. cpp:function:: void CTask::WaitForTermination (void)

	Waits for the termination of the task. This method can only be called by an other task.

.. cpp:function:: void CTask::SetName (const char *pName)

	Sets the specific name ``pName`` for this task.

.. cpp:function:: const char *CTask::GetName (void) const

	Returns a pointer to 0-terminated name string of this task. The default name of a task is constructed from the address of its task object (e.g. ``"@84abc0"``). The main application task has the name ``"main"``.

.. cpp:function:: void CTask::SetUserData (void *pData, unsigned nSlot)

	Sets a user pointer for this task. If you have to associate some data with a task, you can call this method with ``nSlot = TASK_USER_DATA_USER``. ``pData`` is any user pointer to be set.

.. cpp:function:: void *CTask::GetUserData (unsigned nSlot)

	Returns a user pointer for this task, which has previously been set using ``SetUserData()``. ``nSlot`` must be ``TASK_USER_DATA_USER`` for application usage.

CMutex
^^^^^^

Provides a method to provide mutual exclusion (critical sections) across tasks.

.. code-block:: cpp

	#include <circle/sched/mutex.h>

.. cpp:class:: CMutex

.. cpp:function:: void CMutex::Acquire (void)

	Acquires the mutex. The current task blocks, if another task already acquired the mutex. The mutex can be acquired multiple times by the same task.

.. cpp:function:: void CMutex::Release (void)

	Releases the mutex. Another task, which was waiting for the mutex to acquire, will be waken.

CSemaphore
^^^^^^^^^^

Implements the well-known `semaphore <https://en.wikipedia.org/wiki/Semaphore_(programming)>`_ synchronization concept, which was initially defined by Dijkstra. The class maintains a non-negative counter, which is decremented with the ``Down()`` operation. When this is not possible, because the counter is already zero, the calling task waits, until the counter is incremented again. This is possible with the ``Up()`` operation. Semaphores can be used to control the access to a limited number of resources.

.. code-block:: cpp

	#include <circle/sched/semaphore.h>

.. cpp:class:: CSemaphore

.. cpp:function:: CSemaphore::CSemaphore (unsigned nInitialCount = 1)

	Creates a semaphore. ``nInitialCount`` is the initial count of the semaphore.

.. cpp:function:: unsigned CSemaphore::GetState (void) const

	Returns the current count of the semaphore.

.. cpp:function:: void CSemaphore::Down (void)

	Decrements the semaphore count. Blocks the calling task, if the count is already zero.

.. cpp:function:: void CSemaphore::Up (void)

	Increments the semaphore count. Wakes another waiting task, if the count was zero. Can be called from interrupt context.

.. cpp:function:: boolean CSemaphore::TryDown (void)

	Tries to decrement the semaphore count. Returns ``TRUE`` on success or ``FALSE``, if the count is zero.

CSynchronizationEvent
^^^^^^^^^^^^^^^^^^^^^

Provides a method to synchronize the execution of tasks with an event. The event can be set or cleared. If a task is waiting for the event, it is blocked, when the event is cleared (unset) and will continue execution, when the event is set again. Multiple tasks can wait for the event at the same time.

.. code-block:: cpp

	#include <circle/sched/synchronizationevent.h>

.. cpp:class:: CSynchronizationEvent

.. cpp:function:: CSynchronizationEvent::CSynchronizationEvent (boolean bState = FALSE)

	Creates the synchronization event. ``bState`` is the initial state of the event (default cleared).

.. cpp:function:: boolean CSynchronizationEvent::GetState (void)

	Returns the current state for the synchronization event.

.. cpp:function:: void CSynchronizationEvent::Clear (void)

	Clears the synchronization event.

.. cpp:function:: void CSynchronizationEvent::Set (void)

	Sets the synchronization event.  Wakes all tasks currently waiting for the event. Can be called from interrupt context.

.. cpp:function:: void CSynchronizationEvent::Wait (void)

	Blocks the calling task, if the synchronization event is cleared. The task will wake up, when the event is set later. Multiple tasks can wait for the event to be set.

.. cpp:function:: boolean CSynchronizationEvent::WaitWithTimeout (unsigned nMicroSeconds)

	Blocks the calling task for ``nMicroSeconds`` microseconds, if the synchronization event is cleared. The task will wake up, when the event is set later. Multiple tasks can wait for the event to be set. This method returns ``TRUE``, if ``nMicroSeconds`` microseconds have elapsed, before the event has been set. To determine, what caused the method to return, use ``GetState()`` to see, if the event has been set. It is possible to have timed out and the event is set anyway.
