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

### 2. Create a Kinesis Data Producer using Kinesis Agent
Although optional, it is **highly recommended** to use an EC2 instance for this module to avoid long wait times between downloads/uploads of files.

#### High-Level Instructions

Follow the instructions for setting up [Kinesis Agent](https://docs.aws.amazon.com/streams/latest/dev/writing-with-agents.html) and edit the configuration file `/etc/aws-kinesis/agent.json` to send data to your Kinesis Data Stream as JSON.

Once the agent is running, download a copy of the NYC taxi data files from s3://YOUR_BUCKET_NAME/raw/ to the /tmp directory. If you have configured Kinesis Agent correctly, it should automatically detect the new files, transform the data into JSON and send the record to Kinesis Data Stream.

Verify that your Kinesis Data Stream is receiving data inside the Monitoring section of Kinesis AWS Console.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Create a new EC2 instance using the latest AWS Linux AMI with an IAM role that has the following permissions:
    ``` json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "AllowKinesisWrites",
                "Effect": "Allow",
                "Action": [
                    "kinesis:PutRecord"
                ],
                "Resource": [
                    "arn:aws:kinesis:<REGION>:<ACCOUNTID>:stream/*"
                ]
            },
            {
                "Sid": "AllowCloudWatchMetricsWrites",
                "Effect": "Allow",
                "Action": [
                    "cloudwatch:PutMetricData"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }
    ```
1. SSH into your EC2 instance to download and install the agent
    ``` shell
    sudo yum install –y aws-kinesis-agent
    ```
    
1. Open the Kinesis Agent configuration file and edit using your parameters

    ``` shell
    sudo vim /etc/aws-kinesis/agent.json
    ```
    */etc/aws-kinesis/agent.json*
    ``` shell
    {
        "cloudwatch.emitMetrics": true,
        "kinesis.endpoint": "kinesis.<REGION>.amazonaws.com",
        "flows": [
            {
            "filePattern": "/tmp/*.csv*",
            "kinesisStream": "<KINESIS_DATA_STREAM_NANE>",
            "partitionKeyOption": "RANDOM",
            "initialPosition": "START_OF_FILE",
                "dataProcessingOptions": [
                    {
                        "optionName": "CSVTOJSON",
                        "customFieldNames": ["VendorID","tpep_pickup_datetime","tpep_dropoff_datetime","passenger_count","trip_distance","RatecodeID","store_and_fwd_flag","PULocationID","DOLocationID","payment_type","fare_amount","extra","mta_tax","tip_amount","tolls_amount","improvement_surcharge","total_amount"],
                        "delimiter": ","
                    }
                ]
            }
        ]
    }
    ```

1. Start the Kinesis Agent service
    ``` shell
    sudo service aws-kinesis-agent start
    ```

1.  Copy the NYC taxi data files into the /tmp directory to trigger the Kinesis Agent
    ``` shell
    aws s3 cp s3://YOUR_BUCKET/raw/ . --recursive --region YOUR_REGION
    ```

1. Observe the Kinesis Agent processing your files
    ``` shell
    tail -f /var/log/aws-kinesis-agent/aws-kinesis-agent.log
    ```

1. Navigate to your Kinesis Data Stream in the AWS Console and observe the CloudWatch metrics in the Monitoring tab.

</p></details>

### 3. Create a Kinesis Firehose to S3
In this step, you will create a Kinesis Firehose consumer for the Kinesis Data Stream that will output a copy into an S3 bucket for persistent storage.

As Kinesis Data Stream can serve multiple consumers, Kinesis Firehose is an easy way to output a copy of the data into S3, ElasticSearch, Redshift or Splunk. For any other AWS services or data stores, you will need to create a custom application to consume and process the data using methods such as [Kinesis Client Library](https://github.com/awslabs/amazon-kinesis-client).

#### High-Level Instructions

From the AWS console or CLI, create a new Kinesis Firehose that will be a consumer for the Kinesis Data Stream that we created previously. The Firehose should output the data into the `Firehose/` prefix of your S3 Bucket.

eg. S3://YOUR_BUCKET/Firehose/YYYY/MM/DD/..

Verify that the data is now in JSON format by downloading an object from your S3 bucket in the Firehose prefix.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Create a new Kinesis Firehose from the AWS console or CLI and select the *Source* to the Kinesis Data Stream you created previously.

1. Do not change the values for *Transform source records with AWS Lambda* or *Convert record format* and proceed to the next page (we will cover this in the next module).

1. Select *Amazon S3* as the destination and choose **YOUR_BUCKET_NAME** and **Firehose/** for Prefix and proceed to the next page.

1. Select *Create new or choose* under IAM role and use the default generated policy. Proceed to finish the Firehose launch wizard.

1. Navigate to your Amazon S3 Bucket and verify that new objects are being created in S3://YOUR_BUCKET/Firehose/YYYY/MM/DD/.. Download and confirm that the records are in JSON format and automatically batched to 5MB objects (default value).

</p></details>

### 4. [Advanced - Optional] Data Processing (consumer) with AWS Lambda
Examples of Data Consumers for near-real time streams of data include Hadoop Clusters, Apache Storm and Apache Spark for downstream processing and applications.

In this step, you'll be creating a consumer for the Kinesis Data Stream by creating a new lambda function that will decode the JSON payload and output to the console.

#### High-Level Instructions

Create a new lambda function using this [guide](https://docs.aws.amazon.com/lambda/latest/dg/with-kinesis-example.html) or from an existing lambda blueprint that will detect new records in the stream and output a decoded payload in console/CloudWatch logs.

The output should look similar to this:


``` json
Decoded payload:
{
    "VendorID": "400499",
    "tpep_pickup_datetime": "1",
    "tpep_dropoff_datetime": "2017-12-02 03:04:43",
    "passenger_count": "2017-12-02 03:12:24",
    "trip_distance": "2",
    "RatecodeID": "1.1",
    "store_and_fwd_flag": "1",
    "PULocationID": "N",
    "DOLocationID": "249",
    "payment_type": "234",
    "fare_amount": "1",
    "extra": "7.0",
    "mta_tax": "0.5",
    "tip_amount": "0.5",
    "tolls_amount": "1.65",
    "improvement_surcharge": "0.0",
    "total_amount": "0.3"
}
```

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>
Use an existing lambda blueprint such as kinesis-process-record-python and configure the trigger to your Kinesis Data Stream.

</p></details>




