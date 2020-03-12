<template>
  <div class="container">
    <template v-for="y in board.length">
      <div
        v-for="x in board[y - 1].length"
        :key="`${y}-${x}`"
        class="cell"
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

interface Cell {
  x: number
  y: number
  color: number
}

@Component
export default class extends Vue {
  ↑ = 1
  get hasStone() {
    return (x: number, y: number): boolean => this.board[y][x] !== 0
  }

  get isBlack() {
    return (x: number, y: number): boolean => this.board[y][x] === 1
  }

  // dirs = new Map([
  //   // direction by bit mask
  //   [0, {}],
  //   [1, { y: 1, x: 0 }],
  //   [2, { y: 1, x: 1 }],
  //   [4, { y: 0, x: 1 }],
  //   [8, { y: -1, x: 1 }],
  //   [16, { y: -1, x: 0 }],
  //   [32, { y: -1, x: -1 }],
  //   [64, { y: 0, x: -1 }],
  //   [128, { y: 1, x: -1 }]
  // ])

  get hasNeighbor() {
    const sentinel = [
      // edge of an array can be tricky, I set flags for safty
      [65, 1, 1, 1, 1, 1, 1, 5],
      [64, 0, 0, 0, 0, 0, 0, 4],
      [64, 0, 0, 0, 0, 0, 0, 4],
      [64, 0, 0, 0, 0, 0, 0, 4],
      [64, 0, 0, 0, 0, 0, 0, 4],
      [64, 0, 0, 0, 0, 0, 0, 4],
      [64, 0, 0, 0, 0, 0, 0, 4],
      [80, 16, 16, 16, 16, 16, 16, 20]
    ]
    const safeDirs = new Map([
      // eslint-disable-next-line prettier/prettier
      [0,[[-1, 0], [-1, 1], [0, 1], [1, 1], [1, 0], [1, -1], [0, -1], [-1, -1]]],
      [1, [[0, 1], [1, 1], [1, 0], [1, -1], [0, -1]]],
      [4, [[-1, 0], [1, 0], [1, -1], [0, -1], [-1, -1]]],
      [5, [[1, 0], [1, -1], [0, -1]]],
      [16, [[-1, 0], [-1, 1], [0, 1], [0, -1], [-1, -1]]],
      [20, [[-1, 0], [0, -1], [-1, -1]]],
      [64, [[-1, 0], [-1, 1], [0, 1], [1, 1], [1, 0]]],
      [65, [[0, 1], [1, 1], [1, 0]]],
      [80, [[-1, 0], [0, 1], [-1, 1]]]
    ])

    return (cell: Cell): boolean => {
      return (safeDirs.get(sentinel[cell.y][cell.x]) || []).some((dir) => {
        return this.board[cell.y + dir[0]][cell.x + dir[1]] !== 0
      })
    }
  }

  get puttableCells(): Cell[] {
    return this.board // return booleans
      .flatMap((row, y) => row.map((color, x) => ({ x, y, color })))
      .filter((cell) => {
        return this.hasNeighbor(cell)
      })
  }

  // prettier-ignore
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

  // ____________
  // | 128, 1, 2|
  // |  64, 0, 4|
  // |  32,16, 8|
  // ------------
  // this 3x3 table explain direction by bit mask
  // this enable describe multi-direction as a single value
  // for example '1' means (↑)
  // 65 - (1 + 64) means (↑ , ←)
  // 128 = (↖)
  // 255 = (↑ , ➚ , → , ➘ , ↓ , ↙ , ← , ↖)
  // 0 = no direction.

  currentColor = 1

  onClick(x: number, y: number) {
    const canPut =
      this.puttableCells.some((cell) => cell.x === x && cell.y === y) &&
      !this.hasStone(x, y)

    if (canPut) {
      this.board = JSON.parse(JSON.stringify(this.board))
      this.board[y][x] = this.currentColor
      this.currentColor = 3 - this.currentColor
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
