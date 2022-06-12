---
title: jQuery DataTable 使用示例
date: 2022-04-23 20:10:57
tags: [JS, 教程, jqueryDataTable, 表格]
categories: JavaScript
description: JQuery DataTable 常用案例
---

示例代码

```javascript

let settings = {
    bAutoWidth: false,  // 自动宽度
    serverSide: true,   // 服务端启用分页，开启分页后，需要服务端计算总数，false 的时候，需要一次性返回所有数据给表格
    bFilter: false, // 自带搜索框是否显示
    ordering: false,    // 是否需要排序功能
    scrollY: false,   // 垂直滚动 这里后期改成窗口高度
    aoColumns: [
        {
            title: '页面上显示的表格头标题',
            className: 'text-center',   // 表头的样式
            orderable: false,   // 是否需要排序，全局排序为 false，这个值无意义
            width: 45,  // 表头的宽度
            data: 'id', // 对应服务端响应数据的名称
            render: (data, type, full, meta) => {
                // data - 服务端返回的数据值
                // type
                // full - 服务端返回的全部数据
                // meta - 行索引 0 为起始
                return data
            }
        }
        // ....
    ],
    language: {
        decimal: '',
        emptyTable: '暂无数据',
        info: '第 _START_ 到 _END_ 条，共 _TOTAL_ 条 | ',
        infoEmpty: '',
        infoFiltered: '(从 _MAX_ 条结果中过滤)',
        infoPostFix: '',
        thousands: ',',
        lengthMenu: '_MENU_ 条/页',
        loadingRecords: '加载中...',
        processing: '加载中...',
        search: '搜索:',
        zeroRecords: '暂无匹配数据',
        paginate: {
            first: '第一页',
            last: '最后一页',
            next: '>',
            previous: '<'
        },
        aria: {
            sortAscending: ': 以升序排列此列',
            sortDescending: ': 以降序排列此列'
        },
        select: {
            rows: {
                _: "选中 %d 行",
                0: "点击选中行",
                1: "选中 1 行"
            }
        }
    },
    aaSorting: [],  // 允许排序的字段
    bProcessing: true,  // 加载表格的进度条
    buttons: [
        {
            text: '按钮的文字',
            className: 'btn btn-xs btn-success jqtable-btn',    // 按钮的类名
            action: (event, dt, node, config) => {
                // 当点击按钮时，需要做的事情
            }
        }
        // ...
    ],    // 是否需要按钮，这个需要引入 jQuery.DataTables.button 这个 js
    select: false,  // 表格行是否可以被选中
    /**
     * DOM 参考
     *
     * <div class="row">
     *     <div class="col-xs-3">{Button - B}</div>
     *     <div class="col-xs-9 search-area"></div>
     *     {process - r}
     *     {table - t}
     *     <div class="row">
     *         <div class="col-xs-6">
     *             <div class="inline">{info - i}</div>
     *             <div class="inline">{length - l}</div>
     *         </div>
     *         <div class="col-xs-6">{page - p}</div>
     *     </div>
     * </div>
     *  
     *  B - Button 按钮
     *  r - process 进度条
     *  t - table 表格
     *  i - info 数据信息
     *  l - length 数据长度
     *  p - page 分页
     *  < - <div>
     *  > - </div>
     *  'xxx' - '类名'
     */
    dom: "<'row'<'col-xs-3'B><'col-xs-9 search-area'>r>t<'row'<'col-xs-6'<'inline'i><'inline'l>><'col-xs-6'p>>",
    initComplete: (settings, json) => {
        // 当表格第一次初始化后的事件
    },
    drawCallback: (settings) => {
        // 当表格每次重绘的事件
    },
    ajax: (data, callback, settings) => {
        // ajax 请求获取数据
        let pagesize = data.length; // 页面显示记录条数，在页面显示每页显示多少项的时候,页大小
        let start = data.start; // 开始的记录序号
        let page = start / pagesize + 1; //当前页码
        $.ajax({
            type: option.ajax.type || 'GET',
            url: option.ajax.url,
            cache: false,
            data: {},
            success: function (response) {
                // 成功请求后，一定要返回这个对象，对象的键参考如下
                let return_data = {
                    recordsTotal: response.total || 0,
                    recordsFiltered: response.total || 0,
                    data: response.data || []
                }
            },
            complete: function () {

            },
            error: function () {
                let return_data = {
                    recordsTotal: 0,
                    recordsFiltered: 0,
                    data: []
                }
                callback(return_data)
            }
        })

    }
}
$('.elements').dataTable(settings)

// 表格重载事件
$('.elements').api().ajax.reload()
```