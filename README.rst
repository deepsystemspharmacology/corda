|travis| |appveyor| |codecov.io| |Code Health|

CORDA for Python
================

This is a Python implementation based on the papers of Schultz et. al. with
some added optimizations. It is based on the following two publiactions:

- `Reconstruction of Tissue-Specific Metabolic Networks Using
  CORDA <http://journals.plos.org/ploscompbiol/article/authors?id=10.1371%2Fjournal.pcbi.1004808>`_
- `IDENTIFYING CANCER SPECIFIC METABOLIC SIGNATURES USING CONSTRAINT-BASED MODELS
  <http://dx.doi.org/10.1142/9789813207813_0045>`_

This Python package is developed in the
`Human Systems Biology Group <https://resendislab.github.io>`_ of
Prof. Osbaldo Resendis Antonio at the `National Institute of Genomic
Medicine Mexico <https://inmegen.gob.mx>`_ and includes recent updates to
the method (*CORDA 2*).


How to cite?
------------

This particular implementation of CORDA has not been published so far. In the
meantime you should if you cite the respective publications for the method
mentioned above and provide a link to this GitHub repository.

What does it do?
----------------

CORDA, short for Cost Optimization Reaction Dependency Assessment is a
method for the reconstruction of metabolic networks from a given
reference model (a database of all known reactions) and a confidence
mapping for reactions. It allows you to reconstruct metabolic models for
tissues, patients or specific experimental conditions from a set of
transcription or proteome measurements.

How do I install it
-------------------

CORDA for Python works only for Python 3.4+ and requires
`cobrapy <http://github.com/opencobra/cobrapy>`__ to work. After having
a working Python installation with pip (Anaconda or Miniconda works fine
here as well) you can install corda with pip

.. code:: bash

    pip install corda

This will download and install cobrapy as well. I recommend using a
version of pip that supports manylinux builds for faster installation
(pip>=8.1).

For now the master branch is usually working and tested whereas all new
features are kept in its own branch. To install from the master branch
directly use

.. code:: bash

    pip install https://github.com/resendislab/corda/archive/master.zip

What do I need to run it?
-------------------------

CORDA requires a base model including all reactions that could possibly
included such as Recon 1/2 or HMR. You will also need gene expression or
proteome data for our tissue/patient/experimental setting. This data has
to be translated into 5 distinct classes: unknown (0), not
expressed/present (-1), low confidence (1), medium confidence (2) and
high confidence (3). CORDA will then ensure to include as many high
confidence reactions as possible while minimizing the inclusion of
absent (-1) reactions while maintaining a set of metabolic requirements.

How do I use it?
----------------

A small tutorial is found at https://resendislab.github.io/corda.

What's the advantage over other reconstruction algorithms?
----------------------------------------------------------

No commercial solver needed
***************************

It does not require any commercial solvers, in fact it works fastest
with the free glpk solver that already comes together with cobrapy.
For instance for the small central metabolism model (101 irreversible
reactions) included together with CORDA the glpk version is a bout 3 times
faster than the fastest tested commercial solver (cplex).

Fast reconstructions
********************

CORDA for Python uses a strategy similar to FastFVA, where
a previous solution basis is recycled repeatedly.

Some reference times for reconstructing the minimal growing models for
iJO1366 (*E. coli*) and Recon 2.2:

.. code:: python

    In [1]: from cobra.test import create_test_model
    Loading symengine... This feature is in beta testing. Please report any issues you encounter on http://github.com/biosustain/optlang/issues

    In [2]: from cobra.io import read_sbml_model

    In [3]: from corda import CORDA

    In [4]: ecoli = create_test_model("ecoli")

    In [5]: conf = {}

    In [6]: for r in ecoli.reactions:
    ...:     conf[r.id] = -1
    ...:

    In [7]: conf["Ec_biomass_iJO1366_core_53p95M"] = 3

    In [8]: %time opt = CORDA(ecoli, conf)
    CPU times: user 282 ms, sys: 1.81 ms, total: 284 ms
    Wall time: 284 ms

    In [9]: %time opt.build()
    CPU times: user 9.04 s, sys: 93 µs, total: 9.04 s
    Wall time: 9.05 s

    In [10]: print(opt)
    build status: reconstruction complete
    Inc. reactions: 456/2583
    - unclear: 0/0
    - exclude: 455/2582
    - low and medium: 0/0
    - high: 1/1


    In [11]:

    In [12]: recon2 = read_sbml_model("/home/cdiener/Downloads/recon_2.2.xml")
    cobra/io/sbml.py:235 UserWarning: M_h_c appears as a reactant and product RE3453C
    cobra/io/sbml.py:235 UserWarning: M_h_c appears as a reactant and product RE3459C
    cobra/io/sbml.py:235 UserWarning: M_h_x appears as a reactant and product FAOXC24C22x
    cobra/io/sbml.py:235 UserWarning: M_h_c appears as a reactant and product HAS1
    cobra/io/sbml.py:235 UserWarning: M_h2o_x appears as a reactant and product PROFVSCOAhc

    In [13]: conf = {}

    In [14]: for r in recon2.reactions:
        ...:     conf[r.id] = -1
        ...:

    In [15]: conf["biomass_reaction"] = 3

    In [16]: %time opt = CORDA(recon2, conf)
    CPU times: user 1 s, sys: 8.95 ms, total: 1.01 s
    Wall time: 1.01 s

    In [17]: %time opt.build()
    CPU times: user 24.7 s, sys: 240 µs, total: 24.7 s
    Wall time: 24.8 s

    In [28]: print(opt)
    build status: reconstruction complete
    Inc. reactions: 395/7864
    - unclear: 0/0
    - exclude: 394/7863
    - low and medium: 0/0
    - high: 1/1

.. |travis| image:: https://travis-ci.org/resendislab/corda.svg?branch=master
   :target: https://travis-ci.org/resendislab/corda
.. |appveyor| image:: https://ci.appveyor.com/api/projects/status/vr0ibaiqb5lnu3ii/branch/master?svg=true
   :target: https://ci.appveyor.com/project/cdiener/corda-69at9/branch/master
.. |codecov.io| image:: https://codecov.io/github/resendislab/corda/coverage.svg?branch=master
   :target: https://codecov.io/github/resendislab/corda?branch=master
.. |Code Health| image:: https://landscape.io/github/resendislab/corda/master/landscape.svg?style=flat
   :target: https://landscape.io/github/resendislab/corda/master
