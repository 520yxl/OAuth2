# OAuth身份验证API
以下文档描述了用于身份验证和授权的HTTP API
安装了WordPress的远程客户端。
## 框架
WordPress OAuth身份验证API（“OAuth API”）是基于HTTP的API
基于[OAuth 1.0a][RFC5849]。它还基于OAuth 1.0a规范
具有自定义参数。
本文档描述了OAuth API 0.1版。

## 语句
* "access token": 用于访问站点的长期令牌。赠款
基于客户端作用域的权限。
* "client": 访问OAuth API并提供服务的软件程序
给用户。
* "request token": 在OAuth过程中使用的短暂令牌。没有授予任何权限，并且只能用于授权步骤。
* "site": WordPress安装将OAuth API作为服务提供
* "user": API客户端的最终用户。通常是网站上的注册用户。

请注意，任何相对URL都被视为相对于网站的基本URL。

## 动机
OAuth API由三个主要因素驱动：

* 用户只应在网站中输入其凭据。客户不仅应该劝阻用户索要凭据，而且应该劝阻网站还应避免提供使用它们的方法。

* API必须在任何站点上工作。API只能使用大多数网站以提供有用的实用程序。

* API应该易于在客户端中实现。开发人员应该能够通过重用现有库而不是编写完整库来创建客户端定制解决方案。

## 与OAuth 1.0a的差异
OAuth API扩展了OAuth 1.0a以提供附加功能。这个以下差异适用：

* 授权端点（“资源所有者授权端点”）可能接受基于OAuth 2.0“scope”参数的“wp_scope”参数。（请参见第2步：授权）

## 步骤0：评估可用性
在开始授权过程之前，客户应评估网站支持它。由于网站的可自定义性质，这不是可以保证，因为OAuth API可以被禁用或替换。

为了满足这个要求，OAuth API与WordPress JSON接口REST API（“WP API”）。大多数使用OAuth API的客户端都应该具有能够访问WP API。

OAuth API通过WP的索引端点公开自身信息API，通常在`/wp-json/`上提供。WP API可通过RSD机制和使用此数据的OAuth API客户端应使用RSD机制，如WP API文档所述。

### 要求
客户端向WP API的索引端点发送HTTP GET请求。这个索引端点的位置超出了本文档的范围，并且已处理WP API文件。

### 响应
WP API索引端点返回与站点相关的数据的JSON对象。
OAuth API通过“oauth1”中的“authentication”值公开数据`oauth1`财产价值。

`oauth1`值（"API Description object"）是一个JSON对象
定义了以下属性：

* `request` 给出“临时凭据”位置的绝对URL请求端点”（请参阅步骤1：请求令牌）
* `authorize`: 给出“资源所有者”位置的绝对URL授权端点”（请参见步骤2：授权）
* `access`: 给出“令牌请求端点”位置的绝对URL（请参阅步骤3：访问令牌）
* `version`: 表示支持的OAuth API版本的版本字符串通过网站。

## Step 1: 请求令牌
授权过程的第一步是获取请求令牌。这步骤要求站点发布一个临时令牌，仅用于授权过程此令牌是一个短期到期令牌，尚未链接到用户。

此请求遵循的[临时凭据][身份验证请求] 部分
[RFC5948].

### 要求
客户端向`/oauth1/request`发送HTTP POST请求（“临时凭据请求端点“）。此URL也可通过API获得描述对象作为“request”属性，客户端应使用URL而不是硬编码URL。

此请求应与OAuth 1.0a中描述的格式相匹配规范，第2.1节。

此请求还可以包含以下参数作为
OAuth 1.0a参数：

* `wp_scope`: 这是一个OAuth风格的空格或逗号分隔字段
2.0的范围字段。这表示将可用权限缩小到客户。请参阅授权范围。此参数为OPTIONAL，默认值为到“*”（所有权限）。

### 响应
OAuth API返回OAuth请求令牌数据的URL编码响应，如下所示
如OAuth 1.0a规范第2.1节所述。

## 第2步：授权
授权过程的第二步是向用户。此步骤将用户发送到站点，然后用户对客户端进行身份验证并授予请求的权限。这接受与请求数据一起存储在站点上。

此请求遵循[资源所有者授权][身份验证授权]部分的[RFC5849][]，并添加了一些内容。

### 要求
客户端将用户发送到`/oauth1/authorize`（“资源所有者”授权端点”）。此URL也可通过API描述获得对象作为“authorize”属性，客户端应使用API中的URL描述对象，而不是对URL进行硬编码。

此请求应与OAuth 1.0a中描述的格式相匹配
规范，第2.2节。

此请求还可以包含以下参数作为
OAuth 1.0a参数：

* `wp_scope`: 这是一个OAuth风格的空格或逗号分隔字段2.0的范围字段。这表示将可用权限缩小到客户。请参阅授权范围。此参数为OPTIONAL，默认值为或者是请求令牌请求中指定的“wp_scope”参数，或“*”（所有权限）。

### 响应
网站会将用户重定向回“oauth_callback”，如授权步骤。“oauth_token”和“oauth_verifier”参数将是根据OAuth 1.0a标准附加到回调URL。

此外，还将附加一个“wp_scope”参数来描述实际范围授予（参见授权范围）。

## 步骤3：访问令牌
授权过程的第三步是使用现在已授权的请求令牌来请求访问令牌。此步骤要求站点使用请求令牌向客户端授予访问令牌，以便在将来的请求中用作身份验证。

此请求遵循的[Token Credentials][oauth access]部分
[RFC5849]

### 要求
客户端将用户发送到“/oauth1/access”（“令牌请求”端点）。
此URL也可以通过API Description对象作为“访问”属性使用，客户端应该使用API Description中的URL对象，而不是对URL进行硬编码。

此请求应与OAuth 1.0a中描述的格式相匹配
规范，第2.3节。

### 响应
OAuth API返回OAuth访问令牌数据的URL编码响应，如OAuth 1.0a规范第2.3节所述。

## 经过身份验证的请求
...

## 授权范围
OAuth API在请求令牌请求和授权请求期间都支持附加参数。此“wp_scope”参数是所请求作用域的分隔字符串列表。作用域应由U+0020个空格字符分隔，URL编码为“%20”。客户端可能使用U+0020空格字符，URL编码为“+”，或U+002C COMMA字符，URL代码为“%2c”。

OAuth API还将在Authorization步骤中将`wp_scope`参数返回到回调URL，作为授予作用域的空格分隔字符串列表（U+0020空格字符编码为`%20`）。此响应参数指示为令牌授予客户端的作用域。此授予的范围严格等于或小于请求的范围；也就是说，客户端永远不会从请求的权限中获得额外的权限，但用户可能会进一步限制客户端的范围。

未指定“wp_scope”参数的客户端的默认作用域为“*”，表示将授予所有权限。此权限授予执行用户有能力执行的任何操作的能力，包括他们可能被授予的任何未来功能。该范围应谨慎使用，因为它具有较大的攻击面。

### 可用范围
以下作用域可用：

* `read`: 能够读取任何公共网站数据，或用户可以访问的私人数据（如私人发布的帖子）。

  Maps to:
  * `read`
  * `read_private_*` (需要编辑器或更高版本)

* `edit`: 能够编辑用户的任何公共站点数据或私人数据
  有权访问。表示“已读”。

 需要贡献者或以上

  Maps to:
  * `edit_*`
  * `delete_*`
  * `upload_files` (要求作者或以上)
  * `moderate_comments` (需要编辑器或更高版本)
  * `manage_categories` (需要编辑器或更高版本)
  * `edit_others_*` (需要编辑器或更高版本)
  * `edit_private_*` (需要编辑器或更高版本)
  * `edit_published_*` (需要编辑器或更高版本)
  * `delete_others_*` (需要编辑器或更高版本)
  * `delete_private_*` (需要编辑器或更高版本)
  * `delete_published_*` (需要编辑器或更高版本)

* `user.read`: 能够读取大多数用户数据，但用户的电子邮件地址除外。

* `user.email`: 能够读取用户的电子邮件地址。用户电子邮件地址的使用应符合有关垃圾邮件的所有当地法律（针对客户和网站）。暗示`user.read`。

* `user.edit`: 能够编辑任何用户数据。暗示`user.read`和`user.email`。

* `admin.read`: 能够读取仅限管理员的数据。

  需要管理员或超级管理员。

  Maps to:
  * `list_users`

* `admin.edit`: 能够编辑仅限管理员的数据。

  需要管理员或超级管理员。

  Maps to:
  * `manage_options`
  * `install_plugins`
  * `update_plugins`
  * `install_themes`
  * `switch_themes`
  * `update_themes`
  * `edit_theme_options`
  * `update_core`
  * `edit_dashboard`

* `admin.users`: 管理用户的能力。

  需要管理员或超级管理员。暗示`user.edit`。

  Maps to:
  * `list_users`
  * `create_users`
  * `edit_users`
  * `promote_users`
  * `remove_users`
  * `delete_users`

* `admin.import`: 能够导入数据。

  需要管理员或超级管理员。暗示“编辑”。

  Maps to:
  * `import`

* `admin.export`: 能够导出数据。

需要管理员或超级管理员。暗示“已读”。

  Maps to:
  * `export`

对于大多数应用程序，“read”和“user.read”是合适的。对于任何需要访问当前用户信息的应用程序，建议使用“user.read”。

请求的任何权限对当前用户都不可用，这将导致向客户端返回错误。请注意，对于“编辑”之类的权限，没有“上传文件”功能（例如）的用户将**不会**导致错误，因为该权限包含其他功能。但是，没有“edit_*”功能**的用户将导致**错误。

[RFC5849]: http://tools.ietf.org/html/rfc5849
[oauth-request]: http://tools.ietf.org/html/rfc5849#section-2.1
[oauth-authorize]: http://tools.ietf.org/html/rfc5849#section-2.2
[oauth-access]: http://tools.ietf.org/html/rfc5849#section-2.3
