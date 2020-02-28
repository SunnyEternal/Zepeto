# Zepeto文档

## 崽崽`H5`中常用`Api`和`deeplink`

[TOC]

#### 待完善功能

1. 现只有分享到 `Line`和 `Twitter`的功能。暂无分享到 `WeChart`、`QQ`空间等功能。
2. 从站外呼起Zepeto的 `deeplink`，待确认是否有。
3. 因为是 `3D`的，在 `H5`里给崽崽换装比较麻烦，所以暂不支持在 `H5`里直接给崽崽换装。

### 1. 多次连续退出再进入`H5`，页面缩放到很小

**问题描述：**
>`Zepeto`全屏`webview`改成卡片形式后，多次连续退出再进入`H5`，会出现页面根字体字号小于`10px`，导致页面缩放到很小。

**原因：**
>`webview`宽度还没有和设备宽度一致，`js`就已经开始获取可视窗口宽度计算根字体字号大小了。

**解决办法：**
>开启`setInterval`获取可视窗口宽度，并计算根字体字号大小，当大于`10px`赋值给根字体，结束获取。

**备注：**
>不同分辨率的手机，当页面缩放很小时，计算出的根字体字号都小于`10px`。

```JavaScript
function setRootFont() {
  let fontTimer = setInterval(() => {
    let rootFontSize = (document.documentElement.clientWidth || document.body.clientWidth) / 750 * 100;
    if (rootFontSize > 32) { // 我设置默认根字体是32
      document.getElementsByTagName('html')[0].style.fontSize = rootFontSize + 'px';
      document.getElementsByTagName('body')[0].style.width = (document.documentElement.clientWidth || document.body.clientWidth) + 'px';
     clearInterval(fontTimer)
    }
  }, 60)
  let timer = setTimeout(() => {
    clearInterval(fontTimer)
    clearTimeout(timer)
  }, 3000)
}
```

### 2. 获取崽崽个人信息——`window.ZEPETO`全局属性

```JavaScript
function getUserInfo() {
  let timer1 = setInterval(() => {
    if (window.ZEPETO) {
      console.log(window.ZEPETO)
      let headImgUrl = window.ZEPETO.userInfo.profilePic.substring(0, window.ZEPETO.userInfo.profilePic.lastIndexOf('?')) 
      this.userInfo = {
        zepetoId: window.ZEPETO.userInfo.hashCode,
        headImg: headImgUrl,
        nickname: window.ZEPETO.userInfo.name,
        userId: window.ZEPETO.userInfo.userId,
        authToken: window.ZEPETO.userInfo.authToken,
        userAgent: window.ZEPETO.userAgent || '-', // 老版本没有该信息
        duid: window.ZEPETO.duid || '-', // 老版本没有该信息
        UA: 'zepeto.service.zzz.crew/1.0.0'
      }
      localStorage.setItem("token", this.userInfo.authToken)
      localStorage.setItem('$zepetoUserInfo', JSON.stringify(this.userInfo))
      clearInterval(timer1) 
    }
  }, 20)
  let timer2 = setTimeout(() => {
    clearInterval(timer1)
    clearTimeout(timer2)
  }, 3000)
}
```

### 3. 在`App`中打开`H5`调试页面，对应`deeplink`：

```JavaScript
ZEPETO://HOME/WEBVIEW?url=[h5链接]
```

### 4. 关闭崽崽中`H5`页面，对应`deeplink`：

```JavaScript
<a href="ZEPETO://WEBVIEW/CLOSE">关闭图标<a>
```

### 5. 去个人中心，对应`deeplink`：
```JavaScript
window.location.href = `ZEPETO://HOME/PROFILE/CARD?${hashCode}`;
```

### 6. 去换衣服/试衣间，实际是去商店，对应`deeplink`：
```JavaScript
window.location.href = `ZEPETO://HOME/SHOP/COSTUME?referrer=${referrerUri}`; // 去商店
```

### 7. 保存图片到相册，对应`deeplink`：
```JavaScript
function saveImage(callback) {
  let base64 = document.getElementById(`photo`).src; // 获取图片base64字符串
  let newBase64 = base64.substring(base64.indexOf(',')+1); // 去掉开头"data:image/png;base64,"
  if (window.ZEPETO) {
    window.ZEPETO.invoke('SaveImage', newBase64, callback); // callback可以是保存成功的提示
  }
}
```

### 8. 上传`Feed`，先将`base64`格式的图片保存到手机系统相册，再使用`deeplink`拉起上传到`Feed`的页面。

```JavaScript
window.location.href = 'ZEPETO://HOME/FEED/UPLOAD/CAMERAROLL'
```

**备注：**
>韩国已经开发上传到`Feed`的接口，测试阶段。

### 9. 获取透明背景的崽崽人物形象图
**请求方式：`POST`**
**API：https://openapi.zepeto.cn/graphics/booth/`<PHOTOBOOTH_NAME>`?targets=`<HASHCODES>`&service=zaizai&width=`<WIDTH>`&cdn=`<OPTION>`&cache=`<OPTION>`**

**参数列表:**

| 变量 | 描述 |
| --- | --- |
| <PHOTOBOOTH_NAME> | `poseId`序列，多个时逗号分隔
| <HASHCODES> | `hashCode`序列，多个时逗号分隔
| <WIDTH> | 渲染图的宽度 |
| <OPTION> | `true` 或 `false` |

```JavaScript
import request from '@/utils/request'
return request({
  url:`https://openapi.zepeto.cn/graphics/booth/${data.renderData}?targets=${data.hashCode}&service=zaizai&width=500&cdn=on&cache=off`,
  method: 'post'
})
```

### 10. 查看崽崽已有的所有`item`（即不只是身上穿的），一般用于抽奖活动
>抽奖，要先查询抽到的奖品是不是用户已有的（不是身上穿，是已有的所有`item`）

### 11. 打招呼



### 13. `jsBridge`——渲染成背景透明的崽崽人物形象图片【暂不使用】

崽崽[可多人][可不同`pose`]渲染成背景透明的图片/模板，`App`支持版本`2.9.2+`。
`jsBridge`的`demo`地址：
>https://github.com/SunnyEternal/Zepeto_jsBridge

