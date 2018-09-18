# Module 4 - Serverless Query with Athena

## Background
In this module, we will create a Data Catalog for the newly converted parquet files and use Amazon Athena to query directly into our S3 bucket. 


## Prerequisites

* AWS Account

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
In this step, now that we have an analytics optimized parquet format, try creating a new Data Catalog using the Glue crawler and run queries using Amazon Athena.


