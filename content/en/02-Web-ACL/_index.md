+++
title = "Web ACL与托管规则"
weight = 11
chapter = false
pre = "<b>2. </b>"
+++











### Web ACLs

 [Web ACL (Web Access Control List)](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl.html)是AWS WAF部署时的核心模块， 它会对收到的每个请求进行规则评估。Web ACL要绑定在其他AWS服务(CloudFront、ALB、API Gateway)上。

本Workshop使用最新版的AWS WAF，不会使用WAF Classic。

<br>

### 托管规则(Managed Rules)

入门AWS WAF最快的方法是部署[AWS托管规则](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-list)。

[托管规则组](https://docs.aws.amazon.com/waf/latest/developerguide/waf-managed-rule-groups.html)(**Managed Rule Group**)是由AWS或第三方在 AWS Marketplace创建和维护的一组规则。 这些规则提供了针对常见攻击的防护，或对于特定应用的防护。

每个托管规则组都可以防御一些攻击类型，例如SQL注入或命令行形式的攻击。

AWS提供了一系列托管规则组，例如*Amazon IP Reputation list*、 *Known Bad Inputs*、 *Core rule set*，当然还有其他的规则组：

![](https://introduction-to-waf.workshop.aws/images/managed-rules.png)



### 创建Web ACL

>现在假设你是**Juice Shop** 的唯一开发者。该网站是一个由SQL数据库支持的简单Web应用程序， 由于某些原因，一群黑客已经盯上了你的网站！
>
>幸运的是，你最近参加了AWS WAF Workshop，于是决定部署自己的WAF以保护站点。

在WAF控制台中创建Web ACL。

![image-20210506143822739](https://pingfan.s3.amazonaws.com/pic3/2npu9.png)

命名为“ waf-workshop-juice-shop”，*资源类型*是*CloudFront distributions*：

![image-20210506144840387](https://pingfan.s3.amazonaws.com/pic3/wmxr7.png)



将Web ACL与CloudFront绑定:



![image-20210506144932743](https://pingfan.s3.amazonaws.com/pic3/weelv.png)

![image-20210506144949521](https://pingfan.s3.amazonaws.com/pic3/q0fhg.png)

选中刚才添加的CloudFront，并进入下一步：

![image-20210506145111722](https://pingfan.s3.amazonaws.com/pic3/86vd4.png)



现在进入添加**规则组**的步骤，这里我们使用AWS托管的规则组：

![image-20210506145455111](https://pingfan.s3.amazonaws.com/pic3/6ld9p.png)

从**AWS managed rule groups**选中 *Core Rule Set* 和 *SQL Database* ， *Core Rule Set* 涵盖Web应用常见的各种漏洞，*SQL database* 提供防止SQL注入的规则：

![image-20210506145633086](https://pingfan.s3.amazonaws.com/pic3/15odx.png)

接下来一路点击**Next**，直到**Create web ACL**：

![image-20210506185212783](https://pingfan.s3.amazonaws.com/pic3/e2l1i.png)



#### 测试

使用以下命令测试部署的的规则：

```bash
export JUICESHOP_URL=<Your Juice Shop URL>
# 测试XSS攻击，下面的请求将会被拦截
curl -X POST  $JUICESHOP_URL -F "user='<script><alert>Hello></alert></script>'"
# 测试SQL注入，请求同样也会被拦截
curl -X POST $JUICESHOP_URL -F "user='AND 1=1;"
```

如果请求被阻止，将在返回结果中含有以下字段：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
    <title>ERROR: The request could not be satisfied</title>
  </head>
  <body>
    <h1>403 ERROR</h1>
    <!-- Omitted -->
  </body>
</html>
```

如果收到以下返回结果，那么请求就没有被WAF阻拦：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>OWASP Juice Shop</title>
    // Omitted for brevity
  </head>
</html>
```

也可以在浏览器中构造请求进行测试，如果页面显示**The request could not be satisfied**，则请求被WAF拦截：

![image-20210506190319406](https://pingfan.s3.amazonaws.com/pic3/12rqi.png)



### 结论

随着WAF部署完成， web应用可以免受常见攻击。 

本节介绍了WAF的AWS托管规则，托管规则组可以快速保护应用程序免受各种常见攻击的侵害，并且可以从AWS和AWS Marketplace获取其他规则。









