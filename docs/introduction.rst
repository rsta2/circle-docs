Introduction
------------

The `Circle <https://github.com/rsta2/circle>`_ project provides a C++ bare metal environment for the `Raspberry Pi <https://www.raspberrypi.org>`_ single-board computers (SBC). This is a framework for developing applications, which runs on the bare hardware, without using an operating system. This is somewhat equivalent to programming a very performant micro-controller. Frequent areas of application for the bare metal system model are:

* High-speed data acquisition (DAQ)
* Retro computer emulation with accurate timing
* Low latency, high performance audio processing

Advantages of bare metal solutions can be:

* Low interrupt latency
* Full system control (for multiple cores too)
* Light-weighted software architecture (without many layers)
* Direct hardware access (without the need to write a device driver)
* Fast booting
* Can power off the system at any time

This documentation provides the necessary information for developing applications using Circle.
