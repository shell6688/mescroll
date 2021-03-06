<template>
	<view :style="{'padding-top':padTop,'padding-bottom':padBottom}" @touchstart="touchstartEvent" @touchmove="touchmoveEvent" @touchend="touchendEvent" @touchcancel="touchendEvent">
		<!-- 下拉加载区域 -->
		<view v-if="optDown" class="mescroll-downwarp" :class="{'mescroll-downwarp-reset':isDownReset}" :style="{'height': downHight+'px', 'position': 'relative', 'overflow': 'hidden', '-webkit-transition': isDownReset?'height 300ms':''}">
			<view class="downwarp-content" style="text-align: center;position: absolute;left: 0;bottom: 0;width: 100%;padding: 20upx 0;">
				<view v-if="isDownLoading" class="downwarp-progress"></view>
				<view v-if="!isDownLoading" class="downwarp-arrow" :style="{'transform':'rotate(' + downRotate + 'deg)'}"></view>
				<view class="downwarp-tip">{{downText}}</view>
			</view>
		</view>

		<!-- 列表内容 -->
		<slot></slot>

		<!-- 空布局 -->
		<view v-if="optEmpty&&isShowEmpty" class="mescroll-empty">
			<image v-if="optEmpty.icon" class="empty-icon" :src="optEmpty.icon" mode="widthFix" />
			<view v-if="optEmpty.tip" class="empty-tip">{{optEmpty.tip}}</view>
			<view v-if="optEmpty.btnText" class="empty-btn" @click="emptyClick">{{optEmpty.btnText}}</view>
		</view>

		<!-- 上拉加载区域 -->
		<view v-if="optUp" class="mescroll-upwarp">
			<!-- 加载中.. -->
			<template v-if="isUpLoading">
				<view class="upwarp-progress mescroll-rotate"></view>
				<view class="upwarp-tip">{{optUp.textLoading}}</view>
			</template>
			<!-- 无数据 -->
			<view v-if="isUpNoMore" class="upwarp-nodata">{{optUp.textNoMore}}</view>
		</view>

		<!-- 回到顶部按钮 -->
		<image v-if="optToTop" class="mescroll-totop" :class="{'mescroll-fade-in':isShowToTop}" :src="optToTop.src" mode="widthFix" @click="toTopClick" />
	</view>
</template>

<script>
	// 引入mescroll-uni.js,处理核心逻辑
	import MeScroll from '../mescroll-uni/mescroll-uni.js';
	// 引入全局配置
	import GlobalOption from './mescroll-xinlang-option.js';

	export default {
		data() {
			return {
				mescroll: null,
				downHight: 0, //下拉刷新: 容器高度
				downRotate: 0, //下拉刷新: 圆形进度条旋转的角度
				downText: '', //下拉刷新: 提示的文本
				isDownReset: false, //下拉刷新: 是否显示重置的过渡动画
				isDownLoading: false, //下拉刷新: 是否显示加载中
				isUpLoading: false, // 上拉加载: 是否显示 "加载中..."
				isUpNoMore: false, // 上拉加载: 是否显示 "-- END --"
				isShowEmpty: false, // 是否显示空布局
				isShowToTop: false // 是否显示回到顶部按钮
			}
		},
		props: {
			down: Object,
			up: Object,
			top: [String,Number],  // padding-top的数值,单位upx. 目的是使下拉布局往下偏移
			bottom: [String,Number]  // padding-bottom的数值,单位upx. 目的是使上拉布局往上偏移
		},
		computed: {
			// 下拉刷新的配置
			optDown() {
				return this.mescroll ? this.mescroll.optDown : null;
			},
			// 上拉加载的配置
			optUp() {
				return this.mescroll ? this.mescroll.optUp : null;
			},
			// 空布局的配置
			optEmpty() {
				return this.mescroll ? this.mescroll.optUp.empty : null;
			},
			// 回到顶部的配置
			optToTop() {
				return this.mescroll ? this.mescroll.optUp.toTop : null;
			},
			// padding-top的数值,单位upx,需转成px. 目的是使下拉布局往下偏移
			padTop(){
				return uni.upx2px(Number(this.top) || 0)+'px';
			},
			// padding-bottom的数值,单位upx,需转成px 目的是使上拉布局往上偏移
			padBottom(){
				return uni.upx2px(Number(this.bottom) || 0)+'px';
			}
		},
		methods: {
			//注册列表touchstart事件,用于下拉刷新
			touchstartEvent(e) {
				this.mescroll && this.mescroll.touchstartEvent(e);
			},
			//注册列表touchmove事件,用于下拉刷新
			touchmoveEvent(e) {
				this.mescroll && this.mescroll.touchmoveEvent(e);
			},
			//注册列表touchend事件,用于下拉刷新
			touchendEvent(e) {
				this.mescroll && this.mescroll.touchendEvent(e);
			},
			// 点击空布局的按钮回调
			emptyClick(){
				this.$emit('emptyclick',this.mescroll)
			},
			// 点击回到顶部的按钮回调
			toTopClick(){
				this.isShowToTop = false; // 回到顶部按钮需要先隐藏,再执行回到顶部,避免闪动
				uni.pageScrollTo({ // 执行回到顶部
					scrollTop: 0,
					duration: this.mescroll.optUp.toTop.duration
				});
				this.$emit('topclick',this.mescroll) // 派发点击回到顶部按钮的回调
			}
		},
		// 编译到H5,不会执行onReady,只执行mounted
		mounted() {
			let vm = this;

			let diyOption = {
				// 下拉刷新的配置
				down: {
					inOffset(mescroll) {
						// 下拉的距离进入offset范围内那一刻的回调
						vm.isDownReset = false; // 不重置高度 (自定义mescroll组件时,此行不可删)
						vm.isDownLoading = false; // 不显示加载中
						vm.downText = mescroll.optDown.textInOffset; // 设置文本
						vm.downRotate = 0; // 旋转到0
					},
					outOffset(mescroll) {
						// 下拉的距离大于offset那一刻的回调
						vm.isDownReset = false; // 不重置高度 (自定义mescroll组件时,此行不可删)
						vm.isDownLoading = false; // 不显示加载中
						vm.downText = mescroll.optDown.textOutOffset; // 设置文本
						vm.downRotate = -180; // 旋转到-180
					},
					onMoving(mescroll, rate, downHight) {
						// 下拉过程中的回调,滑动过程一直在执行; rate下拉区域当前高度与指定距离的比值(inOffset: rate<1; outOffset: rate>=1); downHight当前下拉区域的高度
						vm.downHight = downHight; // 设置下拉区域的高度 (自定义mescroll组件时,此行不可删)
					},
					showLoading(mescroll, downHight) {
						// 显示下拉刷新进度的回调
						vm.isDownReset = true; // 重置高度 (自定义mescroll组件时,此行不可删)
						vm.isDownLoading = true;// 显示加载中
						vm.downHight = downHight; // 设置下拉区域的高度 (自定义mescroll组件时,此行不可删)
						vm.downText = mescroll.optDown.textLoading; // 设置文本
						vm.downRotate = 0; // 旋转到0
					},
					endDownScroll(mescroll) {
						vm.isDownReset = true;// 重置高度 (自定义mescroll组件时,此行不可删)
						vm.isDownLoading = false; // 不显示加载中
						vm.downHight = 0; // 设置下拉区域的高度 (自定义mescroll组件时,此行不可删)
					},
					// 派发下拉刷新的回调
					callback: function(mescroll){
						vm.$emit('down', mescroll)
					}
				},
				// 上拉加载的配置
				up: {
					// 显示加载中的回调
					showLoading() {
						vm.isUpLoading = true;
						vm.isUpNoMore = false;
					},
					// 显示无更多数据的回调
					showNoMore() {
						vm.isUpLoading = false;
						vm.isUpNoMore = true;
					},
					// 隐藏上拉加载的回调
					hideUpScroll() {
						vm.isUpLoading = false;
						vm.isUpNoMore = false;
					},
					// 空布局
					empty: {
						onShow(isShow) { // 显示隐藏的回调
							if(vm.isShowEmpty != isShow) vm.isShowEmpty = isShow;
						}
					},
					// 回到顶部
					toTop: {
						onShow(isShow) { // 显示隐藏的回调
							if (vm.isShowToTop != isShow) vm.isShowToTop = isShow;
						}
					},
					// 派发上拉加载的回调
					callback: function(mescroll){
						vm.$emit('up', mescroll)
					}
				}
			}

			MeScroll.extend(diyOption, GlobalOption); // 混入全局的配置
			let myOption = MeScroll.extend({
				'down': vm.down ? JSON.parse(JSON.stringify(vm.down)) : vm.down, // 深拷贝,避免对props的影响
				'up': vm.up ? JSON.parse(JSON.stringify(vm.up)) : vm.up // 深拷贝,避免对props的影响
			}, diyOption); // 混入具体界面的配置
			
			// 初始化MeScroll对象
			vm.mescroll = new MeScroll(myOption);
			// init回调mescroll对象
			vm.$emit('init', vm.mescroll);

			// 设置mescroll实例对象的body高度,使down的bottomOffset生效
			uni.getSystemInfo({
				success(res) {
					vm.mescroll.setBodyHeight(res.windowHeight);
				}
			});
		}
	}
</script>

<style>
	@import "../mescroll-uni/mescroll-uni.css";
	
	/*下拉刷新--上下箭头*/
	.mescroll-downwarp .downwarp-arrow{
		display: inline-block;
		width: 20px;
		height: 20px;
		margin: 10px;
		background-image: url(http://www.mescroll.com/img/xinlang/mescroll-arrow.png);
		background-size: contain;
		vertical-align: middle;
		-webkit-transition: all 300ms;
		transition: all 300ms;
	}
	
	/*下拉刷新,上拉加载--旋转进度条*/
	.mescroll-downwarp .downwarp-progress,
	.mescroll-upwarp .upwarp-progress{
		width: 36px;
		height: 36px;
		border: none;
		margin: auto;
	    background-size: contain;
		-webkit-animation: progressRotate .6s steps(6,start) infinite;
		animation: progressRotate .6s steps(6,start) infinite;
	}
	@-webkit-keyframes progressRotate {
		0% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress1.png)}
		16% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress2.png)}
		32% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress3.png)}
		48% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress4.png)}
		64% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress5.png)}
		80% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress6.png)}
		100% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress1.png)}
	}
	@keyframes progressRotate {
		0% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress1.png)}
		16% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress2.png)}
		32% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress3.png)}
		48% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress4.png)}
		64% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress5.png)}
		80% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress6.png)}
		100% {background-image: url(http://www.mescroll.com/img/xinlang/mescroll-progress1.png)}
	}
</style>
