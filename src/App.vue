<template>
	<div class="container">
		<h1>《数据可视化》第五次个人作业</h1>
		<tallest-buildings />
		<div id="line-container">
			<hr id="line" />
		</div>
		<world-cup />
	</div>
</template>

<script setup>
import TallestBuildings from "./components/TallestBuildings.vue";
import WorldCup from "./components/WorldCup.vue";
</script>

<style scoped>
.container {
	display: flex;
	flex-direction: column;
	gap: 40px;
}

#line-container {
	padding: 0;
	margin-top: 60px;
	margin-bottom: 60px;
}

#line {
	border: none; /* 移除默认边框 */
	border-top: 2px dashed #000; /* 设置顶部边框为2px虚线 */
	margin-left: 160px; /* 左边距 */
	margin-right: 160px; /* 右边距 */
}
</style>
