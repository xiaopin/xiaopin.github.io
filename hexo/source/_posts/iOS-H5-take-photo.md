---
title: 修复 iOS 网页在拍照预览时图片被旋转的问题
date: 2020-01-19 14:50:04
tags:
	- iOS
	- JavaScript
---

前几天用 uni-app 写了个上传图片的小案例(编译到 H5 端)，当在 iPhone 上拍照上传时，发现在页面上预览的图片是发生了旋转的。

## 图片的 Exif 信息

Exif 全称 Exchangeable image file format，中文叫`可交换图像文件格式`，Exif 中存储了图片的方向信息。

| EXIF Orientation Value | Row #0 is: | Column #0 is: |
| :-: | :- | :- |	
| 1 | Top | Left side |
| 2 | Top | Right side |
| 3 | Bottom | Right side |
| 4 | Bottom | Left side |
| 5 | Left side | Top |
| 6 | Right side | Top |
| 7 | Right side | Bottom |
| 8 | Left side	| Bottom |

看不懂没关系，网上找了张示例图(如有侵权，请联系删之)，请对号入座：

![](/images/2020/image-orientation.png)


## 获取图片的 Orientation 信息

网上都在推荐 [Exif.js](https://github.com/exif-js/exif-js) 这个库，有兴趣的可以自行研究，我没用过，就不多说了。

```JavaScript
/**
 * 根据File对象获取图片的方向信息
 * @param {File} file
 * @param {Function} callback
 */
getOrientation: function(file, callback) {
    var reader = new FileReader();
    reader.onload = function(e) {
        var view = new DataView(e.target.result);
        if (view.getUint16(0, false) != 0xFFD8)
        {
            return callback(-2);
        }
        var length = view.byteLength, offset = 2;
        while (offset < length) 
        {
            if (view.getUint16(offset+2, false) <= 8) return callback(-1);
            var marker = view.getUint16(offset, false);
            offset += 2;
            if (marker == 0xFFE1) 
            {
                if (view.getUint32(offset += 2, false) != 0x45786966) 
                {
                    return callback(-1);
                }

                var little = view.getUint16(offset += 6, false) == 0x4949;
                offset += view.getUint32(offset + 4, little);
                var tags = view.getUint16(offset, little);
                offset += 2;
                for (var i = 0; i < tags; i++)
                {
                    if (view.getUint16(offset + (i * 12), little) == 0x0112)
                    {
                        return callback(view.getUint16(offset + (i * 12) + 8, little));
                    }
                }
            }
            else if ((marker & 0xFF00) != 0xFF00)
            {
                break;
            }
            else
            { 
                offset += view.getUint16(offset, false);
            }
        }
        return callback(-1);
    };
    reader.readAsArrayBuffer(file);
}
```

如果是采用 `<input type="file" capture="camera" accept="image/png, image/jpeg">` 来拍照，返回的就是一个 [`FileList`](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList) 对象，里面储存的每一个 [File](https://developer.mozilla.org/zh-CN/docs/Web/API/File) 对象代表用户选择的一个图片，可以直接调用上面 getOrientation 方法获取图片的方向信息

但是 uni-app 的 `uni.chooseImage` 接口在 H5 端返回的是一个 `blob:http://xxxxxx/xxxxx` 格式的路径信息，并不是一个 File 对象，所以我们需要做一些额外处理：

1、将路径转成 [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象
	```JavaScript
	const xhr  = new XMLHttpRequest();
	xhr.open("GET", url, true);
	xhr.responseType = "blob";
	xhr.onload = function() {
		const blobObject = this.response;
		// ...
	};
	xhr.send();
	```
2、将 [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 转成 [File](https://developer.mozilla.org/zh-CN/docs/Web/API/File)
	```JavaScript
	const file = new File([blobObject], 'photo.png', {type: blobObject.type});
	```
3、调用 getOrientation，然后做相应的旋转处理


## 修复预览旋转

<!-- 前几天用 uni-app 框架写了一个拍照上传图片的页面，在 iPhone 上拍照时，图片出现了旋转 270° 的问题。 -->
由于我的项目是基于 uni-app 进行构建的，所以这里主要是针对 uni-app 下编译的 H5 页面进行说明。

完整示例代码：
```html
<template>
	<view class="container">
		<view class="content">
			<view class="photo" @click="onTokePhotoTapped">
				<image src="/static/placeholder.png"></image>
				<image :src="faceImage" mode="aspectFill" :style="fixOrientationStyle"></image>
			</view>
			<view class="field-item">
				<view class="field-item-title">工号</view>
				<input class="field-item-input" placeholder="请输入工号" maxlength="30" v-model="jobNumber" />
			</view>
			<view class="submit-button" @click="onSubmitTapped">提交</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				jobNumber: '',
				faceImage: '',
				fixOrientationStyle: '', // 修正图片方向
			}
		},
		
		components: { PlaceholderImage },
		
		onLoad() {

		},
		
		methods: {
		
			onTokePhotoTapped: function() {
				uni.chooseImage({
					count: 1,
					success: (res) => {
						// this.faceImage = res.tempFilePaths[0];
						/**
						 1、URL => Blob => File
						 2、获取方向
						 */
						const url = res.tempFilePaths[0];
						const self = this;
						let xhr  = new XMLHttpRequest();
						xhr.open("GET", url, true);
						xhr.responseType = "blob";
						xhr.onload = function() {
							self.faceImage = url;
							self.fixOrientationStyle = '';
							if (this.status == 200) {
								const blob = this.response;
								const file = new File([blob], 'photo.png', {type: blob.type});
								self.getOrientation(file, (orientation) => {
									switch(orientation){  
										case 6://需要顺时针（向左）90度旋转
											self.fixOrientationStyle = 'transform: rotate(90deg);';
											break;
										case 8://需要逆时针（向右）90度旋转
											self.fixOrientationStyle = 'transform: rotate(-90deg);';
											break;
										case 3://需要180度旋转
											self.fixOrientationStyle = 'transform: rotate(180deg);';
											break;  
									}
								});
							}
						};
						xhr.send();
					}
				});
			},
			
			onSubmitTapped: function() {
				const number = this.jobNumber.replace(/(^\s*)|(\s*$)/g, '');
				if (!number.length) {
					uni.showToast({ icon: 'none', title: '请输入工号' });
					return;
				}
				if (!this.faceImage.length) {
					uni.showToast({ icon: 'none', title: '请选择图片' });
					return;
				}
				uni.showLoading({ mask: true, title: '正在保存...' });
				// TODO: 这里就是图片上传的代码, 直接调用相应接口即可
				// 这里上传的图片还是旋转的，需要后台将图片方向修正过来
			},
			
			/**
			 * 获取图片旋转方向

			 * @param {Object} file			File对象
			 * @param {Object} callback		回调
			 */
			getOrientation: function(file, callback) {
			    var reader = new FileReader();
			    reader.onload = function(e) {
			
			        var view = new DataView(e.target.result);
			        if (view.getUint16(0, false) != 0xFFD8)
			        {
			            return callback(-2);
			        }
			        var length = view.byteLength, offset = 2;
			        while (offset < length) 
			        {
			            if (view.getUint16(offset+2, false) <= 8) return callback(-1);
			            var marker = view.getUint16(offset, false);
			            offset += 2;
			            if (marker == 0xFFE1) 
			            {
			                if (view.getUint32(offset += 2, false) != 0x45786966) 
			                {
			                    return callback(-1);
			                }
			
			                var little = view.getUint16(offset += 6, false) == 0x4949;
			                offset += view.getUint32(offset + 4, little);
			                var tags = view.getUint16(offset, little);
			                offset += 2;
			                for (var i = 0; i < tags; i++)
			                {
			                    if (view.getUint16(offset + (i * 12), little) == 0x0112)
			                    {
			                        return callback(view.getUint16(offset + (i * 12) + 8, little));
			                    }
			                }
			            }
			            else if ((marker & 0xFF00) != 0xFF00)
			            {
			                break;
			            }
			            else
			            { 
			                offset += view.getUint16(offset, false);
			            }
			        }
			        return callback(-1);
			    };
			    reader.readAsArrayBuffer(file);
			}
		}
	}
</script>

<style>
	page {
		background-color: #EFEFF4;
		font-size: 30rpx;
	}
	.container {
		width: 100%;
		min-height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.content {
		width: 600rpx;
		margin-top: 40rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	.photo {
		width: 400rpx;
		height: 400rpx;
		border-radius: 20rpx;
		background-color: white;
		overflow: hidden;
		position: relative;
	}
	.photo image {
		width: 100%;
		height: 100%;
	}
	.photo image:last-child {
		position: absolute;
		top: 0;
		left: 0;
	}
	.field-item {
		width: calc(100% - 40rpx);
		height: 88rpx;
		padding: 0 20rpx;
		margin: 30rpx 0 50rpx;
		border-radius: 10rpx;
		overflow: hidden;
		background-color: white;
		display: flex;
		align-items: center;
	}
	.field-item-title {
		width: 100rpx;
	}
	.field-item-input {
		width: calc(100% - 100rpx);
		height: 100%;
	}
	.submit-button {
		width: 100%;
		height: 80rpx;
		border-radius: 20rpx;
		background-color: #007AFF;
		color: white;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
```


## 参考

- [Exif](https://baike.baidu.com/item/Exif/422825?fr=aladdin)
- [Accessing JPEG EXIF rotation data in JavaScript on the client side](https://stackoverflow.com/questions/7584794/accessing-jpeg-exif-rotation-data-in-javascript-on-the-client-side)
