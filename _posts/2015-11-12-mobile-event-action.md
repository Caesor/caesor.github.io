---
layout: blog
title: 移动端事件冲突处理小计
categories: font-end
tags: mobile events javascript
---
##事出有因
当我们在制作一些手机端动画插件时，总会去使用原生的`touchstart`、`touchmove`、`touchend`去监听用户手指移动的距离、方向、速度。与此同时，我们可能已经在即将引用插件的页面引入了zepto来处理一些常规的事件绑定。这时，`touchstart`、`touchmove`、`touchend`、`tap`和`scroll`事件全部登台亮相，会碰撞出什么样的绚丽火花？还是会引起一场难以想象的事件混乱灾难？

拿一个手头的例子来说：需要制作一个类似与iOS中列表页，使用插件实现左划删除的绑定效果（这个是一个现有插件，直接拿过来发现完全不好用，于是开始改造）。
【此处应有图片】
如上图所示，我们需要给列表的每一项绑定左划删除，又要使用zepto绑定列表项tap进入到详情页，同时列表页面还有一个上下滚动的触发。

说到这里，有些人就看不下去了，这么简单的效果有什么难实现的，况且页面滚动用得着你来添加事件绑定吗？用原生的不就好了？而我会说，为了兼容大部分的android & iOS 用户，我们还是得注意很多细节！

##案发现场
deleter v1.0(原始版)

    // 绑定点击
    $('body').on('tap', '.chatlist-item-delete', function(e){
        //页面跳转
    });
    // touchmove 绑定
    Deleter.prototype.move = function (e) {
        if (this._state >= 1) {
            var cur = pos(e);
            var pre = this._pre;
            var deltaX = pre.x - cur.x;
            var deltaY = pre.y - cur.y;
            if ( (deltaX > 10 || deltaX < -10) && deltaY < 10 && deltaY > -10 ) {
                this._pre = cur;
                this._state = deltaX > 0? 2: 3;
                // 移动列表item，逐渐显露出删除按钮
                this.translate(this._offset + deltaX);            
                e.preventDefault();
                e.stopPropagation();
                return false;
            }
        }
    };

原以为大功告成，结果发现一个问题：轻轻划一下，删除按钮还没完全暴露出来就已经发生页面跳转了。这尼玛还能愉快的玩耍么？！
不用说，显然是触发了 tap 事件。查看zepto实现tap的部分源码：

    if (deltaX < 30 && deltaY < 30) {
        // delay by one tick so we can cancel the 'tap' event if 'scroll' fires
        // ('tap' fires before 'scroll')
        tapTimeout = setTimeout(function() {
            // trigger universal 'tap' with the option to cancelTouch()
            // (cancelTouch cancels processing of single vs double taps for faster 'tap' response)
            var event = $.Event('tap')
            event.cancelTouch = cancelAll
            touch.el.trigger(event)            
            // trigger double tap immediately
            if (touch.isDoubleTap) {
                if (touch.el) touch.el.trigger('doubleTap')
                touch = {}
            }
            // trigger single tap after 250ms of inactivity
            else {
                touchTimeout = setTimeout(function() {
                    touchTimeout = null
                    if (touch.el) touch.el.trigger('singleTap')
                    touch = {}
                }, 250)
            }
        }, 0);
    }

原因一目了然，zepto中将点击的偏移值锁定在 30 内，自然我们定义的touchmove中会触发tap事件啦~修改之
deleter v1.1

    if ((deltaX > 30 || deltaX < -30) && deltaY < 10 && deltaY > -10) {
        ...
    }

大功告成了！拿起手中的iphone玩起来！看起来效果还不错，再来个android试一试,走你...走你...走你...可就是拨不动。在touchmove时打印出log，发现拨一次，touchmove**仅仅触发了一次**，没有连续触发。不用惊奇，这是一个经典的android bug，解决也很简单，在touchmove触发的时候调用`e.preventDefault();`就ok啦。

deleter v1.2

    Deleter.prototype.move = function (e) {
        e.preventDefault();
        ...
    }

这次应该大功告成了吧，试试！嗯嗯，不错，iOS可以，android也是可以的。但是悲剧就是会这么轻易的来临！列表不能滚动了，这是个什么鬼，不能滚动要列表有卵用啊，更别说做什么滚动加载了。按耐住心中的怒火细细想想，`preventDefault`不就是万恶的根源么？！怎么破？细想一下，滑动删除是“左右”划，列表滚动是“上下”划，有了！

deleter v1.3

    Deleter.prototype.move = function(e) {
        if (this._state >= 1) {
            var cur = pos(e);
            var pre = this._pre;
            var deltaX = cur.x - pre.x ;
            var deltaY = cur.y - pre.y;

            // 判断是不是左划 & 右划
            if(deltaX > 10 || deltaX < -10){
                // 如果是左右滑动才阻止默认行为
                e.preventDefault();
                // judge it as not a tap event
                if ((deltaX > 30 || deltaX < -30) && deltaY < 10 && deltaY > -10) {
                    // update pre position
                    this._pre = cur;
                    // update state
                    this._state = deltaX > 0 ? 2 : 3;
                    // translate element
                    this.translate(this._offset + deltaX);
                    e.stopImmediatePropagation();
                    return false;
                }
            }
        }
    };

到此为止：tap事件触发ok，左右滑动列表项ok，上下滚动列表ok，事件触发一切ready。

其实要这么干也只是为了让插件不依赖于zepto，所以才没有直接使用zepto中的 `swipeLeft` 和 `swipeRight` 。而zepto也并不是只是简单实现了 `tap` ，当然还有滑动、双击、长按等等动作。说到强大，Zepto的touch模块也只是实现了tap和swipe相关的动作，不支持复杂手势，如果需要支持复杂手势，建议使用[hammer.js](http://hammerjs.github.io/)，hammer提供了完善的一整套手势支持。

故事并没有到此结束，请接着往下看。

##如丝般滑动
事件绑定是ok啦，但是发现左右滑动列表项怎么都不顺滑。有卡顿，不可以接受！

deleter v1.0(原始版)

    Deleter.prototype.translate = function (x) {
        var move = -Math.round(x);
        if(move <= -this.options.right) this.$element.css('-webkit-transform', 'translate3d(' + move + 'px, 0, 0)');
        // make the latest offset value
        this._offset = x;
    };
    Deleter.prototype.autoTranslate = function () {
        var amplitude = this._amplitude;
        var target = this._target;
        var options = this.options;
        var left = options.left;
        var right = options.right;
        var timeConstant = options.timeConstant;
        var elapsed, delta;
        if (amplitude) {
            elapsed = getTime() - this._timestamp;
            delta = amplitude * Math.exp(-elapsed / timeConstant);
            var x = target - delta;
            // target stop range
            if (delta > 4 || delta < -4) {
                this.translate(target - delta);
                requestAnimationFrame(this.autoTranslate);
            }else{
                // translate to ending position
                x = Math.round(x) - 5;
                this._opened = x <= left && x > right;
                this.translate(this._opened? left: right);
            }
        }
    };

看完这段代码（其实都没用心读，只是看到了使用requestAnimationFrame），心想，何必呢？既然在translate中使用了 `translate3d`，何为还要使用 `requestAnimationFrame` 来实现这样一个简单的*短距离移动*动画效果，加之还在过程中打点记录了时间，使用了回调。直接上改进后的代码。

deleter v1.3

    Deleter.prototype.translate = function(x) {
        if ( x <= this.options.offsetRight && x >= this.options.offsetLeft ) {
            this.$element.css({
                'transition': '300ms cubic-bezier(0.1, 0.5, 0.5, 1)',
                '-webkit-transform':'translate3d(' + x + 'px, 0, 0)'
            });
        }
        // make the latest offset value
        this._offset = x;
    };
    Deleter.prototype.autoTranslate = function() {
        var offset = this._offset;
        // 判断touchend位置在中轴线左还是右，从而自动伸缩
        if( offset > -38){
            this.translate(0);
            this._open = false;
        }else if( offset <= -38){
            this.translate(-75);
            this._open = true;
        }else {
            return false;
        }
    };

代码简单可读，没有使用`requestAnimationFrame`而简单调用开了了GPU硬件加速了的`transform`，加之使用`transition`增强惯性和补间。

在移动端，我比较推崇利用GPU开启硬件加速，即便是我们有时候并不需要对元素应用3D变换效果，我们一样可以开启3D引擎。比如以使用：`transform:translateZ(0)`来开启硬件加速。原生的移动端应用程序总是可以很好的利用GPU，这是它比web应用表现更好的原因。

通过开启GPU硬件加速虽然可以提升动画渲染性能，但使用前一定要严谨测试，否则它反而会大量占用浏览网页用的系统资源，尤其是在移动端，肆无忌惮的开启GPU硬件加速会导致大量消耗设备电量，降低电池寿命。

##这里并不是结局
到此为止，一个简单的列表左划删除插件已经改造完毕。其实常用的移动端动画组建无外乎都是由这些小的东西组成的：

 1. touch事件的准确监听和响应（已有场景使用两点或多点触控）
 2. 记录touch过程中打点的时间、频率结合既定的“摩擦因子”实现加减速运动
 3. 在js中利用CSS3中`transfrom`的硬件加速优势动态改变这些css属性改变大小、位移、角度等

其中的第二点并没有在本文所讨论的案例中使用，但是却是对于用户感知体验最重要的一个环节。将在后续出文探讨。