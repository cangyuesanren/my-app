<template>
	<view class="content">
		<view class="msg-box" ref="msgBox">
			<template v-for="(item, index) in msgList">
				<view v-if="item.type == 0" class="msg-item my-msg">
					<view class="m-m-top">
						<image src="@/static/image/user.png"></image>
						You
					</view>
					<view class="m-m-content">
						{{item.message}}
					</view>
				</view>
				<view v-if="item.type == 1" class="msg-item ai-msg">
					<view class="m-m-top">
						<image src="@/static/image/fill.png"></image>
						面试展示
					</view>
					<view class="m-m-content">
						<pre>{{item.message}}</pre>

					</view>
				</view>

			</template>

		</view>

		<view class="buttom-box">
			<div class="buttom-left">
				<image src="@/static/image/more.png" />

			</div>
			<input v-model="myMsg.message"/>
			<button @click="toSend">
				Send
				<image src="@/static/image/send.png" />
			</button>
		</view>
	</view>
</template>

<script setup lang="ts">
	import { ref, nextTick } from "vue"

	interface msgType {
		type?: string,
		message: string,
	}

	let sendState: boolean = false;
	const myMsg = ref<msgType>({
		message: ""
	})
	const msgList = ref<Array<msgType>>([]);
	const msgBox = ref(null)

	function toSend() {
		if(!myMsg.value.message){
			return;
		}
		if(sendState){
			uni.showToast({
				title: 'ai响应中...',
				duration: 2000,
				icon: "loading"
			});
			return;
		}else{
			sendState = true;
		}

		let msgData = JSON.parse(JSON.stringify(myMsg.value))

		msgList.value.push({
			type: 0,
			message: myMsg.value.message
		});
		msgList.value.push({
			type: 1,
			message: "Typing..."
		});
		nextTick(() => {
			msgBox.value.$el.scrollTop = msgBox.value.$el.scrollHeight + 500
		})
		myMsg.value.message = "";
		uni.request({		//发送请求
			url: "/api/send",
			data:msgData,
			method: "post",
			header: {
				"Access-Control-Allow-Origin":"*",
			},
		}).then(res => {
			sendState = false;
			let msgItem: msgType = msgList.value[msgList.value.length - 1];
			if(res.statusCode == 200){
				msgShow(msgItem, res.data.content);
			}else if(res.statusCode == 413){
				msgShow(msgItem, '查询的文字不能超过6个字，请重新输入！');
			}else{
				msgShow(msgItem, res.data);
			}
			nextTick(() => {
				msgBox.value.$el.scrollTop = msgBox.value.$el.scrollHeight + 500
			})
		})
	}

	function msgShow(msgObj: msgType, text: string) {
		msgObj.message = "";
		if(text.length){
			let textArr = text.split("");
			let i = 0
			let msgTime = setInterval(() => {
				if(i < textArr.length){
					msgObj.message += textArr[i];
					i++
					nextTick(() => {
						msgBox.value.$el.scrollTop = msgBox.value.$el.scrollHeight + 500
					})
				}else {
					clearInterval(msgTime);
				}
			}, 15)
		}
	}
	
</script>

<style lang="scss">
	.content{
		font-size: 22rpx;

		.msg-box{
			height: calc(100vh - 200rpx);
			padding-bottom: 20rpx;
			overflow-y: auto;
			.msg-item{
				border-radius: 10rpx;
				overflow: hidden;
				width: 550rpx;
				margin-top: 15rpx;
				box-shadow: 2px 2px 5px 3px rgba(0, 0, 0, 0.1);
				.m-m-top{
					display: flex;
					padding: 15rpx;
					color: #fff;
					image{
						width: 30rpx;
						height: 30rpx;
						margin-right: 8rpx;
					}
				}
				.m-m-content{
					padding: 15rpx;
					background-color: #fff;
					pre{
						width: 100%;
						white-space: pre-wrap;
						word-wrap: break-word;
					}
				}

			}
			.my-msg{
				margin-left: 180rpx;
				.m-m-top{
					background-color: #2066C7;
				}
			}
			.ai-msg{
				margin-left: 10rpx;
				.m-m-top{
					background-color: #297810;

				}

			}
		}

		.buttom-box{
			position: absolute;
			width: 680rpx;
			bottom: 0;
			background-color: #2066C7;
			border-radius: 30rpx 30rpx 0rpx 0;
			padding: 15rpx 35rpx;
			display: flex;
			.buttom-left{
				padding: 10rpx 10rpx 4rpx;

				background-color: #E8E9EE;
				image{
					width: 40rpx;
					height: 39rpx;
				}
			}
			input{
				background-color: #fff;
				height: 60rpx;
				flex: 1;
				padding: 0 15rpx;
			}
			button{
				background-color: #2066C7;
				color: #fff;
				height: 60rpx;
				border: 1px solid #fff;
				border-radius: 2rpx 6rpx 6rpx 2rpx;
				display: flex;
				line-height: 40rpx;
				padding: 10rpx 12rpx;

				image{
					margin-left: 10rpx;
					width: 40rpx;
					height: 40rpx;
				}
			}


		}
	}
</style>
