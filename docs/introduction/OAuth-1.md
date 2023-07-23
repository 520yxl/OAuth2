#为什么选择OAuth 1.0a？
如果您已经对OAuth进行了研究，您可能会注意到OAuth 2.0的存在。为什么规范授权方案使用旧版本的OAuth？
OAuth有着悠久的历史。OAuth概念源自Twitter（和其他人），它需要对用户帐户进行委托授权，主要用于API访问。然后，随着其他各方（如谷歌）的反馈，这一点继续发展，最终在[RFC 5849]中被标准化为OAuth 1.0a(https://tools.ietf.org/html/rfc5849)。OAuth在OAuth 2.0中得到了进一步的发展和简化，标准化为[RFC 6749](https://tools.ietf.org/html/rfc6749)。
从第1版到第2版的主要变化是删除了复杂的签名系统。该签名系统旨在确保只有客户端才能使用用户令牌，因为它依赖于共享机密。但是，每个请求都必须单独签名。版本2依赖于SSL/TLS来处理消息的真实性。
这意味着**OAuth 2.0需要HTTPS**。WordPress却没有。我们需要能够为所有网站提供身份验证，而不仅仅是那些使用HTTPS的网站。
随着Let's Encrypt证书颁发机构即将对HTTPS的竞争环境进行更改，我们希望未来能够要求SSL并转向OAuth 2.0，但这还不可行。
（注意：虽然OAuth RFC要求某些端点使用SSL，但[OAuth 1.0a](http://oauth.net/core/1.0a/)没有。这是对RFC的故意违反，因为我们需要支持非SSL站点。）
