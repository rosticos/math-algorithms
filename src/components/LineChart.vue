<script>
import { Line } from 'vue-chartjs'
import algebra from 'algebra.js'
import { evaluate } from 'mathjs'
export default {
  extends: Line,
  props: ['func'],
  data () {
    return {
      line: this.func
    }
  },
  mounted () {
    console.log(this.func)
    this.addPlugin({
      beforeDraw: (chartInstance) => {
        const ctx = chartInstance.chart.ctx
        ctx.fillStyle = '#eee'
        ctx.fillRect(0, 0, chartInstance.chart.width, chartInstance.chart.height)
      },
      beforeInit: (chart) => {
        const data = chart.config.data
        for (let i = 0; i < data.datasets.length; i++) {
          for (let j = 0; j < data.labels.length; j++) {
            const fct = data.datasets[i].function
            const x = data.labels[j]
            const y = fct(x)
            data.datasets[i].data.push(y)
          }
        }
      }
    })
    this.renderChart({
      type: 'line',
      labels: [-5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5],
      scales: {
        yAxes: [{
          ticks: {
            max: 10,
            min: 0,
            stepSize: 5
          }
        }]
      },
      datasets: [{
        label: `f(x) = ${this.func}`, // Name it as you want
        function: (x) => {
          const expr = algebra.parse(this.func)
          const val = expr.eval({ x }).simplify().toString()
          return evaluate(val)
        },
        data: [], // Don't forget to add an empty data array, or else it will break
        borderColor: '#FC2525',
        backgroundColor: '#FFF',
        fill: false
      }],
      chartArea: {
        backgroundColor: 'rgba(251, 85, 85, 0.4)'
      },
      responsive: true
    })
  }
}
</script>
