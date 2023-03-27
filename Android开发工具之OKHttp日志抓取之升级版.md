# Android开发工具之OKHttp日志抓取之升级版

## 前言

前面讲到过有关于OkHttp框架模式下的日志打印工具，Android没有像IOS DBDebugToolkit这样的工具框架。所以我自己写了一个http日志调试类，主要是在手机本地记录app的请求日志。

前面写的算是1.0版，主要是没想花太多时间去搞这个，这个方面查看网络日志，当测试有bug提出来的时候先看看网络日志是否有问题，方便自己和测试排查问题，每次通过AS工具来看打印的log或者debug调试比较浪费时间和精力，也有可能是无法复现的一些问题。

## 问题

### 1.0版本问题

1.0是只提供了思路，然后根据思路大致写了一版，在最后也说明了一些存在的问题

1.由于是通过sp来保存的数据，sp是只能存储少量数据才推荐使用的，在时间场景中json数据过大或者提交图片或者音视频数据时会转换承成流提交到服务端，这部分请求也会被记录下来，现在图片一般都1M以上（压缩之后也得几百k)这样就会导致sp存储崩掉。

2.在记录okhttp请求时，我用了“随缘”请求方法，有网络请求就直接走保存sp，因为是异步请求，会有同时请求n次的情况存在，这样sp并没有保存成功就已经开始了下一次保存的情况，中间就会有漏掉的情况（实际运行也是这样的，所以不能偷懒哈）。

3.sp保存操作并没有走子线程，这样保存坏处就在于和实际的网络请求一起在发生，app运行会变慢，影响实际效率（测试环境才会走这个，当然对于我这种程序员感觉不深刻）。

### 优化方案

1.解决sp使用问题，该版本使用文件保存日志方法

2.解决日志数据太大还有漏掉的问题，该版本使用一个单例（其实我对单例用的不好）加队列的方式来解决该问题，让所有的请求先加入一个队列当中，然后循环队列任务，有任务加入队列，完成任务踢出队列，没有任务就不进行操作。

3.文件操作这次都放在一个子线程中来进行，每次只会开启一个子线程来保存网络请求日志，将任务从队列中取出来，放入子线程执行，当保存日志完成之后将该任务结束，继续下一次队列任务，这样减少内存损耗，也解决了漏掉的问题。

## 具体实现

看理论比较枯燥，接下来看代码实现吧！

### 创建HttpLogManager

首先我们创建一个HttpLogManager文件，主要是和主业务代码解耦，把httplog日志处理操作都封装到一个类中，需要那个直接调用那个就可以了

`HttpLogManager.java`

```
public class HttpLogManager {
    private static HttpLogManager httpLogManager = null;

    public static HttpLogManager instance() {
        if (httpLogManager == null)
            httpLogManager = new HttpLogManager();

        return httpLogManager;
    }

    //得到通过搜索关键字的日志列表
    public static List<HttpCacheBean> getShowLogLists(String keyWords) {
        List<HttpCacheBean> listBeans = getLogLists();
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

    //得到日志文件list
    public static List<HttpCacheBean> getLogLists() {
        List<HttpCacheBean> listBeans = null;
        String json = readLog();
        if (GsonUtil.isGoodJson(json))
            listBeans = GsonUtil.getEntityList(json, HttpCacheBean.class);
        else
            listBeans = new ArrayList<>();
        return listBeans;
    }

    //保存日志文件
    public static void saveLogBean(HttpCacheBean bean) {
        postLog(bean);
        //判断运行环境 是否在本地生成日志文件
        if (GlobalEnv.getEnvModel() != GlobalEnv.ENV_RELEASE) {
            ActionLog.recordActionInfoLog(bean);
        } else {
            //postLog(bean);如果有自己的后台 可以把日志上传到自己的后台
        }
    }

    //获取汉字解码过的日志文件
    public static String readLog() {
        String json = null;
        try {
            json = FileUtils.readFileToString(ActionLog.getFilePath());
        } catch (IOException e) {
            json = "";
        }
        return json;
    }
}
```

### 编写日志保存队列

写完httplog日志管理类，接下来要处理的就是同时发起多个异步请求时，如何将这些请求日志存储起来，我这里写了一个ActionLog日志类，主要是仿android手机日志记录的方式。该类使用队列`ConcurrentLinkedQueue`，将所有发起的请求放到一个app的全局队列当中，该类使用单例模式，全局只会有一个日志记录类存在。

这样的好处就是每次一有网络请求，不用关心上次记录记录成功状态，直接加入队列当中，根据先进先出的原则，循环队列依次进行log日志存储；请求日志选择存储到本地文件当中，如果没有文件就创建，然后保存到文件当中，不管该队列保存成功失败都将目前队列结束。

`ActionLog.java`

```
/**
 * 行为日志记录
 *
 * @author Administrator
 */
public class ActionLog {
    private static String filePath = "";
    //日志队列
    public static ConcurrentLinkedQueue tempQueue = new ConcurrentLinkedQueue<>();

    /**
     * 记录行为信息
     * 单例模式创建日志工具类
     * @param bean
     */
    public static synchronized void recordActionInfoLog(HttpCacheBean bean) {
        tempQueue.offer(bean);
        if (!WriteThread.isWriteThreadLive) {
            new WriteThread().start();
        }
    }

    /**
     * 打开日志文件并写入日志
     *
     * @return
     **/
    public static void recordStringLog(String text) {// 新建或打开日志文件
        filePath = getFilePath();
        File file = new File(filePath);
        if (!file.exists()) {
            file.getParentFile().mkdirs();
            try {
                file.createNewFile();
                Logger.d("网络日志：在" + filePath + "创建文件成功！");
            } catch (IOException e) {
                Logger.d("网络日志：在" + filePath + "创建文件失败！");
            }
        }

        //写日志到SD卡
        try {
            FileUtils.writeStringToFile(text, filePath);
            Logger.d("网络日志：网络日志写入成功！", text);
        } catch (IOException e) {
            Logger.d("网络日志：网络日志写入失败！", text);
        }
        tempQueue.poll();//日志记录完成就将队列结束
    }

    /**
     * 判断日志文件是否存在
     *
     * @return
     */
    public static boolean isExitLogFile() {
        File file = new File(filePath);
        if (file.exists() && file.length() > 3) {
            return true;
        } else {
            return false;
        }
    }

    /**
     * 删除日志文件
     */
    public static void deleteLogFile() {
        if (TextUtils.isEmpty(filePath)) return;
        File file = new File(filePath);
        if (file.exists()) {
            file.delete();
        }
    }

    //获取当前日志文件路径
    //判断当前环境 测试 预发 线上 创建不同目录文件
    public static String getFilePath() {
        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) {// 优先保存到SD卡中
            filePath = Environment.getExternalStorageDirectory()
                    .getAbsolutePath() + File.separator + "HttpLog" + (GlobalEnv.isRelease() ? "Release" : GlobalEnv.isBeta() ? "Beta" : "Debug");
        } else {// 如果SD卡不存在，就保存到本应用的目录下
            filePath = BaseApp.getContext().getFilesDir().getAbsolutePath()
                    + File.separator + "HttpLog" + (GlobalEnv.isRelease() ? "Release" : GlobalEnv.isBeta() ? "Beta" : "Debug");
        }
        return filePath;
    }
}
```

### 创建Httplog记录线程

在前面基础上，添加了一个子线程来专门处理记录日志，避免阻塞网络请求和主线程任务，该类直接创建一个子线程，在子线程遍历任务队列，如果有队列任务，就执行保存日志，有异常抛出中断该队列，继续执行下一队列任务。

``

```
public class WriteThread extends Thread {

    public static boolean isWriteThreadLive = false;//写日志线程是否已经在运行了

    public WriteThread() {
        isWriteThreadLive = true;
    }

    @Override
    public void run() {
        isWriteThreadLive = true;
        while (!ActionLog.tempQueue.isEmpty()) {//对列不空时
            try {
                HttpCacheBean bean = (HttpCacheBean) ActionLog.tempQueue.peek();
                if (bean == null) {
                    ActionLog.tempQueue.poll();
                    return;
                }
                List<HttpCacheBean> httpCacheBeans = HttpLogManager.getLogLists();
                httpCacheBeans.add(bean);
                if (!ArrayUtils.isEmpty(httpCacheBeans) && httpCacheBeans.size() > 100) {
                    httpCacheBeans = httpCacheBeans.subList(httpCacheBeans.size() - 100, httpCacheBeans.size());
                }
                //写日志到SD卡
                ActionLog.recordStringLog(GsonUtil.ListToJsonArrary(httpCacheBeans).toString());
            } catch (Exception e) {
                ActionLog.tempQueue.poll();//异常中断队列任务
                Logger.d("网络日志：网络日志写入失败！");
                break;
            }
        }
        isWriteThreadLive = false;//队列中的日志都写完了，关闭线程（也可以常开 要测试下）
    }
}
```

## 为OkHttp设置拦截器

上述工具写好之后，我们再为okhttp设置拦截器来拦截每一次的网络请求并调用HttpLogManager来记录对应的网络日志情况。

```
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
        Request request = chain.request();
        //如果当前设置日志类型为NONE 就按原来方式返回
        if (level == HttpLogInterceptor.Level.NONE) {
            return chain.proceed(request);
        }
        //如果当前设置日志类型为NONE以外 就打印网络日志并记录
        long startNs = System.nanoTime();
        HttpCacheBean bean = new HttpCacheBean();
        bean.requestTime = DateFormatUtils.getCurrentDateTime();
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
            Headers headers = request.headers();
            if (headers.size() > 0) {
                JsonObject jsonHeaders = new JsonObject();
                for (int i = 0, count = headers.size(); i < count; i++) {
                    String value = headers.value(i);
                    if (value != null) {
                        value = URLDecoder.decode(value, "UTF-8");
                    }
                    jsonHeaders.addProperty(headers.name(i), value);
                }
                String reqHeaders = jsonHeaders.toString();
                logger.json("httphead-req", reqHeaders);
                bean.reqHeaders = reqHeaders;
            }

            if (!logBody || !hasRequestBody) {
                logger.log("--> END " + request.method());
            } else if (bodyEncoded(request.headers())) {
                logger.log("--> END " + request.method() + " (encoded body omitted)");
            } else {
                Buffer buffer = new Buffer();
                requestBody.writeTo(buffer);
                Charset charset = UTF8;
                MediaType contentType = requestBody.contentType();
                if (contentType != null) {
                    charset = contentType.charset(UTF8);
                }
                if (isPlaintext(buffer)) {
                    String httpRequestBody = buffer.clone().readString(charset);
                    logger.json("httprequest", httpRequestBody);
                    bean.requestBody = httpRequestBody;
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
            //如果出现异常 直接记录日志文件
            HttpLogManager.instance().saveLogBean(bean);
            throw e;
        }
        long tookMs = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startNs);
        ResponseBody responseBody = response.body();
        long contentLength = responseBody.contentLength();
        String bodySize = contentLength != -1 ? contentLength + "-byte" : "unknown-length";
        String response_msg = "<-- " + response.code() + ' ' + response.message() + ' '
                + response.request().url() + " (" + tookMs + "ms" + (!logHeaders ? ", "
                + bodySize + " body" : "") + ')';
        logger.log(response_msg);
        bean.responseMsg = response_msg;
        bean.resContentLength = bodySize;
        bean.code = response.code();
        bean.msg = response.message();
        bean.time = tookMs;
        if (logHeaders) {
            Headers headers = response.headers();
            JsonObject jsonHeaders = new JsonObject();
            for (int i = 0, count = headers.size(); i < count; i++) {
                String value = headers.value(i);
                if (value != null) {
                    value = URLDecoder.decode(value, "UTF-8");
                }
                jsonHeaders.addProperty(headers.name(i), value);
            }
            String head_rsp = jsonHeaders.toString();
            logger.json("httphead-rsp", head_rsp);
            bean.resHeaders = head_rsp;
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
                if (contentLength != 0) {
                    String response_body = buffer.clone().readString(charset);
                    logger.log("body", response_body);
                    bean.responseBody = response_body;
                }
                logger.log("<-- END HTTP (" + buffer.size() + "-byte body)");
            }
        }
        //记录日志文件
        HttpLogManager.instance().saveLogBean(bean);
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

## OkHttp添加拦截器

为OKHttp请求添加拦截器代码

```
 OkHttpClient.Builder builder = new OkHttpClient.Builder();
        //log相关
        //NONE代表的是当前环境如果release包就不记录日志
        //测试环境的话就记录 也可以更改 如果需要传到后端 就开起来BODY
        HttpLogInterceptor loggingInterceptor = new HttpLogInterceptor().setLevel(
                !GlobalEnv.isRelease() ? HttpLogInterceptor.Level.NONE : HttpLogInterceptor.Level.BODY);
        builder.addInterceptor(new AddCookiesInterceptor());                                                         
        //超时时间设置，默认60秒
        builder.readTimeout(GHttp.DEFAULT_MILLISECONDS, TimeUnit.MILLISECONDS);      //全局的读取超时时间
        builder.writeTimeout(GHttp.DEFAULT_MILLISECONDS, TimeUnit.MILLISECONDS);     //全局的写入超时时间
        builder.connectTimeout(GHttp.DEFAULT_MILLISECONDS, TimeUnit.MILLISECONDS);   //全局的连接超时时间
```

## 用到的工具类

`GsonUtil.java`

```
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

`FileUtils.java`

```
/**
 * external storage
 * 外部存储    Environment.getExternalStorageDirectory()    SD根目录:/mnt/sdcard/ (6.0后写入需要用户授权)
 * context.getExternalFilesDir(dir)    路径为:/mnt/sdcard/Android/data/< package name >/files/…
 * context.getExternalCacheDir()    路径为:/mnt/sdcard/Android/data/< package name >/cach/…
 * internal storage
 * 内部存储    context.getFilesDir()    路径是:/data/data/< package name >/files/…
 * context.getCacheDir()    路径是:/data/data/< package name >/cach/…
 */
public class FileUtils {
    private static final int WRITE_BUFFER_SIZE = 1024 * 8;

    /**
     * @param fileName
     * @param content
     * @throws Exception
     * @desc 保存内容到文件中
     */
    public static void save(Context context, String fileName, String content, int module) {
        try {
            FileOutputStream os = context.openFileOutput(fileName, module);
            os.write(content.getBytes());
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * @param fileName
     * @return
     * @desc 读取文件内容
     */
    public static String read(Context context, String fileName) {

        try {
            FileInputStream fis = context.openFileInput(fileName);
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            byte[] b = new byte[1024];
            int len = 0;
            while ((len = fis.read(b)) != -1) {
                bos.write(b, 0, len);
            }
            byte[] data = bos.toByteArray();
            fis.close();
            bos.close();
            return new String(data);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


    /**
     * @param context
     * @param fileName
     * @param content
     * @throws IOException
     * @desc 将文本内容保存到sd卡的文件中
     */
    public static void saveToSDCard(Context context, String fileName, String content) throws IOException {

        File file = new File(Environment.getExternalStorageDirectory(), fileName);
        FileOutputStream fos = new FileOutputStream(file);
        fos.write(content.getBytes());
        fos.close();
    }

    /**
     * @param fileName
     * @return
     * @throws IOException
     * @desc 读取sd卡文件内容
     */
    public static String readSDCard(String fileName) throws IOException {

        File file = new File(Environment.getExternalStorageDirectory(), fileName);
        FileInputStream fis = new FileInputStream(file);
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int len = 0;
        while ((len = fis.read(buffer)) != -1) {
            bos.write(buffer, 0, len);
        }
        byte[] data = bos.toByteArray();
        fis.close();
        bos.close();

        return new String(data);
    }

    /*获取缓存路径，存储临时文件，可被一键清理和卸载清理*/
    /*
     * 可以看到，当SD卡存在或者SD卡不可被移除的时候，
     * 就调用getExternalCacheDir()方法来获取缓存路径，
     * 否则就调用getCacheDir()方法来获取缓存路径。
     * 前者获取到的就是/sdcard/Android/data/<application package>/cache 这个路径，
     * 而后者获取到的是 /data/data/<application package>/cache 这个路径。*/
    public static File getDiskCacheDir(Context context, String uniqueName) {
        String cachePath;
        if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
                || !Environment.isExternalStorageRemovable()) {
            cachePath = context.getExternalCacheDir().getPath();
        } else {
            cachePath = context.getCacheDir().getPath();
        }
        return new File(cachePath + File.separator + uniqueName);
    }


    public static File getDiskDir(Context context, String dir) {
        String diskPath;
        if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
                || !Environment.isExternalStorageRemovable()) {
            return context.getExternalFilesDir(dir);
        } else {
            diskPath = context.getFilesDir().getPath();
            File file = new File(diskPath + File.separator + dir);
            if (!file.exists()) {//判断文件目录是否存在  
                file.mkdirs();
            }
            return file;
        }
    }


    /*返回缓存路径*/
    public static File getDiskCacheDir(Context context) {
        String cachePath;
        if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
                || !Environment.isExternalStorageRemovable()) {
            cachePath = context.getExternalCacheDir().getPath();
        } else {
            cachePath = context.getCacheDir().getPath();
        }
        return new File(cachePath);
    }

    public static File getFilesDir(Context context, String uniqueName) {
        return new File(context.getFilesDir().getPath() + File.separator + uniqueName);
    }

    public static File getCacheDir(Context context, String uniqueName) {
        return new File(context.getCacheDir().getPath() + File.separator + uniqueName);
    }

    /*判断sd卡可用*/
    public static boolean hasSDCardMounted() {
        String state = Environment.getExternalStorageState();
        if (state != null && state.equals(Environment.MEDIA_MOUNTED)) {
            return true;
        } else {
            return false;
        }
    }

    public static String getFromAssets(Context context, String fileName) {
        //将json数据变成字符串
        StringBuilder stringBuilder = new StringBuilder();
        try {
            //获取assets资源管理器
            AssetManager assetManager = context.getAssets();
            //通过管理器打开文件并读取
            BufferedReader bf = new BufferedReader(new InputStreamReader(
                    assetManager.open(fileName)));
            String line;
            while ((line = bf.readLine()) != null) {
                stringBuilder.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return stringBuilder.toString();
    }

    public static String appendPathComponent(String basePath, String appendPathComponent) {
        return new File(basePath, appendPathComponent).getAbsolutePath();
    }

    public static void copyDirectoryContents(String sourceDirectoryPath, String destinationDirectoryPath) throws IOException {
        File sourceDir = new File(sourceDirectoryPath);
        File destDir = new File(destinationDirectoryPath);
        if (!destDir.exists()) {
            destDir.mkdir();
        }

        for (File sourceFile : sourceDir.listFiles()) {
            if (sourceFile.isDirectory()) {
                copyDirectoryContents(
                        appendPathComponent(sourceDirectoryPath, sourceFile.getName()),
                        appendPathComponent(destinationDirectoryPath, sourceFile.getName()));
            } else {
                File destFile = new File(destDir, sourceFile.getName());
                FileInputStream fromFileStream = null;
                BufferedInputStream fromBufferedStream = null;
                FileOutputStream destStream = null;
                byte[] buffer = new byte[WRITE_BUFFER_SIZE];
                try {
                    fromFileStream = new FileInputStream(sourceFile);
                    fromBufferedStream = new BufferedInputStream(fromFileStream);
                    destStream = new FileOutputStream(destFile);
                    int bytesRead;
                    while ((bytesRead = fromBufferedStream.read(buffer)) > 0) {
                        destStream.write(buffer, 0, bytesRead);
                    }
                } finally {
                    try {
                        if (fromFileStream != null) fromFileStream.close();
                        if (fromBufferedStream != null) fromBufferedStream.close();
                        if (destStream != null) destStream.close();
                    } catch (IOException e) {
                        throw new IllegalArgumentException("Error closing IO resources.", e);
                    }
                }
            }
        }
    }

    public static void deleteDirectoryAtPath(String directoryPath) {
        if (directoryPath == null) {
            return;
        }
        File file = new File(directoryPath);
        if (file.exists()) {
            deleteFileOrFolderSilently(file);
        }
    }

    public static void deleteFileAtPathSilently(String path) {
        deleteFileOrFolderSilently(new File(path));
    }

    public static void deleteFileOrFolderSilently(File file) {
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            for (File fileEntry : files) {
                if (fileEntry.isDirectory()) {
                    deleteFileOrFolderSilently(fileEntry);
                } else {
                    fileEntry.delete();
                }
            }
        }

        if (!file.delete()) {
        }
    }

    public static boolean fileAtPathExists(String filePath) {
        return new File(filePath).exists();
    }

    public static void moveFile(File fileToMove, String newFolderPath, String newFileName) {
        File newFolder = new File(newFolderPath);
        if (!newFolder.exists()) {
            newFolder.mkdirs();
        }

        File newFilePath = new File(newFolderPath, newFileName);
        if (!fileToMove.renameTo(newFilePath)) {
            throw new IllegalArgumentException("Unable to move file from " +
                    fileToMove.getAbsolutePath() + " to " + newFilePath.getAbsolutePath() + ".");
        }
    }

    public static String readFileToString(String filePath) throws IOException {
        FileInputStream fin = null;
        BufferedReader reader = null;
        try {
            File fl = new File(filePath);
            fin = new FileInputStream(fl);
            reader = new BufferedReader(new InputStreamReader(fin));
            StringBuilder sb = new StringBuilder();
            String line = null;
            while ((line = reader.readLine()) != null) {
                sb.append(line).append("\n");
            }

            return sb.toString();
        } finally {
            if (reader != null) reader.close();
            if (fin != null) fin.close();
        }
    }

    public static void unzipFile(File zipFile, String destination) throws IOException {
        FileInputStream fileStream = null;
        BufferedInputStream bufferedStream = null;
        ZipInputStream zipStream = null;
        try {
            fileStream = new FileInputStream(zipFile);
            bufferedStream = new BufferedInputStream(fileStream);
            zipStream = new ZipInputStream(bufferedStream);
            ZipEntry entry;

            File destinationFolder = new File(destination);
            if (destinationFolder.exists()) {
                deleteFileOrFolderSilently(destinationFolder);
            }

            destinationFolder.mkdirs();

            byte[] buffer = new byte[WRITE_BUFFER_SIZE];
            while ((entry = zipStream.getNextEntry()) != null) {
                String fileName = entry.getName();
                File file = new File(destinationFolder, fileName);
                if (entry.isDirectory()) {
                    file.mkdirs();
                } else {
                    File parent = file.getParentFile();
                    if (!parent.exists()) {
                        parent.mkdirs();
                    }

                    FileOutputStream fout = new FileOutputStream(file);
                    try {
                        int numBytesRead;
                        while ((numBytesRead = zipStream.read(buffer)) != -1) {
                            fout.write(buffer, 0, numBytesRead);
                        }
                    } finally {
                        fout.close();
                    }
                }
                long time = entry.getTime();
                if (time > 0) {
                    file.setLastModified(time);
                }
            }
        } finally {
            try {
                if (zipStream != null) zipStream.close();
                if (bufferedStream != null) bufferedStream.close();
                if (fileStream != null) fileStream.close();
            } catch (IOException e) {
                throw new IllegalArgumentException("Error closing IO resources.", e);
            }
        }
    }

    public static void writeStringToFile(String content, String filePath) throws IOException {
        PrintWriter out = null;
        try {
            out = new PrintWriter(filePath);
            out.print(content);
        } finally {
            if (out != null) out.close();
        }
    }

    /**
     * 删除单个文件
     *
     * @param filePath$Name 要删除的文件的文件名
     * @return 单个文件删除成功返回true，否则返回false
     */
    public static boolean deleteSingleFile(Context context, String filePath$Name) {
        File file = new File(filePath$Name);
        // 如果文件路径所对应的文件存在，并且是一个文件，则直接删除
        if (file.exists() && file.isFile()) {
            if (file.delete()) {
                Logger.d("--Method--", "Copy_Delete.deleteSingleFile: 删除单个文件" + filePath$Name + "成功！");
                return true;
            } else {
                Logger.d("删除单个文件" + filePath$Name + "失败！", Toast.LENGTH_SHORT);
                return false;
            }
        } else {
            Logger.d("删除单个文件失败：" + filePath$Name + "不存在！");
            return false;
        }
    }

    /**
     * 删除目录及目录下的文件
     *
     * @param filePath 要删除的目录的文件路径
     * @return 目录删除成功返回true，否则返回false
     */
    public static boolean deleteDirectory(Context context, String filePath) {
        // 如果dir不以文件分隔符结尾，自动添加文件分隔符
        if (!filePath.endsWith(File.separator))
            filePath = filePath + File.separator;
        File dirFile = new File(filePath);
        // 如果dir对应的文件不存在，或者不是一个目录，则退出
        if ((!dirFile.exists()) || (!dirFile.isDirectory())) {
            Logger.d("删除目录失败：" + filePath + "不存在！");
            return false;
        }
        boolean flag = true;
        // 删除文件夹中的所有文件包括子目录
        File[] files = dirFile.listFiles();
        for (File file : files) {
            // 删除子文件
            if (file.isFile()) {
                flag = deleteSingleFile(context, file.getAbsolutePath());
                if (!flag)
                    break;
            }
            // 删除子目录
            else if (file.isDirectory()) {
                flag = deleteDirectory(context, file
                        .getAbsolutePath());
                if (!flag)
                    break;
            }
        }
        if (!flag) {
            Logger.d("删除目录失败！");
            return false;
        }
        // 删除当前目录
        if (dirFile.delete()) {
            Logger.d("--Method--", "Copy_Delete.deleteDirectory: 删除目录" + filePath + "成功！");
            return true;
        } else {
            Logger.d("删除目录：" + filePath + "失败！");
            return false;
        }
    }

    public static boolean isExists(String strFile) {
        try {
            File f = new File(strFile);
            if (!f.exists()) {
                return false;
            }

        } catch (Exception e) {
            return false;
        }

        return true;
    }

    public static boolean isImage(String pathName) {
        if (TextUtils.isEmpty(pathName) || !pathName.contains(".")) {
            return false;
        }
        String c = pathName.substring(pathName.lastIndexOf("."));
        if (TextUtils.equals(c, ".gif") ||
                TextUtils.equals(c, ".jpeg") ||
                TextUtils.equals(c, ".jpg") ||
                TextUtils.equals(c, ".png") ) {
            return true;
        }
        return false;
    }
}
```
