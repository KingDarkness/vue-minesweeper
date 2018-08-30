<template>
  <div id="wapper">
    <div id="board">
      <div class="menu">
        <div class="remaining">
          <div class="select">
            <select v-model="difficulty" @change="setConfig(options[difficulty])">
              <option v-for="(diff, key) in options" :value="key" :key="key">{{ key }}</option>
            </select>
          </div>
          <div>&nbsp;&nbsp;</div>
          <div>
            💣 {{ gameState.minesRemaning }}
          </div>
        </div>
        <div class="control" @click="initGameState">
          <span v-if="gameState.won">😎</span>
          <span v-else-if="gameState.gameOver">😵</span>
          <span v-else>😀</span>
        </div>
        <div class="time">⏱ {{ gameState.coundown }}</div>
      </div>
      <div class="game">
        <div :style="{width: config.columns * 30 + 'px'}" v-if="ready" v-for="row in config.rows" :key="'row_' + row">
          <div
            v-for="col in config.columns"
            :key="'col_' + row + '_' + col"
            class="item"
            :class="[
              {'close': !isOpened(row - 1, col - 1)},
              {'open': isOpened(row - 1, col - 1)},
            ]"
            type="button"
            @click="open(row - 1, col - 1)"
            @contextmenu.prevent="addFlag(row - 1, col - 1)"
          >
            <span class="flag" v-if="getGameState(row - 1, col - 1, 'flaged')">🚩</span>
            <span v-else-if="getGameState(row - 1, col - 1, 'opened')"
              class="number"
              :class="'mine_' + getGameState(row - 1, col - 1, 'value')">
                {{ getGameState(row - 1, col - 1, 'value') !== false ? getGameState(row - 1, col - 1, 'value') : '💣' }}
            </span>
            <span v-else>&nbsp;</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'minesweeper',
  data () {
    return {
      difficulty: 'beginning',
      options: {
        beginning: {
          columns: 8,
          rows: 8,
          mines: 10
        },
        intermediate: {
          columns: 16,
          rows: 16,
          mines: 40
        },
        expert: {
          columns: 30,
          rows: 16,
          mines: 99
        },
        impossible: {
          columns: 40,
          rows: 20,
          mines: 300
        }
      },
      config: {
        columns: 0,
        rows: 0,
        mines: 0
      },
      gameState: {
        board: [],
        gameOver: false,
        won: false,
        minesRemaning: 0,
        realMines: 0,
        coundown: 0,
        timer: null,
        covered: 0,
        minesPos: []
      },
      ready: false
    }
  },
  methods: {
    setConfig (config) {
      this.config = config
      this.initGameState()
    },
    randomMines () {
      let minesRemaning = Object.assign({}, this.config).mines
      while (minesRemaning > 0) {
        let row = Math.floor(Math.random() * this.config.rows)
        let col = Math.floor(Math.random() * this.config.columns)
        this.setGameState(row, col, false, 'value')
        this.gameState.minesPos.push({row, col})
        minesRemaning--
      }
    },
    generateBoard () {
      this.gameState.board = Array.apply(null, Array(this.config.rows)).map(() => {
        return Array.apply(null, Array(this.config.columns)).map(() => {
          return { value: null, opened: false, flaged: false }
        })
      })
    },
    caculatorMines () {
      for(let row = 0; row < this.config.rows; row++){
        for(let col = 0; col < this.config.columns; col++){
          if (this.getGameState(row, col, 'value') !== false) {
            this.setGameState(row, col, this.minesCount(row, col), 'value')
          }
        }
      }
    },
    clearTimer () {
      this.gameState.timer && clearInterval(this.gameState.timer)
    },
    initGameState () {
      this.clearTimer()

      this.gameState = {
        board: [],
        gameOver: false,
        won: false,
        minesRemaning: Object.assign({}, this.config).mines,
        realMines: Object.assign({}, this.config).mines,
        coundown: 0,
        timer: null,
        minesPos: [],
        covered: this.config.columns * this.config.rows - this.config.mines
      }

      this.generateBoard()
      this.randomMines()
      this.caculatorMines()

      this.gameState.timer = setInterval(() => {
        this.gameState.coundown++
      }, 1000)

      this.ready = true
    },
    minesCount (x, y) {
      let count = 0;
      count += this.isMine(x - 1, y)
      count += this.isMine(x + 1, y)
      count += this.isMine(x, y - 1)
      count += this.isMine(x, y + 1)
      count += this.isMine(x - 1, y - 1)
      count += this.isMine(x + 1, y - 1)
      count += this.isMine(x - 1, y + 1)
      count += this.isMine(x + 1, y + 1)
      return count
    },
    isMine (x, y) {
      return this.inBoard(x, y) && this.getGameState(x, y, 'value') === false ? 1 : 0
    },
    inBoard (x, y) {
      return x >= 0 && y>= 0 && x < this.config.rows && y < this.config.columns
    },
    getGameState (row, col, key = null) {
      return key ? this.gameState.board[row][col][key] : this.gameState.board[row][col]
    },
    setGameState (row, col, value, key = null) {
      if (key) {
        this.gameState.board[row][col][key] = value
      } else {
        this.gameState.board[row][col] = value
      }
    },
    open (x, y) {
      if (this.gameState.gameOver) return
      if (!this.inBoard(x, y)) return
      if (this.getGameState(x, y, 'opened')) return
        if (this.getGameState(x, y, 'flaged')) return
      this.setGameState(x, y, true, 'opened')
      if (this.getGameState(x, y, 'value') === false) {
        this.gameOver()
        return
      } else {
        this.gameState.covered--
      }
      if (this.minesCount(x, y) > 0) return

      this.open(x - 1, y)
      this.open(x + 1, y)
      this.open(x, y - 1)
      this.open(x, y + 1)
      this.open(x - 1, y - 1)
      this.open(x + 1, y - 1)
      this.open(x - 1, y + 1)
      this.open(x + 1, y + 1)
    },
    addFlag(x, y) {
      if (this.gameState.gameOver) return
      if (!this.inBoard(x, y)) return
      if (this.getGameState(x, y, 'opened')) return
      this.setGameState(x, y, !this.getGameState(x, y, 'flaged'), 'flaged')
      if (this.getGameState(x, y, 'flaged')) {
        this.gameState.minesRemaning--
        this.getGameState(x, y, 'value') === false && this.gameState.realMines--
      } else {
        this.gameState.minesRemaning++
      }
    },
    isOpened (x, y) {
      return this.getGameState(x, y, 'opened') || this.getGameState(x, y, 'flaged')
    },
    won () {
      this.gameState.won = true
      this.clearTimer()
      this.gameState.minesPos.forEach((i) => {
        if (!this.getGameState(i.row, i.col, 'flaged')) {
          this.addFlag(i.row, i.col)
        }
      })
    },
    gameOver () {
      this.gameState.gameOver = true
      this.clearTimer()
    }
  },
  mounted () {
    this.setConfig(this.options[this.difficulty])
  },
  watch: {
    'gameState.covered': function (value) {
      if (value === 0) {
        this.won()
      }
    },
    'gameState.realMines': function (value) {
      if (value === 0 && this.gameState.minesRemaning === 0) {
        this.won()
      }
    }
  }
}
</script>
<style>
*{
  margin: 0;
  padding: 0;
}
</style>
<style scoped>
#wapper{
  display: flex;
  justify-content: center;
}
#board{
  display: inline-block;
}
.menu{
  height: 30px;
  background-color: #7f7f7f;
  color: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid #fff;
}
.remaining {
  padding-left: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.time {
  padding-right: 5px;
}
.control{
  padding: 0px 5px;
  cursor: pointer;
}
.game{
  overflow: auto;
  max-width: 100vw;
}
.item {
  width: 29px;
  height: 29px;
  display: inline-block;
  border: 0.5px solid #fff;
  text-align: center;
  vertical-align: middle;
  line-height: 30px;
  cursor: pointer;
}
.number{
  font-weight: bold;
}
.mine_0{
  color: transparent;
}
.mine_1{
  color: #0000ca;
}
.mine_2{
  color: #004c00;
}
.mine_3{
  color: #970000;
}
.mine_4{
  color: #00004c;
}
.mine_5{
  color: #791f10;
}
.mine_6{
  color: #00004c;
}
.mine_7{
  color: #171714;
}
.mine_8{
  color: #838383;
}
.flag{
  font-weight: bold;
  color: #000;
}
.close {
  background: #aaaaaafa;
}
.open {
  background: #e1e1e1;
}
/* Reset Select */
select {
  -webkit-appearance: none;
  -moz-appearance: none;
  -ms-appearance: none;
  appearance: none;
  outline: 0;
  box-shadow: none;
  border: 0 !important;
  background: #2c3e50;
  background-image: none;
}
/* Custom Select */
.select {
  position: relative;
  display: block;
  width: 100px;
  height: 1.3em;
  line-height: 1.3em;
  background: #2c3e50;
  overflow: hidden;
  border-radius: .25em;
}
select {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0 0 0 .5em;
  color: #fff;
  cursor: pointer;
}
select::-ms-expand {
  display: none;
}
/* Arrow */
.select::after {
  content: '\25BC';
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  padding: 0 5px;
  background: #34495e;
  pointer-events: none;
  font-size: 10px;
}
/* Transition */
.select::after {
  -webkit-transition: .25s all ease;
  -o-transition: .25s all ease;
  transition: .25s all ease;
}
</style>