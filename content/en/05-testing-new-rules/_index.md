

+++
title = "测试新规则"
weight = 14
chapter = false
pre = "<b>5. </b>"
+++







**在部署新的规则之前，对其进行测试至关重要**，这是为了确保不会意外拦截有效的请求。

在前面两节，对请求采取的操作有两种：*Block*或*Allow*。 其实还有第三个动作*Count*， *Count*可以**统计匹配规则条件的请求数**。

计数(Count)是一个“非终止(non-terminating)”动作, 当请求与规则相匹配时，Web ACL将继续处理其余规则。

托管规则也可以使用*Count*来测试。

<br>

### 查看Count

当行为是Count的规则被匹配时，该事件将发送到CloudWatch。 可以在Cloudwatch控制台查看规则的Count，默认显示WAF指标时使用*Average*，在某些情况下将其更改为*Sum*会很有用。

<br>

### 测试Count

>   假设你为WAF自定义了一套新规则。 在部署它之前必须先对其进行测试，确保不会意外拦截有效的请求。

我们的目标是在控制台创建一条规则，当使用“username”参数时不会阻止请求：

![image-20210509082132032](https://pingfan.s3.amazonaws.com/pic3/qdbru.png)



规则名称为`count-von-count`, 这条规则检查查询参数是否包含**username** :

![image-20210509082508112](https://pingfan.s3.amazonaws.com/pic3/8nyff.png)

Action部分选择*Count*:

![image-20210509082645670](https://pingfan.s3.amazonaws.com/pic3/pcg58.png)

然后保存，并进行下面的测试。

<br>



### 测试

在终端里执行如下命令：

```bash
curl "$JUICESHOP_URL?username=admin"
```

这条请求不会会阻截，而是被*计数*，可以在`Cloudwatch metrics`界面查看统计：

![image-20210509083645964](https://pingfan.s3.amazonaws.com/pic3/b842k.png)















