`std::map_reduce` is a parallel processing framework for Rust inspired by Google's MapReduce. This page describes (or will eventually describe) the design and implementation of `std::map_reduce`.

## Open Issues

Writing a generic MapReduce framework pretty much requires us to be able to send some kind of functions over channels. This was one of the realizations that led us to consider (and perhaps adopt) the three-layer function type stratification.