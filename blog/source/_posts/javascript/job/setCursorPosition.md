---
title: js获取文本框中光标索引位置
date: 2017-05-20
tags: work
---

工作中用到了，就mark下

```javascript
// js获取文本框中光标索引的位置
function getInputTextCursorPosition(){
    var obj = document.getElementById('input');

    // 非IE浏览器
    if (obj.selectionStart) {
        return obj.selectionStart;
    }

    // IE
    var range = document.selection.createRange();
    range.moveStart('character', -obj.value.length);
    return range.text.length;
}

// js设置文本框中光标索引的位置
function setInputTextCursorPosition(index){
    var obj = document.getElementById('input');

    // 非IE浏览器
    if (obj.selectionStart) {
        obj.selectionStart = index;
    }

    // IE
    var range = obj.createTextRange();
    range.move('character', index);
    range.select();
}
```