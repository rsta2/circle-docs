Memory
~~~~~~

Circle enables the Memory Management Unit (MMU) to be able to use the data cache of the CPU to speed up operation, but it does not make use of virtual memory to implement specific system features. The physical-to-virtual address mapping is one-to-one over the whole used memory space. [#ma]_ The memory layout for the different system configurations can be found in `doc/memorymap.txt <https://github.com/rsta2/circle/blob/master/doc/memorymap.txt>`_.

new and delete
^^^^^^^^^^^^^^

Circle supports system heap memory. Memory can be allocated with the normal C++ ``new`` operator and freed with the ``delete`` operator. Allocating and freeing memory blocks is supported from ``TASK_LEVEL`` and ``IRQ_LEVEL``, but not from ``FIQ_LEVEL``. [#el]_ Allocated memory blocks are always aligned to the maximum size of a cache-line in the system. [#al]_

.. note::

	Circle keeps a number of linked lists to manage memory blocks of different sizes. The supported block sizes are defined by the system option ``HEAP_BLOCK_BUCKET_SIZES``. By default the maximum manageable block size is 512 KByte. Larger memory blocks can be allocated, but not re-used after ``delete``.

The ``new`` operator can have a parameter, which specifies the type of memory to be allocated:

==============	=================================================================
Parameter	Description
==============	=================================================================
HEAP_LOW	memory below 1 GByte
HEAP_HIGH	memory above 1 GByte (on Raspberry Pi 4 and 5 only)
HEAP_ANY	memory above 1 GB (if available) or memory below 1 GB (otherwise)
HEAP_DMA30	30-bit DMA-able memory (alias for HEAP_LOW)
==============	=================================================================

This is especially important on the Raspberry Pi 4 and 5, which support different SDRAM memory regions. For instance one can specify to allocate a 256 byte memory block above 1 GByte:

.. code-block:: c++

	#include <circle/new.h>

	unsigned char *p = new (HEAP_HIGH) unsigned char[256];

Further information on using memory type parameters is available in `doc/new-operator.txt <https://github.com/rsta2/circle/blob/master/doc/new-operator.txt>`_.

CMemorySystem
^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/memory.h>

.. cpp:class:: CMemorySystem

The class ``CMemorySystem`` implements most of the memory management function inside Circle. There is normally exactly one instance of this class in each Circle application, which is created by the Circle system initialization code. Earlier versions of Circle required to explicitly create this instance in ``CKernel``. This is deprecated now, but does not disturb either. If another instance of ``CMemorySystem`` is created, it is an alias for the first created instance.

Methods callable from applications are:

.. cpp:function:: size_t CMemorySystem::GetMemSize (void) const

	Returns the total memory size available to the application, as reported by the firmware.

.. cpp:function:: size_t CMemorySystem::GetHeapFreeSpace (int nType) const

	Returns the free space on the heap of the given type, according to the memory type ``HEAP_LOW``, ``HEAP_HIGH`` or ``HEAP_ANY``. Does not cover memory blocks, which have been freed.

.. cpp:function:: static CMemorySystem *CMemorySystem::Get (void)

	Returns a pointer to the instance of ``CMemorySystem``.

.. cpp:function:: static void CMemorySystem::DumpStatus (void)

	Dumps some memory allocation status information. Requires ``HEAP_DEBUG`` to be defined.

CClassAllocator
^^^^^^^^^^^^^^^

The class ``CClassAllocator`` allows to define a class-specific allocator for a class, using a pre-allocated store of memory blocks. This can speed up memory allocation, if the maximum number of instances of the class is known and a class instance does not occupy too much memory space. If you want to use this technique for your own class, the class definition has to look like this:

.. code-block:: c++
	:caption: myclass.h

	#include <circle/classallocator.h>

	class CMyClass
	{
	...

		DECLARE_CLASS_ALLOCATOR
	};

You have to add the following to the end of the class implementation file:

.. code-block:: c++
	:caption: myclass.cpp

	#include "myclass.h"

	...

	IMPLEMENT_CLASS_ALLOCATOR (CMyClass)

Before an instance of your class can be created, one of these (macro-) functions have to be executed:

.. code-block:: c++

	#include "myclass.h"

	INIT_CLASS_ALLOCATOR (CMyClass, Number);			// or:

	INIT_PROTECTED_CLASS_ALLOCATOR (CMyClass, Number, Level);

The second variant initializes a class-specific allocator, which is protected with a spin-lock for concurrent use. *Number* is the number of pre-allocated memory blocks and *Level* the maximum execution level, from which ``new`` or ``delete`` for this class will be called. [#el]_ This variant can be called multiple times with the same *Level* parameter. The class store will be extended then by the given number of objects.

C functions
^^^^^^^^^^^

Circle provides the following C standard library functions for memory allocation:

.. code-block:: c

	#include <circle/alloc.h>

	void *malloc (size_t nSize);
	void *calloc (size_t nBlocks, size_t nSize);
	void *realloc (void *pBlock, size_t nSize);
	void free (void *pBlock);

.. rubric:: Footnotes

.. [#ma] There is one exception from this rule. On the Raspberry Pi 4 the memory mapped I/O register space of the xHCI USB controller, which is connected using a PCIe interface, is re-mapped into the 4 GByte 32-bit address space, because it is physically located above the 4 GByte boundary, and would not be accessible in 32-bit mode otherwise.

.. [#el] System execution levels (e.g. ``TASK_LEVEL``) are described in the section :ref:`synchronization`.

.. [#al] 32 bytes on the Raspberry Pi 1 and Zero, 64 bytes otherwise
