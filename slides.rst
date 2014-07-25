=================================================
Fast Serialization of Numpy Arrays with Bloscpack
=================================================

Blosc
=====

Blosc: A Fast Meta-Codec
------------------------

* Blocking
* Shuffling
* Multithreaded
* Multi-codec

.. image:: images/blosc.pdf

Shuffle Filter
--------------

* Reorder bytes by significance inside a block
* Potentially reduce Lempel-Ziv complexity of the data

.. image:: images/shuffle.pdf

Multi-Codec
-----------

* By default it uses **Blosclz** -- derived from **Fastlz**

* Alternative codecs

  * **LZ4 / LZ4HC**
  * **Snappy**
  * **Zlib**

python-blosc: bindings
----------------------

* Python C-API bindings
* Accepts a pointer as int

Bloscpack
=========

Bloscpack
---------

* Simple serialization format based on Blosc
* Command line interface
* Python API with support for Numpy arrays
* Aimed at developers / power-users

Features
--------

* Chunked, compressed format
* Metadata (optional)
* Checksums (optional)
* Offsets, including pre-allocation for append (optional)

.. image:: images/bloscpack.pdf

Use Cases
---------

* Fast serialization
* Streaming
* On disk columnar storage
* Substrate on which to build high-level abstractions

Existing Users
--------------

* `bcolz <https://github.com/Blosc/bcolz>`_

  * chunked, compressed, columnar container (4C)
  * Uses Bloscpack for serialization and out-of-core computations

* Recent EuroPython presentation: https://www.youtube.com/watch?v=-lKV4zC1gss

API
---

* Concept: sinks and sources

  * ``PlainSource`` --> ``CompressedSink``
  * ``CompressedSource`` --> ``PlainSink``

* Inner loop compress/decompress implemented
* Supply appropriate source and sink
* Sources and sinks must obey an interface/contract
* Get easy Anything -> Anything
* E.g. Numpy -> {string, file, memory, network}

``pack`` and ``unpack``
-----------------------

.. code-block:: python

    def pack(source, sink,
             nchunks, chunk_size, last_chunk,
             metadata=None,
             blosc_args=None,
             bloscpack_args=None,
             metadata_args=None):
        pass

    def unpack(source, sink):
        pass

.. ``PlainSource``
.. ---------------
.. 
.. * Supply plain chunks (e.g. bytes or pointers(ints)) and a method to compress them
.. 
.. .. code-block:: python
.. 
..     class PlainSource(object):
.. 
..         def compress_func(self):
..             pass
.. 
..         def __iter__(self):
..             pass
.. 
.. ``CompressedSource``
.. --------------------
.. 
.. * Supply compressed chunks
.. 
.. .. code-block:: python
.. 
..     class CompressedSource(object):
.. 
..         def __iter__(self):
..             pass
.. 
.. ``PlainSink``
.. -------------
.. 
.. * Accept plain (decompressed) chunks
.. 
.. .. code-block:: python
.. 
..     class PlainSink(object):
.. 
..         def put(self, chunk):
..             pass
.. 
.. ``CompressedSink``
.. ------------------
.. 
.. * Accept compressed chunks, amongst other things
.. 
.. .. code-block:: python
.. 
..     class CompressedSink(object):
.. 
..         def write_bloscpack_header(self):
..             pass
.. 
..         def write_metadata(self, metadata, metadata_args):
..             pass
.. 
..         def init_offsets(self):
..             pass
.. 
..         def finalize(self):
..             pass
.. 
..         def put(self, i, compressed):
..             pass

Numpy Example
-------------

.. code-block:: python

   import numpy as np
   import bloscpack as bp

   a = np.arange(1e7)

   # pack with defaults
   bp.pack_ndarray_file(a, 'a.blp')

   # pack with custom settings
   bp.pack_ndarray_file(a, 'a.blp',
       chunk_size='20M',
       blosc_args=bp.BloscArgs(cname='lz4', clevel=9),
       bloscpack_args=bp.BloscpackArgs(offsets=False),
       )

   # unpack
   b = bp.unpack_ndarray('a.blp')

Extension Example
-----------------

* Idea: how about S3 connectivity?
* Implement CompressedS3Sink and CompressedS3Source
* (These know nothing about Numpy)
* Result: ability to compress a Numpy array to an S3 bucket

.. Somthing along the lines of...
.. ------------------------------
.. 
.. .. code-block:: python
.. 
..    source = bp.PlainNumpySource(a)
..    sink = bp.CompressedS3Sink(bucket)
..    chunk_size = '20M'
..    nchunks, chunk_size, last_chunk_size = \
..        bp.calculate_nchunks(source.size, chunk_size)
..    bp.pack(source, sink,
..            nchunks, chunk_size, last_chunk_size,
..            metadata=source.metadata)

Benchmarks
==========

Background
----------

* Builds on benchmarks presented at EuroScipy 2013
* Those used a laptop with SSD and SD storage
* Showed that Bloscpack can be outperform contenders

See also: `Bloscpack: a compressed lightweight serialization format for
numerical data <http://arxiv.org/abs/1404.6383>`_

Experimental Setup
------------------

* Use Python 3.4
* Use some real-world datasets
* Benchmark new codecs available in Blosc
* Add PyTables to the mix
* Run it in the AWS cloud

Datasets
--------

* **arange**

  * Integers

* **linspace**

  * floats

* **poisson**

  * more or less random numbers

* **neuronal**

  * Neural net spike time stamps
  * Kindly provided by Yuri Zaytsev

* **bitcoin**

  * Historical MtGOX trade data

Contenders
----------

* PyTables

  * HDF5 interface
  * Supports Blosc and others

* NPY

  * Numpy plain serialization

* NPZ

  * Numpy compressed (using zip) serialization

* ZFile

  * Joblib's compressed (using zlib) **pickler** extension

NPY Flaw
--------

* Prior to serialization, array is copied in memory with ``tostring()``
* Fixed by Olivier Grisel to use ``nditer`` (`#4077 <https://github.com/numpy/numpy/pull/4077>`_)
* Available in  ``v1.9.0b1``, which is what I used for the benchmarks

NPZ Flaw
--------

* Create a temporary plain version (``/tmp``)
* Compresses into a Zip archive from there
* Due to issues with the ZipFile module

ZFile Flaw
----------

* Does not support arrays larger than 2GB
* An ``int32`` is used somewhere for the size

Remaining Experimental Parameters
---------------------------------

* Instance

  * c3.2xlarge
  * CPUs: 8
  * RAM: 15GB

* Dataset Sizes

  * 1MB
  * 10MB
  * 100MB

* Storage

  * EBS
  * Ephemeral

Results
-------

Let's look at the ``arange`` and ``neuronal`` datasets in the ``small`` and
``large`` configuration on ``ebs`` --> IPython notebooks

Aggregated Results
------------------

* Single plots can supply insights
* Need to aggregate for a big picture
* Award points to a codec/level combination

  * Slowest receives 1 point
  * Fastest receives 68 points
  * Ratio doesn't count

* Recommendation for a good general purpose codec


Aggregated Results - bottom 10
------------------------------

.. code-block::

    (623, 'tables_zlib_7')
    (642, 'npz_1')
    (645, 'tables_zlib_9')
    (687, 'tables_zlib_5')
    (968, 'tables_blosc_zlib_9')
    (970, 'zfile_9')
    (989, 'tables_blosc_zlib_7')
    (1040, 'zfile_7')
    (1059, 'tables_blosc_zlib_5')
    (1143, 'zfile_3')

* As expected

Aggregated Results - top 10
---------------------------

.. code-block::

    (4994, 'bloscpack_snappy_5')
    (5047, 'bloscpack_blosclz_3')
    (5138, 'bloscpack_snappy_7')
    (5252, 'bloscpack_blosclz_7')
    (5292, 'bloscpack_lz4_9')
    (5342, 'bloscpack_lz4_3')
    (5358, 'bloscpack_blosclz_5')
    (5363, 'bloscpack_lz4_1')
    (5469, 'bloscpack_lz4_7')
    (5508, 'bloscpack_lz4_5')

* ``bloscpack_blosclz_7`` is the current default

Reproducibility
---------------

* Results contained in the talk sources repository
* Lists almost all the hashes and configurations
* All code open source
* All datasets additionally available from backup location on own infrastructure
* AMI available incl. instructions (soon to come / ask me)

Relationship to (Distributed) Analytics Engines
-----------------------------------------------

* Colum-oriented, compressed, chunked storage

  * `bcolz <https://github.com/Blosc/bcolz>`_
  * `Hustle <https://github.com/chango/hustle>`_
  * `Parquet <http://parquet.io/>`_
  * `RCFile / ORCFile <https://code.facebook.com/posts/229861827208629/scaling-the-facebook-data-warehouse-to-300-pb/?_fb_noscript=1>`_

* Fast, partial loading from disk or network
* Reduced storage requirements
* But: need to chose the *right codec™*
* A Bloscpack file translates directly to a serialized column

TODO
----

* Support Bloscpack in Joblib

  * Speed gain
  * Mitigate 2GB issue

* Release Python 3 support
