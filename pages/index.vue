<template>
  <div class="container">
    <template v-for="y in boardSize">
      <div
        v-for="x in boardSize"
        :key="`${y}-${x}`"
        class="cell"
        :class="{
          // available: 0 !== availableBoard[color - 1][y - 1][x - 1]
        }"
        @click="onClick(x, y)"
      >
        <div
          v-if="hasStone(x, y)"
          :class="['stone', isBlack(x, y) ? 'black' : 'white']"
        />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

const BOARDSIZE = 8
// const EMPTY = 0
// const WHITE = -1
const BLACK = 1
// const MARGIN = 2
const dirBit = new Map([
  ['↑', 1],
  ['➚', 2],
  ['→', 4],
  ['➘', 8],
  ['↓', 16],
  ['↙', 32],
  ['←', 64],
  ['↖', 128]
])
const dirs = new Map([
  [dirBit.get('↑'), { y: -1, x: 0 }],
  [dirBit.get('➚'), { y: -1, x: 1 }],
  [dirBit.get('→'), { y: 0, x: 1 }],
  [dirBit.get('➘'), { y: 1, x: 1 }],
  [dirBit.get('↓'), { y: 1, x: 0 }],
  [dirBit.get('↙'), { y: 1, x: -1 }],
  [dirBit.get('←'), { y: 0, x: -1 }],
  [dirBit.get('↖'), { y: -1, x: -1 }]
])
@Component
export default class extends Vue {
  color = BLACK
  boardSize = BOARDSIZE
  beforeCreate() {}
  board = [
    [2, 2, 2, 2, 2, 2, 2, 2, 2, 2],
    [2, 0, 0, 0, 0, 0, 0, 0, 0, 2],
    [2, 0, 0, 0, 0, 0, 0, 0, 0, 2],
    [2, 0, 0, 0, 0, 0, 0, 0, 0, 2],
    [2, 0, 0, 0, 1, -1, 0, 0, 0, 2],
    [2, 0, 0, 0, -1, 1, 0, 0, 0, 2],
    [2, 0, 0, 0, 0, 0, 0, 0, 0, 2],
    [2, 0, 0, 0, 0, 0, 0, 0, 0, 2],
    [2, 0, 0, 0, 0, 0, 0, 0, 0, 2],
    [2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
  ]
  get hasStone() {
    return (x: number, y: number): boolean => this.board[y][x] !== 0
  }
  get isBlack() {
    return (x: number, y: number): boolean => this.board[y][x] === 1
  }
  onClick(x: number, y: number) {
    if (this.canPut(y, x)) {
      this.putStone(y, x)
      this.rerenderBoard()
      this.passTurn()
    }
  }
  canPut(y: number, x: number): number {
    return this.flippableDirs(y, x)
  }
  flippableDirs(y: number, x: number): number {
    let bit = 0
    dirs.forEach((dir, dirBit = 0) => {
      const [dy, dx] = [dir.y, dir.x]
      bit += this.isFlippable(y, x, dy, dx) ? dirBit : 0
    })
    return bit
  }
  isFlippable(y: number, x: number, dy: number, dx: number): boolean {
    let isActivated = false
    ;[y, x] = [y + dy, x + dx]
    while (this.board[y][x] === -this.color) {
      ;[y, x] = [y + dy, x + dx]
      isActivated = true
    }
    const isFlippable = this.board[y][x] === this.color && isActivated
    if (isFlippable) {
      ;[y, x] = [y - dy, x - dx]
      while (this.board[y][x] === -this.color) {
        this.board[y][x] = this.color
        ;[y, x] = [y - dy, x - dx]
      }
    }
    return isFlippable
  }
  isGameOver() {}
  putStone(y: number, x: number) {
    this.board[y][x] = this.color
  }
  passTurn() {
    this.color = this.color * -1
  }
  rerenderBoard() {
    this.board = JSON.parse(JSON.stringify(this.board))
  }
}
</script>

<style scoped>
.container {
  width: 480px;
  height: 480px;
  margin: 20px auto;
  background: #050;
}

.available {
  background: #060;
}

.cell {
  float: left;
  width: 12.5%;
  height: 12.5%;
  border: 1px #000 solid;
}

.stone {
  width: 70%;
  height: 70%;
  margin: 15%;
  border-radius: 50%;
}

.white {
  background: #fff;
}

.black {
  background: #000;
}
</style>