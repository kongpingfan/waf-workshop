


+++
title = "使用JSON自定义复杂规则"
weight = 12

pre = "<b>4. </b>"
+++









上一节创建了一个简单的WAF规则，该规则可以检查请求的一部分(如请求头)。 那么如何创建规则，以检测请求的多个部分(如同时检查请求头、请求体、URL参数)呢？

### JSON格式的规则

所有WAF规则都可以用JSON对象描述。对于复杂的规则，直接编辑JSON格式更有效。

可以使用AWS CLI的`get-rule-group`命令或控制台编辑器查看json格式的规则，编辑完json后使用`update-rule-group`进行更新。

如果不确定语法，可以先在控制台可视化编辑器中创建一个简单的示例，然后切换到JSON编辑器以查看等效的JSON

例如上一节中规则表示成json格式为：

![image-20210506230759282](https://pingfan.s3.amazonaws.com/pic3/f7qko.png)









### 使用JSON创建规则

现在黑客又来攻击你的网站，恶意请求如下：

1.  请求头中包含 `x-milkshake: chocolate`
2.  查询参数中包含 `milkshake=banana`

如果有以上任意一种情况，需要对请求进行拦截。

<br>

再添加一条自定义规则：

![image-20210506231741468](https://pingfan.s3.amazonaws.com/pic3/21921.png)

选择使用JSON编辑器：

![image-20210506231758489](https://pingfan.s3.amazonaws.com/pic3/hw0vt.png)

将以下内容粘贴进去：

```json
{
  "Name": "complex-rule-challenge",
  "Priority": 0,
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "complex-rule-challenge"
  },
  "Statement": {
    "OrStatement": {
      "Statements": [
        {
          "ByteMatchStatement": {
            "FieldToMatch": {
              "SingleHeader": {
                "Name": "x-milkshake"
              }
            },
            "PositionalConstraint": "EXACTLY",
            "SearchString": "chocolate",
            "TextTransformations": [
              {
                "Type": "NONE",
                "Priority": 0
              }
            ]
          }
        },
        {
          "ByteMatchStatement": {
            "FieldToMatch": {
              "SingleQueryArgument": {
                "Name": "milkshake"
              }
            },
            "PositionalConstraint": "EXACTLY",
            "SearchString": "banana",
            "TextTransformations": [
              {
                "Type": "NONE",
                "Priority": 0
              }
            ]
          }
        }
      ]
    }
  }
}
```

保存规则。



### 测试

使用*curl*创建以下请求来测试新规则：

```bash
# Set the JUICESHOP_URL if not already done
JUICESHOP_URL=<Your Juice Shop URL>

# 以下请求被拦截
curl -H "x-milkshake: chocolate" "${JUICESHOP_URL}"
curl  "${JUICESHOP_URL}?milkshake=banana"

# 以下请求被允许
curl -H "x-milkshake: chocolatee" "${JUICESHOP_URL}"
curl  "${JUICESHOP_URL}?milkshake=bananaa"
```

被拦截的请求会返回以下结果：

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

### 结论

在本节中，了解了WAF规则的JSON格式。 可以使用*And*，*Or*和*Not*运算符在规则中定义复杂逻辑.