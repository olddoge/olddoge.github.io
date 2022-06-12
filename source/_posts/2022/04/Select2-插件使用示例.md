---
title: Select2 插件使用示例
date: 2022-04-23 20:27:30
tags: [JS, 教程, select2, 下拉选择]
categories: JavaScript
description: Select2 下拉框使用案例
---

示例代码

```javascript
let select2Setting = {
    width: 260, // 宽度
    minimumResultsForSearch: -1,    // -1 时，下拉框中的输入框不显示
    // ajax 获取下拉选项数据
    ajax: {
        url: '请求地址',
        dataType: 'json',   // 数据类型
        processResults(response) {
            // return 需要返回带 results 键的对象
            return {results: response}
        }
    },
    templateResult: (state) =>  {
        // 下拉时，下拉选项
        let id = state.id   // 对应 option 的 value
        let text = state.text   // 对应 option 的 text
        // 下面是给下拉选项加图片的例子
        return $(`<span><img src="${imgPath}" class="img-responsive" style="width: 10%;display: inline" alt="${text}" /> ${text}</span>`);

    },// 选择时
    templateSelection: (state) => {
        // 这里的方法尽量要和 templateResult 中一致
        let id = state.id    // 对应 option 的 value
        let text = state.text   // 对应 option 的 text
        // 下面是给下拉选项加图片的例子
        return $(`<span><img src="${imgPath}" class="img-responsive" style="width: 10%;display: inline" alt="${text}" /> ${text}</span>`);
    } // 选择后
}
$('.element').select2(select2Setting)
```