<template>
	<view class="warp">
		<view class="text-area">
			<text class="title">{{ title }}</text>
		</view>
		<uni-section title="动态加载" type="line"></uni-section>
		<view class="example-body">
			<view v-if="dynamicList.length !== 0" class="grid-dynamic-box">
				<uni-grid :column="3" :highlight="true" @change="change">
					<uni-grid-item v-for="(item, index) in dynamicList" :index="index" :key="index">
						<view class="grid-item-box" :style="{ backgroundColor: item.color }">
							<image :src="item.url" class="image" mode="aspectFill" />
							<text class="text">{{ item.text }}</text>
						</view>
					</uni-grid-item>
				</uni-grid>
			</view>
			<button type="primary" @click="add">点击添加一个宫格</button>
			<button v-if="dynamicList.length !== 0" type="primary" style="margin-top: 15px;" @click="del">点击删除一个宫格</button>
		</view>
	</view>
</template>

<script>
import { uniBadge, uniGrid, uniSection, uniGridItem } from '@dcloudio/uni-ui';
//import uniBadge from '@dcloudio/uni-ui/lib/uni-badge/uni-badge.vue' //也可使用此方式引入组件

var $mqtt = require('@/common/js/mqttws31.js');

var client;
var host = '192.168.3.117';
var port = 61614;
var timestamp3 = new Date().getTime();
var clientId = 'sampleid' + timestamp3;
var sentMsgs = 0;
var connectOptions = connectOptions || {};
var mqtt;
var topic = 'aa2';
var serviceHost = 'http://generate.51xmm.net';
mqtt = new $mqtt.Client(
	host, //MQTT 域名
	port, //WebSocket 端口，如果使用 HTTPS 加密则配置为443,否则配置80
	clientId //客户端 ClientId
);
var options = {
	userName: 'admin',
	password: 'admin',
	keepAliveInterval: 5,
	// timeout:30,
	reconnect: true, // Enable automatic reconnect
	useSSL: false, //如果使用 HTTPS 加密则配置为 true
	onSuccess: function onConnect() {
		//连接成功
		console.log(new Date() + ': ' + '连接成功');
		// Connection succeeded; subscribe to our topic
		//订阅消息
		mqtt.subscribe(topic, {
			qos: 0
		});
		//测试消息推送
		var message = new $mqtt.Message('APP测试消息推送!!'); //set body
		message.destinationName = topic; // set topic
		// //发送 P2P 消息，topic 设置方式参考https://help.aliyun.com/document_detail/96176.html?spm=a2c4g.11186623.6.586.694f7cb4oookL7
		// message = new $mqtt.Message("Hello mqtt P2P Msg!!"); //set body
		// message.destinationName = topic + "/p2p/" + clientId; // set topic
		mqtt.send(message);
	},
	onFailure: function(message) {
		//连接失败
		console.log(new Date() + ': ' + '连接失败' + JSON.stringify(message));
		//连接失败,重新连接
	}
};
mqtt.onConnectionLost = function onConnectionLost(response) {
	console.log(new Date() + ': ' + '连接中断' + JSON.stringify(response));
	//连接失败,重新连接
};
mqtt.onMessageArrived = function onMessageArrived(message) {
	var topic = message.destinationName;
	var payload = message.payloadString;
	console.log(new Date() + ': ' + '收到新消息' + topic + '   ' + payload);
};
mqtt.connect(options);

export default {
	components: { uniBadge, uniGrid, uniSection, uniGridItem },
	data() {
		return {
			title: 'Hello',
			dynamicList: [
				{
					url: '/static/c1.png',
					text: 'Grid 1',
					badge: '0',
					type: 'primary'
				}
			]
		};
	},
	onLoad() {},
	methods: {
		sendMsg: function(topic, content) {
			var _message = new $mqtt.Message(content);
			_message.destinationName = topic;
			this.title = content;
			mqtt.send(_message);
		},
		change(e) {
			let { index } = e.detail;
			this.list[index].badge && this.list[index].badge++;

			uni.showToast({
				title: `点击第${index + 1}个宫格`,
				icon: 'none'
			});
		},
		add() {
			if (this.dynamicList.length < 9) {
				this.dynamicList.push({
					url: `/static/c${this.dynamicList.length + 1}.png`,
					text: `Grid ${this.dynamicList.length + 1}`,
					color: this.dynamicList.length % 2 === 0 ? '#f5f5f5' : '#fff'
				});
			} else {
				uni.showToast({
					title: '最多添加9个',
					icon: 'none'
				});
			}
			this.sendMsg('aa2', 'add Grid'+this.dynamicList.length);
			/* this.client.publish('aa2', 'add Grid' + this.dynamicList.length); */
		},
		del() {
			this.dynamicList.splice(this.dynamicList.length - 1, 1);
			this.sendMsg('aa2', 'del Grid'+(this.dynamicList.length+1));
			/* this.client.publish('aa2', 'del Grid' + (this.dynamicList.length + 1)); */
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
<style lang="scss">
@import '@/common/uni-nvue.scss';
.image {
	width: 50rpx;
	height: 50rpx;
}

.text {
	font-size: 26rpx;
	margin-top: 10rpx;
}

.example-body {
	/* #ifndef APP-NVUE */
	display: block;
	/* #endif */
}

.grid-dynamic-box {
	margin-bottom: 15px;
}

.grid-item-box {
	flex: 1;
	// position: relative;
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 15px 0;
}

.grid-dot {
	position: absolute;
	top: 5px;
	right: 15px;
}
.swiper {
	height: 420px;
}
</style>
