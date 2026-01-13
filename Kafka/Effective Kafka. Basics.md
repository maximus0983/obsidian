![[Снимок экрана 2024-07-07 в 23.15.23.png]]

.![[]]![[Снимок экрана 2024-07-07 в 23.14.29.png]]


![[Снимок экрана 2024-07-07 в 23.19.13.png]]


![[Снимок экрана 2024-07-07 в 23.22.01.png]]


Because records are totally ordered within their partition, any pair of records in the same partition is bound by a predecessor-successor relationship. This relationship is implicitly assigned by the producer application.

There is no recognised causal ordering across producers; if two (or more) producers emit records simultaneously for the same partition, those records may materialise in arbitrary order. The relative order will not depend on which producer application attempted to publish first, but rather, which record beat the other to the partition leader. That said, whatever the order, it will be consistent — observed uniformly across all consumers. Total order is still preserved, but some record pairs may only be related circumstantially.

Corollary to the above, in the absence of producer synchronisation, ==causal order can only be achieved when a single producer emits records to the same partition==. All other combinations — involving multiple unrelated producers or different partitions — may result in a record stream that fails to depict causality. Whether or not this is an issue will depend largely on the application.


![[Снимок экрана 2024-07-07 в 23.26.46.png]]


So, a consumer siphons records from a topic, pulling from the share of partitions that have been assigned to it by Kafka, alongside the other consumers in its group. As far as load-balancing goes, this is nothing out of the ordinary. But here’s the kicker — consuming a record does not remove it from the topic. This might seem contradictory at first, especially if one associates the act of consuming with depletion. (If anything, a consumer should have been called a ‘reader’, but let’s not dwell on the choice of terminology.) The simple fact is, consumers have absolutely no impact on the topic and its partitions; a topic is an append-only log that may only be mutated by the producer, or by Kafka itself as part of its housekeeping chores. Consumers are ‘cheap’, so to speak — you can have a fair number of them tail the logs without stressing the cluster. This is a yet another point of distinction between an event stream and a traditional message queue, ==and it’s a crucial one==.


A consumer internally maintains an offset that points to the next record in a partition, advancing the offset for every successive read. In fact, a consumer maintains a vector of such offsets — one for each assigned partition. When a consumer first subscribes to a topic, whereby no offsets have been registered for the encompassing consumer group, it may elect to start at either the head-end or the tail-end of the topic. Thereafter, the consumer will acquire an offset vector and will advance the offsets internally, in line with the consumption of records.

Since consumers across different consumer groups do not interfere, there may be any number of them reading concurrently from the same topic. Consumers run at their own pace; ==a slow or backlogged consumer has no impact on its peers==.


==Kafka ensures that a partition may only be assigned to at most one consumer within its consumer group==. (It is said ‘at most’ to cover the case when all consumers are offline.) Because there are three consumers in each group, but only two partitions, one consumer will remain idle — waiting for another consumer in its respective group to depart before being assigned a partition. In this manner, consumer groups are not only a load-balancing mechanism, but also a fence-like exclusion control, used to build highly performant pipelines without sacrificing safety, particularly when there is a requirement that a record may only be handled by one thread or process at any given time.

Consumer groups also ensure availability, satisfying the liveness property of a distributed consumer ecosystem. By periodically reading records from a topic, the consumer implicitly signals to the cluster that it is in a ‘healthy’ state, thereby extending the lease over its partition assignment. Should the consumer fail to read again within the allowable deadline, it will be deemed faulty and its partitions will be reassigned — apportioned among the remaining ‘healthy’ consumers within its group.

![[Снимок экрана 2024-07-07 в 23.37.14.png]]



In the at-least-once scenario, a typical consumer implementation will commit its offsets linearly, in tandem with the processing of a record batch. That is, read a record batch from a topic, process the individual records, commit the outstanding offsets, read the next batch, and so on. This is called a ==poll-process loop==.

A common tactic is to process a batch of records concurrently (where this makes sense), using a thread pool, and only confirm the last record when the entire batch is done. The commit process in Kafka is very efficient, the client library will send commit requests asynchronously to the cluster using an in-memory queue, without blocking the consumer. The client application can register an optional callback, notifying it when the commit has been acknowledged by the cluster. And there is also a blocking variant available should the client application prefer it.


==Free consumers==
The association of a consumer with a consumer group is an optional one, indicated by the presence of a group.id consumer property. If unset, a free consumer is presumed. Free consumers do not subscribe to a topic; instead, the consuming application is responsible for manually assigning a set of topic-partitions to itself, individually specifying the starting offset for each topic-partition pair. Free consumers do not commit their offsets to Kafka; it is up to the application to track the progress of such consumers and persist their state as appropriate, using a datastore of their choosing. The concepts of automatic partition assignment, rebalancing, offset persistence, partition exclusivity, consumer heartbeating and failure detection (safety and liveness, in other words), and other so-called ‘niceties’ accorded to consumer groups cease to exist in this mode.


![[Снимок экрана 2024-07-07 в 23.43.29.png]]





![[Снимок экрана 2024-07-07 в 23.44.18.png]]



![[Снимок экрана 2024-07-07 в 23.44.48.png]]