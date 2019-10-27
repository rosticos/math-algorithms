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
      <button @click="countTable">Count</button>
    </div>
    <vue-mathjax :formula="`$$ ${kek} $$`"></vue-mathjax>
  </div>
</template>

<script>
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

export default {
  data () {
    return {
      fields: [
        { labelX: 'X', valueX: '-5', labelY: 'Y', valueY: '1' },
        { labelX: 'X', valueX: '-3', labelY: 'Y', valueY: '-4' },
        { labelX: 'X', valueX: '-1', labelY: 'Y', valueY: '3' },
        { labelX: 'X', valueX: '1', labelY: 'Y', valueY: '-3' }
      ],
      kek: ''
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
    countTable () {
      // let XiArr = this.fields.map(x => return x.valueX)
      let YiArr = this.fields.map(y => parseInt(y.valueY))
      let resultArr = []
      while (YiArr.length > 1) {
        YiArr = countYiDiff(YiArr)
        console.log(YiArr)
        resultArr.push(YiArr)
      }
      this.kek = makeExpressionLetters(this.fields)
      console.log(resultArr)
      console.log(m)
    }
  }
}
</script>
