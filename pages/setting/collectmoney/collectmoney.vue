<template>
	<view>
		<view class="box01">
			<view class="headBox">
				<text class="iconfont iconleft1" @click="backToBeforePage()"></text>
				<view class="headTitle">添加收款方式</view>
				<navigator class="rightBox">历史记录</navigator>
			</view>
		</view>
		<view class="tishi">请绑定*东方红的支付宝账号</view>
		<view class="form">
			<view class="uni-padding-wrap uni-common-mt">
				<form @submit="formSubmit" @reset="formReset">
					<view class="uni-form-item uni-column">
						<input class="uni-input" type="number" name="mobile" v-model="mobile" placeholder="手机号" />
						<!--<button open-type='getPhoneNumber' @getphonenumber='getPhoneNumber'>自动填入</button>-->
					</view>
					<view class="uni-form-item uni-column yanzhengma">
						<input class="uni-input" name="captcha" v-model="captcha" placeholder="验证码" />
						<view :class="sendMsgBtnClass" @tap="sendMsg">{{sendMsgBtnText}}</view>
					</view>
					<view class="uni-btn-v">
						<button class="btn" formType="submit">提交</button>
					</view>
				</form>
			</view>
		</view>
	</view>
</template>

<script>
	import uniTag from "@/components/uni-tag/uni-tag.vue"
	// import http from '../../../components/utils/http.js'
	// import {
	// 	mapState,
	// 	mapMutations
	// } from 'vuex'
	export default {
		components: {
			uniTag
		},
		// computed: {
		// 	...mapState(['isLogin', 'forcedLogin', 'user_id', 'group_id'])
		// },
		data() {
			return {
				mobile: "",
				captcha: "",
				hasMobile:false,
				sendMsgBtnText: "获取验证码",
				sendMsgBtnClass: "button",
				isCountdown: false,
				isFormSubmit: false,
			}
		},
		onLoad() {
			// this.mobile=this.$store.state.userInfo.mobile;
			if(/^1[23456789]\d{9}$/.test(this.mobile)){
				this.hasMobile=true;
			}
		},
		methods: {
			getPhoneNumber(res) {
				console.log(res);
			},
			formSubmit: function(e) {

				var that = this;
				var mobile = that.mobile;
				var captcha = that.captcha;
				if (this.isFormSubmit) {
					return;
				}
				if ((/^1[23456789]\d{9}$/.test(mobile)) && (/^\d{4}$/.test(captcha))) {
					console.log('form发生了submit事件，携带数据为：' + JSON.stringify(e.detail.value.mobile));
					this.isFormSubmit = true;
					http.apiRequest({
						api: "/api/user/bandmobile",
						data: {
							mobile: mobile,
							captcha: captcha,
						},
						secuss: (res, httpStatus) => {
							this.isFormSubmit = false;
							if (res.code == "1") {
								//短信发送成功，进入倒计时
								console.log("绑定手机号成功");
								
								uni.showToast({title:"手机号绑定成功",duration:1500});
								setTimeout(()=>{
									uni.navigateBack({delta:1});
								},1500);
								
							} else {
								//短信发送失败
								uni.showToast({title:res.msg,duration:1500,icon:"none"});
								console.log("绑定手机号失败");
							}
						},
						error: (res, httpStatus) => {
							//接口调用失败，可能是网络故障，可能是接口报错
							this.isFormSubmit = false;
							console.log("接口调用失败，请检查网络");
						},
					});
				}

			},
			sendMsg(e) {
				var that = this;
				var mobile = that.mobile;

				if (this.isCountdown) {
					return;
				}
				if ((/^1[23456789]\d{9}$/.test(mobile))) {
					this.isCountdown = true;
					http.apiRequest({
						api: "/api/sms/send",
						data: {
							mobile: mobile,
							event: "bandmobile",
						},
						secuss: (res, httpStatus) => {
							if (res.code == "1") {
								//短信发送成功，进入倒计时
								var countdownTimer = 60;
								var countdown = () => {
									countdownTimer--;
									if (countdownTimer > 0) {
										that.sendMsgBtnText = "(" + countdownTimer + ")秒后重新发送";
										that.sendMsgBtnClass = "button2";
										setTimeout(function() {
											countdown();
										}, 1000);
									} else {
										this.isCountdown = false;
										that.sendMsgBtnText = "发送验证码";
										that.sendMsgBtnClass = "button1";
									}
								}
								countdown();
							} else {
								//短信发送失败
								this.isCountdown = false;
								uni.showToast({title:res.msg,duration:1500,icon:"none"});
								console.log("短信验证码发送失败");
							}
						},
						error: (res, httpStatus) => {
							//接口调用失败，可能是网络故障，可能是接口报错
							this.isCountdown = false;
							console.log("接口调用失败，请检查网络");
						},
					});
				} else {
					console.log("请输入正确的是手机号");
				}
			}

		}
	}
</script>

<style>
	.headBox{display: flex;align-items: center;justify-content: space-between;}
	.headBox text{display: inline-block;width: 20%;font-size: 40rpx;color: #fff;}
	.rightBox{width: 20%;font-size: 28rpx;color: #fff;text-align: right;}
	.headTitle{font-size:40rpx;color: #fff;}
	.hideHead{background: #c82418 url(../../../static/images/titleBg.png) no-repeat center bottom 38rpx;position: fixed;width: 100%;left: 0;top:0;text-align: center;font-size: 40rpx;color: #fff;padding:80rpx 0 20rpx;z-index: 999;background-size: 220rpx 22rpx;}
	.box01{background: url(../../../static/images/indexBg.png) no-repeat;background-size: 100%;padding: 80rpx 20rpx 40rpx;}
	.tishi {width: 100%;height: 80rpx;line-height: 80rpx;padding-left: 40rpx;box-sizing: border-box;font-size: 30rpx;color: #999;background-color: #fff9f7;}
	
	.form {
		padding: 40upx;
		padding-top: 0;
		width: 100%;
		box-sizing: border-box;
	}
	.uni-input {
		border-bottom:1px solid #999;
		padding-top: 30upx;
		padding-bottom:30upx;
	}
	.btn {
		width: 520upx;
		height: 88upx;
		font-size: 38upx;
		color: #fff;
		background: #c82418;
		line-height: 88upx;
		border-radius: 44upx;
		text-align: center;
		margin-top: 80upx;
	}
	.yanzhengma {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
	.yanzhengma view {
		font-size: 30upx;
		color: #c82418;
	}
</style>
