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
          :class="['stone', isBlack(x - 1, y - 1) ? 'white' : 'black']"
        />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

@Component
export default class extends Vue {
  get hasStone() {
    return (x: number, y: number): boolean => this.board[y][x] !== 0
  }

  get isBlack() {
    return (x: number, y: number): boolean => this.board[y][x] === 1
  }

  get hasVicinity() {
    return (x: number, y: number): boolean => {
      let result: boolean = false
      for (let yIndex: number = -1; yIndex < 2; yIndex++) {
        for (let xIndex: number = -1; xIndex < 2; xIndex++) {
          if (
            this.board[y + yIndex] &&
            this.board[y + yIndex][x + xIndex] &&
            this.board[y + yIndex][x + xIndex] !== 0
          ) {
            result = true
            break
          }
        }
        if (result) {
          break
        }
      }
      return result
    }
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

  currentColor = 1

  onClick(x: number, y: number) {
    if (!this.hasStone(x, y) && this.hasVicinity(x, y)) {
      this.currentColor = 3 - this.currentColor
      // onClick関数の処理が終わってからbordの値の更新を予約する
      this.board = JSON.parse(JSON.stringify(this.board))
      this.board[y][x] = this.currentColor
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
  border: 1px solid #000;
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
