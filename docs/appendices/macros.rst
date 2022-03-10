Macros
~~~~~~

This appendix lists the C-macros, which are defined globally by the Circle build system and which can be used in applications for conditional compiling:

==============	==================================================================
Macro		Description
==============	==================================================================
__circle__	Circle version number (e.g. 440400 for Circle 44.4, patch level 0)
AARCH		ARM architecture (32 or 64)
RASPPI		Major Raspberry Pi model version (1, 2, 3 or 4)
STDLIB_SUPPORT	Standard library support level [#sl]_ (0, 1, 2 or 3)
NDEBUG		Not defined in checked builds (default)
==============	==================================================================

.. rubric:: Footnotes

.. [#sl] See: `doc/stdlib-support.txt <https://github.com/rsta2/circle/blob/master/doc/stdlib-support.txt>`_
