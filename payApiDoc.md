## 1. 协议规则
传输方式：采用HTTP传输(生产环境建议HTTPS)

提交方式：采用POST方式提交

内容类型：application/x-www-form-urlencoded

字符编码：UTF-8

签名算法：MD5

## 1.1 参数规范
交易金额：默认为人民币交易，单位为分，参数值不能带小数。
## 1.2 安全规范
### 签名算法
`签名生成的通用步骤如下`

***第一步：*** 设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&amp;key2=value2…）拼接成字符串stringA。
特别注意以下重要规则：

◆ 参数名ASCII码从小到大排序（字典序）；

◆ 如果参数的值为空不参与签名；

◆ 参数名区分大小写；

◆ 验证调用返回或支付中心主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。

◆ 支付中心接口可能增加字段，验证签名时必须支持增加的扩展字段


***第二步：*** 在stringA最后拼接上key`[即 StringA + "&amp;key=" + 私钥 ]` 得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign值signValue。

如请求支付系统参数如下（`以下只是举例，真实签名时除了sign字段都参与加签`）：

Map signMap = new HashMap();
signMap.put("userId", "test01");
signMap.put("type", "wechat");
signMap.put("money", Double.valueOf(2));
signMap.put("remark", "");
signMap.put("outTradeNo", "P12312321123");

`待签名值`：money=2.0&amp;outTradeNo=P12312321123&amp;type=wechat&amp;userId=test01&amp;key=EWEFD123RGSRETYDFNGFGFGSHDFGH
`签名结果`：5E0AA05DD4BB4FE5AB65608123EBA591
`最终请求支付系统参数`：money=2.0&amp;outTradeNo=P12312321123&amp;type=wechat&amp;userId=test01&amp;sign=5E0AA05DD4BB4FE5AB65608123EBA591

&gt; 请叫上游开户的人给你查看私钥key。

## 2. 统一下单
&gt; 接口描述

业务通过统一下单接口可以发起任意三方支付渠道的支付订单。业务系统不必关心该如何调用三方支付，统一下单接口会根据业务系统选择的支付渠道ID，选择对应支付渠道的支付产品，发起下单请求，然后响应给业务系统支付请求所需参数。

&gt; 接口链接

URL地址：{payUrl}/api/create_order

&gt; 请求参数

字段名 | 变量名 | 必填 | 类型 | 示例值 | 描述
------- | -------| -------| -------| ------| -------
商户ID | merchantId | 是 | String(30) | 20001222 | 支付中心分配的商户号
支付产品ID | channelId | 是 | String(24) | 8001 |
商户订单号 | mchOrderNo | 是 | String(30) | 20160427210604000490 | 商户订单号,不能超过24位
支付金额 | amount | 是 | int | 100 | 支付金额,单位分
异步回调地址 | notifyUrl | 是 | String(128) | http://shop.xx.com/notify.htm | 支付结果异步回调URL
同步请求地址 | returnUrl | 否 | String(128) | http://shop.xx.com/return.htm | 支付结果同步请求URL
商品主题 | subject | 是 | String(64) | 测试商品1 | 商品主题
请求时间 | timestamp | 是 | String(30) | 20190723141000 | 请求接口时间， yyyyMMddHHmmss格式
签名 | sign | 是 | String(32) | C380BEC2BFD727A4B6845133519F3AD6 | 大写签名值，详见签名算法

&gt; 响应结果

字段名 | 变量名    | 必填 | 类型 | 示例值  | 描述
------- |--------| -------| -------|------| -------
返回状态码 | code   | 是 | Integer | 200  | 200-处理成功，其他-处理有误，详见错误码
返回信息 | msg | 否 | String(128) | 签名失败 | 具体错误原因，例如：签名失败、参数格式校验错误

##### 以下字段在code=200 时有返回

字段名 | 变量名 | 必填 | 类型 | 示例值 | 描述
------- | -------|----| -------| -------| -------
状态码 | code | 是  | Integer | 200 | 返回状态码
状态信息 | msg | 是  | String | 操作成功 | 返回状态信息
返回数据 | data | 是  | Object | | 返回数据

##### data 返回示例

字段名 | 变量名 | 必填 | 类型 | 示例值 | 描述
------- | -------| -------| -------| -------| -------
支付订单号 | payOrderId | 是 | String(30) | P01201907231119090520000 | 返回支付系统订单号
商户订单号 | mchOrderNo | 是 | String(28) | M551165551156 | 返回商户订单号
支付表单地址 | payUrl | 是 | text | http://www.baidu.com/pay/api/P01202504211956578810027?auth_key=5D5D3563C40D325F14CE08DC56D7BD20 |一般为支付链接内容

[//]: # (签名信息 | sign | 是 | String&#40;128&#41; | CCD9083A6DAD9A2DA9F668C3D4517A84 | 签名信息)

&gt; 响应数据示例

URL方式响应数据：
```json
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "mchOrderNo": "D531266666666333",
        "payOrderId": "P01202507151805591290002",
        "payUrl": "http://10696wjxa4860.vicp.fun/api/goPayProductView/P01202507151805591290002?auth_key=3506F224F68193D79FA4953EE1B03D35"
    }
}
```

[//]: # (APP支付时响应数据：)

[//]: # ()
[//]: # (```json)

[//]: # ({)

[//]: # ("payMethod": "alipayApp",)

[//]: # ("payParams": {)

[//]: # ("appStr": "trade_no=2020111704200341411064884895&amp;biz_sub_type=peerpay_trade&amp;presessionid=&amp;app=tb&amp;channel=&amp;type2=gulupay&amp;bizcontext={\"biz_type\":\"share_pp_pay\",\"type\":\"qogirpay\"}")

[//]: # (},)

[//]: # ("retCode": "0",)

[//]: # ("sign": "6734227C78A110D7F9BF94BB9A217D3E")

[//]: # (})

[//]: # (```)

## 3. 查询支付订单
&gt; 接口描述

业务系统通过查询支付订单接口获取最新的支付订单状态，并根据状态结果进一步处理业务逻辑。

&gt; 接口链接

URL地址：{payUrl}/api/query_order

&gt; 请求参数

字段名 | 变量名 | 必填 | 类型 | 示例值 | 描述
------- | -------| -------| -------| -------| -------
商户ID | merchantId | 是 | String(30) | 1000000010 | 支付中心分配的商户号
支付订单号 | payOrderId | 是 | String(30) | P20160427210604000490 | 支付中心生成的订单号，与mchOrderNo二者传一即可
商户订单号 | mchOrderNo | 是 | String(30) | 20160427210604000490 | 商户生成的订单号，与payOrderId二者传一即可
是否执行回调 | executeNotify | 否 | Boolean | true | 是否执行回调，如果为true，则支付中心会再次向商户发起一次回调，如果为false则不发起
请求时间 | timestamp | 是 | String(30) | 20190723141000 | 请求接口时间， yyyyMMddHHmmss格式
签名 | sign | 是 | String(32) | C380BEC2BFD727A4B6845133519F3AD6 | 签名值，详见签名算法

&gt; 响应结果

字段名 | 变量名 | 必填 | 类型 | 示例值  | 描述
------- | -------| -------| -------|------| -------
返回状态码 | code | 是 | String(16) | 200  | 200-处理成功，其他-处理有误，详见错误码
返回信息 | msg | 否 | String(128) | 操作成功 | 具体错误原因，例如：签名失败、参数格式校验错误
返回数据 | data | 是  | Object | | 返回数据

##### 以下字段在code=200 时有返回
字段名 | 变量名 | 必填 | 类型 | 示例值 | 描述
------- | -------| -------| -------| -------| -------
商户ID | merchantId | 是 | String(30) | 20001222 | 支付中心分配的商户号
支付产品ID | channelId | 是 | String(24) | 8001 |
支付订单号 | payOrderId | 是 | String(30) | P01201907231119090520000 | 返回支付系统订单号
商户订单号 | mchOrderNo | 是 | String(30) | 20160427210604000490 | 商户生成的订单号
支付金额 | amount | 是 | int | 100 | 下单时传的支付金额,单位分
订单状态 | status | 是 | String(3) | 2 | 订单状态: -2:订单已关闭,0-订单生成,1-支付中,2-支付成功,3-业务处理完成,4-已退款 5 通知中 6通知失败（2，3，5，6都表示支付成功,3表示支付平台回调商户且返回成功后的状态）
渠道订单号 | channelOrderNo | 否 | String wx20170910211043fb206e92260071822007 | 对应的第三方支付订单号 |
支付成功时间 | paySuccTime | 否 | Long | 1505049094262 | 支付成功时间，精确到毫秒
签名 | sign | 是 | String(32) | C380BEC2BFD727A4B6845133519F3AD6 | 签名值，详见签名算法

##### data 返回示例
```json
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "merchantId": 1003,
        "mchOrderNo": "SFQR23fd8DFEPGFFFDD",
        "payOrderId": "P01202507311314543300009",
        "channelId": 8031,
        "amount": 10000,
        "status": "3",
        "channelOrderNo": "2025073122001486410506619874",
        "paySuccTime": 1753938929000,
        "sign": "40A32551E6923A795B98088964819853"
    }
}
```

## 4. 支付结果通知
&gt; 接口描述

当订单支付成功时，支付中心会向商户的notifyUrl地址发起回调，通知订单状态。

&gt; 接口链接

该链接是通过统一下单接口提交的参数notifyUrl设置，如果无法访问链接，业务系统将无法接收到支付中心的通知。

&gt; 通知参数

字段名 | 变量名 | 必填 | 类型 | 示例值 | 描述
------- | -------| -------| -------| -------| -------
支付订单号 | payOrderId | 是 | String(30) | P20160427210604000490 | 支付中心生成的订单号
商户ID | merchantId | 是 | String(30) | 20001222 | 支付中心分配的商户号
支付产品ID | channelId | 是 | String(24) | 8001 |
商户订单号 | mchOrderNo | 是 | String(30) | 20160427210604000490 | 商户生成的订单号
支付金额 | amount | 是 | int | 100 | 请求支付下单时金额,单位分
付款金额 | income | 是 | int | 100 | 用户实际付款的金额,单位分
状态 | status | 是 | int | 1 | 订单状态: -2:订单已关闭,0-订单生成,1-支付中,2-支付成功,3-业务处理完成,4-已退款 5 通知中 6通知失败（2，3，5，6都表示支付成功,3表示支付平台回调商户且返回成功后的状态）
渠道订单号 | channelOrderNo | 否 | String(64) | wx2016081611532915ae15beab0167893571 | 三方支付渠道订单号
支付成功时间 | paySuccTime | 是 | long | | 精确到毫秒
通知请求时间 | timestamp | 是 | String(30) | 20190723141000 | 通知请求时间，yyyyMMddHHmmss格式
签名 | sign | 是 | String(32) | C380BEC2BFD727A4B6845133519F3AD6 | 签名值，详见签名算法

&gt; 返回结果

业务系统处理后同步返回给支付中心，返回字符串 SUCCESS 则表示成功，返回非SUCCESS则表示处理失败，支付中心会再次通知业务系统。（通知频率为60/120/180/240/300,单位：秒）

`注意：返回的字符串必须是大写，且前后不能有空格。`
