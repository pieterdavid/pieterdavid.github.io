.. title: Testing C++ code with pytest, through cppyy/PyROOT
.. slug: pytestforcppcode_cppyy
.. date: 2021-09-17 13:45:00 UTC+02:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

One of the things that have been extremely helpful in developing the bamboo_
package, and especially some of its C++ components, are a `collection of unit
and regression tests
<https://bamboo-hep.readthedocs.io/en/latest/hacking.html#running-the-tests-or-adding-test-cases>`_.
Of course the value of automatic or easy-to-run tests is well-known in 
software engineering, but they are not as commonly used in code used for
high-energy physics research as they could be: underlying frameworks and
critical algorithms are often covered quite well, but the code used for
final few steps to the results in a publication, and some of the code
supporting this final analysis, often much less so, or not at all.
Therefore this post is another advertisement for pytest_, especially how
it can also be used for testing C++ code through the automatic python bindings
generated by cppyy_.

For pytest_ in general I can only recommend to have a look at its documentation:
it is very easy to use, most of the time setting up a test is really as simple
as writing something like

.. code-block:: python

   def test_calculation_1():
       assert do_calculation(some_inputs) == expected_result

to a ``test_<something>.py`` file, and running ``pytest`` |---| it is also easy
to select just some tests to run with ``pytest -k 'expression'``, or by passing
the name of a test file, add helper methods in the tests, reuse an object
for several tests, or conditionally skip some tests if e.g. a dependency may be
absent, to name just a few features that are used in the bamboo_ tests.

Getting to the point of this post: when using a mix of python and C++ code,
and in case (Py)ROOT_ already used somehow (otherwise it is a rather large
dependency to add just for this), it takes very little extra effort to include
tests for the C++ code in pytest_.
The key point is that cppyy_, the underlying technology used by PyROOT_,
automatically generates python bindings for C++ code that is loaded in the
cling_ interpreter, as is described in
`this part of the PyROOT manual <https://root.cern/manual/python/#loading-user-libraries-and-just-in-time-compilation-jitting>`_.
As a first example the folling code will JIT-compile a C++ function, and use it
in a few tests:

.. code-block:: python

   import pytest
   from math import isclose

   @pytest.fixture(scope="session")
   def my_square():
       import ROOT as gbl
       gbl.gInterpreter.Declare(
           "double my_square(double a) {\n"
           "  return a*a;\n"
           "}")
       yield gbl.my_square
   
   def test_square_0(my_square):
       assert isclose(my_square(0.), 0.)

   def test_square_1(my_square):
       assert isclose(my_square(1.), 1.)

   def test_square_m3(my_square):
       assert isclose(my_square(-3.), 9.)


This is a little bit too simplistic, because we only tested the code that was
defined inside the test file, but the principle remains the same.

C++ headers can be loaded (JIT-compiled) into PyROOT_ and cling_ with
``gbl.gROOT.ProcessLine('#include "myheader.h"')``, and shared libraries with
``gbl.gSystem.Load('libname')``.
It may be useful to append include and linker paths, which can be done with
``gbl.gInterpreter.AddIncludePath`` and ``gbl.gSystem.AddDynamicPath``,
respectively (bamboo_ includes some
`helper functions <https://gitlab.cern.ch/cp3-cms/bamboo/-/blob/master/bamboo/root.py#L23-74>`_
to make this more convenient, and a
`loadDependency <https://bamboo-hep.readthedocs.io/en/latest/apiref.html#bamboo.root.loadDependency>`_
helper method that combines all of them).

It is worth stressing that, since PyROOT_ is based on cling_, the exact same
function calls make externally defined C++ code available for use in JITted
C++ strings elsewhere in ROOT_, e.g. in RDataFrame_.
It should also be possible to use standalone cppyy_ instead of PyROOT_
(and ``cppyy.include`` and ``cppyy.load_library`` instead of the ROOT methods),
but I have no experience with that.

How to compile the shared library is beyond the scope of this post, but the
main constraints are that the same C++ ABI must be used as by cling_, and the
symbols that are used must be visible (that is the default setting on GNU/Linux,
but may need extra care for cross-platform packages).

A more typical example set of unit tests for a C++ class may then look like this:

.. code-block:: python

   import pytest
   from math import isclose

   @pytest.fixture(scope="session")
   def loadMyClass():
       import ROOT as gbl
       gbl.gSystem.Load("lib/libMyClass.so")
       gbl.gInterpreter.ProcessLine('#include "include/MyClass.h"')
       yield

   @pytest.fixture(scope="module")
   def myclass_hello42(loadMyClass):
       import ROOT as gbl
       yield gbl.MyClass("hello", 42)

   def test_myclass_hello42_get(myclass_hello42):
       assert myclass_hello42.getMagic() == 42

   def test_myclass_hello42_msg(myclass_hello42):
       assert str(myclass_hello42.toString()) == "Hello, the magic number is 42"


The fixtures in pytest_ allow to efficiently implement more complicated tests.
Some more examples may be found in the
`bamboo tests <https://gitlab.cern.ch/cp3-cms/bamboo/-/tree/master/tests>`_.
An interesting case are the tests to make sure that the included C++
implementation of the recipes to calculate variations of measured jet and MET
momenta (reconstructed quantities in LHC collisions, the variations are used to
estimate the effect of imprecise knowledge of some corrections on the final
results) gives the same results as the reference implementation.
This is done by including a file with all the inputs and the outputs from the
reference implementation, and then inside the tests compare the calculated
results with the reference results.
All the code for this can be found in
`tests/test_jmesystcalc.py <https://gitlab.cern.ch/cp3-cms/bamboo/-/blob/master/tests/test_jmesystcalc.py>`_;
these calculators, and their tests, are also available as a standalone package
`pieterdavid/CMSJMECalculators <https://github.com/pieterdavid/CMSJMECalculators>`_.


.. _bamboo: https://bamboo-hep.readthedocs.io/

.. _pytest: https://docs.pytest.org/en/

.. _ROOT: https://root.cern

.. _cppyy: https://cppyy.readthedocs.io/

.. _PyROOT: https://root.cern/manual/python/

.. _cling: https://root.cern/cling/

.. _RDataFrame: https://root.cern/doc/master/classROOT_1_1RDataFrame.html

.. |---| unicode:: U+2014
   :trim:
