# kd-barrage
看点弹幕组件，提供弹幕相关的一些原子操作，调用方可根据产品需求操作弹幕。

依赖于类 jQuery 框架，如 Zepto.js。

# 使用方法(Usage)
1. 引入 JS
```HTML
<script src="kd-barrage.js"></script>
```

2. 调用示意
```Javascript
var barrage = new Barrage({
    container: '.comment-layer',
    layout: 'half',
});

barrage.pushData(barrage_json);

$('video').on('play', function(a) {
    barrage.start(true);
});

$('video').on('pause', function() {
    barrage.stop();
});
```

# 配置参数(Config)
```Javascript
new Barrage(option);
```
`option`包括：
* **`container`**(Stirng): 必填，弹幕容器选择器字符串，容器需绝对定位，组件会自动获取容器宽高作为幕布。
* **`maxLength`**(Number): 弹幕长度限制，英文为 1，中文为 2，溢出部分将用...代替，默认值`30`。
* **`isHtmlEncode`**(Boolean): 是否转义 HTML 实体字符，默认值`true`。
* **`layout`**(Stirng): 弹幕布局方式，`top`三分之一，`half`半屏,`full`全屏，默认值`half`。
* **`mode`**(String): 视频类型，`recorded`录播、`live`直播，两种类型的视频对弹幕的调用方案不一样，默认值`recorded`。
* **`showTime`**(Number): 每条弹幕滚动时间，单位`ms`，默认值`3500`。
* **`lineHeight`**(Number): 弹幕行高，单位`px`，默认值`18`。
* **`fontSize`**(Number): 字体大小，单位`px`，默认值`15`。
* **`gapWidth`**(Number): 每条弹幕最小水平空隙，默认值`15`
* **`discard`**(Boolean): 是否抛弃多余弹幕，默认值`ture`，一秒内的弹幕只排在一列之中，当设为`false`时将显示所有弹幕，可能存在弹幕消费不及时的情况。
* **`cleanSetSize`**(Number): 待清理弹幕分包大小，当指定条数的弹幕展示完毕时，执行一次DOM清理，默认值`10`。
* **`style`**(Object): 通用弹幕样式，可自定义弹幕样式，驼峰书写 CSS keyname，定义的样式会覆盖默认值的同名项。默认值：
```Javascript
{
    color: '#ffffff',
    fontFamily: '黑体',
    opacity: '0.75',
    textShadow: 'rgb(0, 0, 0) 1px 1px 2px',
    fontWeight: 'bold', 
}
```
* **`self_style`**(Object)：用户个人弹幕的样式，定义的样式会覆盖通用弹幕样式同名项。默认值`{ color: '#74bb4f' }`。

# 实例方法(Methods)

## .pushData(items[, isCover])
```Javascript
@param items {array|object} 弹幕元数据
@param isCover {boolean} 是否覆盖元数据，只有数据类型为`object`时此选项才生效，默认值为`true`，`false`时则只在指定秒数插入数据
```
添加弹幕元数据（时间，内容）到内存缓存池中，录播类视频使用，在视频到达指定时间时组件将会自动显示弹幕。

`items`格式约定：
```Javascript
// array
[
    {
        "sec": 0,
        "bl": [
            {
                "ms": 0,
                "tx": "测试弹幕1"
            },
        ]
    },
    {
        "sec": 6,
        "bl": [
            {
                "ms": 6100,
                "tx": "测试弹幕2",
                "isSelf": 1 // 是否用户自己发的评论
            },
            {
                "ms": 6200,
                "tx": "测试弹幕3"
            },
        ]
    },
]

// object
{
    "sec": 0,
    "bl": [
        {
            "ms": 0,
            "tx": "测试弹幕1"
        },
    ]
}
```

## .clearData()
清除内存缓存池中的弹幕元数据。

## .setTime(seconds)
```Javascript
@param seconds {number} 设置当前弹幕进度，单位`s`
```
弹幕组件没有直接监听视频进度改变事件，组件内部有计时器模拟视频进度，但需要调用方在一些合适的时机自行设置时间起点。

调用该方法后，弹幕播放状态不会改变。

## .start([isContinue])
```Javascript
@param isContinue {boolean} 可选值，表明是否属于继续播放，默认值为`false`
```
启动弹幕滚动，并暂停组件内部的视频进度计时器。

用户拖动进度条后可调用该方法，该方法会自动调用一次`showByTime()`方法，加载并显示当前时间点的弹幕。

如只是在视频同一位置的暂停播放操作，则`isContinue`可设为`true`，此时将只会继续暂停前的一切操作。

## .stop()
暂停弹幕

## .showByTime(seconds)
```Javascript
@param seconds {number} 指定时间的弹幕，单位`s`
```
立即将缓存池中指定时间的弹幕显示在幕布上，此方法一般不需调用方主动调用。

## .clearBarrage()
清空所有弹幕，此操作只会清空 DOM 结点而不会删除`data`树上的数据。调用`clearBarrage()`方法后，弹幕不会自动暂停，如需暂停则需自行调用`stop()`方法。此方法一般在用户拖动进度条时调用。

## .reset(config)
```Javascript
@param config {object} 可选，需要修改的布局属性
```
重设布局属性并清空当前弹幕。该方法会在内部调用`clearBarrage()`方法清空弹幕，并修改布局属性：`lineHeight`、`layout`，如不传入参数，则相当于调用`clearBarrage()`方法。
```Javascript
barrage.reset({
    lineHeight: 20,
    layout: 'full'
})
```

## .switch()
用于切换视频，立即清空弹幕元数据，清空弹幕 DOM 节点，并将弹幕时间置 0。适用于单例 webview 网页。该方法不会调用`stop()`方法。

## .setProps(props)
```Javascript
@param props {object} 需要修改的属性
```
可立即修改一些与布局无关的属性，不会重新初始化弹幕。目前支持修改的属性：`fontSize`、`gapWidth`、`maxLength`。修改`fontSize`时也会一同修改当前已经生成的弹幕 DOM 节点的字体大小。


## .setShowTime(ms)
```Javascript
@param ms {number} 弹幕滚动速度，单位`ms`
```
动态修改整体弹幕的滚动速度，在窗口尺寸发生变化时可调用。

## .removeNotEnter()
立即清除已经加入 DOM 队列但尚未进入屏幕的弹幕，此方法的使用场景一般是开发者希望当用户拖动进度条后，保留当前已经进入屏幕的弹幕而清除尚未进入的弹幕。

## .showByTime(seconds)
```Javascript
@param seconds {number} 进度秒数，单位`s`
```
立即将指定秒数的弹幕加入 DOM 中，此方法不需要开发者主动调用，但在某些特殊场景可以使用。

## .add(text[, style, isNewAdd])
```Javascript
@param text {string|object} 弹幕内容，可以是字符串，也可以是弹幕对象
@param style {object} 弹幕样式，可选，默认使用全局配置项
@param isNewAdd {boolean} 是否用户新增弹幕，可选，当传入 true 时弹幕将随机排列加入队列中并高亮显示，默认值为 false。text.isSelf == 1 时相当于 isNewAdd 的值为1。
```
立即将新弹幕加入到 DOM 中，此方法可用于添加需要即时显示的弹幕，例如用户发表弹幕时。

`text`弹幕对象格式：
```Javascript
{
    "ms": 0, // 可选项
    "tx": "ganyiwei的测试弹幕_1127",
    "isSelf": 1 // 可选项，表示是否是用户自己发表的弹幕
}
```
调用方法
```Javascript
barrage.add('新的弹幕', true);
```

## .removeDataByTimeRange(minSeconds, maxSeconds)
```Javascript
@param minSeconds {number} 开始区间秒数，单位`s`
@param manSeconds {number} 结束区间秒数，单位`s`
```
清除指定秒数区间中`data`树上的数据。

## .hasData(seconds)
```Javascript
@param seconds {number} 进度秒数，单位`s`
@return {boolean}
```
判断`data`树中指定秒数是否有数据。