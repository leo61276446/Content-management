# 内容管理和模仿评测使用说明

### 摘要
> 本文主要介绍云知声口语评测公有云内容管理系统的基本架构以及调用接口说明，为客户使用预置内容评测提供指导，本文面向用户为项目经理以及客户的开发人员。
**预置内容评测方式主要适用于模仿评测和长文本评测。**
>在用户使用评测之前，将模仿评测的标准音以及文本，或者长文本的文本上传到内容管理的系统，将获取的ID按格式传入我们SDK的文本接口（text参数），即可进行模仿评测或者长文本内容的评测。

### 模仿评测

* **1)上传标准音和文本**

> 目前内容管理系统支持http协议的POST方法上传文本和音频，通过curl上传示例如下：

> curl -X post -H "appkey:$appkey" -F text="$text"  -F voice=@$filename -v **cm.edu.hivoice.cn:20010/{audioformat}**

> 其中，$appkey为客户appkey，需要向云知声项目经理申请；

> $text为跟读题型对应的文本内容

> $filename 为跟读题型的标准音频

> audioformat支持的格式：wav(16k16bit), mp3, opus, amrnb, silk

> **参数顺序必须是先text再voice**



* **2)获取contentid**

> 在通过POST方法上传内容和音频成功后，内容管理系统在返回的http协议body中包含该题的内容ID：
```
*   Trying 192.168.6.128...* Connected to 192.168.6.128 (192.168.6.128) port 4990 (#0)
> POST /pcm HTTP/1.1
> Host: 192.168.6.128:4990
> User-Agent: curl/7.44.0
> Accept: */*
> appkey:e5teo2s7t4pic7s6sj6b3re7qjdemym3cxgtmmis
> Content-Length: 93169
> Expect: 100-continue
> Content-Type: multipart/form-data; boundary=------------------------999024347792e8d0
>
 < HTTP/1.1 100 Continue
< HTTP/1.1 200 OK
< X-Contentid: 09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006
< Date: Mon, 19 Sep 2016 08:30:30 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8

上面X-Contentid字段表示该题的内容ID：
09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006
```

* **3)拼接评测文本**

> 模仿评测的评测文本格式为：
```
{"X-EngineType": "oral.gendu", "ID":"xxx"}
```
ID的值为内容管理服务器返回的内容ID，上面示例的评测文本为：
```
{"X-EngineType": "oral.gendu", "ID":"09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006"}
```
使用shell 下curl发送HTTP评测的命令是：
```
curl -X post -H "appkey:$appkey" -F text='{"X-EngineType": "oral.gendu", "ID":"09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006"}'  -F voice=@$filename -v http://edu.hivoice.cn:8085/eval/{audioformat}

 audioformat支持的格式：mp3, opus, amrnb, silk，例如用户音频是mp3，评测地址是edu.hivoice.cn:8085/eval/mp3
```
**注意：上面curl命令是shell下http评测样例，如果使用的是私有协议SDK，按照SDK接口调用评测即可，只是把评测的文本改为上面拼接的格式**

>此时，预置内容的步骤已经完成，在调用我们SDK进行评测的时候，只需在上传文本的接口上传上面内容即可进行模仿评测。

* **4)获取评测结果**

>模仿评测返回的结果，是在朗读评测返回结果的基础上增加了similarscore字段，表示用户朗读的音频与标准音频的相似度情况。朗读评测返回的Json字段含义请查找<a href=https://github.com/oraleval/FAQ-Docs/blob/master/Json%20Description.md>Json字段说明</a>

### 长文本评测

* **1)上传文本**

> 长文本方式只需要上传文本到内容管理服务器。在用户客户端评测时，只需上传带有内容ID的json字符串即可，不需要上传长文本段落，**很大程度的节约了客户端流量。**

> 和模仿评测上传方式一样，支持http的POST方法上传，但不需要上传音频文件，示例如下：
```
curl -X post -H "appkey:$appkey" -F text="$text" -H "X-Ctrl-Write:TxtOnly" -v cm.edu.hivoice.cn:20010/{audioformat}
```
其中：

> $appkey为客户appkey，不清楚的请向云知声项目经理申请；

> $text为跟读题型对应的文本内容


* **2)获取contentid**

> 在通过POST方法上传内容成功后，内容管理系统在返回的http协议body中包含该题的内容ID：
```
*   Trying 192.168.6.128...* Connected to 192.168.6.128 (192.168.6.128) port 4990 (#0)
> POST /pcm HTTP/1.1
> Host: 192.168.6.128:4990
> User-Agent: curl/7.44.0
> Accept: */*
> appkey:xxxxxxxxxxxxxxxxxxx
> Content-Length: 93169
> Expect: 100-continue
> Content-Type: multipart/form-data; boundary=------------------------999024347792e8d0
>
 < HTTP/1.1 100 Continue
< HTTP/1.1 200 OK
< X-Contentid: 09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006
< Date: Mon, 19 Sep 2016 08:30:30 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8

上面X-Contentid字段表示该题的内容ID：
09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006
```

* **3)拼接评测文本**

> 评测文本格式为：
```
{"X-EngineType": "oral.zuowen", "ID":"xxx"}
```
> ID的值为内容管理服务器返回的内容ID，上面示例的评测文本为：
```
{"X-EngineType": "oral.zuowen", "ID":"09b6009a-798b-4101-8be3-fab8414ad61c:1474273830473256006"}
```

* **4)获取评测结果**

> 通过预置内容方式评测的长文本与正常评测的结果格式一致,返回的Json字段含义请查找<a href=https://github.com/oraleval/FAQ-Docs/blob/master/Json%20Description.md>Json字段说明</a>
