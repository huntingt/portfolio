+++
title = "CGRA Acceleration of Barnes-Hut"
date = "2021-12-06T16:50:16-05:00"
+++

From 2020-2021 I worked on researching irregular algorithm acceleration using
CGRAs. While Barnes-Hut is easily parallelizable, there are interesting methods
that can be used to exploit cache locality to further accelerate it.

Mapping
this algorithm to a CGRA was particularly hard because the tree traversal
appears to require a variable state size (ie. a stack or recursive function
calls). Since the CGRA has a fixed amount of data that can easily fit into the
pipeline, this becomes challenging. Fortunately the algorithm can be mapped so
the tree traversal state is fixed in size (and very small).

I also designed a small hardware accelerator that could be added to the CGRA to
help schedule the memory accesses and further increase the performance.

# Links

- [Paper](/6_UAR_Paper.pdf)
