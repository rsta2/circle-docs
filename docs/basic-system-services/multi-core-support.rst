.. _Multi-core support:

Multi-core support
~~~~~~~~~~~~~~~~~~

Beginning with the Raspberry Pi 2, four cores are provided by a Cortex-A CPU. Circle distinguishes between the primary core 0 and the secondary cores 1 to 3 in a way, that all system operations including interrupt handling are running on the primary core 0. The secondary cores are free to be used by the application. This allows to implement time-critical or time-consuming operations on the secondary cores, without being disturbed by interrupts or other system functions. The optional scheduler and all tasks are running on core 0 too (see :ref:`Multitasking`).

Circle supports multi-core applications by handling the start-up of the secondary cores with the class :ref:`CMultiCoreSupport`, with the synchronization class :ref:`CSpinLock` and with :ref:`Memory Barriers`.

The system option ``ARM_ALLOW_MULTI_CORE`` has to be defined to use multi-core support with the class ``CMultiCoreSupport``. For performance reasons this system option should not be defined for single core applications.

Further information on using the multi-core support is available in the file `doc/multicore.txt <https://github.com/rsta2/circle/blob/master/doc/multicore.txt>`_.

The sample programs `17-fractal` and `26-cpustress` can be build with multi-core support. A more complex multi-core example is the project `MiniSynth Pi <https://github.com/rsta2/minisynth/>`_.

.. _CMultiCoreSupport:

CMultiCoreSupport
^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/multicore.h>

.. cpp:class:: CMultiCoreSupport

If you want to use the secondary CPU cores in your application, you have to define a user class, which is derived from the class ``CMultiCoreSupport``.

.. cpp:function:: CMultiCoreSupport::CMultiCoreSupport (CMemorySystem *pMemorySystem)

	Creates the instance of ``CMultiCoreSupport``. Must be invoked in the first place of the initializer list of the defined user class. The parameter ``pMemorySystem`` must be set to ``CMemorySystem::Get()``, which can be included from ``<circle/memory.h>``.

.. cpp:function:: boolean CMultiCoreSupport::Initialize (void)

	Initializes the multi-core support and starts the secondary cores. It is important, that this method is called, when the other system initialization is already done. Normally it is invoked at the last method in ``CKernel::Run()``.

.. cpp:function:: virtual void CMultiCoreSupport::Run (unsigned nCore) = 0

	Overload this virtual method to define the entry for the secondary cores (1 to 3) into your application. It is invoked three times (once on each secondary core) with ``nCore`` being the number of the executing CPU core (1, 2 or 3).

.. important::

	When a secondary core returns from ``Run()``, the CPU core is automatically halted and will sleep. For unused cores you can simply return from this method.

.. note::

	This method is not executed from the primary CPU core 0 by default. If you want to handle all CPU cores at the same place, you have to explicitly call the ``Run()`` method of your user defined multi-core class from ``CKernel::Run()`` with the parameter 0.

.. cpp:function:: static unsigned CMultiCoreSupport::ThisCore (void)

	Returns the number of the CPU core (0, 1, 2 or 3), which called this method.

.. cpp:function:: static void CMultiCoreSupport::HaltAll (void)

	In a multi-core environment this method halts all CPU cores. The current execution will be interrupted using an Inter-Processor Interrupt (IPI) and each core calls the ``halt()`` function in turn.

.. cpp:function:: static void CMultiCoreSupport::SendIPI (unsigned nCore, unsigned nIPI)

	Sends an Inter-Processor Interrupt (IPI) with the number ``nIPI`` to the core ``nCore`` (0, 1, 2 or 3). If this technique is used for application purposes, ``nIPI`` can have a user defined value from ``IPI_USER`` to ``IPI_MAX``.

.. cpp:function:: virtual void CMultiCoreSupport::IPIHandler (unsigned nCore, unsigned nIPI)

	Overload this virtual method to receive Inter-Processor Interrupts (IPI) from other CPU cores. ``nCore`` is the number of the CPU core, which received the IPI and which is executing ``IPIHandler()``. ``nIPI`` is the IPI number specified in the call to ``CMultiCoreSupport::SendIPI()``.

.. important::

	Be sure to pass calls to this method further to ``CMultiCoreSupport::IPIHandler()`` with the same parameters, if ``nIPI < IPI_USER``. Otherwise the ``CMultiCoreSupport::HaltAll()`` method will not work, which is also invoked on a system panic condition (abort exception, assertion failed).
