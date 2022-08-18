<template>
  <div>
    <canvas ref="canvasRef" />
  </div>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue'
import { Chart } from 'chart.js'
import chartjsPluginAnnotation from 'chartjs-plugin-annotation'

export default defineComponent({
  props: ['calculationResult', 'target', 'calcYears'],
  date () {
    return {myChart: null}
  },
  setup (props) {
    const canvasRef = ref(null)
    // ラベル生成器
    function getlabels () {
      const arr = []
      for (let i = 0; i < parseInt(props.calcYears); i++) {
        arr.push(i + '年目')
      }
      return arr
    }
    function createCharts() {
      if (canvasRef.value === null) return
      const canvas = canvasRef.value.getContext('2d')
      if (canvas === null) return
      if(this.myChart) {
        this.myChart.destroy();
      }
      this.myChart = new Chart(canvas, {
        type: 'bar',
        data: {
          labels: getlabels(),
          datasets: [{
            label: 'シミュレーション結果',
            data: props.calculationResult
          }]
        },
        options: {
          scales: {
          yAxes: [{
              ticks: {
                beginAtZero: true,
                min: 0,
              },
              id: "y-axis-1",
              type: "linear",
            }
          ]},
          annotation: {
            drawTime: 'afterDatasetsDraw',
            annotations: [
                {
                    id: 'hline',
                    type: 'line',
                    mode: 'horizontal',
                    scaleID: 'y-axis-1',
                    value: props.target,
                    borderColor: 'black',
                    borderWidth: 2,
                    label: {
                        backgroundColor: "green",
                        // content: "目標金額",
                        enabled: true
                    },
                },
            ]
          }
        }
      })
    }
    return {
      canvasRef,
      createCharts
    }
  },
})
</script>
