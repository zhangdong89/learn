#The question when using storm
1.An error happend when emitting data to Elasticsearch.
  
    java.lang.RuntimeException: java.lang.NullPointerException
	          at backtype.storm.utils.DisruptorQueue.consumeBatchToCursor(DisruptorQueue.java:128) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.utils.DisruptorQueue.consumeBatchWhenAvailable(DisruptorQueue.java:99) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.disruptor$consume_batch_when_available.invoke(disruptor.clj:80) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.daemon.executor$fn__3441$fn__3453$fn__3500.invoke(executor.clj:748) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.util$async_loop$fn__464.invoke(util.clj:463) ~[storm-core-0.9.3.jar:0.9.3]
	          at clojure.lang.AFn.run(AFn.java:24) [clojure-1.5.1.jar:na]
	          at java.lang.Thread.run(Unknown Source) [na:1.7.0_51]
    Caused by: java.lang.NullPointerException: null
	          at org.codehaus.jackson.util.TextBuffer.findBuffer(TextBuffer.java:207) ~[jackson-core-asl-1.5.0.jar:1.5.0]
	          at org.codehaus.jackson.util.TextBuffer.emptyAndGetCurrentSegment(TextBuffer.java:476) ~[jackson-core-asl-1.5.0.jar:1.5.0]
	          at org.elasticsearch.hadoop.serialization.json.BackportedJsonStringEncoder.quoteAsString(BackportedJsonStringEncoder.java:121) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.util.StringUtils.jsonEncoding(StringUtils.java:360) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.serialization.field.AbstractIndexExtractor.append(AbstractIndexExtractor.java:121) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.serialization.field.AbstractIndexExtractor.field(AbstractIndexExtractor.java:130) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.serialization.bulk.AbstractBulkFactory$FieldWriter.write(AbstractBulkFactory.java:94) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.serialization.bulk.TemplatedBulk.writeTemplate(TemplatedBulk.java:80) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.serialization.bulk.TemplatedBulk.write(TemplatedBulk.java:56) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.hadoop.rest.RestRepository.writeToIndex(RestRepository.java:148) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at org.elasticsearch.storm.EsBolt.execute(EsBolt.java:116) ~[elasticsearch-storm-2.1.0.jar:2.1.0]
	          at backtype.storm.daemon.executor$fn__3441$tuple_action_fn__3443.invoke(executor.clj:633) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.daemon.executor$mk_task_receiver$fn__3364.invoke(executor.clj:401) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.disruptor$clojure_handler$reify__1447.onEvent(disruptor.clj:58) ~[storm-core-0.9.3.jar:0.9.3]
	          at backtype.storm.utils.DisruptorQueue.consumeBatchToCursor(DisruptorQueue.java:125) ~[storm-core-0.9.3.jar:0.9.3]
	          ... 6 common frames omitted  
             
According the up error message, java.lang.NullPointerException happened,  
so I check the org.codehaus.jackson.util.TextBuffer code under version 1.5.x.
We can find that when **_allocator** is null, findBuffer method will thow java.lang.NullPointerException.
``` java
    /**
     * Helper method used to find a buffer to use, ideally one
     * recycled earlier.
     */
    private final char[] findBuffer(int needed)
    {
        return _allocator.allocCharBuffer(BufferRecycler.CharBufferType.TEXT_BUFFER, needed);
    }
```
Then I change org.codehaus.jackson.util.TextBuffer version to 1.9.13 which has juded whether _allocator is null.
``` java
    /**
     * Helper method used to find a buffer to use, ideally one
     * recycled earlier.
     */
    private final char[] findBuffer(int needed)
    {
        if (_allocator != null) {
            return _allocator.allocCharBuffer(BufferRecycler.CharBufferType.TEXT_BUFFER, needed);
        }
        return new char[Math.max(needed, MIN_SEGMENT_LEN)];
    }
```
