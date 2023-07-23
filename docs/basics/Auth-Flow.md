# 授权流程
理解OAuth如何工作的关键是理解授权流。这是客户端链接到站点的过程。
使用OAuth插件的流被称为**三足流**，这要归功于所涉及的三个主要步骤：
* **临时凭据获取**：客户端从服务器获取一组临时凭据。
* **授权**：用户“授权”请求令牌访问其帐户。
* **令牌交换**：客户端将短期临时凭据交换为长期令牌。
## 临时凭据获取
授权的第一步是获取临时凭证（也称为**请求令牌**）。这些凭据是短暂的（通常为24小时），纯粹用于初始授权过程。它们不授予对服务器上数据的任何访问权限，并且不能用于除授权流之外的任何事情。
这些凭证是通过对服务器的初始HTTP请求获取的。客户端首先向临时凭据URL发送POST请求，通常是带有插件的“/oauth1/request”。（应该从API自动发现此URL，因为各个站点可能会移动此路由，或将进程委托给另一个服务器。）这类似于：
此请求包括客户端密钥（`oauth_consumer_key`）、授权回调（`oaAuth_callback`）和请求签名（`oaauth_signature`和`oauth_signatue_method`）。这看起来像：
```
POST /oauth1/request HTTP/1.1
Host: server.example.com
Authorization: OAuth realm="Example",
               oauth_consumer_key="jd83jd92dhsh93js",
               oauth_signature_method="HMAC-SHA1",
               oauth_timestamp="123456789",
               oauth_nonce="7d8f3e4a",
               oauth_callback="http%3A%2F%2Fclient.example.com%2Fcb",
               oauth_signature="..."
```

服务器检查密钥和签名以确保客户端有效。它还检查回调以确保它对客户端有效。
检查完成后，服务器将创建一组新的临时凭据（`oauth_token`和`oauth_token_secret`），并在HTTP响应（URL编码）中返回它们。这看起来像：

```
HTTP/1.1 200 OK
Content-Type: application/x-www-form-urlencoded

oauth_token=hdk48Djdsa&oauth_token_secret=xyz4992k83j47x0b&oauth_callback_confirmed=true
```

然后，这些凭据被用作授权和令牌交换步骤的“oauth_token”和“oauth_token_secret”参数。

（始终返回`oauth_callback_confirmed=true`，表示协议为oauth 1.0a。）


## 批准
流程中的下一步是授权过程。这是一个面向用户的步骤，也是大多数用户都熟悉的步骤。
使用由站点提供的授权URL（通常为“/oauth1/authorize”），客户端将临时凭证密钥（上面的“oauth_token”）附加到URL作为查询参数（再次称为“oauth_token”）。然后客户端将用户引导到此URL。通常，这是通过重定向浏览器内客户端或打开本机客户端的浏览器来完成的。
然后，如果用户还没有登录，则用户会登录并授权客户端。如果他们不想链接客户端，也可以选择取消授权过程。
如果用户授权客户端，则站点会将令牌标记为已授权，并将用户重定向回回调URL。回调URL包括两个额外的查询参数：“oauth_token”（相同的临时凭证令牌）和“oauth_verifier”，一个需要在下一步中传递的CSRF令牌。

## 代币交易所
授权的最后一步是将临时凭据（请求令牌）交换为长期凭据（也称为**访问令牌**）。此请求还会销毁临时凭据。
通过向令牌请求端点发送POST请求（通常为“/oauth1/access”），将临时凭证转换为长期凭证。此请求必须由临时凭据签名，并且必须包括授权步骤中的“oauth_verifier”令牌。请求的内容如下：

```
POST /oauth1/access HTTP/1.1
Host: server.example.com
Authorization: OAuth realm="Example",
               oauth_consumer_key="jd83jd92dhsh93js",
               oauth_token="hdk48Djdsa",
               oauth_signature_method="HMAC-SHA1",
               oauth_timestamp="123456789",
               oauth_nonce="7d8f3e4a",
               oauth_verifier="473f82d3",
               oauth_signature="..."
```

服务器再次检查密钥和签名，并检查验证器令牌以[避免CSRF攻击](http://oauth.net/advisories/2009-1/)。
假设这些检查全部通过，服务器将在HTTP响应主体中使用最后一组凭据（表单数据，URL编码）进行响应：

```
HTTP/1.1 200 OK
Content-Type: application/x-www-form-urlencoded

oauth_token=j49ddk933skd9dks&oauth_token_secret=ll399dj47dskfjdk
```

在这一点上，您现在可以丢弃临时凭据（因为它们现在是无用的）以及验证器令牌。
祝贺您，您的客户现在已链接到该网站！
