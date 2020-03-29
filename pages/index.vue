<template>
  <div class="container">
    <template v-for="y in board.length">
      <div
        v-for="x in board[y - 1].length"
        :key="`${y}-${x}`"
        class="cell"
        :class="{
          //puttable: 0 < flippableDir[color - 1][y - 1][x - 1][0].length
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

interface Cell {
  x: number
  y: number
  color: number
  enemyDir: number[][]
  flippableDir: number[][]
}

@Component
export default class extends Vue {
  // prettier-ignore
  color = 1
  boardSize = 8

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

  searchDir = new Map([
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
  sentinel = [
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

  get hasStone() {
    return (x: number, y: number): boolean => this.board[y][x] !== 0
  }

  get isBlack() {
    return (x: number, y: number): boolean => this.board[y][x] === 1
  }

  onClick(x: number, y: number) {
    const enemies = this.puttableCells
    const hasEnemy = enemies.some((cell) => cell.x === x && cell.y === y)
    const canPut = hasEnemy && !this.hasStone(x, y)

    if (canPut) {
      this.board = JSON.parse(JSON.stringify(this.board))
      this.board[y][x] = this.color
      this.color = 3 - this.color
    }
  }
  get puttableCells(): Cell[] {
    const flatBoard = this.flatBoard.filter((cell) => {
      return this.touchingEnemies(cell).length > 0
    })
    console.log(flatBoard)
    return flatBoard
  }

  get flatBoard() {
    return this.board.flatMap((row, y) =>
      row.map((color, x) => ({ x, y, color, enemyDir: [], flippableDir: [] }))
    )
  }

  get touchingEnemies() {
    const touchingEnemies = (cell: Cell): number[][] => {
      cell.enemyDir = (
        this.searchDir.get(this.sentinel[cell.y][cell.x]) || []
      ).filter((dir) => {
        const target = this.board[cell.y + dir[0]][cell.x + dir[1]]
        return (
          target !== 0 &&
          target !== this.color &&
          this.board[cell.y][cell.x] === 0
        )
      })
      this.flippableEnemies(cell)
      return cell.enemyDir
    }
    return touchingEnemies
  }
  get flippableEnemies() {
    const flippableEnemies = (cell: Cell): number[][] => {
      cell.enemyDir.forEach((dir) => {
        const [dy, dx] = [dir[1], dir[0]]
        let [y, x] = [cell.y + dy, cell.x + dx]
        const tmp = []
        while (
          this.sentinel[y - dy][x - dx] === 0 &&
          this.board[y][x] === 3 - this.color
        ) {
          console.log([y, x])
          tmp.push([y, x])
          y += dy
          x += dx
          if (this.board[y][x] === this.color) {
            tmp.forEach((elm) => {
              cell.flippableDir.push(elm)
            })
            break
          }
        }
      })
      return cell.enemyDir
    }
    return flippableEnemies
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

.touchEnemy {
  background: #060;
}

.puttable {
  background: #070;
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
