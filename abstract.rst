Fast Serialization of Numpy Arrays with Bloscpack
-------------------------------------------------

Bloscpack [1] is a reference implementation and file-format for fast serialization
of numerical data. It features lightweight, chunked and compressed storage,
based on the extremely fast Blosc [2] metacodec and supports serialization of
Numpy arrays out-of-the-box. Recently, Blosc -- being the metacodec that it is
-- has received support for using the popular and widely used Snappy [3], LZ4
[4], and ZLib [5] codecs, and so, now Bloscpack supports serializing Numpy arrays
easily with those codecs!

In this talk I will present recent benchmarks of Bloscpack performance on a
variety of artificial and real-world datasets with a special focus on the newly
available codecs. In these benchmarks I will compare Bloscpack, both
performance and usability wise, to alternatives such as Numpy's native offerings
(NPZ and NPY), HDF5/PyTables [6], and if time permits, to novel bleeding edge
solutions.

Lastly I will argue that compressed and chunked storage format such as
Bloscpack can be and somewhat already is a useful substrate on which to build
more powerful applications such as online analytical processing engines and
distributed computing frameworks.

[1]: https://github.com/Blosc/bloscpack
[2]: https://github.com/Blosc/c-blosc/
[3]: http://code.google.com/p/snappy/
[4]: http://code.google.com/p/lz4/
[5]: http://www.zlib.net/
[6]: http://www.pytables.org/moin
