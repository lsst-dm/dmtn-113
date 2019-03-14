
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
implementation was developed as part of this prototype in `l1dbproto`_
package. As implementation matured it has been moved to `dax_ppdb`_
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
existing in `dax_ppdb`_ is specially instrumented to produce logging
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

Database schema used in prototype is the baseline schema defined in `DPDD`_
and `cat`_ package with some modifications:

- DIAObject table adds few columns that are used by the prototype or AP
  pipeline (these columns will probably be added to official schema).
- Additional tables could be used by PPDB implementation to optimize access
  to most frequently used data (effectively de-normalization of standard
  schema), these tables are internal to implementation and do not change
  client-visible schema.

As mentioned above, prototype does not generate sensible science data for the
bulk of the columns, only spatial information and Object/Source relations are
are filled by prototype.

Hardware Description
====================

Different storage backends were tested on different platforms and different
location, this was driven by hardware and backend availability at a time.

Initial testing and development was performed on a single host at IN2P3 which
allowed us to compare SSD and spinning disk-based storage options for
database, with MySQL or PostgreSQL used as backend. Both clients and server
were running on the same host (for the lack of other comparable hardware).
Machine specifications are:

- CPU: Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz
- 2x14 physical cores, 56 total threads
- 512GB RAM
- SSD storage: 2TB NVMe and 3TB SATA
- spinning disks: 7.3TB in RAID6 array

In second series of tests with Oracle RAC at NCSA database server ran on 3
identical hosts (details are in `NCSA_RAC`_):

- Dual Intel Xeon E5-2650 @ 2.00GHz
- 2x8 physical cores, 32 total threads
- 128GB RAM
- storage: NetApp array with 50x8TB HDDs, 10x1.5TB SSDs (RAID1 array),
  connected via 16Gbps Fibre Channel

Tests with PostgreSQL server on Google Cloud platform were using virtual
Compute Engine, two hosts were set up, one for database server, another
for clients. Specs for both hosts are:

- Server host:

  - 64 vCPU (Intel Sandy Bridge)
  - 416GB RAM
  - storage: 10TB SSD (network-connected)

- Client host:

  - 64 vCPU (Intel Sandy Bridge)
  - 57GB RAM


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


.. _cat: https://github.com/lsst/cat
.. _dax_ppdb: https://github.com/lsst/dax_ppdb
.. _DPDD: http://ls.st/dpdd
.. _l1dbproto: https://github.com/lsst-dm/l1dbproto
.. _NCSA_RAC: https://jira.lsstcorp.org/browse/DM-14712?focusedCommentId=133072&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-133072
