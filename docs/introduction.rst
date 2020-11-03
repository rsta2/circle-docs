Introduction
------------

The `Circle <https://github.com/rsta2/circle>`_ project provides a C++ bare metal environment for the `Raspberry Pi <https://www.raspberrypi.org>`_ single-board computers (SBC). This is a framework for developing applications, which run on the bare hardware, without using an operating system, which is somewhat equivalent to programming a very powerful micro-controller. Frequent areas of application for the bare metal system model are:

* High-speed data acquisition (DAQ)
* Retro computer emulation with accurate timing
* Low latency, high performance audio processing

Characteristics of bare metal solutions can be:

* Low interrupt latency
* Full system control [#sc]_
* Light-weighted software architecture [#sa]_
* Direct hardware access [#hw]_
* Quick system start (boot)
* Can power off the system at any time [#po]_

This documentation provides the necessary information for developing bare metal applications using Circle.

.. rubric:: Footnotes

.. [#sc] Secondary CPU cores can be dedicated to a specific task.
.. [#sa] Common operating systems work with many software layers instead.
.. [#hw] Do not need to write a device driver to access hardware interfaces.
.. [#po] While the green Activity LED is off.
