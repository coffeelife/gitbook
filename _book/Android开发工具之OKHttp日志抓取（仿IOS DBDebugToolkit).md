# Android开发工具之OKHttp日志抓取（仿IOS DBDebugToolkit)

## 声明

本文章基于OkHttp基础进行开发，主要是仿IOS调试工具DBDebugToolkit的网络日志抓取功能，原理十分简单，对OkHttp设置拦截器，在拦截器中拦截请求数据和返回数据，然后存储到本地文件中，最后再从本地文件中读取相应的信息展示，方便开发人员和测试人员进行网络接口调试。

### IOS DBDebugToolkit调试工具

IOS DBDebugToolkit git地址：[https://github.com/dbukowski/DBDebugToolkit](https://github.com/dbukowski/DBDebugToolkit)

本文github地址：[https://github.com/coffeelife/HttpLogCode.git](https://github.com/coffeelife/HttpLogCode.git)

截图 （调试页面）

![7Ir6u85PhABlsTc](https://i.loli.net/2019/08/28/7Ir6u85PhABlsTc.jpg)

截图 （详情页面）

![WzOCgZh26xNBlKu](https://i.loli.net/2019/08/29/WzOCgZh26xNBlKu.jpg)

![Gqe9d3azyJswkVf](https://i.loli.net/2019/08/29/Gqe9d3azyJswkVf.jpg)

![NwoPQ5vhWq61RcZ](https://i.loli.net/2019/08/29/NwoPQ5vhWq61RcZ.jpg)

截图结束

## 开篇

截图结束，以上截图就是我们想要实现的功能，这是IOS的调试工具，Android并没有类似的开源工具，所以我们来实现一个类似的这样工具，方便测试或者后端甚至自己来查看网络请求情况。(Ps:避免测试或者开发过程中，总有人找你，说这里我返回的数据不是这样的，测试来找你说这个状态不应该这么显示，你就可以直接告诉他们，你看看请求日志，看看数据是怎么样的）。

## 原理

最开始也讲了，这个东西并不复杂，本项目是通过OkHttp作为请求基础，OkHttp可以设置拦截器，这个拦截器就是实现该功能的方式。原理：我们通过给我们App的OkHttp请求设置拦截器，拦截发出去的请求和返回的请求数据，将结果打印在log里面，方便开发过程中调试，并将结果保存在一个bean对象中，保存到文件当中（我为了方便保存在了sp中），然后写一个列表页面将自己保存的网络请求数据list显示到页面中。下面上代码，最后上截图。

## 代码编写

### 建立网络拦截器

`HttpLogInterceptor.java`（全部）

```
/**
 * Created by ccx on 2018/07/11
 */
public class HttpLogInterceptor implements Interceptor {
    private static final Charset UTF8 = Charset.forName("UTF-8");

    public enum Level {
        /**
         * No logs.
         */
        NONE,
        /**
         * Logs request and response lines.
         *
         * <p>Example:
         * <pre>{@code
         * --> POST /greeting http/1.1 (3-byte body)
         *
         * <-- 200 OK (22ms, 6-byte body)
         * }</pre>
         */
        BASIC,
        /**
         * Logs request and response lines and their respective headers.
         *
         * <p>Example:
         * <pre>{@code
         * --> POST /greeting http/1.1
         * Host: example.com
         * Content-Type: plain/text
         * Content-Length: 3
         * --> END POST
         *
         * <-- 200 OK (22ms)
         * Content-Type: plain/text
         * Content-Length: 6
         * <-- END HTTP
         * }</pre>
         */
        HEADERS,
        /**
         * Logs request and response lines and their respective headers and bodies (if present).
         *
         * <p>Example:
         * <pre>{@code
         * --> POST /greeting http/1.1
         * Host: example.com
         * Content-Type: plain/text
         * Content-Length: 3
         *
         * Hi?
         * --> END POST
         *
         * <-- 200 OK (22ms)
         * Content-Type: plain/text
         * Content-Length: 6
         *
         * Hello!
         * <-- END HTTP
         * }</pre>
         */
        BODY
    }

    public interface Logger {
        void log(String message);

        void log(String tag, String message);

        void json(String tag, String message);

        /**
         * A {@link HttpLogInterceptor.Logger} defaults output appropriate for the current platform.
         */
        HttpLogInterceptor.Logger DEFAULT = new HttpLogInterceptor.Logger() {
            @Override
            public void log(String message) {
                com.gjhealth.library.utils.log.Logger.t("http").d(message);
            }

            @Override
            public void log(String tag, String message) {
                com.gjhealth.library.utils.log.Logger.t("http" + tag).d(message);
            }

            @Override
            public void json(String tag, String message) {
                com.gjhealth.library.utils.log.Logger.t(tag).json(message);
            }
        };
    }

    public HttpLogInterceptor() {
        this(HttpLogInterceptor.Logger.DEFAULT);
    }

    public HttpLogInterceptor(HttpLogInterceptor.Logger logger) {
        this.logger = logger;
    }

    private final HttpLogInterceptor.Logger logger;

    private volatile HttpLogInterceptor.Level level = HttpLogInterceptor.Level.NONE;

    /**
     * Change the level at which this interceptor logs.
     */
    public HttpLogInterceptor setLevel(HttpLogInterceptor.Level level) {
        if (level == null) throw new NullPointerException("level == null. Use Level.NONE instead.");
        this.level = level;
        return this;
    }

    public HttpLogInterceptor.Level getLevel() {
        return level;
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        HttpLogInterceptor.Level level = this.level;
        long startNs = System.nanoTime();

        Request request = chain.request();
        HttpCacheBean bean = new HttpCacheBean();
        bean.requestTime = DateFormatUtils.getCurrentDateTime();
        if (level == HttpLogInterceptor.Level.NONE) {
            return chain.proceed(request);
        }

        boolean logBody = level == HttpLogInterceptor.Level.BODY;
        boolean logHeaders = logBody || level == HttpLogInterceptor.Level.HEADERS;

        RequestBody requestBody = request.body();
        boolean hasRequestBody = requestBody != null;

        Connection connection = chain.connection();
        Protocol protocol = connection != null ? connection.protocol() : Protocol.HTTP_1_1;
        String requestStartMessage = "--> " + request.method() + ' ' + request.url() + ' ' + protocol;
        bean.method = request.method();
        bean.url = request.url().toString();
        bean.protocol = protocol + "";
        if (!logHeaders && hasRequestBody) {
            requestStartMessage += " (" + requestBody.contentLength() + "-byte body)";
            bean.reqContentLength = requestStartMessage;
        }
        logger.log(requestStartMessage);
        bean.requestMsg = requestStartMessage;
        if (logHeaders) {
//            if (hasRequestBody) {
//                // Request body headers are only present when installed as a network interceptor. Force
//                // them to be included (when available) so there values are known.
//                if (requestBody.contentType() != null) {
//                    logger.log("Content-Type: " + requestBody.contentType());
//                }
//                if (requestBody.contentLength() != -1) {
//                    logger.log("Content-Length: " + requestBody.contentLength());
//                }
//            }

            Headers headers = request.headers();
//            StringBuffer sbReq = new StringBuffer();
            if (headers.size() > 0) {
                JsonObject jsonHeaders = new JsonObject();
                for (int i = 0, count = headers.size(); i < count; i++) {
                    String value = headers.value(i);
                    if (value != null) {
                        value = URLDecoder.decode(value, "UTF-8");
                    }
                    jsonHeaders.addProperty(headers.name(i), value);
//                    sbReq.append(headers.name(i) + ":" + value + "\n");
                }
                logger.json("httphead-req", jsonHeaders.toString());
                bean.reqHeaders = jsonHeaders.toString();
            }

            if (!logBody || !hasRequestBody) {
//                logger.log("--> END " + request.method());
            } else if (bodyEncoded(request.headers())) {
//                logger.log("--> END " + request.method() + " (encoded body omitted)");
            } else {
                Buffer buffer = new Buffer();
                requestBody.writeTo(buffer);

                Charset charset = UTF8;
                MediaType contentType = requestBody.contentType();
                if (contentType != null) {
                    charset = contentType.charset(UTF8);
                }
                if (isPlaintext(buffer)) {
                    logger.json("httprequest", buffer.clone().readString(charset));
                    bean.requestBody = buffer.readString(charset);
                } else {
                    logger.log("--> END " + request.method() + " (binary "
                            + requestBody.contentLength() + "-byte body omitted)");
                }
                bean.reqContentType = contentType.toString();
            }
        }
        Response response;
        try {
            response = chain.proceed(request);
        } catch (Exception e) {
            logger.log("<-- HTTP FAILED: " + e);
            bean.error = e.toString();
            throw e;
        }
        long tookMs = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startNs);

        ResponseBody responseBody = response.body();
        long contentLength = responseBody.contentLength();
        String bodySize = contentLength != -1 ? contentLength + "-byte" : "unknown-length";
        logger.log("<-- " + response.code() + ' ' + response.message() + ' '
                + response.request().url() + " (" + tookMs + "ms" + (!logHeaders ? ", "
                + bodySize + " body" : "") + ')');
        bean.responseMsg = "<-- " + response.code() + ' ' + response.message() + ' '
                + response.request().url() + " (" + tookMs + "ms" + (!logHeaders ? ", "
                + bodySize + " body" : "") + ')';
        bean.resContentLength = bodySize;
        bean.code = response.code();
        bean.msg = response.message();
        bean.time = tookMs + "ms";

        if (logHeaders) {
//            StringBuffer sbRes = new StringBuffer();
            Headers headers = response.headers();
            JsonObject jsonHeaders = new JsonObject();
            for (int i = 0, count = headers.size(); i < count; i++) {
                String value = headers.value(i);
                if (value != null) {
                    value = URLDecoder.decode(value, "UTF-8");
                }
                jsonHeaders.addProperty(headers.name(i), value);
//                sbRes.append(headers.name(i) + ":" + value + "\n");
//                logger.log(headers.name(i) + ": " + headers.value(i));

            }
            logger.json("httphead-rsp", jsonHeaders.toString());
            bean.resHeaders = jsonHeaders.toString();

            if (!logBody || !HttpHeaders.hasBody(response)) {

            } else if (bodyEncoded(response.headers())) {
                logger.log("<-- END HTTP (encoded body omitted)");
            } else if (isPlaintext(responseBody.contentType())) {
                BufferedSource source = responseBody.source();
                source.request(Long.MAX_VALUE); // Buffer the entire body.
                Buffer buffer = source.buffer();

                Charset charset = UTF8;
                MediaType contentType = responseBody.contentType();
                if (contentType != null) {
                    try {
                        charset = contentType.charset(UTF8);
                    } catch (UnsupportedCharsetException e) {
                        logger.log("");
                        logger.log("Couldn't decode the response body; charset is likely malformed.");
                        logger.log("<-- END HTTP");

                        return response;
                    }
                }
                bean.resContentType = contentType.toString();

//                if (!isPlaintext(buffer)) {
//                    logger.log("<-- END HTTP (binary " + buffer.size() + "-byte body omitted)");
//                    return response;
//                }

                if (contentLength != 0) {
//                    logger.log("");
                    logger.log("body", buffer.clone().readString(charset));
                    bean.responseBody = buffer.clone().readString(charset);
                }

                logger.log("<-- END HTTP (" + buffer.size() + "-byte body)");
            }
        }

        HttpLogManager.saveLogBean(bean);
        return response;
    }

    private static boolean isPlaintext(MediaType mediaType) {
        if (mediaType == null) return false;
        if (mediaType.type() != null && mediaType.type().equals("text")) {
            return true;
        }
        String subtype = mediaType.subtype();
        if (subtype != null) {
            subtype = subtype.toLowerCase();
            if (subtype.contains("x-www-form-urlencoded") || subtype.contains("json") || subtype.contains("xml") || subtype.contains("html")) //
                return true;
        }
        return false;
    }

    /**
     * Returns true if the body in question probably contains human readable text. Uses a small sample
     * of code points to detect unicode control characters commonly used in binary file signatures.
     */
    static boolean isPlaintext(Buffer buffer) {
        try {
            Buffer prefix = new Buffer();
            long byteCount = buffer.size() < 64 ? buffer.size() : 64;
            buffer.copyTo(prefix, 0, byteCount);
            for (int i = 0; i < 16; i++) {
                if (prefix.exhausted()) {
                    break;
                }
                int codePoint = prefix.readUtf8CodePoint();
                if (Character.isISOControl(codePoint) && !Character.isWhitespace(codePoint)) {
                    return false;
                }
            }
            return true;
        } catch (EOFException e) {
            return false; // Truncated UTF-8 sequence.
        }
    }

    private boolean bodyEncoded(Headers headers) {
        String contentEncoding = headers.get("Content-Encoding");
        return contentEncoding != null && !contentEncoding.equalsIgnoreCase("identity");
    }
}
```

内部枚举类Level

```
public enum Level {
 /**
 * No logs.
 */
 NONE,
 /**
 * Logs request and response lines.
 *
 * <p>Example:
 * <pre>{@code
 * --> POST /greeting http/1.1 (3-byte body)
 *
 * <-- 200 OK (22ms, 6-byte body)
 * }</pre>
 */
 BASIC,
 /**
 * Logs request and response lines and their respective headers.
 *
 * <p>Example:
 * <pre>{@code
 * --> POST /greeting http/1.1
 * Host: example.com
 * Content-Type: plain/text
 * Content-Length: 3
 * --> END POST
 *
 * <-- 200 OK (22ms)
 * Content-Type: plain/text
 * Content-Length: 6
 * <-- END HTTP
 * }</pre>
 */
 HEADERS,
 /**
 * Logs request and response lines and their respective headers and bodies (if present).
 *
 * <p>Example:
 * <pre>{@code
 * --> POST /greeting http/1.1
 * Host: example.com
 * Content-Type: plain/text
 * Content-Length: 3
 *
 * Hi?
 * --> END POST
 *
 * <-- 200 OK (22ms)
 * Content-Type: plain/text
 * Content-Length: 6
 *
 * Hello!
 * <-- END HTTP
 * }</pre>
 */
 BODY
 }
 //拦截代码内部片段 Level为NONE，不进行日志打印
if (level == HttpLogInterceptor.Level.NONE) {
 return chain.proceed(request);
 }
```

代码中有一个枚举类型`Level`，这个类用来标注log和网络请求记录类型，用来区分打包类型对应不同类型的Log打印和是否记录网络请求日志，当打debug和beta包时，可以打印log和记录网络请求，当打release包时，因为是线上，就将log和网络请求日志功能关闭。

在Okhttp中添加拦截器HttpLogInterceptor

```
 OkHttpClient.Builder builder = new OkHttpClient.Builder();
        //log相关
        HttpLogInterceptor loggingInterceptor = new HttpLogInterceptor().setLevel(
                !GlobalEnv.isRelease() ? HttpLogInterceptor.Level.BODY : HttpLogInterceptor.Level.NONE);
                               //添加OkGo默认debug日志
        builder.addInterceptor(loggingInterceptor);
```

代码讲解:新建okhttp builder，然后新建HttpLogInterceptor拦截器，判断app包类型，如果是release包就关闭网络请求日志功能，最后给OkHttp添加拦截器

核心代码部分（日志收集）

```
@Override
 public Response intercept(Chain chain) throws IOException {
 HttpLogInterceptor.Level level = this.level;
 long startNs = System.nanoTime();
 Request request = chain.request();
 HttpCacheBean bean = new HttpCacheBean();
 bean.requestTime = DateFormatUtils.getCurrentDateTime();
 if (level == HttpLogInterceptor.Level.NONE) {
 return chain.proceed(request);
 }
 boolean logBody = level == HttpLogInterceptor.Level.BODY;
 boolean logHeaders = logBody || level == HttpLogInterceptor.Level.HEADERS;
 RequestBody requestBody = request.body();
 boolean hasRequestBody = requestBody != null;
 Connection connection = chain.connection();
 Protocol protocol = connection != null ? connection.protocol() : Protocol.HTTP_1_1;
 String requestStartMessage = "--> " + request.method() + ' ' + request.url() + ' ' + protocol;
 bean.method = request.method();
 bean.url = request.url().toString();
 bean.protocol = protocol + "";
 if (!logHeaders && hasRequestBody) {
 requestStartMessage += " (" + requestBody.contentLength() + "-byte body)";
 bean.reqContentLength = requestStartMessage;
 }
 logger.log(requestStartMessage);
 bean.requestMsg = requestStartMessage;
 if (logHeaders) {
// if (hasRequestBody) {
// // Request body headers are only present when installed as a network interceptor. Force
// // them to be included (when available) so there values are known.
// if (requestBody.contentType() != null) {
// logger.log("Content-Type: " + requestBody.contentType());
// }
// if (requestBody.contentLength() != -1) {
// logger.log("Content-Length: " + requestBody.contentLength());
// }
// }
 Headers headers = request.headers();
// StringBuffer sbReq = new StringBuffer();
 if (headers.size() > 0) {
 JsonObject jsonHeaders = new JsonObject();
 for (int i = 0, count = headers.size(); i < count; i++) {
 String value = headers.value(i);
 if (value != null) {
 value = URLDecoder.decode(value, "UTF-8");
 }
 jsonHeaders.addProperty(headers.name(i), value);
// sbReq.append(headers.name(i) + ":" + value + "\n");
 }
 logger.json("httphead-req", jsonHeaders.toString());
 bean.reqHeaders = jsonHeaders.toString();
 }
 if (!logBody || !hasRequestBody) {
// logger.log("--> END " + request.method());
 } else if (bodyEncoded(request.headers())) {
// logger.log("--> END " + request.method() + " (encoded body omitted)");
 } else {
 Buffer buffer = new Buffer();
 requestBody.writeTo(buffer);
 Charset charset = UTF8;
 MediaType contentType = requestBody.contentType();
 if (contentType != null) {
 charset = contentType.charset(UTF8);
 }
 if (isPlaintext(buffer)) {
 logger.json("httprequest", buffer.clone().readString(charset));
 bean.requestBody = buffer.readString(charset);
 } else {
 logger.log("--> END " + request.method() + " (binary "
 + requestBody.contentLength() + "-byte body omitted)");
 }
 bean.reqContentType = contentType.toString();
 }
 }
 Response response;
 try {
 response = chain.proceed(request);
 } catch (Exception e) {
 logger.log("<-- HTTP FAILED: " + e);
 bean.error = e.toString();
 throw e;
 }
 long tookMs = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startNs);
 ResponseBody responseBody = response.body();
 long contentLength = responseBody.contentLength();
 String bodySize = contentLength != -1 ? contentLength + "-byte" : "unknown-length";
 logger.log("<-- " + response.code() + ' ' + response.message() + ' '
 + response.request().url() + " (" + tookMs + "ms" + (!logHeaders ? ", "
 + bodySize + " body" : "") + ')');
 bean.responseMsg = "<-- " + response.code() + ' ' + response.message() + ' '
 + response.request().url() + " (" + tookMs + "ms" + (!logHeaders ? ", "
 + bodySize + " body" : "") + ')';
 bean.resContentLength = bodySize;
 bean.code = response.code();
 bean.msg = response.message();
 bean.time = tookMs + "ms";
 if (logHeaders) {
// StringBuffer sbRes = new StringBuffer();
 Headers headers = response.headers();
 JsonObject jsonHeaders = new JsonObject();
 for (int i = 0, count = headers.size(); i < count; i++) {
 String value = headers.value(i);
 if (value != null) {
 value = URLDecoder.decode(value, "UTF-8");
 }
 jsonHeaders.addProperty(headers.name(i), value);
// sbRes.append(headers.name(i) + ":" + value + "\n");
// logger.log(headers.name(i) + ": " + headers.value(i));
 }
 logger.json("httphead-rsp", jsonHeaders.toString());
 bean.resHeaders = jsonHeaders.toString();
 if (!logBody || !HttpHeaders.hasBody(response)) {
 } else if (bodyEncoded(response.headers())) {
 logger.log("<-- END HTTP (encoded body omitted)");
 } else if (isPlaintext(responseBody.contentType())) {
 BufferedSource source = responseBody.source();
 source.request(Long.MAX_VALUE); // Buffer the entire body.
 Buffer buffer = source.buffer();
 Charset charset = UTF8;
 MediaType contentType = responseBody.contentType();
 if (contentType != null) {
 try {
 charset = contentType.charset(UTF8);
 } catch (UnsupportedCharsetException e) {
 logger.log("");
 logger.log("Couldn't decode the response body; charset is likely malformed.");
 logger.log("<-- END HTTP");
 return response;
 }
 }
 bean.resContentType = contentType.toString();
// if (!isPlaintext(buffer)) {
// logger.log("<-- END HTTP (binary " + buffer.size() + "-byte body omitted)");
// return response;
// }
 if (contentLength != 0) {
// logger.log("");
 logger.log("body", buffer.clone().readString(charset));
 bean.responseBody = buffer.clone().readString(charset);
 }
 logger.log("<-- END HTTP (" + buffer.size() + "-byte body)");
 }
 }
 HttpLogManager.saveLogBean(bean);
 return response;
 }
```

此部分代码是拦截器拦截到网络请求之后的对数据进行日志收集的代码，主要是新建一个Log日志Bean对象（HttpCacheBean)，通过Request request = chain.request();得到请求详情，并赋值给log日志bean 对象，主要包括请求url、发送请求时间time（通过得到当前时间）、请求头reqHeaders、请求body数据requestBody、请求类型method、请求协议protocol等信息。

返回数据主要通过response = chain.proceed(request);得到返回对象之后，依旧复制给log日志bean 对象，主要包括计算的请求时长requestTime、请求状态码code（200、400、401、500等）、请求错误error、返回内容长度reqContentLength、请求返回msg responseMsg、返回body数据responseBody等。

`HttpCacheBean.java`

```
public class HttpCacheBean implements Serializable {
    public static String TAG = HttpCacheBean.class.getSimpleName();
    public String url;
    public String requestTime;
    public int code;
    public String time;
    public String reqHeaders;
    public String resHeaders;
    public String requestBody;
    public String protocol;
    public String method;
    public String msg;
    public String responseBody;
    public String reqContentType;
    public String resContentType;
    public String error;
    public String reqContentLength;
    public String resContentLength;
    public String requestMsg;
    public String responseMsg;
}
```

最后，将bean 对象保存到sp中，我这里面写了一个manager类，来保存和读取该文件

```
HttpLogManager.saveLogBean(bean);
```

`HttpManager.java`

```
package cn.api.gjhealth.cstore.http;

import android.text.TextUtils;
import com.gjhealth.library.http.model.HttpCacheBean;
import com.gjhealth.library.utils.ArrayUtils;
import com.gjhealth.library.utils.SharedUtil;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLDecoder;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import cn.api.gjhealth.cstore.app.BaseApp;
import cn.api.gjhealth.cstore.utils.jsonutils.GsonUtil;

public class HttpLogManager {
    public static List<HttpCacheBean> getLogLists(String keyWords) {
        List<HttpCacheBean> listBeans = null;
        String json = SharedUtil.instance(BaseApp.getContext()).getString(HttpCacheBean.TAG);
        if (GsonUtil.isGoodJson(json)) listBeans = GsonUtil.getEntityList(json, HttpCacheBean.class);
        if (GsonUtil.isBadJson(json) || listBeans == null) listBeans = new ArrayList<>();
        if (TextUtils.isEmpty(keyWords)) {
            Collections.reverse(listBeans);
            return listBeans;
        } else {
            List<HttpCacheBean> listBeansTemp = new ArrayList<>();
            if (!ArrayUtils.isEmpty(listBeans)) {
                for (HttpCacheBean bean : listBeans) {
                    if (bean != null && !TextUtils.isEmpty(bean.url)) {
                        try {
                            URL url = new URL(URLDecoder.decode(bean.url));
                            if (url.getPath().contains(keyWords)) {
                                listBeansTemp.add(bean);
                            }
                        } catch (MalformedURLException e) {
                        }
                    }
                }
            }
            Collections.reverse(listBeansTemp);
            return listBeansTemp;
        }
    }

    public static void saveLogBean(HttpCacheBean bean) {
        List<HttpCacheBean> httpCacheBeans = null;
        String json = SharedUtil.instance(BaseApp.getContext()).getString(HttpCacheBean.TAG);
        if (GsonUtil.isGoodJson(json))
            httpCacheBeans = GsonUtil.getEntityList(json, HttpCacheBean.class);
        if (GsonUtil.isBadJson(json) || httpCacheBeans == null) httpCacheBeans = new ArrayList<>();
        httpCacheBeans.add(bean);
        if (!ArrayUtils.isEmpty(httpCacheBeans) && httpCacheBeans.size() > 100) {
            httpCacheBeans = httpCacheBeans.subList(httpCacheBeans.size() - 100, httpCacheBeans.size());
        }
        SharedUtil.instance(BaseApp.getContext()).saveObject(HttpCacheBean.TAG, httpCacheBeans);
    }
}
```

因为我是list显示的，而且我觉得很久之前的数据都是无用的，所以我在保存数据的时候都是取数据的最后的一百条来存储，然后recycleview显示的时候都是直接取所有数据显示。

getLogLists里面有个keywords参数，这个是方便用于搜索查询的关键字，传空默认返回全部数据。可以根据接口url来匹配查询接口

`GsonUtil.java`工具类扩展

```
package cn.api.gjhealth.cstore.utils.jsonutils;

import android.text.TextUtils;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonArray;
import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParseException;
import com.google.gson.JsonParser;

import java.io.Reader;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by lc on 2017/5/17.
 * Gson解析工具类
 */

public class GsonUtil {
    // 是否json数据
    public static boolean isGoodJson(String json) {
        if (TextUtils.isEmpty(json)) {
            return false;
        }
        try {
            new JsonParser().parse(json);
            return true;
        } catch (JsonParseException e) {
            return false;
        }
    }

    public static boolean isBadJson(String json) {
        return !isGoodJson(json);
    }

    // 是否存在指定key
    public static boolean fieldIsNull(JsonObject command, String methodName) {

        return GsonUtil.getKeyValue(command, methodName) == null;
    }

    /**
     * 解析单个
     *
     * @param jsonStr
     * @param entityClass
     * @param <T>
     * @return
     */
    public static <T> T getEntity(String jsonStr, Class<T> entityClass) {
        Gson gson = new Gson();
        T entity = null;
        entity = gson.fromJson(jsonStr, entityClass);
        return entity;
    }

    /**
     * 解析list集合
     *
     * @param jsonStr
     * @param entityClass
     * @param <T>
     * @return
     */
    public static <T> List<T> getEntityList(String jsonStr, Class<T> entityClass) {
        Gson gson = new Gson();
        List<T> list = new ArrayList<T>();
        JsonArray jsonArray = new JsonParser().parse(jsonStr).getAsJsonArray();
        for (JsonElement element : jsonArray) {
            list.add(gson.fromJson(element, entityClass));
        }
        return list;
    }

    // Gson解析器
    public static Gson gson = new GsonBuilder()
            .enableComplexMapKeySerialization()
            .setExclusionStrategies(new FooAnnotationExclusionStrategy())//过滤不上传的字段值
            .serializeNulls()// 支持Map的key为复杂对象的形式
            .setDateFormat("yyyy-MM-dd HH:mm:ss") // 时间转化为特定格式
            .setPrettyPrinting().create();

    /****************** json拆分 ***************************************/

    /**
     * 从父JsonObject对象中获取子JsonObject对象
     *
     * @param obj
     * @param key
     * @return
     */
    public static JsonObject getJsonObject(JsonObject obj, String key) {
        JsonElement element = obj.get(key);
        return element.getAsJsonObject();
    }

    /**
     * 从父JsonObject对象中获取子JsonArray对象
     *
     * @param obj
     * @param key
     * @return
     */
    public static JsonArray getJsonArray(JsonObject obj, String key) {
        JsonElement element = obj.get(key);
        return element.getAsJsonArray();
    }

    /**
     * 得到根JsonObject
     *
     * @param jsonStr
     * @return
     */
    public static JsonObject getRootJsonObject(String jsonStr) {
        JsonObject obj = new JsonParser().parse(jsonStr).getAsJsonObject();
        return obj;
    }

    /**
     * 得到根JsonArray
     *
     * @param jsonStr
     * @return
     */
    public static JsonArray getRootJsonArray(String jsonStr) {
        JsonArray obj = new JsonParser().parse(jsonStr).getAsJsonArray();
        return obj;
    }

    /**
     * 从JsonObject对象中获取键值
     *
     * @param obj
     * @param key
     * @return
     */
    public static JsonElement getKeyValue(JsonObject obj, String key) {
        JsonElement element = obj.get(key);
        return element;
    }

    /**
     * JsonArrary---->>>List<JsonObject>
     *
     * @param jsonArray
     * @return
     */
    public static List<JsonObject> JsonArrayToList(JsonArray jsonArray) {
        List<JsonObject> obj = new ArrayList<JsonObject>();
        for (int i = 0; i < jsonArray.size(); i++) {
            obj.add(jsonArray.get(i).getAsJsonObject());
        }
        return obj;
    }

    public Gson getGson() {
        return gson;
    }

    /****************** bean-jsonString配合builder使用 ***************************************/

    // TODO 简单bean,带泛型的List------>>>>>String
    public static <T> String toJsonString(List<T> json) {
        return gson.toJson(json);
    }

    // TODO 简单bean,带泛型的List------>>>>>String
    public static <T> String toJsonString(T json) {
        return gson.toJson(json);
    }

    /****************** bean-jsonobject配合builder使用 ***************************************/
    // TODO bean-------->>>>jsonObject
    public static <T> JsonObject beanToJsonObject(T json) {
        return getRootJsonObject(toJsonString(json));
    }

    // TODO List<bean>------>>>>>jsonArrary
    public static <T> JsonArray ListToJsonArrary(List<T> json) {
        return getRootJsonArray(toJsonString(json));
    }

    /********************* json--bean配合Gson解析 *************************************/
    // TODO 转换JSONObject对象中的JSONObject为Bean
    public static <T> T toBean(JsonObject obj, String key, Class<T> type) {
        return gson.fromJson(obj.get(key).toString(), type);
    }

    // TODO 转换JSONObject对象中的JsonArray为List<Bean>
    public static <T> List<T> toList(JsonObject obj, String key, Class<T> type) {
        return JsonArrayToListBean(obj.get(key).getAsJsonArray(), type);
    }

    // TODO 直接把JsonArrary对象转化为List<Bean>
    public static <T> List<T> JsonArrayToListBean(JsonArray jsonArray,
                                                  Class<T> type) {
        List<T> obj = new ArrayList<T>();
        for (int i = 0; i < jsonArray.size(); i++) {
            obj.add(gson.fromJson(jsonArray.get(i).toString(), type));
        }
        return obj;
    }

    // TODO 直接把json对象转化为bean
    public static <T> T JsonObjectToBean(JsonObject jsonObject, Class<T> type) {
        return gson.fromJson(jsonObject.toString(), type);
    }

    // 检查对象是否存在
    public static boolean checkJsonObejct(JsonObject root, String key) {
        if ((!getKeyValue(root, key).isJsonNull())
                && getKeyValue(root, key).isJsonObject()) {
            return true;
        }
        return false;

    }

    // 检查数组是否存在
    public static boolean checkJsonArray(JsonObject root, String key) {
        if ((!getKeyValue(root, key).isJsonNull())
                && getKeyValue(root, key).isJsonArray()) {
            return true;
        }
        return false;
    }

    public static JsonArray getRootJsonArrayByInputStream(Reader jsonStr) {
        JsonArray obj = new JsonParser().parse(jsonStr).getAsJsonArray();
        return obj;
    }
}
```

## 网络请求数据显示

显示页面很简单，如果想看效果，直接到最下面看UI效果图。

页面就一个搜索框加一个recycleview列表显示，显示网络请求url和请求状态码，点击搜索，实时显示搜索结果。直接贴代码

`HttpListActivity.java`

```
public class HttpListActivity extends BaseSwipeBackActivity {
    @Bind(R.id.index_app_name)
    TextView indexAppName;
    @Bind(R.id.recycler_view)
    RecyclerView recyclerView;
    @Bind(R.id.smart_rl)
    SmartRefreshLayout refreshLayout;
    @Bind(R.id.et_name)
    EditText etName;
    private HttpListAdapter adapter;
    private String mKeyWords = "";
    List<HttpCacheBean> mListBeans;

    @Override
    protected void onInitialization(Bundle bundle) {
    }

    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            adapter.setNewData(mListBeans);
            refreshLayout.finishRefresh();
        }
    };

    @Override
    protected void initView(Bundle bundle) {
        indexAppName.setText("Http请求Log日志");
        initList();
        setListDemo();
        etName.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
                mKeyWords = charSequence.toString().trim();
                setListDemo();
            }

            @Override
            public void afterTextChanged(Editable editable) {
            }
        });
    }

    private void setListDemo() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                mListBeans = HttpLogManager.getLogLists(mKeyWords);
                mHandler.sendEmptyMessage(0);
            }
        }).start();
    }


    private void initList() {
        ListEmptyView emptyView = new ListEmptyView(getContext());
        adapter = new HttpListAdapter();
        recyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
        recyclerView.setAdapter(adapter);
        adapter.setOnItemClickListener(new BaseQuickAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(BaseQuickAdapter adapter, View view, int position) {
                Bundle bundle = new Bundle();
                bundle.putSerializable(HttpCacheBean.TAG, (HttpCacheBean) adapter.getItem(position));
                gStartActivity(HttpDetailsActivity.class, bundle);
            }
        });
        adapter.setEmptyView(emptyView);
        refreshLayout.setEnableLoadMore(false);
        refreshLayout.setOnRefreshListener(new OnRefreshListener() {
            @Override
            public void onRefresh(@NonNull RefreshLayout refreshLayout) {
                setListDemo();
            }
        });
    }

    @Override
    protected void initData(Bundle bundle) {

    }

    @Override
    protected int getLayoutId() {
        return R.layout.activity_httplog_layout;
    }


    @OnClick(R.id.img_back)
    public void onViewClicked() {
        finish();
    }
}
```

`注意:这里因为有些请求返回数据量太大，直接查询100条显示页面会卡住，所以这里用了handler进行耗时处理，显示更加流畅`

`activity_http_layout.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/bg_color"
    android:orientation="vertical">

    <include layout="@layout/titlebar_base_layout" />

    <EditText
        android:id="@+id/et_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/dimen_8dp"
        android:background="@color/color_EDEDED"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:hint="接口名称"
        android:padding="@dimen/dimen_10dp"
        android:textSize="@dimen/dimen_14dp" />

    <com.scwang.smartrefresh.layout.SmartRefreshLayout
        android:id="@+id/smart_rl"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_below="@+id/title_bar" />
    </com.scwang.smartrefresh.layout.SmartRefreshLayout>

</LinearLayout>
```

http日志详情显示页面

 `HttpDetailsActivity.java`

```
public class HttpDetailsActivity extends BaseSwipeBackActivity {
    HttpCacheBean httpCacheBean;
    @Bind(R.id.ll_back)
    LinearLayout llBack;
    @Bind(R.id.index_app_name)
    TextView indexAppName;
    @Bind(R.id.tv_title_right)
    TextView tvTitleRight;
    @Bind(R.id.tv_req_allUrl)
    TextView tvReqAllUrl;
    @Bind(R.id.tv_req_url)
    TextView tvReqUrl;
    @Bind(R.id.tv_req_code)
    TextView tvReqCode;
    @Bind(R.id.tv_req_type)
    TextView tvReqType;
    @Bind(R.id.tv_req_header)
    TextView tvReqHeader;
    @Bind(R.id.tv_req_data)
    TextView tvReqData;
    @Bind(R.id.ll_request)
    LinearLayout llRequest;
    @Bind(R.id.tv_res_allUrl)
    TextView tvResAllUrl;
    @Bind(R.id.tv_res_url)
    TextView tvResUrl;
    @Bind(R.id.tv_res_time)
    TextView tvResTime;
    @Bind(R.id.tv_res_code)
    TextView tvResCode;
    @Bind(R.id.tv_res_type)
    TextView tvResType;
    @Bind(R.id.tv_res_header)
    TextView tvResHeader;
    @Bind(R.id.tv_res_error)
    TextView tvResError;
    @Bind(R.id.tv_res_body)
    TextView tvResBody;
    @Bind(R.id.ll_response)
    LinearLayout llResponse;
    @Bind(R.id.tv_content_type)
    TextView tvContentType;
    @Bind(R.id.tv_req_time)
    TextView tvReqTime;

    @Override
    protected void onInitialization(Bundle bundle) {
        httpCacheBean = (HttpCacheBean) bundle.getSerializable(HttpCacheBean.TAG);
    }

    @Override
    protected void initView(Bundle bundle) {
        indexAppName.setText("请求详情");

    }

    @Override
    protected void initData(Bundle bundle) {
        if (httpCacheBean != null) {
            if (!TextUtils.isEmpty(httpCacheBean.url) && !TextUtils.isEmpty(httpCacheBean.method)) {
                if (httpCacheBean.method.equals("GET")) {
                    try {
                        URL url = new URL(URLDecoder.decode(httpCacheBean.url));
                        httpCacheBean.requestBody = url.getQuery();
                    } catch (MalformedURLException e) {
                    }
                }

            }
            tvReqTime.setText(httpCacheBean.requestTime);
            tvReqAllUrl.setText(TextUtils.isEmpty(httpCacheBean.requestMsg) ? "--" : URLDecoder.decode(httpCacheBean.requestMsg));
            tvReqUrl.setText(TextUtils.isEmpty(httpCacheBean.url) ? "--" : URLDecoder.decode(httpCacheBean.url));
            tvReqCode.setText(TextUtils.isEmpty(httpCacheBean.code + "") ? "--" : httpCacheBean.code + "");
            tvReqHeader.setText(TextUtils.isEmpty(httpCacheBean.reqHeaders) ? "--" : StringUtil.formatJson(httpCacheBean.reqHeaders));
            tvReqData.setText(TextUtils.isEmpty(httpCacheBean.requestBody) ? "--" : StringUtil.formatJson(httpCacheBean.requestBody));
            tvReqType.setText(TextUtils.isEmpty(httpCacheBean.method) ? "--" : httpCacheBean.method);
            tvContentType.setText(TextUtils.isEmpty(httpCacheBean.reqContentType) ? "--" : httpCacheBean.reqContentType);
            tvResAllUrl.setText(TextUtils.isEmpty(httpCacheBean.responseMsg) ? "--" : URLDecoder.decode(httpCacheBean.responseMsg));
            tvResUrl.setText(TextUtils.isEmpty(httpCacheBean.url) ? "--" : URLDecoder.decode(httpCacheBean.url));
            tvResCode.setText(TextUtils.isEmpty(httpCacheBean.code + "") ? "--" : httpCacheBean.code + "");
            tvResHeader.setText(TextUtils.isEmpty(httpCacheBean.resHeaders) ? "--" : StringUtil.formatJson(httpCacheBean.resHeaders));
            tvResBody.setText(TextUtils.isEmpty(httpCacheBean.responseBody) ? "--" : httpCacheBean.responseBody);
            tvResTime.setText(TextUtils.isEmpty(httpCacheBean.time) ? "--" : httpCacheBean.time);
            tvResType.setText(TextUtils.isEmpty(httpCacheBean.resContentType) ? "--" : httpCacheBean.resContentType);
            tvResError.setText(TextUtils.isEmpty(httpCacheBean.error) ? "--" : httpCacheBean.error);
            if (httpCacheBean.code >= 200 && httpCacheBean.code < 300) {
                tvReqCode.setTextColor(Color.parseColor("#00FF00"));
                tvResCode.setTextColor(Color.parseColor("#00FF00"));
            }else if (httpCacheBean.code >= 400) {
                tvReqCode.setTextColor(Color.parseColor("#FF0000"));
                tvResCode.setTextColor(Color.parseColor("#FF0000"));
            } else {
                tvReqCode.setTextColor(Color.parseColor("#0000FF"));
                tvResCode.setTextColor(Color.parseColor("#0000FF"));
            }
        }

    }

    @Override
    protected int getLayoutId() {
        return R.layout.activity_http_details_layout;
    }

    @OnClick(R.id.ll_back)
    public void onViewClicked() {
        finish();
    }
}
```

`activity_http_details_layout.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/bg_color"
    android:orientation="vertical">

    <include layout="@layout/titlebar_base_simple_layout" />

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="none">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:background="@color/color_EDEDED"
                android:padding="@dimen/dimen_5dp"
                android:text="Request:"
                android:textSize="15dp" />

            <LinearLayout
                android:id="@+id/ll_request"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="@dimen/dimen_5dp"
                android:orientation="vertical">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求时间:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_time"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求整体:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_allUrl"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求Url:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_url"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求数据:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_data"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="状态码:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_code"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求类型:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_type"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求ContentType:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_content_type"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="请求头:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_req_header"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

            </LinearLayout>

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="@dimen/dimen_10dp"
                android:background="@color/color_EDEDED"
                android:padding="@dimen/dimen_5dp"
                android:text="Response:"
                android:textSize="15dp" />

            <LinearLayout
                android:id="@+id/ll_response"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="@dimen/dimen_5dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="返回请求整体:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_allUrl"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />


                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="返回请求Url:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_url"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />


                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="返回请求时间:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_time"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="状态码:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_code"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="返回内容类型:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_type"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="返回头:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_header"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:background="@color/color_EDEDED"
                    android:textIsSelectable="true"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="错误信息error:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_error"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:textColor="@color/color_4A4A4A"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:textIsSelectable="true"
                    android:background="@color/color_EDEDED"
                    android:textSize="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:padding="@dimen/dimen_5dp"
                    android:text="body:"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />

                <TextView
                    android:id="@+id/tv_res_body"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_margin="@dimen/dimen_5dp"
                    android:lineSpacingMultiplier="1.2"
                    android:background="@color/color_EDEDED"
                    android:padding="@dimen/dimen_5dp"
                    android:textIsSelectable="true"
                    android:textColor="@color/color_4A4A4A"
                    android:textSize="12dp" />


            </LinearLayout>

        </LinearLayout>

    </android.support.v4.widget.NestedScrollView>
</LinearLayout>
```

`HttpListAdapter.java`

```
public class HttpListAdapter extends BaseQuickAdapter<HttpCacheBean, BaseViewHolder> {

    public HttpListAdapter() {
        super(R.layout.item_http_view);
    }

    @Override
    protected void convert(BaseViewHolder helper, HttpCacheBean item) {
        helper.setText(R.id.tv_code, TextUtils.isEmpty(item.code + "") ? "none" : item.code + "");
        helper.setText(R.id.tv_method, TextUtils.isEmpty(item.method) ? "none" : item.method);
        helper.setText(R.id.tv_url, TextUtils.isEmpty(item.url) ? "--" : URLDecoder.decode(item.url));
    }
}
```

`item_http_view.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="10dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/tv_method"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/white"
            android:background="@color/gray"
            android:paddingLeft="@dimen/dimen_10dp"
            android:paddingRight="@dimen/dimen_10dp"
            android:textSize="14dp" />

        <TextView
            android:id="@+id/tv_code"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="@dimen/dimen_5dp"
            android:textColor="@color/white"
            android:paddingLeft="@dimen/dimen_10dp"
            android:paddingRight="@dimen/dimen_10dp"
            android:background="@color/color_FA6469"
            android:textSize="14dp" />

    </LinearLayout>

    <TextView
        android:id="@+id/tv_url"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="@color/color_2D2D2D"
        android:textSize="14dp" />

    <View
        android:layout_width="match_parent"
        android:layout_height="@dimen/dimen_1dp"
        android:background="@color/color_EDEDED"/>

</LinearLayout>
```

全部代码粘贴完毕。大家学习借鉴，本文章都是自己研究创作，有瑕疵，请大家多多包涵，共同学习进步。

本文只截取部分代码上传到github，属于项目集成的，没有时间分离，要借鉴移步github项目下载部分代码，合并到自己分支。

Android版截图

![6hQXy5dakfYvSNR](https://i.loli.net/2019/08/29/6hQXy5dakfYvSNR.jpg)

![XZGoiMsjElhYqng](https://i.loli.net/2019/08/29/XZGoiMsjElhYqng.jpg)

![6YucS8HtDky49vr](https://i.loli.net/2019/08/29/6YucS8HtDky49vr.jpg)

![LZBykW7hmobq4nN](https://i.loli.net/2019/08/29/LZBykW7hmobq4nN.jpg)

github地址：https://github.com/coffeelife/HttpLogCode.git
