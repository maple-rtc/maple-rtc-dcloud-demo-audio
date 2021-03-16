<template>
	<scroll-view class="page" :style="[windowHeight ? {height: windowHeight + 'px'} : '']" scroll-y="true">
    
    <text class="page-title">多人语音房间号: {{roomId}}</text>
    
		<view class="voiceroom">

		  <view class="voiceroom-cel voiceroom-pusher">
			<view class="voiceroom-maplertc" ref="我" ></view>
			<text class="voiceroom-cel-name">我</text>
		  </view>

		  <view class="voiceroom-cel voiceroom-player" v-for="(item, index) in players" :key="index">
			<view class="voiceroom-maplertc" :ref="'remoteUser'">{{item.userId}}</view>
			<text class="voiceroom-cel-name">{{item.userId}}</text>
		  </view>
		</view>
		
		<view class="tools">
			<view class="circleBoxAll">
				<view class="circleBox" @click="menuAction()">
					<text class="circleIco phone">&#xe626;</text>
					<text class="hint">菜单操作</text>
				</view>
			</view>
		</view>

	</scroll-view>
</template>

<script>
	import { appId } from '@/common/app.js';
	const mapleCloud = uni.requireNativePlugin('LM-mapleRTCA');
	const modal = uni.requireNativePlugin('modal');
	var ownUid = "";
	var isMuted = false;
	var is_open_mic = true;
	var isSpeakerphone = true;
	var isMicAndSpeakerphone = true;
	var isPausedMedia = true;
	var isMutedMedia = true;

export default {
  data() {
    return {
      windowHeight: 0,
      roomId: 123,
	  players:[],
	  session:{},
	  appId:appId
    }
  },

  onLoad(params) {
	  
	this.windowHeight = uni.getSystemInfoSync().windowHeight;
	this.roomId = params.channelId
	
	this.session = {
			channelId: params.channelId,
			userId: params.userId
		};
	
	//初始化
	mapleCloud.initialize({
		appId:appId
	},function(res){
		uni.showToast({
			title: res,
			icon:'none'
		});
	});
	
	//注册事件回调
	//加入频道成功监听
	mapleCloud.addEventListen({name: 'joinChannelSuccessListener'}, function(ret) {
		ownUid = ret.uid;
		uni.showToast({
			title: JSON.stringify(ret),
			icon:'none'
		});
	});
	
	//加入频道失败监听
	mapleCloud.addEventListen({name: 'joinChannelFailedListener'}, function(ret) {
		uni.showToast({
			title: JSON.stringify(ret),
			icon:'none'
		});
	});
	
	//连接丢失监听
	mapleCloud.addEventListen({name: 'connectionLostListener'}, function(ret) {
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
	});
	
	//离开频道监听
	mapleCloud.addEventListen({name: 'leaveChannelListener'}, function(ret) {
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
	});
	
	//媒体播放停止监听
	mapleCloud.addEventListen({name: 'mediaStopListener'}, function(ret) {
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
	});
	
	//踢下线监听
	mapleCloud.addEventListen({name: 'forceKickOutChannelListener'}, function(ret) {
		if(ret.reason == 4200)
			modal.toast({ message: "强行被其他用户踢出频道！", duration: 1.5});
		if(ret.reason == 4201)
			modal.toast({ message: "强行被服务器踢出频道！！", duration: 1.5});
	});
	
	//SDK错误监听
	mapleCloud.addEventListen({name: 'errorListener'}, function(ret) {
	if(ret.error == 4007)
		modal.toast({ message: "设备没有麦克风权限,在系统设置里打开相关权限！", duration: 1.5});
	else
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
	});

	//声音音量提示监听
	mapleCloud.addEventListen({name: 'audioVolumeIndicationListener'}, function(ret) {
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
	});
	//用户加入通知监听
	mapleCloud.addEventListen({name: 'userJoinedNoticeListener'}, this.onUserJoinedNoticeListener);
	//用户离开通知监听
	mapleCloud.addEventListen({name: 'userOfflineNoticeListener'}, this.onUserOfflineNoticeListener);

	//设置媒体模式
	mapleCloud.setAudioProfile({audioProfile:'musicHighQuality'});
  },
  
  onReady() {
    this.join();
  },
  
  onResize() {
    // #ifdef APP-PLUS-NVUE
    this.resizeWinHeight()
    // #endif
  },
  
  onNavigationBarButtonTap() {
  	uni.navigateBack({
  	    delta: 1,
  	    animationType: 'pop-out',
  	    animationDuration: 200
  	});
  },
  
  onBackPress() {
	//离开频道
	mapleCloud.leaveChannel(function(ret) {
	      modal.toast({ message: ret, duration: 1.5});
	   });
  },
  
  methods: {
    join(){
	   //加入频道
	   mapleCloud.joinChannel(this.session, function(ret) {
	      modal.toast({ message: ret, duration: 1.5});
	   });
    },
	
	//有用户加入通知
	onUserJoinedNoticeListener(ret) {
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
		for(var i=0; i < ret.uids.length; i++){
			
			if(ownUid == ret.uids[i]){
				continue;
			}
			this.addPlayer(ret.uids[i]);
		}
	},
	
	//有用户离开通知
	onUserOfflineNoticeListener(ret) {
		modal.toast({ message: JSON.stringify(ret), duration: 1.5});
		for(var i=0; i < ret.uids.length; i++){
			this.removePlayer(ret.uids[i]);
		}
	},
   
    addPlayer(userId) {
      this.players.push({
        userId: userId
      })
    },
    
    removePlayer(userId) {
      for (let i = 0; i < this.players.length; i++) {
        let d = this.players[i]
        if (d.userId === userId) {
          this.players.splice(i, 1)
          break
        }
      }
    },
	
	menuAction(){
		var me = this;
		 uni.showActionSheet({
			itemList: ["开关扬声器", "开关麦克风",'切换听筒和扬声器','播放媒体文件','暂停媒体','停止媒体','静音媒体','调节媒体音量','设置媒体播放位置','获取总时长','获取当前播放位置', '开关本地音频流'],
			success(res) {
				var index = res.tapIndex;
				if (index == 0) {
					var value;
					isSpeakerphone = !isSpeakerphone;
					if(isSpeakerphone){
						value = 100;
					}else{
						value = 0;
					}
					//开关扬声器
					mapleCloud.adjustPlaybackSignalVolume({volume:value}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 1) {
					
					//开关麦克风
					if(is_open_mic){
						mapleCloud.stopLocalAudio(function(ret) {
							uni.showToast({ title: JSON.stringify(ret),icon:'none'});
						});
					}else{
						mapleCloud.startLocalAudio(function(ret) {
							uni.showToast({ title: JSON.stringify(ret),icon:'none'});
						});
					}
					is_open_mic = !is_open_mic;
				} else if (index == 2) {
					isMicAndSpeakerphone = !isMicAndSpeakerphone;
					//切换听筒和扬声器
					mapleCloud.setEnableSpeakerphone({enabled : isMicAndSpeakerphone},  function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 3) {
					mapleCloud.startPlayMedia({path: "http://share.lmaple.com/test.mp3", type:3, cycle:1, index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
						isPausedMedia = false;
						isMutedMedia = false;
					});
				} else if (index == 4) {
					isPausedMedia = !isPausedMedia;
					mapleCloud.pausePlayMedia({paused: isPausedMedia, index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 5) {
					mapleCloud.stopPlayMedia({index:1},function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 6) {
					isMutedMedia = !isMutedMedia;
					mapleCloud.mutePlayMedia({muted: isMutedMedia, index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 7) {
					mapleCloud.adjustPlayMediaVolume({volume: 30, index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 8) {
					mapleCloud.setPlayMediaPosition({position: 10000, index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 9) {
					mapleCloud.getPlayMediaTotalDuration({index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 10) {
					 mapleCloud.getPlayMediaCurrentPosition({index:1}, function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				} else if (index == 11) {
					isMuted = !isMuted;
					//开关本地音频流
					mapleCloud.muteLocalAudioStream({muted : isMuted},  function(ret) {
						uni.showToast({ title: JSON.stringify(ret),icon:'none'});
					});
				}
			}
		 })
	},
    
    resizeWinHeight() {
      uni.getSystemInfo({
        success: (res) => {
          this.windowHeight = res.windowHeight
        }
      });
    },
  }
}
</script>

<style lang="scss">
.page {
  height: 100%;
  background-color: #271506;
}
.page-title {
  width: 750rpx;
  height: 80rpx;
  line-height: 80rpx;
  color: #fff;
  font-size: 28rpx;
  text-align: center;
}

.tools {
  position: fixed;
  right: 0;
  bottom: 100rpx;
  z-index: 11;

  width: 150rpx;
}

.circleBoxAll{
	// flex-direction: row;
	justify-content: center;
	align-items: center;
}
.circleIco{
	font-size: 36rpx;
	width:64rpx;
	height:64rpx;
	line-height: 64rpx;
	border-radius: 500px;
	border-style: solid;
	border-width: 1px; 
	border-color:#FFFFFF;
	text-align: center;
	font-family:iconfont;
	color: #FFFFFF;
}
.phone{
	background-color:#c5433a;
	border-style: solid;
	border-width: 1px; 
	border-color:#c5433a;
}
.circleBox{
	width: 150rpx;
	padding: 10rpx 0;
	margin:10rpx;
	align-items: center;
	flex-direction: column;
  text-align: center;
}
.hint{
	font-size: 22rpx;
	color: #FFFFFF;
	padding-top: 20rpx;
}


.voiceroom {
  flex-direction: row;
  flex-wrap: wrap;

  width: 750rpx;
}
$cel-width: 0.5 * 750rpx;
.voiceroom-cel {
  width: $cel-width;
  height: 1.382 * $cel-width + 40rpx;
  padding: 20rpx;
}
.voiceroom-maplertc {
  width: $cel-width - 40rpx;
  height: 1.382 * $cel-width - 40rpx;
  background-color: #444;
}

.voiceroom-cel-name {
  width: $cel-width - 40rpx;
  height: 40rpx;
  line-height: 40rpx;
  text-align: center;
  color: #999;
  font-size: 24rpx;
}
</style>
