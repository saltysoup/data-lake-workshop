# Module 2 - Kinesis, Lambda and S3


## Background
In this module, we will create a Kinesis Data Stream to continuously capture and temporarily store real-time data. To continually generate the incoming data, we will run a local python script as a [Data Producer](https://docs.aws.amazon.com/streams/latest/dev/amazon-kinesis-producers.html) to stream the NYC taxi data set into the Kinesis Data Stream. Additionally, we will also create [Data Consumers](https://docs.aws.amazon.com/streams/latest/dev/amazon-kinesis-consumers.html) using Kinesis Firehose and Lambda function to process the data within the stream.
 

## Prerequisites

* AWS Account
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) with [credentials configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)
* Python 2.7
* Your favourite IDE or Code Editor such as Visual Studio Code

## Implementation Instructions

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### Region Selection

This workshop can be deployed in any AWS region that supports the following services:

- Amazon S3
- AWS Lambda

You can refer to the [region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services. Among the supported regions you can choose are N. Virginia, Ohio, Oregon, Ireland, London, Frankfurt, Tokyo, Seol, Mumbai, and Sydney.

Once you've chosen a region, you should deploy all of the resources for this workshop there. Make sure you select your region from the dropdown in the upper right corner of the AWS Console before getting started.
### 1. Create a Kinesis Data Stream
kinesis data stream

### 2. Create a Kinesis Firehose to S3
kinesis firehose to s3 different prefix

### 4. [Advanced - Optional] Data Processing with AWS Lambda
Data consumer for processing such as Hadoop clusters, Apache Storm, Spark
kinesis-process-record-python blue print




