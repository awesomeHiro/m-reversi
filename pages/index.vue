<template>
  <div class="container">
    <template v-for="y in board.length">
      <div
        v-for="x in board[y - 1].length"
        :key="`${y}-${x}`"
        class="cell"
        :class="{
          // available: 0 !== availableBoard[color - 1][y - 1][x - 1]
        }"
        @click="onClick(x - 1, y - 1)"
      >
        <div
          v-if="hasStone(x - 1, y - 1)"
          :class="['stone', isBlack(x - 1, y - 1) ? 'black' : 'white']"
        />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

// const EMPTY = 0
// const WHITE = -1
// const BLACK = 1
// const dirBit = new Map([
//   ['NO', 0],
//   ['↑', 1],
//   ['➚', 2],
//   ['→', 4],
//   ['➘', 8],
//   ['↓', 16],
//   ['↙', 32],
//   ['←', 64],
//   ['↖', 128]
// ])
// dir = new Map([
//   [this.dirBit.get('NO'), { y: 0, x: 0 }],
//   [this.dirBit.get('↑'), { y: 1, x: 0 }],
//   [this.dirBit.get('➚'), { y: 1, x: 1 }],
//   [this.dirBit.get('→'), { y: 0, x: 1 }],
//   [this.dirBit.get('➘'), { y: -1, x: 1 }],
//   [this.dirBit.get('↓'), { y: -1, x: 0 }],
//   [this.dirBit.get('↙'), { y: -1, x: -1 }],
//   [this.dirBit.get('←'), { y: 0, x: -1 }],
//   [this.dirBit.get('↖'), { y: 1, x: -1 }]
// ])
// safeDirs = new Map([
//   // eslint-disable-next-line prettier/prettier
//   [this.dirBit.get('NO'),[[-1, 0], [-1, 1], [0, 1], [1, 1], [1, 0], [1, -1], [0, -1], [-1, -1]]],
//   [this.dirBit.get('↑'), [[0, 1], [1, 1], [1, 0], [1, -1], [0, -1]]],
//   [this.dirBit.get('➚'), [[-1, 0], [1, 0], [1, -1], [0, -1], [-1, -1]]],
//   [this.dirBit.get('→'), [[1, 0], [1, -1], [0, -1]]],
//   [this.dirBit.get('➘'), [[-1, 0], [-1, 1], [0, 1], [0, -1], [-1, -1]]],
//   [this.dirBit.get('↓'), [[-1, 0], [0, -1], [-1, -1]]],
//   [this.dirBit.get('↙'), [[-1, 0], [-1, 1], [0, 1], [1, 1], [1, 0]]],
//   [this.dirBit.get('←'), [[0, 1], [1, 1], [1, 0]]],
//   [this.dirBit.get('↖'), [[-1, 0], [0, 1], [-1, 1]]]
// ])

@Component
export default class extends Vue {
  color = 1
  boardSize = 8
  beforeCreate() {
    const row = [] as any
    const board = []
    board.push(row)
    this.boardSize = 8
  }

  board = [
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 1, 2, 0, 0, 0],
    [0, 0, 0, 2, 1, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0]
  ]

  get hasStone() {
    return (x: number, y: number): boolean => this.board[y][x] !== 0
  }

  get isBlack() {
    return (x: number, y: number): boolean => this.board[y][x] === 1
  }

  onClick(x: number, y: number) {
    const canPut = true
    if (canPut) {
      this.board = JSON.parse(JSON.stringify(this.board))
      this.board[y][x] = this.color
      this.color = 3 - this.color
    }
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
