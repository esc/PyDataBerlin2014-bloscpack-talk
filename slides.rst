=================================================
Fast Serialization of Numpy Arrays with Bloscpack
=================================================

Blosc
=====

Blosc: a fast compressor
------------------------

pass

python-blosc: bindings
----------------------

pass

Bloscpack
=========

Use cases
---------

* Seri

API
---

* Concept: sinks and sources

  * PlainSource --> CompressedSink
  * CompressedSource --> PlainSink

* Inner loop compress/decompres implemented
* Supply appropriate source and sink
* Get easy N -> N
* E.g. Numpy -> {string, file, memory}

pack
----

.. code-block:: python

    def pack(source, sink,
             nchunks, chunk_size, last_chunk,
             metadata=None,
             blosc_args=None,
             bloscpack_args=None,
             metadata_args=None):


API Design, sources and sinks

Streaming

S3CompressedSink / S3CompressedSource

Results
