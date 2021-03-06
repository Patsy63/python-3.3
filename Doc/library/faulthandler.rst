:mod:`faulthandler` --- Dump the Python traceback
=================================================

.. module:: faulthandler
   :synopsis: Dump the Python traceback.

This module contains functions to dump Python tracebacks explicitly, on a fault,
after a timeout, or on a user signal. Call :func:`faulthandler.enable` to
install fault handlers for the :const:`SIGSEGV`, :const:`SIGFPE`,
:const:`SIGABRT`, :const:`SIGBUS`, and :const:`SIGILL` signals. You can also
enable them at startup by setting the :envvar:`PYTHONFAULTHANDLER` environment
variable or by using :option:`-X` ``faulthandler`` command line option.

The fault handler is compatible with system fault handlers like Apport or the
Windows fault handler. The module uses an alternative stack for signal handlers
if the :c:func:`sigaltstack` function is available. This allows it to dump the
traceback even on a stack overflow.

The fault handler is called on catastrophic cases and therefore can only use
signal-safe functions (e.g. it cannot allocate memory on the heap). Because of
this limitation traceback dumping is minimal compared to normal Python
tracebacks:

* Only ASCII is supported. The ``backslashreplace`` error handler is used on
  encoding.
* Each string is limited to 100 characters.
* Only the filename, the function name and the line number are
  displayed. (no source code)
* It is limited to 100 frames and 100 threads.

By default, the Python traceback is written to :data:`sys.stderr`. To see
tracebacks, applications must be run in the terminal. A log file can
alternatively be passed to :func:`faulthandler.enable`.

The module is implemented in C, so tracebacks can be dumped on a crash or when
Python is deadlocked.

.. versionadded:: 3.3


Dump the traceback
------------------

.. function:: dump_traceback(file=sys.stderr, all_threads=True)

   Dump the tracebacks of all threads into *file*. If *all_threads* is
   ``False``, dump only the current thread.


Fault handler state
-------------------

.. function:: enable(file=sys.stderr, all_threads=True)

   Enable the fault handler: install handlers for the :const:`SIGSEGV`,
   :const:`SIGFPE`, :const:`SIGABRT`, :const:`SIGBUS` and :const:`SIGILL`
   signals to dump the Python traceback. If *all_threads* is ``True``,
   produce tracebacks for every running thread. Otherwise, dump only the current
   thread.

.. function:: disable()

   Disable the fault handler: uninstall the signal handlers installed by
   :func:`enable`.

.. function:: is_enabled()

   Check if the fault handler is enabled.


Dump the tracebacks after a timeout
-----------------------------------

.. function:: dump_tracebacks_later(timeout, repeat=False, file=sys.stderr, exit=False)

   Dump the tracebacks of all threads, after a timeout of *timeout* seconds, or
   every *timeout* seconds if *repeat* is ``True``.  If *exit* is ``True``, call
   :c:func:`_exit` with status=1 after dumping the tracebacks.  (Note
   :c:func:`_exit` exits the process immediately, which means it doesn't do any
   cleanup like flushing file buffers.) If the function is called twice, the new
   call replaces previous parameters and resets the timeout. The timer has a
   sub-second resolution.

   This function is implemented using a watchdog thread and therefore is not
   available if Python is compiled with threads disabled.

.. function:: cancel_dump_tracebacks_later()

   Cancel the last call to :func:`dump_tracebacks_later`.


Dump the traceback on a user signal
-----------------------------------

.. function:: register(signum, file=sys.stderr, all_threads=True, chain=False)

   Register a user signal: install a handler for the *signum* signal to dump
   the traceback of all threads, or of the current thread if *all_threads* is
   ``False``, into *file*. Call the previous handler if chain is ``True``.

   Not available on Windows.

.. function:: unregister(signum)

   Unregister a user signal: uninstall the handler of the *signum* signal
   installed by :func:`register`. Return ``True`` if the signal was registered,
   ``False`` otherwise.

   Not available on Windows.


File descriptor issue
---------------------

:func:`enable`, :func:`dump_tracebacks_later` and :func:`register` keep the
file descriptor of their *file* argument. If the file is closed and its file
descriptor is reused by a new file, or if :func:`os.dup2` is used to replace
the file descriptor, the traceback will be written into a different file. Call
these functions again each time that the file is replaced.


Example
-------

Example of a segmentation fault on Linux: ::

    $ python -q -X faulthandler
    >>> import ctypes
    >>> ctypes.string_at(0)
    Fatal Python error: Segmentation fault

    Current thread 0x00007fb899f39700:
      File "/home/python/cpython/Lib/ctypes/__init__.py", line 486 in string_at
      File "<stdin>", line 1 in <module>
    Segmentation fault

