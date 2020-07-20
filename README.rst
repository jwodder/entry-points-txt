.. image:: http://www.repostatus.org/badges/latest/wip.svg
    :target: http://www.repostatus.org/#wip
    :alt: Project Status: WIP — Initial development is in progress, but there
          has not yet been a stable, usable release suitable for the public.

.. image:: https://travis-ci.com/jwodder/entry-points-txt.svg?branch=master
    :target: https://travis-ci.com/jwodder/entry-points-txt

.. image:: https://codecov.io/gh/jwodder/entry-points-txt/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/jwodder/entry-points-txt

.. image:: https://img.shields.io/github/license/jwodder/entry-points-txt.svg
    :target: https://opensource.org/licenses/MIT
    :alt: MIT License

`GitHub <https://github.com/jwodder/entry-points-txt>`_
| `Issues <https://github.com/jwodder/entry-points-txt/issues>`_

``entry-points-txt`` provides functions for reading & writing
``entry_points.txt`` files according to `the spec`_.  That is the one thing it
does, and it endeavors to do it well.

.. _the spec: https://packaging.python.org/specifications/entry-points/

Installation
============
``entry-points-txt`` requires Python 3.6 or higher.  Just use `pip
<https://pip.pypa.io>`_ for Python 3 (You have pip, right?) to install
``entry-points-txt``::

    python3 -m pip install git+https://github.com/jwodder/entry-points-txt.git


API
===

``EntryPoint``
--------------

.. code:: python

    class EntryPoint(NamedTuple)

A representation of an entry point as a namedtuple.  Instances have the
following attributes and methods:

``group: str``
   The name of the entry point group (e.g., ``"console_scripts"``)

``name: str``
   The name of the entry point

``module: str``
   The module portion of the object reference (the part before the colon)

``object: Optional[str]``
   The object/attribute portion of the object reference (the part after the
   colon), or ``None`` if not specified

``extras: Tuple[str, ...]``
   Extras required for the entry point

``load() -> Any``
   Returns the object referred to by the entry point's object reference

``to_line() -> str``
   Returns the representation of the entry point as a line in
   ``entry_points.txt``, i.e., a line of the form ``name = module:object
   [extras]``

``load()``
----------

.. code:: python

    entry_points_txt.load(fp: IO[str]) -> Dict[str, Dict[str, EntryPoint]]

Parse a file-like object as an ``entry_points.txt``-format file and return the
results.  The parsed entry points are returned in a ``dict`` mapping each group
name to a ``dict`` mapping each entry point name to an ``EntryPoint`` object.

For example, the following input:

.. code:: ini

    [console_scripts]
    foo = package.__main__:main
    bar = package.cli:klass.attr

    [thingy.extension]
    quux = package.thingy [xtr]

would be parsed as:

.. code:: python

    {
        "console_scripts": {
            "foo": EntryPoint(group="console_scripts", name="foo", module="package.__main__", object="main", extras=()),
            "bar": EntryPoint(group="console_scripts", name="bar", module="package.cli", object="klass.attr", extras=()),
        },
        "thingy.extension": {
            "quux": EntryPoint(group="thingy.extension", name="quux", module="package.thingy", object=None, extras=("xtr",)),
        },
    }

``loads()``
-----------

.. code:: python

    entry_points_txt.loads(s: str) -> Dict[str, Dict[str, EntryPoint]]

Like ``load()``, but reads from a string instead of a filehandle

``dump()``
----------

.. code:: python

    entry_points_txt.dump(eps: Dict[str, Dict[str, EntryPoint]], fp: IO[str]) -> None

Write a collection of entry points (in the same format as returned by
``load()``) to a file-like object

``dumps()``
-----------

.. code:: python

    entry_points_txt.dumps(eps: Dict[str, Dict[str, EntryPoint]]) -> str

Like ``dump()``, but returns a string instead of writing to a filehandle

``dump_list()``
---------------

.. code:: python

    entry_points_txt.dump_list(eps: Iterable[EntryPoint], fp: IO[str]) -> None

Write an iterable of entry points to a file-like object

``dumps_list()``
----------------

.. code:: python

    entry_points_txt.dumps_list(eps: Iterable[EntryPoint]) -> str

Like ``dump_list()``, but returns a string instead of writing to a filehandle

``ParseError``
--------------

.. code:: python

    class ParseError(ValueError)

Exception raised by ``load()`` or ``loads()`` when given invalid input