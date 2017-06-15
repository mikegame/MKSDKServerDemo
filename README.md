# 米壳SDK服务端说明文档




##	接口说明
###	1、用户会话验证
* 1) **请求地址：** https://api.open.mikegame.cn/open/verifyAccessToken
* 2) **调用方式：** HTTPS Post
* 3) **接口描述：**
验证 accessToken 是否为有效的登录用户会话，若有效则返回其 userId。**“游戏客户端”**通过“SDK 客户端”获取到accessToken，传到**“游戏服务器”**，**“游戏服务器”**到**“SDK 服务器”**验证用户会话accessToken的有效性，获取用户的userId，供游戏使用。</br>**注意：进行接口调用前请确认accessToken是否具备值，如accessToken值为空时请勿调用此接口。**
* 4) **请求方：** 游戏服务器
* 5) **响应方：** SDK 服务器
* 6) **请求内容：** 

<table>
    <thead>
        <tr>
            <th>字段名称</th>
            <th>字段说明</th>
            <th>类型</th>
            <th>必填</th>
            <th>备注</th> 
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>gameId</td>
            <td>米壳后台申请(游戏Id)</td>
            <td>string</td>
            <td>是</td>
            <td></td>
        </tr>
		<tr>
            <td>subGameId</td>
            <td>米壳后台申请(游戏子包Id)</td>
            <td>string</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>accessToken</td>
            <td>accessToken</td>
            <td>string</td>
            <td>是</td>
            <td>accessToken令牌</td>
        </tr>
        <tr>
            <td>sign</td>
            <td>签名参数</td>
            <td>string</td>
            <td>是</td>
            <td>所有参数key(不包括sign)按照A-Z字段升序拼接key之后md5加密</td>
        </tr>
    </tbody>
</table>

请求例子：   
```php
Public function notify(){
    $params['gameId'] = 1;
    $params['subGameId']=1;
    $params['accessToken'] = 'tokendemo';
    ksort($params);
    $key = 'key';//由米壳颁发
    
    $str = str_replace("&","",http_build_query($params).$key);
    $params['sign'] = md5($str);
    
    $res = curl_https（$url,$params） ;//发起https请求
    //处理$res数据
}
```

返回内容（ json 格式）：

```
{
    "code": 0,
    "msg": "ok",
    "data": {
        "userId": 8,
        "userName": "zxwzxw"
    }
}
```



###	2、支付结果异步通知
* 1）**请求地址：** 由游戏cp方提供
* 2）**调用方式：** HTTP Post
* 3）**接口描述：**
**“游戏客户端”**购买成功后，**“SDK 服务器”**通过该接口通知**”游戏服务器“**。支付成功一定会及时回调通知，支付失败可能不会及时通知（如支付超时等无法及时通知），如果需要知道订单状态，使用支付结果查询结构。
支付通知有失败重试机制，正式环境通知时间间隔为15,15,30,180,1800,1800,1800,1800,3600 单位为秒<br>
使用支付key签名；
* 4）**请求方：** SDK 服务器  
* 5）**响应方：** 游戏服务器 
* 6）**请求内容：** JSON 格式
<table>
    <thead>
        <tr>
            <th>字段名称</th>
            <th>字段说明</th>
            <th>类型</th>
            <th>必填</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>cpOrderId</td>
            <td>CP方订单</td>
            <td>String</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>userId</td>
            <td>用户Id</td>
            <td>int</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>orderId</td>
            <td>米壳平台订单</td>
            <td>String</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>gameId</td>
            <td>游戏Id</td>
            <td>int</td>
            <td>是</td>
            <td></td>
        </tr>
	<tr>
            <td>subGameId</td>
            <td>子包Id</td>
            <td>int</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>platform</td>
            <td>平台类型</td>
            <td>string</td>
            <td>是</td>
            <td>1，ios；2，android</td>
        </tr>
        <tr>
            <td>totalFee</td>
            <td>支付金额</td>
            <td>int</td>
            <td>是</td>
            <td>单位是：分</td>
        </tr>
        <tr>
            <td>orderStatus</td>
            <td>订单状态</td>
            <td>int</td>
            <td>是</td>
            <td>1：已支付，0：未支付</td>
        </tr>
        <tr>
            <td>sign</td>
            <td>所有参数key(不包括sign)按照A-Z字段升序拼接key之后,md5加密</td>
            <td>string</td>
            <td>是</td>
            <td></td>
        </tr>
    </tbody>
</table>
	
CP方处理例子：
```php
Public function notify(){
    $params['cpOrderId'] = $_POST['cpOrderId'];
    $params['XSOrderId'] =  $_POST['XSOrderNo'];
    $params['gameId'] =  $_POST['gameId'];
    $params['subGameId'] =  $_POST['subGameId'];
    $params['serverId'] =  $_POST['serverId'];
    $params['amount'] =  $_POST['amount'];
    $params['orderStatus'] =  $_POST['orderStatus'];
    $params['userId'] = $_POST['userId'];
    $sign = $_POST['sign'];

    ksort($params);
    $key = 'key';//由米壳颁发使用支付key
    $sign = md5(urldecode(str_replace("&","",http_build_query($params).$key)));
    if($sign==$_POST['sign']){
        echo 'success';
        //CP方数据处理
    }else{
    	echo 'failure';
    }
}
```

返回内容（string）：
<table>
    <thead>
        <tr>
            <th>输出值</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>success</td>
            <td>成功,表示游戏服务器成功接收了该次充值结果通知</td>
        </tr>
        <tr>
            <td>failure</td>
            <td>失败,表示游戏服务器无法接收或识别该次充值结果通知,如:签名检验不正确、游戏服务器接收失败</td>
        </tr>
    </tbody>
</table>
	
	
	

许可证
==============
HGSDK 使用 MIT 许可证，详情见 LICENSE 文件。

