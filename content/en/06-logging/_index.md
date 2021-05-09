


+++
title = "保存日志记录"
weight = 15
chapter = false
pre = "<b>6. </b>"
+++









WAF使用Kinesis Firehose来接收日志，所以日志可以传递到任何Kinesis Firehose的目标，如Amazon S3，Amazon Redshift、Amazon Elastic Search。 要在Web ACL中开启日志记录，必须首先创建Kinesis Data Firehose。

以下是WAF日志示例， 日志中包含对请求采取的操作：

```json
{
  "timestamp": 1576280412771,
  "formatVersion": 1,
  "webaclId": "arn:aws:wafv2:ap-southeast-2:EXAMPLE12345:regional/webacl/STMTest/1EXAMPLE-2ARN-3ARN-4ARN-123456EXAMPLE",
  "terminatingRuleId": "STMTest_SQLi_XSS",
  "terminatingRuleType": "REGULAR",
  "action": "BLOCK",
  "terminatingRuleMatchDetails": [
    {
      "conditionType": "SQL_INJECTION",
      "location": "HEADER",
      "matchedData": ["10", "AND", "1"]
    }
  ],
  "httpSourceName": "-",
  "httpSourceId": "-",
  "ruleGroupList": [],
  "rateBasedRuleList": [],
  "nonTerminatingMatchingRules": [],
  "httpRequest": {
    "clientIp": "1.1.1.1",
    "country": "AU",
    "headers": [
      {
        "name": "Host",
        "value": "localhost:1989"
      },
      {
        "name": "User-Agent",
        "value": "curl/7.61.1"
      },
      {
        "name": "Accept",
        "value": "*/*"
      },
      {
        "name": "x-stm-test",
        "value": "10 AND 1=1"
      }
    ],
    "uri": "/foo",
    "args": "",
    "httpVersion": "HTTP/1.1",
    "httpMethod": "GET",
    "requestId": "rid"
  }
}
```

### 日志分析与可视化

日志分析对于确保WAF规则的有效性至关重要。 可以将日志导入到ElasticSearch并进行查询，Kibana可以与ElasticSearch一起使用，可视化WAF日志。S3 Select可用于对S3中的单个WAF日志文件执行SQL查询。

例如，以下S3 Select查询统计了日志文件中被阻止的请求数：

```sql
SELECT *
FROM S3Object s
WHERE  s."action" = 'BLOCK'

/*
Result

{
    "count": 18
}
*/
```





### 为WAF开启日志

>    你的web应用有了一组规则，但现在越来越难以推断具体是哪个规则负责阻止了请求。通过分析日志可以解决这个问题，为此需要启用日志记录。
>
>   应用日志的请求头中携带了Cookie，你不希望将其存储在日志，否则会违反相关安全数据法规。 

<br>

创建S3桶，用于接收Kinesis Firehose的数据。桶名可命名为`aws-waf-logs-workshop-xxxx`：

![image-20210507000237637](https://pingfan.s3.amazonaws.com/pic3/ihuac.png)

在`us-east-1`区域(必须是美东1区)创建Kinesis Data Firehose，名称必需以`aws-waf-logs-`开头，并选择上一步创建的S3做为目标存储：

![image-20210507000451843](https://pingfan.s3.amazonaws.com/pic3/t7qzh.png)

![image-20210507000538535](https://pingfan.s3.amazonaws.com/pic3/dv69j.png)



在Web ACL界面选择*Logging and metrics*，并选择*Enable logging*:

![image-20210506235703416](https://pingfan.s3.amazonaws.com/pic3/dk6we.png)

选择上一步创建的Kinesis Data Firehose，在 *Redacted fields*部分, 钩中 *Header*，然后添加 `Cookie`:

![image-20210507001016478](https://pingfan.s3.amazonaws.com/pic3/l9yfd.png)

点击保存，创建完成。

运行几条`curl`命令或刷新网页以制造一些流量：

```bash 
curl "$JUICESHOP_URL?username=admin"
curl "${JUICESHOP_URL}?milkshake=banana&favourite-topping=sauce"
curl -H "x-milkshake: chocolate" "${JUICESHOP_URL}"
```

等过一段时间后，下载S3桶中的日志。在日志文件中查看是否有`Cookie`字段：

![image-20210507001605506](https://pingfan.s3.amazonaws.com/pic3/56ou4.png)

### 结论

WAF可以将请求日志存储在任何Kinesis Data Firehose目标中。 日志提供了请求的详细信息，还提供了请求所触发的操作和规则。 这些信息对于数据分析非常宝贵。 

另外可以避免记录一些敏感信息。