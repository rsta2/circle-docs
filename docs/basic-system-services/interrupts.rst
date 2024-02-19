Interrupts
~~~~~~~~~~

This section describes the low-level hardware-interrupt support in Circle. This should be only of interest, if one wants to develop its own device driver or driver-like functions.

In the ARM architecture there are two types of interrupt request, IRQ and FIQ. The IRQ is the basic interrupt request type, can have multiple active sources on all Raspberry Pi models and is used to control most interrupt-driven devices. The FIQ (Fast Interrupt Request) is used for low-latency interrupts and can have only one active interrupt source at a time on the Raspberry Pi 1-3 and Zero. The Raspberry Pi 4 has an new interrupt controller (GIC-400) and may theoretically support multiple simultaneous FIQ sources, but for a homogeneous solution this currently not supported in Circle. Therefore there are normally multiple active interrupt sources in a system, which use the IRQ, but only up to one, which uses the FIQ.

.. important::

	The FIQ is currently not supported on the Raspberry Pi 5.

CInterruptSystem
^^^^^^^^^^^^^^^^

The class ``CInterruptSystem`` is the provider of hardware-interrupt support in Circle. Hardware-interrupt support is not mandatory in an application, but if it is used, there is exactly one instance of this class in the system.

.. code-block:: c++

	#include <circle/interrupt.h>

.. cpp:class:: CInterruptSystem

.. cpp:function:: boolean CInterruptSystem::Initialize (void)

	Initializes the interrupt system. Returns ``TRUE`` on success.

.. note::

	There is a two step initialization required for the interrupt system. Step one is done in the constructor of ``CInterruptSystem``, step two in ``Initialize()``.

.. cpp:function:: void CInterruptSystem::ConnectIRQ (unsigned nIRQ, TIRQHandler *pHandler, void *pParam)

	Connects an interrupt handler to an IRQ source (vector). The known interrupt sources are defined in ``<circle/bcm2835int.h>`` for the Raspberry Pi 1-3 and Zero and in ``<circle/bcm2711int.h>`` for the Raspberry Pi 4. An IRQ handler has the following prototype:

.. code-block:: c++

	void IRQHandler (void *pParam);

``pParam`` can be any user parameter and gets the value specified in the call to ``ConnectIRQ()`` for this IRQ source.

.. cpp:function:: void CInterruptSystem::DisconnectIRQ (unsigned nIRQ)

	Disconnects the interrupt handler from the given IRQ source.

.. cpp:function:: void CInterruptSystem::ConnectFIQ (unsigned nFIQ, TFIQHandler *pHandler, void *pParam)

	Connects an interrupt handler to a FIQ source. Only one active FIQ source is allowed at a time. An FIQ handler has the same prototype as an IRQ handler (see above).

.. cpp:function:: void CInterruptSystem::DisconnectFIQ (void)

	Disconnects the interrupt handler of the active FIQ source.

.. cpp:function:: static CInterruptSystem *CInterruptSystem::Get (void)

	Returns a pointer to the only instance of ``CInterruptSystem``.

.. important::

	If one or more IRQ handlers in a system make use of floating point registers, the system option ``SAVE_VFP_REGS_ON_IRQ`` has to be enabled. The same applies accordingly to ``SAVE_VFP_REGS_ON_FIQ`` for FIQ handlers. These system options are enabled by default, when a toolchain is used to build Circle, which is based on GNU-C 12.1 or later.
