<template>
	<div class="item-container">
		<div class="title-container">
			<span class="title-text">The World's Tallest Buildings</span>
		</div>
		<div id="buildings-container">
			<div ref="chart"></div>
			<div id="infoPanel">
				<img :src="buildingImage" />
				<table id="building-info" v-if="currentBuilding">
					<tr>
						<th colspan="2" class="table-title">
							{{ currentBuilding.building }}
						</th>
					</tr>
					<tr>
						<th>Height</th>
						<td>{{ currentBuilding.height_m }}</td>
					</tr>
					<tr>
						<th>City</th>
						<td>{{ currentBuilding.city }}</td>
					</tr>
					<tr>
						<th>Country</th>
						<td>{{ currentBuilding.country }}</td>
					</tr>
					<tr>
						<th>Floors</th>
						<td>{{ currentBuilding.floors }}</td>
					</tr>
					<tr>
						<th>Completed</th>
						<td>{{ currentBuilding.completed }}</td>
					</tr>
				</table>
			</div>
		</div>
	</div>
</template>

<script setup>
import * as d3 from "d3";
import { onMounted, ref } from "vue";

const chart = ref(null);

const width = 600;
const height = 500;

const buildingImage = ref(null);
const currentBuilding = ref(null);

onMounted(() => {
	const svg = d3
		.select(chart.value)
		.append("svg")
		.attr("width", width)
		.attr("height", height);

	d3.csv("/buildings/buildings.csv").then((data) => {
		// 将 height_m 列的值转换为数字
		data.forEach((d) => {
			d.height_m = +d.height_m; // 使用 + 运算符将字符串转换为数字
			d.height_px = +d.height_px;
		});
		// 按高度排序
		data.sort((a, b) => b.height_m - a.height_m);

		// 设置默认展示的图片和数据是第一高的
		currentBuilding.value = data[0];
		buildingImage.value = `buildings/img/${currentBuilding.value.image}`;

		// 绑定数据到矩形
		svg.selectAll(".bar")
			.data(data)
			.enter()
			.append("rect")
			.attr("class", "bar")
			.attr("y", (d, i) => i * 50 + 10) // 每个矩形宽度为50，间隔为10
			.attr("x", (d) => 260)
			.attr("width", (d) => d.height_px) // 使用宽度减去宽度像素值
			.attr("height", 20) // 设置固定高度
			.attr("fill", "steelblue")
			.attr("cursor", "pointer") /* 设置鼠标指针样式为可点击 */
			.on("click", (d) => {
				// 更新数据
				currentBuilding.value = d.currentTarget.__data__;
				buildingImage.value = `buildings/img/${currentBuilding.value.image}`;
			});

		// 添加建筑物名称标签
		svg.selectAll(".label")
			.data(data)
			.enter()
			.append("text")
			.attr("class", "label")
			.attr("y", (d, i) => i * 50 + 25) // 向下移动
			.attr("x", 260 - 10) //
			.attr("text-anchor", "end")
			.text((d) => d.building);

		// 添加建筑物高度标签
		svg.selectAll(".height")
			.data(data)
			.enter()
			.append("text")
			.attr("class", "height")
			.attr("y", (d, i) => i * 50 + 25) // 向下移动
			.attr("x", (d) => d.height_px + 260 - 45) // 放置在矩形右侧
			.attr("fill", "#fff")
			.attr("font-size", "14px")
			.text((d) => `${d.height_m}m`);
	});
});
</script>

<style scoped>
.item-container {
	width: 1600px;
	margin: 2rem auto;
}
.title-container {
	text-align: start;
	font-size: 40px;
	margin-bottom: 40px;
}

.title-text {
	display: inline-block;
	/* background-color: #bde3e8; */
	padding: 0 5px; /* 可选，添加一些内边距使背景颜色更明显 */
	color: #3f5c6d;
}
#buildings-container {
	display: flex;
	width: 1600px;
	height: 500px;
	gap: 60px;
}

#infoPanel {
	display: flex;
	padding-bottom: 20px;
	padding-top: 10px;
	gap: 100px;
}

#building-info {
	height: 300px;
}
</style>
