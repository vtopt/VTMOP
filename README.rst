=============================================================================
ACM TOMS Algorithm 1028: VTMOP: Solver for Blackbox Multiobjective Optimization Problems
=============================================================================

VTMOP is a Fortran 2008 package containing a robust, portable solver and
a flexible framework for solving MOPs. Designed for efficiency and
scalability to an arbitrary number of objectives, VTMOP attempts to generate
uniformly spaced points on a (possibly nonconvex) Pareto front with minimal
cost function evaluations. The driver subroutine is VTMOP_SOLVE, which
can be run both serially and with parallel function evaluations.
Minimal subsets of dependencies such as VTDIRECT, QNSTOP, SHEPPACK,
DELAUNAYSPARSE, SLATEC, LAPACK, and BLAS are also provided. Comments at
the top of each subroutine document their usage, and examples demonstrating
the driver's usage are given in ``src/sample.f90``.

A Python interface for VTMOP is also available through libEnsemble:
    https://github.com/Libensemble/libe-community-examples

-------------
Usage Summary
-------------

VTMOP features two distinctly different user-interfaces, with varying
levels of flexibility.

The first and simplest user-interface is the driver subroutine
``VTMOP_SOLVE``, which is included in the module ``VTMOP_LIB``.
``VTMOP_SOLVE`` accepts problem dimensions, simple bound constraints, an
objective function, some (optional) parameter settings, and (optionally) any
user-defined surrogate models and optimization routines. The default setting
is to perform an adaptive search using the VTDIRECT95 software package,
fit LSHEP surrogate models, and optimize the surrogate models using direct
search with the polling strategy generalized pattern search (GPS).

In order to use ``VTMOP_SOLVE``, the blackbox multiobjective cost function
must be available as a Fortran 2008 subroutine. For information on how to
achieve this, including cases where F is an ISO C/C++ function and when
F is a command line executable, see ``src/OBJ_FUNC_README``. Additional
information on ``VTMOP_SOLVE`` is provided in the comments around each
subroutine definition, in the file ``vtmop.f90``.

The second interface is the return-to-caller interface, which allows
advanced users to perform function evaluations in batches, in an independent
environment. The return-to-caller interface is contained in the module
``VTMOP_MOD``, and contains four driver subroutines (below). Additional
information on these subroutines is provided in the code documentation, in
the file ``vtmop.f90``.

 - ``VTMOP_INIT`` initializes an instance of the data type ``VTMOP_TYPE`` for
   passing data between subroutines.
 - ``VTMOP_LTR`` identifies the most isolated point on the Pareto front,
   constructs a local trust region (LTR) about that point, and returns
   a region of the design for the user to explore independently.
 - ``VTMOP_OPT`` fits several surrogate models to the current dataset, and uses
   these models to propose a set of candidate points in the LTR, for evaluation
   by the user.
 - ``VTMOP_FINALIZE`` post processes the dataset gathered throughout the
   iterations of the algorithm, returns the nondominated and efficient point
   sets, and frees all internal memory.

VTMOP also includes a checkpointing system, a detailed error handling
system, and maintains a database of all function evaluations ever taken,
which can be retrieved using optional output arrays.

For more detailed information on usage, see the corresponding sections in
the ``src/USERS`` file.

------------------------------
Source Code and Package Layout
------------------------------

The directory ``src`` contains all source code and documentation,
exactly as it appeared in *ACM TOMS Algorithm 1028*.

The physical organization of the ``src`` directory is as follows:

 - The file ``depend_graph.txt`` contains a diagram of the dependency tree
   for the VTMOP package and its sample main program.
 - The file ``vtmop.f90`` is the main Fortran 2008 file containing the
   VTMOP_MOD and VTMOP_LIB libraries containing worker and driver interfaces,
   respectively.
 - The file ``delsparse.f90`` contains the module and driver subroutines
   for DELAUNAYSPARSE (ACM TOMS Algorithm 1012).
 - The file ``linear_shepard.f90`` is a Fortran 95 module for computing the
   LSHEP surrogate model (ACM TOMS Algorithm 905).
 - The file ``qnstop.f90`` contains the LATINDESIGN function from QNSTOP
   (ACM TOMS Algorithm 1007).
 - The file ``sVTdirect.f90`` contains a serial implementation of the
   Fortran 95 algorithm VTdirect95 (ACM TOMS Algorithm 897).
 - The file ``bVTdirect.f90`` contains a slight modification to
   ``sVTdirect.f90``, which allows for usage in VTMOP_SOLVE's parallel
   paradigm.
 - The file ``shared_modules.f90`` contains modules and subroutines that
   are used by VTdirect95, as well as the module REAL_PRECISION, which is
   used for approximately 64 bit arithmetic.
 - The file ``slatec.f`` contains the subroutine DWNNLS and its dependencies
   from the SLATEC library. This library has been slightly modified to
   comply with the modern Fortran standards. Additionally, legacy
   implementations of the BLAS subroutines DROTM and DTROMG have been
   included under different names to avoid dependency issues.
 - The files ``lapack.f`` and ``blas.f`` contain all LAPACK and BLAS
   subroutines that are referenced (both directly and indirectly) in
   VTMOP or its dependencies.
 - ``vtmop_func.f90`` contains the module VTMOP_FUNC_MOD, which contains
   several multiobjective test problems.
 - ``samples.f90`` contains sample code for building and running the test
   problems in ``vtmop_func.f90`` and checks the serial installation for
   correctness.
 - ``samplep.f90`` contains sample code for building and running the test
   problems in ``vtmop_func.f90`` and checks the parallel installation for
   correctness.
 - ``cl_objfunc.f90`` contains sample code for implementing a command line
   executable as a Fortran subroutine, while matching the interface
   expected by VTMOP_SOLVE.
 - A sample GNU ``Makefile`` is also provided.

--------------------
Building and Running
--------------------

From here on, the files ``samples.f90`` and ``samplep.f90`` will be referred
to collectively as ``sample{s|p}.f90``.

To build the sample code and test the installation, use the following command,
where ``$(F90)`` is a Fortran 2008 compiler and ``$(OPTS)`` contains the
compiler options (including the option to build with OpenMP).

If BLAS and LAPACK exist on your system, then ``$(LIBS)`` should contain flags
to link those libraries and ``blas.f`` and ``lapack.f`` can be removed from
the following command; otherwise, ``$(LIBS)`` can be ignored.

.. code-block:: bash

    $(F90) $(OPTS) $(LIBS) shared_modules.f90 blas.f lapack.f slatec.f \
        qnstop.f90 sVTdirect.f90 bVTdirect.f90 delsparse.f90           \
        linear_shepard.f90 vtmop.f90 vtmop_func.f90 sample{s|p}.f90    \
        -o sample{s|p}

To test the installation, use

.. code-block:: bash

    ./sample{s|p}

Before running the parallel driver "samplep", set the following environment
variables:

 - ``export OMP_NESTED=TRUE``
 - ``export OMP_NUM_THREADS=T1,T2``

where ``T1 = MAX(NUMBER OF PROCESSORS, NUMBER OF OBJECTIVES)``
and ``T2 = CEILING(NUMBER OF PROCESSORS / T1)``.
For the sample code provided, the number of objectives is three.

This code has been tested with the GNU 5.4.0 (and newer) and the Intel 17.0.4
Fortran compilers. Other modern Fortran compilers may not offer full support
for the Fortran 2008 standard. If using a different compiler, first check
whether it supports

 - passing internal procedures as actual arguments
 - usage of the ``IEEE_ARITHMETIC`` intrinsic module.

-----------------------------------
Citations and Additional References
-----------------------------------

If you use VTMOP as part of a published work, please cite the following
publication:

.. code-block:: bibtex

    @article{chang2022algorithm,
      author={Chang, Tyler H. and Watson, Layne T. and Larson, Jeffrey and Neveu, Nicole and Thacker, William I. and Deshpande, Shubhangi and Lux, Thomas C. H.},
      year={2022},
      title={Algorithm {1028}: {VTMOP}: {S}olver for blackbox multiobjective optimization problems},
      journal={ACM Transactions on Mathematical Software},
      volume={48},
      number={3},
      articleno={36},
      pages={1-34},
      doi={10.1145/3529258}
    }

-------------------------
Contacts and Contributors
-------------------------

For further inquiries, contact
 - Tyler Chang, ``tchang@anl.gov``
 - Layne Watson, ``ltw@cs.vt.edu``

For a full list of contributors, including contributions to the
source code, theoretical contributions, and proof-reading/writing
documentation, see the ``src/CONTRIBUTORS`` document.

Special thanks to the editors and anonymous reviewers of ACM TOMS,
for their helpful comments and suggestions.
