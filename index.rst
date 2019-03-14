
:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   Summary of performance studies with PPDB prototype


Introduction
============

This technical note describes an summarizes performance tests with PPDB
prototype. Performance of PPDB operations is crucial for Alert Production (AP)
as it has to fetch and save large number of records in PPDB.


AP Prototype
============

Alert Production prototype is an application that simulates AP pipeline and
uses PPDB interface to fetch/store DIA objects and sources. Initially PPDB
implementation was developed as part of this prototype in ``lsst-dm/l1dbproto``
package. As implementation matured it has been moved to ``lsst/dax_ppdb``
package which is also used now by AP group for pipeline implementation.

Prototype application (``ap_proto``) contains mock implementation of Difference
Image Analysis which is based on pre-defined position of a variable objects.
The average density of the variable objects is defined so that it produces
about 10,000 sources per visit. On top of that mock DIA adds additional 50%
random sources representing noise. Mock DIA does not generate any physical
quantities for the sources except their positions. ``ap_proto`` mocks matching
of DIA sources to existing objects using its knowledge of the source origin
thus avoiding position-based match. This likely works more perfectly than
actual matching in real AP pipeline, but it should be sufficient for
purpose of this prototype. Overall AP pipeline simulation in prototype runs
extremely fast but its output is not usable for anything except performance
studies.

PPDB implementation that was developed as part of the prototype and currently
existing in ``dax_ppdb`` is specially instrumented to produce logging
records which include timing information for individual PPDB operations.
``ap_proto`` adds more timing information to the logging output. Separate
script is used to analyze log files produced by prototype and generate summary
CSV file with per-visit timing information which is later analyzed in Jupyter.

To simulate concurrency aspect of AP pipeline ``ap_proto`` splits circular
FOV region into a number of square tiles (e.g. 15 by 15) and processed
these tiles concurrently. Two different approaches were used for tile
processing:

- fork mode: after DIA mock stage script forks itself into the number of
  sub-processes, one sub-process per tile, and all of them run concurrently
  on the same host (this is repeated on every visit).
- MPI mode: it starts one process per tile on MPI cluster, one process is
  responsible for DIA mock stage which then distributes generated sources
  to all other processes, each running on data from its separate tile.

Obvious drawback of the fork mode is a competition for CPU resources on single
host with typical number of tiles much larger than number of cores even on
relatively large hosts, though tests show that this is not an issue due to a
bottleneck on database server side. MPI mode should produce more faithful
simulation of actual pipeline running if there is a sufficient number of hosts
to run all processes.


PPDB Schema
===========


Hardware Description
====================


Individual Tests
================


MySQL and PostgreSQL at IN2P3
-----------------------------


Oracle RAC at NCSA
------------------


PostgreSQL at Google Cloud
--------------------------


Test Summary
============
