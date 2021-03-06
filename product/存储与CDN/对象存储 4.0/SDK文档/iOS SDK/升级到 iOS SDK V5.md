## 升级到 iOS SDK V5

如果您细心对比过 iOS SDK V4 和 V5 的文档，您会发现并不是一个简单的增量更新。COS V5 在架构、可用性和安全性上有了非常大的提升，我们的 SDK 在易用性、健壮性和传输性能上也做了非常大的改进。如果您想要升级到 iOS SDK V5，请参考下面的指引，一步步完成 SDK 的升级工作。

### 功能对比

下表列出了 V4 和 V5 SDK 的主要功能对比：

| 功能       | V5         | V4                         |
| -------- | :------------: | :------------------:    |
| 文件上传 | 支持本地文件、二进制数据、分片上传<br>默认覆盖上传<br>智能判断上传模式 | 只支持本地文件上传<br>可选择是否覆盖<br>需要手动选择是简单还是分片上传 |
| 文件删除 | 支持批量删除 | 只支持单文件删除 |
| 存储桶基本操作 | 创建存储桶<br>获取存储桶<br>删除存储桶   | 不支持 |
| 存储桶ACL操作 | 设置存储桶ACL<br>获取设置存储桶ACL<br>删除设置存储桶ACL   | 不支持 |
| 存储桶生命周期 | 创建存储桶生命周期<br>获取存储桶生命周期<br>删除存储桶生命周期 | 不支持 |
| 目录操作 | 不支持   | 创建目录<br>查询目录<br>删除目录 |


### 总览

1. 更新您的 iOS SDK
2. 我们的鉴权方式发生了变化，建议后台集成我们的临时密钥（STS）方案，并提供接口，客户端携带获取到的临时密钥执行 COS 请求
3. 根据指引修改 SDK 的初始化方式
4. 我们的 `存储桶名称` 和 `可用区域简称` 有了更新，请对应修改
5. 一些操作的 API 发生了变化，我们做了封装让 SDK 更加易用，具体请参考我们的示例和 [接口文档](https://cloud.tencent.com/document/product/436/12258)

### 更新 SDK
您可以通过 cocoapods 或下载打包好的动态库的方式来集成 SDK。在这里我们推荐您使用 cocoapods 的方式来进行导入。
- **使用 Cocoapods 导入（推荐）**  
在 Podfile 文件中使用：
```
  pod 'QCloudCOSXML'
```

- **使用打包好的动态库导入（手动集成方式）**  
可以从 [realease](https://github.com/tencentyun/qcloud-sdk-ios/releases) 中选择需要的版本进行下载  
将 **QCloudCOSXML.framework, QCloudCore.framework 和 libmtasdk.a** 拖入到工程中：
![](https://main.qcloudimg.com/raw/14c8f5773ea19bc681b7f862dd6384fb.png)  

并添加以下依赖库：
> 1. CoreTelephony
> 2. Foundation
> 3. SystemConfiguration
> 4. libc++.tbd

#### 工程配置
在 Build Settings 中设置 Other Linker Flags，加入以下参数：
```
-ObjC
-all_load
```

![参数配置](https://main.qcloudimg.com/raw/3bee5d2c3cb7e7f80b94c5f6bbe2ce5e.png)

### 签名和权限

COS V5 使用了新的鉴权算法，在 V4 中您需要自己在后台计算好签名，再返回客户端使用，在 V5 中，我们强烈建议您后台接入我们的临时密钥（STS）方案。您不需要了解签名计算过程，只需要在服务器端接入 CAM，将拿到的临时密钥返回到客户端，并设置到 SDK 中，SDK 会负责管理密钥和计算签名。临时密钥在一段时间后会自动失效，而永久密钥不会泄露。您还可以按照不同的粒度来控制访问权限。具体的步骤请参考 [快速搭建移动应用直传服务](https://cloud.tencent.com/document/product/436/9068) 以及 [权限控制实例](https://cloud.tencent.com/document/product/436/30172)。

### 初始化

在 V5 中，我们的初始化接口发生了一些变化：

* 为了区分，`QCloudCOSXMLService` 代替了 `COSClient`，但他们的作用相同,同时增加了`QCloudServiceConfiguration`来配置更多的信息。
* 您需要在初始化时实例化一个密钥提供者 `QCloudAuthentationV5Creator`，用于提供一个有效的密钥，建议使用临时密钥。

v4的初始化方式如下：

```
COSClient *client= [[COSClient alloc] initWithAppId:appId withRegion:@“sh”];
```

v5的初始化方式如下：(示例代码中给出的是通过推荐使用的临时密钥的方式获取签名)

```
- (void) signatureWithFields:(QCloudSignatureFields*)fileds
                     request:(QCloudBizHTTPRequest*)request
                  urlRequest:(NSURLRequest*)urlRequst
                   compelete:(QCloudHTTPAuthentationContinueBlock)continueBlock
{
    /*向签名服务器请求临时的 Secret ID,Secret Key,Token*/
    QCloudCredential* credential = [QCloudCredential new];
    credential.secretID = @"从 CAM 系统获取的临时 Secret ID";
    credential.secretKey = @"从 CAM 系统获取的临时 Secret Key";
    credential.token = @"从 CAM 系统返回的 Token，为会话 ID"
    credential.expiretionDate     = /*签名过期时间*/
    QCloudAuthentationV5Creator* creator = [[QCloudAuthentationV5Creator alloc] initWithCredential:credential];
    QCloudSignature* signature =  [creator signatureForData:urlRequst];
    continueBlock(signature, nil);

}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
     QCloudServiceConfiguration* configuration = [QCloudServiceConfiguration new];
     configuration.appID = @"*****";
     configuration.signatureProvider = self;
     QCloudCOSXMLEndPoint* endpoint = [[QCloudCOSXMLEndPoint alloc] init];
     endpoint.regionName = @"ap-beijing";//服务地域名称，可用的地域请参考注释
     configuration.endpoint = endpoint;

     [QCloudCOSXMLService registerDefaultCOSXMLWithConfiguration:configuration];
     [QCloudCOSTransferMangerService registerDefaultCOSTransferMangerWithConfiguration:configuration];

}
```


### Bucket 和 Region 变化

V5 的存储桶名称发生了变化，在 V5 中，存储桶名称由两部分组成：用户自定义字符串 和 APPID，两者以中划线“-”相连。例如 `mybucket1-1250000000`，其中 `mybucket1` 为用户自定义字符串，`1250000000` 为 APPID。APPID 是腾讯云账户的账户标识之一，用于关联云资源。在用户成功申请腾讯云账户后，系统自动为用户分配一个 APPID。可通过 腾讯云控制台 【账号信息】查看 APPID。

在设置 Bucket 时，请参考下面的示例代码：

```
NSString *bucket = "mybucket1-1250000000";
```

V5 的存储桶可用区域简称发生了变化，下面列出了不同区域在 V4 和 V5 中的对应关系：

| 地域       | V5 地域简称         | V4 地域简称                         |
| -------- | ------------ | ---------------------------------------- |
| 北京一区（华北） | ap-beijing-1 | tj |
| 北京       | ap-beijing   | bj |
| 上海（华东）   | ap-shanghai  | sh |
| 广州（华南）   | ap-guangzhou | gz |
| 成都（西南）   | ap-chengdu   | cd |
| 重庆       | ap-chongqing | 无 |
| 新加坡      | ap-singapore | sgp |
| 香港       | ap-hongkong  | hk |
| 多伦多      | na-toronto   | ca |
| 法兰克福     | eu-frankfurt | ger |
| 孟买       | ap-mumbai    | 无 |
| 首尔       | ap-seoul     | 无 |
| 硅谷       | na-siliconvalley     | 无 |
| 弗吉尼亚       | na-ashburn     | 无 |
| 曼谷       | ap-bangkok     | 无 |
| 莫斯科       | eu-moscow     | 无 |

在初始化时，请将存储桶所在区域简称设置到 `QCloudServiceConfiguration`的 `regionName`中：

```
//AppDelegate.m

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
     QCloudServiceConfiguration* configuration = [QCloudServiceConfiguration new];
     configuration.appID = @"*****";
     configuration.signatureProvider = self;
     QCloudCOSXMLEndPoint* endpoint = [[QCloudCOSXMLEndPoint alloc] init];
     endpoint.regionName = @"ap-beijing";//服务地域名称，可用的地域请参考注释
     configuration.endpoint = endpoint;

     [QCloudCOSXMLService registerDefaultCOSXMLWithConfiguration:configuration];
     [QCloudCOSTransferMangerService registerDefaultCOSTransferMangerWithConfiguration:configuration];

}
```

### API变化

#### 不再支持目录操作

在 V5 中，我们不再支持目录操作。

对象存储中本身是没有文件夹和目录的概念的，对象存储不会因为上传对象 project/a.txt 而创建一个 project 文件夹。为了满足用户使用习惯，对象存储在控制台、COS browser 等图形化工具中模拟了「 文件夹」或「 目录」的展示方式，具体实现是通过创建一个键值为 project/，内容为空的对象，展示方式上模拟了传统文件夹。

例如：上传对象 project/doc/a.txt ，分隔符 / 会模拟「 文件夹」的展示方式，于是可以看到控制台上出现「 文件夹」project 和 doc，其中 doc 是 project 下一级「 文件夹」，并包含了 a.txt 。

因此，如果您的应用场景只是上传文件，可以直接上传即可，不需要先创建文件夹。

如果您的使用场景里面有文件夹的概念，需要提供创建文件夹的功能，您可以上传一个路径以 '/' 结尾的 0KB 文件。这样在您调用 `GetBucket` 接口时，就可以将这样的文件当做文件夹。


#### QCloudCOSTransferMangerService

在 V5 SDK 中，我们封装了上传、下载和复制操作，命名为 `QCloudCOSTransferMangerService`，对 API 设计和传输性能都做了优化，建议您直接使用。`QCloudCOSTransferMangerService`的主要特性有：

* 支持断点上传
* 支持根据文件大小智能选择简单上传（copy）还是分片上传（copy）

使用 `QCloudCOSTransferMangerService`上传的示例代码：

```
  QCloudCOSXMLUploadObjectRequest* put = [QCloudCOSXMLUploadObjectRequest new];
  NSURL* url = /*文件的URL*/;
  put.object = @"文件名.jpg";
  put.bucket = @"test-123456789";
  put.body =  url;
  [put setSendProcessBlock:^(int64_t bytesSent, int64_t totalBytesSent, int64_t totalBytesExpectedToSend) {
      NSLog(@"upload %lld totalSend %lld aim %lld", bytesSent, totalBytesSent, totalBytesExpectedToSend);
  }];
  [put setFinishBlock:^(id outputObject, NSError* error) {

  }];
  [[QCloudCOSTransferMangerService defaultCOSTransferManager] UploadObject:put];

```
上传文件时的断点续传

```
QCloudCOSXMLUploadObjectRequest* put = [QCloudCOSXMLUploadObjectRequest new];
  //···设置一些上传的参数
  put.initMultipleUploadFinishBlock = ^(QCloudInitiateMultipartUploadResult * multipleUploadInitResult, QCloudCOSXMLUploadObjectResumeData resumeData) {
	//在初始化分片上传完成以后会回调该block，在这里可以获取 resumeData，并且可以通过 resumeData 生成一个分片上传的请求
	 QCloudCOSXMLUploadObjectRequest* request = [QCloudCOSXMLUploadObjectRequest requestWithRequestData:resumeData];
  };

  [[QCloudCOSTransferMangerService defaultCOSTransferManager] UploadObject:put];
  //···在完成了初始化，并且上传没有完成前
  NSError* error;
  //这里是主动调用取消，并且产生 resumetData 的例子
  resumeData = [put cancelByProductingResumeData:&error];
  if (resumeData) {
    QCloudCOSXMLUploadObjectRequest* request = [QCloudCOSXMLUploadObjectRequest requestWithRequestData:resumeData];
  }
  //生成的用于恢复上传的请求可以直接上传
  [[QCloudCOSTransferMangerService defaultCOSTransferManager] UploadObject:request];

```
> 注意  
注意的是，按照分片上传的运行原理，只有当一个分片上传完了，那么后台服务器才会将该分片记录下来，并且叠加进度。并且以下几种情况无法进行断点续传，而是重新开始一次上传过程：
 - 上传的文件小于 1M,没有进行分片上传
 - 没有使用 QCloudCOSXMLUploadObjectRequest 类进行上传，而是直接使用简单上传接口
 - 取消生成 resumeData 时候初始化分片上传还没有完成（完成初始化上传的回调还没有调用）
#### 新增API

V5 增加了很多新的API，包括：

* 存储桶的操作，如 QCloudPutBucketRequest, QCloudGetBucketRequest, QCloudListBucketRequest 等
* 存储桶ACL的操作，如 QCloudPutBucketACLRequest，QCloudGetBucketACLRequest 等
* 存储桶生命周期的操作，如 PQCloudutBucketLifecycleRequest, QCloudGetBucketLifecycleRequest 等

具体请参考我们的 [iOS SDK 接口文档](https://cloud.tencent.com/document/product/436/12258)。
