`std::map_reduce` is a parallel processing framework for Rust inspired by Google's MapReduce. This page describes (or will eventually describe) the design and implementation of `std::map_reduce`.

## Open Issues

Writing a generic MapReduce framework pretty much requires us to be able to send some kind of functions over channels. This was one of the realizations that led us to consider (and perhaps adopt) the three-layer function type stratification.

Writing distributed MapReduce will make this even more problematic, as it would require serializing functions over the network. Why not instead provide a way to instantiate a MapReduce computation with the required functions in the mean time? (i.e. statically fix available functions during process instantiation) -- boggle.

It will be easier to implement this, once type classes or something similar have been implemented (generic type of comparable items, etc.)

## Extensions

### Multi-Step-MapReduce and Join-Step

It is nice, if multiple MapReduces can be easily joined and by adding support for a join step, many more queries can be implented. Paper below has the details for that:

http://www.comp.nus.edu.sg/~epic/tkdesi-2010-03-0153.R1_Jiang.pdf

### Streaming MapReduce

There is quite a bit of research on this.  Implementing streaming MapReduce should give message sending and task scheduling of the runtime a nice workout.

### Distributed MapReduce

Going over the network. Needs way to spawn remote processes, distribute data and handling of failures.

### Strom/Kafka

I'm (boggle) quite intrigued by the storm framework and kafka event broker. It is more powerful than mapreduce and allows to implement all of the above use cases. So maybe we should implement something like storm for multicore processing right away and put mapreducejoin atop as an example. I think this would also be useful for implementing rendering pipelines, and streaming to webpages and all kinds of other things Mozilla might like. 