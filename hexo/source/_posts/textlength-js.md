---
title: JavaScript实时统计输入框字符数
tags:
  - JavaScript
abbrlink: c2ec592b
date: 2016-01-12 16:34:32
---

## 该插件提供一下功能
* 实时统计文本字符数，提醒用户当前还可输入多少字符
* 限制用户输入的文本长度

``` JavaScript
/*!
 * 字符统计工具
 * @example
 * <textarea name="desc" id="desc"></textarea><p>还可以输入<em></em>字符</p>
 * new WordCountTool({id:'#desc', charset:'UTF-8', maxlength:2000});
 */
;(function(window, $){
    var WordCountTool = function(options) {
        var config = null;
 
        var init = function(options) {
            var self = this;
            config = $.extend({}, {id:'#wordcount', charset:'UTF-8', maxlength: 1000}, options);
            $(config.id).on('keyup', function(){
                startStatistics.call(self, this);
            });
            $(config.id).trigger('keyup');
        };
 
        var startStatistics = function(obj){
            var val = $(obj).val(),
                curLength = config.maxlength,
                strLength = strlen(val),
                isUtf8 = config.charset.toUpperCase() == 'UTF-8';
                if ($.trim(val) == '') val = '';
                for (var i = 0; i < val.length; i++) {
                    var charCode = val.charCodeAt(i);
                    if (charCode < 0 || charCode > 255) {
                        curLength -= (isUtf8 ? 2 : 1);
                    }
                }
 
                if (curLength >= strLength) {
                    $(obj).next('p').children('em').html(curLength - strLength);
                } else {
                    $(obj).val(strcut(val, config.maxlength));
                }
        };
 
        var strlen = function(str) {
            if($.trim(str) == '') return 0;
            return ($.browser.msie && str.indexOf('\n') != -1) ? str.replace(/\r?\n/g, '_').length : str.length;
        };
 
        var strcut = function(str, maxlen) {
            var curLength = 0,
                result = '',
                charByteLength = config.charset.toUpperCase() == 'UTF-8' ? 3 : 2;
            for (var i = 0; i < str.length; i++) {
                var charCode = str.charCodeAt(i);
                curLength += (charCode < 0 || charCode > 255) ? charByteLength : 1;
                if (curLength > maxlen) {
                    break;
                }
                result += str.substr(i, 1);
            }
            return result;
        };
 
        init.call(this, options);
    };
 
    window.WordCountTool = WordCountTool;
}(window, jQuery));
```