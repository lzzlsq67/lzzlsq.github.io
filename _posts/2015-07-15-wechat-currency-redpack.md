---
layout: post
title:  "微信红包开发Java版"
keywords: "wechat"
description: "使用微信的红包接口，向指定用户（openId）发送红包"
category: wechat
tags: wechat java
---
本代码用于企业向微信用户个人发现金红包目前支持向指定微信用户的openid发放指定金额红包。

###配置红包信息
商户必须要有`微信支付`的权限，使用之前要在<https://pay.weixin.qq.com/> 上面配置一下红包信息，包括红包活动的名称，商户名称，以及红包的链接url，红包的图片，证书下载安装，以及向红包总额中充值等。这些内容不再赘述，
微信的官方文档上有介绍，地址如下<https://pay.weixin.qq.com/wiki/doc/api/cash_coupon.php?chapter=13_5>


微信红包接口调用流程
　◆　后台API调用：待进入联调过程时与开发进行详细沟通；
　◆　告知服务器：告知服务器接收微信红包的用户openID，告知服务器该用户获得的金额；
　◆　从商务号扣款：服务器获取信息后从对应的商务号扣取对应的金额；
　◆　调用失败：因不符合发送规则，商务号余额不足等原因造成调用失败，反馈至调用方；
　◆　发送成功：以微信红包公众账号发送对应红包至对应用户；

##开发步骤
###1.根据微信红包接口请求参数创建请求对象

```java
package com.sz.kcygl.kernel.weixin.entity.gift;
/**
 * 微信发送红包请求参数包
 * @author LittleXuan
 *
 */
public class SendRedPackPo {
	private String nonce_str;
	private String sign;
	private String mch_billno;
	private String mch_id;
	private String sub_mch_id;
	private String wxappid;
	private String nick_name;
	private String send_name;
	private String re_openid;
	private String total_amount;
	private String min_value;
	private String max_value;
	private String total_num;
	private String wishing;
	private String client_ip;
	private String act_name;
	private String remark;
	private String logo_imgurl;
	private String share_content;
	private String share_url;
	private String share_imgurl;

	public String getNonce_str() {
		return nonce_str;
	}

	public void setNonce_str(String nonceStr) {
		nonce_str = nonceStr;
	}

	public String getSign() {
		return sign;
	}

	public void setSign(String sign) {
		this.sign = sign;
	}

	public String getMch_billno() {
		return mch_billno;
	}

	public void setMch_billno(String mchBillno) {
		mch_billno = mchBillno;
	}

	public String getMch_id() {
		return mch_id;
	}

	public void setMch_id(String mchId) {
		mch_id = mchId;
	}

	public String getSub_mch_id() {
		return sub_mch_id;
	}

	public void setSub_mch_id(String subMchId) {
		sub_mch_id = subMchId;
	}

	public String getWxappid() {
		return wxappid;
	}

	public void setWxappid(String wxappid) {
		this.wxappid = wxappid;
	}

	public String getNick_name() {
		return nick_name;
	}

	public void setNick_name(String nickName) {
		nick_name = nickName;
	}

	public String getSend_name() {
		return send_name;
	}

	public void setSend_name(String sendName) {
		send_name = sendName;
	}

	public String getRe_openid() {
		return re_openid;
	}

	public void setRe_openid(String reOpenid) {
		re_openid = reOpenid;
	}

	public String getTotal_amount() {
		return total_amount;
	}

	public void setTotal_amount(String totalAmount) {
		total_amount = totalAmount;
	}

	public String getMin_value() {
		return min_value;
	}

	public void setMin_value(String minValue) {
		min_value = minValue;
	}

	public String getMax_value() {
		return max_value;
	}

	public void setMax_value(String maxValue) {
		max_value = maxValue;
	}

	public String getTotal_num() {
		return total_num;
	}

	public void setTotal_num(String totalNum) {
		total_num = totalNum;
	}

	public String getWishing() {
		return wishing;
	}

	public void setWishing(String wishing) {
		this.wishing = wishing;
	}

	public String getClient_ip() {
		return client_ip;
	}

	public void setClient_ip(String clientIp) {
		client_ip = clientIp;
	}

	public String getAct_name() {
		return act_name;
	}

	public void setAct_name(String actName) {
		act_name = actName;
	}

	public String getRemark() {
		return remark;
	}

	public void setRemark(String remark) {
		this.remark = remark;
	}

	public String getLogo_imgurl() {
		return logo_imgurl;
	}

	public void setLogo_imgurl(String logoImgurl) {
		logo_imgurl = logoImgurl;
	}

	public String getShare_content() {
		return share_content;
	}

	public void setShare_content(String shareContent) {
		share_content = shareContent;
	}

	public String getShare_url() {
		return share_url;
	}

	public void setShare_url(String shareUrl) {
		share_url = shareUrl;
	}

	public String getShare_imgurl() {
		return share_imgurl;
	}

	public void setShare_imgurl(String shareImgurl) {
		share_imgurl = shareImgurl;
	}

}
```

###2.创建签名

```java
/**
	 * 得到带签名的数据包
	 * @param opendid
	 * @param totalAmount
	 * @return
	 * @throws Exception
	 */
	public static SendRedPackPo getSendPack(String opendid,String totalAmount) throws Exception{
		// 获取uuid作为随机字符串
				String nonceStr = UUID.randomUUID().toString();
				String today = new SimpleDateFormat("yyyyMMdd").format(new Date());
				String code = createCode(10);
				String mch_id = PropertiesLoader.getPropertiesByName("partnerId");// 商户号
				String appid = PropertiesLoader.getPropertiesByName("appId");;
				SendRedPackPo sendRedPackPo = new SendRedPackPo();

				sendRedPackPo.setNonce_str(nonceStr);
				sendRedPackPo.setMch_billno(mch_id + today + code);
				sendRedPackPo.setMch_id(mch_id);
				sendRedPackPo.setWxappid(appid);
				sendRedPackPo.setNick_name("赵小轩");
				sendRedPackPo.setSend_name("东吴有限公司");
				sendRedPackPo.setRe_openid(opendid);
				sendRedPackPo.setTotal_amount(totalAmount);
				sendRedPackPo.setMin_value(totalAmount);
				sendRedPackPo.setMax_value(totalAmount);
				sendRedPackPo.setTotal_num("1");
				sendRedPackPo.setWishing("恭喜你得到一个红包！");
				sendRedPackPo.setClient_ip(Inet4Address.getLocalHost().toString()); // IP
				sendRedPackPo.setAct_name("转盘抢红包");
				sendRedPackPo.setRemark("快来抢红包！");

				// 把请求参数打包成数组
				Map<String, String> sParaTemp = new HashMap<String, String>();
				sParaTemp.put("nonce_str", sendRedPackPo.getNonce_str());
				sParaTemp.put("mch_billno", sendRedPackPo.getMch_billno());
				sParaTemp.put("mch_id", sendRedPackPo.getMch_id());
				sParaTemp.put("wxappid", sendRedPackPo.getWxappid());
				sParaTemp.put("nick_name", sendRedPackPo.getNick_name());
				sParaTemp.put("send_name", sendRedPackPo.getSend_name());
				sParaTemp.put("re_openid", sendRedPackPo.getRe_openid());
				sParaTemp.put("total_amount", sendRedPackPo.getTotal_amount());
				sParaTemp.put("min_value", sendRedPackPo.getMin_value());
				sParaTemp.put("max_value", sendRedPackPo.getMax_value());
				sParaTemp.put("total_num", sendRedPackPo.getTotal_num());
				sParaTemp.put("wishing", sendRedPackPo.getWishing());
				sParaTemp.put("client_ip", sendRedPackPo.getClient_ip());
				sParaTemp.put("act_name", sendRedPackPo.getAct_name());
				sParaTemp.put("remark", sendRedPackPo.getRemark());

				// 除去数组中的空值和签名参数
				Map<String, String> sPara = paraFilter(sParaTemp);
				String prestr = createLinkString(sPara); // 把数组所有元素，按照“参数=参数值”的模式用“&”字符拼接成字符串
				String partnerKey=PropertiesLoader.getPropertiesByName("partnerKey");//从配置文件中读取商户key
				String key = "&key="+partnerKey; // 商户支付密钥
				String mysign = sign(prestr, key, "utf-8").toUpperCase();

				sendRedPackPo.setSign(mysign);
				return sendRedPackPo;
	}


	/**
	 * 签名字符串
	 * 
	 * @param text
	 *            需要签名的字符串
	 * @param key
	 *            密钥
	 * @param input_charset
	 *            编码格式
	 * @return 签名结果
	 */
	public static String sign(String text, String key, String input_charset) {
		text = text + key;
		return DigestUtils.md5Hex(getContentBytes(text, input_charset));
	}

	/**
	 * 签名字符串
	 * 
	 * @param text
	 *            需要签名的字符串
	 * @param sign
	 *            签名结果
	 * @param key
	 *            密钥
	 * @param input_charset
	 *            编码格式
	 * @return 签名结果
	 */
	public static boolean verify(String text, String sign, String key,
			String input_charset) {
		text = text + key;
		String mysign = DigestUtils
				.md5Hex(getContentBytes(text, input_charset));
		if (mysign.equals(sign)) {
			return true;
		} else {
			return false;
		}
	}

		/**
	 * @param content
	 * @param charset
	 * @return
	 * @throws SignatureException
	 * @throws UnsupportedEncodingException
	 */
	private static byte[] getContentBytes(String content, String charset) {
		if (charset == null || "".equals(charset)) {
			return content.getBytes();
		}
		try {
			return content.getBytes(charset);
		} catch (UnsupportedEncodingException e) {
			throw new RuntimeException("MD5签名过程中出现错误,指定的编码集不对,您目前指定的编码集是:"
					+ charset);
		}
	}

	/**
	 * 生成6位或10位随机数 param codeLength(多少位)
	 * 
	 * @return
	 */
	private static String createCode(int codeLength) {
		String code = "";
		for (int i = 0; i < codeLength; i++) {
			code += (int) (Math.random() * 9);
		}
		return code;
	}

	private static boolean isValidChar(char ch) {
		if ((ch >= '0' && ch <= '9') || (ch >= 'A' && ch <= 'Z')
				|| (ch >= 'a' && ch <= 'z'))
			return true;
		if ((ch >= 0x4e00 && ch <= 0x7fff) || (ch >= 0x8000 && ch <= 0x952f))
			return true;// 简体中文汉字编码
		return false;
	}

	/**
	 * 除去数组中的空值和签名参数
	 * 
	 * @param sArray
	 *            签名参数组
	 * @return 去掉空值与签名参数后的新签名参数组
	 */
	public static Map<String, String> paraFilter(Map<String, String> sArray) {

		Map<String, String> result = new HashMap<String, String>();

		if (sArray == null || sArray.size() <= 0) {
			return result;
		}

		for (String key : sArray.keySet()) {
			String value = sArray.get(key);
			if (value == null || value.equals("")
					|| key.equalsIgnoreCase("sign")
					|| key.equalsIgnoreCase("sign_type")) {
				continue;
			}
			result.put(key, value);
		}

		return result;
	}

	/**
	 * 把数组所有元素排序，并按照“参数=参数值”的模式用“&”字符拼接成字符串
	 * 
	 * @param params
	 *            需要排序并参与字符拼接的参数组
	 * @return 拼接后字符串
	 */
	public static String createLinkString(Map<String, String> params) {

		List<String> keys = new ArrayList<String>(params.keySet());
		Collections.sort(keys);

		String prestr = "";

		for (int i = 0; i < keys.size(); i++) {
			String key = keys.get(i);
			String value = params.get(key);

			if (i == keys.size() - 1) {// 拼接时，不包括最后一个&字符
				prestr = prestr + key + "=" + value;
			} else {
				prestr = prestr + key + "=" + value + "&";
			}
		}

		return prestr;
	}
```

###3.将参数post给微信服务器（需要证书）

```java
/**
	 * 发送现金红包
	 * @param opendid  发送给指定openId红包
	 * @param totalAmount 发送金额
	 * @throws Exception  未找到属性
	 */
	public static  Map<String, String> sendRedPack(String opendid,String totalAmount) throws Exception {
		
		SendRedPackPo sendRedPackPo=getSendPack(opendid,totalAmount);
		String respXml = MessageUtil.messageToXml(sendRedPackPo);
		String mch_id = PropertiesLoader.getPropertiesByName("partnerId");// 商户号
		// 打印respXml发现，得到的xml中有“__”不对，应该替换成“_”
		respXml = respXml.replace("__", "_");

		// 将解析结果存储在HashMap中
		Map<String, String> map = new HashMap<String, String>();

		KeyStore keyStore = KeyStore.getInstance("PKCS12");
		FileInputStream instream = new FileInputStream(new File(
				"/home/apiclient_cert.p12")); // 此处为证书所放的绝对路径

		try {
			keyStore.load(instream, mch_id.toCharArray());
		} finally {
			instream.close();
		}

		// Trust own CA and all self-signed certs
		SSLContext sslcontext = SSLContexts.custom()
				.loadKeyMaterial(keyStore, mch_id.toCharArray()).build();
		// Allow TLSv1 protocol only
		SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
				sslcontext, new String[] { "TLSv1" }, null,
				SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);
		CloseableHttpClient httpclient = HttpClients.custom()
				.setSSLSocketFactory(sslsf).build();

		try {
			
			HttpPost httpPost = new HttpPost(
					"https://api.mch.weixin.qq.com/mmpaymkttransfers/sendredpack");

			StringEntity reqEntity = new StringEntity(respXml, "utf-8");

			// 设置类型
			reqEntity.setContentType("application/x-www-form-urlencoded");

			httpPost.setEntity(reqEntity);

			System.out.println("executing request" + httpPost.getRequestLine());

			CloseableHttpResponse response = httpclient.execute(httpPost);
			try {
				HttpEntity entity = response.getEntity();
				System.out.println(response.getStatusLine());
				if (entity != null) {

					// 从request中取得输入流
					InputStream inputStream = entity.getContent();
					// 读取输入流
					SAXReader reader = new SAXReader();
					Document document = reader.read(inputStream);
					// 得到xml根元素
					Element root = document.getRootElement();
					// 得到根元素的所有子节点
					List<Element> elementList = root.elements();

					// 遍历所有子节点
					for (Element e : elementList)
						map.put(e.getName(), e.getText());

					// 释放资源
					inputStream.close();
					inputStream = null;

				}
				//关闭流
				//EntityUtils.consume();
				entity.getContent().close();
			} finally {
				response.close();
			}
		} finally {
			httpclient.close();
		}

		return map;

	}
```
