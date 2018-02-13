---
title: GET/POST下载excel
date: 2018-02-01
categories: work
tags: excel
---

一般的后台管理系统有个很常见的需求是下载excel。
前端实现有两种方法
1.get请求，利用诸如a标签等直接发起一次get请求，浏览器会自行处理后端接口返回的二进制流
优点：简单
缺点：如果需要先上传大量数据，后端根据上传的数据生成excel，这种场景满足不了。（当然可以先用post提交数据，再用get二次请求，这是另外一回事了）

2.post请求
上面的GET方法不能很好的满足先上传数据，再下载excel的要求，我们使用POST上传数据和下载excel。
利用jquery/axios等封装好的POST方法，保存的excel是无法打开的。原因是这些库提供的方法会将返回的数据转换为string，而不是blob类型。

```javascript
    $.post(url, data, function(data) {
        console.log(typeof data);        // 得到的是string类型
        ...
    });
```

这里提供一个利用原生ajax实现post下载excel的方法
```javascript
    function onDownloadExcel(options) {
        const {
            url,
            data
        } = options;

        const xhr = new XMLHttpRequest();
        xhr.open('POST', url, true);

        // 很重要
        xhr.responseType = 'blob';
        xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
        xhr.onload = function () {
            if (xhr.readyState === 4 && xhr.status === 200) {
                const reader = new FileReader();

                // 转换为base64，可以直接放入a标签href
                reader.readAsDataURL(xhr.response);
                reader.onload = function (e) {
                    const a = document.createElement('a');
                    a.download = 'myExcel.xlsx';
                    a.href = e.target.result;
                    $('body').append(a);
                    a.click();
                    $(a).remove();
                }
            }
        };

        // 发送你自己的数据data
        xhr.send(...);
    }

```