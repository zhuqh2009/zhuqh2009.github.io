---
layout:     post
title:      从Apache PoolingHttpClientConnectionManager看连接
category: blog
description: 从Apache PoolingHttpClientConnectionManager看连接
---


### 从Apache PoolingHttpClientConnectionManager看连接

缘由： 微服务，以及多子系统外部服务调用的架构中，经常要借用REST方式访问，如果每次都握手连接访问，效率低下。如果没有dubbo这样的RPC框架，又想提高HTTP访问效率，那么HTTP连接池就是比较合适的方式，这样的实现基于keep-alive机制，虽然这样的长连接没有心跳的支持，并且受限于服务提供方的keep-alive时长，但对于连续频繁的服务访问，对比于每次都建立tcp连接，这样的连接复用方式还是相对高效的。

关闭TCP连接需要双方互相挥手完成, 经常出现的CLOSE_WAIT，FIN_WAIT,TIME_WAIT状态就是挥手未结束的中间状态。



PoolingHttpClientConnectionManager管理的是客户端的http连接池。
服务端主动关闭连接时，FIN到client, client 本地的TCP协议栈收到FIN并ACK，但是上层应用程序只有在Read呈现-1时，才回知道server不再发送数据而主动关闭了连接， 然后client调用close关闭连接。


### 连接池
PoolingHttpClientConnectionManager中的连接放回连接池和创建连接方法：
releaseConnection
connect


释放连接的时机：
//CloseableHttpResponse；
InputStream in = response.getEntity().getContent();

中的in实际上是
org.apache.http.client.entity.LazyDecompressingInputStream  
其包装了org.apache.http.conn.EofSensorInputStream
其中有watcher org.apache.http.conn.EofSensorWatcher


```java
//class EofSensorInputStream

 @Override
    public void close() throws IOException {
        // tolerate multiple calls to close()
        selfClosed = true;
        checkClose();
    }

 protected void checkClose() throws IOException {

if (wrappedStream != null) {
    try {
        boolean scws = true; // should close wrapped stream?
        if (eofWatcher != null) {
            scws = eofWatcher.streamClosed(wrappedStream);  //call releaseConnection finally
        }
        if (scws) {
            wrappedStream.close();
        }
    } finally {
        wrappedStream = null;
    }
}
}
```




### 附，基于httpClient4.5.2,httpCore4.4.4的HttpClient工具类

```java

public  class HttpClientUtil {
	
	private static final Logger LOGGER = LoggerFactory.getLogger(HttpClientUtil.class);
	
	private static CloseableHttpClient httpclient = createHttpClientDefault();

	private static final String DEFAULT_CHARSET_UTF8 = "UTF-8";
	private static final String DEFAULT_CONTENT_TYPE_JSON = "application/json";
	private static final int MAX_TIMEOUT = 10000;
	private static final int MAX_RETRY_TIMES = 10;
	private static final int MAX_THREAD_TOTAL = 50;


	/**
	 * 发送http post请求
	 * @param action
	 * @param bodyParam
	 * @return
	 * @throws Exception
	 */
	public static  String post(String action, Object bodyParam) throws Exception{
		return post(action, null, bodyParam, null, null);
	}
	
	
	/**
	 * 发送http post请求
	 * @param action
	 * @param bodyParam
	 * @return
	 * @throws Exception
	 */
	public static String post(String action, Map<String, String> headerParam, Object bodyParam) throws Exception{
		return post(action, headerParam, bodyParam, null, null);
	}
	
	/**
	 * 发送http post请求
	 * 
	 * @param action
	 * @return
	 * @throws Exception 
	 * @throws UnsupportedEncodingException
	 */
	public static String post(String action, Map<String, String> headerParam, Object bodyParam, String contentType, String charSet) throws Exception{
		
		
		String content_type = contentType;
		if (content_type == null || "".equals(content_type)) content_type = DEFAULT_CONTENT_TYPE_JSON;
		
		String char_set = charSet;
		if (char_set == null || "".equals(char_set)) char_set = DEFAULT_CHARSET_UTF8;		
		
		String url = action;
		LOGGER.info("Post请求地址：" + url);
		
		HttpPost httpPost = new HttpPost(url);
		
		//header参数
		if (headerParam != null && headerParam.size() >0){
			LOGGER.info("Post请求Header：" + JSON.toJSONString(headerParam));
			for (String key : headerParam.keySet()){
				httpPost.addHeader(key, headerParam.get(key));
			}
		}			
		
		//entity数据
		if (bodyParam != null ) {
			LOGGER.info("Post请求Body：" + JSON.toJSONString(bodyParam));
			StringEntity entity = new StringEntity(JSON.toJSONString(bodyParam),char_set);	
			entity.setContentEncoding(char_set);
			entity.setContentType(content_type);
			httpPost.setEntity(entity);		
		}
		
		String resultStr = "";	
		CloseableHttpResponse response = null;
		try {
			response = httpclient.execute(httpPost);
			processCertainStatus(response.getStatusLine().getStatusCode());
			resultStr = EntityUtils.toString(response.getEntity(), "utf-8");
			httpPost.reset();
		} catch (IOException e) {
			LOGGER.error("execute http get connection", e);
		} finally {
			try {
				if(response != null)
					response.close();
			} catch (IOException e) {
				LOGGER.error("close http get connection", e);
			}
		}
		LOGGER.info("Post请求返回：" + resultStr);
		return resultStr;
	}



	/**
	 * 发送http get请求
	 * @param action
	 * @return
	 * @throws Exception
	 */
	public static String get(String action) throws Exception{
		

		String url =  action;
		LOGGER.info("Get请求地址：" + url);
		HttpGet httpGet = new HttpGet(url);
		
		httpGet.addHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");		

		CloseableHttpResponse response = null;
		String resultStr = "";
		try {
			response = httpclient.execute(httpGet);
			processCertainStatus(response.getStatusLine().getStatusCode());
			resultStr = EntityUtils.toString(response.getEntity(),"utf-8");
			httpGet.reset();
		} catch (IOException e) {
			LOGGER.error("execute http get connection", e);
		} finally {
			try {
				if(response != null)
					response.close();
			} catch (IOException e) {
				LOGGER.error("close http get connection", e);
			}
		}

		LOGGER.info("Get请求返回：" + resultStr);
		return resultStr;
	}
	
	
	private static  void processCertainStatus(int statusCode){
		if(statusCode == 401){
			throw new TokenInvalidException("401 token invalid!");
		}
	}

	
	/**
	 * 发送http get请求
	 * @param action
	 * @return
	 * @throws Exception
	 */
	public static String get(String action, Map<String,String> params) throws Exception{
		
		URIBuilder uriBuilder = new URIBuilder();
		uriBuilder.setPath(action);
		if (params != null){
			for (String key: params.keySet()){
				uriBuilder.setParameter(key, params.get(key));
			}			
		}			
		return get(uriBuilder.build().toString());
	}
	
	
	/**
	 * 创建httpclient
	 * @return
	 */
	private static synchronized CloseableHttpClient createHttpClientDefault() {
		CloseableHttpClient httpclient = null;
		try {
			SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, 
					new TrustStrategy() {
						public boolean isTrusted(
								java.security.cert.X509Certificate[] chain,
								String authType)
								throws java.security.cert.CertificateException {							
							return true;
						}
					}).build();
		   SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, new NoopHostnameVerifier());
		   ConnectionSocketFactory psf = PlainConnectionSocketFactory.getSocketFactory();  
		   
		   Registry<ConnectionSocketFactory> registryBuilder = RegistryBuilder.<ConnectionSocketFactory>create()
		                .register("https", sslsf)
		                 .register("http", psf)
		                .build();

			RequestConfig config = RequestConfig.custom()
					  .setSocketTimeout(MAX_TIMEOUT)
					  .setConnectTimeout(MAX_TIMEOUT)
					  .setConnectionRequestTimeout(MAX_TIMEOUT)
					  .build();
			//超时重试处理			
			HttpRequestRetryHandler retryHandler = new DefaultHttpRequestRetryHandler(MAX_RETRY_TIMES, true);
			
			//连接管理池
			PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(registryBuilder);
			cm.setValidateAfterInactivity(60000);
			cm.setMaxTotal(MAX_THREAD_TOTAL);
			cm.setDefaultMaxPerRoute(MAX_THREAD_TOTAL);
			
			httpclient = HttpClients.custom().setConnectionManager(cm).setDefaultRequestConfig(config).setRetryHandler(retryHandler).build();
		
		} catch (KeyManagementException e) {
			LOGGER.error("KeyManagementException", e);
		} catch (NoSuchAlgorithmException e) {
			LOGGER.error("NoSuchAlgorithmException", e);
		} catch (KeyStoreException e) {
			LOGGER.error("KeyStoreException", e);
		}
		return httpclient;
	}
}
```