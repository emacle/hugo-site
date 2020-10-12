+++
title = "vue token过期无缝刷新"
author = ["emacle"]
date = 2020-01-17T10:39:00+08:00
tags = ["vue", "jwt", "token"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/graph.png"
pinned = true
+++

_过期刷新流程图:_
![](/ox-hugo/ditaa-refresh-token.png)

思路:

1.  登录时, 后端生成 _access\_token, refresh\_token_ 返回前端, 前端保存两个token在 cookie或localstorge中
2.  当前端发送正常请求时,请求头字段携带 _access\_token_ , 后端提取该 _access\_token_
    -   判断是否过期, 不过期则返回 HTTP 200 OK
    -   过期返回 _HTTP\_UNAUTHORIZED_ 401, 并且加上自定义响应数据 code = 50014 表示access\_token 过期
3.  VUE前端使用 **响应拦截器** , 对收到的 HTTP 401 进行拦截, 如果 http 401 且 code =50014 则先以 _refresh\_token_
    去获取新 _access\_token_

    -   如果正常获得 access\_token, 则再次以新 access\_token 发送原请求, 即可实现无缝刷新
    -   如果 _refresh\_token_ 也过期, 则服务器也返回 401, 但是加上了自定义响应数据 code= 50015, 前端的响应拦截器
        再次捕获到 error , 校验code =50015后, 则强制退出需要重新登录

    <!--listend-->

    ```js
    // response interceptor
    service.interceptors.response.use(
        response => {
       const res = response.data
       // 一些处理...
       return response.data
        },
        error => {
       // http 401 只能在 error 里被截获
       // console.log(error) *** 控制台不能输出返回的响应数据 ***
       // console.log(error.response) *** 可使用此命令进行调试 ***
       if (error.response.status === 401 && error.response.data.code === 50014) {
           // message: 'access_token过期,自动续期', code = 50014 access_token 过期
           return againRequest(error) // 此函数先以refresh_token 去获取新access_token, 然后再次以新 access_token 发送原请求
       }

       // 这里是 code = 50015 refresh_token 也过期的情况
       if (error.response.status === 401 && error.response.data.code === 50015) {
           // message: 'refresh_token过期,重定向登录', code = 50015 refresh_token 过期
           console.log('refresh_token过期 超时......')
           MessageBox.confirm('你已被登出，可以取消继续留在该页面，或者重新登录', '确定登出', {
         	   confirmButtonText: '重新登录',
         	   cancelButtonText: '取消',
         	   type: 'warning'
           }).then(() => {
         	   store.dispatch('FedLogOut').then(() => {
         	       location.reload() // 为了重新实例化vue-router对象 避免bug
         	   })
           })
       }

       return Promise.reject(error)
        }
    ) // response 拦截结束

    async function againRequest(error) {
        await store.dispatch('handleCheckRefreshToken') // 同步以获取刷新 access_token 并且保存在 cookie/localstorage
        const config = error.response.config
        config.headers['X-Token'] = getToken()  // 以新的 access_token
        const res = await axios.request(config) // 重新进行原请求
        return res.data // 以error.response.config重新请求返回的数据包是在函数内是 被封装在data里面
    }
    ```
4.  第3步以 refresh\_token 去获取 access\_token 时, 必须在 **请求拦截器** 里重新配置请求头, 以 refresh\_token 作为新的 token 头
    否则后端token认证判断还是原过期的 access\_token

    ```js
    // request interceptor
    service.interceptors.request.use(
        config => {
       // Do something before request is sent
       if (store.getters.token) {
           // 让每个请求携带token-- ['X-Token']为自定义key 请根据实际情况自行修改
           config.headers['X-Token'] = getToken()
       }

       // 监听是否 /sys/user/refreshtoken 是则重置token为 refresh_token
       const url = config.url
       if (url.split('/').pop() === 'refreshtoken') {
           // console.log('config.url', config.url, getRefreshToken())
           config.headers['X-Token'] = getRefreshToken() // 登录时本地 cookie/localstorage 存储的
       }
       return config
        },
        error => {
       // Do something with request error
       console.log(error) // for debug
       Promise.reject(error)
        }
    )
    ```

    ![](https://raw.githubusercontent.com/emacle/vue-php-admin/master/vue-element-admin/static/screenshot/view%5Fjwt.gif)
    ![](https://raw.githubusercontent.com/emacle/vue-php-admin/master/vue-element-admin/static/screenshot/edit%5Fjwt.gif)
    ![](https://raw.githubusercontent.com/emacle/vue-php-admin/master/vue-element-admin/static/screenshot/del%5Fjwt.gif)

    ```text
    用户编辑时比较有用, 防止长时间编辑后提交时 access_token 过期, 导致编辑内容丢失
    ```

5.  完整代码 <https://github.com/emacle/vue-php-admin>

6.  VUE 请求拦截器与响应拦截器代码修改自 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin/) 中的 [request.js](https://github.com/PanJiaChen/vue-element-admin/blob/master/src/utils/request.js)

参考: [php firebase/php-jwt token验证](https://blog.csdn.net/cjs5202001/article/details/80228937)
