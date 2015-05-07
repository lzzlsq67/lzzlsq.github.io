---
layout: post
title:  "微信JS-SDK对接分享接口Java版"
keywords: "wechat"
description: "使用最新的JS-SDK进行微信接口的分享开发"
category: wechat
tags: wechat java
---

在2015年1月腾讯推出了微信JS-SDK,分享到朋友圈必须使用JS-SDK，而且还必须要绑定安全域名，进一步规范了朋友圈分享鱼龙混杂的场面。
下面主要介绍使用JS-SDK来对接微信的分享接口，主要包括，分享到朋友圈，分享给朋友，分享到QQ，分享到腾讯微博。
##1.绑定域名
  先登录微信公众平台进入`公众号设置`的`功能设置`里填写`JS接口安全域名`。需要注意的是安全域名需要的是`ICP备案`的一级域名，二级域名无效。


##2.引入JS文件

在需要调用JS接口的页面引入如下JS文件，（支持https）：http://res.wx.qq.com/open/js/jweixin-1.0.0.js

备注：支持使用 AMD/CMD 标准模块加载方法加载
##3.通过config接口注入权限验证配置

所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次）

```javascript
	wx.config({
	    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
	    appId: '', // 必填，公众号的唯一标识
	    timestamp: , // 必填，生成签名的时间戳
	    nonceStr: '', // 必填，生成签名的随机串
	    signature: '',// 必填，jsapi_ticket
	    jsApiList: [] // 必填，需要使用的JS接口列表
	});
```
这里使用ajax异步调用后台java生成的接口,`getConfig`是在下面使用Spring MVC配置的生成ticket的接口

```javascript
//从服务端得到配置信息
$(document).ready(function(){
 var timestamp;//时间戳
 var nonceStr;//随机字符串
 var signature;//得到的签名
 $.ajax({
		url:"getConfig",// 跳转到 action  
		async:true,
		type:'get',  
		data:{url:location.href.split('#')[0]},//得到需要分享页面的url
		cache:false,    
		dataType:'json',    
		success:function(data) {    
				if(data!=null ){    
					timestamp=data.timestamp;//得到时间戳
					nonceStr=data.nonceStr;//得到随机字符串
					signature=data.signature;//得到签名
					//微信配置
					wx.config({
					    debug: false, 
					    appId: "wx0a458933e469f816", 
					    timestamp:timestamp, 
					    nonceStr: nonceStr, 
					    signature: signature,
					    jsApiList: ['onMenuShareTimeline', 'onMenuShareAppMessage'] // 功能列表
					});
				}else{   
				}    
		},    
		error : function() {    
		}    
  });
});

```

全局票据`jsapi_ticket`生成方法

jsapi_ticket是公众号用于调用微信JS接口的临时票据。正常情况下，jsapi_ticket的有效期为7200秒，通过access_token来获取。
由于获取jsapi_ticket的api调用次数非常有限，频繁刷新jsapi_ticket会导致api调用受限，影响自身业务，要自己的服务全局缓存jsapi_ticket 。
使用Java获得jsapi_ticket
jsapi_ticket实体

```java
package com.sz.kcygl.kernel.weixin.entity;

/**
 * @author little轩轩
 */
public class JsApiTicket {
    /**
     * 获取到的jsApiTicket凭证
     */
    private String ticket;
    /**
     * 凭证有效时间，单位：秒
     */
    private long expires_in;


    public long getExpires_in() {
        return expires_in;
    }

    public void setExpires_in(long expires_in) {
        this.expires_in = expires_in;
    }

	public String getTicket() {
		return ticket;
	}

	public void setTicket(String ticket) {
		this.ticket = ticket;
	}
    
}

```

生成jsapi_ticket的工具类JsApiTicketUtil

```java
package com.sz.kcygl.kernel.weixin.util;
import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Formatter;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;

import com.sz.kcygl.common.util.HttpClientUtil;
import com.sz.kcygl.common.util.JSONUtil;
import com.sz.kcygl.common.util.PropertiesLoader;
import com.sz.kcygl.kernel.weixin.entity.ErrorMessage;
import com.sz.kcygl.kernel.weixin.entity.JsApiTicket;


public class JsApiTicketUtil {
	static Logger logger = Logger.getLogger(JsApiTicketUtil.class);
	    /**
	     * 获得JsAPITicket
	     * @param token
	     * @return
	     */
	    public static String getJsApiTicket(){
	    	String token="";
			try {
				token = AccessTokenUtil.getAccessToken();
			} catch (Exception e) {
				logger.info("获取Token失败");
			}
			logger.info("得到token"+token);
	    	String reqUrl="https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token="+token+"&type=jsapi";
	    	String resDoc="";
			try {
				resDoc = HttpClientUtil.postRequest(reqUrl, "");
			} catch (Exception e) {
				logger.info("获取jsApiTicket出错");
			}
			if(StringUtils.isNotEmpty(resDoc)){
				JsApiTicket ticket=(JsApiTicket) JSONUtil.jsonToObject(resDoc, JsApiTicket.class);
				if(ticket!=null&&StringUtils.isNotEmpty(ticket.getTicket())){
					logger.info("jsticket:"+ticket.getTicket());
					logger.info("expires_in	:"+ticket.getExpires_in());
					return ticket.getTicket();
				}else {
	                //出错了
	                ErrorMessage error = (ErrorMessage) JSONUtil.jsonToObject(resDoc, ErrorMessage.class);
	                logger.error("clll wx error,errcode="+error.getErrcode()+", errmsg="+error.getErrmsg());
	            }
			}
	    	return "error"; 
	    	
	    }
	    /**
	     * 用于给定的jsticket和url按照SHA-1签名
	     * @param jsapi_ticket
	     * @param url
	     * @return
	     */
	    public static Map<String, String> sign(String jsapi_ticket, String url) {
	        Map<String, String> ret = new HashMap<String, String>();
	        String nonce_str = create_nonce_str();
	        String timestamp = create_timestamp();
	        String string1;
	        String signature = "";
	        //注意这里参数名必须全部小写，且必须有序
	        string1 = "jsapi_ticket=" + jsapi_ticket +
	                  "&noncestr=" + nonce_str +
	                  "&timestamp=" + timestamp +
	                  "&url=" + url;
	        System.out.println(string1);

	        try
	        {
	            MessageDigest crypt = MessageDigest.getInstance("SHA-1");
	            crypt.reset();
	            crypt.update(string1.getBytes("UTF-8"));
	            signature = byteToHex(crypt.digest());
	        }
	        catch (NoSuchAlgorithmException e)
	        {
	            e.printStackTrace();
	        }
	        catch (UnsupportedEncodingException e)
	        {
	            e.printStackTrace();
	        }
	        ret.put("url", url);
	        ret.put("jsapi_ticket", jsapi_ticket);
	        ret.put("nonceStr", nonce_str);
	        ret.put("timestamp", timestamp);
	        ret.put("signature", signature);

	        return ret;
	    }

	    private static String byteToHex(final byte[] hash) {
	        Formatter formatter = new Formatter();
	        for (byte b : hash)
	        {
	            formatter.format("%02x", b);
	        }
	        String result = formatter.toString();
	        formatter.close();
	        return result;
	    }

	    private static String create_nonce_str() {
	        return UUID.randomUUID().toString();
	    }

	    private static String create_timestamp() {
	        return Long.toString(System.currentTimeMillis() / 1000);
	    }

}

```
使用缓存保存jsapi_ticket对象，使用Quartz框架每隔一段时间从微信服务器更新一次数据，当JsApiTicket的时间超过了7200秒就从服务器更新数据。

```java
package com.sz.kcygl.web.controller.wx;

import java.util.Date;

import org.apache.log4j.Logger;

import com.sz.kcygl.common.constants.CacheConstants;
import com.sz.kcygl.kernel.weixin.entity.JsApiTicket;
import com.sz.kcygl.kernel.weixin.util.JsApiTicketUtil;
import com.sz.kcygl.web.cache.JsApiTicketCache;

public class JsApiTicketJob {

	public static Logger logger = Logger.getLogger(JsApiTicketJob.class);


	public String getJsApiTicket() throws Exception {
		//得到当前ticket缓存
		JsApiTicketCache ticketCache=JsApiTicketCache.getInstance();
		logger.info("得到缓存对象："+ticketCache);
		JsApiTicket ticket=(JsApiTicket) ticketCache.getJsApiTicketCache(CacheConstants.JS_API_TICKET);
		Long validMin=null;
		String jsApiTicket="";
		if(ticket!=null&&ticket.getUpdatedDate()!=null){//如果票据对象不为空
			jsApiTicket=ticket.getTicket();//得到Ticket
			Date cdate = ticket.getUpdatedDate();// ticket更新时间
			logger.info("=========================================");
			logger.info("得到ticket更新时间:"+cdate);
			Date ndate = new Date();// 当前时间
			// 秒
			Long miao = (((ndate).getTime() - cdate.getTime())) / 1000;
			logger.info("相隔:" + miao + "秒");
			validMin = 7200-miao;
//			 如果相隔时间超过900秒
			if (validMin <900) {
				logger.info("间隔时间超过");
				jsApiTicket = JsApiTicketUtil.getJsApiTicket();
				ticket.setTicket(jsApiTicket);
				ticket.setUpdatedDate(new Date());
				logger.info("将新的jsApiTicket更新到缓存");
				ticketCache.setJsApiTicketCache(ticket);
			}
		}else{
			ticket=new JsApiTicket();
			jsApiTicket = JsApiTicketUtil.getJsApiTicket();
			ticket.setTicket(jsApiTicket);
			ticket.setCreatedDate(new Date());
			ticket.setUpdatedDate(new Date());
			logger.info("将新的jsApiTicket放入缓存");
			ticketCache.setJsApiTicketCache(ticket);
		}
		logger.info("得到jsApiTicket："+ticket.getTicket());
		logger.info("=========================================");
		return ticket.getTicket();
	}
}

```
JsApiTicket缓存类

```java
package com.sz.kcygl.web.cache;

import java.util.HashMap;
import java.util.Map;

import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;

import com.sz.kcygl.common.constants.CacheConstants;
import com.sz.kcygl.kernel.weixin.entity.JsApiTicket;
/**
 * JsApiTicket缓存
 * @author little轩轩
 *
 */
public class JsApiTicketCache {
	private Logger logger = Logger.getLogger(JsApiTicketCache.class);
	private  Map<String,Object> cacheMap = new HashMap<String,Object>();
	//单例模式
	 private static JsApiTicketCache ticketChe;  
	    private JsApiTicketCache (){}  
	  
	    public static JsApiTicketCache getInstance() {  
	    if (ticketChe == null) {  
	    	ticketChe = new JsApiTicketCache();  
	    }  
	    return ticketChe;  
	    }  
	    /**
		 * 设置JsApiTicket缓存
		 */
		
	public void  setJsApiTicketCache(JsApiTicket ticket){
		logger.info("进入JsApiTicket缓存");
		if(StringUtils.isNotEmpty(ticket.getTicket())){
			if(!ticket.getTicket().equals("error")){
				cacheMap.put(CacheConstants.JS_API_TICKET,ticket );
				logger.info("将JsApiTicket放入缓存success");
			}else{
				logger.info("JsApiTicket获取失败，将JsApiTicket放入缓存failure");
			}
		}else{
			logger.info("将JsApiTicket放入缓存failure");
		}
	}
	
	/**
	 * 得到JsApiTicket缓存
	 * @param key
	 * @return
	 */
	public Object getJsApiTicketCache(String key){
		return cacheMap.get(key);
	}
	
}
```

使用Quartz框架配置JsApiTicketJob，每隔15分钟更新一次

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-2.5.xsd">

	<!-- 配置jsApiTicket -->
		<bean name ="jsApiTicketJob" class="com.sz.kcygl.web.controller.wx.JsApiTicketJob"/>
		
		<bean id="methodInvokingTicketJobDetail"	
		class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
		<property name="targetObject">
			<ref bean="jsApiTicketJob" />
		</property>
		<property name="targetMethod">
			<value>getJsApiTicket</value>
		</property>
	</bean>
	
	<!-- 配置触发器 -->
	 <bean id="jsTicketTrigger"  class="org.springframework.scheduling.quartz.CronTriggerBean">    
       <property name="jobDetail">    
           <ref bean="methodInvokingTicketJobDetail"/>    
       </property>    
		<property name="cronExpression">
			<!--<value>0 * 08-21 * * ?</value>-->
			<!-- <value>*/5 * * * * ?</value> -->
			<value>0 0/15 * * * ?</value>
		</property>
    </bean>  

	<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
       <property name="triggers">
           <list>
              <ref local="cronTrigger" />
               <ref local="jsTicketTrigger" />  
           </list>
       </property>
    </bean>
</beans>
```
使用springMvc配置获取jsapi_ticket的接口

```java
package com.sz.kcygl.web.controller.wx;


import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import net.sf.json.JSONObject;

import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.sz.kcygl.common.constants.CacheConstants;
import com.sz.kcygl.kernel.weixin.entity.JsApiTicket;
import com.sz.kcygl.kernel.weixin.util.JsApiTicketUtil;
import com.sz.kcygl.web.cache.JsApiTicketCache;
import com.sz.kcygl.web.controller.BaseController;
/**
 * 获得微信Js-Api初始化参数
 * @author 赵宏轩
 *
 */
@Controller
public class JsApiController extends BaseController {

	Logger logger = Logger.getLogger(this.getClass());
	/**
	 * 微信JS-SDK获得初始化参数
	 * 
	 * @param request
	 * @param response
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getConfig", method = RequestMethod.GET)
	public String getConfig(HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		String strConfig="";
		JsApiTicketCache ticketCache=JsApiTicketCache.getInstance();
		String jsapi_ticket=((JsApiTicket)ticketCache.getJsApiTicketCache(CacheConstants.JS_API_TICKET)).getTicket();
		logger.info("通过缓存得到jsapi_ticket："+jsapi_ticket);
		if(StringUtils.isEmpty(jsapi_ticket)){
			jsapi_ticket=JsApiTicketUtil.getJsApiTicket();
			logger.info("缓存为空进入工具类，通过和微信交互得到JsAPITicket："+jsapi_ticket);
		}
		String url=getParameter(request, "url");
		logger.info("分享的url"+url);
		Map<String,String>config=JsApiTicketUtil.sign(jsapi_ticket, url);
		JSONObject jsonObject = JSONObject.fromObject(config);
		strConfig=jsonObject.toString();
		logger.info("返回的json对象为："+strConfig);
		this.writerResponse(response, strConfig);//将结果写进相应中
		return null;
		}

}

```

##4.通过ready接口处理成功验证

```javascript
wx.ready(function () {
	var shareUrl="http://zeusjava.com/weixin/toStartReunion";
	var imageUrl="http://zeusjava.com/weixin/reunion/images/smallReunion.jpg";
	var title="微信分享测试！";
	var desc="微信分享描述";
	 wx.onMenuShareAppMessage({
	      title: title,			//分享标题
	      desc: desc,		//分享描述
	      link: shareUrl,	//分享链接
	      imgUrl: imageUrl,//分享图片
	      success: function (res) {
	      },
	      cancel: function (res) {
	      }
	  });
	  wx.onMenuShareTimeline({
		    title:title,	//分享标题
		    desc: desc,//分享描述
		    link: shareUrl, // 分享链接
		    imgUrl: imageUrl, // 分享图标
		    success: function () { 
		    },
		    cancel: function () { 
		    }
		 });
	 wx.showOptionMenu();
  });
```

  