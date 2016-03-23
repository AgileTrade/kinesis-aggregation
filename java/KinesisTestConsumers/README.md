# Kinesis Producer Library Compatible Java Lambda Processor

This Lambda function will read events from Kinesis and perform the necessary deaggregation of KPL encoded data. It provides a few examples of how to process Kinesis data in Java from AWS Lambda which you can use to build new applications, depending on the style of code you prefer.

## Lambda Function

The code that provides our Lambda function is `KinesisLambdaReceiver.java`, which implements the `handleRequest` interface twice, once for each style of code. This code works in the same way, but you may find you prefer the Java 8 style using Streams, or alternatively the pre-Java 8 method of using Lists. If you want to use the List style method, then you'll need to change the method `handleRequestWithLists` to `handleRequest`.

## KPL Deaggregator

For details on how to use the `KplDeaggregator`, see the `KinesisDeaggregator` module in this repository.

## Instructions for Use

1. Run Maven->Install to build the project
2. Create a new Lambda function in your AWS account
3. Skip blueprint selection
4. Choose Java 8 as the runtime
5. Choose the built file (from step #2) KinesisLambdaTestConsumer-1.0-dev.jar as the code for the function (NOT the KinesisLambdaTestConsumer-0.0.1.jar file).
6. Choose com.amazonaws.KinesisLambdaReceiver as the Handler
7. Set the default batch size as required for your Stream throughput
8. Set the Role, Memory and Timeout appropriately.
9. Connect your new Lambda function to the Kinesis stream you'll be publishing to

## IAM Role

This is a sample IAM policy for the Lambda execution role:

```
 {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:InvokeFunction"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "kinesis:GetRecords",
        "kinesis:GetShardIterator",
        "kinesis:DescribeStream",
        "kinesis:ListStreams",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

----

Copyright 2014-2015 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License. A copy of the License is located at

    http://aws.amazon.com/asl/

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the License for the specific language governing permissions and limitations under the License.