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
        >
          x:{{ x - 1 }}y:{{ y - 1 }}
        </div>
        <div v-else>x:{{ x - 1 }}y:{{ y - 1 }}</div>
      </div>
    </template>
    <p>{{ currentStone }}のターン</p>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

/* cell: マス
stone: 石
border: ボード、マス情報
y: マスの縦位置
x: マスの横位置 */
type Board = Array<Array<number>>
type Cell = { x: number; y: number; color: number }
enum CellTypeEnum {
  WHITE = 'WHITE',
  BLACK = 'BLACK',
  NONE = 'NONE'
}
enum CellTypeNumberEnum {
  WHITE = 1,
  BLACK = 2,
  NONE = 0
}

@Component
export default class extends Vue {
  board: Board = [
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 1, 2, 2, 2, 2],
    [0, 0, 0, 2, 1, 1, 1, 1],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0]
  ]

  get currentStone(): CellTypeEnum {
    return this.currentColor === 1 ? CellTypeEnum.WHITE : CellTypeEnum.BLACK
  }

  currentColor: number = 1

  hasStone = (x: number, y: number): boolean => this.board[y][x] !== 0

  isBlack = (x: number, y: number): boolean => this.board[y][x] === 2

  /* 選択されたセルの周りに相手の石が置いてある場合はそのセル情報を返す。無ければ空配列を返す。 */
  getAroundOpponentCellsInfo = (
    selectX: number,
    selectY: number,
    selectColor: number
  ): Cell[] => {
    return this.getSelectedAroundCells(selectX, selectY).filter(
      (cell: Cell): boolean =>
        cell.color !== CellTypeNumberEnum.NONE && cell.color !== selectColor
    )
  }

  /* 選択した周りと自身のセルを取得 */
  getSelectedAroundCells = (selectX: number, selectY: number): Cell[] => {
    const row = [selectX - 1, selectX, selectX + 1]
    const collum = [selectY - 1, selectY, selectY + 1]
    /* TODO:処理を外に出す */
    if (selectX === 0) {
      row.shift()
    } else if (selectX + 1 === this.board.length) {
      row.pop()
    }
    if (selectY === 0) {
      collum.shift()
    } else if (selectY + 1 === this.board.length) {
      collum.pop()
    }
    return row
      .map((x: number): Cell[] => {
        return collum.map(
          (y: number): Cell => {
            return { x, y, color: this.board[y][x] }
          }
        )
      })
      .flat()
      .filter(
        (cell: Cell): boolean => !(cell.x === selectX && cell.y === selectY)
      )
  }

  /* 選択したセルが端か判定する
  getSelectedAroundCells = (selectIndex: number, pointers: number[]): number[] => {
  } */

  canPut = (selectX: number, selectY: number, selectColor: number): boolean => {
    const aroundOpponentCellsInfo: Cell[] = this.getAroundOpponentCellsInfo(
      selectX,
      selectY,
      selectColor
    )
    const selectCell: Cell = {
      x: selectX,
      y: selectY,
      color: this.board[selectY][selectX]
    }
    return (
      aroundOpponentCellsInfo.length > 0 &&
      selectCell.color === CellTypeNumberEnum.NONE
    )
  }

  onClick(x: number, y: number) {
    if (this.canPut(x, y, this.currentColor)) {
      this.$set(this.board[y], x, this.currentColor)
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
  color: #f00;
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
