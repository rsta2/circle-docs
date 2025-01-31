.. _Basic system services:

Basic system services
---------------------

This section describes the basic system services, which are provided for applications by the Circle base library `libcircle.a`. Only those classes are discussed here, which are directly used by applications. All Circle classes are listed in `doc/classes.txt <https://github.com/rsta2/circle/blob/master/doc/classes.txt>`_.

The Circle project does not provide a single centralized C++ header file. Instead the header file(s), which must be included for a specific class, function or macro definition are specified in the related subsection.

.. toctree::
	:maxdepth: 1

	system-information
	memory
	synchronization
	system-log
	interrupts
	time
	direct-memory-access
	gpio-access
	multi-core-support
	cpu-clock-rate-management
	firmware-access
	direct-hardware-access
	font-service
	utilities
	debugging-support
