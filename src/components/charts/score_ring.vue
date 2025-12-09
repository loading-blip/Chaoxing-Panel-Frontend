<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { ElMessageBox } from 'element-plus'

const chartRef = ref(null)
let chart = null
const FULL_SCORE = ref(60)

const current_work =  ref("")
const is_ready = ref(true)

async function initChart() {
	const resp = await fetch('http://localhost:5000/data')
	const json = JSON.parse(await resp.json())['data']
	const typeData = json.type_data || []
	const records = json.records || []
	const categories = typeData.map(td => {
		const name = td.name
		const children = records
			.filter(r => r.creditName === name)
			.map(r => ({ name: r.activityName, value: Number(r.score) || 0 }))

		const childrenSum = children.reduce((s, c) => s + (c.value || 0), 0)
		const value = td.totalIntegral != null ? Number(td.totalIntegral) : childrenSum

		return {
			name,
			value,
			children: children.length ? children : undefined,
			itemStyle: { borderWidth: 1, borderColor: '#fff' }
		}
	})

	let obtained = categories.reduce((s, c) => s + (c.value || 0), 0)

	const rootChildren = categories.map(c => (c.children ? { name: c.name, value: c.value, children: c.children } : { name: c.name, value: c.value }))

	if (obtained < FULL_SCORE.value) {
		rootChildren.push({ name: '未获得', value: FULL_SCORE.value - obtained, itemStyle: { color: '#f0f0f0' } })
	}

	const rootData = [{ name: '总分：' + FULL_SCORE.value, value: FULL_SCORE.value, children: rootChildren }]

	const echarts = await import('echarts')
	chart = echarts.init(chartRef.value)

	const truncate = (s, n = 12) => (s && s.length > n ? s.slice(0, n) + '…' : s)

	const option = {
		tooltip: {
			trigger: 'item',
			formatter: params => {
				// show full name and value in tooltip
				return `${params.data.name}: ${params.data.value}`
			}
		},
		series: [
			{
				type: 'sunburst',
				startAngle: 180,
				radius: [0, '90%'],
				data: rootData,
				sort: null,
				label: {
					// use tangential rotation so labels follow the ring direction
					rotate: 'tangential',
					color: '#333',
					formatter: params => {
						// truncate long labels to avoid overlap
						return truncate(params.data.name, 5)
					}
				},
				emphasis: {
					label: {
						// show full label when emphasized (hover)
						formatter: params => params.data.name,
						color: '#000'
					}
				},
				itemStyle: {
					borderWidth: 1,
					borderColor: '#fff'
				}
			}
		]
	}

	chart.setOption(option)

	const ro = new ResizeObserver(() => chart && chart.resize())
	ro.observe(chartRef.value)
	chartRef.value._ro = ro
}

async function get_backend_status() {
const api = "http://localhost:5000/status"
    try{
        const response = await fetch(api,{
            method: "GET",
            headers: {"Content-Type": "application/json"},
        })
        return response.json()
    }catch{
        if (error.name === 'AbortError') {
            console.error('请求超时');
        }
        throw error;
    }
}

onMounted(async () => {
	
	const status_json = JSON.parse(await get_backend_status())
	if (status_json["complete"] && status_json["session"]==="transcript"){
		initChart().catch(err => console.error('chart init failed', err))
		return
	}
	if (status_json["session"] !== "" && (status_json["status"] === "working" && status_json["session"] !== "transcript")){
		ElMessageBox.alert('后端正在执行其他任务，请等待其完成','提示',{confirmButtonText:'确定'})
	}
	await fetch('http://localhost:5000/service',
		{	method: 'POST',
			body:'{"use":"transcript"}'
		}
	)
	const backend = setInterval(() => {
		get_backend_status().then((status_json)=>{
			status_json = JSON.parse(status_json)
			is_ready.value = status_json["complete"]
			current_work.value = status_json["current_work"]
		})
		
		

		if (is_ready.value){
			clearInterval(backend)
			initChart().catch(err => console.error('chart init failed', err))
		}
	}, 500);

	is_ready.value = false

})

onBeforeUnmount(() => {
	if (chart) chart.dispose()
	if (chartRef.value && chartRef.value._ro) chartRef.value._ro.disconnect()
})
</script>

<template>
	<div ref="chartRef" class="chart" v-loading="!is_ready && {text:`${current_work}...`}"></div>
</template>

<style scoped>
.chart {
	width: 100%;
	height: 600px;
}
[hidden]{
	display: none;
}
</style>

