# Taro小程序dva框架开发学习

本篇介绍的是Taro小程序开发，仅供学习参考。使用的框架是Taro(3.3.9)+dva+Taro UI架构开发，其中封装了dva框架来管理整个应用的state，基于Taro UI进行组建开发，封装了网络请求方法（大部分借鉴网络），网络返回值进行了处理，方便数据处理。

Taro链接:https://taro-docs.jd.com/taro/docs/README/index.html

Taro UI链接：[https://taro-ui.jd.com/#/docs/introduction](https://taro-ui.jd.com/#/docs/introduction)

Taro Demo链接(Gitee)：[TaroDvaDemo: Taro+dva demo](https://gitee.com/guomeng/taro-dva-demo.git)

Taro Demo链接(Github)：[https://github.com/coffeelife/TaroDvaDemo.git](https://github.com/coffeelife/TaroDvaDemo.git)

Taro新旧版本迁移说明：[https://taro-docs.jd.com/taro/docs/next/migration](https://taro-docs.jd.com/taro/docs/next/migration)

项目组成介绍

*   UI：Taro UI和原生组件组成

*   dva：使用redux来进行state同意管理，网络请求过程封装etc

*   http.js封装taro请求，统一处理参数和返回结果

*   版本详情

*   ```
    "dependencies": {
        "@babel/runtime": "^7.7.7",
        "@tarojs/async-await": "^2.2.10",
        "@tarojs/cli": "3.3.9",
        "@tarojs/components": "3.3.9",
        "@tarojs/react": "3.3.9",
        "@tarojs/redux-h5": "^2.2.10",
        "@tarojs/runtime": "3.3.9",
        "@tarojs/taro": "3.3.9",
        "babel-runtime": "^6.26.0",
        "dva-core": "^1.6.0-beta.7",
        "dva-loading": "^3.0.22",
        "jsrsasign": "^10.4.1",
        "lodash": "4.17.15",
        "nervjs": "^1.5.7",
        "react": "^17.0.0",
        "react-dom": "^17.0.0",
        "react-redux": "^7.2.5",
        "redux": "^4.1.1",
        "redux-logger": "^3.0.6",
        "redux-thunk": "^2.3.0",
        "regenerator-runtime": "^0.13.9",
        "taro-ui": "^3.0.0-alpha.3",
        "vconsole": "^3.9.1"
      },
    ```

## 说明

因为平时都是使用dva+antd开发网页、dva+antd mobile开发手机网页、dva+react native开发手机app，所以在考虑这个框架的时候会在想自己如何无感知来切换到小程序开发当中，所以就像使用Taro(react) + dva的方式来开发小程序，然后Taro提供了自己的UI库Taro UI，所以顺便使用taro UI作为组件库。

## 开发前准备

taro cli工具安装(copy官网)

```
# 使用 npm 安装 CLI
$ npm install -g @tarojs/cli

# OR 使用 yarn 安装 CLI
$ yarn global add @tarojs/cli

# OR 安装了 cnpm，使用 cnpm 安装 CLI
$ cnpm install -g @tarojs/cli
```

项目初始化(copy官网)

```
taro init mypp
```

[图片上传失败...(image-17a4dd-1642501786545)]

项目目录结构

[图片上传失败...(image-d0abae-1642501786545)]

```
pages//目录页面目录
app.config,js//目录主配置文件
app.js//主入口文件
app.less//主样式文件
project.config.js//小程序配置文件
package.json依赖配置文件
```

`project.config.js`将该文件的appid改成自己的appid

```
{
    "miniprogramRoot": "dist/",
    "projectname": "myapp",
    "description": "测试app",
    "appid": "自己的appId",
    "setting": {
        "urlCheck": true,
        "es6": false,
        "postcss": false,
        "preloadBackgroundData": false,
        "minified": false,
        "newFeature": true,
        "autoAudits": false,
        "coverView": true,
        "showShadowRootInWxmlPanel": false,
        "scopeDataCheck": false,
        "useCompilerModule": false
    },
    "compileType": "miniprogram",
    "simulatorType": "wechat",
    "simulatorPluginLibVersion": {},
    "condition": {}
}
```

`npm run dev:weapp`运行查看效果

[图片上传失败...(image-5667e7-1642501786545)]

## dva框架的搭建

### 安装redux、 react-redux 等依赖包(老版)

```
npm install --save react-redux

npm install --save redux @tarojs/redux @tarojs/redux-h5 redux-thunk redux-logger
```

### 安装redux、 react-redux 等依赖包(新版)

Taro新旧版本迁移说明：[Menu](https://taro-docs.jd.com/taro/docs/next/migration)

```
使用第三方 React库#
如果你需要引入 React 相关生态的库，直接通过 npm install 安装然后引入使用即可，Taro 不会再维护类似于 taro-redux 、taro-mobx 之类的库。

// 当使用了 JSX 时，babel 会隐式地调用 React.createElement
// 因此只要你使用了 JSX，就要把 React 或 Nerv 引入
import React from 'react'
import { useSelector }  from 'react-redux'
// 如果是使用的是 Nerv
// import { useSelector }  from 'nerv-redux'
function Excited () {
  const counter = useSelector(state => state.counter)
  return ...
}
```

```
npm install --save react-redux

npm install --save redux @tarojs/redux @tarojs/redux-h5 redux-thunk redux-logger
```

### 安装dva

dva-core：封装了 redux 和 redux-saga 的一个插件

dva-loading：管理页面的 loading 状态

```
npm install --save dva-core dva-loading
```

`package.json`依赖详情

```
"dependencies": {
    "@babel/runtime": "^7.7.7",
    "@tarojs/cli": "3.3.9",
    "@tarojs/components": "3.3.9",
    "@tarojs/react": "3.3.9",
    "@tarojs/redux": "^2.2.10",
    "@tarojs/redux-h5": "^2.2.10",
    "@tarojs/runtime": "3.3.9",
    "@tarojs/taro": "3.3.9",
    "dva-core": "^1.6.0-beta.7",
    "dva-loading": "^3.0.22",
    "lodash": "4.17.15",
    "react": "^17.0.0",
    "react-dom": "^17.0.0",
    "react-redux": "^7.2.6",
    "redux": "^4.1.2",
    "redux-logger": "^3.0.6",
    "redux-thunk": "^2.4.0",
    "taro-ui": "^3.0.0-alpha.3"
  },
```

### 在项目中配置dva

在`src`目录下新建一个`utils`目录，新建一个`dva.js`文件，文件代码如下

```
import { create } from 'dva-core';
import { createLogger } from 'redux-logger';
import createLoading from 'dva-loading';

let app;
let store;
let dispatch;

function createApp(opt) {
  // redux日志
  // opt.onAction = [createLogger()];
  app = create(opt);
  app.use(createLoading({}));

  if (!global.registered) opt.models.forEach(model => app.model(model));
  global.registered = true;
  app.start();

  store = app._store;
  app.getStore = () => store;

  dispatch = store.dispatch;

  app.dispatch = dispatch;
  return app;
}

export default {
  createApp,
  getDispatch() {
    return app.dispatch;
  }
}
```

在`src`下新建目录`models`目录，里面新建一个默认的`index.js`和`model.js`文件

`index.js`

```
import model from "./model"
export default [model];
```

`model.js`

```
import * as indexApi from './service';

 export default {
   namespace: 'index',
   state: {

   },

   effects: {
     * effectsDemo(_, { call, put }) {
       const { status, data } = yield call(indexApi.demo, {});
       if (status === 'ok') {
         yield put({ type: 'save',
           payload: {
             topData: data,
           } });
       }
     },
   },

   reducers: {
     save(state, { payload }) {
       return { ...state, ...payload };
     },
   },

 };
```

在`src`下新建`service`文件夹，在目录下新建`index.js`文件

`service.js`文件

```
import httpRequest from '../../utils/http';

 export const demo = (data) => {
   return httpRequest.get("",data);
 };
```

#### app.js改造

将`utils`下的`dva.js`引入，然后使用`dva`和`model`

`app.js`

```
import "@tarojs/async-await";
import React, { Component } from "react";
import dva from "./utils/dva";
import models from "./models";
import { Provider } from "react-redux";
import "./app.less";

const dvaApp = dva.createApp({
  initialState: {},
  models: models
});
const store = dvaApp.getStore();
// 如果需要在 h5 环境中开启 React Devtools
// 取消以下注释：
// if (process.env.NODE_ENV !== 'production' && process.env.TARO_ENV === 'h5')  {
//   require('nerv-devtools')
// }

export default class App extends Component {
  config={}

  componentDidMount() {}

  componentDidShow() {}

  componentDidHide() {}

  componentDidCatchError() {}

  // this.props.children 是将要会渲染的页面
  render() {
    return (
      <Provider store={store}>
        {this.props.children}
      </Provider>
    );
  }
}
```

新建`login`和`index`目录然后新建对应的less、js和config.js文件,如图

`login`下的`index.js`文件

```
import React, { Component } from "react";
import Taro from "@tarojs/taro";
import { View, Text, Button } from "@tarojs/components";
import "./index.less";
import { connect } from "react-redux";
import { AtButton } from "taro-ui";

@connect(({ index }) => ({
  index
}))
export default class Login extends Component {
  componentDidMount() {
    console.log(this.props);
  }

  goMain = () => {
    Taro.redirectTo({ url: "/pages/index/index?a=1&b=2&c=3" });
  };

  render() {
    return (
      <View className="login-page">
        登录页面
        <Button className="loginBtn" onClick={this.goMain}>
          登录
        </Button>
      </View>
    );
  }
}
```

`index`下的`index.js`文件

```
import React, { Component } from "react";
import Taro, { getCurrentInstance } from "@tarojs/taro";
import { View, Text, OpenData, Button, Textarea } from "@tarojs/components";
import { connect } from "react-redux";
import "./index.less";

@connect(({ index }) => ({
  index
}))
export default class Index extends Component {
  state = {
    userInfo: {},
    hasUserInfo: false
  };

  componentDidMount = () => {
    Taro.hideHomeButton();
    let { router } = getCurrentInstance();
    let { a, b, c } = router.params;
    console.log(this.props, router, a, b, c);
  };

  getUserProfile = () => {
    // 推荐使用wx.getUserProfile获取用户信息，开发者每次通过该接口获取用户个人信息均需用户确认
    // 开发者妥善保管用户快速填写的头像昵称，避免重复弹窗
    Taro.getUserProfile({
      desc: "用于完善会员资料", // 声明获取用户个人信息后的用途，后续会展示在弹窗中，请谨慎填写
      success: res => {
        this.setState(
          {
            userInfo: res.userInfo,
            hasUserInfo: true
          },
          () => {
            Taro.showModal({
              title: "用户信息",
              content: JSON.stringify(res.userInfo),
              showCancel: false,
              confirmText: "确定"
            });
          }
        );
      }
    });
  };

  render() {
    let { hasUserInfo, userInfo } = this.state;
    return (
      <View className="index-page">
        <View className="userInfo">
          <OpenData className="userAvatar" type="userAvatarUrl" />
          <View className="userDetail">
            <OpenData type="userNickName" lang="zh_CN" />
            <View>
              <OpenData type="userGender" lang="zh_CN" />{" "}
              <OpenData type="userCountry" lang="zh_CN" />{" "}
              <OpenData type="userProvince" lang="zh_CN" />{" "}
              <OpenData type="userCity" lang="zh_CN" />
            </View>
          </View>
        </View>
        <Button className="userBtn" onClick={this.getUserProfile}>
          获取用户信息
        </Button>
      </View>
    );
  }
}
```

在 `index`的主文件中打印this.props就可以看到对应的state数据，这里的获取上一页传递使用过`let { router } = getCurrentInstance();let { a, b, c } = router.params;`来获取。这样dva框架就已经加入到小程序当中，就可以像使用网页开发的思维来开发小程序。

[图片上传失败...(image-9fbb73-1642501786544)]

## taro网络请求的封装

此处借鉴的是网上的demo,可以根据自己的需求来进行开发调整。主要设计请求方式get、post、put、delete的封装，拦截器的封装、返回结果的封装，统一返回`taro.request`这部分我们会做修改，上代码

`http.js`

```
import Taro from "@tarojs/taro";
import getBaseUrl from "./baseUrl";
import interceptors from "./interceptors";

interceptors.forEach(interceptorItem => Taro.addInterceptor(interceptorItem));

class httpRequest {
  baseOptions(params, method = "GET") {
    let { url, data } = params;
    const BASE_URL = getBaseUrl(url);
    let contentType = "application/json";
    contentType = params.contentType || contentType;
    const option = {
      url: BASE_URL + url, //地址
      data: data, //传参
      method: method, //请求方式
      timeout: 50000, // 超时时间
      header: {
        "content-type": contentType,
        Authorization: Taro.getStorageSync("Authorization")
      }
    };
    return Taro.request(option);
  }

  get(url, data = "") {
    let option = { url, data };
    return this.baseOptions(option);
  }

  post(url, data, contentType) {
    let params = { url, data, contentType };
    return this.baseOptions(params, "POST");
  }

  put(url, data = "") {
    let option = { url, data };
    return this.baseOptions(option, "PUT");
  }

  delete(url, data = "") {
    let option = { url, data };
    return this.baseOptions(option, "DELETE");
  }
}

export default new httpRequest()；
```

拦截器封装`interceptors.js`

```
import Taro from "@tarojs/taro"
import { pageToLogin } from "./utils"
import { HTTP_STATUS } from './config'

const customInterceptor = (chain) => {

  const requestParams = chain.requestParams

  return chain.proceed(requestParams).then(res => {
    // 只要请求成功，不管返回什么状态码，都走这个回调
    if (res.statusCode === HTTP_STATUS.NOT_FOUND) {
      return Promise.reject("请求资源不存在")

    } else if (res.statusCode === HTTP_STATUS.BAD_GATEWAY) {
      return Promise.reject("服务端出现了问题")

    } else if (res.statusCode === HTTP_STATUS.FORBIDDEN) {
      Taro.setStorageSync("Authorization", "")
      pageToLogin()
      // TODO 根据自身业务修改
      return Promise.reject("没有权限访问");

    } else if (res.statusCode === HTTP_STATUS.AUTHENTICATE) {
      Taro.setStorageSync("Authorization", "")
      pageToLogin()
      return Promise.reject("需要鉴权")

    } else if (res.statusCode === HTTP_STATUS.SUCCESS) {
      return res.data

    }
  })
}

// Taro 提供了两个内置拦截器
// logInterceptor - 用于打印请求的相关信息
// timeoutInterceptor - 在请求超时时抛出错误。
const interceptors = [customInterceptor, Taro.interceptors.logInterceptor]

export default interceptors
```

获取链接的方式`baseUrl.js`

```
const getBaseUrl = (url) => {
  let BASE_URL = '';
  if (process.env.NODE_ENV === 'development') {
    //开发环境 - 根据请求不同返回不同的BASE_URL
    if (url.includes('/api/')) {
      BASE_URL = ''
    } else if (url.includes('/iatadatabase/')) {
      BASE_URL = ''
    }
  } else {
    // 生产环境
    if (url.includes('/api/')) {
      BASE_URL = ''
    } else if (url.includes('/iatadatabase/')) {
      BASE_URL = ''
    }
  }
  return BASE_URL
}

export default getBaseUrl;
```

请求状态文件`config.js`

```
export const HTTP_STATUS = {
  SUCCESS: 200,
  CREATED: 201,
  ACCEPTED: 202,
  CLIENT_ERROR: 400,
  AUTHENTICATE: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  SERVER_ERROR: 500,
  BAD_GATEWAY: 502,
  SERVICE_UNAVAILABLE: 503,
  GATEWAY_TIMEOUT: 504
}
```

叮叮，这样整体的封装就完成了

## 扩展

### 添加分包

[图片上传失败...(image-7e87db-1642501786544)]

`app.config.js`

```
export default {
  pages: [
    'pages/login/index',
    'pages/index/index'
  ],
  subPackages: [
    { root: "modules", pages: ["test/index", "detail/index"] }
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: 'WeChat',
    navigationBarTextStyle: 'black'
  }
}
```

### http返回修改

结果返回从promise修改为现在直接返回结果

`http.js`新

```
import Taro from "@tarojs/taro";
import getBaseUrl from "./baseUrl";
import { HTTP_STATUS } from "./config";
import { STORAGE_DATA } from "./constant";
import interceptors from "./interceptors";
import { createNonceSt, MD5withRSA, pageToLogin, signjs } from "./utils";

interceptors.forEach(interceptorItem => Taro.addInterceptor(interceptorItem));

class httpRequest {
  baseOptions(params, method = "GET") {
    let { url, data } = params;
    //添加token 随机数 sign
    if (Taro.getStorageSync(STORAGE_DATA.TOKEN))
      data.token = Taro.getStorageSync(STORAGE_DATA.TOKEN);
    if (!data.retrieveIntervalTime) data.retrieveIntervalTime = 60000;
    if (!data.retrieveTime) data.retrieveTime = new Date().getTime();

    data.nonce = createNonceSt();
    data.sign = MD5withRSA(signjs(data));
    const BASE_URL = getBaseUrl(url);
    let contentType = "application/json";
    contentType = params.contentType || contentType;
    const option = {
      url: BASE_URL + url, //地址
      data: data, //传参
      method: method, //请求方式
      timeout: 50000, // 超时时间
      header: {
        "content-type": contentType,
        Authorization: `Bearer ${Taro.getStorageSync(STORAGE_DATA.TOKEN)}`
      }
    };
    // return Taro.request(option);
    if (process.env.NODE_ENV === "development") {
      console.log(
        `${new Date().toLocaleString()}【 M=${option.url} 】P=${JSON.stringify(
          option.data
        )}`
      );
    }
    return Taro.request(option)
      .then(res => {
        console.log("resres", res);
        if (res) {
          if (
            res.code >= HTTP_STATUS.SUCCESS &&
            res.code < HTTP_STATUS.MULTI_SELECT
          ) {
            if (process.env.NODE_ENV === "development") {
              console.log(
                `${new Date().toLocaleString()}【 M=${
                  option.url
                } 】【接口响应：】`,
                res
              );
            }
            return res;
          } else {
            throw new Error(
              `${res.desc || "网络请求错误，状态码"}(${res.code})`
            );
          }
        } else {
          throw new Error(`数据返回异常，请重试`);
        }
      })
      .catch(error => {
        console.log("http error", error);
        Taro.showToast({ title: error.message || "服务异常", icon: "none" });
        // pageToLogin()
        // return null;
      });
  }

  get(url, data = "") {
    let option = { url, data };
    return this.baseOptions(option);
  }

  post(url, data, contentType) {
    let params = { url, data, contentType };
    return this.baseOptions(params, "POST");
  }

  put(url, data = "") {
    let option = { url, data };
    return this.baseOptions(option, "PUT");
  }

  delete(url, data = "") {
    let option = { url, data };
    return this.baseOptions(option, "DELETE");
  }
}

export default new httpRequest();
```

### 验签过程添加

//公共验签

1 公共验签方法是用于生成访问api的sign签的一个公共方法，生成的sign将用于api接口有效性与权限的验证，以保证接口互通的安全性。

2 签名算法：

签名生成的通用步骤如下：

第一步，设所有发送或者接收到的数据为集合M（不包含sign），将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

特别注意以下重要规则：

1.◆ 参数名ASCII码从小到大排序（字典序）；

2.◆ 如果参数的值为空不参与签名；

3.◆ 参数名区分大小写；

4.◆ 验证调用返回或主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。

5.◆ 接口可能增加字段，验证签名时必须支持增加的扩展字段

第二步，在stringA最后拼接上apikey得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到signMD5，对signMD5值进行RSA运算得到sign。

举例：

假设传送的参数如下：

appid： wxd930ea5d5a258f4f

mch_id： 10000100

device_info： 1000

body： test

nonce： ibuaiVcKdpRxkhJA

第一步：对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：

stringA="appid=wxd930ea5d5a258f4f&body=test&device_info=1000&mch_id=10000100&nonce=ibuaiVcKdpRxkhJA";

第二步：拼接API密钥：

MD5签名方式：

stringSignTemp=stringA+"&key=192006250b4c09247ec02edce69f6a2d" //注：key为用户的userCode

signMD5=MD5(stringSignTemp).toUpperCase()="9A0A8659F005D6984697E2CA0A9CF3B7" //注：MD5签名方式

RSA签名方式：

sign=RSA(signMD5,appSecret)="6A9AE1657590FD6257D693A078E1C3E4BB6BA4DC30B23E0EE2496E54170DACD6"

注：

1 RSA签名方式中的加密算法为RSA，签名算法为MD5withRSA

2生成随机数算法

API接口协议中包含字段nonce，主要保证签名不可预测。我们推荐生成随机数算法如下：调用随机数函数生成，将得到的值转换为字符串。

2.1.3字段介绍：

nonce 随机数

key 为用户的userCode

`utils.js`中的`MD5withRSA`、`signjs`、`sortObj`、`createNonceSt`为封装的验签加密算法（已验证可以使用）

```
import Taro from "@tarojs/taro";
import md5 from "./md5js";
import { STORAGE_DATA } from "./constant";
import { hextob64, KEYUTIL, KJUR, RSAKey } from "jsrsasign";
/**
 * @description 获取当前页url
 */
export const getCurrentPageUrl = () => {
  let pages = Taro.getCurrentPages();
  let currentPage = pages[pages.length - 1];
  let url = currentPage.route;
  return url;
};

export const pageToLogin = () => {
  let path = getCurrentPageUrl();
  Taro.clearStorageSync();
  if (!path.includes("login")) {
    Taro.reLaunch({
      url: "/pages/login/index"
    });
  }
};

/** 对象转url参数 */
export const stringify = params => {
  const obj = typeof params === "object" && params !== null ? params : {};
  const isEmpty = v => v === "" || v === null || v === undefined;
  return Object.keys(obj)
    .filter(k => !isEmpty(obj[k]))
    .map(key => `${key}=${encodeURIComponent(obj[key])}`)
    .join("&");
};

export const repeat = (str = "0", times) => new Array(times + 1).join(str);
// 时间前面 +0
export const pad = (num, maxLength = 2) =>
  repeat("0", maxLength - num.toString().length) + num;
/** 时间格式的转换 */
export const formatTime = time =>
  `${pad(time.getHours())}:${pad(time.getMinutes())}:${pad(
    time.getSeconds()
  )}.${pad(time.getMilliseconds(), 3)}`;

export var globalData = {}; // 全局公共变量

// 使用es6的padStart()方法来补0
export const getYMDHMS = timestamp => {
  let time = timestamp ? new Date(timestamp) : new Date();
  let year = time.getFullYear();
  const month = (time.getMonth() + 1).toString().padStart(2, "0");
  const date = time
    .getDate()
    .toString()
    .padStart(2, "0");
  const hours = time
    .getHours()
    .toString()
    .padStart(2, "0");
  const minute = time
    .getMinutes()
    .toString()
    .padStart(2, "0");
  const second = time
    .getSeconds()
    .toString()
    .padStart(2, "0");

  return (
    year +
    "年" +
    month +
    "月" +
    date +
    "日 " +
    hours +
    "时" +
    minute +
    "分" +
    second +
    "秒"
  );
};

export function getFileName(filePath) {
  if (!filePath) return null;
  let index = filePath.lastIndexOf("/");
  const name = filePath.substring(index + 1);
  return name;
}

// 使用es6的padStart()方法来补0
export const getYMDHMS2 = timestamp => {
  let time = timestamp ? new Date(timestamp) : new Date();
  let year = time.getFullYear();
  const month = (time.getMonth() + 1).toString().padStart(2, "0");
  const date = time
    .getDate()
    .toString()
    .padStart(2, "0");
  const hours = time
    .getHours()
    .toString()
    .padStart(2, "0");
  const minute = time
    .getMinutes()
    .toString()
    .padStart(2, "0");
  const second = time
    .getSeconds()
    .toString()
    .padStart(2, "0");

  return (
    year + "-" + month + "-" + date + " " + hours + ":" + minute + ":" + second
  );
};

export function dateStr() {
  let date = new Date();
  if (date.getHours() >= 0 && date.getHours() < 12) {
    return "上午好";
  } else if (date.getHours() >= 12 && date.getHours() < 18) {
    return "下午好";
  } else {
    return "晚上好";
  }
}

/**
 * 校验手机号是否正确
 * @param phone 手机号
 */

export const verifyPhone = phone => {
  const reg = /^1[0-9]{10}$/;
  const _phone = phone.toString().trim();
  let toastStr =
    _phone === ""
      ? "手机号不能为空~"
      : !reg.test(_phone) && "请输入正确手机号~";
  return {
    errMsg: toastStr,
    done: !toastStr,
    value: _phone
  };
};

export const verifyStr = (str, text) => {
  const _str = str.toString().trim();
  const toastStr = _str.length ? false : `请填写${text}～`;
  return {
    errMsg: toastStr,
    done: !toastStr,
    value: _str
  };
};

// 截取字符串

export const sliceStr = (str, sliceLen) => {
  if (!str) {
    return "";
  }
  let realLength = 0;
  const len = str.length;
  let charCode = -1;
  for (var i = 0; i < len; i++) {
    charCode = str.charCodeAt(i);
    if (charCode >= 0 && charCode <= 128) {
      realLength += 1;
    } else {
      realLength += 2;
    }
    if (realLength > sliceLen) {
      return `${str.slice(0, i)}...`;
    }
  }

  return str;
};

/**
 * JSON 克隆
 * @param {Object | Json} jsonObj json对象
 * @return {Object | Json} 新的json对象
 */
export function objClone(jsonObj) {
  var buf;
  if (jsonObj instanceof Array) {
    buf = [];
    var i = jsonObj.length;
    while (i--) {
      buf[i] = objClone(jsonObj[i]);
    }
    return buf;
  } else if (jsonObj instanceof Object) {
    buf = {};
    for (var k in jsonObj) {
      buf[k] = objClone(jsonObj[k]);
    }
    return buf;
  } else {
    return jsonObj;
  }
}

export function MD5withRSA(signMd5) {
  if (!Taro.getStorageSync(STORAGE_DATA.PRIVATEKEY)) return;
  const pem = `-----BEGIN PRIVATE KEY-----${Taro.getStorageSync(
    STORAGE_DATA.PRIVATEKEY
  )}-----END PRIVATE KEY-----`;

  let key = KEYUTIL.getKey(pem);
  // 创建 Signature 对象
  let signature = new KJUR.crypto.Signature({ alg: "MD5withRSA" });
  // 传入key实例, 初始化signature实例
  signature.init(key);
  // 传入待签明文
  signature.updateString(signMd5);
  const signInfo = signature.sign();
  const sign = hextob64(signInfo);
  console.log("signature:", signInfo, sign);
  // 签名, 得到16进制字符结果
  return sign;
}

// 支付md5加密获取sign
export function signjs(jsonobj) {
  var signstr = obj2str(jsonobj);
  if (Taro.getStorageSync(STORAGE_DATA.USERID)) {
    signstr = signstr + "&key=" + Taro.getStorageSync(STORAGE_DATA.USERID);
  }
  var sign = md5(signstr); //验证调用返回或微信主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。
  console.log("signstrkey:", signstr, signstr.length, sign, sign.toUpperCase());
  return sign.toUpperCase();
}

export function sortObj(obj) {
  if (obj instanceof Array) {
    let newArr = [];
    newArr = obj.map((item, index) => {
      return sortObj(item);
    });
    return newArr;
  } else if (obj instanceof Object) {
    let keys = Object.keys(obj);
    if (!(keys && keys.length > 0)) return null;
    keys = keys.sort(); //参数名ASCII码从小到大排序（字典序）；
    let newObj = {};
    keys.forEach(function(key) {
      if (obj[key] !== "" && obj[key] !== undefined && obj[key] !== null) {
        //如果参数的值为空不参与签名；
        newObj[key] = sortObj(obj[key]); //参数名区分大小写；
      }
    });
    return newObj;
  } else {
    return obj;
  }
}

//object转string,用于签名计算
export function obj2str(args) {
  if (!args) return;
  let newArgs = sortObj(args);
  console.log("sortObj\n", args, "\n", newArgs);
  let string = "";
  for (let k in newArgs) {
    string +=
      "&" +
      k +
      "=" +
      (typeof newArgs[k] == "object" ? JSON.stringify(newArgs[k]) : newArgs[k]);
  }
  string = string.substr(1);
  return string;
}

//随机函数的产生：
export function createNonceSt() {
  return Math.random()
    .toString(36)
    .substr(2, 15); //随机小数，转换36进制，去掉0.，保留余下部分
}

//时间戳产生的函数, 当前时间以证书表达，精确到秒的字符串
export function createTimeStamp() {
  return parseInt(new Date().getTime() / 1000) + "";
}

export function sort_ASCII(obj) {
  var arr = new Array();
  var num = 0;
  for (var i in obj) {
    arr[num] = i;
    num++;
  }
  var sortArr = arr.sort();
  var sortObj = {};
  for (var i in sortArr) {
    sortObj[sortArr[i]] = obj[sortArr[i]];
  }
  return sortObj;
}

/** 检查更新 */
export function checkForUpdate() {
  if (Taro.canIUse("getUpdateManager")) {
    const updateManager = Taro.getUpdateManager();
    updateManager.onCheckForUpdate(res => {
      // 请求完新版本信息的回调
      if (res.hasUpdate) {
        updateManager.onUpdateReady(() => {
          Taro.showModal({
            title: "更新提示",
            content: "新版本已经准备好，是否重启应用？",
            showCancel: false,
            confirmColor: "#40A3FF",
            success: res => {
              if (res.confirm) {
                // 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
                updateManager.applyUpdate();
              }
            }
          });
        });

        updateManager.onUpdateFailed(() => {
          // 新版本下载失败
          Taro.showModal({
            title: "更新提示",
            content: "新版本已经上线，请删除当前小程序，重新扫码进入",
            confirmColor: "#40A3FF",
            showCancel: false
          });
        });
      }
    });
  }
}
```
