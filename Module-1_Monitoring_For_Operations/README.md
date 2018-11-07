# Module 1:

1. Launch CloudFormation template (2 tier wordpress on EC2 and RDS)
1. Set up CloudWatch Agent for memory and Disk
1. CloudWatch Dashboard

1. CloudWatch Logs and ElasticSearch
1. Health Dashboard on ES (CPU/Memory/DiskQueueLength/DatabaseConnections/FreeStorageSpace)

## Introduction
In this module, we will create a monitoring dashboard to gain visibility into the operational health of our infrastructure. By surfacing these metrics, we can detect potential problems earlier on to spot capacity failures, and gain insights on demand pattern for time series analysis and cost savings through elasticity. We will also use AWS WAF to log incoming HTTP requests and create a rule to observe Blocked and Allowed requests through CloudWatch Logs and Kibana on Elasticsearch. 

For our working environment, we will use a 2-tier Wordpress Site running on EC2 and Amazon RDS for MySQL DB. Scroll down to get started!

## Architecture
![Module_1_Architecture1](images/Module1_Architecture1.png)
![Module_1_Architecture2](images/Module1_Architecture2.png)

## Creating our Wordpress Environment
CloudFormation instructions


## Setting up CloudWatch Agent
We do this because there are metrics for EC2 instance that are not visible at the hypervisor level such as Memory and Disk free space. If we were to run in an hybrid environment, we can use CloudWatch Agent for on-premises servers as well.


## CloudWatch Dashboard
Create a Dashboard through CloudWatch as it has integration with our Metrics. If we wanted to run our CloudWatch Dashboard outside of AWS Console, here is a [blog article](https://aws.amazon.com/blogs/devops/building-an-amazon-cloudwatch-dashboard-outside-of-the-aws-management-console/)


## Creating a Kinesis Firehose
Now that our ETL Lambda has been created, let's create a new Kinesis Data Firehose that will use our Elasticsearch domain as the delivery destination.


1. In the AWS Management Console select **Services** then select **Kinesis** under Analytics.

1. In the service console, select **Get started** and select **Create delivery stream** for the Kinesis Firehose wizard.

1. Under **New delivery stream**, give your Delivery stream name such as `webserver_stream`.

1. Under **Choose source**, verify that **Direct PUT or other sources** is selected. Although Kinesis Data Firehose can be configured to ingest data from a Kinesis Data stream (non-firehose), we will be using an agent to push into our Firehose stream.

1. Proceed to the next page by selecting **Next**.

1. In Step 2: Process records, enable **Record transformation** using the Lambda function we created previously (eg. apache_ETL).

1. Verify that Record format conversion is **Disabled** under **Convert record format**. If we wanted to deliver the data within Firehose for running analytics within Redshift or S3 (via Athena), this would be a great way to automatically transform the data into a columnar format such as Parquet for a more efficient file format for analytics.

    ![Firehose_Lambda](images/Firehose_Lambda.png)

1. Proceed to the next page by selecting **Next**.

1. Under **Select destination**, select **Amazon Elasticsearch Service** to view our existing domain.

1. Under **Amazon Elasticsearch Service destination**, select our existing cluster for **Domain**.

1. For **Index**, enter a name such as `apache`.

1. Select **No rotation** for index rotation frequency and enter a name for **Type** such as `clickstream`.

1. Under **S3 backup**, we can select whether a copy of the records from our Firehose is automatically backed up into an S3 bucket, or only for records that fails to be processed. For this workshop, select **All records** to view the data later on to have a look at the ingested data.

1. Create or use an existing S3 bucket and for **Backup S3 bucket prefix**, enter a name followed by underscore such as `apache_`. This will make it easier later on to identify which prefix our firehose backups the records into.

1. Your settings should look similar to this

    ![Firehose_ES](images/Firehose_ES.png)


1. Proceed to the next page by selecting **Next**.

1. In Step 4: Configure Settings, select a **Buffer size** of 1 MB and B**uffer interval** of 60 seconds. As Firehose automatically buffers and aggregates the data in the stream, it will wait until either of these conditions are met before triggering the delivery. If you need to ensure faster (lower) availability of data in the stream, Kinesis Data Stream allows a more immediate window.

1. For **S3 compression and encryption** check that the settings are set to **Disabled**, and for **Error logging**, ensure that it is **Enabled** for future troubleshooting if required.

1. Under **IAM Role**, select **Create new or choose** to bring up a new IAM role creation page. Under IAM Role, use the drop down menu to select **Create a new IAM Role**. This will automatically generate the permissions required for our Firehose to use the configured settings for CloudWatch, S3, Lambda and ElasticSearch.

    ![Firehose_ES](images/Firehose_IAM.png)

1. Proceed to the next step by selecting **Allow**.

1. Verify that the settings are configured as above, and finish the wizard by selecting **Create delivery stream**. This will take 5-7 min to complete creating the new stream.

<details>
<summary><strong>Creating a Kinesis Firehose to Elasticsearch Step-By-Step Instructions (expand for details)</strong></summary><p>

</p></details>


## RDS Log to Firehose

## Create an Amazon Elasticsearch cluster
With data being generated from multiple different sources, we need a way to search, analyze and visualize it to extract meaningful information and insights. To enable this, we can use Amazon Elasticsearch, a fully managed service that helps with the deployment, operations and scaling the open source Elasticsearch engine. Using the integrated Kibana tool, we can create custom visualizations and dashboards to enable us to make faster and more informed decisions about our environment!

1. In the AWS Management Console select **Services** then select **Elasticsearch Service** under Analytics.

1. In the service console, select **Create a new domain**

1. Under **Domain Name** enter a name for your cluster and ensure that the selected **Version** is **6.3**. Proceed to the **Next** step.

1. In this page, we can configure the number of nodes and HA settings for our Elasticsearch cluster. We will use the default single node setting, but for production environments, you would use multiple nodes across different availability zones using the **zone awareness** setting. Continue by leaving all settings as-is under **Configure cluster** and proceed to the **Next** step. 

1. In step 3: Set up access page, you can configure Network level access and Kibana login authentication through integration with Amazon Cognito (SAML with existing IdP is supported through Cognito). In production environments, you would launch the Elasticsearch cluster in a private subnet (where it is accessible via VPN or a proxy in a DMZ). However for this workshop, we will use **Public Access** to allow access to the Kibana dashboard over the internet.

1. Select **Public access** under **Network configuration** and leave the **Node-to-node** encryption unchecked.

1. Leave **Enable Amazon Cognito for authentication** unticked under **Kibana authentication**.

1. Under **Access policy**, use the drop down menu on **Select a template**, and select **Allow access to the domain from specific IP(s)**.

1. Enter your current IP address in the pop up window (you can find out your current public address by googling "Whats my IP"). This should automatically generate an access policy in JSON like below.

    ```json
        {
        	"Version": "2012-10-17",
        	"Statement": [{
        		"Effect": "Allow",
        		"Principal": {
        			"AWS": "*"
        		},
        		"Action": [
        			"es:*"
        		],
        		"Condition": {
        			"IpAddress": {
        				"aws:SourceIp": [
        					"54.240.193.1"
        				]
        			}
        		},
        		"Resource": "arn:aws:es:ap-southeast-1:708252083442:domain/realtimeworkshop/*"
        	}]
        }
    ```
1. Proceed to the next page by selecting **Next**.

1. Review that all the settings are correct using the above configurations and create the cluster by selecting **Confirm**. The cluster will take 10-15 min to launch.


## AWS WAF
