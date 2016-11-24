---
title: JavaScript广告轮播插件
date: 2016-01-08 14:43:41
tags:
	- JavaScript
---

## 使用示例

``` HTML
<html>
	<head>
		<title></title>
	</head>
	<body>
		<div>
			<div id="adprev">Prev</div>
			<div id="adnexa">Next</div>
			<ul id="pageControl">
				<li></li>
				<li></li>
				<li></li>
			</ul>
			<ul id="adCarousel">
				<li><a href="#" target="_blank"><img src=""></a></li>
				<li><a href="#" target="_blank"><img src=""></a></li>
				<li><a href="#" target="_blank"><img src=""></a></li>
			</ul>
		</div>
	</body>
	<script type="text/javascript">
	new AdCarousel({
        pageUlId : 'pageControl',
        pageSelectClass : 'thistitle',
        prevButtonId : 'adprev',
        nextButtonId : 'adnext',
        imageCarouselId : 'adCarousel',
        speed : 20,
        scrollDuration : 2000,
        autoScroll : true,
        direction : 'horizontal',
    });
	</script>
</html>
```

## 插件源码

``` JavaScript
/*!
 * 广告幻灯片轮播插件
 * @author xiaopin
 * @version 1.0
 * @example
 *  new AdCarousel({
 *      pageUlId : 'pageControl', // 页码ul标签的id
 *      pageSelectClass : 'page-selected', // 页码被选中的样式类
 *      pageAttribute : 'data-page',
 *      prevButtonId : 'prev', // 上一页按钮的id
 *      nextButtonId : 'next', // 下一页按钮的id
 *      imageCarouselId : 'adCarousel', // 广告图片的ul标签id
 *      speed : 20, // 滚动速度,数值越大,速度越慢
 *      scrollDuration : 2000, // 滚动间隔时间
 *      autoScroll : true,
 *      direction : 'horizontal' // 滚动方向(vertical|horizontal)
 *      perMoveDistance : 50 // 每次移动的距离
 *  });
 */
; (function(window) {
    window.AdCarousel = function(options) {
        var timer = null;
        var currentImageIndex = 0;
        var adImageCount = 0;
        var currentPageLi = null;
        var allPages = null;
        var config = {
            pageUlId: "pageControl",
            pageSelectClass: "page-selected",
            pageAttribute: "data-page",
            prevButtonId: "prev",
            nextButtonId: "next",
            imageCarouselId: "adCarousel",
            speed: 20,
            scrollDuration: 2000,
            autoScroll: true,
            direction: "horizontal",
            perMoveDistance: 50,
        };
        var init = function(options) {
            var self = this;
            extend(config, options);
            if (config.autoScroll) {
                this.startScroll()
            }
            adImageCount = children(document.getElementById(config.imageCarouselId), "LI").length;
            var prevButton = document.getElementById(config.prevButtonId);
            bindEvent(prevButton, "click",
            function() {
                self.previousImage()
            });
            var nextButton = document.getElementById(config.nextButtonId);
            bindEvent(nextButton, "click",
            function() {
                self.nextImage(true)
            });
            var pageUl = document.getElementById(config.pageUlId);
            allPages = children(pageUl, "LI");
            for (var i in allPages) {
                var li = allPages[i];
                if (i == 0) {
                    currentPageLi = li;
                    li.className = config.pageSelectClass
                }
                li.setAttribute(config.pageAttribute, i);
                bindEvent(li, "click",
                function() {
                    pageClicked.call(self, this)
                })
            }
        };
        this.startScroll = function() {
            var self = this;
            var ad = document.getElementById(config.imageCarouselId);
            if (!ad || children(ad, "LI").length == 0) {
                return
            }
            if (timer != null) {
                clearInterval(timer)
            }
            timer = setInterval(function() {
                self.nextImage()
            },
            config.scrollDuration)
        };
        this.previousImage = function() {
            var index = currentImageIndex - 1;
            if (index < 0) {
                return
            }
            currentPageHighlight(index);
            imageScrollAnimate.call(this, index)
        };
        this.nextImage = function(stop) {
            var index = currentImageIndex + 1;
            if (index >= adImageCount) {
                if (stop === true) {
                    return
                } else {
                    index = 0
                }
            }
            currentPageHighlight(index);
            imageScrollAnimate.call(this, index)
        };
        var pageClicked = function(li) {
            var index = li.getAttribute(config.pageAttribute);
            currentPageHighlight(index);
            imageScrollAnimate.call(this, index)
        };
        var currentPageHighlight = function(index) {
            if (index < 0 || index >= allPages.length) {
                return
            }
            var li = allPages[index];
            if (currentPageLi != null) {
                currentPageLi.className = ""
            }
            li.className = config.pageSelectClass;
            currentPageLi = li
        };
        var imageScrollAnimate = function(targetIndex) {
            var self = this;
            var moveDistance = config.perMoveDistance * (targetIndex > currentImageIndex ? 1 : -1);
            var ad = document.getElementById(config.imageCarouselId);
            targetIndex = parseInt(targetIndex);
            if (!ad || isNaN(targetIndex) || targetIndex < 0) {
                return
            }
            if (timer != null) {
                clearInterval(timer)
            }
            currentImageIndex = targetIndex;
            var attr = getScrollAttribute();
            var targetValue = targetIndex * getScrollDistance();
            timer = setInterval(function() {
                var val = parseInt(elementStyle(ad, attr));
                val = (isNaN(val) ? 0 : Math.abs(val)) + moveDistance;
                if (moveDistance > 0 && val > targetValue) {
                    val = targetValue
                } else {
                    if (moveDistance < 0 && val < targetValue) {
                        val = targetValue
                    }
                }
                if (val == targetValue) {
                    clearInterval(timer);
                    val = targetValue;
                    if (config.autoScroll) {
                        timer = setInterval(function() {
                            self.nextImage()
                        },
                        config.scrollDuration)
                    }
                }
                var style = attr + ":-" + val + "px";
                ad.setAttribute("style", style)
            },
            config.speed)
        };
        var isEmptyObject = function(obj) {
            if (typeof obj !== "object") {
                return true
            }
            for (var prop in obj) {
                return false
            }
            return true
        };
        var elementStyle = function(obj, style) {
            if (typeof(obj) !== "object" || obj == null) {
                return null
            } else {
                if (obj.currentStyle) {
                    return obj.currentStyle[style]
                } else {
                    return window.getComputedStyle(obj, null)[style]
                }
            }
        };
        var extend = function(obj, ext) {
            if (typeof obj !== "object" || isEmptyObject(ext)) {
                return
            }
            for (var prop in ext) {
                obj[prop] = ext[prop]
            }
        };
        var bindEvent = function(obj, event, callback) {
            if (typeof obj !== "object" || obj == null || typeof event != "string" || typeof callback != "function") {
                return
            }
            if (obj.addEventListener) {
                event = event.replace(/^on/i, "").toLowerCase();
                obj.addEventListener(event, callback)
            } else {
                if (obj.attachEvent) {
                    if (!/^on/i.test(event)) {
                        event = ("on" + event).toLowerCase()
                    }
                    obj.attachEvent(event, callback)
                }
            }
        };
        var getScrollAttribute = function() {
            if (config.direction == "horizontal") {
                return "left"
            } else {
                if (config.direction == "vertical") {
                    return "top"
                }
            }
            return ""
        };
        var getScrollDistance = function() {
            if (config.distance === undefined) {
                var ad = document.getElementById(config.imageCarouselId);
                var images = children(ad, "LI");
                if (images.length) {
                    var img = images[0];
                    config.distance = img.offsetWidth
                }
            }
            return config.distance
        };
        var children = function(obj, childTagName) {
            var elements = [];
            childTagName = childTagName ? childTagName.toUpperCase() : "";
            if (typeof obj === "object" && obj != null) {
                var length = obj.childNodes.length;
                for (var i = 0; i < length; i++) {
                    var child = obj.childNodes[i];
                    if (child.nodeType == 1) {
                        if (childTagName != "" && child.tagName != childTagName) {
                            continue
                        }
                        elements.push(child)
                    }
                }
            }
            return elements
        };
        init.call(this, options)
    }
} (window));
```