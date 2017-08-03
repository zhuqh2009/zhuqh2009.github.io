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
