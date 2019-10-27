<template>
  <div>
    <div v-for="(field, index) in fields" :key="index">
      <label>'{{ index }}'x) </label>
      <input type="text" v-model.number="field.valueX">
      <label>'{{ index }}'y) </label>
      <input type="text" v-model.number="field.valueY">
    </div>
    <label>'Point) </label>
    <input type="text" v-model.number="point">
    <div>
      <button @click="addField" v-if="fields.length < 10">Add field</button>
      <button @click="removeField" v-if="fields.length">Remove field</button>
      <button @click="countResultsL">Count</button>
      <button @click="paintChart">Paint</button>
    </div>
    <div v-if="resultsL.length">
      <p>Начальные уравнения:</p>
      <div style="display: flex; flex-wrap: wrap; ">
        <div v-for="(value, index) in resultsL" :key="`res-${index}`" style="margin-bottom: 32px; flex: 1 1 100%;">
          <vue-mathjax v-if="value != null" :formula="`$$l_${index} = ${fields[index].valueY} * {${value.numerator.join('')} \\over${value.denominator.join('')}} = ${fields[index].valueY} *  {${countedResultL[index].numerator} \\over${countedResultL[index].denominator}}= ${answerL[index].replace(/([-\d]*)\/([-\d]*)/g, '\\frac{$1}{$2}')} $$`"></vue-mathjax>
        </div>
      </div>
      <p>График интерполяционного многочлена Лагранджа</p>
      <vue-mathjax :formula="`$$ L_${fields.length - 1}(x) = ${grapg.replace(/([-\d]*)\/([-\d]*)/g, '\\frac{$1}{$2}')} $$`"></vue-mathjax>
    </div>
    <div style="display: flex; justify-content: center; margin-top: 20px;">
      <line-chart
        v-if="renderChart"
        :func="grapg"
        style="max-width: 500px; max-height: 500px;"
        ref="line-chart"
      />
    </div>
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
      renderChart: false,
      point: ''
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
  },
  computed: {
    numeratorIndex1 () {
      return `$$ ${this.countedResultL[0].numerator} $$`
    }
  }
}
</script>
