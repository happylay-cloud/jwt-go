# jwt-go

[![Build Status](https://travis-ci.org/dgrijalva/jwt-go.svg?branch=master)](https://travis-ci.org/dgrijalva/jwt-go)
[![GoDoc](https://godoc.org/github.com/dgrijalva/jwt-go?status.svg)](https://godoc.org/github.com/dgrijalva/jwt-go)

[JSON Web Tokens](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 的[go](http://www.golang.org)实现（或搜索引擎友好的"golang"）


**新版本即将推出：** 自2016年发布3.0.0版本以来，已经提出了许多改进建议。
我现在正在努力削减两个不同的发行版：3.2.0将包含所有不间断的更改或增强。 
4.0.0即将发布，其中将包括重大更改。 请参阅4.0.0里程碑以了解即将发生的事情。 
如果您有其他想法，或者想参加4.0.0，现在是时候了。 
如果您依赖于此库并且不想被中断，建议您使用依赖项管理工具将其固定到版本3。

**安全通知：** 一些较旧版本的Go在cryotp/eliplic中存在安全问题。 建议至少升级到1.8.3。有关更多详细信息，请参见问题＃216。

**安全通知：** 重要的是您[验证所提供的`alg`是您所期望的](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/) 。 
这个库试图通过要求键类型与预期的alg匹配来简化正确的操作，但是您应该在使用中采取额外的步骤来验证它。请参阅提供的示例。

## JWT到底是什么？

JWT.io对JSON Web令牌有[非常好的介绍](https://jwt.io/introduction) 。

简而言之，它是一个签名的JSON对象，可以执行一些有用的操作（例如，身份验证）。 
它通常用于Oauth 2中的`Bearer`令牌。令牌由三部分组成，以`.`分隔。 
前两个部分是JSON对象，它们已被[base64url](http://tools.ietf.org/html/rfc4648)编码。 
最后一部分是签名，以相同的方式编码。

第一部分称为标题。它包含验证最后一部分（签名）的必要信息。
例如，签名使用哪种加密方法，使用什么密钥。

中间的部分是有趣的部分。 它被称为"声明"，包含了你真正关心的东西。 请参阅[RFC](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) ，
了解有关保留密钥的信息以及添加自己的保留密钥的正确方法。

## 盒子里有什么东西？

该库支持JWT的解析和验证以及JWT的生成和签名。 
目前支持的签名算法是HMAC SHA、RSA、RSA-PSS和ECDSA，不过有钩子可以添加您自己的签名算法。

## 示例

请参阅[项目文档](https://godoc.org/github.com/dgrijalva/jwt-go) 了解使用示例：

* [解析和验证令牌的简单示例](https://godoc.org/github.com/dgrijalva/jwt-go#example-Parse--Hmac)
* [构建和签名令牌的简单示例](https://godoc.org/github.com/dgrijalva/jwt-go#example-New--Hmac)
* [示例目录](https://godoc.org/github.com/dgrijalva/jwt-go#pkg-examples)

## 扩展

此库发布了添加您自己的签名方法所需的所有组件。
只需实现`SigningMethod`接口，并使用`RegisterSigningMethod`注册工厂方法。

下面是一个扩展的例子，集成了多个谷歌云平台签名工具（AppEngine, IAM API, Cloud KMS）：https://github.com/someone1/gcp-jwt-go

## 合规

该库最后一次审查符合[RTF 7519](http://www.rfc-editor.org/info/rfc7519) 日期为2015年5月，但有几个显著差异：

* 为了防止意外使用[不安全的JWTs](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html#UnsecuredJWT) ，
使用`alg=none`的令牌 仅当提供常量`jwt.UnsafeAllowNoneSignatureType`作为密钥时，才会被接受。
  
## 项目状态和版本控制

该库被认为可以投入生产。感谢您的反馈和功能要求。
该API应该被认为是稳定的。
除了主要版本更新之外（并且仅出于充分的理由），应该很少有向后不兼容的更改。

该项目使用[语义版本2.0.0](http://semver.org)。 
接受的拉取请求将落在`master`上。 会定期从`master`标记版本。 
您可以在[项目发行页面](https://github.com/dgrijalva/jwt-go/releases) 上找到所有发行版本。

尽管我们尝试在进行重大更改时使其变得明显，但是并没有一种很好的机制将通知发布给用户。
您可能希望使用此替代软件包，包括：：`gopkg.in/dgrijalva/jwt-go.v3`。
它将在语义版本控制方面做正确的事情。

**破坏性 更改:*** 

* 版本3.0.0包含了 _许多_ 与2.x系列不同的变化，包括一些破坏API的变化。
  我们试着尽可能少地破坏一些东西，所以应该只改变一些类型签名。
  `VERSION_HISTORY.md`中提供了重大更改的完整列表。 
  有关更新代码的更多信息，请参见`MIGRATION_GUIDE.md`。

## 使用技巧

### 签名与加密

令牌只是由其作者签名的JSON对象。 这正好告诉您有关数据的两件事：
* 令牌的作者拥有签名密钥
* 数据自从签名后就没有被修改过

重要的是要知道JWT不提供加密，这意味着有权访问令牌的任何人都可以读取其内容。
如果您需要保护（加密）数据，有一个配套规范`JWE`提供了此功能。
JWE目前不在这个库的范围内。

### 选择签名方法

有几种可用的签名方法，在选择一种方法之前，您可能需要花时间了解各种选项。
主要的设计决策很可能是对称与非对称。

对称签名方法，如HSA，仅使用一个秘钥。 
这可能是最简单的签名方法，因为任何`[]byte`都可以用作有效密钥。
它们的计算速度也略快一些， 尽管这并不足以引起重视。
当令牌的生产者和使用者均受信任甚至是同一系统时，对称签名方法将发挥最佳作用。
由于使用相同的秘密对令牌进行签名和验证，因此您无法轻松分发用于验证的密钥。

非对称签名方法，如RSA，使用不同的密钥对令牌进行签名和验证。
这使得生成带有私钥的令牌成为可能，并允许任何使用者访问公钥进行验证。

### 签名方法和密钥类型

每个签名方法都要求其签名密钥具有不同的对象类型。
有关详细信息，请参阅软件包文档。以下是最常见的：

* [HMAC签名方法](https://godoc.org/github.com/dgrijalva/jwt-go#SigningMethodHMAC) (`HS256`,`HS384`,`HS512`) 需要`[]byte`值进行签名和验证
* [RSA签名方法](https://godoc.org/github.com/dgrijalva/jwt-go#SigningMethodRSA) (`RS256`,`RS384`,`RS512`) 需要 `*rsa.PrivateKey`私钥进行签名和`*rsa.PublicKey`公钥进行验证
* [ECDSA签名方法](https://godoc.org/github.com/dgrijalva/jwt-go#SigningMethodECDSA) (`ES256`,`ES384`,`ES512`) 需要 `*ecdsa.PrivateKey`私钥进行签名和`*ecdsa.PublicKey`公钥进行验证

### JWT和OAuth

值得一提的是，OAuth和JWT不是一回事。
JWT令牌只是一个签名的JSON对象。
它可以用在任何有用的地方。
不过，有一些混淆，因为JWT是OAuth2身份验证中使用的最常见的承载令牌类型。

在不费吹灰之力的情况下，以下是对这些技术相互作用的描述：：

* OAuth是一种协议，用于允许身份提供者与用户登录的服务分开。 
  例如，每当您使用Facebook登录到其他服务（Yelp，Spotify等）时，都在使用OAuth。
  
* OAuth定义了一些用于传递身份验证数据的选项。 
  一种流行的方法称为"bearer token"（承载令牌）。 
  承载令牌只是一个字符串，仅 _应_ 由经过身份验证的用户持有。 
  因此，只需出示此令牌即可证明您的身份。 
  您可能可以从这里得出为什么JWT可以成为一个好的承载令牌的原因。
  
* 由于承载令牌用于身份验证，因此将它们保密是很重要的。 
  这就是使用承载令牌的事务通常通过SSL进行的原因。

### 故障排除

该库尽可能使用描述性错误消息。 
如果没有得到预期的结果，请查看错误。
人们最常遇到的问题是为解析器提供正确类型的密钥。
请参阅以上有关签名方法和密钥类型的部分。

## 更多

文档可以在[godoc.org](http://godoc.org/github.com/dgrijalva/jwt-go) 上找到。

此项目中包含的命令行实用程序（cmd/jwt）提供了一个令牌创建和解析的简单示例，
以及一个用于调试您自己的集成的有用工具。您还可以在文档中找到几个实现示例。
