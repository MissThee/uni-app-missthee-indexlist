<template>
	<missthee-indexlist @select-item='selectHandler' :listData="listData"></missthee-indexlist>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				listData: {
					"A": [{
						"id": 150,
						"name": "淮北市",
					}, ],

					"B": [{
							"id": 35, //城市id
							"name": "北京市", //城市名称

						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						},
						{
							"id": 369,
							"name": "北海市",
						}
					],
					"C": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"D": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"E": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"F": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"G": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"H": [{
						"id": 150,
						"name": "淮北市",
						"pid": 13,
						"level": 2,
						"initials": "H"
					}, ],
					"I": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"J": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"K": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"L": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"M": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"N": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"O": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"P": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"Q": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"R": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"S": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"T": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"U": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"V": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"W": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"X": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"Y": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"Z": [{
						"id": 150,
						"name": "淮北市",
					}, ],
					"#": [{
						"id": 150,
						"name": "淮北市",
					}, ],
				}
			}
		},
		onLoad() {

		},
		methods: {
			selectHandler(e) {
				console.log('阿斯顿发斯蒂芬', e)
			}
		}
	}
</script>

<style>

</style>
