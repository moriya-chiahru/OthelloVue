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
        <!-- x:{{ x - 1 }}y:{{ y - 1 }} -->
        <!-- <div v-else>x:{{ x - 1 }}y:{{ y - 1 }}</div> -->
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
type Cell = {
  x: number
  y: number
  color?: number
  direction?:
    | 'center'
    | 'north'
    | 'northeast'
    | 'east'
    | 'southeast'
    | 'south'
    | 'southwest'
    | 'west'
    | 'northwest'
}
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
    [0, 0, 0, 1, 2, 0, 0, 0],
    [0, 0, 0, 2, 1, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0]
  ]

  get currentStone(): CellTypeEnum {
    return this.currentColor === 1 ? CellTypeEnum.WHITE : CellTypeEnum.BLACK
  }

  directionList: Cell[] = [
    { x: 0, y: -1, direction: 'north' },
    { x: 1, y: -1, direction: 'northeast' },
    { x: 1, y: 0, direction: 'east' },
    { x: 1, y: 1, direction: 'southeast' },
    { x: 0, y: 1, direction: 'south' },
    { x: -1, y: 1, direction: 'southwest' },
    { x: -1, y: 0, direction: 'west' },
    { x: -1, y: -1, direction: 'northwest' }
  ]

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
    return this.directionList.map(
      (direction: Cell): Cell => {
        const x: number = selectX + direction.x
        const y: number = selectY + direction.y
        return {
          x,
          y,
          color:
            this.getSelectedConner(x) && this.getSelectedConner(y)
              ? this.board[y][x]
              : 0,
          direction: direction.direction
        }
      }
    )
  }

  /* 選択したセルが端か判定する */
  getSelectedConner = (position: number): boolean => {
    if (position === -1 || position === 8) {
      return false
    }
    return true
  }

  /* 裏返すセルを返す */
  getReturnCells = (
    aroundOpponentCellsInfos: Cell[],
    selectColor: number
  ): Cell[] => {
    return aroundOpponentCellsInfos
      .map((aroundOpponentCellsInfo: Cell): Cell[] => {
        const returnCells: Cell[] = [aroundOpponentCellsInfo]
        // ループする方向を得る
        const direction: Cell | undefined = this.directionList.find(
          (directionList: Cell): boolean =>
            directionList.direction === aroundOpponentCellsInfo.direction
        )

        if (direction) {
          let searchX: number = aroundOpponentCellsInfo.x + direction.x
          let searchY: number = aroundOpponentCellsInfo.y + direction.y
          // 端までループし状態を確認
          while (
            this.getSelectedConner(searchX) &&
            this.getSelectedConner(searchY) &&
            this.board[searchY][searchX] !== CellTypeNumberEnum.NONE
          ) {
            const getCellColor: number = this.board[searchY][searchX]
            returnCells.push({
              x: searchX,
              y: searchY,
              color: getCellColor,
              direction: direction.direction
            })
            // 同じ色のセルまで行き着いたら抜ける
            if (getCellColor === selectColor) {
              break
            }
            // 位置情報を更新
            searchX += direction.x
            searchY += direction.y
          }
        }
        // 最後のセルが同じ色ではなかったら、空を返す
        const endReturnCell: Cell | undefined = returnCells.pop()
        return endReturnCell && endReturnCell.color === selectColor
          ? returnCells
          : []
      })
      .flat()
  }

  canPut = (
    selectX: number,
    selectY: number,
    aroundOpponentCellsInfos: Cell[]
  ): boolean => {
    return (
      aroundOpponentCellsInfos.length > 0 &&
      this.board[selectY][selectX] === CellTypeNumberEnum.NONE
    )
  }

  onClick(x: number, y: number) {
    const aroundOpponentCellsInfos: Cell[] = this.getAroundOpponentCellsInfo(
      x,
      y,
      this.currentColor
    )
    const returnCells: Cell[] = this.getReturnCells(
      aroundOpponentCellsInfos,
      this.currentColor
    )

    if (this.canPut(x, y, returnCells)) {
      // 現在の位置を追加
      returnCells.unshift({
        x,
        y,
        color: this.currentColor,
        direction: 'center'
      })
      // セルを更新
      this.currentColor = 3 - this.currentColor
      returnCells.forEach((returnCell: Cell) => {
        this.$set(this.board[returnCell.y], returnCell.x, 3 - this.currentColor)
      })
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
