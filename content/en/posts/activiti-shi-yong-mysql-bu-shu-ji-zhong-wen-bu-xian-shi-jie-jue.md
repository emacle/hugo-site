+++
title = "activiti使用mysql部署及中文不显示解决"
author = ["emacle"]
date = 2020-05-05T21:55:00+08:00
tags = ["activiti"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/workflow.png"
pinned = false
+++

1.  官网下载 activiti-6.0.0.zip 包解压
2.  将wars/目录下 `activiti-app.war`, `activiti-rest.war` 包放入apache-tomcat-9.0.30/webapps目录下

```text
apache-tomcat-9.0.30/bin/catalina.bat start   # 启动 tomcat，stop
```

自动解压war包并布署，webapps/ 下生成两个目录 `activiti-app/` , `activity-rest/`

1.  `activiti-app` 会创建用户 admin 密码 test
2.  `activiti-rest` 会创建用户 fozzie gonzo kermit 密码同用户名
3.  测试activiti-app <http://localhost:8080/activiti-app/>  admin:test
4.  测试activiti-rest <http://localhost:8080/activiti-rest/docs/> swagger文档 kermit:kermit
    <http://kermit:kermit@localhost:8080/activiti-rest/service/management/engine> 必须使用用户名密码进行 basic auth

默认使用H2内存数据库，数据不能持久化，mysql库替换，这里使用的mysql库是v8.0.12

1.  新建数据库名 `activiti6ui`
2.  下载相应mysql-connector-java-8.0.12.jar包放入apache-tomcat-9.0.30/lib/, 重命名mysql-connector-java.jar 可选
3.  activiti-app修改mysql连接配置

    cat apache-tomcat-9.0.30\webapps\activiti-app\WEB-INF\classes\META-INF\activiti-app\activiti-app.properties

    ```bash
    datasource.driver=com.mysql.jdbc.Driver
    datasource.url=jdbc:mysql://127.0.0.1:3306/activiti6ui?useUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
    datasource.username=root
    datasource.password=root
    hibernate.dialect=org.hibernate.dialect.MySQLDialect
    ```

    注意:  <span class="underline">&nullCatalogMeansCurrent=true</span> 参数 必须是要带的，否则创建表时会出错 大坑

4.  activiti-rest修改mysql连接配置

    cat apache-tomcat-9.0.30\webapps\activiti-rest\WEB-INF\classes\db.properties

    ```bash
    db=mysql
    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql://127.0.0.1:3306/activiti6ui?useUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true
    jdbc.username=root
    jdbc.password=root
    ```

重启 tomcat 部署war包的时候，如果有对应的表则会更新，否则会自动创建所有结构及数据

`activiti-app` 里显示流程图中文显示正常， `activiti-rest` 使用 'runtime/process-instances/{processInstanceId}/diagram'
接口获取流程图状态的时候，中文显示为方框

github下载 activiti-6.0.0 源码解压并使用IDEA 导入

定位 activiti-6.0.0\modules\activiti-image-generator\src\main\java\org\activiti&image;\impl\DefaultProcessDiagramCanvas.java

如下图修改字体，然后重新打包成 [activiti-image-generator-6.0.0.jar](https://github.com/emacle/php-activiti-api/blob/master/doc/activiti-image-generator-6.0.0.jar)
![](/ox-hugo/work.org_20200506_081933.png)

替换apache-tomcat-9.0.30\webapps\activiti-rest\WEB-INF\lib中旧jar包, 效果如下
![](/ox-hugo/Snipaste_2020-05-05_19-25-12.png)

system\_time\_zone
time\_zone SYSTEM
set global time\_zone='+8:00'
