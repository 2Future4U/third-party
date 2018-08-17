---


copyright:
  years: 2018
lastupdated: "2018-07-27"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# 步骤 4：开发认证流程

定义产品时，资源管理控制台的**访问管理**页面向您提供了 {{site.data.keyword.Bluemix_notm}} Identity and Access Management (IAM) 客户机标识和密钥、服务标识和 API 密钥。现在，使用这些值来完成以下步骤：

1. 根据对 `dashboard_url` 的处理来派生重定向 URI，返回到资源管理控制台并将其添加到 IAM 选项卡，从而确保更新客户机标识。
2. 开发用于认证的 OAuth 流程。要完成此流程，您将使用重定向 URI、客户机标识和客户机密钥作为 `token_endpoint` IAM REST API 的参数。
3. 验证用户授权：
   1. 与 IAM 通信以从 API 密钥获取访问令牌
   2. 使用 `authorization_endpoint` 验证是否授予用户对服务仪表板的权限 (/v2/authz POST)

此步骤假定已批准您交付 Integrated Billing 服务。如果您尚未完成 PWB 中的初始注册和批准，请参阅：[在 {{site.data.keyword.Bluemix_notm}} 中发布第三方产品入门](/docs/third-party/index.html)
{: tip}

## 开始之前

确保已开始执行步骤 1，并完成了步骤 2 和 3：
1. [编写服务文档和市场营销公告](/docs/third-party/cis1-docs-marketing.html)。
2. [在资源管理控制台中定义产品](/docs/third-party/cis2-rmc-define.html)。
3. [开发和托管服务代理程序](/docs/third-party/cis3-broker.html)。


## 派生 IAM 重定向 URI（资源管理控制台：IAM 页面）

在资源管理控制台中定义服务时，您已生成客户机标识，但请注意，此时您可能并没有重定向 URI。这意味着 IAM 创建的是设置为 false 的客户机标识。在使用“重定向 URI”返回到资源管理控制台之前，您不会拥有设置为 true 的客户机标识。

好消息是，在先前的开发步骤中，您已开发了 OSB 并对其进行了托管（可能在样本代理程序代码中会看到 IAM 值）。`redirect_uri` 通常是应用程序所在的主机 URL，以及其他一些将处理认证/授权的 URL。

 下面是一些示例重定向 URI

```
https://myapp.bluemix.net/integrate/auth/callback
http://localhost:3000/auth/callback <-- for testing locally
```

返回到资源管理控制台，并将重定向 URI 添加到 IAM 选项卡：

1. 登录到控制台
2. 抓取重定向 URI
3. 返回到资源管理控制台
4. 在 **IAM** 选项卡中，将重定向 URI 粘贴到**重定向 URI** 字段中。
5. 单击**更新客户机标识**以更新客户机标识。

现在，您应该拥有一个能够识别重定向 URI 且设置为 true 的客户机标识！您可以在后续步骤中使用该客户机标识来开发 OAuth 流程。

## 开发用于认证的 OAuth 流程
{: #oauth}


**认证 - 步骤 0：**通过调用 `https://iam.bluemix.net/identity/.well-known/openid-configuration` 来找到离已部署应用程序更近的 UI 登录的 IAM 区域端点。

```
curl -X GET \
  https://iam.bluemix.net/identity/.well-known/openid-configuration
```

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "issuer": "https://iam.bluemix.net/identity",
  "authorization_endpoint": "https://iam-region2.bluemix.net/identity/authorize",
  "token_endpoint": "https://iam-region2.bluemix.net/identity/token",
...
}
```

此请求可在应用程序启动时执行一次，并且如果 `authorization_endpoint` 失败，可再次执行此请求。您应该能够对 `authorization_endpoint` 值进行短时间高速缓存，并在高速缓存到期或遇到错误后进行刷新。


**认证 - 步骤 1：**用户浏览到 `dashboard_url` 时，将浏览器重定向到 `<authorization_endpoint>?client_id=<your-client-id>&redirect_uri=<your-redirect-uri>&response-type=code&state=<your-resource-instance-id>`


-> 将显示登录提示

-> 用户输入凭证

-> 浏览器回调到重定向 URI，此 URI 提供“代码”响应参数和“状态”值


**认证 - 步骤 2：**用代码交换访问令牌调用

### POST <token_endpoint>

#### 头：
{: #headers1}

  - Authorization: Basic *[client id]:[client secret]*
  - Content-Type: application/x-www-form-urlencoded
  - Accept: application/json

#### 参数：
{: #parameters1}

  - client_id=*[client id]*
  - client_secret=*[client secret]*
  - grant_type=authorization_code
  - response_type=cloud_iam
  - redirect_uri=*[same uri as redirect_uri from step 1]*
  - code=*[code from callback]*

```
curl -k -X POST \
  -u "<your-client-id>:<your-client-secret>" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Accept: application/json" \
  --data-urlencode "client_id=<your-client-id>" \
  --data-urlencode "client_secret=<your-client-secret>" \
  --data-urlencode "grant_type=authorization_code" \
  --data-urlencode "response_type=cloud_iam" \
  --data-urlencode "code=<code-from-the-callback>" \
  --data-urlencode "redirect_uri=<redirect_uri>" \
  "https://iam-region2.bluemix.net/identity/token"
```
{: codeblock}

### 响应：
{: #response1}

```
{
  "access_token": "eyJraWQiOiI......XmpBTIDdR5w",
  "refresh_token": "SPrXw5tBE3......KBQ+luWQVY=",
  "token_type": "Bearer",
  "expires_in": 3600,
  "expiration": 1473188353
}
```
{: codeblock}

  确保存储此响应中返回的用户 access_token，因为在接下来用户授权期间将使用此项。

请参阅样本代理程序中的示例：https://github.com/IBM/sample-resource-service-brokers

## 下面将验证用户授权
{: #validate}

1. 与 IAM 通信以获取 API 密钥的访问令牌
2. 验证是否授予用户对服务实例的权限 (/v2/authz POST)

### 授权 - 步骤 1：使用 API 密钥获取 {{site.data.keyword.Bluemix_notm}} IAM 令牌。
{: #iam_token_using_api_key}

### POST /identity/token

#### 头：
{: #headers2}

  - Authorization: Basic *[client id]:[client secret]*
  - Content-Type: application/x-www-form-urlencoded
  - Accept: application/json

#### 参数：
{: #parameters2}

  - grant_type=urn:ibm:params:oauth:grant-type:apikey
  - response_type=cloud_iam
  - apikey=*[Api key]*

```
curl -k -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Accept: application/json" \
  --data-urlencode "grant_type=urn:ibm:params:oauth:grant-type:apikey" \
  --data-urlencode "response_type=cloud_iam" \
  --data-urlencode "apikey=<apikey>" \
  "https://iam-region2.bluemix.net/identity/token"
```
{: codeblock}

### 响应：
{: #response2}

```
{
  "access_token": "eyJhbGciOiJIUz......sgrKIi8hdFs",
  "refresh_token": "SPrXw5tBE3......KBQ+luWQVY=",
  "token_type": "Bearer",
  "expires_in": 3600,
  "expiration": 1473188353
}
```
{: codeblock}

**注：**此令牌一小时内有效，在一小时的时间范围内可以根据需要多次复用此令牌。强烈建议将此令牌进行高速缓存，以避免对 `dashboard_url` 的每次访问都执行此请求。


### 授权 - 步骤 2：验证是否授予用户对服务实例的权限 (/v2/authz POST)

现在，我们已认证用户并拥有自己的访问令牌，接下来我们需要验证用户是否能够访问服务仪表板。首先，我们需要用户访问令牌中包含的一些信息，将在步骤 2.1 中对其解码。然后，在步骤 2.2 中，我们将使用这些信息来调用 IAM，以检查用户是否有权访问仪表板。

**步骤 2.1**：对用户的访问令牌（在上述 `**认证 - 步骤 2：**用代码交换访问令牌`期间返回）解码。这是 JWT 令牌，可使用任何兼容 JWT 的库进行解码。例如，请参阅[样本代理程序代码](https://github.com/IBM/sample-resource-service-brokers)中包含的库。对令牌解码后，格式如下所示；我们需要抽取 `iam_id` 和 `scope` 字段，以在下一步中使用：

```
{
  "iam_id": "IBMid-XXXXXXX",
  "id": "IBMid-XXXXXXX",
  "realmid": "IBMid",
  "identifier": "XXXXXXX",
  "given_name": "Don",
  "family_name": "Quixote",
  "name": "Don Quixote",
  "email": "quixote@email.com",
  "sub": "quixote@email.com",
  "account": {
    "bss": "123123123123123"
  },
  "iat": 1522114004,
  "exp": 1522117604,
  "iss": "https://iam.bluemix.net/identity",
  "grant_type": "urn:ibm:params:oauth:grant-type:apikey",
  "scope": "openid <your serviceName>",
  "client_id": "bx",
  "acr": 1,
  "amr": [
    "pwd"
  ]
}
```

**步骤 2.2**：调用 IAM 以检查用户是否有权访问仪表板

```
curl -X POST \
  -H "Accept: application/json" \
  -H "Authorization: <access token from step 1>" \
  -d '[ \
    { \
      "subject" : \
      { \
        "attributes": \
        { \
          "id": "<iam_id field value from user's token>", \
          "scope": "<scope field value from user's token>" \
        } \
      }, \
      "resource" : \
      { \
        "crn" : "<resource instance CRN>" \
      }, \
      action : <your service name> + ".dashboard.view" \
    } \
  ]' \
  https://iam.bluemix.net/v2/authz
```

请参阅样本代理程序中的示例：https://github.com/IBM/sample-resource-service-brokers

## 面向第三方采用者的 {{site.data.keyword.Bluemix_notm}} Identity and Access Management 令牌作用域限定
{: #token_scoping}

使用您的客户机标识创建的用户访问令牌只能用于访问您的服务 API。使用此令牌向其他云 API 发出的请求将导致访问被拒绝，即使该用户已配置相应的策略也是如此。

作为第三方集成的一部分，令牌作用域限定用于确保令牌具有实现用户目标所需的最小访问作用域。为了帮助达到此目的，IAM 令牌具有的访问权将基于创建令牌的客户机标识。这意味着如果 IAM 令牌是由第三方服务创建的，那么最终用户将无法执行某些 API 和功能，即使该用户已配置相应的策略也是如此。

对授权的影响（对 `https://iam.bluemix.net/v2/authz` 的所有调用）需要在主题中向下传递 `scope` 信息。此信息包含在 `scope` 声明中的 IAM 令牌（base64 编码）内。

下面是在授权调用中添加的内容的示例：
```
  [
   {  Headers
   `Authorization` -> 表示第三方服务和/或仪表板的 JWT 令牌
   `Transaction-ID` -> "唯一的 GUID 允许我们帮助以端到端方式跟踪请求"
   `Accept` -> `application/vnd.authz.v2+json`
   `Content-Type` -> `application/json`

      Body
      "subject":{
         "attributes":{
            "id":"IBMid-123",
            "scope":"libraryservice openid"
         }
      },
      "action":"libraryservice.books.read",
      "resource":{
         "attributes":{
            "serviceName":"libraryservice",
            "serviceInstance":"12345",
            "accountId":"123456789"
         }
      }
   }
]
```
{: codeblock}

这适用于所有用途（`user、serviceId、crn`），并且所有 `subject.attributes` 都需要作用域。

## 后续步骤

现在该把所有内容融会贯通在一起了！返回到资源管理控制台，在有限的可视范围内发布服务，并在目录中验证产品。请参阅：[发布和测试服务](/docs/third-party/cis4-rmc-publish.html)。