#######################
DM Pybind11 Style Guide
#######################

This is the DM Pybind11 Coding Standard.

Changes to this document must be approved by the System Architect (`RFC-24 <https://jira.lsstcorp.org/browse/RFC-24>`_).
To request changes to these standards, please file an :doc:`RFC </communications/rfc>`.

.. contents::
    :depth: 4

.. _style-guide-pybind11-modules-and-source-files:

Introduction
============

This document lists pybind11 coding recommendations.
The recommendations are based on conventions in upstream pybind11 and best practices developed within DM.

.. _style-guide-pybind11-intro-vocab:

Recommendation Importance
-------------------------

In the guideline sections, the terms **required**, **must**, **should**, amongst others, have special meaning.
Refer to :ref:`Stringency Level <style-guide-rfc-2119>` reference.
DM uses the spirit of the IETF organization's `RFC 2199 Reference <http://www.ietf.org/rfc/rfc2119.txt>`_ definitions.

General
=======

.. _style-guide-pybind11-cpp-rules:

All rules from the DM C++ style guide SHALL also apply to pybind11 wrappers
---------------------------------------------------------------------------

Fundamentally pybind11 wrappers are just C++. Therefore the rules from the DM C++ Style Guide also apply, unless they are in direct conflict with one of the other rules in this guide.

.. _style-guide-pybind11-py-rules:

All rules from the DM Python style guide SHALL also apply to pybind11 wrappers
------------------------------------------------------------------------------

The generated extension modules are used from Python and should closely resemble Python types.
Therefore the rules from the DM Python Style Guide also apply, unless they are in direct conflict with one of the other rules in this guide.

This applies in particular to pure Python code sections, except in cases where deviation is explicitly allowed.

Two rules in the Python coding guide (inherited from PEP8) are particularly relevant to pybind11-related wrappers:

 - ``from <module> import *`` should only be used in ``__init__.py`` modules that "lift" symbols to package level (and contain no other code).

 - ``__all__`` should be defined by any module whose symbols will be thusly lifted.

.. _style-guide-pybind11-deprecation:

Deprecated wrapped functions SHOULD be re-wrapped using the ``deprecated`` decorator
------------------------------------------------------------------------------------

For more on deprecating code, see :doc:`/stack/deprecating-interfaces`.

Modules and source files
========================

.. _style-guide-pybind11-module-naming:

C++ wrapper code for a package SHOULD go in a module and source file named by adding an underscore prefix to the package name
-----------------------------------------------------------------------------------------------------------------------------

For example, the Python wrapper module for ``lsst.geom`` should be ``_geom``, defined (at least in part) in a source file ``_geom.cc``.

.. _style-guide-pybind11-cxx-source:

Wrappers for a C++ header file SHOULD go in a source file with the same name as the header and a leading underscore
-------------------------------------------------------------------------------------------------------------------

For example, C++ code wrapping ``LinearTransform.h`` should be in a source file ``_LinearTransform.cc``.

If a group of headers together provide functionality that cannot be used independently, they may be wrapped into a single module.
The headers wrapped by such a module must be prominently listed in a comment near the top of the source file.

Very small packages MAY put all wrapper code in the ``_<package>.cc`` file instead of using multiple source files.

.. _style-guide-pybind11-python-source:

Additional pure-Python wrappers for a header SHOULD go in a module with the same name as the header and a leading underscore
----------------------------------------------------------------------------------------------------------------------------

For example, for header file ``LinearTransform.``, we would have::

    _LinearTransform.cc:
        <C++ wrappers>

    _LinearTransform.py:
        <Python extensions to the wrappers>

    _geom.cc:
        <C++ module definition, calls into _LinearTransform.cc>

    __init__.py:
        <package definition, imports _geom.cc and _LinearTransform.py>

.. _style-guide-pybind11-cpp-vs-python:

Trivial extensions to wrappers SHOULD be implemented in C++
-----------------------------------------------------------

Simple extensions such as ``__repr__`` or ``__reduce__`` should be implemented via lambdas in compiled modules, utilizing the pybind11 Python C++ API (e.g. ``pybind11::object``) as necessary.

Longer extensions that involve significant logic or language constructs difficult to implement using the C++ Python API (e.g. generators) should go in pure-Python files.

This rule applies regardless of whether a pure-Python extension module already exists; this prevents the correct code organization from becoming a function of history.

Using pure-Python modules only when necessary minimizes the number of source files and helps keep class definitions together.

.. _style-guide-pybind11-include:

Pybind11 headers should precede all other headers in the include ordering
-------------------------------------------------------------------------

``pybind11.h`` includes ``Python.h`` and `must hence be included before all other headers <https://docs.python.org/3/c-api/intro.html#include-files>`_.
To keep a reasonable grouping, all other pybind11 headers should be included in this same include block.

.. _style-guide-pybind11-cross-module-code-location:

Wrapper code shared across modules MUST be placed in a python.h file (or subdirectory) in the relative include path
-------------------------------------------------------------------------------------------------------------------

For example, common code to wrap ``lsst::afw::table`` shall go either into::

    include/lsst/afw/table/python.h

or::

    include/lsst/afw/table/python/myname.h

When multiple headers are added to a ``python`` subdirectory, in general we SHOULD NOT add an aggregating ``python.h`` file; the presence of such a file encourages including more headers than are actually needed, leading to slower compilation times.

.. seealso::

    :ref:`The namespace rules. <style-guide-pybind11-common-code-namespace>`

.. _style-guide-pybind11-cross-package-code-location:

Wrapper code shared across packages SHOULD go into utils
--------------------------------------------------------

More specifically it SHOULD go into ``include/lsst/utils/python/*.h`` in the ``utils`` package.

The only exception should be utility code that depends on other code that is not already in utils' dependency tree.

Naming conventions
==================

.. _style-guide-pybind11-alias:

The pybind11 namespace MUST be aliased to ``py`` in source files
----------------------------------------------------------------

All pybind11 wrapper modules should include:

.. code-block:: cpp

    namespace py = pybind11;

This alias MUST NOT be defined at namespace scope in header files (see :ref:`C++ rule 4-13 <style-guide-cpp-4-13>`), though it MAY be defined locally within functions in headers.  For example:

.. code-block:: cpp

    #include "pybind11/pybind11.h"

    namespace py = pybind11;  // required in .cc, not allowed in .h

    namespace lsst { namespace afw { namespace geom { namespace {

    void declareFunctions(py::module & mod) {
        namespace py = pybind11; // okay in .h, unnecessary in .cc
        ...
    }

    }}}} // namespace lsst::afw::geom::<anonymous>

.. _style-guide-pybind11-module-prefix:

Module object names MUST be "mod" or camel case prefixed with "mod"
-------------------------------------------------------------------

If a wrapper only contains one module instance the name of the object shall be ``mod``.
Otherwise (e.g. if another module is imported into a local variable) it shall be camel case prefixed with ``mod`` as in ``modExample``.

.. _style-guide-pybind11-class-prefix:

Class object names MUST be "cls" or camel case prefixed with "cls"
------------------------------------------------------------------

If a wrapper only wraps one class the name of the pybind11 class object shall be ``cls``.
Otherwise it shall be camel case prefixed with ``cls`` as in ``clsExample``.

When a wrapper wraps multiple classes it is recommended you define a separate function to wrap each class.
Each wrapper function takes the module as an argument and uses ``cls`` as the variable name for the pybind11 class object.

When using a ``cls`` prefix, it is **strongly** encouraged to use the full class name for the remainder.
However you MAY also use an abbreviated name.

.. _style-guide-pybind11-method-chaining:

Method chaining MAY be used to increase code readability
--------------------------------------------------------

When a named class object is not needed, chaining methods can reduce boilerplate.

For example:

.. code-block:: cpp

    py::class_<Example>(mod, "Example")
        .def("foo", &Example::foo)
        .def("bar", &Example::bar);

This syntax is essentially always used with ``enum`` (see :ref:`enum syntax <style-guide-pybind11-enum-scoping>`).

.. _style-guide-pybind11-lambda-self-argument:

Lambda arguments referring to the current object MUST be named "self"
---------------------------------------------------------------------

For example:

.. code-block:: cpp

    clsExample.def("f", [](Example const & self, ... ) { ... });

.. _style-guide-pybind11-lambda-other-argument:

Lambda arguments referring to the second object in a copy constructor or operator wrapper MUST be named "other"
---------------------------------------------------------------------------------------------------------------

For example:

.. code-block:: cpp

    clsExample.def("__eq__", [](Example const & self, Example const & other) { ... });

.. _style-guide-pybind11-class-alias:

Names of generic class types SHOULD be called "Class"
-----------------------------------------------------

It is sometimes desirable to give a class type a generic name (either as ``typename``, ``typedef`` or ``using`` alias).
In such cases prefer to call the type ``Class``.
This is especially common in :ref:`declare functions <style-guide-pybind11-declare-template-wrappers>`.

.. _style-guide-pybind11-class-object-alias:

Names of generic pybind11 class types SHOULD be called "PyClass"
----------------------------------------------------------------

When a generic type name or alias refers to a ``pybind11::class_<Ts...>`` object prefer to call it ``PyClass``.
This is especially again common in :ref:`declare functions <style-guide-pybind11-declare-template-wrappers>`.

Organization
============

.. _style-guide-pybind11-wrap-split-modules:

Wrappers for a header SHALL be implemented in functions prefixed with "wrap"
----------------------------------------------------------------------------

Wrappers for ``LinearTransform`` (declared in ``LinearTransform.h``, with wrappers in ``_LinearTransform.cc``) shall go in a function called ``wrapLinearTransform``.

Whenever possible, this function should take a single ``lsst::utils::python::WrapperCollection &`` argument, return void, and be called only within the ``PYBIND11_MODULE`` block for the package.

When multiple headers are wrapped in a single source file, that source file must define a single "wrap" function with a name related to that of the source file.

.. _style-guide-pybind11-declare-template-wrappers:

Wrappers for templates SHALL be declared in functions prefixed with "declare"
-----------------------------------------------------------------------------

The wrapper for the templated type ``Example<T>`` shall be added by a declare function:

.. code-block:: cpp

    namespace {
        template <typename T>
        void declareExample(py::module & mod, std::string const & suffix) {
            using Class = Example<T>;
            py::class<Class, std::shared_ptr<Class>> cls(mod, ("Example" + suffix).c_str());

            cls.def("test", &Class::test);
            ...
        }
    }

    ...

    void wrapExample(utils::python::WrapperCollection & wrappers) {
        declareExample<float>(wrappers, "F");
        declareExample<int>(wrappers, "I");
        ...
    }

The return type may be non-void in case more functionality needs to be added later.
The suffix argument may be omitted when not needed (e.g. when adding function overloads).

.. _style-guide-pybind11-declare-usage:

Separate declare functions MAY also be used to avoid code duplication and increase readability
----------------------------------------------------------------------------------------------

In some cases it is useful to split up wrapping over multiple (non-templated) declare functions.
For instance when multiple classes are defined in a single module, or when classes share many related methods.

For example:

.. code-block:: cpp

    template <typename Class, typename PyClass>
    void declareCommon(PyClass & cls) {
        cls.def("read", &Class::read);
    }

    void declareFoo(py::module & mod) {
        py::class_<Foo> cls(mod, "Foo");

        declareCommon<Foo>(cls);
    }

    void declareBar(py::module & mod) {
        py::class_<Bar> cls(mod, "Bar");

        declareCommon<Bar>(cls);
    }

.. _style-guide-pybind11-common-code-namespace:

Common wrapper code in headers MUST be placed in the nested python namespace
----------------------------------------------------------------------------

For example:

.. code-block:: cpp

    namespace lsst {
    namespace sphgeom {
    namespace python {

    ...  // declare functions...

    }  // namespace python
    }  // namespace sphgeom
    }  // namespace lsst

.. _style-guide-pybind11-class-object-dupplication:

``py::class_`` instantiations MUST be declared only once
--------------------------------------------------------

Because ``py::class_`` objects take many template arguments (which may change), an instantiation for a C++ type must be declared in exactly one place.
If this type must appear in places other than the declaration of ``py::class_`` instance, such as a ``declare`` function, a type alias or template type deduction should be used to avoid repeating the full ``py::class_`` type.

When no template deduction is needed, a type alias is usually preferable:

.. code-block:: cpp

    using PyThing = py::class_<Thing>;

    declareCommon(PyThing & cls) {
        ...
    }

    PYBIND11_MODULE(_Thing, mod) {
        PyThing cls(...);
        declareThingMethods(cls);
    }

If template deduction is used, it should be used on the full type, not the template parameters for ``py::class_`` itself:

.. code-block:: cpp

    template <typename PyClass>
    declareCommon(PyClass & cls) {
        ...
    }

    PYBIND11_MODULE(_Thing, mod) {
        py::class_<Thing> cls(...);
        declareCommon(cls);
    }

There should be no need to provide the template parameters explicitly when calling ``declareCommon`` here; they are inferred from the type passed to it.

Use of pybind11 features
========================

.. _style-guide-pybind11-overload-disambiguation:

overload_cast SHALL be used to disambiguate overloads
----------------------------------------------------------------------

Example:

.. code-block:: cpp

    // overloaded function
    mod.def("test", py::overload_cast<int>(test));
    mod.def("test", py::overload_cast<double>(test));

    // overloaded class member function
    cls.def("computeSomething",
            py::overload_cast<int, double>(&MyClass::computeSomething, py::const_),
            "firstParam"_a, "anotherParam"_a);
    cls.def("computeSomething",
            py::overload_cast<int, std::string>(&MyClass::computeSomething, py::const_),
            "firstParam"_a, "anotherParam"_a="foo");

Note that ``py::const_`` is necessary for a const member function.

.. _style-guide-pybind11-holder-type:

The shared_ptr holder type SHOULD be used for all non-trivial classes
---------------------------------------------------------------------

By not specifying a holder type explicitly it becomes ``unique_ptr``, but it is hard to anticipate when wrapping a class whether any downstream code will later use it with ``shared_ptr``.
Moreover, C++ functions taking ``unique_ptr`` arguments can never be wrapped intuitively in Python (because Python has no output arguments or ownership transfer), so we do not need to worry about wrapped instances held by ``shared_ptr`` that must be converted to ``unique_ptr`` for a function call.

The only classes that should be wrapped with ``unique_ptr`` are non polymorphic classes that are always passed by value or reference in C++ and are small enough that ``shared_ptr`` represents a significant overhead.

Note that this does not mean that ``shared_ptr`` must be used in C++ code in preference to other options; the :ref:`C++ coding guidelines on when to use them <style-guide-cpp-5-24b>` still apply.

.. _style-guide-pybind11-keyword-arguments:

Keyword names MUST be provided for functions with multiple overloads or more than two arguments
-----------------------------------------------------------------------------------------------

Keyword arguments make Python code significantly more readable, especially when distinguishing between overloads or in long function signatures.

Keyword arguments MAY be provided for non-overloaded functions with two or fewer arguments, and are strongly encouraged if the meaning or order of the arguments is not apparent from the function name.

.. _style-guide-pybind11-keyword-argument-literals:

Literals MUST be used for all named arguments
----------------------------------------------

The ``_a`` argument literal from ``pybind11::literals`` MUST be used for all named arguments (e.g. ``mod.def("f", f, "arg1"_a, "arg2"_a);``).
The ``py::arg()`` construct SHALL NOT be used.

.. _style-guide-pybind11-enum-scoping:

Enum scoping SHALL follow usage in C++
--------------------------------------

* Unscoped enums SHALL export their names into the class scope using ``.export_values``:

.. code-block:: cpp

    py::enum_<Class::State>(cls, "State")
        .value("RED", &Class::State::RED)
        .value("GREEN", &Class::State::GREEN)
        .export_values();

* Scoped enums (i.e. ``enum class`` in C++) SHALL NOT use ``.export_values``.

.. _style-guide-pybind11-enums-as-integers:

Enums used as integers SHALL be wrapped as integer attributes
-------------------------------------------------------------

Regular (non-class) enums are frequently used in C++ to define a set of related integer constants rather than an actual enumeration.
Enums whose values are defined to be distinct bits (e.g. ``0x01``, ``0x02``, ``0x04``) are almost certainly used only as integer constants.

These enums should be wrapped as simple integer class attributes rather than pybind11 enums, e.g.::

    cls.attr("NAME1") = py::cast(int(Class::NAME1));
    cls.attr("NAME2") = py::cast(int(Class::NAME2));

This avoids a need for casts in Python code to deal with the fact that pybind11 enumerations are not implicitly convertible to ``int`` (unlike C++).
Anonymous enums or enums with explicit values that are usable in bitwise operations should almost always be wrapped as integer attributes.

All other enums (those that are not used as a collection of integer constants) SHOULD be wrapped with ``py::enum_``.

.. _style-guide-pybind11-arithmetic-enum:

Enums that have a natural ordering SHALL use py::arithmetic
-----------------------------------------------------------

If enums exposed to Python have a natural ordering, and hence can be expected to be used in comparisons, ``py::enum_<ExampleEnum>(..., py::arithmetic())`` SHALL be used (instead of either not having comparison operators or wrapping them explicitly).

.. _style-guide-pybind11-virtual-methods:

Derived-class overrides of virtual methods MUST be wrapped OR noted with a comment
----------------------------------------------------------------------------------

Because C++ polymorphism ensures the right C++ implementation is always called, only the base class version of a virtual method strictly needs to be wrapped to get the right behavior.
And in some cases not wrapping a derived-class override can represent a significant reduction in code duplication.
But within a pybind11 file it is hard to identify which methods are virtual, and the absence of a method in wrappers is potentially confusing unless a comment indicates that the method is not wrapped because it is an override.

.. _style-guide-pybind11-stl-containers:

Default automatic conversions SHALL be used for all STL containers
------------------------------------------------------------------

The pybind11 header ``pybind11/stl.h`` provides automatic conversion support (to standard Python ``list``, ``set``, ``tuple`` and ``dict`` types) for most STL containers (i.e. ``std::vector``, ``std::set``, ``std::unordered_set``, ``std::pair``, ``std::tuple``, ``std::list``, ``std::map`` and ``std::unordered_map``).
These conversions shall always be used instead of manual wrapping.

Manual wrapping of a standard library type is not a local operation: defining such a wrapper can break code in other modules that use the same type but expect it to be returned to Python as a native Python container.

.. _style-guide-pybind11-stl-containers-alternative:

Where copying of STL containers is undesirable an ndarray type SHOULD be used instead
-------------------------------------------------------------------------------------

The ``ndarray`` C++ types can share storage with NumPy arrays.
This may sometimes require changes to the C++ API.

.. _style-guide-pybind11-operator:

Default operator support SHALL NOT be used
------------------------------------------

Support from the ``pybind11/operators.h`` header cannot be applied consistently and SHALL NOT be used.

Instead all operators are to be wrapped either directly as any other function:

.. code-block:: cpp

    clsExample.def("__eq__", &Example::operator==, py::is_operator());

or using a lambda function:

.. code-block:: cpp

    clsExample.def("__eq__", [](Example const & self, Example const & other) {
        return self == other;
    }, py::is_operator());

Please prefer only one style within a given module for readability.

.. note::

    ``py::is_operator()`` is necessary to get the correct ``NotImplemented`` return when called with unsupported types.  It should not be used in wrapping in-place operators (e.g. ``__iadd__``), however, as this can lead to confusing behavior.

.. _style-guide-pybind11-division:

Division MUST be wrapped as ``__truediv__`` and possibly ``__floordiv__``, not ``__div__``
---------------------------------------------------------------------------------------------

Wrapping ``__div__`` allows old-style division to work, which should be disallowed in all LSST Python code.  Not defining it turns subtle differences into easy-to-spot (and fix) exceptions.

The same rule applies for in-place operators:  ``__itruediv__`` and ``__ifloordiv__`` may be defined, but ``__idiv__`` should not.

.. _style-guide-pybind11-properties:

All rules from the Python style guide regarding properties SHALL also apply to C++ wrappers
-------------------------------------------------------------------------------------------

See :ref:`style-guide-py-properties` in the python guide.

.. _style-guide-pybind11-module-docstrings:

Module docstrings SHOULD be empty
---------------------------------

Wrapper module docstrings are not visible by users (since all classes are lifted into the package namespace by ``__init__.py``), and hence do not need to follow the usual requirements for module-level docstrings.
Empty docstrings are preferable to trivial strings that just duplicate information implicit in the naming conventions (e.g. "The 'thing' module provides wrappers for thing.h").

.. _style-guide-pybind11-str:

Classes SHOULD define ``__str__`` to return a human readable string representation of the object
------------------------------------------------------------------------------------------------

``__str__`` is intended to return a human readable string representation of the object.
Typically this can be the output of ``operator<<``:

.. code-block:: cpp

    cls.def("__str__", [](Class const& self) {
        std::ostringstream os;
        os << self;
        return os.str();
    });

.. _style-guide-pybind11-repr:

Classes SHOULD define ``__repr__`` to return a minimal summary of the object including the fully-qualified name of the class
----------------------------------------------------------------------------------------------------------------------------

``__repr__`` is intended to return a **minimal** summary of the object. It MUST include the fully-qualified name of the class, but MAY be defined to include per-instance values or a summary thereof.
For small objects, producing a string that can be passed to ``eval`` to reproduce the object is often a good guideline:

.. code-block:: cpp

    clsPoint2D.def("__str__", [](Point2D const& self) {
        return py::str("lsst.geom.Point2D({}, {})").format(self.getX(), self.getY());
    });

