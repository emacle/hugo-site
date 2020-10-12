+++
title = "hugo-dusk theme 模板,utterances评论修改"
author = ["emacle"]
date = 2020-01-16T09:30:00+08:00
tags = ["hugo", "utterances", "theme"]
categories = ["hugo", "emacs"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/color-palette.png"
pinned = false
+++

Hugo 以 theme 中的静态模板文件来生成静态css文件

1.  **favicon** 图标

    ```bash
    [params]
    # 自定义 favicon 变量 ~/hugo/static/favicon.ico -> /public/favicon.ico
    # ~/hugo/static/ 目录下的东西会自动发布到时 /public 根目录 i.e. ox-hugo/ orgmode 导出的静态图片
    # 使用绝对路径 防止 /posts/ 页面404 favicon
    favicon = "https://emacle.github.io/favicon.ico"
    ```

    head.html 模板头部加入

    ```text
    <link rel="icon" href="{{ .Site.Params.favicon }}">
    ```

2.  启用 github **utterances** 评论, 修改.Type 变量, ox-hugo默认导出目录为 `<HUGO_BASE_DIR>/content/posts/` 因此 .Type = "posts"

    ```text
    cat ~/hugo/themes/hugo-dusk/layouts/partials/postfooter.html
    ```

<!--listend-->

```html
<footer class="post-footer">

  <div class="post-footer-data">
       {{ partial "tags.html" . }}
       <!-- 修改默认日期显示格式 -->
       <!-- <div class="date"> {{ .Date.Format "2006-01-01" }} </div> -->
       <div class="date"> {{ .Date.Format "Mon, 02 Jan 2006 15:04:05 MST" }} </div>
    {{if or .Params.categories .Params.series}}
    <hr>
       {{ partial "categories.html" . }}
       {{ partial "series.html" . }}
    {{end}}
  </div>
</footer>
    <!-- .Type 变量 自动分配或继承前端 https://gohugo.io/variables/page/ Page Variables .Type -->
    <!-- new file created at content/posts/new-post.md will automatically be assigned the type posts -->
    <!-- 测试 .Type 变量输出 以双大括号 变量输出 -->
    <!--  if eq .Type "post"  -->
{{ if eq .Type "posts" }}
  {{ template "_internal/disqus.html" . }}
  {{ template "partials/utterances.html" . }}
{{ end }}
```

1.  **代码高亮**

    ```bash
    # Configure syntax highlight
    # The Chroma php lexer doesn’t support a startinline option. You’ll have to add a starting <?php tag or use Pygments.
    # php 语言高亮需要加入 <?php 标记才能正常高亮
    [markup]
      [markup.highlight]
        style = "paraiso-dark" # dark themes: emacs, vi, vs, paraiso-dark,monokai, api, fruity, native, rrt, swapoff   https://xyproto.github.io/splash/docs/all.html
    ```

    ```php
    <?php
    $provider = new \League\OAuth2\Client\Provider\GenericProvider([
        'clientId'                => 'demoapp',    // The client ID assigned to you by the provider
        'clientSecret'            => 'demopass',   // The client password assigned to you by the provider
        'redirectUri'             => 'http://example.com/your-redirect-url/',
        'urlAuthorize'            => 'http://brentertainment.com/oauth2/lockdin/authorize',
        'urlAccessToken'          => 'http://brentertainment.com/oauth2/lockdin/token',
        'urlResourceOwnerDetails' => 'http://brentertainment.com/oauth2/lockdin/resource'
    ]);

    // If we don't have an authorization code then get one
    if (!isset($_GET['code'])) {

        // Fetch the authorization URL from the provider; this returns the
        // urlAuthorize option and generates and applies any necessary parameters
        // (e.g. state).
        $authorizationUrl = $provider->getAuthorizationUrl();

        // Get the state generated for you and store it to the session.
        $_SESSION['oauth2state'] = $provider->getState();

        // Redirect the user to the authorization URL.
        header('Location: ' . $authorizationUrl);
        exit;

    // Check given state against previously stored one to mitigate CSRF attack
    } elseif (empty($_GET['state']) || (isset($_SESSION['oauth2state']) && $_GET['state'] !== $_SESSION['oauth2state'])) {

        if (isset($_SESSION['oauth2state'])) {
       unset($_SESSION['oauth2state']);
        }

        exit('Invalid state');

    } else {

        try {

       // Try to get an access token using the authorization code grant.
       $accessToken = $provider->getAccessToken('authorization_code', [
           'code' => $_GET['code']
       ]);

       // We have an access token, which we may use in authenticated
       // requests against the service provider's API.
       echo 'Access Token: ' . $accessToken->getToken() . "<br>";
       echo 'Refresh Token: ' . $accessToken->getRefreshToken() . "<br>";
       echo 'Expired in: ' . $accessToken->getExpires() . "<br>";
       echo 'Already expired? ' . ($accessToken->hasExpired() ? 'expired' : 'not expired') . "<br>";

       // Using the access token, we may look up details about the
       // resource owner.
       $resourceOwner = $provider->getResourceOwner($accessToken);

       var_export($resourceOwner->toArray());

       // The provider provides a way to get an authenticated API request for
       // the service, using the access token; it returns an object conforming
       // to Psr\Http\Message\RequestInterface.
       $request = $provider->getAuthenticatedRequest(
           'GET',
           'http://brentertainment.com/oauth2/lockdin/resource',
           $accessToken
       );

        } catch (\League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {

       // Failed to get the access token or user details.
       exit($e->getMessage());

        }

    }
    ```

    ```emacs-lisp
    (use-package magit
      :ensure t
      :about A Git porcelain inside Emacs
      :homepage https://github.com/magit/magit
      :info (info "(magit) Top")
      :bind (("C-x g"   . magit-status))
      :hook (company-mode . company-box-mode)
      :config
      (setq-default magit-diff-refine-hunk t)
      (setq magit-save-repository-buffers 'dontask)
      ;; M-x `magit-list-repositories'
      (setq magit-repository-directories
       '(("~/.emacs.d"                . 0)
         ("~/.emacs.d/straight/repos" . 1)
         ("~/src"                     . 1))))
    ```

    ```bash
    #!/bin/bash
    # 以下配置信息请自己修改
    mysql_user="USER" #MySQL备份用户
    mysql_password="PASSWORD" #MySQL备份用户的密码
    mysql_host="localhost"
    mysql_port="3306"
    mysql_charset="utf8" #MySQL编码
    backup_db_arr=("db1" "db2") #要备份的数据库名称，多个用空格分开隔开 如("db1" "db2" "db3")
    backup_location=/opt/mysql #备份数据存放位置，末尾请不要带"/",此项可以保持默认，程序会自动创建文件夹
    expire_backup_delete="ON" #是否开启过期备份删除 ON为开启 OFF为关闭
    expire_days=3 #过期时间天数 默认为三天，此项只有在expire_backup_delete开启时有效

    # 连接到mysql数据库，无法连接则备份退出
    mysql -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password <<end
    use mysql;
    select host,user from user where user='root' and host='localhost';
    exit
    end

    flag=`echo $?`
    if [ $flag != "0" ]; then
    echo "ERROR:Can't connect mysql server! backup stop!"
    exit
    else
    echo "All database backup success! Thank you!"
    exit
    fi
    ```

2.  页面底部 "Powered by" 没有参数可配置取消, 可grep定位文件位置 注释掉

    ```text
    cat ~/hugo/themes/hugo-dusk/layouts/partials/footer.html
    ```

3.  copyright 部分可以 config.toml 里配置后启用

    ```text
    copyright = "Copyright (c) 2020, all rights reserved."
    ```
