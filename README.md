# 移动CRM Ver1.0项目设计

标签（空格分隔）： 项目设计

---

##项目概述

---
##主要功能
###1.整体样式布局

 - 详情请见PMT

###2.门店KP展开收缩显示

 - 经济人详细页面，门店KP点击展开所有图标和显示进行变化

###3.搜索交互

 - 搜索类型，从经纪人列表进入时默认为“经纪人”，从门店列表进入时默认为“门店”
 - 点击下来按钮对门店和经纪人进行切换
 - 进入页面时自动激活光标（弹出手机输入法），默认显示：“请输姓名、手机号或门店名称”。输入后手机软键盘回车变为开始搜索。
 - 点击input框自动补全
 - 点击回车键进行搜索
 - 记录并存储历史搜索记录
 - 如果是第一次进来不显示搜索记录，显示该类型最近的5条搜索记录。
 - 点击清除历史记录，删除搜索记录
 - 点击X清除输入内容
 - 点击取消鼠标失去input焦点

---
##技术方案
###1.整体样式布局

 - reset.css重置默认浏览器样式
 - common.css公共部分样式（包括头部，底部，公共的字体，连接，线条，以及图标）
 - page.css各个页面的CSS样式

###2.门店KP展开收缩显示
```javascript
function onOff(obj){
    //元素展开收缩
}
```
###3.搜索交互
- 搜索类型，从经纪人列表进入时默认为“经纪人”，从门店列表进入时默认为“门店”
```
function getTag(){
    //判断从哪个列表进入
}
```
- 进入页面时自动激活光标（弹出手机输入法），默认显示：“请输姓名、手机号或门店名称”。输入后手机软键盘回车变为开始搜索。
```javascript
functon inputFocus(e){
    //默认激活状态及显示'请输姓名、手机号或门店名称'
}

```
- 经济人和门店切换
```javascript
function selectSearch(id) {
            var obj = J.g(id),
                list = obj.s('.item'),
                showBox = J.g(id + '_box'),
                selectUl = J.g(id + '_ul')
            selectBg = J.g(id + '_bg'),

            list.each(function(i, v) {
                v.on('click', function(e) {
                    selectBg.setStyle({
                        'display': 'none'
                    });
                    e.stopPropagation()
                    e.preventDefault();
                    tab(i);


                })
            })

            function tab(i) {
                var sel_i = list.eq(i);
                showBox.down(0).html('').html(sel_i.s('span').eq(0).html());
                selectUl.setStyle({
                    'display': 'none'
                });
                obj.attr('data-name', sel_i.attr('data-name'));
                obj.attr('searchTag', sel_i.attr('data-name'));
                searchBox.up().attr('searchTag', sel_i.attr('data-name'))
                searchHidden.val(sel_i.attr('data-type'));
                searchInput.attr('placeholder',sel_i.attr('data-input'))
                tag = sel_i.attr('data-name');
                startHistory(tag);
                searchInput.val('');
                autoComplateObj.config.url = getAjaxUrl();
            }
            selectBg.on('click', function(e) {
                selectUl.setStyle({
                    'display': 'none'
                });
                selectBg.hide();
                e.stop();
            })
        }
    };
```
- 搜索自动补全
```javascript
        /*搜索自动补全*/
        function crmSearchAutoComplate(inputobj, url) {
            var autoComplate_config = {
                url: url
            }
            r = {
                offset: {
                    y: 0
                },
                parentWidth: '100%',
                autoSubmit: false,
                cache: false,
                tpl: 'search_result',
                itemBuild: function(item) {
                    if (item.name && item.isSkip) {
                        return {
                            l: item.name,
                            v: inputVal,
                            isSkip: true
                        }
                    } else if (item.name) {
                        return {
                            l: item.name,
                            v: new String(item.name)
                        }
                    }

                },
                onSelect: function(data) {
                    var obj = {
                        id: data.id,
                        name: data.name
                    }
                    searchJump(data.name, data.id);
                    if (pos !== 'Add_Broker_TelFollow') {
                        crmsearch.addHistory(tag, obj);
                    }
                },
                source: function(paras, resoponse) {
                    if (paras.q === "") return resoponse([]);
                    J.get({
                        url: autoComplate_config.url + '?searchText=' + paras.q,
                        type: 'json',
                        onSuccess: function(data) {
                            if ((/broker/).test(autoComplate_config.url)) {
                                getData(data.brokerList, resoponse);
                            } else if ((/store/).test(autoComplate_config.url)) {
                                getData(data.storeList, resoponse);
                            }
                            if (J.s('.search_result').eq(0).s('p').length > 0) {
                                J.s('.search_result').eq(0).s('p').eq(0).on('click', function() {
                                    location.href = getUrl() + inputVal;
                                })
                            }
                        }
                    })

                },
                onChange: function(e) {

                }
            }

            var autocompleteObj = inputobj.autocomplete(r);
            autocompleteObj.config = autoComplate_config;
            return autocompleteObj;
        }
```
- 搜索获取数据
```javascript
//点击回车键自动提交
function getData(data, resoponse) {
            if (data.length == 0) {
                noResult.setStyle({
                    'display': 'block'
                });
                J.s('.search_result').eq(0).setStyle({
                    'display': 'none'
                });
            } else {
                noResult.setStyle({
                    'display': 'none'
                });
                J.s('.search_result').eq(0).setStyle({
                    'display': 'block'
                });
                if (pos == 'Add_Store_VisitLog' || pos == 'Add_Broker_TelFollow') {
                    resoponse(data);
                } else {
                    data.unshift({
                        name: '搜索“<span>' + inputVal + '</span>”<i class="crmicon search_arrow"></i>',
                        isSkip: true
                    })
                    resoponse(data);

                }
            }
        }
```
-获取页面url
```
function setPara(args){
            var cls="";
            var url="";
            switch (args) {
                case 'store_visitLog_list':
                    cls='visited-list-items';
                    url=basePath+"/broker/viewbrokerInfoListAjax";
                    break;
                case '2':
                    break;
            }
            return {
                cls:cls,
                url:url
            }
        }

        var args=setPara(J.s("head").eq(0).attr("data-page"));
        
        args.url
        args.cls
        
        window.location(args.url)
```

- 历史记录存取
```javascript
 //记录搜索历史组件
    var crmHistory = function(option) {
        var defaultOptions = {
            maxNum: 5,
            storageKey: '',
            storageName: 'id'
        }
        var opt = J.mix(defaultOptions, option || {}, true);
        var tag = opt.storageKey;

        function getHistorys(tag) {
            var history = window.localStorage.getItem(tag);
            if (history) {
                return JSON.parse(history)
            }
        }

        function addHistory(tag, item) {
            var items = getHistorys(tag);
            console.log(items);
            if (items && items.length > 0) {
                for (var i = 0, len = items.length; i < len; i++) {
                    if (item[opt.storageName] && items[i][opt.storageName] && item[opt.storageName] == items[i][opt.storageName]) {
                        items.splice(i, 1);
                        break;
                    }
                }
                items.push(item);
                if (opt.maxNum > 0 && items.length > opt.maxNum) {
                    items.splice(0, 1)
                }
                var json = JSON.stringify(items);
                window.localStorage.setItem(opt.storageKey, json);
            } else {
                var temp = [item];
                window.localStorage.setItem(opt.storageKey, JSON.stringify(temp));
            }
        }

        function hasHistory(tag) {
            var items = getHistorys(tag);
            if (items && items.length > 0 && items != 'undefined') {
                return true;
            } else {
                return false;
            }
        }

        function clearHistory() {
            window.localStorage.setItem(opt.storageKey, '');
        }

        function changeKey(n) {
            opt.storageKey = n;
        }
        return {
            getHistorys: getHistorys,
            addHistory: addHistory,
            clearHistory: clearHistory,
            changeKey: changeKey,
            hasHistory: hasHistory
        }
    }
```


---



##前后端数据接口
####1. 自动补全接口
```javascript
自动补全需要的传输参数：url?sid='门店'&q='搜索内容'
[
"李安琪",
"李新",
"李里",
"李飞"
]
```
####2. 搜索结果后端接口
```javascript
搜索结果需要的传输参数：url?stores='门店'&q='搜索内容'
    [{
        "id":"218748",
        "name":"李安琪",
        "stores":"我爱我家某浦店路店",
        "account_over":"2500",
        "sign_time":"2014-07-30",
        "status":"活跃"
    },
    {
        "id":"218748",
        "name":"李丽",
        "stores":"我爱我家某浦店路店",
        "account_over":"2500",
        "sign_time":"2014-07-30",
        "status":"注册"
    }]
```
---
###问题

 - 什么时候开始记录历史搜索（有无点击搜索）
 - 门店和经纪人是否为公用接口

##时间排期
| 功能        | 项目估期   |
| --------   | -----:  |
| 整体样式布局     | 24h |
| 门店KP展开收缩显示        |   4h   |
| 嵌入后台数据        |   16h   |
|搜索交互|32h|




