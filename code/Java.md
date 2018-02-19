## 简繁体转换 - Java调用示例代码

#### 简繁体转换
简繁体转换

#### 该产品拥有以下APIs：
1. 简繁体转换

**注意，该示例代码仅适用于 [www.apishop.net](www.apishop.net "www.apishop.net") 网站下API**
**使用该产品前，您需要通过 [https://www.apishop.net/#/api/detail/?productID=121](https://www.apishop.net/#/api/detail/?productID=121 "https://www.apishop.net/#/api/detail/?productID=121") 申请API服务**

---

#### 1.简繁体转换

```
package net.apishop.www.controller;

import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Map;
import com.alibaba.fastjson.JSONObject;

/**
 * httpUrlConnection访问远程接口工具
 */
public class Api
{

	/**
	 * 方法体说明：向远程接口发起请求，返回字节流类型结果
	 * param url 接口地址
	 * param requestMethod 请求方式
	 * param params 传递参数     重点：参数值需要用Base64进行转码
	 * return InputStream 返回结果
	 */
	public static InputStream httpRequestToStream(String url, String requestMethod, Map<String, String> params)
	{
		InputStream is = null;
		try
		{
			String parameters = "";
			boolean hasParams = false;
			// 将参数集合拼接成特定格式，如name=zhangsan&age=24
			for (String key : params.keySet())
			{
				String value = URLEncoder.encode(params.get(key), "UTF-8");
				parameters += key + "=" + value + "&";
				hasParams = true;
			}
			if (hasParams)
			{
				parameters = parameters.substring(0, parameters.length() - 1);
			}

			// 请求方式是否为get
			boolean isGet = "get".equalsIgnoreCase(requestMethod);
			// 请求方式是否为post
			boolean isPost = "post".equalsIgnoreCase(requestMethod);
			if (isGet)
			{
				url += "?" + parameters;
			}

			URL u = new URL(url);
			HttpURLConnection conn = (HttpURLConnection) u.openConnection();

			// 请求的参数类型(使用restlet框架时，为了兼容框架，必须设置Content-Type为“”空)
			conn.setRequestProperty("Content-Type", "application/octet-stream");
			// conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
			// 设置连接超时时间
			conn.setConnectTimeout(50000);
			// 设置读取返回内容超时时间
			conn.setReadTimeout(50000);
			// 设置向HttpURLConnection对象中输出，因为post方式将请求参数放在http正文内，因此需要设置为ture，默认false
			if (isPost)
			{
				conn.setDoOutput(true);
			}
			// 设置从HttpURLConnection对象读入，默认为true
			conn.setDoInput(true);
			// 设置是否使用缓存，post方式不能使用缓存
			if (isPost)
			{
				conn.setUseCaches(false);
			}
			// 设置请求方式，默认为GET
			conn.setRequestMethod(requestMethod);

			// post方式需要将传递的参数输出到conn对象中
			if (isPost)
			{
				DataOutputStream dos = new DataOutputStream(conn.getOutputStream());
				dos.writeBytes(parameters);
				dos.flush();
				dos.close();
			}

			// 从HttpURLConnection对象中读取响应的消息
			// 执行该语句时才正式发起请求
			is = conn.getInputStream();
		}
		catch(UnsupportedEncodingException e)
		{
			e.printStackTrace();
		}
		catch(MalformedURLException e)
		{
			e.printStackTrace();
		}
		catch(IOException e)
		{
			e.printStackTrace();
		}
		return is;
	}

	public static void main(String args[])
    {
		String url = "https://api.apishop.net/common/postcode/getPostCodeByAddr";
		String requestMethod = "POST";
		Map<String, String> params = new HashMap<String, String>();			
		params.put("words", ""); //带转换的中文(最大长度65535)			
		params.put("type", ""); //转换类型[1:简体到繁体,2:繁体到简体,3:香港繁体（香港小学学习字词表标准）到简体,4:简体到香港繁体（香港小学学习字词表标准）,5:简体到台湾正体,6:简体到繁体（台湾正体标准）并转换为台湾常用词汇,7: 繁体到香港繁体（香港小学学习字词表标准）,8:繁体到台湾正体,9:台湾正体到简体,10:繁体（台湾正体标准）到简体并转换为中国大陆常用词汇],默认为1		
		String result = null;
		try
		{
			InputStream is = httpRequestToStream(url, requestMethod, params);
			byte[] b = new byte[is.available()];
			is.read(b);
			result = new String(b);
		}
		catch(IOException e)
		{
		    e.printStackTrace();
		}
		if (result != null)
		{
		    JSONObject jsonObject = JSONObject.parseObject(result);
		    String status_code = jsonObject.getString("statusCode");
		    if (status_code == "000000")
		    {
			// 状态码为000000, 说明请求成功
			System.out.println("请求成功：" + jsonObject.getString("result"));
		    }
		    else
		    {
			// 状态码非000000, 说明请求失败
			System.out.println("请求失败：" + jsonObject.getString("desc"));
		    }
		}
		else
		{
		    // 返回内容异常，发送请求失败，以下可根据业务逻辑自行修改
		    System.out.println("发送请求失败");
		}
	}
}
```