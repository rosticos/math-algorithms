<script>
import { Line } from 'vue-chartjs'
import algebra from 'algebra.js'
import { evaluate } from 'mathjs'
import chartjsPluginAnnotation from 'chartjs-plugin-annotation' // eslint-disable-line

const countDecimals = (value) => {
  console.log('Comes', value.toString())
  if (Math.floor(value) !== value)
    return value.toString().split('.')[1].length || 0
  return 0
}

export default {
  extends: Line,
  props: ['func', 'labels'],
  data () {
    return {
      line: this.func
    }
  },
  mounted () {
    this.addPlugin({
      beforeDraw: (chartInstance) => {
        const ctx = chartInstance.chart.ctx
        // ctx.fillStyle = '#eee'
        ctx.fillRect(0, 0, chartInstance.chart.width, chartInstance.chart.height)
      },
      beforeInit: (chart) => {
        const data = chart.config.data
        for (let i = 0; i < 1; i++) {
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
      labels: this.labelValues,
      datasets: [{
        data: [],
        label: `F(x) = ${this.func}`,
        function: (x) => {
          let afterComma = 0
          if (Number(x) === x && x % 1 !== 0) {
            if (x.toString().indexOf(',') !== -1) {
              // console.log('COmmal!')
              afterComma = countDecimals(x.replace(',', '.'))
            } else {
              // console.log('Without COmma', x)
              afterComma = countDecimals(x)
            }
          }
          const expr = algebra.parse(this.func)
          // console.log('KEK', countDecimals(x))
          const pointFraction = new algebra.Fraction(x * Math.pow(10, afterComma), Math.pow(10, afterComma))
          const val = expr.eval({ x: pointFraction }).simplify().toString()
          console.log(val)
          return evaluate(val)
        },
        borderColor: '#FC2525',
        backgroundColor: '#FFF',
        fill: false
      }]
    },
    {
      responsive: true,
      tooltips: {
        mode: 'index',
        intersect: true
      },
      annotation: {
        annotations: [
          {
            type: 'line',
            mode: 'horizontal',
            scaleID: 'y-axis-0',
            value: 0,
            borderColor: 'rgb(75, 192, 192)',
            borderWidth: 1
          },
          {
            type: 'line',
            mode: 'vertical',
            scaleID: 'x-axis-0',
            value: 0,
            borderColor: 'rgb(75, 192, 192)',
            borderWidth: 1
          }
        ]
      }
    })
  },
  methods: {

  },
  computed: {
    labelValues () {
      let arr = this.labels.map(el => parseFloat(el.valueX))
      arr.push(Math.max(...arr) + 2)
      arr.unshift(Math.min(...arr) - 2)
      arr = arr.sort((a, b) => a - b)

      console.log(arr)
      return arr
    },
    dataValues () {
      return this.labels.map(el => parseFloat(el.valueX))
    }
  }
}
</script>
