+++
title = "Git Hooks 使用 husky/yorkie lint-staged git 提交前进行校验"
author = ["emacle"]
date = 2020-08-17T08:46:00+08:00
tags = ["vue", "npm", "husky", "eslint", "yorkie"]
categories = ["vue"]
draft = false
authorEmoji = "🎅"
image = "https://cn.eslint.org/img/favicon.512x512.png"
pinned = false
+++

新版本的@vue/cli-service 内置了 yorkie, 见package-lock.json 不再需要额外安装husky
在安装之后，@vue/cli-service 也会安装 yorkie，它会让你在 package.json 的 gitHooks 字段中方便地指定 Git hook：

每次它只会在你本地 commit 之前，校验你提交的内容是否符合你本地配置的
eslint规则(这个见文档 ESLint )，如果符合规则，则会提交成功。
如果不符合它会自动执行 eslint --fix 尝试帮你自动修复， `注意 .eslintignore 如果包含则会忽略eslint校验`
`如果修复成功则会帮你把修复好的代码提交，如果失败，则会提示你错误，让你修好这个错误之后才能允许你提交代码。`
安装后husky/yorkie 它会在我们项目根目录下面的.git/hooks文件夹下面创建pre-commit、pre-push等hooks。可以检查对应的hooks
这些hooks可以让我们直接在package.json的script里运行我们想要在某个hook阶段执行的命令

```text
npm install lint-staged -D 或者 yarn add lint-staged -D
```

然后修改 package.json，增加配置

```json
"gitHooks": {
  "pre-commit": "lint-staged"
},
"lint-staged": {
  "src/**/*.{js,vue}": [
    "eslint --fix",
    "git add"
  ]
}
```

<https://panjiachen.github.io/vue-element-admin-site/zh/guide/advanced/git-hook.html#husky>
