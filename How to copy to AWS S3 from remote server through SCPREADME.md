# How to copy to AWS S3 from remote server through SCP 

### Introduction

This article will demonstrate how to copy files straight from a remote server to S3 without first copying them to your local machine.and amazon s3 scp


scp->remote server->s3 from bastion host Know more about [developer associate training]

[//]: # (Any comments) 
[developer associate training]: <https://www.netcomlearning.com/certification/aws-certified-developer-associate/602/> 


I'm going to rename it to make it easier to grasp.
Remote Server as A,
Bastion Host as B,
AWS S3 as C.

## Before:
copy from A->B then B->C.
You might wonder why someone would wish to duplicate in this manner. It could be because server B is on a private network and can only be accessing from A. (Bastion host, jump-box, etc.). In a situation comes like this, it's advisable to use DB.

### Now:
copy from A->C by SCPing in B
Now we are going to use Secure Copy Protocol to copy files directly from server A->C I mean Remote Server->S3
scpuser@ip-of-A:/path/to/file >(aws s3 cp - s3:/s3uri/)
This cmd is like regular SCP to copy from remote to local with some change. The stdin that is provided to the aws cli using bash's process substitution is represented by -. () (-will be replaced with the file specified in the SCP command's /path/to/file source section)
Add the file name to the S3 URI like this to modify the file name at the time of copying:
scpuser@ip-of-A:/path/to/file >(aws s3 cp - s3:/s3uri/filename)
Replace the filename with the name you want. That concludes this page; please email me if you have any questions or would like to update the information.

### Conclusion 
In this article, you can learn about the procedure of the AWS S3 copies from the remote server. You get a step-by-step working method of copying AWS S3 from a remote server using SCP.
If you are looking for training on how to build applications in AWS cloud, you can take up the [Developing on AWS course]. It is an intermediate level course designed for cloud developers. [AWS Developer Training in Arlington]

[//]: # (Any comments) 
[Developing on AWS course]: <https://www.netcomlearning.com/developing-on-aws/course/26175/>  

[//]: # (Any comments) 
[AWS Developer Training in Arlington]: <https://www.netcomlearning.com/aws-developer-arlington-tx/local/product/amazon-web-services/us/1355-362/> 
