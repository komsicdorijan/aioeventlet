aiogreen
========

aiogreen implements the asyncio API on top of eventet. It makes possible to
write asyncio code in a project currently written for eventlet.

aiogreen allows to use greenthreads in asyncio coroutines, and to use asyncio
coroutines, tasks and futures in greenthreads: see :func:`link_future` and
:func:`wrap_greenthread` functions.

The main visible difference between trollius and aiogreen is that
``run_forever()``: ``run_forever()`` blocks with trollius, whereas it runs in a
greenthread with aiogreen. It means that it's possible to call
``run_forever()`` in the main thread and execute other greenthreads in
parallel.

aiogreen:

* `aiogreen documentation <http://aiogreen.readthedocs.org/>`_
* `aiogreen project in the Python Cheeseshop (PyPI)
  <https://pypi.python.org/pypi/aiogreen>`_
* `aiogreen project at Bitbucket <https://bitbucket.org/haypo/aiogreen>`_
* Copyright/license: Open source, Apache 2.0. Enjoy!

Event loops:

* `asyncio documentation <http://docs.python.org/dev/library/asyncio.html>`_
* `trollus documentation <http://trollius.readthedocs.org/>`_
* `eventlet documentation <http://eventlet.net/doc/>`_
* `eventlet project <http://eventlet.net/>`_
* `greenlet documentation <http://greenlet.readthedocs.org/>`_
* `Tulip project <http://code.google.com/p/tulip/>`_


Usage
=====

Use aiogreen with trollius
-------------------------

To support Python 2, you can use Trollius which uses ``yield`` instead
of ``yield from`` for coroutines:
http://trollius.readthedocs.org/

To use aiogreen with trollius, set the event loop policy before using an event
loop, example::

    import aiogreen
    import trollius

    trollius.set_event_loop_policy(aiogreen.EventLoopPolicy())
    # ....

Hello World::

    import aiogreen
    import trollius as asyncio

    def hello_world():
        print("Hello World")
        loop.stop()

    asyncio.set_event_loop_policy(aiogreen.EventLoopPolicy())
    loop = asyncio.get_event_loop()
    loop.call_soon(hello_world)
    loop.run_forever()
    loop.close()


Use aiogreen with asyncio
-------------------------

aiogreen implements the asyncio API, see asyncio documentation:
https://docs.python.org/dev/library/asyncio.html

eventlet 0.15 supports Python 3 if monkey-patching is not used.

To use aiogreen with asyncio, set the event loop policy before using an event
loop, example::

    import aiogreen
    import asyncio

    asyncio.set_event_loop_policy(aiogreen.EventLoopPolicy())
    # ....

Adding this code should be enough to try examples of the asyncio documentation.

Hello World::

    import aiogreen
    import asyncio

    def hello_world():
        print("Hello World")
        loop.stop()

    asyncio.set_event_loop_policy(aiogreen.EventLoopPolicy())
    loop = asyncio.get_event_loop()
    loop.call_soon(hello_world)
    loop.run_forever()
    loop.close()


API
===

aiogreen specific functions:

.. function:: link_future(future)

   Wait for a future (or a task) from a greenthread.
   Return the result or raise the exception of the future.

   Example with asyncio::

       def coro_slow_sum(x, y):
           yield from asyncio.sleep(1.0)
           return x + y

       def green_sum():
           task = asyncio.async(coro_slow_sum(1, 2))
           value = aiogreen.link_future(task)
           return value

.. function:: wrap_greenthread(gt)

   Wrap a greenthread into a Future object.

   The Future object waits for the completion of a greenthread.

   Example with asyncio::

       def slow_sum(x, y):
           eventlet.sleep(1.0)
           return x + y

       @asyncio.coroutine
       def coro_sum():
           gt = eventlet.spawn(slow_sum, 1, 2)
           fut = aiogreen.wrap_greenthread(gt, loop=loop)
           result = yield from fut
           return result

   .. note::
      If the debug mode of event loop is set, when a greenthread raises an
      exception, the exception is logged to ``sys.stderr`` by eventlet, even if
      the exception is copied to the Future object as expected.


Installation
============

Requirements:

- eventlet 0.14 or newer
- asyncio or trollius:

  * Python 3.4 and newer: asyncio is now part of the stdlib
  * Python 3.3: need Tulip 0.4.1 or newer (pip install asyncio),
    but Tulip 3.4.1 or newer is recommended
  * Python 2.6-3.2: need Trollius 0.3 or newer (pip install trollius),
    but Trollius 1.0 or newer is recommended

Type::

    pip install aiogreen

or::

    python setup.py install


Run tests
=========

Run tests with tox
------------------

The `tox project <https://testrun.org/tox/latest/>`_ can be used to build a
virtual environment with all runtime and test dependencies and run tests
against different Python versions (2.6, 2.7, 3.2, 3.3).

For example, to run tests with Python 2.7, just type::

    tox -e py27

To run tests against other Python versions:

* ``py26``: Python 2.6
* ``py27``: Python 2.7
* ``py27_patch``: Python 2.7 with eventlet monkey patching
* ``py32``: Python 3.2
* ``py33``: Python 3.3
* ``py34``: Python 3.4

Run tests manually
------------------

Run the following command from the directory of the aiogreen project:

    python runtests.py -r


Changelog
=========

Version 0.2 (development version)
---------------------------------

The core of the event loop was rewritten to fits better in asyncio and
eventlet. aiogreen now reuses more code from asyncio/trollius. The code
handling file descriptors was also fixed to respect asyncio contract:
only call the callback once per loop iteration.

Changes:

* Add a Sphinx documentation published at http://aiogreen.readthedocs.org/
* Add the :func:`link_future` function: wait for a future from a
  greenthread.
* Add the :func:`wrap_greenthread` function: wrap a greenthread into a Future
* Support also eventlet 0.14, not only eventlet 0.15 or newer
* Support eventlet with monkey-patching
* Rewrite the code handling file descriptors to ensure that the listener is
  only called once per loop iteration, to respect asyncio specification.
* Simplify the loop iteration: remove custom code to reuse instead the
  asyncio/trollius code (_run_once)
* Reuse call_soon, call_soon_threadsafe, call_at, call_later from
  asyncio/trollius, remove custom code
* sock_connect() is now asynchronous
* Add a suite of automated unit tests
* Fix EventLoop.stop(): don't stop immediatly, but schedule stopping the event
  loop with call_soon()
* Add tox.ini to run tests with tox
* Setting debug mode of the event loop doesn't enable "debug_blocking" of
  eventlet on Windows anymore, the feature is not implemented on Windows
  in eventlet.
* add_reader() and add_writer() now cancels the previous handle and sets
  a new handle
* In debug mode, detect calls to call_soon() from greenthreads which are not
  threadsafe (would not wake up the event loop).
* Only set "debug_exceptions" of the eventlet hub when the debug mode of the
  event loop is enabled.

2014-11-19: version 0.1
-----------------------

* First public release


Implemented
===========

Methods:

* call_at()
* call_later()
* call_soon()
* run_forever()
* run_in_executor()
* run_until_complete()
* create_connection(): TCP client
* stop()
* coroutines and tasks

Tests of aiogreen 0.1:

* Tested on Python 2.7, 3.3 and 3.5
* Tested on Linux and Windows
* Tested with Trollius 1.0, 1.0.1 and 1.0.2
* Tested with asyncio 0.4.1 and 3.4.2


To do (Not supported)
=====================

* add_reader() does only support one callback per file descriptor currently.
* run an event loop in a thread different than the main thread
* sockets: create_server, sock_recv
* pipes: connect_read_pipe
* subprocesses: need pipes
* signal handlers: add_signal_handler (only for pyevent hub?)
* tox.ini: add py33_patch. eventlet with Python 3 and monkey-patch causes
  an issue in importlib.


eventlet issues
===============

* eventlet monkey patching on Python 3 is incomplete. The most blocking issue
  is in the importlib: the thread module is patched to use greenthreads, but
  importlib really need to work on real threads. Pull request:
  https://github.com/eventlet/eventlet/pull/168
* eventlet.tpool.setup() seems to be broken on Windows in eventlet 0.15.
  Pull request:
  https://github.com/eventlet/eventlet/pull/167
* hub.debug_blocking is implemented with signal.alarm() which is is not
  available on Windows.


eventlet and Python 3
=====================

Issues:

* https://github.com/eventlet/eventlet/issues/6 (root py3 issue)
* https://github.com/eventlet/eventlet/issues/157 (py3 related?)
* https://github.com/eventlet/eventlet/issues/153 (py3 related?)

Pull requests:

* https://github.com/eventlet/eventlet/pull/99 : complete monkey-patching
* => commit: https://github.com/therve/eventlet/commit/9c3118162cf1ca1e50be330ba2a289f054c48d3c
* https://github.com/eventlet/eventlet/pull/160 (py3 related?)

OpenStack Kilo Summit:

* https://etherpad.openstack.org/p/kilo-oslo-python-3
* https://etherpad.openstack.org/p/kilo-oslo-oslo.messaging
* https://etherpad.openstack.org/p/py34-transition (tangentially related)