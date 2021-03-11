<template>
	<view class='index-list'>
		<view class='index-list__head'>
			<input v-model="citySearch" placeholder="输入城市名称查询" class="index-list__input"
				placeholder-class="index-list__input__placeholder" />
		</view>
		<scroll-view :scroll-y='true'
			:scroll-into-view='"list-archor-" + encodeURIComponent(this.currentActiveIndex).replace("%", "_")'
			class='index-list__scroller'>
			<div v-for='groupKey in Object.keys(dataFiltered)' :key='groupKey'>
				<div class="index-list__group-head" :id='"list-archor-"+ encodeURIComponent(groupKey).replace("%","_")'>
					{{groupKey}}
				</div>
				<div class="index-list__group-cell" v-for='city in dataFiltered[groupKey]'
					@click="()=>{cityClickHandler(city)}" :clickable='true' :key="city.id">{{city.name}}
				</div>
			</div>
			<div style='height: 45px;width: 100%;'></div>
		</scroll-view>
		<view v-if='dataFiltered&&Object.keys(dataFiltered).length>0' class='index-list__index'
			@touchmove="indexTouchMoveHandler" @touchstart="indexTouchMoveHandler" @touchend="isZoomActiveIndex=false">
			<div class='index-list__index__wrapper'>
				<div v-for='groupKey in Object.keys(dataFiltered)' :key='groupKey' class='index-list__index__letter'
					:class='{"index-list__index__letter--active":currentActiveIndex===groupKey&&isZoomActiveIndex}'>
					{{groupKey}}
				</div>
			</div>
		</view>
	</view>
</template>

<script>
	export default {
		name: "missthee-indexlist",
		props: {
			data: {
				type: Object,
				default: {}
			}
		},
		data() {
			return {
				dataFiltered: {},
				currentActiveIndex: '',
				isZoomActiveIndex: false,
				citySearch: '',
			}
		},
		watch: {
			citySearch: {
				immediate: true,
				handler(newVal) {
					if (!this.data) {
						return {}
					}
					const resultObj = {}
					for (const groupkey of Object.keys(this.data)) {
						const cityListResult = this.data[groupkey].filter(item => item.name.indexOf(newVal) >= 0)
						if (cityListResult && cityListResult.length > 0) {
							resultObj[groupkey] = cityListResult
						}
					}
					this.dataFiltered = resultObj
				}
			}
		},
		methods: {
			indexTouchMoveHandler(e) {
				this.isZoomActiveIndex = true
				const touch = e.touches[0];
				const target = e.target
				let currentIndex = Math.floor((touch.clientY + Object.keys(this.dataFiltered).length * 15 / 2 - target.offsetTop) / 15)
				currentIndex = Math.min(Object.keys(this.dataFiltered).length - 1, currentIndex)
				currentIndex = Math.max(0, currentIndex)
				this.currentActiveIndex = Object.keys(this.dataFiltered)[currentIndex]
			},
			cityClickHandler(item) {
				this.$emit('select-item', item)
			},
		}
	}
</script>

<style lang='scss' scoped>
	* {
		font-family: Consolas, sans-serif;
		box-sizing: border-box !important;
		margin: 0;
		padding: 0;
	}

	.index-list {
		position: absolute;
		top: 0;
		bottom: 0;
		width: 100%;

		&__head {
			position: absolute;
			top: 0;
			width: 100%;
			height: 50px;
			overflow: hidden;
			background-color: $uni-bg-color;
			border-bottom: 1px solid $uni-bg-color-grey;
		}

		&__input {
			box-sizing: border-box;
			width: 90%;
			height: 30px;
			padding: 0 20px;
			margin: auto;
			height: 30px;
			margin: 10px auto;
			line-height: 30px;
			background-color: $uni-bg-color-grey;
			border-radius: 30px;
			font-size: 12px;
			color: $uni-text-color;

			&__placeholder {
				text-align: center;
				color: $uni-text-color-placeholder;
			}
		}

		&__scroller {
			position: absolute;
			top: 50px;
			bottom: 0;
			width: 100%;
			background-color: $uni-bg-color-grey;
		}

		&__group-head {
			box-sizing: border-box;
			background-color: $uni-bg-color-grey;
			color: $uni-text-color-grey;
			height: 35px;
			line-height: 35px;
			padding: 0 0 0 15px;
		}

		&__group-cell {
			box-sizing: border-box;
			background-color: $uni-bg-color;
			color: $uni-text-color;
			font-size: 14px;
			font-family: SourceHanSansSC-Regular, SourceHanSansSC;
			height: 45px;
			line-height: 45px;
			padding: 0 0 0 15px;

			&:after {
				display: block;
				position: absolute;
				content: '';
				width: 100%;
				height: 1px;
				background-color: $uni-bg-color-grey;
			}

			&:active {
				background-color: $uni-bg-color-hover;
			}
		}

		&__index {
			position: absolute;
			right: 0;
			top: 50%;
			transform: translateY(-50%);

			&__wrapper {
				width: 20px;
				margin: 0 7px 0 0;
				text-align: center;
				background-color: rgba(0, 0, 0, 0.03);
				border-radius: 10px;
				color: $uni-text-color-grey;
				padding: 10px 0;
				font-family: Consolas, sans-serif;
			}

			&__letter {
				display: block;
				height: 15px !important;
				line-height: 15px !important;
				transition: transform 0.1s;

				&--active {
					transition: none;
					font-weight: bold;
					font-size: 50px;
					transform: translateX(-70px);
				}
			}
		}
	}
</style>
