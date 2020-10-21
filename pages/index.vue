<template>
  <div class="container">
    Укажите количество видов продукции <b-input v-model="noc" type="number" class="input" />
    Укажите количество оборудования, используемого при производстве <b-input v-model="top" type="number" class="input" />
    <div v-if="err != ''" class="error_div">
      {{ err }}
    </div>
    <div class="matrix">
      <div v-for="(i,indexi) in matrix" :key="indexi">
        <div class="mat-row">
          {{ `Продукт ${indexi+1}` }}
          <div v-for="(j,indexj) in i" :key="indexj">
            <b-input v-model="matrix_elements[indexi][indexj]" :class="{err: isError}" type="number" class="mitem" :placeholder="'a'+(indexj+1)" />
          </div>
        </div>
      </div>
    </div>
    <div v-if="matrix.length != 0">
      Введите прибыль от изделий
      <div v-for="(el,key) in matrix.length" :key="key">
        <b-input v-model="profit[key]" :class="{err: isError}" type="number" class="input1" />
      </div>
      Введите время работы оборудования
      <div v-for="(el,key) in matrix[0].length" :key="key">
        <b-input v-model="working_hours[key]" :class="{err: isError}" type="number" class="input1" />
      </div>
      <b-button :variant="primary" @click="calculate">
        Вычислить
      </b-button>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      top: '',
      noc: '',
      matrix: [],
      array: [],
      matrix_elements: [],
      profit: [],
      working_hours: [],
      err: '',
      isError: false
    }
  },
  watch: {
    noc () {
      if (this.noc && this.top) {
        if (this.noc == 1 && this.top == 1) {
          console.log('error')
        } else {
          this.createMatrix()
        }
      }
    },
    top () {
      if (this.noc && this.top) {
        if (this.noc == 1 && this.top == 1) {
          console.log('error')
        } else {
          this.createMatrix()
        }
      }
    }
  },
  mounted () {
    this.calculate()
  },
  methods: {
    showMatrix () {
      console.log(this.matrix)
    },
    createMatrix () {
      const ii = this.noc
      const jj = this.top
      const profit = []
      const workingHours = []
      const mx = []
      for (let i = 0; i < ii; i++) {
        mx.push([])
        profit.push([])
        for (let j = 0; j < jj; j++) {
          mx[i].push('')
        }
      }
      mx[0].forEach((el) => {
        workingHours.push([])
      })
      this.working_hours = workingHours
      this.profit = profit
      this.matrix_elements = mx
      this.matrix = mx
    },
    ok (i, j) {
      if (j === 0) {
        return (i + 1)
      }
    },
    ifErr_matr (r) {
      r.forEach((el) => {
        console.log(el)
        el.forEach((i) => {
          if (i == '') {
            console.log('error')
            this.err = 'Проверьте правильность введенных значений'
            this.isError = true
          }
        })
      })
    },
    ifErr (r) {
      r.forEach((el) => {
        if (el == '') {
          console.log('error')
          this.err = 'Проверьте правильность введенных значений'
          this.isError = true
        }
      })
    },
    simMethod (simMatrix, emptyMatrix) {
      let min = 0
      let minCol = false
      let minRow = {
        min: Infinity,
        index: 0,
        x: ''
      }
      let k = 0
      while (k < simMatrix[0].length - 1) {
        simMatrix.forEach((el, index) => {
          if (index + 1 == simMatrix.length) {
            el.forEach((el1, index1) => {
              if (index1 != 0) {
                if (el1 < min) {
                  min = el1
                  minCol = {
                    min: el1,
                    index: index1,
                    x: simMatrix[0][index1]
                  }
                }
              }
            })
          }
        })

        for (let i = 0; i < simMatrix.length; i++) {
          if (i != simMatrix.length - 1 && i != 0) {
            for (let j = 0; j < simMatrix[i].length; j++) {
              if (j == minCol.index) {
                if (j != 0) {
                  const division = (simMatrix[i][1] / simMatrix[i][j])
                  if (division >= 0 && division < minRow.min) {
                    minRow.min = division
                    minRow.index = i
                    minRow.x = simMatrix[i][0]
                  }
                }
              }
            }
          }
        }

        for (let i = 0; i < simMatrix.length; i++) {
          for (let j = 0; j < simMatrix[i].length; j++) {
            emptyMatrix[minRow.index][0] = minCol.x
          }
        }

        for (let i = 0; i < simMatrix.length; i++) {
          for (let j = 0; j < simMatrix[i].length; j++) {
            if (j != 0 && i != 0) {
              if (i == minRow.index) {
                emptyMatrix[i][j] = simMatrix[i][j] / simMatrix[minRow.index][minCol.index]
              } else {
                emptyMatrix[i][j] = simMatrix[i][j] - (emptyMatrix[minRow.index][j] * simMatrix[i][minCol.index])
              }
            }
          }
        }

        for (let i = 0; i < simMatrix.length; i++) {
          for (let j = 0; j < simMatrix[i].length; j++) {
            if (j != 0 && i != 0) {
              if (i != minRow.index) {
                emptyMatrix[i][j] = simMatrix[i][j] - (emptyMatrix[minRow.index][j] * simMatrix[i][minCol.index])
              }
            }
          }
        }

        simMatrix = emptyMatrix

        for (let i = 0; i < emptyMatrix.length; i++) {
          for (let j = 0; j < emptyMatrix[i].length; j++) {
            if (i == emptyMatrix.length - 1 && j != 0 && emptyMatrix[i][j] >= 0) {
              k++
            }
          }
        }

        if (k < simMatrix[0].length - 1) {
          k = 0
        }

        min = 0
        minCol = false
        minRow = {
          min: Infinity,
          index: 0,
          x: ''
        }
      }
      return simMatrix
    },
    calculate () {
      // const mEl = [[0.7, 0.4, 0.5], [0.2, 0.4, 0.6]]
      // const prof = [5, 3]
      // const wH = [40, 36, 36]

      const mEl = [[2, 0], [1, 2]]
      const prof = [2, 5]
      const wH = [430, 460]

      const simMatrix = []
      const emptyMatrix = []
      for (let i = 0; i < wH.length + 2; i++) {
        simMatrix.push([])
        for (let j = 0; j < prof.length + 2 + mEl[0].length; j++) {
          if (i == 0 && j >= 2) {
            simMatrix[i][j] = `x${j - 1}`
          } else if (i == 0 && j == 0) {
            simMatrix[i][j] = ''
          } else if (i == 0 && j == 1) {
            simMatrix[i][j] = 'B'
          } else if (i != 0 && i != wH.length + 1 && j == 0) {
            simMatrix[i][j] = `x${i + mEl.length}`
          } else if (i != 0 && i == wH.length + 1 && j == 0) {
            simMatrix[i][j] = 'f(x)'
          } else {
            simMatrix[i].push(0)
          }
        }
      }

      for (let i = 0; i < wH.length + 2; i++) {
        emptyMatrix.push([])
        for (let j = 0; j < prof.length + 2 + mEl[0].length; j++) {
          if (i == 0 && j >= 2) {
            emptyMatrix[i][j] = `x${j - 1}`
          } else if (i == 0 && j == 0) {
            emptyMatrix[i][j] = ''
          } else if (i == 0 && j == 1) {
            emptyMatrix[i][j] = 'B'
          } else if (i != 0 && i != wH.length + 1 && j == 0) {
            emptyMatrix[i][j] = `x${i + mEl.length}`
          } else if (i != 0 && i == wH.length + 1 && j == 0) {
            emptyMatrix[i][j] = 'f(x)'
          } else {
            emptyMatrix[i].push(0)
          }
        }
      }

      for (let i = 0; i < simMatrix.length; i++) {
        for (let j = 0; j < simMatrix[i].length; j++) {
          if (i != 0 && j != 0) {
            if (i != simMatrix.length - 1) {
              if (j == 1) {
                simMatrix[i][j] = wH[i - 1]
              } else if (j < mEl.length + 2) {
                simMatrix[i][j] = mEl[j - 2][i - 1]
              } else if (j >= mEl.length + 1) {
                if (j - mEl.length - 1 == i) {
                  simMatrix[i][j] = 1
                } else {
                  simMatrix[i][j] = 0
                }
              }
            } else if (j <= prof.length + 1) {
              if (j == 1) {
                simMatrix[i][j] = 0
              }
              simMatrix[i][j + 1] = -prof[j - 1]
            } else {
              simMatrix[i][j] = 0
            }
          }
        }
      }

      console.log(this.simMethod(simMatrix, emptyMatrix))
    }
  }
}
</script>

<style>
.matrix{
  /* display: flex;
  flex-direction: row;
  align-items: center; */
}
.mat-row{
  display: flex;
  align-items: center;
  transition-duration: .5s;
}
.mat-row:hover{
  background-color: rgba(122, 122, 122, 0.2);
}
.name{
  display: flex;
  flex-direction: column;
  align-items: center;
}
.mitem{
  margin: 5px 2px;
  width: 50px;
  height: 50px;
  text-align: center;
}
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
    /* display: none; <- Crashes Chrome on hover */
    -webkit-appearance: none;
    margin: 0; /* <-- Apparently some margin are still there even though it's hidden */
}
.input{
  width: 50px;
  height: 50px;
  text-align: center;
}
.input1{
  width: 100px;
  height: 50px;
  margin: 5px;
}
*{
  margin: 0;
  padding: 0;
}
.err{
 border: 2px solid red;
}
.error_div{
  color: red;
  border: 1px solid red;
  width: 300px;
  padding: 10px;
  text-align: center;
  margin: 5px;
}
</style>
