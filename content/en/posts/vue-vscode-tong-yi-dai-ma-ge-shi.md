+++
title = "vue vscode 统一代码格式"
author = ["emacle"]
date = 2020-08-17T08:34:00+08:00
tags = ["vue", "npm", "vscode", "vetur", "eslint"]
categories = ["vue"]
draft = false
authorEmoji = "🎅"
image = "https://cn.vuejs.org/images/logo.png"
pinned = false
+++

1.  vscode 安装 vetur，eslint 插件
    使用 vetur 作为vue文件格式化程序, prettier不用安装
    `注意 .eslintignore 文件一定不能忽略/src，如果包含则src下面的所有文件都不会使用eslint校验`  即使用命令 eslint --fix src/ 也会忽略

    ```json
    {
        // 指定vue格式化程序为vetur 只安装vetur时不需要指定，安装有其他格式化插件时如 prettier时需要指定，避免冲突
        "[vue]": {
       "editor.defaultFormatter": "octref.vetur"
        },
        // vetur格式化js时默认使用prettier 会在语句后添加;后，最后一个对象元素添加,号
        // 指定 vscode-typescript 或 prettier-eslint可以不添加,; 与eslint保持一致
        "vetur.format.defaultFormatter.js": "vscode-typescript",
        "eslint.alwaysShowStatus": true,
        // 文件保存时, eslint插件集成自动lint修复，不需要再手动执行命令，不能自动修复的代码会报红
        // 注意 .eslintignore 文件一定不能忽略，如果/src则src下面的所有文件保存时不会使用eslint校验
        // 如果不想vscode里面文件报红，可以在.eslintignore 里指定忽略文件，不建议
        "editor.codeActionsOnSave": {
       // "source.fixAll": true,
       "source.fixAll.eslint": true
        }
    }
    ```

2.  使用vue-element-admin的 .eslintrc.js 配置文件
    需要packages.json devDependencies字段 安装下列包（不仅限于）

    ```text
    npm i -D eslint babel-eslint eslint-plugin-vue
    ```

3.  package.json 增加配置

    ```json
    "scripts": {
    "lint": "eslint --ext .js,.vue src",
    }
    ```

    校验代码

    ```text
    npm run lint 或 npx eslint --ext .js,.vue src
    ```

    自动修复代码

    ```text
    npm run lint -- --fix  # 这里 -- 必须有，因为不是直接执行的eslint命令，或 npx eslint --ext .js,.vue src --fix
    ```

    在执行npm run lint进行代码检测的时候，总是在最后出现npm报错。这个报错并不影响上面显示的lint错误
    eslint exits with status 1 if there are any linting errors. npm interprets this as an error, and it's telling you so.
    可以使用-s 参数抑制错误错误 尽量不使用该参数

    ```text
    npm run lint -s
    ```

    <https://panjiachen.github.io/vue-element-admin-site/zh/guide/advanced/eslint.html#vscode-%E9%85%8D%E7%BD%AE-eslint>
