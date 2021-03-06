# 域名绑定指南

根据法律法规和有关部门的要求，使用 LeanCloud 服务需要绑定自有域名。
绑定自有域名也有利于从域名层面做好应用隔离，确保业务稳定。

这篇指南假定你了解域名解析的基本知识，如果你不太熟悉这方面的内容，可以参考我们博客上的[这篇文章][basic]。

[basic]: https://leancloudblog.com/Domain-Name-Story-confirm/

## 自定义域名的种类

LeanCloud 服务涉及以下三种自定义域名：

| 域名种类 | 涉及服务 | 备案 | 在 LeanCloud 备案或接入备案 | SSL | 裸域名 | A 记录 | 绑定目标 | 可绑定多个域名 | 
| - | - | - | - | - | - | - | - | - | 
| 文件域名 | 文件服务 | 必须 | 可选 | 可选 | 不支持 | 不支持 | 应用 | 否 |
| 云引擎域名 | 云引擎 **网站托管** | 必须 | 必须 | 可选 | 支持 | 支持 | 分组 | 是 |
| API 域名 | 存储、即时通讯、短信、推送、**云函数** | 必须 | 必须 | 必须 | 不支持 | 不支持 | 应用 | 是 |

说明：

0. 每个域名只能绑定到一个应用的一种服务，可以使用同一主域名下不同的子域名。
1. API 域名需要启用 SSL，绑定时会自动申请 SSL 证书，当然你也可以自行上传证书。
2. 为了避免影响同一域名下的其他子域名的解析，文件域名和 API 域名不支持直接绑定裸域名（例如 `example.com`）。
3. 部分用户希望云引擎网站托管的 URL 尽量简短，因此有绑定裸域名的需求。我们推荐使用 ANAME 或 CNAME Flattening 记录在云引擎网站托管服务上绑定裸域名。
4. 如果域名服务商不支持 ANAME 或 CNAME Flattening 记录，云引擎网站托管服务也支持使用 A 记录直接绑定裸域名。由于 IP 可能发生变动，因此我们不建议使用 A 记录绑定云引擎自定义域名。
5. 绑定目标中的「分组」指应用下的云引擎实例分组的生产环境。
6. 可绑定多个域名指在一个应用（文件域名、API 域名）或分组（云引擎域名）上可以绑定多个域名。

## 文件域名

如果你的应用使用了 LeanCloud 文件服务，请前往 **控制台 > 存储 > 设置 > 自定义文件域名** 绑定文件域名。

注意，即使您并未使用 LeanCloud 的结构化数据存储服务，但是使用了即时通讯的多媒体消息（图像、音频、视频等），那么就有可能使用了文件服务。
一个简单的判断方法是到控制台查看你的应用的 **存储 > 结构化数据** 页面，如果 `_File` 表中有数据，就表明你的应用使用了文件服务。

绑定文件域名时，可以选择是否启用 HTTPS：

- 不启用 HTTPS，则客户端只能通过 HTTP 访问。

- 启用 HTTPS 后，客户端同时可以通过 HTTPS 和 HTTP 访问文件。但是，受限于文件服务提供商，无论客户端使用 HTTP URL 还是 HTTPS URL 访问：

  - 启用 HTTPS 域名的自定义文件域名的流量均按照 HTTPS 流量计费。
  - 同理，应用控制台的文件流量统计也总是归入 HTTPS 流量。

更换文件域名后，之前保存在 `_File` 表中的文件 URL 会自动更新。
即时通讯历史消息（包括富媒体消息）中的 URL 不会自动更新。类似地，如果开发者把文件 URL 单独保存在别的地方，更换文件域名后，需要在客户端自行实现相应的替换逻辑。
因此，我们建议开发者在开始使用文件服务和即时通讯服务时就绑定自定义文件域名，以免给以后迁移增加困难。

之前使用 LeanCloud 公共文件域名的旧应用，在绑定文件域名后，现存的使用共享域名的 URL 仍然可以访问。

如果在不同应用间绑定了 `_File` Class，那么需要在源应用绑定自定义文件域名，目标应用无需绑定自定义文件域名。

由于底层文件服务商的限制，如果你在底层文件服务商（七牛）处有账号，并且在自己的七牛账号绑定了泛域名（例如 `*.example.com`），那么该域名（`example.com`）下的所有子域名均无法绑定 LeanCloud 文件服务。
如在七牛取消泛域名绑定，下一个账期以后可以绑定该域名下的子域名至 LeanCloud 文件服务。

目前国际版不支持绑定自定义文件域名。

## 云引擎域名

使用云引擎网站托管服务的应用，需要前往**应用控制台 > 账号设置 > 域名绑定**绑定云引擎域名。

如前所述，仅使用云函数（包括 hook 函数）的应用，无需绑定云引擎域名，不过需要绑定 API 域名。

控制台绑定域名时，可以自动申请 SSL 证书。相应地，`/.well-known/acme-challenge/` 路径用于验证，开发者无法使用该路径。

`stg-` 开头的自定义域名（例如 `stg-web.example.com`）会被自动地绑定到预备环境。

国际版可以在「控制台 > 云引擎 > 设置」配置 `avosapps.us` 子域名，也可以绑定云引擎自定义域名（不要求备案）。

## API 域名

如果你使用了以下服务：

- 结构化数据存储
- 云函数（包括 hook 函数）
- 即时通讯
- 多人在线对战、排行榜

那么你需要前往**应用控制台 > 设置 > 域名绑定** 绑定 API 域名。

只使用推送和短信功能的应用目前不强制要求绑定域名，但我们仍然建议用户绑定自己的 API 域名，以免受到共享域名可用性的影响。

已绑定自有域名的应用，旧版本客户端仍可继续访问原来由 LeanCloud 提供的域名，但我们不对共享域名的可用性做保证。

处于开发测试阶段的应用，可以使用我们提供的测试域名，这个域名仅供测试使用，可能被回收。
正式上线的应用请绑定自有域名。
测试域名可以在「控制台 > 设置 > 应用 Keys > 服务器地址」查看。

目前，绑定 API 域名后，即时通讯，Live Query 以及多人对战的 WebSocket 连接暂时仍会使用共享域名。我们后续会支持这类 WebSocket 连接使用自有域名。另外，用户反馈组件暂时也仍然使用共享域名。

国际版不要求绑定 API 自定义域名。
如果绑定，也不要求备案。

### 更新代码

绑定自有 API 域名后，需要更新客户端代码以使用自定义域名。

以下部分假定绑定的自定义域名是 `xxx.example.com`，且开启了 HTTPS。

#### REST API

参考 [REST API 使用详解](rest_api.html#base-url) 配置。

#### JavaScript SDK

##### 存储 SDK

请参考 [SDK 安装指南](sdk_setup-js.html#安装与引用 SDK) 配置。

旧版本的 SDK 请参考以下方法配置（建议使用最新版本的 SDK）：

<details>

<p><code>&gt;=3.5.5, &lt;3.11.1</code> 的版本可能会碰到仍然使用缓存默认配置的 bug，它可能会导致更新后的第一次请求失败</p>

<pre><code>
AV.init({
  // appId, appKey,
  serverURLs: 'https://xxx.example.com',
});
</code></pre>

<p><code>&gt;= 3.0.0, &lt;3.5.5</code></p>

<pre><code>
AV.init({
  // appId, appKey,
  serverURLs: {
    push: 'https://xxx.example.com',
    stats: 'https://xxx.example.com',
    engine: 'https://xxx.example.com',
    api: 'https://xxx.example.com',
  },
});
</code></pre>

<p><code>&lt;3.0.0</code> 的即时通讯 SDK 不支持自定义域名。</p>
</details>



##### 即时通讯 SDK

请参考 [即时通讯开发指南](realtime-guide-beginner.html#创建 IMClient) 配置。

旧版本的 SDK 请参考以下方法配置（建议使用最新版本的 SDK）：

<details>

<p><code>&gt;=4.0.0, &lt;=4.3.1</code> 的即时通讯 SDK 的 server 参数只能填写域名（不含协议），不支持未启用 HTTPS 的自定义域名：</p>

<pre><code>
new Realtime({
  // appId, appKey,
  server: 'xxx.example.com',
};
<code></pre>


<p><code>&lt;4.0.0</code> 的即时通讯 SDK 不支持自定义域名。</p>

<p>如果使用了 LiveQuery 功能，建议使用<code>&gt;=3.14.0</code> 的存储 SDK。
旧版本（<code>&gt;=3.5.0, &lt;=3.13.2</code>）的 SDK 还需要在初始化的时候额外配置 LiveQuery 模块的域名：</p> 

<pre><code>
AV.init({
  // appId, appKey,
  // serverURLs,
  realtime: new AV._sharedConfig.liveQueryRealtime({
    appId,
    appKey,
    server: 'xxx.example.com',
  }),
});
<code></pre>

<p><code>&gt;=3.5.0, &lt;3.13.2</code> 的 SDK 不支持未启用 HTTPS 的自定义域名。</p>

<p><code>&lt;3.5.0</code> 存储 SDK 的 LiveQuery 不支持自定义域名。</p>

</details>

##### 多人在线对战

请参考 [入门指南](multiplayer-quick-start-js.html#初始化) 或 [开发指南](multiplayer-guide-js.html#初始化) 进行配置。

##### 微信小程序白名单

前往 [LeanCloud 控制台 > 设置 > 应用 Keys > 域名白名单][weapp-domains]，获取域名白名单（不同应用对应不同的域名）。

[weapp-domains]: https://leancloud.cn/dashboard/app.html?appid={{appid}}#/key

登录[微信公众平台]，前往 **设置 > 开发设置 > 服务器配置 > 「修改」** 链接，**增加**上述域名白名单中的域名。

[微信公众平台]: https://mp.weixin.qq.com

#### Objective-C SDK

请参考 [SDK 安装指南](sdk_setup-objc.html#初始化) 配置。

`<12.0.0` 的版本请参考以下方法配置：

<details>
<pre><code>
// 配置 SDK 储存
[AVOSCloud setServerURLString:@"https://xxx.example.com" forServiceModule:AVServiceModuleAPI];
// 配置 SDK 推送
[AVOSCloud setServerURLString:@"https://xxx.example.com" forServiceModule:AVServiceModulePush];
// 配置 SDK 云引擎（用于访问云函数，使用 API 自定义域名，而非云引擎自定义域名）
[AVOSCloud setServerURLString:@"https://xxx.example.com" forServiceModule:AVServiceModuleEngine];
// 配置 SDK 即时通讯
[AVOSCloud setServerURLString:@"https://xxx.example.com" forServiceModule:AVServiceModuleRTM];
// 配置 SDK 统计
[AVOSCloud setServerURLString:@"https://xxx.example.com" forServiceModule:AVServiceModuleStatistics];
// 初始化应用
[AVOSCloud setApplicationId:@"{{appid}}" clientKey:@"{{appkey}}"];
</code></pre>

<strong>部分旧版本 SDK（&lt; 8.2.3）存在 SSL Pinning，它可能导致配置后的自定义服务器地址无法使用，如果出现了「证书非法」的相关错误，请至少升级 SDK 到 8.2.3，建议升级至最新版。</strong>
</details>

`<4.6.0` 的版本不支持自定义域名。

#### Swift SDK

`>= 17.0.0` 的版本请参考 [SDK 安装指南](sdk_setup-swift.html#初始化) 配置。

`>= 16.1.0, < 17.0.0` 的版本请参考以下方法配置：

<details>
<pre><code>
let configuration = LCApplication.Configuration(
    customizedServers: [
        .api("https://xxx.example.com"),
        .engine("https://xxx.example.com"),
        .push("https://xxx.example.com"),
        .rtm("https://xxx.example.com")
    ]
)
do {
    try LCApplication.default.set(
        id: "{{appid}}",
        key: "{{appkey}}",
        configuration: configuration
    )
} catch {
    fatalError("\(error)")
}
</code></pre>

</details>

`<16.1.0` 的版本不支持自定义域名。

#### Java Unified SDK

Java Unified SDK （ `>= 6.0.0`） 请参考 [SDK 安装指南](sdk_setup-java.html#初始化) 配置。 

旧版本 SDK 请参考以下方法配置：


<details>

使用 Java Unified SDK (`< 6.0.0`) 的 Android 项目，需在 `Application` 类的 `onCreate` 方法添加：

<pre><code>
import cn.leancloud.AVOSCloud;

public class MyLeanCloudApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // 配置 SDK 储存
        AVOSCloud.setServer(AVOSService.API, "https://xxx.example.com");
        // 配置 SDK 云引擎（用于访问云函数，使用 API 自定义域名，而非云引擎自定义域名）
        AVOSCloud.setServer(AVOSService.ENGINE, "https://xxx.example.com");
        // 配置 SDK 推送
        AVOSCloud.setServer(AVOSService.PUSH, "https://xxx.example.com");
        // 配置 SDK 即时通讯
        AVOSCloud.setServer(AVOSService.RTM, "https://xxx.example.com"); 

        // 提供 this、App ID 和 App Key 作为参数
        // 注意这里千万不要调用 cn.leancloud.core.AVOSCloud 的 initialize 方法，否则会出现 NetworkOnMainThread 等错误。
        AVOSCloud.initialize(this, "{{appid}}", "{{appkey}}");
    }
}
</code></pre>

老的 Android SDK 请参考以下方法配置：

<pre><code>
// 配置 SDK 储存
AVOSCloud.setServer(AVOSCloud.SERVER_TYPE.API, "https://xxx.example.com");
// 配置 SDK 云引擎
AVOSCloud.setServer(AVOSCloud.SERVER_TYPE.ENGINE, "https://xxx.example.com");
// 配置 SDK 推送
AVOSCloud.setServer(AVOSCloud.SERVER_TYPE.PUSH, "https://xxx.example.com");
// 配置 SDK 即时通讯
AVOSCloud.setServer(AVOSCloud.SERVER_TYPE.RTM, "https://xxx.example.com");
// 初始化应用
AVOSCloud.initialize(this, "{{appid}}", "{{appkey}}");
</code></pre>

<p><code>&lt;4.4.4</code> 的 Android SDK 不支持自定义域名。</p>

</details>

#### .NET SDK

请参考 [SDK 安装指南](sdk_setup-dotnet.html#初始化) 配置。 

`< v20190925.1` 的版本请参考以下方法配置：

<details>
<pre><code>
AVClient.Initialize(new AVClient.Configuration {
                ApplicationId = "{{appid}}",
                ApplicationKey = "{{appkey}}",
                ApiServer = new Uri("https://xxx.example.com"),
                EngineServer = new Uri("https://xxx.example.com"),
                PushServer = new Uri("https://xxx.example.com")
});
<code></pre>
</details>

#### PHP & Python SDK

注意，云引擎内部访问 API 是通过内网，所以不需要也不应该配置 API 自定义域名。
模板项目和云引擎网站托管开发指南中的示例代码均未配置 API 自定义域名，请勿设置自定义域名，以免变成公网访问，影响性能。

在云引擎以外的服务端使用 PHP 或 Python SDK，可以不配置 API 自定义域名。

#### App ID 后缀不为 `-MdYXbMMI` 的国际版应用如何初始化 SDK

国际版应用不要求绑定自定义域名。
相应地，如果你的国际版应用没有绑定自定义域名，初始化 SDK 时也就不用配置自定义域名。
但是，有极个别国际版应用的 App ID 后缀不为 `-MdYXbMMI`（包括个别老应用以及进行过特殊配置迁移到国际版的应用），初始化 SDK 时不配置自定义域名可能会报错
（因为较新版本的 SDK 增加了自定义域名配置参数的检查，检查时会根据 App ID 后缀判断是否为国际版应用）。
如果你的国际版应用 App ID 后缀不为 `-MdYXbMMI`，那么我们建议你绑定一个自定义域名，然后在初始化 SDK 时配置自定义域名，这样可以避免这个问题。

如果你的国际版应用 App ID 后缀不为 `-MdYXbMMI`，但因为某些原因无法绑定域名，那么需要这样初始化 SDK：

<details>

<p>注意，请使用你的应用 App ID 的前 8 位替换以下 url 地址中的 <code>aaaaaaaa</code>。<p>

<p><strong>JavaScript 存储</strong></p>

<pre><code>
AV.init({
  // appId, appKey,
  serverURLs: {
    push: 'https://aaaaaaaa.push.lncldglobal.com',
    stats: 'https://aaaaaaaa.stats.lncldglobal.com',
    engine: 'https://aaaaaaaa.engine.lncldglobal.com',
    api: 'https://aaaaaaaa.api.lncldglobal.com',
  },
});
</code></pre>

<p><strong>JavaScript 即时通讯</strong></p>

<p>请参考 <a href="realtime-guide-beginner.html#创建_IMClient">即时通讯开发指南</a> 配置。
其中 <code>server</code> 参数的值为 <code>aaaaaaaa.rtm.lncldglobal.com</code>。</p>

<p><strong>JavaScript 多人在线对战</strong></p>

<p>请参考 <a href="multiplayer-quick-start-js.html#初始化">入门指南</a> 或 <a href="multiplayer-guide-js.html#初始化">开发指南</a> 进行配置。
其中 <code>playServer</code> 参数的值为 <code>aaaaaaaa.play.lncldglobal.com</code>。</p>

<p><strong>Objective-C</strong></p>

<pre><code>
// 配置 SDK 储存
[AVOSCloud setServerURLString:@"https://aaaaaaaa.api.lncldglobal.com" forServiceModule:AVServiceModuleAPI];
// 配置 SDK 推送
[AVOSCloud setServerURLString:@"https://aaaaaaaa.push.lncldglobal.com" forServiceModule:AVServiceModulePush];
// 配置 SDK 云引擎（用于访问云函数，使用 API 自定义域名，而非云引擎自定义域名）
[AVOSCloud setServerURLString:@"https://aaaaaaaa.engine.lncldglobal.com" forServiceModule:AVServiceModuleEngine];
// 配置 SDK 即时通讯
[AVOSCloud setServerURLString:@"https://aaaaaaaa.rtm.lncldglobal.com" forServiceModule:AVServiceModuleRTM];
// 配置 SDK 统计
[AVOSCloud setServerURLString:@"https://aaaaaaaa.stats.lncldglobal.com" forServiceModule:AVServiceModuleStatistics];
// 初始化应用
[AVOSCloud setApplicationId:@"{{appid}}" clientKey:@"{{appkey}}"];
</code></pre>

<p><strong>Swift</strong></p>

<pre><code>
let configuration = LCApplication.Configuration(
    customizedServers: [
        .api("https://aaaaaaaa.api.lncldglobal.com"),
        .engine("https://aaaaaaaa.engine.lncldglobal.com"),
        .push("https://aaaaaaaa.push.lncldglobal.com"),
        .rtm("https://aaaaaaaa.rtm.lncldglobal.com")
    ]
)
do {
    try LCApplication.default.set(
        id: "{{appid}}",
        key: "{{appkey}}",
        configuration: configuration
    )
} catch {
    fatalError("\(error)")
}
</code></pre>

<p><strong>Java</strong></p>

<pre><code>
import cn.leancloud.AVOSCloud;

public class MyLeanCloudApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // 配置 SDK 储存
        AVOSCloud.setServer(AVOSService.API, "https://aaaaaaaa.api.lncldglobal.com");
        // 配置 SDK 云引擎（用于访问云函数，使用 API 自定义域名，而非云引擎自定义域名）
        AVOSCloud.setServer(AVOSService.ENGINE, "https://aaaaaaaa.engine.lncldglobal.com");
        // 配置 SDK 推送
        AVOSCloud.setServer(AVOSService.PUSH, "https://aaaaaaaa.push.lncldglobal.com");
        // 配置 SDK 即时通讯
        AVOSCloud.setServer(AVOSService.RTM, "https://aaaaaaaa.rtm.lncldglobal.com"); 

        // 提供 this、App ID 和 App Key 作为参数
        // 注意这里千万不要调用 cn.leancloud.core.AVOSCloud 的 initialize 方法，否则会出现 NetworkOnMainThread 等错误。
        AVOSCloud.initialize(this, "{{appid}}", "{{appkey}}");
    }
}
</code></pre>

<p><strong>.NET</strong></p>

<pre><code>
AVClient.Initialize(new AVClient.Configuration {
                ApplicationId = "{{appid}}",
                ApplicationKey = "{{appkey}}",
                ApiServer = new Uri("https://aaaaaaaa.api.lncldglobal.com"),
                EngineServer = new Uri("https://aaaaaaaa.engine.lncldglobal.com"),
                PushServer = new Uri("https://aaaaaaaa.push.lncldglobal.com")
});
<code></pre>


</details>

## CNAME 设置

绝大部分情况下，绑定域名均需要设置 CNAME，因此这里我们简单解释下如何设置 CNAME。

绑定域名过程中，需要设置域名的 CNAME。控制台会提示 CNAME 的地址，按照控制台的提示到域名注册商或域名解析服务提供商处设置（如果自己架设域名解析服务器的话，请根据域名解析服务器文档配置）。

例如，按控制台提示输入待绑定域名 `xxx.example.com` 后，控制台会首先检查备案，检查通过后， 会显示「等待配置 CNAME」及 CNAME 值。
假设控制台显示「CNAME: yyy.zzz.com」：

![控制台域名绑定界面](images/dashboard-domain-setup.png)

那么对应的 DNS Zone 记录为：

```
xxx.example.com.	3600	IN	CNAME	yyy.zzz.com.
```

其中 3600 为 TTL，可根据自己的需要设置。
大多数域名注册商或域名解析服务提供商都提供图形化的设置界面，按照其说明配置即可。

![以 DnsPod 添加 CNAME 记录为例](images/dnspod-add-cname-record.png)

设置完成后，需要等待一段时间，CNAME 记录生效后，LeanCloud 控制台会显示「已绑定」。

如果长时间卡在「等待配置 CNAME 阶段」，那么请点击「等待配置 CNAME 阶段」后的问号图标，依其提示运行相应的 dig 命令检查 CNAME 记录是否生效。
如 dig 命令查不到相应的 CNAME 记录，请返回域名商控制台检查配置是否正确，如仍有疑问，请联系域名商客服。
如 dig 命令能查到预期的 CNAME 记录，但控制台仍显示「等待配置 CNAME 阶段」，请通过工单或 <support@leancloud.rocks> 联系我们。

## SSL 证书

在控制台绑定自定义域名时，可以选择自动管理 SSL 证书或手动管理 SSL 证书。
如果选择自动模式，LeanCloud 会自动申请、续期 [Let's Encrypt] 证书。
如果选择手动模式，则需要上传自己的 SSL 证书（通常是 `.crt` 或 `.pem` 文件）和 SSL 私钥（通常是 `.key` 文件），并在证书过期前自行续期及再次上传。
SSL 证书通常可以在你的域名服务商处购买，你也可以自行申请免费的证书。
Let's Encrypt 之外，比较知名的免费 SSL 证书提供商有 [buypass] 和 [TrustAsia]。

[Let's Encrypt]: https://letsencrypt.org/
[buypass]: https://www.buypass.com/ssl/products/acme
[TrustAsia]: https://freessl.cn/

## 备案

根据工信部规定，在 LeanCloud 华北、华东节点绑定的文件域名、云引擎域名、API 域名，都需要备案。

### 新增备案

如果你的主域名没有备案（没有 ICP 备案号），那么主域名本身及其子域名均无法在控制台绑定。
需要先在 LeanCloud 或其他云服务商办理备案。管局审核通过后，方可在 LeanCloud 控制台绑定。

如果你的主域名没有备案，我们建议你通过 LeanCloud 新增备案。
主域名在 LeanCloud 备案后，使用子域名在 LeanCloud 绑定文件域名、云引擎域名、API 域名不需要额外备案或接入备案。

商用版应用请进入 **应用控制台 > 账号设置 > 域名备案**，按照步骤填写资料，并根据控制台提示进行新增备案。

没有商用版应用的用户，如果不打算升级应用为商用版，可以通过以下方式新增备案：

1. 如果你同时也是 LeanCloud 底层 IaaS 服务商（华北节点为 UCloud）的用户，那么也可以自行在底层 IaaS 服务商新增备案。备案通过后，等价于在 LeanCloud 新增备案。
2. 如果只需要绑定文件域名，可以在其他云服务商处新增备案，然后直接在 LeanCloud 控制台绑定文件域名，因为绑定文件域名不需要接入备案。

注意，你也可以选择在其他云服务商处新增备案。
但对于大部分用户而言，因为需要绑定 API 域名或云引擎域名，在其他云服务商处新增备案之后，仍然需要在 LeanCloud 处接入备案。
这意味着需要重复提交许多类似的资料，经过两次审核周期，由管局先后审核两次，徒然增加时间和精力。
因此，对于主域名没有备案的用户，我们一般推荐在 LeanCloud 新增备案。

### 接入备案

工信部规定，网站接入多个云服务商时，需要在各云服务商处接入备案：

- 接入备案只是在备案信息中新增一个服务商，不会影响之前服务商，可以同时使用。
- 和新增备案不同，接入备案可以在服务上线后进行，不要求关站或停止解析，不影响当前网站访问。
- 一般而言，网站使用的所有云服务商皆需备案或接入备案（在使用的第一家云服务商处新增备案，在其他云服务商接入备案）。

因此，如果你的主域名之前通过 LeanCloud 新增备案，那么不需要再操心接入备案事项。
如果你的主域名是通过其他云服务商新增备案，那么你在 LeanCloud 控制台绑定 API 域名或云引擎域名后，需要在 LeanCloud 接入备案。
由于 CDN 服务一般不需要接入备案，因此，如果该子域名仅用于 LeanCloud 文件域名，那么无需在 LeanCloud 接入备案。

商用版应用如需接入备案，请先在 LeanCloud 控制台完成相应域名的绑定，
然后进入 **应用控制台 > 账号设置 > 域名备案**，按照步骤填写资料，并根据控制台提示进行接入备案。
办理接入备案期间域名、服务可以正常使用。

没有商用版应用的用户，云引擎、API 域名可以先绑定已备案的域名，进行测试开发。在产品正式上线时，升级应用为商用版，通过控制台接入备案。
或者，如果你同时也是 LeanCloud 底层 IaaS 服务商（华北节点为 UCloud）的用户，那么也可以自行在底层 IaaS 服务商接入备案。

华东节点底层 IaaS 服务商（腾讯云）暂不支持通过 LeanCloud 以用户主体身份提交备案资料。
因此，希望在华东节点所在机房做备案接入或直接办理新备案的用户，可以自行创建腾讯云账号并通过腾讯云完成备案。如需授权码，商用版应用用户可提交工单获取。

## 推荐阅读

如需了解域名解析和备案的基本知识，可以参考以下三篇文章：

1. [域名背后那些事](https://nextfe.com/domain-introduction/)
2. [域名之殇](https://nextfe.com/domain-problems/)
3. [备案那些事儿](https://nextfe.com/icp-introduction/)

另外，我们也收集了近期反馈较多的问题，以问答的形式展现出来，供大家参考：

[域名绑定 Q&A](https://leancloudblog.com/domain-question-answers/)
