###地址栏指令
都是以 / 开头


####正式环境 测试环境切换

	/test true  
	/test false
输入后两秒会自动退出程序，必须手动重新打开app。切换后永久有效

####js调用显示结果

	/alert true
	/alert false
	
####显示渠道

	/show channel
	
####获取友盟token到剪切板

	/umtoken
	
####跳转到m.wb.ngx

	/m.wb.ngx
	




###微信支付：

jsapi 3，appVersion：2.6.0以上

nativePay

参数：

pay_sn：字符串类型，流水号

amount：字符串类型，支付金额

回调返回：

	{

		error_code:0

		error_message:""

		user_message:""

		data:{

			pay_sn:""

			pay_amount:""

			pay_type:"wx"jsapi 4后加入 
			resoult_code:int 
		} 
	} 
		
		
doesSupportWXPay 不要调用，已经废弃 

###支付宝支付 
jsapi：4, appVersion 2.6.3以上 

aliNativePay 参数： pay_sn：字符串类型，流水号 amount：字符串类型，支付金额 

回调返回： 

	{ 
		error_code:0 
		error_message:"" 
		user_message:"" 
		data:{ 
			pay_sn:"" 
			pay_amount:""
			pay_type:"alipay" 
			resoult_code:int 
		} 
	} 
	
	
###招商银行支付
 
	
####error_code:
code | 说明 
----|------
0 | 支付成功  
-1 | 支付错误（第三方返回的）  
-2 | 用户取消支付（如果是支付宝，还可能是签名错误或其他错误，支付宝没有区分开）
1010|没有安装微信或微信版本不支持
-1024|  程序异常

####支付宝code映射
9000→0

8000→8000

4000,6002→-1

6001→-2



0 支付成功；-1: 支付错误（第三方返回的）；-2：用户取消支付（如果是支付宝，还可能是签名错误或其他错误，支付宝没有区分开）；1010：没有安装微信或微信版本不支持；-1024:程序异常 支付宝映射
9000→0 8000→8000 4000,6002→-1 6001→-2





微信支付：

jsapi 3，appVersion：2.6.0以上

nativePay

参数：

pay_sn：字符串类型，流水号

amount：字符串类型，支付金额

回调返回：

{

error_code:0

error_message:""

user_message:""

data:{

pay_sn:""

pay_amount:""

pay_type:"wx"jsapi 4后加入 resoult_code:int } } doesSupportWXPay 不要调用，已经废弃 支付宝支付 jsapi：4, appVersion 2.6.3以上 aliNativePay 参数： pay_sn：字符串类型，流水号 amount：字符串类型，支付金额 回调返回： { error_code:0 error_message:"" user_message:"" data:{ pay_sn:"" pay_amount:"" pay_type:"alipay" resoult_code:int } } 招商银行支付 error_code:0 支付成功；-1: 支付错误（第三方返回的）；-2：用户取消支付（如果是支付宝，还可能是签名错误或其他错误，支付宝没有区分开）；1010：没有安装微信或微信版本不支持；-1024:程序异常 支付宝映射
9000→0 8000→8000 4000,6002→-1 6001→-2