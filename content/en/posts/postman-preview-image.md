+++
title = "postman预览二进制图片"
author = ["emacle"]
date = 2020-05-06T09:30:00+08:00
tags = ["postman"]
draft = false
authorEmoji = "🎅"
image = "images/feature2/workflow.png"
pinned = false
+++

使用 base64 编码二进制图片数据，置入 `<img>` 标签

```php
<?php
try {
    // getDiagram 获得二进制流图片 可显示流程进度
    $diagramBinary = $serviceFactory->createProcessInstanceService()->getDiagram($processInstanceId);
    $base64 = 'data:image/png' . ';base64,' . base64_encode($diagramBinary); // base64加密二进制图片
    echo '<img src="' .  $base64 . '" />'; // binary image to base64 so postman can preview image
} catch (ActivitiException\ActivitiException $e) {
    var_dump($e->getMessage());
}
```

效果图：
![](/ox-hugo/work.org_20200506_092753.png)
