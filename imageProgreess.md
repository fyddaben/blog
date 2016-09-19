# 模仿Iphone 产品站 - 图片懒加载效果

> 发现很多`魅族`的产品站页面，图片加载很难受，然而`苹果`的产品站，却很舒服，于是小小研究了下。

### 魅族产品站效果

![meizu](https://cloud.githubusercontent.com/assets/3349863/18623319/4bf88bf6-7e6e-11e6-89e4-b972e975caee.gif)

### 苹果产品站效果
![apple](https://cloud.githubusercontent.com/assets/3349863/18623215/3c7d8b82-7e6d-11e6-9538-7c718de0af00.gif)

### 原理说明
- 首先，图片必须放在背景元素里面，便于过度效果的控制
- 图片默认都是隐藏的，即浏览器不需要加载
- 当页面滚动的某个位置，触发加载图片
- 图片加载完成，进行显示


### 代码讲解

#### 页面控制元素- 用 class 操作

- `progress-mainObj` 用于选择，需要使用图片延迟加载的DOM元素
- `progress-image` 用于隐藏背景图片，
- `progress-image-animated` 用于图片显示过度效果的，初始状态
- `progress-image-ready` 用于图片显示过度效果的，最终态

#### 代码部分

```html
<div class="section section-foc white   progress-mainObj progress-image-animated  progress-image">
        <div class="abs-text">
            <h2 class="title niu-font">
               德国博世高性能电机<br>野兽般的动力输出 
            </h2>
            <p class="content">
                与德国博世联合研发的博世电机，依然是出色性能的表现。在110N·M的扭矩下，驱动整车快速启动成为一件轻而易举的事情。中段加速输出依然出色，配合小牛自主开发的 FOC 矢量控制器，平滑中持续优雅提速。
            </p>
        </div>
    </div>
```
```css
.progress-image-animated{
     opacity: 0;
     -webkit-transition: opacity 0.5s ease-out;
     transition: opacity 0.5s ease-out;
 }
 .progress-image {
     background-image: none !important;
 }
 .progress-image-animated.progress-image-ready{
     opacity: 1;
 }

```
```javascript
  var tarList =  [];
  /*
     * 滚到某个位置触发事件
     * 初始化调用scrollTarList方法
     * 然后向tarList数组中，添加obj
     * obj.tar 表示目标高度
     * obj.gtcallback 表示到达目标高度后的回调
     */
    var scrollCallback = function(_this) {
        var top = _this.scrollTop(); 
        if (tarList.length > 0) {
            var list = tarList; 
            for(var i in list) {
                var curli = list[i];
                var tar = curli.tar; 
                if (top >= tar) {
                    if (curli.gtcallback) {
                        if (curli.option) {
                            curli.option.top = top;
                            curli.gtcallback(curli.option);
                        } else {
                            curli.gtcallback(top, tar);
                        }
                    }
                } else {
                    if (curli.ltcallback) {
                        if (curli.option) {
                            curli.option.top = top;
                            curli.ltcallback(curli.option);
                        } else {
                            curli.ltcallback(top, tar);
                        }
                    }
                }
            }
        } 
    }
    $(window).on('scroll.page', function() {
        scrollCallback($(this));
    });         
    window.onload = function() {
        scrollCallback($(window));
    }
    $('.progress-mainObj').each(function() {
        var curObj = $(this); 
        tarList.push({
            tar: curObj.offset().top - $(window).height() * 2, 
            gtinit: true,
            option: {
                gtFlag: false,    
                curObj: curObj
            },
            gtcallback: function(option) {
                if (!option.gtFlag) {
                    option.curObj.removeClass('progress-image');
                    var imageBack = option.curObj.css('background-image'); 
                    var url = imageBack.replace(/url\(\"(.*?)\"\)/g, function(word, p1) {return p1});
                    var img = new Image(); //创建一个Image对象，实现图片的预下载
                    img.src = url;
                    var imgCallback = function() {
                        option.curObj.addClass('progress-image-ready');
                    }
                    if(img.complete) { // 如果图片已经存在于浏览器缓存，直接调用回调函数
                        imgCallback();
                        return; // 直接返回，不用再处理onload事件
                    }
                    img.onload = function () { //图片下载完毕时异步调用callback函数。
                        imgCallback();
                    }; 
                    option.gtFlag = true;
                } 
            }
        });
    });


```

### PS: 使用感想

- 这种加载方式，还是比较适合纯色背景的图片
- 因为这样，图片加载未完成的时候，有个底色补充
- 颜色过度也比较自然
- 也可以使用于，占位图，之类的页面
