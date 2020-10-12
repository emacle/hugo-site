+++
title = "Vue Element-ui Github oAuth 三方登录"
author = ["emacle"]
date = 2020-03-06T11:42:00+08:00
tags = ["vue", "element", "github", "oauth"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/graph.png"
pinned = true
+++

{{< figure src="/ox-hugo/github_login.gif" >}}

1.  Github 网站创建 OAuth Apps 记录 `Client ID` / `Client Secret` / `Authorization Callback URL` 3个参数
    此处 `Authorization Callback URL` 为默认回调网址，程序代码里没有明确指定的话，以此为默认值，代码里指定的话
    指定的redirect `一定要与此处一致防止提交时校验出错` ip/域名都可，在弹出子窗口进行授权确定时，三方服务器会校验
    url里的redirect参数(程序里指定的)与创建三方应用里指定的 callback 地址，如果不一致会出错

    ```sh
    http://host:port/auth-redirect # 必须与创建应用时填入值一致
    http://localhost:9527/auth-redirect
    http://172.17.1.110:10000/auth-redirect
    http://www.vueauth.org:10000/auth-redirect  # 浏览器的主机 hosts/dns 里需要解析 www.vueauth.org
    ```

    {{< figure src="/ox-hugo/github-oauth.png" >}}

2.  socialsignin.vue 三方登录模块里，构造弹出窗口 github 认证 url

    ```js
    githubHandleClick(thirdpart) {
        // 1. 指定授权 client_id 及 redirect_uri 的 URL
        //    如果不指定 redirect_uri, 则默认使用 gihtub => Settings => Developer settings =>  OAuth Apps 里 Authorization callback URL 配置的地址
        //    为了无歧义尽量在程序代码里指定redirect_uri
        // const url = 'https://github.com/login/oauth/authorize?client_id=94aae05609c96ffb7d3b&redirect_uri=http://localhost:9527/auth-redirect'
        githubAuth().then(response => { // githubAuth() 参数 code 为空，由后端返回 authorize_url
       // 2. 弹出子窗口进行授权, 子窗口完成授权后, 子窗口地址栏URL 会是 redirect_uri 并带上 ?code= 参数与&state= 参数
       //    子窗口单击确定按钮授权时，会校验此时的redirect 与 创建三方应用时填写的 callback 不一致会出错
       const url = response.data.auth_url
       // console.log('auth url....', url)
       // =>  https://github.com/login/oauth/authorize?state=137caabc2b409f0cccd14834fc848041&response_type=code&approval_prompt=auto&redirect_uri=http://localhost:9527/auth-redirect&client_id=94aae05609c96ffb7d3b
       openWindow(url, thirdpart, 540, 540)
        }).catch(error => {
       console.log(error)
        })
    }
    ```
3.  authredirect.vue 里，即第2步url参数 redirect\_uri 指定的回调地址对应的路由组件(/auth-redirect)，在此弹出的子窗口里进行授权，授权完成后，调用js 的 window.opener 方法 给 父窗口 的 location.href 赋值，父窗口地址栏变为 会带有 github 返回的 ?code 参数，同时关闭子窗口，代码逻辑转至父窗口进行后续处理

    ```js
    name: 'AuthRedirect',
    created() {
      this.githubLogin()
    },
    methods: {
          githubLogin() {
          // 1.  授权成功后, github 返回给 AuthRedirect子窗口的浏览器 回调地址 并带上 ?code=8789d613d1fa9a19732a&state=xyz 参数
          //     地址栏URL如 http://localhost:9527/auth-redirect?code=8789d613d1fa9a19732a&state=xyz
          //     其中 http://localhost:9527/auth-redirect 是定义 githubHandleClick() 里定义或服务端返回的回调地址
          //     const url = 'https://github.com/login/oauth/authorize?client_id=94aae05609c96ffb7d3b&redirect_uri=http://localhost:9527/auth-redirect'
          //     此时 window.location.href   => http://localhost:9527/auth-redirect?code=8789d613d1fa9a19732a&state=137caabc2b409f0cccd14834fc848041
          //         window.location.search  => ?code=8789d613d1fa9a19732a&state=137caabc2b409f0cccd14834fc848041

          // 2. 调用 window.opener 方法 给 父窗口 的 location.href 赋值 => http://localhost:9527/login?code=8789d613d1fa9a19732a
          window.opener.location.href = window.location.origin + '/login' + window.location.search
          //    注意：此处加入 /login 是因为 router /login 在 permission.js的whiteLis里不会发生重定向而导致href里丢失?code等参数,从而
          //         可以在login/index.vue里直接通过location或vue route 获取code，state参数，不必在permission.js里获取并保存至store里
          //         ***非常关键***，不加的话默认/ 是等价于/dashboard不在白名单里路由根据permission.js逻辑会发生重定向至/login里此时会丢失?code等参数
          //    可在此 使用未定义的变量 来 debug  // Error in created hook: "ReferenceError: hash is not defined" found in window.opener.location.href = window.location.origin + '?' + hash
          // 3. 关闭 AuthRedirect 子窗口。同时代码逻辑至父窗口 在 permission.js => router.beforeEach 进行 code 处理
          window.close()
        }
    }
    ```

4.  login/index.vue 代码逻辑转至此处， created 生命周期里，判断 如果存在则为github三方登录,前端调用githubAuth 函数带code参数向后台进行认证正确后会返回给前面 token/refresh\_token，前面配置token正确后却可正常登录 (与后端交互时可加入三方登录loading动画效果提示)

    ```js
    created() {
        this.githubLogin()
    },
    methods: {
        githubLogin() {
       // console.log('in login/index.vue....', window.location, this.$route, this.$router)
       // 获取三方登录 code
       // 更可靠稳定的获取code方法 使用 vue router to 对象来获取
       // 如果路由存在code 与 state 参数，如http://localhost:9527/login?code=8789d613d1fa9a19732a&state=xyz
       if (this.$route.query.hasOwnProperty('code') && this.$route.query.hasOwnProperty('state')) { // this.$route.query 如果存在 code 则为三方登录则写入store 变量
           const code = this.$route.query.code
           const state = this.$route.query.state
           console.log('github code: ', code)
           console.log('github state: ', state)
           // store.state.user.code = code
           // store.state.user.code_state = state
           // console.log(store.state.user) // 该code 在store/modules/user.js 里定义有 作为第三方登录使用 参见其中 LoginByThirdparty

           this.thirdLogin = true
           const loading = this.$loading({
         	   lock: true,
         	   text: 'github 认证登录中...',
         	   spinner: 'el-icon-loading',
         	   background: 'rgba(0, 0, 0, 0.7)'
           })

           const authParms = { code, state }
           // 执行 GET githubAuth 根据 code 获取 github userinfo 结合业务逻辑生成 token / refreshtoken (jwt)
           this.$store.dispatch('githubAuth', authParms).then(() => {
         	   this.$router.push({ path: '/' })
         	   loading.close()
           }).catch((err) => {
         	   console.log('this.$store.dispatchgithubAuth catch....', err.response)
         	   this.thirdLogin = false
         	   loading.close()
           }).finally((e) => {
         	   console.log('this.$store.dispatchgithubAuth finally....', e)
         	   this.thirdLogin = false
         	   loading.close()
           })
       }
        }
    }
    ```

    ```js
    // github认证
    githubAuth({ commit }, authParms) {
        return new Promise((resolve, reject) => {
       githubAuth(authParms.code, authParms.state).then(response => {
           console.log('githubAuth response...', response)
           const data = response.data
           commit('SET_TOKEN', data.token)
           commit('SET_REFRESH_TOKEN', data.refresh_token)
           setToken(data.token)
           setRefreshToken(data.refresh_token)
           resolve()
       }).catch(error => {
           reject(error)
       })
        })
    },

    // github 认证api
    export function githubAuth(code) {
        return request({
       url: '/sys/user/githubauth',
       method: 'get',
       params: { code }
        })
    }
    ```

5.  后端php /sys/user/githubauth 接口, 引用 `league/oauth2-client` 包生成 `授权链接`  根据code与github进行交互，主要有获取github `access_token` 与 获取github `userinfo`
    后端也可以自动封装函数来获取github交互信息 (如果是php可利用composer安装一些三方登录插件错误校验比较完善，一般需要php开启 开启php.ini中的session.auto\_start配置 使用session，
    测试 league/oauth2-client 包没有开启 php session `也可以使用 $_SESSION['oauth2state']` 变量，也可以 `redis` 替换)

    <https://oauth2-client.thephpleague.com/providers/league/> 可在此网站查找部分官方包及第三方封装的 leageue php 登录包

    ```bash
    # league/oauth2-client 是通用客户端 github,gitee ok!!!
    # In most cases, you'll want to use a specific provider client library rather than this base library.
    composer require league/oauth2-github  # github ok!!!
    composer require spoonwep/oauth2-qq    # QQ OAuth 2.0 support for the PHP League's OAuth 2.0 Client
    composer require oakhope/oauth2-wechat # 未测试
    ```

    oauth2-client 包官方 demo 捕获异常时 指定IdentityProviderException 不能捕获其依赖包 guzzlehttp 里的错误异常 使用 Exception $e 替换

    ```text
    GuzzleHttp\Exception\ConnectException  Message: cURL error 35: OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to api.github.com:443
    ```

<!--listend-->

```php
<?php
try{
    # getaccessToken...
    # get userinfo...
    //} catch (\League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
} catch (Exception $e) {
    // Failed to get the access token or user details.
    exit($e->getMessage());
}
```

1.  vue路由模式最好使用history, hash模式的话url中的#字符认证中转时不太好处理，history模式在 `生产环境` 中需要配置后端服务器支持(此后端为vue编译生成的静态网页面代码部署的服务器，如 [apache route history mode配置参考](https://panjiachen.github.io/vue-element-admin-site/zh/guide/essentials/deploy.html#%25E5%258F%2591%25E5%25B8%2583) ， `并不是后端接口的服务器` )

    ```js

    @router/index.js
    export default new Router({
      mode: 'history', // require service support 前端部署的时候 apache 需要配置文件 .htaccess
    scrollBehavior: () => ({ y: 0 }),
    routes: constantRouterMap
    })

    @dist/.htaccess  // 防止直接刷新页面服务端会直接报 404 错误。
    <IfModule mod_rewrite.c>
      RewriteEngine On
      RewriteBase /
      RewriteRule ^index\.html$ - [L]
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteCond %{REQUEST_FILENAME} !-d
      RewriteRule . /index.html [L]
    </IfModule>
    ```

2.  完整代码 <https://github.com/emacle/vue-php-admin>

3.  其它问题
    -   cURL error 60: SSL certificate problem: unable to get local issuer certifica 解决
        从 <https://curl.haxx.se/docs/caextract.html> 上下载cacert.pem

        ```php
        # vi php.ini  搜索curl.cainfo 与 openssl.cafile,将其配置成你自己cacert.pem文件的路径
        curl.cainfo="D:\phpstudy_pro\Extensions\php\php7.3.4nts\extras\ssl\cacert.pem"
        openssl.cafile="D:\phpstudy_pro\Extensions\php\php7.3.4nts\extras\ssl\cacert.pem"
        ```

    -   [OAuth 2.0 筆記 (4.1) Authorization Code Grant Flow 細節](https://blog.yorkxin.org/2013/09/30/oauth2-4-1-auth-code-grant-flow.html)
    -   github 登录报错 回调地址不一致出错提示 github 与 gitee 不太一样

        ```text
        github 在地址栏提示
        ```

        <http://localhost:9527/auth-redirect?error=redirect%5Furi%5Fmismatch&error%5Fdescription=The> redirect\_uri MUST match the registered callback URL for this application.& error\_uri=<https://developer.github.com/apps/managing-oauth-apps/troubleshooting-authorization-request-errors/#redirect-uri-mismatch%23/auth-redirect>

        ```text
        gitee 提示子窗口显示 回调地址有误
        ```
