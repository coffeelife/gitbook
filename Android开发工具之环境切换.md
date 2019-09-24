# Android开发工具之环境切换

## 背景介绍

该工具是续篇，上一篇介绍的是Http日志抓取工具，这一篇讲解环境切换，环境切换为什么需要这个，现在稍微大点的公司基本都是两套以上环境，分为测试、预发和线上，有些公司没有预发这个部分。原来开发hybrid App的时候，可以直接修改配置文件的一个配置值可以切换当前环境，但是App如何这么设置的划，开发比较耗费时间，每次调试测试或者预发环境的接口时，总是得重新打包，对于一个开发者来说是十分痛苦的。

所以，环境切换就孕育而生，我们就打一次debug包，然后需要调试测试环境接口时，就摇一摇进入调试界面，然后点击切换环境，切换成测试环境，然后自动重启app，再次打开之后，app就进入测试环境，接口切换为测试环境的接口，数据配置都自动切换成测试环境的东西，无需打包，无需修改代码、无需重新运行。

页面截图

![mrxgX3K2JMTwlHF](https://i.loli.net/2019/08/28/mrxgX3K2JMTwlHF.jpg)

上面就是Debug界面，这个跟上一篇介绍的都是放在`BaseActivity`中，全局摇一摇就可以显示该界面，点击当前环境就可以选择当前环境。

## 代码实现

其实这部分也很简单，主要是通过Sp存储当前环境状态，默认第一次是测试环境，然后每次切换环境之后，将新的环境状态存储到sp当中，然后清除App的存储的数据状态，或者将数据存储到每个状态对应的目录下，这里见仁见智，退出登录并重新启动App，让App的新环境生效，重新加载新的App。

首先创建一个DebugActivity，来处理App所有的调试界面

`DebugActivity.java`

```
/**
 * Created by mguo on 2018/5/3.
 */
@Route(path = RouterConstant.ACTIVITY_DEBUG)
public class DebugActivity extends BaseSwipeBackActivity {
    @Bind(R.id.tv_env)
    TextView tvEnv;
    @Bind(R.id.tv_device_bind)
    TextView tvDeviceBind;
    @Bind(R.id.rl_device_bind)
    RelativeLayout rlDeviceBind;
    String[] items = new String[]{"测试", "生产", "预生产", "本地"};
    String[] itemBind = new String[]{"开启校验", "取消校验"};
    @Bind(R.id.img_back)
    ImageView imgBack;
    @Bind(R.id.index_app_name)
    TextView indexAppName;
    @Bind(R.id.tv_title_right)
    TextView tvTitleRight;
    @Bind(R.id.rl_wechat_share)
    RelativeLayout rlWechatShare;

    @Override
    protected int getLayoutId() {
        return R.layout.activity_debug_layout;
    }

    @Override
    protected void onInitialization(Bundle bundle) {
        ARouter.getInstance().inject(this);
    }

    @Override
    protected void initView(Bundle bundle) {
        if (GlobalEnv.isRelease()) {
            rlDeviceBind.setVisibility(View.GONE);
        }
        rlDeviceBind.setVisibility(View.VISIBLE);
        tvEnv.setText(items[GlobalEnv.getEnvModel()]);
        tvDeviceBind.setText(itemBind[GlobalEnv.isVerifyDevice() ? 0 : 1]);
        indexAppName.setText("环境调试");
        tvTitleRight.setText(getString(R.string.string_txt_cancel));
        tvTitleRight.setVisibility(View.VISIBLE);
    }

    @Override
    protected void initData(Bundle bundle) {

    }

    @OnClick({R.id.rl_cd,R.id.rl_http,R.id.layout_debug, R.id.rl_demo_list, R.id.layout_webview, R.id.rl_device_bind, R.id.tv_title_right, R.id.img_back, R.id.rl_update, R.id.rn_setting,R.id.rl_wechat_share})
    public void onViewClicked(View view) {

        switch (view.getId()) {
            case R.id.img_back:
                finish();
                break;
            case R.id.rl_demo_list:
                gStartActivity(DemoListActivity.class);
                break;
            case R.id.tv_title_right:
                finish();
                break;
            case R.id.rl_device_bind:
                AlertDialog.Builder builder1 = new AlertDialog.Builder(this);
                builder1.setTitle("选择是否校验");
                builder1.setSingleChoiceItems(itemBind, GlobalEnv.getEnvModel(), new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int i) {
                        dialog.dismiss();
                        GlobalEnv.setVerifyDevice(getContext(), i == 0);
                        tvDeviceBind.setText(itemBind[i]);
                    }
                });
                builder1.show();
                break;
            case R.id.layout_debug:
                AlertDialog.Builder builder = new AlertDialog.Builder(this);
                builder.setTitle("选择环境");
                builder.setSingleChoiceItems(items, GlobalEnv.getEnvModel(), new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int i) {
                        dialog.dismiss();
                        if (GlobalEnv.getEnvModel() == i) {
                            return;
                        }
                        showToast("应用重启中");
                        GlobalEnv.setEnvModel(i);
                        tvEnv.setText(items[i]);
                        UserManager.getInstance().logout();
                        AppExitUtil.restart(DebugActivity.this);
                    }
                });
                builder.show();
                break;
            case R.id.layout_webview:
                getRouter().showWebviewWithUrl("file:///android_asset/index.html");
                break;
            case R.id.rn_setting:
                gStartActivity(RNDebugActivity.class);
//                down();
                break;
            case R.id.rl_cd:
                openBrowser("http://10.8.16.25:7701/zentao/my/");
                break;
            case R.id.rl_http:
                gStartActivity(HttpListActivity.class);
                break;
            case R.id.rl_update:
                new SweetAlertDialog.Builder(this).setMessage("是否打开浏览器").setPositiveButton("确定", new SweetAlertDialog.OnDialogClickListener() {
                            @Override
                            public void onClick(Dialog dialog, int which) {
                                String url = "https://www.pgyer.com/XaM8";
                                int envModel = GlobalEnv.getEnvModel();
                                switch (envModel) {
                                    case ENV_DEBUG:
                                        url = "https://www.pgyer.com/ijZY";
                                        break;
                                    case ENV_RELEASE:
                                        url = "https://www.pgyer.com/XaM8";
                                        break;
                                    case ENV_BETA:
                                        url = "https://www.pgyer.com/yEs0";
                                        break;
                                }
                                openBrowser(url);
                            }
                        }
                ).setNegativeButton("取消", null)
                        .show();

                break;
            case R.id.rl_wechat_share:
                gStartActivity(SendToWXActivity.class);
                break;
        }
    }

    @Override
    protected String initPageName() {
        return "调试页面";
    }
}
```

`GlobalEnv.java`

```
public class GlobalEnv {

    public static final int ENV_DEBUG = 0;//测试环境
    public static final int ENV_RELEASE = 1;//生产环境
    public static final int ENV_BETA = 2;//预生产环境
    public static final int ENV_LOCAL = 3;//本地环境

    private static boolean mVerifyDevice = true;
    private static int mEnvModel = 0;
    private static String mtaChannel = "cstore";
    private static String versionName = "";

    public static String getChannel() {
        return mtaChannel;
    }

    public static String getVersionName() {
        return versionName;
    }

    public static boolean isDebug() {
        return TextUtils.equals(BuildConfig.BUILD_TYPE, "debug");
    }

    public static boolean isBeta() {
        return TextUtils.equals(BuildConfig.BUILD_TYPE, "beta");
    }

    public static boolean isRelease() {
        return TextUtils.equals(BuildConfig.BUILD_TYPE, "release");
    }

    public static int getEnvModel() {
        return mEnvModel;
    }

    public static void setVerifyDevice(Context context, boolean verify) {
        mVerifyDevice = verify;
        SharedUtil.instance(context).saveBoolean("env_device_verify", verify);
    }

    public static boolean isVerifyDevice() {
        return mVerifyDevice;
    }

    public static void setEnvModel(int envModel) {
        mEnvModel = envModel;
        SharedUtil.instance(BaseApp.getContext()).saveInt("mEnvModel", envModel);
        switch (envModel) {
            case ENV_DEBUG:
                ApiManager.getInstance().setBaseUrl(ApiConstant.BASETESTURL);
                ApiManager.getInstance().setWebUrl(ApiConstant.WEBTESTURL);
                ApiManager.getInstance().setTaskUrl(ApiConstant.TASKTESTURL);
                break;
            case ENV_RELEASE:
                ApiManager.getInstance().setBaseUrl(ApiConstant.BASEURL);
                ApiManager.getInstance().setWebUrl(ApiConstant.WEBURL);
                ApiManager.getInstance().setTaskUrl(ApiConstant.TASKURL);
                ApiManager.getInstance().setEmosUrl(ApiConstant.EOMSURL);
                break;
            case ENV_BETA:
                ApiManager.getInstance().setBaseUrl(ApiConstant.BASESTAGEURL);
                ApiManager.getInstance().setWebUrl(ApiConstant.WEBSTAGEURL);
                ApiManager.getInstance().setTaskUrl(ApiConstant.TASKSTAGEURL);
                ApiManager.getInstance().setEmosUrl(ApiConstant.EOMSURL);
                break;
            case ENV_LOCAL:
                ApiManager.getInstance().setBaseUrl(ApiConstant.BASELOCALURL);
                ApiManager.getInstance().setWebUrl(ApiConstant.WEBLOCALURL);
                ApiManager.getInstance().setTaskUrl(ApiConstant.TASKLOCALURL);
                break;
            default:
                ApiManager.getInstance().setBaseUrl(ApiConstant.BASETESTURL);
                ApiManager.getInstance().setWebUrl(ApiConstant.WEBTESTURL);
                ApiManager.getInstance().setTaskUrl(ApiConstant.TASKTESTURL);
                break;
        }

    }

    public static void init(Application application) {
        mVerifyDevice = SharedUtil.instance(application).getBoolean("env_device_verify", true);
        mtaChannel = DeviceUtil.getAppMetaData(application, "GJ_CHANNEL");
        versionName = DeviceUtil.getVersionName(application);
        Log.v("GlobalEnv", "channel=" + mtaChannel);
//        if (GlobalEnv.isRelease()) {
//            GlobalEnv.setEnvModel(GlobalEnv.ENV_RELEASE);
//            return;
//        }
        if (ApiManager.getInstance().isSetting()) {
            mEnvModel = SharedUtil.instance(application).getInt("mEnvModel", 0);
            return;
        }
        if (TextUtils.equals(BuildConfig.BUILD_TYPE, "debug")) {
            setEnvModel(ENV_DEBUG);
        } else if (TextUtils.equals(BuildConfig.BUILD_TYPE, "release")) {
            setEnvModel(ENV_RELEASE);
        } else if (TextUtils.equals(BuildConfig.BUILD_TYPE, "beta")) {
            setEnvModel(ENV_BETA);
        }
    }
}
```

主要操作就是，切换环境-->保存新的环境配置-->退出登陆，并清除数据-->重启App，里面有一部分操作是个人项目里面将网络基础BaseUrl和webview需要设置的环境配置的url进行重置。

## 举例: Http配置Url

```
// 其他统一的配置
        GHttp.getInstance().init(context)                           //必须调用初始化
                .setBaseUrl(ApiManager.getInstance().getBaseUrl())
                .setOkHttpClient(builder.build())               //建议设置OkHttpClient，不设置会使用默认的
                .setCacheMode(CacheMode.NO_CACHE)               //全局统一缓存模式，默认不使用缓存，可以不传
//                .setCacheTime(CacheEntity.CACHE_NEVER_EXPIRE)   //全局统一缓存时间，默认永不过期，可以不传
                .setCacheTime(1000 * 60 * 60 * 24)           //全局统一缓存时间，默认永不过期，可以不传
                .setRetryCount(3)                               //全局统一超时重连次数，默认为三次，那么最差的情况会请求4次(一次原始请求，三次重连请求)，不需要可以设置为0
                .addCommonHeaders(headers)                      //全局公共头
                .addCommonParams(params);                       //全局公共参数
```

Url配置管理类`ApiManager.java`

```
public class ApiManager {
    private static ApiManager apiManager;
    private Context mContext;

    public ApiManager(Context context) {
        mContext = context;
    }

    public static ApiManager getInstance() {
        if (apiManager == null) {
            synchronized (ApiManager.class) {
                if (apiManager == null) {
                    apiManager = new ApiManager(BaseApp.getContext());
                }
            }
        }
        return apiManager;
    }

    private String baseUrl;
    private String webUrl;
    private String taskUrl;

    private String emosUrl;

    public boolean isSetting() {
        return !TextUtils.isEmpty(SharedUtil.instance(mContext).getString("baseUrl"));
    }

    public String getBaseUrl() {
        return TextUtils.isEmpty(baseUrl) ? SharedUtil.instance(mContext).getString("baseUrl", ApiConstant.BASEURL) : baseUrl;
    }

    public String getWebURL() {
        return TextUtils.isEmpty(webUrl) ? SharedUtil.instance(mContext).getString("webUrl", ApiConstant.WEBURL) : webUrl;
    }

    public String getTaskUrl() {
        return TextUtils.isEmpty(taskUrl) ? SharedUtil.instance(mContext).getString("taskUrl", ApiConstant.TASKURL) : taskUrl;
    }

    public void setBaseUrl(String baseUrl) {
        this.baseUrl = baseUrl;
        SharedUtil.instance(mContext).saveString("baseUrl", baseUrl);
    }

    public void setTaskUrl(String taskUrl) {
        this.taskUrl = taskUrl;
        SharedUtil.instance(mContext).saveString("taskUrl", taskUrl);
    }

    public void setWebUrl(String webUrl) {
        this.webUrl = webUrl;
        SharedUtil.instance(mContext).saveString("webUrl", webUrl);
    }

    public String getEmosUrl() {
//        return emosUrl;
        return TextUtils.isEmpty(emosUrl) ? SharedUtil.instance(mContext).getString("emosUrl", ApiConstant.EOMSURL) : emosUrl;
    }

    public void setEmosUrl(String emosUrl) {
        this.emosUrl = emosUrl;
        SharedUtil.instance(mContext).saveString("emosUrl", emosUrl);
    }
}
```

这只是部分环境代码，剩下的逻辑都可以这样来处理，switch一下环境，然后处理相应的逻辑就OK，环境切换主要是为了满足公司测试、预发和线上环境的切换，比如测试测完测试环境，然后服务端发布了预发环境，直接摇一摇切换环境，直接环境就切换成了预发环境的接口链接，线上也是这种原理，然后就可以直接进行测试。

此篇over。
