# Module 3

1. Set up Connect instance
1. Set up call flow and call recording to firehose
1. Generate call logs through producer script
1. In the data streaming Firehose, use transformation Lambda to ETL and use API (Injae making this) to search for username based on phone number as input
1. Publish to ElasticSearch
1. Verify with Kibana showing number of incoming calls, average wait time, available agents.
1. Because we have username + phone number, we can use the index from clickstream logs to look up historical clickstream activity