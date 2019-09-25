<template>
  <div>
    <div v-for="(field, index) in fields" :key="index">
      <label>'{{ index }}'x) </label>
      <input type="text" v-model.number="field.valueX">
      <label>'{{ index }}'y) </label>
      <input type="text" v-model.number="field.valueY">
    </div>
    <div>
      <button @click="addField" v-if="fields.length < 10">Add field</button>
      <button @click="removeField" v-if="fields.length">Remove field</button>
      <button @click="countResultsL">Count</button>
      <button @click="paintChart">Paint</button>
    </div>
    <p v-if="resultsL.length">Начальные уравнения:</p>
    <div v-if="resultsL.length">
      <p v-for="(l, index) in resultsL" :key="`L-${index}`">L{{ index }} = {{ l.numerator }} / {{ l.denominator }}</p>
    </div>
    <div v-if="countedResultL.length">
      {{ countedResultL }}
    </div>
    <br>
    <div v-if="answerL.length">
      {{ answerL }}
    </div>
    <br>
    <div>
      {{ grapg }}
    </div>
    <line-chart
      v-if="renderChart"
      :func="grapg"
      style="max-width: 500px; max-height: 500px;"
      ref="line-chart"
    />
  </div>
</template>

<script>
import algebra from 'algebra.js'
import LineChart from './LineChart'
export default {
  components: { LineChart },
  data () {
    return {
      /* eslint-disable */
      countedResultL : [],
      /* eslint-enable */
      fields: [
        { labelX: 'X', valueX: '-1', labelY: 'Y', valueY: '4' },
        { labelX: 'X', valueX: '1', labelY: 'Y', valueY: '1' },
        { labelX: 'X', valueX: '3', labelY: 'Y', valueY: '-1' },
        { labelX: 'X', valueX: '5', labelY: 'Y', valueY: '2' }
      ],
      resultsL: [],
      answerL: [],
      grapg: '',
      renderChart: false
    }
  },
  methods: {
    addField () {
      this.fields.push(
        { labelX: 'X', valueX: '', labelY: 'Y', valueY: '' }
      )
    },
    removeField () {
      this.fields.pop()
    },
    countResultsL () {
      this.resultsL = []
      this.countedResultL = []
      this.answerL = []
      this.grapg = ''
      this.renderChart = false
      for (let i = 0; i < this.fields.length; i++) {
        let numerator = []
        let denominator = []
        for (let j = 0; j < this.fields.length; j++) {
          if (j !== i) {
            if (this.fields[j].valueX < 0) {
              numerator.push(`(x + ${Math.abs(parseInt(this.fields[j].valueX))})`)
              denominator.push(`(${parseInt(this.fields[i].valueX)} + ${Math.abs(parseInt(this.fields[j].valueX))})`)
            } else {
              numerator.push(`(x - ${parseInt(this.fields[j].valueX)})`)
              denominator.push(`(${parseInt(this.fields[i].valueX)} - ${Math.abs(parseInt(this.fields[j].valueX))})`)
            }
          }
        }
        this.resultsL.push({ numerator, denominator })
      }
      console.log(this.resultsL)
      this.resultsL.map(el => {
        let exprNumer = algebra.parse(el.numerator[0])
        let exprDenom = algebra.parse(el.denominator[0])
        for (let i = 1; i < el.numerator.length; i++) {
          exprNumer = exprNumer.multiply(algebra.parse(el.numerator[i]))
          exprDenom = exprDenom.multiply(algebra.parse(el.denominator[i]))
        }
        this.countedResultL.push({
          'numerator': exprNumer.toString(),
          'denominator': exprDenom.toString()
        })
      })
      for (let i = 0; i < this.countedResultL.length; i++) {
        const expr = algebra.parse(`(${this.fields[i].valueY} * 1 / (${this.countedResultL[i].denominator}))(${this.countedResultL[i].numerator})`)
        this.answerL.push(expr.toString())
      }
      let graphExp = algebra.parse(this.answerL[0])
      for (let i = 1; i < this.answerL.length; i++) {
        graphExp = graphExp.add(algebra.parse(this.answerL[i]))
      }
      this.grapg = graphExp.toString()
      // this.resultsL.map(exprs => {
      //   let expr = algebra.parse(`${exprs[0]}${exprs[1]}`)
      //   console.log('Before loop ->', expr.toString())
      //   for (let i = 2; i < exprs.length; i++) {
      //     for (let i = 2; i < exprs.length; i++) {
      //       console.log(exprs[i])
      //     }
      //     expr = expr.multiply(algebra.parse(exprs[i]))
      //     console.log('After iter ->', expr.toString())
      //   }
      // })
      this.renderChart = true
    },
    paintChart () {
      this.renderChart(this.answerL)
    }
  }
}
</script>
