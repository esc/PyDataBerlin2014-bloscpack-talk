=================================================
Fast Serialization of Numpy Arrays with Bloscpack
=================================================

Blosc
=====

Blosc: a fast meta-codec
------------------------

* Blocking
* Shuffling
* Multithreaded
* Multi-codec

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

Features
--------

* Chunked compressed format (no 2GB limit)
* Metadata
* Checksums
* Offsets, including pre-allocation for append

API
---

* Concept: sinks and sources

  * ``PlainSource`` --> ``CompressedSink``
  * ``CompressedSource`` --> ``PlainSink``

* Inner loop compress/decompres implemented
* Supply appropriate source and sink
* Get easy N -> N
* E.g. Numpy -> {string, file, memory}
* Sources and Sinks can be generators
* Abstract classes with some convenience provided

API
---

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

``PlainSource``
---------------

* Supply plain chunks (e.g. bytes or pointers(ints)) and a method to compress them

.. code-block:: python

    class PlainSource(object):

        def compress_func(self):
            pass

        def __iter__(self):
            pass

``CompressedSource``
--------------------

* Supply compressed chunks

.. code-block:: python

    class CompressedSource(object):

        def __iter__(self):
            pass

``PlainSink``
-------------

* Accept plain (decompressed) chunks

.. code-block:: python

    class PlainSink(object):

        def put(self, chunk):
            pass

``CompressedSink``
------------------

* Accept compressed chunks, amongst other things

.. code-block:: python

    class CompressedSink(object):

        def write_bloscpack_header(self):
            pass

        def write_metadata(self, metadata, metadata_args):
            pass

        def init_offsets(self):
            pass

        def finalize(self):
            pass

        def put(self, i, compressed):
            pass

Numpy Example
-------------

.. code-block:: python

   import numpy as np
   import bloscpack as bp

   a = np.arange(1e7)

   # use defaults
   bp.pack_ndarray_file(a, 'a.blp')

   # use custom settings
   bp.pack_ndarray_file(a,
       chunk_size='20M',
       blosc_args=bp.BloscArgs(cname='lz4', clevel=9),
       bloscpack_args=bp.BloscpackArgs(offsets=False),
       )

Extension Example
-----------------

* Idea: how about S3 connectivity?
* Implement CompressedS3Sink and CompressedS3Source
* (These know nothing about Numpy)

Somthing along the lines of...
------------------------------

.. code-block:: python

   source = bp.PlainNumpySource(a)
   sink = bp.CompressedS3Sink(bucket)
   chunk_size = '20M'
   nchunks, chunk_size, last_chunk_size = \
       bp.calculate_nchunks(source.size, chunk_size)
   bp.pack(source, sink,
           nchunks, chunk_size, last_chunk_size,
           metadata=source.metadata)


Rest
----


Streaming

S3CompressedSink / S3CompressedSource

Results

Joblib integration
