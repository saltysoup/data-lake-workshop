# Module 2 - Kinesis, Lambda and S3


## Background
In this module, we will create a Kinesis Data Stream to continuously capture and temporarily store real-time data. To continually generate the incoming data, we will run a local python script as a [Data Producer](https://docs.aws.amazon.com/streams/latest/dev/amazon-kinesis-producers.html) to stream the NYC taxi data set into the Kinesis Data Stream. Additionally, we will also create [Data Consumers](https://docs.aws.amazon.com/streams/latest/dev/amazon-kinesis-consumers.html) using Kinesis Firehose and Lambda function to process the data within the stream.


## Prerequisites

* AWS Account
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) with [credentials configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)
* Python 2.7 or 3.6
* Your favourite IDE or Code Editor such as Visual Studio Code
* EC2 instance

## Implementation Instructions

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### Region Selection

This workshop can be deployed in any AWS region that supports the following services:

- Amazon Kinesis Data Stream
- Amazon Kinesis Data Firehose
- Amazon S3
- AWS Lambda
- Amazon EC2


You can refer to the [region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services. Among the supported regions you can choose are N. Virginia, Ohio, Oregon, Ireland, London, Frankfurt, Tokyo, Seol, Mumbai, and Sydney.

Once you've chosen a region, you should deploy all of the resources for this workshop there. Make sure you select your region from the dropdown in the upper right corner of the AWS Console before getting started.

### 1. Create a Kinesis Data Stream
You can use [Kinesis Data Streams](https://aws.amazon.com/kinesis/data-streams/) for rapid and continuous data intake and aggregation. The type of data used can include IT infrastructure log data, application logs, social media, market data feeds, and web clickstream data. Because the response time for the data intake and processing is in real time, the processing is typically lightweight. In this step, you'll create a new Data Stream and put new records in, which we will later consume to process downstream.


#### High-Level Instructions

Use the console or AWS CLI to create a Kinesis Data Stream with 1 Shard (1MB Write/sec and 2MB Read/sec)

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **Services** then select **Kinesis** under Analytics.

1. Choose **+Create Data Stream**

1. Provide a name for your stream such as `Stream-firstname-lastname`.

1. For Number of shards, use 1 (sufficient for this module)

1. Select Create Kinesis Stream

Alternatively, use the AWS CLI command
aws kinesis create-stream --stream-name YOUR_STREAM_NAME --shard-count 1 --region YOUR_REGION

</p></details>

### 2. Create a Kinesis Data Producer
Although optional, it is **highly recommended** to use an EC2 instance for this module to avoid long wait times between downloads/uploads of files.

Download a copy of all the NYC taxi data files from s3://YOUR_BUCKET_NAME/raw/ and create a script that will convert the CSV files and upload to your Data Stream as a JSON payload.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. SSH into working instance and download taxi data
    ssh -i <your_local_ssh_key.pem> ec2-user@<EC2_Public_IP_Address>
    mkdir downloads
    cd downloads
    aws s3 sync s3://YOUR_BUCKET/raw/ .

1. Create a new python script that will convert the downloaded CSV files into JSON

``` python

```

3. Create a new Kinesis Data Producer script that will put records into our Data Stream 

``` python



```



</p></details>

### 3. Create a Kinesis Firehose to S3
In this step, you will create a Kinesis Firehose consumer for the Kinesis Data Stream that will read off the stream's records and output a copy into an S3 bucket for persistent storage.

As Kinesis Data Stream can serve multiple consumers, Kinesis Firehose is an easy way to output a copy of the data into S3, ElasticSearch, Redshift or Splunk. For any other AWS services or data sources, you will need to create a custom application to consume and process the data using methods such as [Kinesis Client Library](https://github.com/awslabs/amazon-kinesis-client)

1. Create a new Kinesis Firehose from the AWS console or CLI and select the Source to the Kinesis Stream you created previously.

1. Do not change the values for *Transform source records with AWS Lambda* or *Convert record format* and proceed to the next page (we will cover this in the next module)

1. Select Amazon S3 as the destination and choose YOUR_BUCKET_NAME and 

### 4. [Advanced - Optional] Data Processing with AWS Lambda
Data consumer for processing such as Hadoop clusters, Apache Storm, Spark
kinesis-process-record-python blue print




