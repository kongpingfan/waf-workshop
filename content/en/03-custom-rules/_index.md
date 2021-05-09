



+++
title = "自定义规则"
weight = 11

pre = "<b>3. </b>"
+++








### 自定义规则(Custom Rules)

WAF允许用户创建自己的规则(参考： https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html )来处理web请求。

<br>

创建自定义规则的最简单方法是在WAF控制台中使用编辑器：

![Add a custom rule](https://pingfan.s3.amazonaws.com/pic3/51q1g.png)

创建的自定义规则可以检测HTTP请求的**源IP、请求头、请求体、URL、请求参数**，根据检测的结果决定是阻拦还是放行请求。

<br>

### Request Sampling

WAF允许查看已处理的请求样本， 从Web ACL页面查看：

![image-20210506192121770](https://pingfan.s3.amazonaws.com/pic3/dwjyf.png)

通过查看收到了哪些请求以及如何处理它们，对于快速调试很有用。

也可以通过日志记录Web ACL收到的所有请求(稍后将介绍)。

<br>

### WCU(Web ACL Capacity Unit)

上一节创建两个托管规则时，你可能已经注意到*WCU*这个单位。 WAF使用WCU计算规则的运行成本，与更复杂的规则相比，简单的规则使用的WCU更少。

一个Web ACL的最大WCU为1500，可以通过联系AWS Support来增加此值。[WCU文档](https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works.html#aws-waf-capacity-units)中有详细介绍。

<br>

### 创建自定义规则

背景：

>   上一节中我们成功阻拦了常见的恶意请求。 但现在越来越多的黑客正在攻击该网站，并且请求构造更加具有针对性，此时你决定使用WAF Web ACL的自定义规则阻止这些攻击。
>
>   所有的攻击似乎都包含一个奇怪的请求头——`X-TomatoAttack`，于是拦截含有该请求头的请求可以阻止攻击。

在Web ACL上创建一个规则，该规则判断请求头`X-TomatoAttack`来阻止请求：

![image-20210506193049801](https://pingfan.s3.amazonaws.com/pic3/ovje0.png)



规则配置如下：

![image-20210506193220228](https://pingfan.s3.amazonaws.com/pic3/2wnfl.png)

然后点击**Add Rule**并保存：

![image-20210506193317649](https://pingfan.s3.amazonaws.com/pic3/nxthh.png)



### 测试

该用例将向Web应用发送测试请求。 如果WAF规则有效则应阻止恶意请求，并将收到如下所示的*403*回复：

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

在CLI中运行如下命令：

```bash
# Set the JUICESHOP_URL variable if not already done
# JUICESHOP_URL=<Your Juice Shop URL>

# 下面的请求将被阻止
curl -H "X-TomatoAttack: Red" "${JUICESHOP_URL}"
# 下面的请求将被阻止
curl -H "X-TomatoAttack: Green" "${JUICESHOP_URL}"
```

检查**Sample Requests**页面，可以看到上面被阻止的两条请求：

![image-20210506193751460](https://pingfan.s3.amazonaws.com/pic3/0wbc5.png)

### 结论

*   自定义规则允许用户实现自己的逻辑来处理WAF中的请求。 自定义规则可以检查请求格式，然后在规则语句为true时阻止或允许请求。

*   每个Web ACL都有一个最大容量单位（WCU），默认是1500，但可以根据需要增加。 Web ACL中的每个规则和规则组都会有特定的容量单位。

