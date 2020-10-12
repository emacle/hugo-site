+++
title = "OAuth 2 Authorization Code Grant Flow"
author = ["emacle"]
date = 2020-03-09T10:37:00+08:00
tags = ["oauth"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/color-palette.png"
pinned = false
+++

-   [转自 OAuth 2.0 筆記 (4.1) Authorization Code Grant Flow 細節](https://blog.yorkxin.org/2013/09/30/oauth2-4-1-auth-code-grant-flow.html)

    在 Authorization Grant Code Flow 裡，Client 不直接向 Resource Owner 要求許可，而是把 Resource Owner 導去
    Authorization Server 要求許可， Authorization Server 再透過轉址來告訴 Client 授權許可的代碼 (code) 。
    在轉址回去之前， Authorization Server 會先認證 Resource Owner 並取得授權。因為 Resource Owner 只跟
    Authorization Server 認證，所以 Client 絕對不會拿到 Resource Owner 的帳號密碼。

{{< figure src="/ox-hugo/ditaa-auth-code.png" >}}

註: (A), (B), (C) 這三步的線拆成兩段，因為會經過 user-agent
_其 Client 指自己的应用程序（包括前端及后端api） User-Agent 一般指流览器_

-   (D) 步 客户端向 authorization server 发送 auth\_code 时(手工封装时) 未带有redirect\_uri貌似也能成功最好还是按标准带上该参数？

使用 oauth2-client 包时应该是封装好的 authorization\_code 与 前面定义的 redirect\_uri 一同发向 authorization server

```php
<?php
// composer require league/oauth2-client
$provider = new \League\OAuth2\Client\Provider\GenericProvider([
    'clientId' => $client_id,    // The client ID assigned to you by the provider
    'clientSecret' => $client_secret,   // The client password assigned to you by the provider
    'redirectUri' => 'http://localhost:9527/auth-redirect',
    'urlAuthorize' => 'https://github.com/login/oauth/authorize',
    'urlAccessToken' => 'https://github.com/login/oauth/access_token',
    'urlResourceOwnerDetails' => 'https://api.github.com/user'
]);
// getAccessToken 时应该带有 redirect_uri?
// http://localhost:9527/auth-redirect?code=8789d613d1fa9a19732a&state=xyz
$accessToken = $provider->getAccessToken('authorization_code', [
    'code' => $code
]);
```

-   [转自 github 授权登录教程与如何设计第三方授权登录的用户表](https://segmentfault.com/a/1190000018372841) 流程图

{{< figure src="/ox-hugo/work.org_20200309_103003.png" >}}

{{< figure src="/ox-hugo/work.org_20200309_103130.png" >}}
