<template>
	<div class="container">
		<div class="title-container">
			<span class="title-text">FIFA World Cup™</span>
		</div>
		<div v-if="rawData" id="item-container">
			<div id="filter-chart-container">
				<div id="filter-container">
					<div class="tip-texts">Chart Data:</div>
					<div id="selected-y-axis">
						<v-select
							v-model="selectedYAxis"
							:items="Object.keys(yAxisItems)"
							item-title="value"
							item-value="key"
						></v-select>
					</div>
					<div style="width: 50px"></div>
					<div class="tip-texts">Time Period:</div>
					<div class="year-text-field">
						<v-text-field
							v-model="startYear"
							type="number"
							min="1930"
							max="2014"
						></v-text-field>
					</div>
					<div id="line-container">
						<hr class="full-width-line" />
					</div>
					<div class="year-text-field">
						<v-text-field
							v-model="endYear"
							type="number"
							min="1930"
							max="2014"
						></v-text-field>
					</div>
					<div class="button-container">
						<button @click="filterData">Filter</button>
					</div>
				</div>
				<div ref="chart" id="chart"></div>
			</div>
			<div id="info-table">
				<table v-if="currentData">
					<tr>
						<th colspan="2" class="table-title">
							{{ currentData[0].edition }}
						</th>
					</tr>
					<tr>
						<th>Winner</th>
						<td>{{ currentData[0].winner }}</td>
					</tr>
					<tr>
						<th>Teams</th>
						<td>{{ currentData[0].teams }}</td>
					</tr>
					<tr>
						<th>Matches</th>
						<td>{{ currentData[0].matches }}</td>
					</tr>
					<tr>
						<th>Goals</th>
						<td>{{ currentData[0].goals }}</td>
					</tr>
					<tr>
						<th>Average Goals</th>
						<td>{{ currentData[0].averageGoals }}</td>
					</tr>
					<tr>
						<th>Average Attendance</th>
						<td>{{ currentData[0].averageAttendance }}</td>
					</tr>
				</table>
			</div>
		</div>
	</div>
	<div id="tooltip" class="hidden">
		<p><span id="tooltip-edition"></span></p>
		<p><span id="tooltip-value"></span></p>
	</div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3";
import { select, scaleTime, scaleLinear, axisBottom, axisLeft, line } from "d3";

const rawData = ref([]); // 储存原始数据
const filteredData = ref([]); // 储存被筛选的数据
const startYear = ref(1930);
const endYear = ref(2014);
const selectedYAxis = ref("Goals"); // 筛选y轴数据类型
const currentData = ref(null);
// 画图表的容器
const chart = ref(null);
const width = 800;
const height = 500;
const margin = { top: 20, right: 30, bottom: 30, left: 40 };

const yAxisItems = {
	Goals: "goals",
	"Average Goals": "averageGoals",
	"Average Attendance": "averageAttendance",
};
// 读取CSV数据
async function loadData() {
	const data = await d3.csv("/fifa-world-cup.csv", d3.autoType);
	return data;
}

onMounted(async () => {
	const data = await loadData();

	// 转换数据格式
	rawData.value = data.map((d) => ({
		edition: d.EDITION,
		year: d.YEAR,
		location: d.LOCATION,
		winner: d.WINNER,
		teams: d.TEAMS,
		matches: d.MATCHES,
		goals: d.GOALS,
		averageGoals: d.AVERAGE_GOALS,
		averageAttendance: d.AVERAGE_ATTENDANCE,
	}));

	// 用于绑定到图表的筛选后数据
	filteredData.value = [...rawData.value];
	filterData();
	createChart();
});

// 筛选数据的方法
function filterData() {
	const parseDate = d3.utcParse("%Y"); // 使用UTC解析器，因为它假设输入是UTC时间
	filteredData.value = rawData.value
		.filter((d) => d.year >= startYear.value && d.year <= endYear.value)
		.map((d) => ({
			date: parseDate(d.year),
			value: d[yAxisItems[selectedYAxis.value]],
			edition: d.edition,
		}));

	const tooltip = d3.select("#tooltip");
	tooltip.classed("hidden", true);
	updateChart();
}

function createChart() {

	const svg = select(chart.value)
		.append("svg")
		.attr("width", width)
		.attr("height", height)
		.attr("display", "block")
		.attr("margin-left", margin.left);

	const minDate = d3.min(filteredData.value, (d) => d.date);
	const maxDate = d3.max(filteredData.value, (d) => d.date);

	// 调整起始和结束日期，确保包括1930年和2014年
	const adjustedMinDate = new Date(minDate.getFullYear(), 0, 1);
	const adjustedMaxDate = new Date(maxDate.getFullYear() + 8, 0, 1);

	const xWithTicks = d3.timeYear.range(adjustedMinDate, maxDate, 4);

	// 使用转换后的数据创建比例尺
	const x = scaleTime()
		.domain([minDate, adjustedMaxDate])
		.range([margin.left, width - margin.right]);

	// 更新y轴域
	const yDomain = [0, d3.max(filteredData.value, (d) => d.value)];
	const y = scaleLinear()
		.domain(yDomain)
		.nice()
		.range([height - margin.bottom, margin.top]);

	const lineGenerator = line()
		.defined((d) => !isNaN(d.value))
		.x((d) => x(d.date))
		.y((d) => y(d.value));

	drawChart(svg, x, y, xWithTicks, lineGenerator);
}

function drawChart(svg, x, y, xWithTicks, lineGenerator) {
	const xAxis = (g) =>
		g.attr("transform", `translate(0,${height - margin.bottom})`).call(
			axisBottom(x)
				.tickValues(xWithTicks) // 使用 xWithTicks 来设置刻度
				.tickSizeOuter(0)
				.tickFormat(d3.timeFormat("%Y")) // 使用四位数年份格式
		);

	const yAxis = (g) =>
		g
			.attr("transform", `translate(${margin.left},0)`)
			.call(axisLeft(y))
			.call((g) => g.select(".domain"));

	svg.selectAll("g.x-axis")
		.data([null])
		.join(
			(enter) => enter.append("g").attr("class", "x-axis").call(xAxis),
			(update) => update.call(xAxis),
			(exit) => exit.remove()
		);

	svg.selectAll("g.y-axis")
		.data([null])
		.join(
			(enter) => enter.append("g").attr("class", "y-axis").call(yAxis),
			(update) => update.call(yAxis),
			(exit) => exit.remove()
		);

	svg.selectAll("path.line")
		.data([filteredData.value], (d) => d[0].date)
		.join(
			(enter) =>
				enter
					.append("path")
					.attr("class", "line")
					.attr("fill", "none")
					.attr("stroke", "#82c2a3")
					.attr("stroke-width", 3.5)
					.attr("d", lineGenerator),
			(update) => update.transition().attr("d", lineGenerator),
			(exit) => exit.remove()
		);

	// 添加圆点和点击事件
	svg.selectAll(".dot")
		.data(filteredData.value)
		.join(
			(enter) =>
				enter
					.append("circle")
					.attr("class", "dot")
					.attr("r", 4)
					.attr("cx", (d) => x(d.date))
					.attr("cy", (d) => y(d.value))
					.attr("fill", "#456c59")
					.attr("cursor", "pointer") /* 设置鼠标指针样式为可点击 */
					.on("mouseover", (event, d) => {
						const tooltip = d3.select("#tooltip");
						tooltip.classed("hidden", false);
						tooltip
							.style("left", event.pageX - 122 + "px")
							.style("top", event.pageY + 20 + "px");
						d3.select("#tooltip-edition").text(d.edition);
						d3.select("#tooltip-value").text(
							`${selectedYAxis.value}: ${d.value}`
						);
					})
					.on("mouseout", (event, d) => {
						// 当鼠标离开时，隐藏tooltip
						d3.select("#tooltip").classed("hidden", true);
					})
					.on("click", (event, d) => {
						currentData.value = rawData.value.filter(
							(data) => data.edition === d.edition
						);
					}),
			(update) =>
				update
					.attr("cx", (d) => x(d.date))
					.attr("cy", (d) => y(d.value)),
			(exit) => exit.remove()
		)
		.raise(); // 这将确保圆点始终在折线上方;;

	// 添加图例
	svg.append("g")
		.attr("transform", `translate(${width - 45}, ${height - 40})`)
		.append("text")
		.attr("font-size", "15px")
		.text("Year");

	// 选择当前存在的图例元素，如果没有则返回空选择
	const legend = svg
		.selectAll(".legend-group")
		.data([selectedYAxis.value], (d) => d); // 重新绑定数据，使用一个键函数确保唯一性

	// 处理退出（exit）的元素，移除它们
	legend.exit().remove();

	// 更新或添加新的图例元素
	legend
		.enter()
		.append("g")
		.attr("class", "legend-group")
		.merge(legend) // 合并进入和更新选择
		.attr("transform", `translate(${40}, ${15})`)
		.append("text")
		.attr("font-size", "15px")
		.text(`${selectedYAxis.value}`);
}

function updateChart() {
	const svg = select(chart.value).select("svg");

	const minDate = d3.min(filteredData.value, (d) => d.date);
	const maxDate = d3.max(filteredData.value, (d) => d.date);

	// 调整起始和结束日期，确保包括1930年和2014年
	const adjustedMinDate = new Date(minDate.getFullYear(), 0, 1);
	const adjustedMaxDate = new Date(maxDate.getFullYear() + 8, 0, 1);

	const xWithTicks = d3.timeYear.range(adjustedMinDate, maxDate, 4);

	// 使用转换后的数据创建比例尺
	const x = scaleTime()
		.domain([minDate, adjustedMaxDate])
		.range([margin.left, width - margin.right]);

	// 更新y轴域
	const yDomain = [0, d3.max(filteredData.value, (d) => d.value)];
	const y = scaleLinear()
		.domain(yDomain)
		.nice()
		.range([height - margin.bottom, margin.top]);

	const lineGenerator = line()
		.defined((d) => !isNaN(d.value))
		.x((d) => x(d.date))
		.y((d) => y(d.value));

	drawChart(svg, x, y, xWithTicks, lineGenerator);
}
</script>

<style scoped>
.container {
	width: 1600px;
	display: flex;
	flex-direction: column;
	justify-content: center; /* 水平居中 */
	margin: auto;
}
.title-container {
	text-align: start;
	font-size: 40px;
	margin-bottom: 40px;
	margin-left: 20px;
}

.title-text {
	display: inline-block;
	/* background-color: #82c2a3; */
	color: #456c59;
	padding: 0 5px;
}

#item-container {
	display: flex;
}
#filter-chart-container {
	display: flex;
	flex-direction: column;
}

#filter-container {
	display: flex;
	align-items: center; /* 水平居中 */
	margin: 10px 40px;
}

.tip-texts {
	margin-right: 15px;
}
.hidden {
	display: none;
}

#tooltip {
	position: absolute;
	padding: 10px;
	background: lightgray;
	border: 1px solid gray;
	border-radius: 4px;
	pointer-events: none;
	z-index: 10;
	width: 300;
}

#tooltip::before {
	content: "";
	position: absolute;
	top: -10px;
	left: 50%;
	margin-left: -5px;
	border-width: 5px;
	border-style: solid;
	border-color: transparent transparent lightgray transparent;
}

#line-container {
	width: 20px;
	padding: 0;
	margin: 12px;
}
.full-width-line {
	width: 100%;
	border: none;
	border-top: 1px solid black;
	margin: 0;
}

.year-text-field {
	width: 100px;
}

#chart {
	margin: 20px;
	margin-left: auto;
}

.button-container {
	height: 100%;
	width: 100px;
}

#selected-y-axis {
	width: 220px;
}

#info-table {
	margin-left: 50px;
}
button {
	margin: 0 40px;
	width: 100%;
	height: 100%;
	display: block;
	padding: 0 20px;
	font-size: 16px;
	text-align: center;
	text-decoration: none;
	background-color: hsla(145, 54%, 78%, 0.527);
	color: #0c0c0c;
	border: none;
	border-radius: 10px;
	cursor: pointer;
	transition: background-color 0.3s, box-shadow 0.3s;
}

/* 悬停效果 */
button:hover {
	background-color: #ade8c6;
	box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}
tr {
	height: 50px;
}
th,
td {
	width: 220px;
}
</style>
