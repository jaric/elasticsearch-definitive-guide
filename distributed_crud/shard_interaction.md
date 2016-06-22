### 主從庫之間是如何通信的

For explanation purposes, let's imagine that we have a cluster
consisting of 3 nodes. It contains one index called `blogs` which has
two primary shards. Each primary shard has two replicas. Copies of
the same shard are never allocated to the same node, so our cluster
looks something like <<img-distrib>>.

[[img-distrib]]
.A cluster with three nodes and one index
image::images/04-01_index.png["A cluster with three nodes and one index"]

We can send our requests to any node in the cluster. Every node is fully
capable of serving any request.  Every node knows the location of every
document in the cluster and so can forward requests directly to the required
node. In the examples below, we will send all of our requests to `Node 1`,
which we will refer to as  the _requesting node_.

TIP: When sending requests, it is good practice to round-robin through all the
nodes in the cluster, in order to spread the load.
