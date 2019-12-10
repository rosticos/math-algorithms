<template>
  <div class="input-form_middle">
    <div class="input-form_middle__item">
      <label>Шаг между узловыми точками (h):</label>
      <input type="text" class="input-form__item-header" v-model.number="diff">
    </div>
    <div class="input-form_middle__item">
      <label>Количество точек:</label>
      <input type="text" class="input-form__item-header" v-model.number="pointsAmmount">
    </div>
    <div class="input-form_middle__item">
      <label>Начальная точка:</label>
      <input type="text" class="input-form__item-header" v-model.number="start">
    </div>
    <div class="input-form_middle__item">
      <label>Точка:</label>
      <input type="text" class="input-form__item-header" v-model.number="point">
    </div>
    <div style="margin-top: 40px">
      <el-button
        type="warning"
        v-if="countExpDec.length === 0"
        @click="init"
      >
        Заполнить начальные данные
      </el-button>

      <el-button
        type="warning"
        v-else
        @click="reset"
      >
        Очистить поля
      </el-button>
    </div>

    <div style="margin: 40px 0">
      <div>
        <div v-if="fields.length > 0" class="input-form">
          <input type="text" value="i" disabled class="input-form__item input-form__item_disabled">
          <input type="text" value="X" disabled class="input-form__item input-form__item_disabled">
          <input type="text" value="Y" disabled class="input-form__item input-form__item_disabled">
        </div>
        <div v-for="(field, index) in fields" :key="index" class="input-form">
          <input type="text" :value="index" disabled class="input-form__item input-form__item_disabled">
          <input type="text" v-model.number="field.valueX" class="input-form__item input-form__item_disabled">
          <input type="text" v-model.number="field.valueY" class="input-form__item">
        </div>
      </div>
    </div>

    <el-button
      v-if="checkFields && countExpDec.length === 0"
      type="warning"
      @click="countTable"
    >
      Рассчитать
    </el-button>

    <div class="ended-table" v-if="fields.length > 0">
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
    <div v-if="newtonFormule !== ''" class="input-form_middle">
      <div class="initial-eq__item">
        <vue-mathjax :formula="`$$ P_${fields.length - 1} = ${ newtonFormule } $$`"></vue-mathjax>
      </div>
      <div class="initial-eq__item">
        <vue-mathjax :formula="`$$ P_${fields.length - 1} = ${ countExpWithX } $$`"></vue-mathjax>
      </div>
      <div class="initial-eq__item">
        <vue-mathjax :formula="`$$ ${ xExpLetters } = ${this.countExp.replace(/([-\d]*)\/([-\d]*)/g, '\\frac{$1}{$2}')} = ${this.countExpDec}$$`"></vue-mathjax>
      </div>
    </div>

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
  let countExpWithX = `${y0}`

  for (let i = 0; i < arr.length; i++) {
    if (i === 0) {
      xExpLetters += `+ \\frac{${arr[i][0]}}{${h}}`
      countExp += `+ (${arr[i][0]} / ${h})`
      countExpWithX += `+ \\frac{${arr[i][0]}}{${h}}`

      if (xArr[i] > 0) {
        xExpLetters += `(${point} - ${xArr[i]})`
        countExp += `(${point} - ${xArr[i]})`
        countExpWithX += `(\\tilde{x} - ${xArr[i]})`
      } else {
        xExpLetters += `(${point} + ${Math.abs(xArr[i])})`
        countExp += `(${point} + ${Math.abs(xArr[i])})`
        countExpWithX += `(\\tilde{x} + ${Math.abs(xArr[i])})`
      }
    } else {
      xExpLetters += `+ \\frac{${arr[i][0]}}{${h}^${i + 1} \\cdot ${i + 1}!}`
      countExp += `+ (${arr[i][0]} / (${h}^${i + 1} * ${factorial(i + 1)}))`
      countExpWithX += `+ \\frac{${arr[i][0]}}{${h}^${i + 1} * ${factorial(i + 1)}}`

      for (let j = 0; j <= i; j++) {
        if (xArr[j] > 0) {
          xExpLetters += `(${point} - ${xArr[j]})`
          countExp += `(${point} - ${xArr[j]})`
          countExpWithX += `(\\tilde{x} - ${xArr[j]})`
        } else {
          xExpLetters += `(${point} + ${Math.abs(xArr[j])})`
          countExp += `(${point} + ${Math.abs(xArr[j])})`
          countExpWithX += `(\\tilde{x} + ${Math.abs(xArr[j])})`
        }
      }
    }
  }

  return {
    xExpLetters,
    countExp,
    countExpWithX
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
      countExpDec: '',
      countExpWithX: ''
    }
  },
  computed: {
    checkFields () {
      if (this.fields.length === 0) return false

      const showArray = this.fields.map(el => {
        return Object.values(el)
      })

      return showArray.flat().every(el => el !== '')
    }
  },
  methods: {
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
    reset () {
      this.fields = []
      this.newtonFormule = ''
      this.resultArr = []
      this.diff = 2
      this.start = -5
      this.pointsAmmount = 4
      this.point = -4
      this.xExpLetters = ''
      this.countExp = ''
      this.countExpDec = ''
      this.countExpWithX = ''
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
      console.log(this.newtonFormule)

      let xArr = this.fields.map(el => { return el.valueX })
      let yArr = this.fields.map(el => { return el.valueY })
      const { xExpLetters, countExp, countExpWithX } = makeExpressionNumbers(this.resultArr, xArr, yArr[0], this.diff, this.point)
      this.countExpWithX = countExpWithX
      this.xExpLetters = xExpLetters
      this.countExp = algebra.parse(countExp).toString()
      this.countExpDec = evaluate(this.countExp)
    }
  }
}
</script>
