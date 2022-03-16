### GCP BigTable vs. AWS DynamoDB.

![What the Future of Azure Work Looks Like After Coronavirus (56)](https://user-images.githubusercontent.com/97124802/158589017-c8cb3396-f418-4f47-a20c-e528d7585f5d.jpg)


## Introduction

As you may recall from my previous 2020 blog post, one of my new goals is to become skilled in using AWS, Azure, and GCP data services. Finding patterns and detecting discrepancies is one of the process’ building components. And before completing the exercise for BigTable (GCP) and DynamoDB (AWS), I felt both were practically the same. You can't fathom how completely incorrect I was with this assumption!
The post will briefly describe these two services and their primary applications. The differences between them will be in subsequent sections, and the similarities summarized in the last quarter. The goal isn't to determine which is superior. Instead, I wanted to highlight the contrasts and similarities to provide you with a quick start guide.

## Architecture

Let's start with the architecture itself, or at least what we can deduce from the documentation. DynamoDB is a serverless design, which means all you have to do is provide the table's read and write capabilities. On the other hand, you have a bit more setup to complete with BigTable. You must not just define an instance. The instance is a container for the clusters in charge of data storage. For the definition, you can provide the aspects like storage type (SSD, HDD) or the application profile to determine how the consumers will connect to the instance. From this initial architecture comparison, you can see that DynamoDB uses the serverless paradigm and that you work at the table scope, i.e., define a table's throughput capacity. BigTable is unique in that you can work with all tables directly through the instances you've established.

The operational overhead is also affected by serverless vs. server-based differences. It would help if you provisioned your infrastructure for BigTable, whereas you may utilize DynamoDB's auto-scaling feature to adapt the throughput to the current workload. But don't get me wrong: I'm not a snob. BigTable can also be scaled! The difference is that you'll have to create your scaling application and have it run in response to metrics like CPU or request throughput that are monitoring.

However, scaling isn't the only issue we may address in this architecture-related part. Another one concerns replication and, more broadly, availability. Data storage is on SSDs duplicated over many zones within a single region via DynamoDB. It also includes bespoke, on-demand backups and continuous backups to enable Point-In-Time Recovery (PITR) - you can travel back in time up to 35 days!
In the case of BigTable, replication is explicit, which means you must declare whether you want it or not. However, it allows you more freedom because you may delegate the load to a single cluster using application profiles. Also, the backups aren't precisely the same. Because they're stored in the group and bound to the cluster's lifecycle if the cluster zone goes down, so does the backup, and there is no option to export backups to another location (e.g., GCS). It doesn't happen with DynamoDB's PITR feature, which preserves the most recent backup for 35 days after deleting the table. However, the distinction is due to a conceptual difference: BigTable is a container of tables, whereas DynamoDB is a single table.

## Storage of data

Apart from the variations in architecture, these data stores have different storage issues. The access is key-based, but that's where the similarity ends. The support for additional indexes is the first difference. Local (same partition key but distinct sort key) and global (different partition key) indexes are both supported by DynamoDB. Secondary indices don't exist in BigTable. And, while we're talking about table keys, it's worth noting that BigTable doesn't support the sort key either. Because the rows are always sorted lexicographically by the row key, it must be unique. If the sort key column is available in DynamoDB, you can store multiple rows that share the table's same "primary" key.
Even though it looks like a drawback, BigTable offsets it partly with the enormous sparse storage. The maximum size of aBigTable row (recommended maximum size is 100MB) is significantly larger than that of DynamoDB (400 KB). This fact can compensate for the lack of a secondary index and generate large tables with multiple events per row. At the cell level, BigTable storage is sparse. Therefore, a missing attribute does not occur. However, please notice that even if it's a theoretically conceivable option. The suggested method for time series is to utilize tall and narrow tables (fewer events per row) and finally fall back to the wide tables only If it's possible to manage the number of columns.
By the way, this also brings up two other points. The storage of row attributes is the first of them. The columns in BigTable must be like column families. With each column's columns arranged lexicographically. And it's one of the performance levers because every column family's data is stored and cached together. If you query it every time, you can optimize the reading path unconsciously. I couldn't discover any reference of the attributes collocation in DynamoDB.
The second point is the advice on how to deal with time-series data. According to DynamoDB's documentation, working with time-partitioned tables is the best practice. On the other hand, BigTable recommends having fewer tables, and the general guideline is not to build a new table for datasets with the same schema. Multiple tables can raise connection overhead and reduce the efficiency of automatic load balancing. learn more [aws solutions architect associate certification]

[//]: # (Any comments) 
[aws solutions architect associate certification]: <https://www.netcomlearning.com/certification/aws-certified-solutions-architect-associate/599/> 

## Reading the information

So, we know previously about data storage, but what to do to query this data? The single-row read, utilizing only the primary key, is the most straightforward read operation in the BigTable. On the DynamoDB side, the identical method exists, but because it supports the table's secondary key, you can specify different fetch criteria.
The read with row key prefix is the second exciting query type in BigTable. Even though there is an operation in DynamoDB, it does not exist. This prefix-based search, unlike BigTable, can only be used on the sort keys.
The range read is the final read type in which BigTable surpasses DynamoDB. You must provide the start and end of the interval if you want rows to be within a row key range. For example, to receive a player's goals scored between January 1, 2020, and March 1, 2020, you need to set two range keys, player#20200101 and player#20200301. In DynamoDB, you'd have to generate all dates between these two and query the duplicate keys to get the same data. The excellent thing to keep in mind here is that you can accomplish it with batch queries to increase throughput.
Apart from these significant query options, both data stores allow table scans, which are, unsurprisingly, less optimal than key-based fetches in both cases. If you don't have the data to process in more scalable and parallelizable storage like an object store, you can utilize application profiles in BigTable to enhance this scanning feature. Yes, I repeat it once again, but it's something that doesn't exist in DynamoDB and seems to be a means to optimize several things.
Suppose you set up numerous clusters in your BigTable instance. In that case, you can construct an application profile with the single-cluster routing policy to direct all batch consumer queries to a specified group. You can then separate the real-time and offline tasks in this manner. For this scenario, you can't control the reads in DynamoDB, but you can always export the data to S3 and query your items.

## Conclusion

Why did I think they were the same? Mainly due to the suggested use scenarios. For use cases needing low latency access. Both are query-driven databases in which the schemas - and, more specifically, the tables' primary keys- are defined according to the query patterns. Finally, both data stores are schemaless and dependent on crucial access. However, this is merely a high-level picture, and if we dig deeper, we'll discover that they are just appearances.
If you are looking for training on how to develop cloud infrastructure for AWS, you can take up the [Architecting on AWS] course. It is an intermediate level course designed for cloud architects. Know more about [AWS Architect Training Courses in Calgary]

[//]: # (Any comments) 
[Architecting on AWS]: <https://www.netcomlearning.com/architecting-on-aws/course/34767/> 

[//]: # (Any comments) 
[AWS Architect Training Courses in Calgary]: <https://www.netcomlearning.com/aws-architect-calgary-ab/local/product/amazon-web-services/ca/1354-117/>
