<template>
	<view class='location-bar'>
		<div class='location-bar__city' @click="selectCityHandler">
			<span>{{isCityLoading?'定位中...':cityShow}}</span>
			<span class='location-bar__icon--arrow-down' />
		</div>
	</view>
</template>

<script>
	const API_KEY = 'SO2BZ-N4EL4-CHUUR-XSJMO-4A3LJ-IGBG4'
	const QQMapWX = require('./libs/qqmap-wx-jssdk.min.js');
	export default {
		props: {
			cityObj: {
				type: Object,
				default: {
					name: "",
					id: null,
				}
			},
			disableAutoLocation: {
				type: Boolean
			},
		},
		data() {
			return {
				mapSDK: null,
				cityShow: '',
				isCityLoading: false,
				failbackLocation: {
					name: '北京市',
					id: null
				}
			};
		},
		mounted() {
			// this.checkCode()
			this.initQQMapSDK()
			uni.$on('CityList:SelectedObj', this.setCityValue)
			this.initCity()

		},
		beforeDestroy() {
			uni.$off('CityList:SelectedObj', this.setCityValue)
		},
		methods: {
			// checkCode() {
			// 	if (!uni.getStorageSync('LocationBar:ActiveCity:Check')) {
			// 		uni.showModal({
			// 			title: 'location-bar弹窗',
			// 			content: "代码可能缺失，App.vue 中未检测到 onShow 中清除位置信息的方法，uni.setStorageSync('LocationBar:ActiveCity','empty')",
			// 			showCancel:false 
			// 		})
			// 	}
			// },
			initQQMapSDK() {
				this.mapSDK = new QQMapWX({
					key: API_KEY
				});
			},
			initCity() {
				//1、选过城市，就用选过的 
				const savedCity = uni.getStorageSync('LocationBar:ActiveCity')
				if (savedCity) {
					// console.log('选择记录',savedCity,typeof savedCity) 
					const savedCityObj = JSON.parse(savedCity)
					if (savedCityObj.name) {
						this.setCityValue(savedCityObj)
						return
					}
				}
				//2、没有选过城市。
				if (this.disableAutoLocation) {
					// console.log('父级传参',this.cityObj) 
					// 用父级参数传的
					if (this.cityObj) {
						this.setCityValue({ ...this.cityObj
						})
					}
				} else {
					// console.log('定位获取') 
					// 通过定位获取
					this.setAutoLocation()
				}
			},
			selectCityHandler() {
				uni.navigateTo({
					url: '/pages/cityList/cityList',
				})
			},
			// 修改选中city对象值的逻辑全部通过这个函数实现：自动定位改变，菜单选择改变等
			setCityValue(item) {
				if (!item || !item.name) {
					if (this.failbackLocation) {
						item = this.failbackLocation
					}
				}
				this.cityShow = item && item.name ? item.name : '定位失败'
				this.$emit('update:cityObj', item)
				uni.setStorageSync('LocationBar:ActiveCity', JSON.stringify(item))
			},
			setAutoLocation() {
				// console.log('坐标',latitude,longitude)
				// 腾讯位置服务平台 https://lbs.qq.com/dev/console
				// sdk文档 https://lbs.qq.com/miniProgram/jsSdk/jsSdkGuide/methodReverseGeocoder
				this.isCityLoading = true
				this.mapSDK.reverseGeocoder({
					// location: {// 不传输则默认当前位置
					// 	latitude,
					// 	longitude
					// },
					coord_type: 1, //坐标系
					// get_poi:0,
					// poi_options:{},
					// sig:'',
					success: (re) => {
						// console.log('位置信息re',re) 
						if (re.status === 0) {
							let autoCity = (re.result && re.result.address_component && re.result.address_component.city) || ''
							this.setCityValue({
								name: autoCity,
								id: null,
							})
						}
					},
					fail: (autoAdressFail) => {
						this.setCityValue({
							name: '',
							id: null,
						})
						console.log('autoAdressFail', autoAdressFail)
					},
					complete: () => {
						this.isCityLoading = false
					}
				})
			},
		}
	}
</script>

<style lang='scss' scoped>
	.location-bar {
		box-sizing: border-box;
		position: fixed;
		top: 0;
		width: 100%;
		height: 24px;
		z-index: 1000;

		&__city {
			box-sizing: border-box;
			height: 24px;
			background-color: #FFFFFF;
			color: #333333;
			font-size: 13px;
			padding: 0 20px 0 20px;
			overflow: visible;
		}

		&__icon--arrow-down {
			box-sizing: border-box;
			border-left: 1px solid #333333;
			border-bottom: 1px solid #333333;
			border-radius: 1px 0 1px 0;
			height: 6px;
			width: 6px;
			transform: translate(8px, -50%) rotate(-45deg);
			border-right: 2px solid transparent;
			border-top: 2px solid transparent;
			display: inline-block;

		}
	}
</style>
