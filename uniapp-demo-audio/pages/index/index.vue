<template>
	<view class="info-list">
		<view class="line-top">
			<view class="line"></view>
		</view>
		
		<view>
			<neil-modal :show="show2" @close="closeModal('2')" content="当前appId 为空, 从蓝蘑云网站[http://account.lmaple.com/]注册appID">
			</neil-modal>
		</view>
		
		<view>
		    <button type="default" @click="bindClick('1')">多人语音聊天</button>
		    <neil-modal :show="show1" @close="closeModal('1')" @confirm="confirmModal('1')" title="请输入房间和用户名">
		        <view class="input-view">
		            <view class="input-name">
		                <view>用户名</view>
		                <input type="text" placeholder="请输入用户Id" v-model="userId" />
		            </view>
		            <view class="input-name">
		                <view>房间ID</view>
		                <input type="text" placeholder="请输入房间ID" v-model="roomId" />
		            </view>
		        </view>
		    </neil-modal>
		</view>
	</view>
</template>

<script>
	const modal = uni.requireNativePlugin('modal');
	const mapleCloud = uni.requireNativePlugin('LM-mapleRTCA');
	import { appId } from '@/common/app.js';
	
	import neilModal from '@/components/neil-modal.vue';
	export default {
		components: {
		    neilModal
		},
		
		data() {
			return {
				show1: false,
				show2: false,
				userId:"",
				roomId:""
			};
		},
		onReady() {
			if(uni.getSystemInfoSync().platform == 'android'){
				mapleCloud.checkPermission();
			}
			
			if(appId == ""){
				
				 this[`show2`] = true
			return;
			}
		},
		
		methods: {
			
			bindClick(type) {
			    console.log(this[`show${type}`])
			    this[`show${type}`] = true
			    console.log(this[`show${type}`])
			},
			closeModal(type) {
			    console.log(`监听到close`)
			    this[`show${type}`] = false
			},
			confirmModal(type) {
			    console.log(`监听到confirm`)
				
				var userId = this.userId;
				var channelId = this.roomId;
				
				if(channelId == "" || userId == ""){
					uni.showToast({
						title: '用户id和房间Id为空',
						icon:'none'
					});
					this[`show${type}`] = false
				return;
				}
				
				if (type == 1) {
					uni.navigateTo({
						url:"../audioroom/audioroom?channelId="+channelId+"&userId="+userId
					})
					
				} 
				
				
			    this[`show${type}`] = false
			}
		}
	}
</script>

<style>
	
	.info-list {
		/* margin-top: 20upx; */
		padding: 0upx 30upx;
	}
	
	.item-wapper {
		display: flex;
		/* margin-top: 20upx; */
		flex-direction: row;
		justify-content: flex-start;
	}
	
	.info-words {
		color: #333333;
		font-size: 16px;
		width: 25%;
		line-height: 80upx;
		/* font-weight: bold; */
	}
	
	.right-wapper {
		width: 80%;
		display: flex;
		flex-direction: row;
		justify-content: flex-end;
	}
	
	.arrow-block {
		margin-left: 10upx;
		line-height: 86upx;
	}
	
	.arrow-ico {
		width: 30upx;
		height: 30upx;
	}
	
	.input {
		height: 80upx;
		line-height: 80upx;
		width: 500upx;
		margin-left: 40upx;
	}
	
	.graywords {
		color: #EAEAEA;
	}
	
	.input-name {
	    height: 80upx;
	    width: 100%;
	    display: flex;
	    flex-direction: row;
	    justify-content: center;
	    align-items: center;
	    position: relative;
	    padding-left: 30upx;
	    box-sizing: border-box;
	}
	
	.input-name::after {
	    content: " ";
	    position: absolute;
	    left: 30upx;
	    bottom: 0;
	    right: 0;
	    height: 1px;
	    border-top: 1px solid #e5e5e5;
	    transform-origin: 0 0;
	    transform: scaleY(.5);
	}
	
	.input-name view {
	    width: 120upx;
	    height: 50upx;
	    line-height: 50upx;
	    font-size: 28upx;
	    color: #333333;
	}
	
	.input-name input{
	    flex: 1;
	    height: 50upx;
	    line-height: 50upx;
	}
	
</style>