.. _VC4:

VC4
~~~

The VC4 subsystem in `addon/vc4 <https://github.com/rsta2/circle/tree/master/addon/vc4>`_ provides the VCHIQ driver as an interface to the audio and accelerated graphics services, which are offered by the Raspberry Pi firmware. The accelerated graphics support is not available on the Raspberry Pi 4 and 5 and with ``AARCH = 32`` only. This section describes the components of the VC4 subsystem.

.. _VCHIQ driver:

VCHIQ driver
^^^^^^^^^^^^

.. code-block:: cpp

	#include <vc4/vchiq/vchiqdevice.h>

.. cpp:class:: CVCHIQDevice : public CLinuxDevice

	This class is a driver for the VC host interface queue, which implements an interface to a number of service processes, which are running on the video processing unit (VPU) of the Raspberry Pi computers. Because this driver has been ported from Linux, it is based on the Linux kernel device driver emulation code in `addon/linux <https://github.com/rsta2/circle/tree/master/addon/linux>`_. The API of the VCHIQ driver is based on the C language, and is not covered by this documentation.

.. cpp:function:: CVCHIQDevice::CVCHIQDevice (CMemorySystem *pMemory, CInterruptSystem *pInterrupt)

	Creates an instance of the VCHIQ driver class. There can be only one. ``pMemory`` and ``pInterrupt`` are pointers to the Circle memory and interrupt system objects.


.. cpp:function:: boolean CVCHIQDevice::Initialize (void)

	Initializes the VCHIQ driver. Returns ``TRUE`` on success. This method is inherited from the base class ``CLinuxDevice``.

VCHIQ sound
^^^^^^^^^^^

The VCHIQ sound driver class :cpp:class:`CVCHIQSoundBaseDevice` is described in the :ref:`Audio devices` section.

Accelerated graphics
^^^^^^^^^^^^^^^^^^^^

The accelerated graphics support in `addon/vc4/interface <https://github.com/rsta2/circle/tree/master/addon/vc4/interface>`_ has been ported from the Raspberry Pi OS (former Raspbian) userland libraries, which implement the following APIs:

* EGL 1.4
* OpenGL ES 1.1 and 2.0
* OpenVG 1.1
* Dispmanx (proprietary)

Please see `this website <https://www.khronos.org/>`_ for detailed information about the first three APIs, which are not specific to Circle and are based on the C language.

.. note::

	The accelerated graphics support is not available on the Raspberry Pi 4 and 5 and with ``AARCH = 32`` only.
