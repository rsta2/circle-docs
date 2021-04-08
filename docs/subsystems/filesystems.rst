Filesystems
~~~~~~~~~~~~

Circle provides two different implementations for FAT filesystem support:

* A native filesystem subsystem with FAT16 and FAT32 support using C++ classes, which has several limitations: short filenames (8.3) only, access to the root directory only, sequential file access only, subset of common file operations only, relatively slow
* A port of `FatFs - Generic FAT Filesystem Module <http://elm-chan.org/fsw/ff/00index_e.html>`_ (by ChaN), written in C, but with full FAT filesystem support and much faster

CFATFileSystem
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/fs/fat/fatfs.h>

.. cpp:class:: CFATFileSystem

	This class provides the API of the native FAT filesystem support in Circle, which has several limitations (see above). If you want to use this variant, you should instantiate this class as a member of the class ``CKernel``.

.. cpp:function:: int CFATFileSystem::Mount (CDevice *pPartition)

	Mounts the block device ``pPartition`` as FAT filesystem. Returns non-zero on success. A pointer to the block device can be requested using ``CDeviceNameService::GetDevice()``. This method is usually invoked with the partition name ``"emmc1-1"`` (first partition of the SD card) or ``"umsd1-1"`` (first partition of an USB mass-storage device, e.g. USB flash drive) for this purpose.

.. cpp:function:: void CFATFileSystem::UnMount (void)

	Un-mounts the filesystem.

.. cpp:function:: void CFATFileSystem::Synchronize (void)

	Flushes the buffer cache, without un-mounting the filesystem.

.. cpp:function:: unsigned CFATFileSystem::RootFindFirst (TDirentry *pEntry, TFindCurrentEntry *pCurrentEntry)

	Finds the first file entry in the root directory and returns non-zero, if it was found. ``pEntry`` is a pointer to a buffer, which receives the information about the found file. ``pCurrentEntry`` points to the current entry variable, which must be maintained, until the directory scan has been completed. ``TDirentry`` is defined as follows:

.. code-block:: cpp

	#define FS_TITLE_LEN		12	// length of a file name (may include a dot)

	struct TDirentry
	{
		char		chTitle[FS_TITLE_LEN+1];	// 0-terminated
		unsigned	nSize;				// number of bytes
		unsigned	nAttributes;
	#define FS_ATTRIB_NONE		0x00	// no attribute set
	#define FS_ATTRIB_SYSTEM	0x01	// HIDDEN attribute set
	#define FS_ATTRIB_EXECUTABLE	0x02	// file is executable (always set for FAT)
	};

.. cpp:function:: unsigned CFATFileSystem::RootFindNext (TDirentry *pEntry, TFindCurrentEntry *pCurrentEntry)

	Finds the next file entry in the root directory and returns non-zero, if it was found, or zero, if there are no more entries. ``pEntry`` is a pointer to a buffer, which receives the information about the found file. ``pCurrentEntry`` points to the current entry variable, which must be maintained, until the directory scan has been completed. See ``RootFindFirst()`` for the definition of TDirentry.

.. cpp:function:: unsigned CFATFileSystem::FileOpen (const char *pTitle)

	Opens the file with the filename ``pTitle`` (8.3 name without path, may include a dot) for read. Returns the file handle or zero on failure.

.. cpp:function:: unsigned CFATFileSystem::FileCreate (const char *pTitle)

	Creates a new file with the filename ``pTitle`` (8.3 name without path, may include a dot) for write. Returns the file handle or zero on failure (e.g. read-only file with this name exists).

.. warning::

	This method truncates the file, if it already exists with the filename ``pTitle``.

.. cpp:function:: unsigned CFATFileSystem::FileClose (unsigned hFile)

	Closes the file with the file handle ``hFile``. This handle has been returned by a previous call to ``FileOpen()`` or ``FileCreate()``. Returns non-zero on success.

.. cpp:function:: unsigned CFATFileSystem::FileRead (unsigned hFile, void *pBuffer, unsigned nCount)

	Reads sequentially up to ``nCount`` bytes from the file with file handle ``hFile`` into ``pBuffer``. Returns the number of bytes read, zero when the end of file has been reached, or FS_ERROR on general failure (e.g. invalid parameter).

.. cpp:function:: unsigned CFATFileSystem::FileWrite (unsigned hFile, const void *pBuffer, unsigned nCount)

	Writes sequentially ``nCount`` bytes from ``pBuffer`` to the file with file handle ``hFile``. Returns the number of bytes written, or FS_ERROR on general failure (e.g. invalid parameter).

.. cpp:function:: int CFATFileSystem::FileDelete (const char *pTitle)

	Deletes the file with the name ``pTitle`` from the root directory. Returns a positive value on success, zero, if the file was not found, or a negative value, if the file has the read-only attribute.

FatFs library
^^^^^^^^^^^^^

The `FatFs - Generic FAT Filesystem Module <http://elm-chan.org/fsw/ff/00index_e.html>`_ (by ChaN) has been ported to Circle, to provide a full function support for the FAT filesystem. The related files can be found in the subdirectory `addon/fatfs`. The associated sample program demonstrates some basic features of FatFs. Please see the subsection "Application Interface" on this website for a description of the different functions of this library.

The Circle port of FatFs supports the following volume ID strings for logical drives:

======	======	======================	==============================
ID	Drive	Partition		Device
======	======	======================	==============================
SD:	0:	first FAT partition	SD card
USB:	1:	first FAT partition	first USB mass-storage device
USB2:	2:	first FAT partition	second USB mass-storage device
USB3:	3:	first FAT partition	third USB mass-storage device
======	======	======================	==============================

.. important::

	FatFs may support the exFAT filesystems too. This support has been disabled in Circle for legal reasons. You have to read the subsection "exFAT Filesystem" on the page `FatFs Module Application Note <http://elm-chan.org/fsw/ff/doc/appnote.html#exfat>`_ first, if you want to use exFAT support! This may require a license fee.
