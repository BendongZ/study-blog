# 开发过程中的问题集合

## 1.vue中使用异步组件问题，通过一种自动引入组件并注册是不能实现异步的
![yibu](/work/Vue/problem/problem1-2.png)
上图中不能把组件模块单独抽离出来，只有使用原始的方法才行，如下图
```js
import rollTxt from './roll-txt.vue'
```
![yibu](/work/Vue/problem/problem1-1.png)

## 2.组件多，图片转成base64也多，导致业务js文件大
分离出这样的含有图片多的组件，把该组件当成异步组件处理，打包的时候会被分割，与业务js分离

## 3.微信禁止复制，禁止刷新
```js
wx.hideMenuItems({
    menuList: ['menuItem:copyUrl','menuItem:openWithSafari','menuItem:openWithQQBrowser'] // 要隐藏的菜单项，只能隐藏“传播类”和“保护类”按钮，所有menu项见附录3
})
```

## 4.ie版本判断
```js
// 判断IE的版本
checkIEVersion: function (version) {
    var b = document.createElement('b');
    b.innerHTML = '<!--[if IE ' + version + ']><i></i><![endif]-->';
    return b.getElementsByTagName('i').length === 1
}

// 返回值的布尔值，使用checkIEVersion(9)
```

## 5.合并对象的函数
```js
export default function (target) {
    for (let i = 1, j = arguments.length; i < j; i++) {
        let singleData = arguments[i] || {};
        for (let key in singleData) {
            if (singleData.hasOwnProperty(key)) {
                let value = singleData[key]
                if (value !== undefined) {
                    target[key] = value
                }
            }
        }
    }
    return target
}
```

## 6.iscroll滑动加载更多，多次请求问题
如果用户短时间内多次滑动，会出现pageNo一直增加，导致请求数据是最后一次的，所以在要对该情况做请求限制，在第一次请求没有完成，不允许发送第二次请求

## 7.华为meta7，android版本是4.4.2，滑动不加载数据
监听touchstart，touchmove，touchend三个事件，如果是滑动的话，只会触发touchstart，touchmove，如果只点击的，会触发touchstart，touchend，很尴尬没有像其他手机一样，有滑动一下子触发三个事件，为了排查问题，引入腾讯的vconsole.js文件，触发滚动请求加载数据就出错，但是在其他手机上不会，删除vconsole.js，也不要touchend事件，就只要touchstart，touchmove，但是如果不做限制的话，滑动很多次，会触发很多次的请求，浪费资源，做一个请求限制，数据没有回来之前不去下一次请求，最后代码如下：
```js
var startY, offsetY, maxScrollY;
    $("body").on('touchstart', function (e) {
        startY = e.touches[0].clientY;
    });
    $("body").on('touchmove', function (e) {
        offsetY = e.touches[0].clientY - startY;
        maxScrollY = $(this).height() - window.innerHeight;
        if ((window.scrollY + 2) > maxScrollY && offsetY < 0) {
            if (_this.loading) {
                return false;
            }
            //判断是否需要分页加载
            _this.setLoadTxtDown(true, false, false);
            setTimeout(function () {
                if (_this.loading) {
                    _this.pageNo++;
                    _this.ajaxFn(_this.url, {
                        userId: _this.userId,
                        token: _this.token,
                        pageNo: _this.pageNo,
                        pageSize: _this.pageSize
                    }, function (oData) {
                        var result = oData;
                        _this.setLoadTxtDown(false, true, false);
                        _this.contentList = _this.contentList.concat(result.lotteryRecordList);
                        if (!result.lotteryRecordList.length) {
                            _this.setLoadTxtDown(false, false, true);
                            // 如果没有数据了就撤销touch事件
                            $("body").off();
                        }
                    });
                }
            }, 1500);
        }
    });
```

## 8.ios 微信浏览器中，图片作为背景，会产生一条白色的底边
解决方法

html布局

![vue](/work/Vue/vue4.png)

css样式如下：

![vue](/work/Vue/vue5.png)

给背景层的div加上如下的css样式
```css
margin-bottom: -1px;
```

## 9.ios中keyup事件有时候监听没有效果
最后改用原生js的input事件，解决问题

## 10.手机中低版本的系统不支持css3的一些属性，比如calc动态计算
计算没有成功，导致滚动加载外框和内容的高度是一样，这样不会触发滚动条件，改用document.body.clientHeight获取手机的实际高度

## 11.魅蓝note5上边，滚动加载，发现问题 window.scrollY 始终比 maxScrollY 小2px
```js
// 导致了 window.scrollY >= maxScrollY && offsetY < 0 该条件不成立，不会滚动加载
$("body").on('touchmove', function (e) {
    // console.log(2);
    offsetY = e.touches[0].clientY - startY;
    maxScrollY = $(this).height() - window.innerHeight;
});

// 解决方式：
( window.scrollY+2) >= maxScrollY && offsetY < 0
```

## 12.数据埋点在路由中的使用问题
数据埋点中，使用了location.hash = route.fullpath 手动触发hash改变，以便能监听到hashchange的改变，然后在window.addEventListener('hashchange',function(){//处理页面PV,UV上报})，这时实际上跳转页面的时候，hash值已经改变，然后我们又手动的改变了hash值，只是改变的值和原来的hash值是一样的，用户看不出变化了，但这条记录已经在浏览器的历史记录中了，使用replace只是替换了第一条历史记录，手动触发hash值改变的这条并没有被替换，上报数据有误，故该方案pass

解决方式：
```js
// 数据埋点js中添加一个上报PV,UV的方法，比如pushReport

    /**
     * 路由页面PV,UV上报
     * pushReport
     * @param {*} fromPath
     * @param {*} toPath
     * @memberof DataReport
     */
    pushReport(fromPath, toPath) {
        let times = Date.now();
        this.pushData.isPv = 1
        this.pushData.timestampLeft = times
        this.pushData.url = fromPath
        this.pushData.pushData(false)
        this.pushData.timestampLeft = ''
        this.pushData.timestampEnter = times
        this.pushData.url = toPath
        this.pushData.pushData(false)
    }
// 在入口文件中Vue的实例下添加watch对$route的监听，切换路由上报数据埋点信息
new Vue({
	el: '#app',
    router,
    store,
    watch: {
        $route(to, from) {
            let toPath = `${location.origin}${location.pathname}#${to.path}`,
            fromPath = `${location.origin}${location.pathname}#${from.path}`
            // 绑定在Vue原型上的数据埋点的实例化对象
            Vue.prototype.DR.pushReport(fromPath, toPath)
        }
    },
	render: h => h(App)
})


```

## 13.打开构建好的页面，在低版本的android手机上显示空白页面
低版本安卓手机访问页面发现是空白，不知道是怎么回事儿，找客户端的同学打印了加载网页的日志，发现是语法错误，怎么回事儿呢，仔细查看发现中转页的代码时es6写的，比如let，const，箭头函数都没有转换成es5，所以加载不了，进入到众邦宝的首页，发现还是空白，继续搜索，找到了打包之后的vendors.js文件发现有些es6的关键字没有被转化，排查原因，babel-loader转换js时，没有转化node_modules中的代码文件，但是我们私有包使用es6写的，修改babel-loader解析范围，最终解决问题

## 14.输入框中的字体被光标挡住一点
输入框光标压字问题，设置input的文字间距即可（letter-spacing: 1.5px）

## 15.路由某些页面缓存
在根路由下又嵌套了路由，这个时候在嵌套了的路由中使用keep-alive可以缓存嵌套中的路由页面，但是在根路由下的一些页面跳回到嵌套的页面中时，发现keep-alive不起作用了

最后在根路由下添加keep-alive包裹，然后在路由表中设置需要缓存页面的值，比如名字叫keepAlive: true
```html
<template>
    <div>
        <keep-alive>
            <router-view v-if="$route.meta.keepAlive"></router-view>
        </keep-alive>
            <router-view v-if="!$route.meta.keepAlive"></router-view>
    </div>
</template>
```
不需要缓存的页面就不设置keepAlive，v-if会判断然后显示路由

## 16.ios是不允许自动聚焦的
ios是不允许自动聚焦的，但android是可以的，如果写上自动聚焦的代码，不会报错，只是不会起作用，但会导致ios聚焦的时候，出现光标飞入的情况，影响用户体验

## 17.全局样式设定，比如修改html，body的样式，如果不设置scoped会影响其他组件
全局样式设定，比如修改html，body的样式，最好在每个组件中都写上scoped，如果要修改全局的样式，请在组件中另写一个style不加scoped，又或者使用原生js的方式去获取html，body元素修改样式，一个组件中不写scoped的话会影响到其他组件的样式

## 18.swiper修改分页器样式
但是在style中scoped作用内始终没有效果，原来swiper是挂在到vue mounted之后的，并没有在template模板内，所以加不上，在同一个vue的组件中，在写一个style不加scoped，重写swiper的样式，就是全局的了，能覆盖swiper的样式，如果每个页面的body的样式不一样，那么我们可以在组件中写一个style去设置body的颜色

## 19.rsa使用加密
```js
import JSEncrypt from '@zdmission/rsa'
console.log(JSEncrypt)
let encrypt = new JSEncrypt();
encrypt.setPublicKey('-----BEGIN RSA PRIVATE KEY-----\n'+
        'MIIBOQIBAAJAVY6quuzCwyOWzymJ7C4zXjeV/232wt2ZgJZ1kHzjI73wnhQ3WQcL\n'+
        'DFCSoi2lPUW8/zspk0qWvPdtp6Jg5Lu7hwIDAQABAkBEws9mQahZ6r1mq2zEm3D/\n'+
        'VM9BpV//xtd6p/G+eRCYBT2qshGx42ucdgZCYJptFoW+HEx/jtzWe74yK6jGIkWJ\n'+
        'AiEAoNAMsPqwWwTyjDZCo9iKvfIQvd3MWnmtFmjiHoPtjx0CIQCIMypAEEkZuQUi\n'+
        'pMoreJrOlLJWdc0bfhzNAJjxsTv/8wIgQG0ZqI3GubBxu9rBOAM5EoA4VNjXVigJ\n'+
        'QEEk1jTkp8ECIQCHhsoq90mWM/p9L5cQzLDWkTYoPI49Ji+Iemi2T5MRqwIgQl07\n'+
        'Es+KCn25OKXR/FJ5fu6A6A+MptABL3r8SEjlpLc=\n'+
        '-----END RSA PRIVATE KEY-----');
console.log(encrypt.encrypt('zdmission'))
```

## 20.弹层按钮出现错位
弹层按钮出现错位，为什么，因为点击输入框会弹出原生键盘，这时候会把页面网上推，导致页面实际可视距离减少，当输入框失去焦点之后，提示弹层出现，但是绘制按钮的高度就存在高度差，导致按钮错位，弹出弹层时把执行window.scrollTo(0,0)滚动到顶部，这样可以解决问题

## 21.页面卡顿，在某些需要的页面添加-webkit-overflow-scrolling: touch
页面卡顿，在某些需要的页面添加-webkit-overflow-scrolling: touch，增强页面的滚动性，某些特定的页面需要设定，通过当前的路由来判别，路由中fullPath是指路由后边的全部字符，包括query，但是我们设定的数组中的路径值是不包括外加参数的，所以判断失效了
```js
document.querySelector('html').style.cssText = `-webkit-overflow-scrolling: touch;height:100%;overflow-x: hidden;`
```

## 22.ios手机中，页面使用了css3 transition横向滚动动画，会出现闪动的情况，导致用户体验不好
布局方式：
```html
<div class="gundong-wrap">
   <div class="gundong" :class="{'donghua':isAnimation}"  :style="{'animation-duration':sumDuration+'s','-webkit-animation-duration':sumDuration+'s'}">
      <p ref="noticeP" class="msg pd-r-310 font-ws">{{$root.homeInfo.notice&&$root.homeInfo.notice.noticeContent}}</p>
      <p v-show="isAnimation" class="msg special font-ws">{{$root.homeInfo.notice&&$root.homeInfo.notice.noticeContent}}</p>
    </div>
</div>
```

css样式
```scss
.gundong-wrap {
    flex: 1;
    overflow: hidden;
    position: relative;
    .gundong {
        display: flex;
        justify-content: center;
        align-items: center;
        width:max-content;
        position: relative;
        .empty {
            height: 48px;
            width: 300px;
        }
        .msg {
            font-size: 28px;
            white-space: nowrap;
            line-height: 48px;
            color: #666;
            backface-visibility: hidden;
        }
        .pd-r-310 {
            padding-right: 308px;
        }
        .special {
            position: absolute;
            top: 0;
            left: 100%;
            padding-right: 300px;
        }
    }
}

.donghua {
    // animation-fill-mode: forwards;
    // animation: 8s move infinite linear;
    animation-name: move;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    transform-style: preserve-3d;
    perspective: 1000;
}
```
gundong是一个滚动层，采用flex横向布局，需要给gundong加上-webkit-transform-style: preserve-3d;perspective: 1000;属性以及给p元素加上-webkit-backface-visibility: hidden;即子元素背面隐藏

## 23.页面缓存，如果使用了keep-alive缓存了组件，如何更新页面数据
页面缓存，如果使用了keep-alive缓存了组件，那么该组件就会存在于内存中，下次加载的时候不会重新加载渲染组件而是直接从内存中取出该组件，但是我们的数据和用户状态是可能改变的，这时候需要显示到该组件上，但是该组件中的生命周期函数大多数是不会运行的，vue针对这种情况补充了两个生命周期钩子函数<font color=red>activated（组件被激活时调用）和deactivated（组件被停用是调用）</font>，所以我们把请求接口数据，更新view的操作放在该生命周期钩子函数中，当页面被缓存时也能更新页面数据

## 24.手机号，银行卡号，姓名脱敏等处理
```js
var str = '11012345678'
// 1.手机号按110 1234 5678这样的形式处理
var reg = str.match(/^(\d{3})(\d{4})(\d{4})$/)
`${reg[1]} ${reg[2]} ${reg[3]}`    // 110 1234 5678
// 2.手机号中间4位加*
`${reg[1]} **** ${reg[3]}`  // 110 **** 5678
`${str.slice(0,3)} **** ${str.slice(-4)}`  // 110 **** 5678

// 3.银行卡号每隔4位加空格
'6214564585796854'.replace(/(.{4})/g, "$1 ").replace(/\s$/,'') // '6214 5645 8579 6854'

// 4.银行卡号前四后四显示，中间用*代替
`${'6214564585796854'.slice(0, 4)} **** **** ${'6214564585796854'.slice(-4)}` // 6214********6854

// 5.姓名处理
/**
 * [sensitiveName 姓名脱敏]
 * @return void
 */
export function sensitiveName(name) {
    if(!name) {
        return ""
    }

    if(name.length > 2) {
        let arr = new Array(name.length - 1)
        return name[0] + arr.join("*") + name[name.length - 1]
    } else {
        return name[0] + "*"
    }
}
/**
 * [sensitiveHandel 脱敏处理]
 * @param  {[String]} val      [需要处理的值]
 * @param  {[Number]} startNum [前几位不脱敏的值]
 * @param  {[Number]} endNum   [后几位不脱敏的值]
 * @return {[String]}          [处理以后的值]
 */
export function sensitiveHandel(val, startNum, endNum) {
    if(!val) {
        return ""
    }

    return val.replace(new RegExp(`(\\d{${startNum}})(\\d+)(\\d{${endNum}}|\\d{${endNum - 1}}X)`), ($0, $1, $2, $3) => {
        for (let i = 0, len = $2.length; i < len; i++) {
            $1 += "*";
        }
        return $1 + $3;
    })
}

```