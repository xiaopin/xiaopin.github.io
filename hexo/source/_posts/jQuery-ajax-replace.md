---
title: 重载jQuery的ajax方法解决JSON解析的问题
date: 2019-03-25 09:23:12
tags:
	- JavaScript
---

> 当服务器返回的JSON字符串中包含`回车符`等特殊字符时，ajax的解析会报错，导致程序执行异常。

为了解决这个问题，在不侵入任何代码的情况下，只能替换 jQuery ajax 的方法实现了。

将下面的代码复制到公共js的顶部，确保在发送网络请求之前就已经执行了替换操作，这样我们就能自定义解析服务器返回的JSON数据了。

```JavaScript
$(() => {
	// Make sure the replacement operation is only performed once.
	if (window._isReplacedAjaxImp) return;
	window._isReplacedAjaxImp = true;

	const replaceAjaxImp = ($) => {
		// Convert string to JSON object (if success).
		const convertToJSON = (jsonText) => {
			let result = null;
			try {
				result = JSON.parse(jsonText);
			} catch (e) {
				try {
					result = JSON.parse(jsonText.replace(/(\r\n|\r|\n)/g, ''));
				} catch (e) {}
			}
			return result || jsonText;
		};

		// Reference to the original jQuery ajax function
		const originalAjax = $.ajax;

		// Override jQuery ajax function
		$.ajax = function() {
			// Note: `arguments` cannot be used in arrow functions
			let options = arguments[0];
			if (typeof options === 'object' && options != null) {
				const isJsonFormat = Reflect.has(options, 'dataType') && Reflect.get(options, 'dataType').toString().toLocaleLowerCase() == 'json';
				const originalSuccessFunc = Reflect.get(options, 'success');
				if (isJsonFormat && typeof originalSuccessFunc === 'function') {
					// Custom parsing JSON
					Reflect.set(options, 'dataType', 'text');
					Reflect.set(options, 'success', function(data, textStatus, xhr) {
						if (arguments.length) {
							const resp = arguments[0];
							arguments[0] = convertToJSON(resp);
						}
						originalSuccessFunc.apply(this, arguments);
					});
				}
			}
			// Call original ajax function
			originalAjax.apply($, arguments);
		};
	};

	replaceAjaxImp(jQuery);
});
```