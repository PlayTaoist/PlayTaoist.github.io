---
title:  "SpringBoot2.0微信小程序支付多次回调问题"
categories: SpringBoot2.0 Shiro
tags: SpringBoot2.0 Shiro
author: LIUREN
---

# SpringBoot2.0微信小程序支付多次回调问题

>  WxJava - 微信开发 Java SDK（开发工具包); 支持包括微信支付、开放平台、公众号、企业微信/企业号、小程序等微信功能的后端开发。



## 第一步： 支付回调问题

微信小程序使用请参考上一篇博客：<https://www.codepeople.cn/2019/04/15/SpringBoot2.x-WX-Pay/>

这里主要讲的是微信小程序回调问题

微信这个坑，对第一次接触微信支付功能的人可能都会不由自主的踩一下，不是程序猿不给力而是微信回调敌人太强大。

1.首先确定支付回调是返回的String

2.确定返回给微信的通知是正确的

首先看一下微信支付回调地址中的方法怎么写吧？

```java
@SuppressWarnings("deprecation")
	@ApiOperation("微信支付回调地址")
	@PostMapping("/notify") // 返回订单号
	public String payNotify(HttpServletRequest request, HttpServletResponse response) {
		log.info("======================>>>微信支付回调<<======================");
		log.info("======================>>>微信支付回调<<======================");
		String resXml = "";
		try {
			/*
			 * HttpSession session = request.getSession(); String mobile = (String)
			 * session.getAttribute("userphone"); if (StringUtils.isBlank(mobile)) { return
			 * R.error(401, "session获取不到授权手机号！"); } //获取用户手机号,根据用户手机号获取用户ID
			 * AuthorizationEntity user = authorizationService.getOneByMobile(mobile);
			 */
			String xmlResult = IOUtils.toString(request.getInputStream(), request.getCharacterEncoding());
			WxPayOrderNotifyResult notifyResult = wxService.parseOrderNotifyResult(xmlResult);
			// 结果正确 outTradeNo
			String orderId = notifyResult.getOutTradeNo();
			String tradeNo = notifyResult.getTransactionId();
			String totalFee = BaseWxPayResult.fenToYuan(notifyResult.getTotalFee());
			log.info("微信支付回调付款总金额==>{}元", totalFee);
			// 自己处理订单的业务逻辑，需要判断订单是否已经支付过，否则可能会重复调用
			// 通知微信.异步确认成功.必写.不然会一直通知后台.十次之后就认为交易失败了.
			resXml = "<xml>" + "<return_code><![CDATA[SUCCESS]]></return_code>"
					+ "<return_msg><![CDATA[OK]]></return_msg>" + "</xml> ";
			response.setContentType("text/xml");
			BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
			out.write(resXml.getBytes());
			out.flush();
			out.close();
//			return WxPayNotifyResponse.success("成功");
			return resXml;
		} catch (Exception e) {
			log.error("微信回调结果异常,异常原因{}", e.getMessage());
			// WxPayNotifyResponse.fail(e.getMessage());

			return WxPayNotifyResponse.success("code:" + 9999 + "微信回调结果异常,异常原因:" + e.getMessage());
		}
	}
```

注意：

`@PostMapping("/notify") ` 这个注解使用，使其返回的是是String，***千万不要加`@ResponseBody`注解***

`response.setContentType("text/xml");` ***这个方法一定要加上***

使用这种格式的返回数据给微信，微信就会收到通知，不会发生多次回调

如果方法上加上了`@ResponseBody`该注解，微信支付会一直重复回调10次。

为了防止微信支付回调的多次调用。支付逻辑一定要判断是否已经支付完成。还有订单生成问题也一定要判断。不然可能会造成多个订单数据问题。



==================================================================

[博客地址](https://www.codepeople.cn)：<https://www.codepeople.cn>

==================================================================

微信公众号：

![](https://www.codepeople.cn/imges/weixin_icon/weixin.jpg)