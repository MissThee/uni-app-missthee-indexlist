<template>
	<view class='index-list'>
		<view class='index-list__head'>
			<input v-model="citySearch" :placeholder='placeholder' class="index-list__input"
				placeholder-class="index-list__input__placeholder" />
		</view>

		<div>
			<scroll-view v-if="!isBigData" :scroll-y='true'
				:scroll-into-view='"list-archor-" + encodeURIComponent(this.currentActiveIndex).replace("%", "_")'
				class='index-list__scroller'>
				<div v-for='groupKey in Object.keys(dataFiltered)'
					:id='"list-archor-"+ encodeURIComponent(groupKey).replace("%","_")' :key='groupKey'>
					<div class="index-list__group-head">
						{{groupKey}}
					</div>
					<div class="index-list__group-cell" v-for='item in dataFiltered[groupKey]'
						@click="()=>{itemClickHandler(item)}" :clickable='true' :key="item.id">{{item.name}}
					</div>
				</div>
				<div style='height: 45px;width: 100%;'></div>
			</scroll-view>
			<div v-else class='index-list__scroller'>
				<div ref="bigDataTablePanel" class="index-list__big-data__table-panel">
					<div ref="bigDataTableBody" class="index-list__big-data__table-body">
						<div ref="bigDataTableDataPanel" class="index-list__big-data__table-data-panel">
							<div ref="bigDataTableDataMarginSpace"></div>
							<div v-for='item in bigDataFilteredFlatWithLineInfoCurrent' :key='item.id'>
								<div v-if='item.__line_info__.isEmptySpace'
									class="index-list__group-cell index-list__group-cell--empty-space"></div>
								<div v-else-if='!item.__line_info__.isEmptySpace&&item.__line_info__.isHead'
									class="index-list__group-head">{{item.name}}</div>
								<div v-else-if='!item.__line_info__.isEmptySpace&&!item.__line_info__.isHead'
									class="index-list__group-cell" @click="()=>{ itemClickHandler(item)}"
									:key="item.id">
									{{item.name}}
								</div>

							</div>
							<div v-for='groupKey in Object.keys(bigDataFiltered)' style='' :key='groupKey'>
								<div class="index-list__group-head"
									:id='"list-archor-"+ encodeURIComponent(groupKey).replace("%","_")'>
									{{groupKey}}
								</div>
								<div class="index-list__group-cell" v-for='city in dataFiltered[groupKey]'
									@click="()=>{itemClickHandler(city)}" :clickable='true' :key="city.id">
									{{city.name}}
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>

		<view v-if='useIndex&&dataFiltered&&Object.keys(dataFiltered).length>0' class='index-list__index'
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
				default: function() {
					return {}
				}
			},
			placeholder: {
				type: String,
				default: function() {
					return '输入关键字查询'
				}
			},
			useIndex: {
				type: Boolean,
				default: function() {
					return true
				}
			},
			isBigData: {
				type: Boolean,
				default: function() {
					return false
				}
			},
		},
		data() {
			return {
				dataFiltered: {},
				currentActiveIndex: '',
				isZoomActiveIndex: false,
				citySearch: '',
				citySeatchDebounce: null,
				bigDataFilteredLineHeightTotal: 0,
				bigDataFiltered: {},
				bigDataFilteredFlatWithLineInfo: [],
				bigDataFilteredFlatWithLineInfoCurrent: [],
				bigDataTableDataPanelEl: null,
				bigDataTableDataMarginSpaceEl: null,
				bigDataParam: {
					scrollHeight: 0,
					dataHeadHeight: 35, // 索引行高
					dataRowHeight: 45, // 数据行高
				},
				bigDataComputedParam: {
					dataRowTopMax: 0, //数据行滑到底，移动的最大距离
					dataRowTop: 0, //数据行当前移动的距离
				},

				bigDataScrollActionParam: {
					speed: 0,
					speedMax: 50,
					resistance: 0.5,
					prePositionY: null,
					preTimestamp: null,
					isMouseDown: false,
					timeoutObj: null
				}
			}
		},
		watch: {
			data: {
				immediate: true,
				handler() {
					this.buildDataFiltered()
				}
			},
			citySearch() {
				if(this.citySeatchDebounce){
					clearTimeout(this.citySeatchDebounce)
					this.citySeatchDebounce = null
				}
				this.citySeatchDebounce=setTimeout(()=>{
					this.buildDataFiltered()
				}, 200)
			},
			dataFiltered: {
				deep: true,
				handler() {
					if (this.isBigData) {
						this.bigDataComputedParam.dataRowTop = 0
						this.bigDataBuildData()
						this.bigDataUpdateRows()
					}
				}
			}
		},
		mounted() {
			if (this.isBigData) {
				this.bigDataTableDataPanelEl = this.$refs.bigDataTableDataPanel
				this.bigDataTableDataMarginSpaceEl = this.$refs.bigDataTableDataMarginSpace
				this.bigDataParam.scrollHeight = this.bigDataTableDataPanelEl.offsetHeight
				this.bigDataAddEvent()
				this.bigDataBuildData()
				this.bigDataUpdateRows()
				this.bigDataScrollAminate()
			}
		},
		beforeDestroy() {
			if (this.isBigData) {
				this.bigDataClearScrollAnimate()
				this.bigDataRemoveEvent()
			}
		},
		methods: {
			indexTouchMoveHandler(e) {
				this.isZoomActiveIndex = true
				const touch = e.touches[0];
				const target = e.target
				let currentIndex = Math.floor((touch.clientY + Object.keys(this.dataFiltered).length * 15 / 2 -
					target.offsetTop) / 15)
				currentIndex = Math.min(Object.keys(this.dataFiltered).length - 1, currentIndex)
				currentIndex = Math.max(0, currentIndex)
				if (Object.keys(this.dataFiltered)[currentIndex] !== this.currentActiveIndex) {
					this.currentActiveIndex = Object.keys(this.dataFiltered)[currentIndex]
					if (this.isBigData) {
						let currentStartLine = this.bigDataFilteredFlatWithLineInfo.find(item => item.id === 'Head:' + this
							.currentActiveIndex)
						this.bigDataComputedParam.dataRowTop = currentStartLine.__line_info__.totalHeight -
							currentStartLine
							.__line_info__.currentHeight
						this.bigDataUpdateRows()
					}
				}
				event.preventDefault()
				event.stopPropagation()
			},
			itemClickHandler(item) {
				let result = {
					...item
				}
				for (let key of Object.keys(result)) {
					if (key === '__line_info__') {
						delete result[key]
					}
				}

				this.$emit('select-item', result)
			},
			buildDataFiltered() {
				if (!this.data) {
					return {}
				}
				const resultObj = {}
				for (const groupkey of Object.keys(this.data)) {
					const cityListResult = this.data[groupkey].filter(item => item.name.toLowerCase().indexOf(this
						.citySearch.toLowerCase()) >= 0)
					if (cityListResult && cityListResult.length > 0) {
						resultObj[groupkey] = cityListResult
					}
				}
				this.dataFiltered = resultObj
			},
			bigDataAddEvent() {
				this.bigDataTableDataPanelEl.addEventListener('mousewheel', this.bigDataScrollEvent);
				this.bigDataTableDataPanelEl.addEventListener('touchmove', this.bigDataScrollEvent);
				this.bigDataTableDataPanelEl.addEventListener('touchstart', this.bigDataTouchStartEvent);
				this.bigDataTableDataPanelEl.addEventListener('touchend', this.bigDataTouchEndEvent);
			},
			bigDataRemoveEvent() {
				this.bigDataTableDataPanelEl.removeEventListener('mousewheel', this.bigDataScrollEvent);
				this.bigDataTableDataPanelEl.removeEventListener('touchmove', this.bigDataScrollEvent);
				this.bigDataTableDataPanelEl.removeEventListener('touchstart', this.bigDataTouchStartEvent);
				this.bigDataTableDataPanelEl.removeEventListener('touchend', this.bigDataTouchEndEvent);
			},
			bigDataTouchStartEvent(e) {
				this.bigDataScrollActionParam.isMouseDown = true
				this.bigDataScrollActionParam.speed = 0
			},
			bigDataTouchEndEvent(e) {
				this.bigDataScrollActionParam.isMouseDown = false
				this.bigDataScrollActionParam.prePositionY = null
			},
			bigDataScrollEvent(e) {
				let scrollEvent = e;
				if (scrollEvent instanceof WheelEvent) {
					this.bigDataComputedParam.dataRowTop += (scrollEvent.wheelDelta < 0 ? 1 : -1) * 30;
					if (this.bigDataComputedParam.dataRowTop <= 0) {
						this.bigDataComputedParam.dataRowTop = 0; //滑块最多滑到顶部
					} else if (this.bigDataComputedParam.dataRowTop >= this.bigDataComputedParam.dataRowTopMax) {
						this.bigDataComputedParam.dataRowTop = this.bigDataComputedParam.dataRowTopMax; //滑块最多滑到最底部
					} else {
						e.preventDefault(); //禁用滑轮滚动默认事件。当表格滑动到边缘时再启用默认滚动事件
					}
				} else if (scrollEvent instanceof TouchEvent) {
					if (this.bigDataScrollActionParam.isMouseDown) {
						let positionY = scrollEvent.changedTouches[0].screenY
						let timeStamp = scrollEvent.timeStamp
						let step = 0
						this.bigDataScrollActionParam.prePositionY = this.bigDataScrollActionParam.prePositionY ||
							positionY
						this.bigDataScrollActionParam.preTimeStamp = this.bigDataScrollActionParam.preTimeStamp ||
							timeStamp
						let timeDuration = timeStamp - this.bigDataScrollActionParam.preTimeStamp;
						let velocity;
						if (timeDuration === 0) {
							velocity = 0
						} else {
							velocity = (positionY - this.bigDataScrollActionParam.prePositionY) / timeDuration
						}
						if (this.bigDataScrollActionParam.prePositionY !== null) {
							step = positionY - this.bigDataScrollActionParam.prePositionY
							this.bigDataComputedParam.dataRowTop -= step
						}
						this.bigDataScrollActionParam.prePositionY = positionY
						this.bigDataScrollActionParam.preTimeStamp = timeStamp
						this.bigDataScrollActionParam.speed = velocity * 16
						event.preventDefault()
						event.stopPropagation()
					}
				}
				this.bigDataUpdateRows();
			},
			bigDataBuildData() {
				this.bigDataFilteredFlatWithLineInfo = []
				this.bigDataFilteredLineHeightTotal = 0
				for (let key of Object.keys(this.dataFiltered)) {
					if (this.dataFiltered[key] && this.dataFiltered[key].length > 0) {
						this.bigDataFilteredLineHeightTotal += this.bigDataParam.dataHeadHeight
						this.bigDataFilteredFlatWithLineInfo.push({
							id: "Head:" + key,
							name: key,
							__line_info__: {
								isEmptySpace: false,
								isHead: true,
								currentHeight: this.bigDataParam.dataHeadHeight,
								totalHeight: this.bigDataFilteredLineHeightTotal
							}
						})
						this.dataFiltered[key].forEach((item) => {
							this.bigDataFilteredLineHeightTotal += this.bigDataParam.dataRowHeight
							this.bigDataFilteredFlatWithLineInfo.push({
								id: "Row:" + item.id,
								name: item.name,
								__line_info__: {
									isEmptySpace: false,
									isHead: false,
									currentHeight: this.bigDataParam.dataRowHeight,
									totalHeight: this.bigDataFilteredLineHeightTotal
								}
							})
						})
					}
				}
				this.bigDataFilteredLineHeightTotal += this.bigDataParam.dataRowHeight
				this.bigDataFilteredFlatWithLineInfo.push({
					id: "EmptyFooter",
					name: '',
					__line_info__: {
						isEmptySpace: true,
						isHead: false,
						currentHeight: this.bigDataParam.dataRowHeight,
						totalHeight: this.bigDataFilteredLineHeightTotal
					}
				})
				//计算出数据区域滑动到最后一条数据，所需要的位移距离
				this.bigDataComputedParam.dataRowTopMax = Math.max(this.bigDataFilteredLineHeightTotal - this.bigDataParam
					.scrollHeight, 0)
			},
			bigDataUpdateRows() {
				this.bigDataComputedParam.dataRowTop = Math.max(0, this.bigDataComputedParam.dataRowTop)
				this.bigDataComputedParam.dataRowTop = Math.min(this.bigDataComputedParam.dataRowTop, this
					.bigDataComputedParam.dataRowTopMax)
				if (this.bigDataFilteredFlatWithLineInfo.length > 0) {
					//更新数据区的数据，根据滚动的距离，确定可见的第一条数据下标（此方法保证可见数据行内容的更新。注释掉此方法，拖动滚动条，数据会在开始部分一直循环）
					//遍历数据，插入窗口中可见的数据行
					let startIndex = null
					let endIndex = null

					for (let i = parseInt(this.bigDataComputedParam.dataRowTop / Math.max(this.bigDataParam.dataRowHeight,
							this.bigDataParam.dataHeadHeight), 10) - 1; i < this.bigDataFilteredFlatWithLineInfo
						.length; i++) {
						if (i < 0) {
							continue
						}
						const totalHeight = this.bigDataFilteredFlatWithLineInfo[i].__line_info__.totalHeight
						if (totalHeight >= this.bigDataComputedParam.dataRowTop && startIndex === null) {
							startIndex = i
						}
						if (totalHeight >= (this.bigDataComputedParam.dataRowTop + this.bigDataParam.scrollHeight) &&
							endIndex === null) {
							endIndex = Math.min(i + 3, this.bigDataFilteredFlatWithLineInfo.length)
						}
						if (startIndex !== null && endIndex !== null) {
							break
						}
					}
					startIndex = startIndex || 0
					endIndex = endIndex || 0
					this.bigDataFilteredFlatWithLineInfoCurrent = this.bigDataFilteredFlatWithLineInfo.slice(startIndex,
						endIndex)
					if (this.bigDataFilteredFlatWithLineInfoCurrent && this.bigDataFilteredFlatWithLineInfoCurrent[0]) {
						const tmp = this.bigDataFilteredFlatWithLineInfoCurrent[0]
						let firstRowHeight = tmp.__line_info__.currentHeight || 0
						let firstRowTotalHeight = tmp.__line_info__.totalHeight || 0
						//数据区实现拖动滑条的位移效果（保证数据行的滚动效果，注释此方法，拖动滚动条，数据无滚动效果，但内容会更新）
						let newMarginTop = firstRowTotalHeight - this.bigDataComputedParam.dataRowTop - firstRowHeight
						this.bigDataTableDataMarginSpaceEl.style.marginTop = newMarginTop + "px"
					}
				}
			},
			bigDataScrollAminate() {
				this.bigDataScrollActionParam.timeoutObj = setInterval(() => {
					if (Math.abs(this.bigDataScrollActionParam.speed) > 0 && !this.bigDataScrollActionParam
						.isMouseDown) {
						this.bigDataScrollActionParam.speed = (this.bigDataScrollActionParam.speed >= 0 ? 1 : -1) *
							Math.min(this.bigDataScrollActionParam.speedMax, Math.max(0, Math.abs(this
									.bigDataScrollActionParam.speed) - this.bigDataScrollActionParam
								.resistance))
						this.bigDataComputedParam.dataRowTop -= this.bigDataScrollActionParam.speed
						this.bigDataUpdateRows()
					}
				}, 16)
			},
			bigDataClearScrollAnimate() {
				if (this.bigDataScrollActionParam.timeoutObj) {
					clearInterval(this.bigDataScrollActionParam.timeoutObj)
					this.bigDataScrollActionParam.timeoutObj = null
				}
			}
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
			background-color: white;
			overflow-x: hidden;
			overflow-y: auto;
		}

		&__big-data {

			&__table-panel {
				height: 100%;
				width: 100%;
			}

			&__table-body {
				/*因为数据区与滚动条要使用position:absolute做成左右固定布局，table-body作为其两节点的父节点，使用position: relative*/
				position: relative;
				height: 100%;
				overflow: hidden;
				background-color: white;
			}

			&__table-data-panel {
				overflow: hidden;
				position: absolute;
				left: 0;
				/*right值宽度等于滚动条的宽度。也可不设置，但滚动条会覆盖表格的最右侧*/
				/*right: 15px;*/
				width: 100%;
				height: 100%;
			}
		}

		&__group-head {
			box-sizing: border-box;
			background-color: $uni-bg-color-grey;
			color: $uni-text-color-grey;
			height: 35px;
			line-height: 45px;
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

			&--empty-space {
				&:active {
					background-color: white;
				}
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
