---
title: Flume 的 http source 源码学习
date: 2019-03-09 11:29:24
author: 陈锦华
password: 123
toc: true
categories: Flume 
tags:
  - Flume 
---



# Flume 的 http source 源码学习

## 一.Http source 

**类继承图**

```
public class HTTPSource extends SslContextAwareAbstractSource implements EventDrivenSource, Configurable
```

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/1570580489807.png)



- http source主要有 configure，start，stop 方法三个

- 通过 http post 和 get 接受 flume 事件的源。使用 get 仅用于实验，生产环境应该使用 post 方法

- http 请求可添加自定义的的“处理程序”，它必须实现 httpsourcehandler接口。这个处理程序需要传入httpservletrequest 参数并返回 flume 事件列表  List<Event>

- http source 接收参数有：port，handler，

  其中端口 port 应该绑定服务器

  handler 将 httpservletrequest 反序列化为 flume 事件列表，这个类必须实现 HttpSourceHandler，默认使用 JSONHandler，JSONHandler 是将 json 对象转换为 flume 事件

###  1.configure 方法

获取配置文件中的 port，host，HTTPSourceHandler

> 因继承 Configurable 类，有 public void configure(Context context) 方法
> 该方法用于重新实现自身配置的

```java
  @Override
  public void configure(Context context) {
    configureSsl(context);
    sourceContext = context;
    try {
      port = context.getInteger(HTTPSourceConfigurationConstants.CONFIG_PORT);
      host = context.getString(HTTPSourceConfigurationConstants.CONFIG_BIND,
          HTTPSourceConfigurationConstants.DEFAULT_BIND);

      Preconditions.checkState(host != null && !host.isEmpty(),
                "HTTPSource hostname specified is empty");
      Preconditions.checkNotNull(port, "HTTPSource requires a port number to be"
          + " specified");

      String handlerClassName = context.getString(
              HTTPSourceConfigurationConstants.CONFIG_HANDLER,
              HTTPSourceConfigurationConstants.DEFAULT_HANDLER).trim();

      @SuppressWarnings("unchecked")
      Class<? extends HTTPSourceHandler> clazz =(Class<? extends HTTPSourceHandler>)
              Class.forName(handlerClassName);
      handler = clazz.getDeclaredConstructor().newInstance();

      Map<String, String> subProps =context.getSubProperties(HTTPSourceConfigurationConstants.CONFIG_HANDLER_PREFIX);
      handler.configure(new Context(subProps));
    } catch (ClassNotFoundException ex) {
      LOG.error("Error while configuring HTTPSource. Exception follows.", ex);
      Throwables.propagate(ex);
    } catch (ClassCastException ex) {
      LOG.error("Deserializer is not an instance of HTTPSourceHandler. Deserializer must implement HTTPSourceHandler.");
      Throwables.propagate(ex);
    } catch (Exception ex) {
      LOG.error("Error configuring HTTPSource!", ex);
      Throwables.propagate(ex);
    }
    if (sourceCounter == null) {
      sourceCounter = new SourceCounter(getName());
    }
  }
```

### 2.start 方法

完成 httpserver 的启动

> LifeCycleAware接口
>
> 从代码层面我们可以看到，LifeCycleAware有三个方法分别是:
>
> 1. public void start()
> 2. public void stop()
> 3. public LifeCycleState getLifeCycleState()

```java
 @Override
  public void start() {
    Preconditions.checkState(srv == null,
            "Running HTTP Server found in source: " + getName()
            + " before I started one."
            + "Will not attempt to start.");
    QueuedThreadPool threadPool = new QueuedThreadPool();
    if (sourceContext.getSubProperties("QueuedThreadPool.").size() > 0) {
      FlumeBeanConfigurator.setConfigurationFields(threadPool, sourceContext);
    }
    srv = new Server(threadPool);

    //Register with JMX for advanced monitoring
    MBeanContainer mbContainer = new MBeanContainer(ManagementFactory.getPlatformMBeanServer());
    srv.addEventListener(mbContainer);
    srv.addBean(mbContainer);

    HttpConfiguration httpConfiguration = new HttpConfiguration();
    httpConfiguration.addCustomizer(new SecureRequestCustomizer());

    FlumeBeanConfigurator.setConfigurationFields(httpConfiguration, sourceContext);
    ServerConnector connector = getSslContextSupplier().get().map(sslContext -> {
      SslContextFactory sslCtxFactory = new SslContextFactory();
      sslCtxFactory.setSslContext(sslContext);
      sslCtxFactory.setExcludeProtocols(getExcludeProtocols().toArray(new String[]{}));
      sslCtxFactory.setIncludeProtocols(getIncludeProtocols().toArray(new String[]{}));
      sslCtxFactory.setExcludeCipherSuites(getExcludeCipherSuites().toArray(new String[]{}));
      sslCtxFactory.setIncludeCipherSuites(getIncludeCipherSuites().toArray(new String[]{}));

      FlumeBeanConfigurator.setConfigurationFields(sslCtxFactory, sourceContext);

      httpConfiguration.setSecurePort(port);
      httpConfiguration.setSecureScheme("https");

      return new ServerConnector(srv,
        new SslConnectionFactory(sslCtxFactory, HttpVersion.HTTP_1_1.asString()),
        new HttpConnectionFactory(httpConfiguration));
    }).orElse(
        new ServerConnector(srv, new HttpConnectionFactory(httpConfiguration))
    );

    connector.setPort(port);
    connector.setHost(host);
    connector.setReuseAddress(true);

    FlumeBeanConfigurator.setConfigurationFields(connector, sourceContext);

    srv.addConnector(connector);

    try {
      ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
      context.setContextPath("/");
      srv.setHandler(context);

      context.addServlet(new ServletHolder(new FlumeHTTPServlet()),"/");
      context.setSecurityHandler(HTTPServerConstraintUtil.enforceConstraints());
      srv.start();
    } catch (Exception ex) {
      LOG.error("Error while starting HTTPSource. Exception follows.", ex);
      Throwables.propagate(ex);
    }
    Preconditions.checkArgument(srv.isRunning());
    sourceCounter.start();
    super.start();
  }

```



### 3..stop 方法

stop 方法也是来自 LifeCycleAware 接口，主要作用是停止一个 jetty 服务

```java
public void stop() {
    try {
      srv.stop();
      srv.join();
      srv = null;
    } catch (Exception ex) {
      LOG.error("Error while stopping HTTPSource. Exception follows.", ex);
    }
    sourceCounter.stop();
    LOG.info("Http source {} stopped. Metrics: {}", getName(), sourceCounter);
  }
```



## 二.httpsource 的 FlumeHTTPServlet 拦截器

> start 方法中加入了 context.addServlet(new ServletHolder(new FlumeHTTPServlet()),"/");
>
> 因此所有 http 请求都会经过 FlumeHTTPServlet 的处理

FlumeHTTPServlet 主要方法是 doPost

```java
@Override
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
      List<Event> events = Collections.emptyList(); //create empty list
      try {
        events = handler.getEvents(request);
      } catch (HTTPBadRequestException ex) {
        LOG.warn("Received bad request from client. ", ex);
        sourceCounter.incrementEventReadFail();
        response.sendError(HttpServletResponse.SC_BAD_REQUEST,
                "Bad request from client. "
                + ex.getMessage());
        return;
      } catch (Exception ex) {
        LOG.warn("Deserializer threw unexpected exception. ", ex);
        sourceCounter.incrementEventReadFail();
        response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
                "Deserializer threw unexpected exception. "
                + ex.getMessage());
        return;
      }
      sourceCounter.incrementAppendBatchReceivedCount();
      sourceCounter.addToEventReceivedCount(events.size());
      try {
        getChannelProcessor().processEventBatch(events);
      } catch (ChannelException ex) {
        LOG.warn("Error appending event to channel. "
                + "Channel might be full. Consider increasing the channel "
                + "capacity or make sure the sinks perform faster.", ex);
        sourceCounter.incrementChannelWriteFail();
        response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                "Error appending event to channel. Channel might be full."
                + ex.getMessage());
        return;
      } catch (Exception ex) {
        LOG.warn("Unexpected error appending event to channel. ", ex);
        sourceCounter.incrementGenericProcessingFail();
        response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
                "Unexpected error while appending event to channel. "
                + ex.getMessage());
        return;
      }
      response.setCharacterEncoding(request.getCharacterEncoding());
      response.setStatus(HttpServletResponse.SC_OK);
      response.flushBuffer();
      sourceCounter.incrementAppendBatchAcceptedCount();
      sourceCounter.addToEventAcceptedCount(events.size());
    }
```

## 三.默认的处理器 JSONHandler

>  http source中的 start 方法中有这么一句。
>  String handlerClassName = context.getString(        HTTPSourceConfigurationConstants.CONFIG_HANDLER,        HTTPSourceConfigurationConstants.DEFAULT_HANDLER).trim();
>
> 其中HTTPSourceConfigurationConstants DEFAULT_HANDLER 就是      "org.apache.flume.source.http.JSONHandler";
>
>  JSONHandler 将 httpservletrequest 反序列化为 flume 事件列表，这个类必须实现 HttpSourceHandler，

### 1.HTTPSourceHandler

```
public interface HTTPSourceHandler extends Configurable {
  public List<Event> getEvents(HttpServletRequest request) throws
          HTTPBadRequestException, Exception;

}
```

- HTTPSourceHandler 获取 httpservletrequest 并返回flume列表事件。如果此请求无法根据格式化此方法将引发异常。此方法还可以抛出如果有其他错误，则出现异常
- 参数 request 将会解析为 Flume 事件
- 如果由于请求的格式不符合预期格式而未正确解析为事件 抛出HTTPBadRequestException

### 2.HTTPBadRequestException 异常

```java
public class HTTPBadRequestException extends FlumeException {

  private static final long serialVersionUID = -3540764742069390951L;
    
  public HTTPBadRequestException(String msg) {
    super(msg);
  }
  public HTTPBadRequestException(String msg, Throwable th) {
    super(msg, th);
  }

  public HTTPBadRequestException(Throwable th) {
    super(th);
  }
}
```

### 3. JSONHandler

```java
package org.apache.flume.source.http;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonSyntaxException;
import com.google.gson.reflect.TypeToken;
import java.io.BufferedReader;
import java.lang.reflect.Type;
import java.nio.charset.UnsupportedCharsetException;
import java.util.ArrayList;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.event.EventBuilder;
import org.apache.flume.event.JSONEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * JSONHandler for HTTPSource that accepts an array of events.
 * JSONHandler接收的是HTTPSource的events数组
 *
 * This handler throws exception if the deserialization fails because of bad
 * format or any other reason.
 * 如果反序列化由于格式错误或任何其他原因失败，此处理程序将引发异常
 *
 *
 * Each event must be encoded as a map with two key-value pairs.
 * 每个事件都必须编码为具有两个键值对的映射
 * <p> 1. headers - the key for this key-value pair is "headers". 这个键值对的键是“headers”
 *
 * The value for this key is another map, which represent the event headers.这个键的值是另一个映射，它表示事件头
 * These headers are inserted into the Flume event as is.这些headers按原样插入到flume事件中
 *
 * <p> 2.body - The body is a string which represents the body of the event. body 是表示事件主体的字符串
 * The key for this key-value pair is "body".这个键值对的键是“body”
 *
 * All key-value pairs are considered to be headers.所有键值对都被视为headers
 * An example: 例如：
 *[{"headers": {"a":"b", "c":"d"},"body": "random_body"}, {"headers" : {"e": "f"},"body":
 * "random_body2"}]
 *
 *  would be interpreted as the following two flume events
 *  将被解释为以下两个flume事件
 * <p> * Event with body: "random_body" (in UTF-8/UTF-16/UTF-32 encoded bytes) and headers : (a:b, c:d) <p> *
 * 事件的 body：{随机体}  UTF-8/UTF-16/UTF-32编码是字节码 headers: (a:b, c:d)
 *
 * Event with body: "random_body2" (in UTF-8/UTF-16/UTF-32 encoded bytes) and headers : (e:f) <p>
 * 事件的 body：{随机体2}  UTF-8/UTF-16/UTF-32编码是字节码 headers: (e:f)
 *
 * The charset of the body is read from the request and used. If no charset is
 * set in the request, then the charset is assumed to be JSON's default - UTF-8.
 * The JSON handler supports UTF-8, UTF-16 and UTF-32.
 * body的字符集从请求中读取并使用。如果没有字符集在请求中设置，则假定字符集是json的默认值-utf-8。
 *json处理程序支持utf-8、utf-16和utf-32
 *
 * To set the charset, the request must have content type specified as
 * "application/json; charset=UTF-8" (replace UTF-8 with UTF-16 or UTF-32 as
 * required).
 *
 * 若要设置字符集，请求必须将内容类型指定为“application/json；charset=utf-8”（将utf-8替换为utf-16或utf-32必须）
 *
 * One way to create an event in the format expected by this handler, is to
 * use {@linkplain JSONEvent} and use {@linkplain Gson} to create the JSON
 * string using the
 * {@linkplain Gson#toJson(java.lang.Object, java.lang.reflect.Type) }
 * method. The type token to pass as the 2nd argument of this method
 * for list of events can be created by: <p>
 * {@code
 * Type type = new TypeToken<List<JSONEvent>>() {}.getType();
 * }
 * 使用{jsonevent}和{gson}使用该方法创建json字符串
 */

public class JSONHandler implements HTTPSourceHandler {

  private static final Logger LOG = LoggerFactory.getLogger(JSONHandler.class);
  private final Type listType = new TypeToken<List<JSONEvent>>() {}.getType();
  private final Gson gson;

  public JSONHandler() {
    gson = new GsonBuilder().disableHtmlEscaping().create();
  }

  /**
   * {@inheritDoc}
   */
  @Override
  public List<Event> getEvents(HttpServletRequest request) throws Exception {
    BufferedReader reader = request.getReader();
    String charset = request.getCharacterEncoding();
    //UTF-8 is default for JSON. If no charset is specified, UTF-8 is to be assumed.
    //utf-8是json的默认值。如果未指定字符集，则假定为utf-8
    if (charset == null) {
      LOG.debug("Charset is null, default charset of UTF-8 will be used.");
      charset = "UTF-8";
    } else if (!(charset.equalsIgnoreCase("utf-8")
            || charset.equalsIgnoreCase("utf-16")
            || charset.equalsIgnoreCase("utf-32"))) {
      LOG.error("Unsupported character set in request {}. "
              + "JSON handler supports UTF-8, "
              + "UTF-16 and UTF-32 only.", charset);
      throw new UnsupportedCharsetException("JSON handler supports UTF-8, "
              + "UTF-16 and UTF-32 only.");
    }

    /*
     * Gson throws Exception if the data is not parseable to JSON.
     * Need not catch it since the source will catch it and return error.
     */
    List<Event> eventList = new ArrayList<Event>(0);
    try {
      eventList = gson.fromJson(reader, listType);
    } catch (JsonSyntaxException ex) {
      throw new HTTPBadRequestException("Request has invalid JSON Syntax.", ex);
    }

    for (Event e : eventList) {
      ((JSONEvent) e).setCharset(charset);
    }
    return getSimpleEvents(eventList);
  }

  @Override
  public void configure(Context context) {
  }

  private List<Event> getSimpleEvents(List<Event> events) {
    List<Event> newEvents = new ArrayList<Event>(events.size());
    for (Event e:events) {
      newEvents.add(EventBuilder.withBody(e.getBody(), e.getHeaders()));
    }
    return newEvents;
  }
}

```

