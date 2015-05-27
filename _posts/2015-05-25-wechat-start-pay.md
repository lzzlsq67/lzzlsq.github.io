---
layout: post
title:  "微信实现JS-API网页内支付(Java)"
keywords: "wechat"
description: "使用Java语言实现微信JS-API网页内支付"
category: wechat
tags: wechat java
---
本例使用的是JS-API接口网页内支付。
#业务逻辑为：
1. 在商城中选择商品，下单调用`getBrandWCPayRequest`接口，发起微信支付，进入微信支付流程
2. 用户支付完成后，商城的前端会受到支付完成微信服务器传回的JavaScript值，后台通知发货，前台可直接跳转到支付成功的页面
Tips. 商户应该在微信后台返回支付成功的回调方法中进行发货通知
时序图如下：

![时序图](http://i1.tietuku.com/3e46cc306ef70ac0s.png)

#开发步骤
## 1.响应页面支付按钮事件，得到订单流水号以及签名需要的各种参数，触发函数ajaxSeqNoAndSignInfo()的执行

```javascript
// 当微信内置浏览器完成内部初始化后会触发WeixinJSBridgeReady事件
document.addEventListener('WeixinJSBridgeReady', function onBridgeReady() {
  //公众号支付
  jQuery('a#getBrandWCPayRequest').click(function(e){
	  ajaxSeqNoAndSignInfo();
  });
                  
  WeixinJSBridge.log('yo~ ready.');
  
}, false);
```
## 2.ajaxSeqNoAndSignInfo()返回成功时进入 向微信发起支付请求
```javascript
var tempData;
function ajaxSeqNoAndSignInfo(){
  $.ajax({
    type: "POST",
    url:"ajaxSeqNo",
    data:{"orderNo":$("#orderNo").val()},
    dataType:"json",
    cache:false,
    async: true,
    success:function(json){
      if(json.errorMessage){
    	  alert(json.errorMessage);
      }else{
    	  tempData = $.parseJSON(JSON.stringify(json));
    	  payRequest();
      }
    }
  });
```

### 2.1 获取订单流水号
获取订单流水号java代码,获取订单流水号以及支付信息，订单流水号，由商户生成，保证流水号在商户是唯一的，否则，微信服务器回调会不知道回调哪一个
进而发生错误。

```java
	/**
	 * 获取订单流水号及支付信息
	 * 
	 * @param request
	 * @param response
	 * @ret	urn
	 * @throws Exception 
	 */
	@RequestMapping("/ajaxSeqNo")
	public String paySignInfo(HttpServletRequest request,
			HttpServletResponse response, ModelMap model) throws Exception {
		//请求地址
		String requestUrl = PropertiesLoader.getPropertiesByName(FrontConstants.QUERY_ORDER_SEQ_NO_URL);
		//请求参数
		String reqStr = getOrderDetailRequestStr(request);
		//响应信息
		String responseStr = postRequestHandler(requestUrl,reqStr,"获取订单流水号");
		//响应实体
		String responseObjectStr = getResponseObjectStr(responseStr,model);
		if(responseObjectStr==null){
			model.put("errorMessage", "亲，未获取到流水号或您的订单不是待支付状态。");
		  this.writerResponse(response, JSONObject.fromObject(model).toString());
			return null;
		}
		ResponseOrderDetailsQueryObject responseObj = (ResponseOrderDetailsQueryObject) JSONUtil
		 .getObjectfromJacksonJson(responseObjectStr,ResponseOrderDetailsQueryObject.class);
		//流水号为空提示用户
		if(responseObj.getSeqNo()==null || "".equals(responseObj.getSeqNo())){
			model.put("errorMessage", "亲，未获取到流水号或您的订单不是待支付状态。");
		  this.writerResponse(response, JSONObject.fromObject(model).toString());
		  return null;
		}
		
		model.put("seqNo", responseObj.getSeqNo());
		model.put("body", responseObj.getProductName());//商品描述
		String productMode = PropertiesLoader.getPropertiesByName(FrontConstants.PRODUCT_MODE);
		if(FrontConstants.PRODUCT_MODE_YES.equals(productMode)){
			BigDecimal fee = new BigDecimal(responseObj.getOrderFee());
			fee = fee.multiply(new BigDecimal("100"));
			//金额以分为单位
			model.put("total_fee", fee);
		}else{
			model.put("total_fee", "1");
		}
		model.put("notify_url", PropertiesLoader.getPropertiesByName("notify_url"));
		model.put("partner", PropertiesLoader.getPropertiesByName("partnerId"));
		model.put("partnerKey", PropertiesLoader.getPropertiesByName("partnerKey"));
		model.put("appKey", PropertiesLoader.getPropertiesByName("appkey"));
		model.put("appId", PropertiesLoader.getPropertiesByName("appId"));
		String returnParam = JSONObject.fromObject(model).toString();
		logger.info("获取流水号接口返回页面参数："+returnParam);
		this.writerResponse(response, returnParam);
		return null;
	}
```

### 2.2 获得组包参数的js代码

```javascript
//下面是app进行签名的操作：

var oldTimeStamp ;//记住timestamp，避免签名时的timestamp与传入的timestamp时不一致
var oldNonceStr ; //记住nonceStr,避免签名时的nonceStr与传入的nonceStr不一致
         
function getTimeStamp()
{
    var timestamp=new Date().getTime();
    var timestampstring = timestamp.toString();//一定要转换字符串
    oldTimeStamp = timestampstring;
    return timestampstring;
}

function getNonceStr()
{
    var $chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    var maxPos = $chars.length;
    var noceStr = "";
    for (i = 0; i < 32; i++) {
        noceStr += $chars.charAt(Math.floor(Math.random() * maxPos));
    }
    oldNonceStr = noceStr;
    return noceStr;
}

function getSignType()
{
    return "SHA1";
}

function getSign()
{
    var app_id = getAppId().toString();
    var app_key = getAppKey().toString();
    var nonce_str = oldNonceStr;
    var package_string = oldPackageString;
    var time_stamp = oldTimeStamp;
    //第一步，对所有需要传入的参数加上appkey作一次key＝value字典序的排序
    var keyvaluestring = "appid="+app_id+"&appkey="+app_key+"&noncestr="+nonce_str+"&package="+package_string+"&timestamp="+time_stamp;
    sign = CryptoJS.SHA1(keyvaluestring).toString();
    return sign;
} 
```

### 2.3 组成Package的过程

```javascript
function getPackage()
{
    var banktype = "WX";
    var body = getBody();//商品名称信息，这里由测试网页填入。
    var fee_type = "1";//费用类型，这里1为默认的人民币
    var input_charset = "UTF-8";//字符集，可以使用GBK或者UTF-8
    var notify_url = getNotifyUrl();//支付成功后将通知该地址
    var out_trade_no = getSeqNo();//订单号，商户需要保证该字段对于本商户的唯一性
    var partner = getPartnerId();//测试商户号
    var spbill_create_ip = returnCitySN["cip"];
    //var spbill_create_ip = "127.0.0.1";//用户浏览器的ip，这个需要在前端获取。这里使用127.0.0.1测试值
    var total_fee = getTotalFee();//总金额。
    var partnerKey = getPartnerKey();//这个值和以上其他值不一样是：签名需要它，而最后组成的传输字符串不能含有它。这个key是需要商户好好保存的。
    
    //首先第一步：对原串进行签名，注意这里不要对任何字段进行编码。这里是将参数按照key=value进行字典排序后组成下面的字符串,在这个字符串最后拼接上key=XXXX。由于这里的字段固定，因此只需要按照这个顺序进行排序即可。
    var signString = "bank_type="+banktype+"&body="+body+"&fee_type="+fee_type+"&input_charset="+input_charset+"&notify_url="+notify_url+"&out_trade_no="+out_trade_no+"&partner="+partner+"&spbill_create_ip="+spbill_create_ip+"&total_fee="+total_fee+"&key="+partnerKey;
    
    var md5SignValue =  ("" + CryptoJS.MD5(signString)).toUpperCase();
    //然后第二步，对每个参数进行url转码，如果您的程序是用js，那么需要使用encodeURIComponent函数进行编码。
    banktype = encodeURIComponent(banktype);
    body=encodeURIComponent(body);
    fee_type=encodeURIComponent(fee_type);
    input_charset = encodeURIComponent(input_charset);
    notify_url = encodeURIComponent(notify_url);
    out_trade_no = encodeURIComponent(out_trade_no);
    partner = encodeURIComponent(partner);
    spbill_create_ip = encodeURIComponent(spbill_create_ip);
    total_fee = encodeURIComponent(total_fee);
    
    //然后进行最后一步，这里按照key＝value除了sign外进行字典序排序后组成下列的字符串,最后再串接sign=value
    var completeString = "bank_type="+banktype+"&body="+body+"&fee_type="+fee_type+"&input_charset="+input_charset+"&notify_url="+notify_url+"&out_trade_no="+out_trade_no+"&partner="+partner+"&spbill_create_ip="+spbill_create_ip+"&total_fee="+total_fee;
    completeString = completeString + "&sign="+md5SignValue;
    
    
    oldPackageString = completeString;//记住package，方便最后进行整体签名时取用
    
    return completeString;
}
```
##3.向微信服务器发起支付请求

```javascript
/**
 * 向微信发起支付请求
 */
function payRequest(){
	var params = {
        "appId" : getAppId(), //公众号名称，由商户传入
	    "timeStamp" : getTimeStamp(), //时间戳
	    "nonceStr" : getNonceStr(), //随机串
	    "package" : getPackage(),//扩展包
	    "signType" : getSignType(), //微信签名方式:1.sha1
	    "paySign" : getSign() //微信签名
	};
	WeixinJSBridge.invoke('getBrandWCPayRequest',
		params,
		function(res){
 	   WeixinJSBridge.log(res.err_msg);
 	   var params={};
 	   params.orderNo = $("#orderNo").val();
 	   params.seqNo = getSeqNo();
 	   params.totalFee = getTotalFee();
 	   params.errCode = res.err_code;
 	   params.errDesc = res.err_desc;
       if(res.err_msg == "get_brand_wcpay_request:ok") { 
    	   params.payResult = "1";
    	   payResultCallBack(params);
       }else if(res.err_msg == "get_brand_wcpay_request:fail"){
    	   params.payResult = "0";
       }else if(res.err_msg == "get_brand_wcpay_request:cancel"){
    	   params.payResult = "2";
       }else{
    	   params.payResult = "3";
    	   alert(JSON.stringify(res));
       }
       
    });
}
```

##4.支付完成之后，回调商户，商户通知发货，然后发送模板消息给买家

