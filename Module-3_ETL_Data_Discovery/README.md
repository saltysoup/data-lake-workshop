# Module 3 - Glue, Kinesis Firehose and ETL

## Background
In this module, we will create an automated ETL pipeline for our data lake that will convert our NYC taxi data into an optimal file format for analytics. Traditionally, a time-consuming ETL (extract, transform, and load) process is required to massage and convert the data to a optimal columnar format. This situation is less than ideal, especially for real-time data that loses its value over time.

To solve this challenge, we will be using Kinesis Data Firehose and AWS Glue integration to save the incoming data stream into an Apache Parquet format. These are optimized columnar formats that are [highly recommended](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/) for best performance and cost-savings when querying data in S3. This feature directly benefits you if you use Amazon Athena, Amazon Redshift, AWS Glue, Amazon EMR, or any other big data tools that are available from the AWS Partner Network and through the open-source community.

We will then view the content of the parquet file to verify that the ETL has been carried out successfully.

## Prerequisites

* AWS Account
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) with [credentials configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)
* Python 2.7 or 3.6
* Your favourite IDE or Code Editor such as Visual Studio Code

## Implementation Instructions

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### Region Selection

This workshop can be deployed in any AWS region that supports the following services:

- Amazon Kinesis Data Firehose
- Amazon S3
- Amazon Glue


You can refer to the [region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services. Among the supported regions you can choose are N. Virginia, Ohio, Oregon, Ireland, London, Frankfurt, Tokyo, Seol, Mumbai, and Sydney.

Once you've chosen a region, you should deploy all of the resources for this workshop there. Make sure you select your region from the dropdown in the upper right corner of the AWS Console before getting started.



### 1. Build your Data Catalog with AWS Glue
To create an ETL process for converting our JSON files into a Parquet format, you will need to first create a Glue Data Catalog that will discover and store the files' associated metadata (e.g. table definition and schema). After this, Glue will create an Apache Hive compatible Metastore that will allow various use cases such as big data applications running on Amazon EMR, Amazon Athena and Kinesis Data Firehose record format conversion.


#### High-Level Instructions

Use the [crawler feature](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html) of AWS Glue to traverse the JSON files (created in Module 2) in the `Firehose/` prefix of our S3 bucket. Verify that the newly created Data Catalog contains the schema of our files. If the Data type for the schema is incorrect, change this manually referring to the [Data Dictionary](http://www.nyc.gov/html/tlc/downloads/pdf/data_dictionary_trip_records_yellow..pdf). DO NOT run an Athena Query before altering the Data Type within Schema.


<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Navigate to the Glue AWS Console and Add a new crawler

1. After inputting a name, verify that the **Data Store** is set to S3, and the **Include Path** is set to the JSON file store
eg. S3://YOUR_BUCKET/Firehose/

1. Select No for **Add another data store** and proceed to the next screen

1. Choose an existing IAM role or create a new one by inputting a name.

1. Set **Frequency** to Run on Demand and proceed to the next screen

1. Select **Add database**, input a name and Create. Leave the optional Prefix field blank and proceed to finish the creation wizard.

1. After successfully creating the crawler, select and **Run Crawler**

1. After a few minutes, verify that the Data Catalog has been created under **Databases** and **Tables**, with the JSON schema being detected.

1. Verify if the Data Type for the schema is correct by referring to the [data dictionary](http://www.nyc.gov/html/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf) and manually specifying if required. DO NOT run an Athena Query before altering the Data Type within Schema.

</p></details>


### 2. Create a Kinesis Data Firehose with Record Conversion/ETL

#### High-Level Instructions

After creating the Data Catalog and Tables, create a new Kinesis Data Firehose that will [convert the incoming record](https://docs.aws.amazon.com/firehose/latest/dev/record-format-conversion.html) from the Kinesis Data Stream into a parquet format using the Glue Data Catalog. The output for the Firehose should be in the `Parquet/` prefix of your S3 bucket.

eg. S3://YOUR_BUCKET/Parquet/YYYY/MM/DD/..

If you need to generate more data into the Kinesis Data Stream, re-run the Kinesis Agent in Module 2, which will be consumed by the new Kinesis Firehose.


<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Create a new Kinesis Firehose from the AWS console or CLI and select the *Source* to the Kinesis Data Stream you created previously.

1. Set **Record Transformation** to Disabled and use the following configuration:

    - Record format conversion: Enabled
    - Output format: Apache Parquet
    - AWS Glue region: <Your Glue Region>
    - AWS Glue database: <Your Glue Database>
    - AWS Glue table: <Your Glue Table>
    - AWS Glue table version: Latest

1. In Step 3: Choose Destination, choose Amazon S3 for **Destination**

1. For S3 destination, select your S3 bucket and `Parquet/` for **Prefix**

1. Leave **Source record S3 backup** to Disabled and proceed to the next screen

1. In Step 4: Use the default existing settings, and select **Create new or choose** under IAM role and use the default generated policy. Proceed to finish the Firehose launch wizard.

1. After a few minutes, verify that the converted parquet files are being created in the `Parquet` prefix within your S3 bucket.

Hint: If you need to generate more data, run the Kinesis Agent again (Module 2)
</p></details>


### 3. [Advanced - Optional] Verify the contents of converted Parquet files

#### High-Level Instructions

Verify the contents of the parquet files by downloading a local copy, and using a tool such as [pandas](https://pandas.pydata.org/). Parquet files are binary and cannot be viewed normally like a text file.

Install pandas for python
```shell
pip install pandas
```

Use pandas to read parquet and output as a Data Frame
``` python
import pandas as pd

pd.read_parquet('<YOUR PARQUET FILE>')

"""
Example output

tip_amount tolls_amount improvement_surcharge total_amount  
0             0.5          2.0                   0.0          0.3  
1             0.5         1.86                   0.0          0.3  
2             0.5          0.0                   0.0          0.3  
3             0.5         9.39                  5.76          0.3  
"""
```

