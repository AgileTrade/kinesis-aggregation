# Kinesis Producer Library Compatible Java Record Aggregator

This library provides a set of convenience functions to perform in-memory record aggregation that is compatible with the same Google Protobuf format used by the full Kinesis Producer Library.  The focus of this module is purely record aggregation though; if you want the load balancing and other features of the KPL, you will still need to leverage the full Kinesis Producer Library.

## KPL Aggregator

The `KplAggregator` is the class that does the work of accepting individual Kinesis user records and aggregating them into a single aggregated Kinesis record (using the same Google Protobuf format as the full Kinesis Producer Library).  The `KplAggregator` provides two interfaces for aggregating records: batch-based and callback-based.

### Batch-based Aggregation

The batch aggregation method involves adding records one at a time to the `KplAggregator` and checking the response to determine when a full aggregated record is available.  The `addUserRecord` method returns `null` when there is room for more records in the existing aggregated record or it returns a `KplAggRecord` object when a full record is available for transmission.

A sample implementation of batch-based aggregation is shown below.

```
for (int i = 0; i < numRecordsToTransmit; i++)
{
    String pk = /* get record partition key */;
    String ehk = /* get record explicit hash key */;
    byte[] data = /* get record data */;

    KplAggRecord aggRecord = aggregator.addUserRecord(pk, ehk, data);
    if (aggRecord != null)
    {
        ForkJoinPool.commonPool().execute(() ->
        {
            kinesisClient.putRecord(aggRecord.toPutRecordRequest("myStreamName"));
        });
    }
}
```

The `ForkJoinPool.commonPool().execute()` method above executes the actual transmission to Amazon Kinesis on a separate thread from the Java 8 shared `ForkJoinPool`. 

You can find a full working sample of batch-based aggregation in the `SampleAggregatorProducer.java` class in the `KinesisTestProducers` project.

### Callback-based Aggregation

For those that prefer more asynchronous programming models, the callback-based aggregation method involves register a callback function (which can be a Java 8 lambda function) via the `onRecordComplete` function that will be notified when an aggregated record is available.

A sample implementation of callback-based aggregation is shown below.

```
aggregator.onRecordComplete((aggRecord) ->
{
    kinesisClient.putRecord(aggRecord.toPutRecordRequest("myStreamName"));
});

for (int i = 0; i <= numRecordsToTransmit; i++)
{
    String pk = /* get record partition key */;
    String ehk = /* get record explicit hash key */;
    byte[] data = /* get record data */;
    
   aggregator.addUserRecord(pk, ehk, data);
}
```

By default, the KPL Aggregator will use a new thread from the Java 8 shared `ForkJoinPool` to execute the callback function, but you may also supply your own `ExecutorService` to the `onRecordComplete` method if you want tighter control over the thread pool being used.

You can find a full working sample of batch-based aggregation in the `SampleAggregatorProducer.java` class in the `KinesisTestProducers` project.

### Other Implementation Details

When using the batch-based and callback-based aggregation methods, it is important to note that you're only given a KplAggRecord (via return value or callback) when the `KplAggregator` object has a full record (i.e. as close to the 1MB PutRecord limit as possible).  There are certain scenarios, however, where you want to be able to flush records to Kinesis before the aggregated record is 100% full.  Some example scenarios include flushing records at application shutdown or making sure that records get flushed every N minutes.

To solve this problem, the `KplAggregator` object provides a method called `flushAndFinish` that will returned an aggregated record that contains all the existing records in the `KplAggregator` as a `KplAggRecord` object (even if it's not completely full).

----

Copyright 2014-2015 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License. A copy of the License is located at

    http://aws.amazon.com/asl/

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the License for the specific language governing permissions and limitations under the License.