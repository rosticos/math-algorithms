<template>
  <div>
    <div>
      <label>Шаг между узловыми точками (h):</label>
      <input type="text" v-model.number="diff">
    </div>
    <br>
    <div>
      <label>Количество точек:</label>
      <input type="text" v-model.number="pointsAmmount">
    </div>
    <br>
    <div>
      <label>Начальная точка:</label>
      <input type="text" v-model.number="start">
    </div>
    <br>

    <div v-for="(field, index) in fields" :key="index">
      <label>'{{ index }}'x) </label>
      <input type="text" v-model.number="field.valueX">
      <label>'{{ index }}'y) </label>
      <input type="text" v-model.number="field.valueY">
    </div>

    <div>
      <button @click="init" v-if="fields.length < 10">Init fields</button>
      <button @click="countTable">Count</button>
    </div>

    <div class="ended-table">
      <div class="ended-table__item">
        <vue-mathjax :formula="`$$ i $$`" style="border"></vue-mathjax>
        <p v-for="(item, index) in fields" :key="`fields-${index}`">{{ index }}</p>
      </div>
      <div class="ended-table__item">
        <vue-mathjax :formula="`$$ x_i $$`"></vue-mathjax>
        <p v-for="(item, index) in fields" :key="`fields-${index}`">{{ item.valueX }}</p>
      </div>
      <div class="ended-table__item">
        <vue-mathjax :formula="`$$ y_i $$`"></vue-mathjax>
        <p v-for="(item, index) in fields" :key="`fields-${index}`">{{ item.valueY }}</p>
      </div>
      <div v-for="(arr, index) in resultArr" :key="`deltaY-${index}`" class="ended-table__item">
        <vue-mathjax v-if="index === 0" :formula="`$$ \\Delta y_i $$`"></vue-mathjax>
        <vue-mathjax v-else :formula="`$$ \\Delta^${index + 1} y_i $$`"></vue-mathjax>
        <p v-for="(item, num) in arr" :key="`resArr[${index}]-${num}`">{{ item }}</p>
      </div>
    </div>

    <vue-mathjax :formula="`$$ ${ newtonFormule } $$`"></vue-mathjax>
    <vue-mathjax :formula="`$$ P_${fields.length - 1} = ${ newtonFormule } $$`"></vue-mathjax>
    <vue-mathjax :formula="`$$ ${ xExpLetters } = ${this.countExp.replace(/([-\d]*)\/([-\d]*)/g, '\\frac{$1}{$2}')} = ${this.countExpDec}$$`"></vue-mathjax>
  </div>
</template>

<script>
import algebra from 'algebra.js'
import { evaluate } from 'mathjs'

function countYiDiff (arr) {
  let detArr = []
  for (let i = 1; i < arr.length; i++) {
    const detY = arr[i] - arr[i - 1]
    detArr.push(detY)
  }
  return detArr
}

const makeExpressionLetters = (arr) => {
  let xExpLetters = 'y_0'
  for (let i = 1; i < arr.length; i++) {
    if (i === 1) {
      xExpLetters += '+ \\frac{\\Delta y_0}{h}(\\tilde{x}-x_0)'
    } else {
      xExpLetters += `+ \\frac{\\Delta^${i}y_0}{h^${i} \\cdot ${i}!}`
      for (let j = 0; j < i; j++) {
        xExpLetters += `(\\tilde{x}-x_${j})`
      }
    }
  }
  return xExpLetters
}

const factorial = (n) => {
  let result = 1
  while (n) {
    result *= n--
  }
  return result
}

const makeExpressionNumbers = (arr, xArr, y0, h, point) => {
  let xExpLetters = `${y0}`
  let countExp = `${y0}`
  for (let i = 0; i < arr.length; i++) {
    if (i === 0) {
      xExpLetters += `+ \\frac{${arr[i][0]}}{${h}}`
      countExp += `+ (${arr[i][0]} / ${h})`

      if (xArr[i] > 0) {
        xExpLetters += `(${point} - ${xArr[i]})`
        countExp += `(${point} - ${xArr[i]})`
      } else {
        xExpLetters += `(${point} + ${Math.abs(xArr[i])})`
        countExp += `(${point} + ${Math.abs(xArr[i])})`
      }
    } else {
      xExpLetters += `+ \\frac{${arr[i][0]}}{${h}^${i + 1} \\cdot ${i + 1}!}`
      countExp += `+ (${arr[i][0]} / (${h}^${i + 1} * ${factorial(i + 1)}))`
      
      for (let j = 0; j <= i; j++) {
        if (xArr[j] > 0) {
          xExpLetters += `(${point} - ${xArr[j]})`
          countExp += `(${point} - ${xArr[j]})`
        } else {
          xExpLetters += `(${point} + ${Math.abs(xArr[j])})`
          countExp += `(${point} + ${Math.abs(xArr[j])})`
        }
      }
    }
  }

  return {
    xExpLetters,
    countExp
  }
}

export default {
  data () {
    return {
      fields: [],
      newtonFormule: '',
      resultArr: [],
      diff: '2',
      start: '-5',
      pointsAmmount: '4',
      point: '-4',
      xExpLetters: '',
      countExp: '',
      countExpDec: ''
    }
  },
  methods: {
    addField () {
      this.fields.push(
        { labelX: 'X', valueX: '', labelY: 'Y', valueY: '' }
      )
    },
    init () {
      this.fields = []
      for (let i = 0; i < this.pointsAmmount; i++) {
        this.fields.push({
          labelX: 'X',
          valueX: `${Number(this.start) + this.diff * i}`,
          labelY: 'Y',
          valueY: '' }
        )
      }
    },
    removeField () {
      this.fields.pop()
    },
    countTable () {
      // let XiArr = this.fields.map(x => return x.valueX)
      let YiArr = this.fields.map(y => parseInt(y.valueY))
      this.resultArr = []
      while (YiArr.length > 1) {
        YiArr = countYiDiff(YiArr)
        this.resultArr.push(YiArr)
      }
      this.newtonFormule = makeExpressionLetters(this.fields)
      console.log(this.resultArr)

      let xArr = this.fields.map(el => { return el.valueX })
      let yArr = this.fields.map(el => { return el.valueY })
      const { xExpLetters, countExp } = makeExpressionNumbers(this.resultArr, xArr, yArr[0], this.diff, this.point)
      this.xExpLetters = xExpLetters
      this.countExp = algebra.parse(countExp).toString()
      this.countExpDec = evaluate(this.countExp)
    }
  }
}
</script>

<style>
  .ended-table {
    display: flex;
    flex-wrap: nowrap;
  }
  .ended-table__item {
    border: 1px solid #666;
    padding: 20px;
    text-align: center;
    margin-right: -1px;
  }
</style>
