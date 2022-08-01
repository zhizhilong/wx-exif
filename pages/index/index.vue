<template>
	<view class="content"><button class="title" @click="click()">选择图片</button></view>
</template>

<script>
	var myexif = require('./myexif.js');
// import { getImageData, getFloatLocationByExif } from '@/js_sdk/izExif/izExif.js';
export default {
	data() {
		return {
			title: 'Hello'
		};
	},
	onLoad() {},
	methods: {
		click() {
			const that = this;
			uni.chooseImage({
				sizeType: ['original'],
				success: e => {
					var array = wx.getFileSystemManager().readFileSync(e.tempFilePaths[0]);
					var r = myexif.handleBinaryFile(array);
					console.log(r);
					// console.log(e);
					if (r && r.data) {
						this.shootTime =
							r.data.DateTime.split(' ')[0]
								.split(':')
								.join('-') +
							' ' +
							r.data.DateTime.split(' ')[1];
					}
					//  这里结束
					// 下面是其他方法获取
					let src = e.tempFilePaths[0];
					uni.getFileSystemManager().readFile({
						filePath: src, //选择图片返回的相对路径
						encoding: 'base64', //编码格式
						success: res => {
							uni.base64ToArrayBuffer(res.data);
							//成功的回调
							console.log(uni.base64ToArrayBuffer(res.data));
							that.parseImage(uni.base64ToArrayBuffer(res.data));
						}
					});
					// getImageData(src).then(res=>{
					// 	console.log(res)
					// 	console.log(getFloatLocationByExif(res.exif))
					// }).catch(e=>{
					// 	console.log(e)
					// })
				}
			});
		},
		parseImage(data) {
			// debugger
			// data为ArrayBuffer类型的JPG文件的二进制数据
			let arr = new Uint8Array(data);
			let base = 0; // TIFF数据头开始地址
			let timeTagIndex = 0; // EXIF时间信息标记开始地址
			// this.saveFile(arr)

			for (let i = 0; i < arr.length; i++) {
				// 获取TIFF数据头地址
				if (arr[i] == 69 && arr[i + 1] == 120 && arr[i + 2] == 105 && arr[i + 3] == 102 && arr[i + 4] == 0 && arr[i + 5] == 0) {
					base = i + 6;
				}
				// 获取时间标签地址
				if (arr[i] == 0x90 && arr[i + 1] == 0x03) {
					timeTagIndex = i;
					break; // 因为这个if的条件比较容易重复，但是我们要的是第一个，所以这里就可以直接退出了
				}
				if (arr[i] == 16 && arr[i + 1] == 58) {
					console.log('------');
					console.log(i);
					console.log(arr[i],arr[i+1],arr[i+2],arr[i+3]);
				}
			}
			console.log(base);
			console.log(timeTagIndex);

			let bias_str = '0x'; // 偏移地址
			for (let i = 0; i <= 3; i++) {
				console.log(arr[timeTagIndex + 8 + i]);
				bias_str += arr[timeTagIndex + 8 + i].toString(16);
			}
			console.log(bias_str);
			let datetime_addr_index = base + parseInt(bias_str); // 实际地址

			let datetimestr = ''; // 日期字符串
			for (let i = datetime_addr_index; i <= datetime_addr_index + 19; i++) {
				datetimestr += String.fromCharCode(arr[i]);
			}

			console.log('日期时间字符串', datetimestr); // 接着，我们可以将日期字符串进行一定的字符串和日期时间处理，以满足需要
		}
	}
};
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.logo {
	height: 200rpx;
	width: 200rpx;
	margin-top: 200rpx;
	margin-left: auto;
	margin-right: auto;
	margin-bottom: 50rpx;
}

.text-area {
	display: flex;
	justify-content: center;
}

.title {
	font-size: 36rpx;
	color: #8f8f94;
}
</style>
