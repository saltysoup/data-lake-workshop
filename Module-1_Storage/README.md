# Module 1 - S3, AWS CLI and Lambda

## Background
In this module, we will create a S3 bucket that will hold all of our raw data, which will serve as the starting point for the data lake. As [Amazon S3](https://aws.amazon.com/s3/) is a durable, cost effective object storage service, it serves as an ideal storage medium for holding both structured and unstructued data.

We will also explore how to use the AWS CLI to interact with the S3 bucket, and use S3 events notification feature to trigger an event driven function via [AWS Lambda](https://aws.amazon.com/lambda/).

## Data Set

The data set that you are going to use is a public data set that includes trip records from all trips completed by Yellow taxis in New York City in the month of [December, 2017](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml) (~9508278 rows). Records include fields capturing pick-up and drop-off dates/times, pick-up and drop-off locations, trip distances, itemized fares, rate types, payment types, and driver-reported passenger counts. The data set has been split into smaller, individual files of 50k rows to better reflect a time series of streaming data.

By the end of the workshop, our goal is to convert the row based, CSV file format to a more efficient, column based Apache Parquet format.

## Prerequisites

* AWS Account
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) with [credentials configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)
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


### 1. Create an S3 Bucket

Amazon S3 can be used to host static websites without having to configure or manage any web servers. In this step you'll create a new S3 bucket that will be used to host all of the static assets (e.g. HTML, CSS, JavaScript, and image files) for your web application.

#### High-Level Instructions

Use the console or AWS CLI to create an Amazon S3 bucket. Keep in mind that your bucket's name must be globally unique across all regions and customers. We recommend using a name like `RawData-firstname-lastname`. If you get an error that your bucket name already exists, try adding additional numbers or characters until you find an unused name.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **Services** then select **S3** under Storage.

1. Choose **+Create Bucket**

1. Provide a globally unique name for your bucket such as `RawData-firstname-lastname`.

1. Select the Region you've chosen to use for this workshop from the dropdown.

1. Choose **Create** in the lower left of the dialog without selecting a bucket to copy settings from.

</p></details>

### 2. Upload Content

#### High-Level Instructions

Upload the NYC data set for this module to your S3 bucket inside prefix /raw. You can use the AWS CLI tool or the AWS Management Console (requires Google Chrome browser). If your local internet speed is too slow or the upload process takes a long time, [launch a new EC2 instance](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html) and use the AWS CLI on the remote terminal.

<details>
<summary><strong>CLI step-by-step instructions (expand for details)</strong></summary><p>

Execute the following command making sure to replace `YOUR_BUCKET_NAME` with the name you used in the previous section and `YOUR_BUCKET_REGION` with the region code (e.g. ap-southeast-2) where you created your bucket.
    ``` shell
    ssh -i <your_local_ssh_key.pem> ec2-user@<EC2_Public_IP_Address>
    ```
    ``` shell
    aws s3 cp s3://injae-groupm/ s3://YOUR_BUCKET_NAME/raw/ --recursive --region YOUR_BUCKET_REGION
    ```
If the command was successful, you should see a list of objects that were copied to your bucket.
</p></details>

### 3. Event Driven Notification with S3

#### High-Level Instructions

[Enable S3 event notification](https://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html) for your bucket where a `ObjectCreate(All)` operation inside prefix `/raw` will trigger a Lambda function. You can use an existing blue print lambda function called `s3-get-object` and test to see if it is triggered automatically by reuploading the taxi data set to your bucket.

You can view successful lambda invocation in the lambda console and the console output in CloudWatch Logs.

<details>
<summary><strong>S3 Events and Lambda step-by-step instructions (expand for details)</strong></summary><p>

1. Create a new lambda function via AWS console in the same region as your S3 bucket.

1. Select Blueprints then add keyword `s3-get-object` to the filter

1. Select either nodejs or python runtime

1. Use the following configurations:
    - Name: <Your Function Name>
    - Role: <Create new role from template(s)>
    - Role Name: <Your Lambda Role Name>
    - Policy templates: <Leave Blank or S3 object read-only permissions>

1. Enable S3 Trigger for your S3 bucket
    - Event type: Object Created (All)
    - Prefix: raw/
    - Enable trigger: Checked

1. Finish Creating the function

1. Upload the taxi data set again to the S3 bucket (check upload path to /raw) and look for Invocations in the Monitoring Tab from Lambda console. Check for console output by clicking on View logs in CloudWatch. You should have an output similar to this CONTENT TYPE: text/csv
    
    ``` shell
    aws s3 cp s3://YOUR_BUCKET/raw/ s3://YOUR_BUCKET/raw/ --recursive --metadata-directive REPLACE --region YOUR_BUCKET_REGION
    ```

</p></details>

### 4. [Advanced - Optional] Basic ETL with AWS Lambda

#### High-Level Instructions

Create a Lambda function that will parse the taxi data files and look for rows where the value for VendorID column is 2. Then, output the results into new CSV files called `FILENAME-VendorID-2.csv` in prefix VendorID2.

eg. S3://YOUR_BUCKET/VendorID2/0-VendorID-2.csv

Normally, you would use an alternative compute method such as EC2 or distributed MapReduce/Spark for long running jobs involving large amount of data. This task is demonstrate a serverless way to ETL (or when the incoming file sizes are small enough for lambda)

<details>
<summary><strong>Example Python Lambda code (expand for details)</strong></summary><p>

1. Create a new lambda function that will be triggered by new objects created in the /raw prefix (you will need to include the pandas module as part of the lambda deployment file)

1. Lambda function with S3 event trigger
``` python
import io
import os
import json
import urllib.parse
import boto3
import pandas as pd

print('Loading function')

s3_client = boto3.client('s3')


def lambda_handler(event, context):
    #print("Received event: " + json.dumps(event, indent=2))

    # Get the object from the event
    BUCKET_NAME = event['Records'][0]['s3']['bucket']['name']
    FILE_NAME = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')
    PREFIX_NAME = "raw"

    # load the data in memory (lambda's /tmp allows 512MB ephemeral disk space)
    s3_response_object = s3_client.get_object(Bucket=BUCKET_NAME, Key=PREFIX_NAME + "/" + FILE_NAME)

    # using pandas dataframe for interacting with data
    df = pd.read_csv(io.BytesIO(s3_response_object['Body'].read()))
    df2 = df.loc[df['VendorID'] == 2]

    try:                               
        print ("writing data to file")
        fileName = "/tmp/{}".format(FILE_NAME)
        df2.to_csv(fileName, sep=',', encoding='utf-8')
        data = open(fileName, 'rb')
        # upload to s3 and delete tmp file
        response = s3_client.put_object(
            Bucket=BUCKET_NAME,
            Key="VendorID2" + "/" + FILE_NAME + "-VendorID-2.csv",
            Body=data
        )
        print ("S3 upload response: {}".format(response))
        data.close()
        os.remove(fileName) # clean up
    except Exception as e:
        print ("something went wrong: {}".format(e))
        pass

    


```
</p></details>