Subsystems
----------

This section describes the Circle subsystems. A subsystems is a group of classes, which implements services for a specific purpose, which are different from the :ref:`Basic system services` and are normally provided by its own library (see :ref:`libraries`). Only those classes are discussed here, which are directly used by applications. All Circle classes are listed in `doc/classes.txt <https://github.com/rsta2/circle/blob/master/doc/classes.txt>`_.

The Circle project does not provide a single centralized C++ header file. Instead the header file(s), which must be included for a specific class, function or macro definition are specified in the related subsection.

.. toctree::
	:maxdepth: 1

	multitasking
	usb
	filesystems
	tcp-ip-networking
	graphics
	vc4
