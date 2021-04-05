System data types
~~~~~~~~~~~~~~~~~

This appendix lists the system data types, which are defined and used by Circle. You can also include ``<stdint.h>`` to use POSIX types. This file is provided by the toolchain and is not available, if your application is built with ``STDLIB_SUPPORT = 0``.

.. code-block:: c

	#include <circle/types.h>

==============	=================================================
Type		Description
==============	=================================================
u8		8-bit unsigned value
u16		16-bit unsigned value
u32		32-bit unsigned value
u64		64-bit unsigned value
s8		8-bit signed value
s16		16-bit signed value
s32		32-bit signed value
s64		64-bit signed value
uintptr		unsigned value with the size of a pointer
intptr		signed value with the size of a pointer
size_t		count of bytes, result of the ``sizeof`` operator
ssize_t		count of bytes or an error value
boolean		can be TRUE or FALSE
==============	=================================================

.. note::

	``boolean`` is a synonym for the standard type ``bool``, which can be used instead, with the values ``true`` or ``false``. The definition of ``boolean`` has historical reasons, but is still used for an uniform source code.
