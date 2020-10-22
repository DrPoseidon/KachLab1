<template>
  <div class="container">
    <div v-if="success" class="A">
      <div class="a_text">
        A=
      </div>
      <div class="a_row">
        <div v-for="(el,index) in firstMatrix" :key="index">
          <div style="display:flex">
            <div v-for="(el1,index1) in el" :key="index1">
              <div class="a_row_el">
                <div v-if="index !=0 && index1 !=0">
                  {{ el1.toFixed(3) }}
                </div>
                <div v-else>
                  {{ el1 }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="success" class="Y">
      <div class="y_text">
        Y=
      </div>
      <div class="y_row">
        <div v-for="(el,index) in y" :key="index">
          <div v-for="(el1,index1) in el" :key="index1">
            <div class="y_row_el">
              {{ el1.toFixed(3) }}
            </div>
          </div>
        </div>
      </div>
      <div v-if="success">
        <div v-for="(el,index) in xes" :key="index" class="xes">
          <div v-if="index!='f(x)'" class="xx">
            {{ index }} = {{ el.toFixed(3) }}
          </div>
          <div v-else class="fx">
            {{ index }} = {{ el.toFixed(3) }}
          </div>
        </div>
      </div>
    </div>

    <div v-if="success == false">
      Укажите количество видов продукции <b-input v-model="noc" type="number" class="input" />
    </div>
    <div v-if="success == false">
      Укажите количество оборудования, используемого при производстве <b-input v-if="success == false" v-model="top" type="number" class="input" />
    </div>
    <div v-if="!success">
      <b-button @click="fill = true">
        Заполнить
      </b-button>
    </div>
    <div v-if="err != ''" class="error_div">
      {{ err }}
    </div>
    <div v-if="success == false" class="matrix">
      <div v-for="(i,indexi) in matrix" :key="indexi">
        <div class="mat-row">
          {{ `Продукт ${indexi+1}` }}
          <div v-for="(j,indexj) in i" :key="indexj">
            <b-input v-model="matrix_elements[indexi][indexj]" :class="{err: isError}" type="number" class="mitem" :placeholder="'a'+(indexj+1)" />
          </div>
        </div>
      </div>
    </div>
    <div v-if="matrix.length != 0 && success == false">
      Введите прибыль от изделий
      <div v-for="(el,key) in matrix.length" :key="key">
        <b-input v-model="profit[key]" :class="{err: isError}" type="number" class="input1" :placeholder="key+1" />
      </div>
      Введите время работы оборудования
      <div v-for="(el,key) in matrix[0].length" :key="key">
        <b-input v-model="working_hours[key]" :class="{err: isError}" type="number" :placeholder="key+1" class="input1" />
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
      isError: false,
      xes: false,
      y: false,
      success: false,
      fill: false,
      firstMatrix: false
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
    },
    fill () {
      if (this.fill) {
        this.calculate()
      }
    }
  },
  mounted () {
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
    simMethod (simMatrix, emptyMatrix, mEl) {
      let k = 0
      let min = 0
      let minCol = false
      let devider = false
      let minRow = {
        min: Infinity,
        index: 0,
        x: ''
      }
      // Начало
      while (k < simMatrix[0].length - 1) {
        // simMatrix.forEach((el) => {
        //   console.log(el)
        // })
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
        devider = simMatrix[minRow.index][minCol.index]
        for (let i = 0; i < emptyMatrix.length; i++) {
          for (let j = 0; j < emptyMatrix[i].length; j++) {
            if (i != 0) {
              if (i == minRow.index) {
                if (j == 0) {
                  emptyMatrix[i][j] = minCol.x
                } else {
                  emptyMatrix[i][j] = simMatrix[i][j] / devider
                }
              }
            }
          }
        }
        // console.log(minRow, minCol)
        // emptyMatrix.forEach((el) => {
        //   console.log(el)
        // })
        for (let i = 0; i < emptyMatrix.length; i++) {
          for (let j = 0; j < emptyMatrix[i].length; j++) {
            if (i != 0 && j != 0 && i != minRow.index) {
              emptyMatrix[i][j] = simMatrix[i][j] - emptyMatrix[minRow.index][j] * simMatrix[i][minCol.index]
            }
          }
        }
        k = 0
        min = 0
        minCol = false
        devider = false
        minRow = {
          min: Infinity,
          index: 0,
          x: ''
        }

        for (let j = 0; j < emptyMatrix[emptyMatrix.length - 1].length; j++) {
          if (j != 0) {
            if (emptyMatrix[emptyMatrix.length - 1][j] >= 0) {
              k++
            }
          }
        }
        if (k < simMatrix[0].length - 1) {
          for (let i = 0; i < emptyMatrix.length; i++) {
            for (let j = 0; j < emptyMatrix[i].length; j++) {
              simMatrix[i][j] = emptyMatrix[i][j]
              if (i != 0 && j != 0) {
                emptyMatrix[i][j] = 0
              }
            }
          }
        } else {
          for (let i = 0; i < emptyMatrix.length; i++) {
            for (let j = 0; j < emptyMatrix[i].length; j++) {
              simMatrix[i][j] = emptyMatrix[i][j]
              if (i != 0 && j != 0) {
                emptyMatrix[i][j] = 0
              }
            }
          }
          break
        }
      }

      // Конец
      const xes = []

      simMatrix.forEach((el, index) => {
        el.forEach((el1, index1) => {
          if (index == 0 && index1 > 1 && index1 < mEl.length + 2) {
            xes.push(el1)
          }
        })
      })
      const xes1 = {}
      simMatrix.forEach((el, index) => {
        el.forEach((el1, index1) => {
          if (index1 == 0 && index != 0) {
            if (index != simMatrix.length - 1) {
              if (xes.includes(el1)) {
                xes1[el1] = simMatrix[index][1]
              }
            } else {
              xes1[el1] = simMatrix[index][1]
            }
          }
        })
      })
      return xes1
      /// ////////////////////////////////////////////////////////////////
      // console.log('simMatrix')
      // simMatrix.forEach((el) => {
      //   console.log(el)
      // })
      // console.log('emptyMatrix')
      // emptyMatrix.forEach((el) => {
      //   console.log(el)
      // })
      // console.log(minRow, minCol, devider)
      // console.log(k)
      /// ////////////////////////////////////////////////////////////////
      // const k = 0
    },
    calculate () {
      let mEl = false
      let prof = false
      let wH = false

      if (this.fill == true) {
        mEl = [[1.1, 7, 3, 5], [20.5, 3, 5, 2], [13.4, 3, 10, 11]]
        prof = [4.2, 5, 9]
        wH = [150, 120.3, 100, 130]
      } else {
        mEl = this.matrix_elements
        prof = this.profit
        wH = this.working_hours
      }

      mEl.forEach((el) => {
        el.forEach((el1) => {
          if (typeof el1 == 'string') {
            el1 = parseFloat(el1)
          }
        })
      })

      prof.forEach((el) => {
        if (typeof el == 'string') {
          el = parseFloat(el)
        }
      })

      wH.forEach((el) => {
        if (typeof el == 'string') {
          el = parseFloat(el)
        }
      })

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
      const firstMatrix = []
      for (let i = 0; i < simMatrix.length; i++) {
        firstMatrix.push([])
        for (let j = 0; j < simMatrix[i].length; j++) {
          firstMatrix[i].push(simMatrix[i][j])
        }
      }
      this.firstMatrix = firstMatrix
      const xes = this.simMethod(simMatrix, emptyMatrix, mEl)
      this.xes = xes

      for (let i = 0; i < firstMatrix.length; i++) {
        for (let j = 0; j < firstMatrix[i].length; j++) {
          if (j < mEl.length + 2 && j > 1 && j != 0 && i != 0) {
            firstMatrix[i][j] = xes[firstMatrix[0][j]] * firstMatrix[i][j]
          }
        }
      }

      const y = []
      for (let i = 0; i < firstMatrix.length - 2; i++) {
        y.push([])
        for (let j = 0; j < mEl.length; j++) {
          y[i].push(firstMatrix[i + 1][j + 2])
        }
      }
      this.y = y
      this.success = true
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
.y_row{
  display: flex;
  flex-direction: row;
  align-items: center;
}
.a_row{
  display: flex;
  flex-direction: column;
  align-items: center;
}
.y_row_el,.a_row_el{
  border: 2px solid black;
  margin: 10px;
  min-width: 60px;
  height: 60px;
  padding: 5px;
  align-items: center;
  display: flex;
  flex-direction: row;
}
.Y,.A{
  display: flex;
  flex-direction: row;
  align-items: center;
}
.y_text,.a_text{
  font-size: 50px;
}
</style>
