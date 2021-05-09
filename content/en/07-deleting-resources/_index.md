


+++
title = "Cleanup"
weight = 17
chapter = false
pre = "<b>7. </b>"
+++










在结束Workshop之前，请确保删除不再需要的资源。以下是在此Workshop中创建的资源列表：

-   示例Web应用程序。通过CloudFormation创建，请删除对应Stack
-   Web ACL。删除Web ACL请参考 [Deleting a Web ACL](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-deleting.html)
-   Kinesis Data Firehose
-   S3桶（Kinesis Firehose目的地）

请将上面的资源一一删除



#### 删除CloudFormation

![image-20210509085239978](https://pingfan.s3.amazonaws.com/pic3/rxsmo.png)

#### 删除Web ACL

 ![image-20210509085323802](https://pingfan.s3.amazonaws.com/pic3/go0qc.png)

先`Disassociate`再删除：

![image-20210509085417022](https://pingfan.s3.amazonaws.com/pic3/lpgn1.png)



#### 删除Kinesis Firehose

![image-20210509085553284](https://pingfan.s3.amazonaws.com/pic3/5d9la.png)

#### 删除S3桶

先Empty再Delete：

![image-20210509085827116](https://pingfan.s3.amazonaws.com/pic3/s0v5u.png)

![image-20210509085857002](https://pingfan.s3.amazonaws.com/pic3/a5exp.png)
