Utilities
~~~~~~~~~

This section lists utility classes and functions, which help to implement Circle applications.

.. _CString:

CString
^^^^^^^

.. code-block:: cpp

	#include <circle/string.h>

.. cpp:class:: CString

This class encapsulates a character string and allows different manipulations on it. The methods of this class are not reentrant.

.. cpp:function:: CString::CString (void)

	Creates an empty string object (``""``);

.. cpp:function:: CString::CString (const char *pString)

	Creates a string object. Sets the string initially to ``pString``.

.. cpp:function:: CString::CString (const CString &rString)

	Copy constructor. Creates a new string object. Sets the string initially to the value of ``rString``. ``rString`` remains unchanged.

.. cpp:function:: CString::CString (CString &&rrString)

	Move constructor. Creates a new string object. Sets the string initially to the value of ``rrString``. ``rrString`` is set to an empty string.

.. cpp:function:: CString::operator const char *(void) const

	Returns a pointer to the string buffer, which is terminated with a zero-character.

.. cpp:function:: const char *CString::operator = (const char *pString)

	Assigns a new string. Returns a pointer to the string buffer, which is terminated with a zero-character.

.. cpp:function:: CString &CString::operator = (const CString &rString)

	Assigns a new string. Returns a reference to the string object.

.. cpp:function:: CString &CString::operator = (CString &&rrString)

	Move assignment. Assigns a new string. ``rrString`` is set to an empty string. Returns a reference to the string object.

.. cpp:function:: size_t CString::GetLength (void) const

	Returns the length of the string in number of characters (zero for empty string).

.. cpp:function:: void CString::Append (const char *pString)

	Appends ``pString`` to the string.

.. cpp:function:: int CString::Compare (const char *pString) const

	Compares ``pString`` with the string. Returns:

	* zero, if the strings are identical
	* a negative value, if the string is smaller than ``pString``
	* a positive value, if the string is greater than ``pString``

.. cpp:function:: int CString::Find (char chChar) const

	Searches for ``chChar`` in the string. Returns the zero-based index of the character or -1, if it is not found.

.. cpp:function:: int CString::Replace (const char *pOld, const char *pNew)

	Replaces all occurrences of ``pOld`` with ``pNew`` in the string. Returns the number of occurrences.

.. cpp:function:: void CString::Format (const char *pFormat, ...)

	Formats a string as known from ``sprintf()``. Does support only a subset of the known format specifiers:

	``%[#][[-][0]len][.prec][l|ll]{c|d|f|i|o|p|s|u|x|X}``

	======	=================================================================================
	Field	Description
	======	=================================================================================
	#	insert prefix 0, 0x or 0X for %o, %x or %X
	\-	left justify output
	0	insert leading zeros
	len	decimal number specifying the length of the field
	.prec	decimal number specifying the precision for %f
	l	type is ``long``
	ll 	type is ``long long`` (with STDLIB_SUPPORT >= 1 only)
	c	insert ``char``
	d	insert decimal ``int``, ``long`` or ``long long`` (maybe with sign)
	f	insert ``double``
	i	same as %d
	o	insert octal ``unsigned``, ``unsigned long`` or ``unsigned long long``
	p	same as %x
	s	insert string (type is ``const char *``)
	u	insert decimal ``unsigned``, ``unsigned long`` or ``unsigned long long``
	x	insert hex ``unsigned``, ``unsigned long`` or ``unsigned long long`` (lower case)
	X	insert hex ``unsigned``, ``unsigned long`` or ``unsigned long long`` (upper case)
	======	=================================================================================

.. cpp:function:: void CString::FormatV (const char *pFormat, va_list Args)

	Same as ``Format()``, but ``Args`` are given as ``va_list``.

CPtrArray
^^^^^^^^^

.. code-block:: cpp

	#include <circle/ptrarray.h>

.. cpp:class:: CPtrArray

This class implements a dynamic array of pointers. The methods of this class are not reentrant.

.. cpp:function:: CPtrArray::CPtrArray (unsigned nInitialSize = 100, unsigned nSizeIncrement = 100)

	Creates a ``CPtrArray`` object with initially space for ``nInitialSize`` elements. The memory allocation will be increased by ``nSizeIncrement`` elements, when the array is full.

.. cpp:function:: unsigned CPtrArray::GetCount (void) const

	Returns the current number of used elements in the array.

.. cpp:function:: void *CPtrArray::operator[] (unsigned  nIndex) const

	Returns the pointer for the array element at ``nIndex`` (based on zero). ``nIndex`` must be smaller than the value returned from ``GetCount()``.

.. cpp:function:: void *&CPtrArray::operator[] (unsigned nIndex)

	Returns a reference to the pointer for the array element at ``nIndex`` (based on zero). ``nIndex`` must be smaller than the value returned from ``GetCount()``.

.. cpp:function:: unsigned CPtrArray::Append (void *pPtr)

	Appends ``pPtr`` to end of the array.

.. cpp:function:: void CPtrArray::RemoveLast (void)

	Removes the last element from the array.

CPtrList
^^^^^^^^

.. code-block:: cpp

	#include <circle/ptrlist.h>

.. cpp:class:: CPtrList

This class implements a linked list of pointers. The methods of this class are not reentrant.

.. c:type:: TPtrListElement

	Opaque type definition.

.. cpp:function:: TPtrListElement *CPtrList::GetFirst (void) const

	Returns the first element, or 0 if list is empty.

.. cpp:function:: TPtrListElement *CPtrList::GetNext (TPtrListElement *pElement) const

	Returns the next element following ``pElement``, or 0 if nothing follows.

.. cpp:function:: static void *CPtrList::GetPtr (TPtrListElement *pElement)

	Returns the pointer for ``pElement``.

.. cpp:function:: void CPtrList::InsertBefore (TPtrListElement *pAfter, void *pPtr)

	Inserts ``pPtr`` before the element ``pAfter``, which must not be 0.

.. cpp:function:: void CPtrList::InsertAfter (TPtrListElement *pBefore, void *pPtr)

	Inserts ``pPtr`` after the element ``pBefore``. Use ``pBefore == 0`` to set the first element in the list (list must be empty).

.. cpp:function:: void CPtrList::Remove (TPtrListElement *pElement)

	Removes the element ``pElement`` from the list.

.. cpp:function:: TPtrListElement *CPtrList::Find (void *pPtr) const

	Searches the element, whose pointer is equal to ``pPtr`` and returns it, or 0 if ``pPtr`` was not found.

CPtrListFIQ
^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/ptrlistfiq.h>

.. cpp:class:: CPtrListFIQ

	Same as :cpp:class:`CPtrList`, but can be used from ``FIQ_LEVEL``.

.. cpp:function:: CPtrListFIQ::CPtrListFIQ (unsigned nMaxElements)

	Creates a pointer list with up to ``nMaxElements`` elements.

CNumberPool
^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/numberpool.h>

.. cpp:class:: CNumberPool

This class implements an allocation pool for numbers. The methods of this class are not reentrant.

.. cpp:member:: static const unsigned Limit = 63

	Allowed maximum of an allocated number.

.. cpp:member:: static const unsigned Invalid = Limit+1

	Returned by ``AllocateNumber()`` on failure.

.. cpp:function:: CNumberPool::CNumberPool (unsigned nMin, unsigned nMax = Limit)

	Creates a number pool. ``nMin`` is the minimal returned number by ``AllocateNumber()``. ``nMax`` is the maximal returned number.

.. cpp:function:: unsigned CNumberPool::AllocateNumber (boolean bMustSucceed, const char *pFrom = "numpool")

	Allocates a number from the number pool and returns it. If there are no more numbers available, this method returns ``CNumberPool::Invalid``, if ``bMustSucceed`` is ``FALSE``, or the system halts with a panic message otherwise. This message has the prefix ``pFrom``.

.. cpp:function:: void CNumberPool::FreeNumber (unsigned nNumber)

	Returns ``nNumber``, which has been allocated earlier, to the number pool for reuse.

Atomic memory access
^^^^^^^^^^^^^^^^^^^^

.. code-block:: c

	#include <circle/atomic.h>

This header file defines some functions, which implement an atomic access to an aligned ``int`` variable in memory. These functions can be useful for synchronization purposes, especially for multi-core applications, where using a spin lock would be too time consuming. All accesses to such a variable must use one of the following functions, to ensure them being atomic.

.. c:function:: int AtomicGet (const volatile int *pVar)

	Returns the value of the ``int`` variable at ``pVar``.

.. c:function:: int AtomicSet (volatile int *pVar, int nValue)

	Sets the ``int`` variable at ``pVar`` to ``nValue`` and returns ``nValue``.

.. c:function:: int AtomicExchange (volatile int *pVar, int nValue)

	Sets the ``int`` variable at ``pVar`` to ``nValue`` and returns the previous value.

.. c:function:: int AtomicCompareExchange (volatile int *pVar, int nCompare, int nValue)

	Sets the ``int`` variable at ``pVar`` to ``nValue``, if the previous value of the variable was ``nCompare``, and returns the previous value of the variable.

.. c:function:: int AtomicAdd (volatile int *pVar, int nValue)

	Adds ``nValue`` to the ``int`` variable at ``pVar``. Returns the result of the operation.

.. c:function:: int AtomicSub (volatile int *pVar, int nValue)

	Subtracts ``nValue`` from the ``int`` variable at ``pVar``. Returns the result of the operation.

.. c:function:: int AtomicIncrement (volatile int *pVar)

	Increments the ``int`` variable at ``pVar`` by 1. Returns the result of the operation.

.. c:function:: int AtomicDecrement (volatile int *pVar)

	Decrements the ``int`` variable at ``pVar`` by 1. Returns the result of the operation.

C standard library functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c

	#include <circle/util.h>

This header file defines some functions, known from the C standard library.

Memory functions
""""""""""""""""

.. c:function:: void *memset (void *pBuffer, int nValue, size_t nLength)
.. c:function:: void *memcpy (void *pDest, const void *pSrc, size_t nLength)
.. c:function:: void *memmove (void *pDest, const void *pSrc, size_t nLength)
.. c:function:: int memcmp (const void *pBuffer1, const void *pBuffer2, size_t nLength)

String functions
""""""""""""""""

.. c:function:: size_t strlen (const char *pString)
.. c:function:: int strcmp (const char *pString1, const char *pString2)
.. c:function:: int strcasecmp (const char *pString1, const char *pString2)
.. c:function:: int strncmp (const char *pString1, const char *pString2, size_t nMaxLen)
.. c:function:: int strncasecmp (const char *pString1, const char *pString2, size_t nMaxLen)
.. c:function:: char *strcpy (char *pDest, const char *pSrc)
.. c:function:: char *strncpy (char *pDest, const char *pSrc, size_t nMaxLen)
.. c:function:: char *strcat (char *pDest, const char *pSrc)
.. c:function:: char *strchr (const char *pString, int chChar)
.. c:function:: char *strstr (const char *pString, const char *pNeedle)
.. c:function:: char *strtok_r (char *pString, const char *pDelim, char **ppSavePtr)

Number conversion
"""""""""""""""""

.. c:function:: unsigned long strtoul (const char *pString, char **ppEndPtr, int nBase)
.. c:function:: unsigned long long strtoull (const char *pString, char **ppEndPtr, int nBase)
.. c:function:: int atoi (const char *pString)

Other functions
^^^^^^^^^^^^^^^

.. code-block:: c

	#include <circle/util.h>

.. c:function:: u16 bswap16 (u16 usValue)
.. c:function:: u32 bswap32 (u32 ulValue)

	Swaps the byte order of a 16- or 32-bit value.

.. c:function:: int parity32 (unsigned nValue)

	Returns the number of 1-bits in ``nValue`` modulo 1.

Macros
^^^^^^

.. code-block:: c

	#include <circle/macros.h>

.. c:macro:: PACKED

	Packs a ``struct`` definition. The members will be stored tightly, not aligned as usual.

.. c:macro:: ALIGN(n)

	Aligns a variable or member to a boundary of ``n`` in memory.

.. c:macro:: NORETURN

	Append this to the prototype of a function, which never returns.

.. c:macro:: BIT(n)

	Returns the bit mask ``(1UL << (n))``.

.. c:macro:: likely(exp)
.. c:macro:: unlikely(exp)

	In time critical code this gives the compiler a hint, which result of the boolean expression ``exp`` is normally expected. This can result in faster code.
