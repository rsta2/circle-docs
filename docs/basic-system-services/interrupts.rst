Interrupts
~~~~~~~~~~

This section describes the low-level hardware-interrupt support in Circle. This should be only of interest, if one wants to develop its own device driver or driver-like functions.

In the ARM architecture there are two types of interrupt request, IRQ and FIQ. The IRQ is the basic interrupt request type, can have multiple active sources on all Raspberry Pi models and is used to control most interrupt-driven devices. The FIQ (Fast Interrupt Request) is used for low-latency interrupts and can have only one active interrupt source at a time on the Raspberry Pi 1-3 and Zero. The Raspberry Pi 4 has an new interrupt controller (GIC-400) and may theoretically support multiple simultaneous FIQ sources, but for a homogeneous solution this currently not supported in Circle. Therefore there are normally multiple active interrupt sources in a system, which use the IRQ, but only up to one, which uses the FIQ.

.. important::

	The FIQ is currently not supported on the Raspberry Pi 5 by Circle.

CInterruptSystem
^^^^^^^^^^^^^^^^

The class ``CInterruptSystem`` is the provider of hardware-interrupt support in Circle. Hardware-interrupt support is now mandatory in every application and an instance of this class is created in the system initialization. For compatibility it is still possible, to create a second instance of this class. A call to a method of the second instance will be automatically routed to the first instance.

.. code-block:: c++

	#include <circle/interrupt.h>

.. cpp:class:: CInterruptSystem

.. cpp:function:: boolean CInterruptSystem::Initialize (void)

	Initializes the interrupt system. Returns ``TRUE`` on success.

.. note::

	There is a two step initialization required for the interrupt system. Step one is done in the constructor of ``CInterruptSystem``, step two in ``Initialize()``.

.. cpp:function:: void CInterruptSystem::ConnectIRQ (unsigned nIRQ, TIRQHandler *pHandler, void *pParam)

	Connects an interrupt handler to an IRQ source (vector). The known interrupt sources are defined in ``<circle/bcm2835int.h>`` for the Raspberry Pi 1-3 and Zero, in ``<circle/bcm2711int.h>`` for the Raspberry Pi 4 and 5 and in ``<circle/rp1int.h>`` for interrupt sources from the RP1 soundbridge. An IRQ handler has the following prototype:

.. c:type:: void IRQHandler (void *pParam)

	``pParam`` can be any user parameter and gets the value specified in the call to :cpp:func:`CInterruptSystem::ConnectIRQ()` for this IRQ source.

.. note::

	For the Raspberry Pi 5 Circle uses a new scheme for IRQ numbers. This allows to distinguish between IRQ interrupt sources, which are directly connected to the main GIC-400 interrupt controller and interrupt sources, which are routed from the RP1 southbridge to the GIC.

	IRQ numbers, which have the bit 11 set, are second level IRQs, which are routed from the RP1 to the GIC. All other IRQs are directly connected to the GIC. An IRQ number, which has the bit 10 set, is edge-triggered (otherwise level-triggered). Edge-triggered interrupts are currently only supported from inside the RP1.

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
