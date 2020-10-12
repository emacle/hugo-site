+++
title = "Hugo with OrgMode"
author = ["emacle"]
date = 2020-01-15T17:04:00+08:00
tags = ["hugo"]
categories = ["hugo", "emacs"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/graph.png"
pinned = false
+++

1.  elpa 安装 ox-hugo

2.  优先使用基于子树模式 两个必填 <hugo\_base\_dir> 与 EXPORT\_FILE\_NAME
    -   org 文件头 设置配置 `#+hugo_base_dir` 参数指向 `hugo` 工程目录

        ```text
        #+hugo_base_dir: ../hugo
        ```
    -   subtree 属性必须要设置 EXPORT\_FILE\_NAME 即slug <span class="underline">不可重复</span>, 作为导出的 .md 文件名

        ```text
        :EXPORT_FILE_NAME: hugo-with-orgmode
        ```
    -   导出sub-tree到 `<HUGO_BASE_DIR>/content/posts/`

        ```text
        C-c e H H 或 C-c e H O
        ```

        子树如果是 DONE 则 会直接发布, 如果是 TODO 则会做为草稿

3.  Sub-tree 属性 EXPORT\_HUGO\_TAGS 与 EXPORT\_HUGO\_CATEGORIES

    -   多个参数以 `空格` 分隔

    -   会覆盖 orgmode 里的 tags 属性, 留空或不设置则会继承 orgmode tags

    -   以 `@` 开头的 orgmode tags 可表示 CATEGORIES

    <!--listend-->

    ```text
    :EXPORT_HUGO_TAGS: hugo haha
    :EXPORT_HUGO_CATEGORIES: orgmode emacs
    ```

    可不用使用 CATEGORIES 能与 orgmode 保持一致, 前端配置可去除该链接
    EXPORT\_HUGO\_TAGS 也可以留空, 只使用org tags 方式

    属性插入可编写 `yasnippet` , 利用自定义函数 hugoslug-to-pinyin, 插入时自动将
    head title里的中文转换成拼音

4.  org本地图片可以无描述插入可直接导出正确的url格式

    ```text
    [[file:images/80px-Heckert_GNU_white.svg.png]] 描述为空不能单击图片链接
    ```

    {{< figure src="/ox-hugo/80px-Heckert_GNU_white.svg.png" >}}

    ```text
    [[desc][link]] 描述不为空 图片可单击打开，i.e. (org-store-link) 保存的链接
    ```

    {{< figure src="/ox-hugo/ditaa-wlan-eth0-internet.png" link="/ox-hugo/ditaa-wlan-eth0-internet.png" >}}

    ```text
    [[https://raw.githubusercontent.com/emacle/picgo/master/img/icon_a.png]]  远程 image url
    ```

    {{< figure src="https://raw.githubusercontent.com/emacle/picgo/master/img/icon%5Fa.png" >}}

    ```text
    MiniCap.exe 截图 / FastStone 简单特效/标注/绘画马赛克/文字放大镜/中心聚光模糊效果 / picpick.exe 卡
    ```

    {{< figure src="/ox-hugo/computer.org_20200119_220125.png" link="/ox-hugo/computer.org_20200119_220125.png" >}}

    ```sh
    # 1.初始化一个hugo工程目录
    hugo new site mysite
    # 2. 必须在 mysite/themes 里 git 下载主题 如无mysite/themes目录 则手动创建
    # 3. cd mysite 测试
    # > hugo.exe new posts/my-first.md 生成 .md -> 编辑
    hugo.exe serve
    # 4. 发布, 在hugo文件夹下自动生成一个public文件夹 将public托管 -b 指定baseURL
    hugo.exe -b https://emacle.github.io/
    ```
