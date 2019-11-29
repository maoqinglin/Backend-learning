

## 1、js 打开 App

```javascript

$('.keymap-share').on('click', function () {
    if (KeySchema.schemaLoaded) {
        //keyadapter://jp.app/openwith?sharetime=10086&sn=1111111
        var url = "keyadapter://jp.app/openwith?sharetime=" + 
            KeySchema.shareTime + "&sn=" + KeySchema.sn;
        $(this).attr("href", url);
    } else {
        $.toast("分享失败，请确保使用的是摩奇i7及以上版本", "text");
    }
})
```



## 2、Ajax封装

```javascript
function doRequest(options, callback) {
    $.ajax({
        type: options.type,
        url: options.url,
        data: options.data,
        dataType: 'json',
        cache: false,
        success: function (data) {
            if (data == null) {
                //$.toast('返回数据为空');
                callback.failCallback();
                return;
            }
            if (data.CODE == IS_SUCCESS) {
                callback.successCallback(data);
            } else {
                callback.failCallback(data);
            }
        },
        error: function (xhr, type) {
            //$.toast('Ajax error! xhr.status = ' + xhr.status);
            if (typeof callback.errorCallback !== 'undefined') {
                callback.errorCallback(xhr);
            }
        }
    })
}
```

## 3、获取 url 中的参数

```javascript
function getQueryString(name) {
    var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
    var r = window.location.search.substr(1).match(reg);
    if (r != null) return unescape(r[2]);
    return null;
}
```

## 4、数组排序

```javascript
valueOfSchemeWithKeys: function (schemes) {
    var keyList = [];
    var appName = schemes.APPNAME;
    var pkgName = schemes.APPPACKAGENAME;
    var verCode = schemes.VERSIONCODE;
    var startKey = schemes.DEFAULT_KEY;
    var orderId = schemes.ORDER_ID;
    var operator = schemes.SCHEME_PLATFORM;
    var scheme = this.valueOfSchema(appName, pkgName, verCode, startKey, orderId, operator)
    for (var key of schemes.KEYORDER) {
        var keyWrap = this.valueOfKey(key.ID, key.POSITION, key.TIME);
        keyList.push(keyWrap);
    }
    //按键位置排序
    keyList = keyList.sort(function (obj1, obj2) {
        var pos1 = obj1.keyPos;
        var pos2 = obj2.keyPos;
        if (pos1 < pos2) {
            return -1;
        } else if (pos1 > pos2) {
            return 1;
        } else {
            return 0;
        }
    });
    scheme.keys = keyList;//添加方案的按键列表
    return scheme;
}
```

