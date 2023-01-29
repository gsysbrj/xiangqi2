<script setup>
import { reactive } from 'vue';
import QiZi from './QiZi.vue'

// 棋子信息
class QiZiInfo {
  color = ''
  name = ''
  location = [-1 , -1]
  picked = false

  constructor(color, name, location, picked = false) {
    this.color = color
    this.name = name
    this.location = location
    this.picked = picked
  }
}

// 局面信息
class Situation {
  qiZiInfoList = []
}

const state = reactive({
  picked: null,
  situation: {
    qiZiInfoList: [
      new QiZiInfo('黑', '車', [0, 0]),
      new QiZiInfo('黑', '車', [0, 8]),
      new QiZiInfo('黑', '馬', [0, 1]),
      new QiZiInfo('黑', '馬', [0, 7]),
      new QiZiInfo('黑', '象', [0, 2]),
      new QiZiInfo('黑', '象', [0, 6]),
      new QiZiInfo('黑', '士', [0, 3]),
      new QiZiInfo('黑', '士', [0, 5]),
      new QiZiInfo('黑', '將', [0, 4]),
      new QiZiInfo('黑', '炮', [2, 1]),
      new QiZiInfo('黑', '炮', [2, 7]),
      new QiZiInfo('黑', '卒', [3, 0]),
      new QiZiInfo('黑', '卒', [3, 2]),
      new QiZiInfo('黑', '卒', [3, 4]),
      new QiZiInfo('黑', '卒', [3, 6]),
      new QiZiInfo('黑', '卒', [3, 8]),
      new QiZiInfo('红', '車', [9, 0]),
      new QiZiInfo('红', '車', [9, 8]),
      new QiZiInfo('红', '馬', [9, 1]),
      new QiZiInfo('红', '馬', [9, 7]),
      new QiZiInfo('红', '相', [9, 2]),
      new QiZiInfo('红', '相', [9, 6]),
      new QiZiInfo('红', '仕', [9, 3]),
      new QiZiInfo('红', '仕', [9, 5]),
      new QiZiInfo('红', '帥', [9, 4]),
      new QiZiInfo('红', '炮', [7, 1]),
      new QiZiInfo('红', '炮', [7, 7]),
      new QiZiInfo('红', '兵', [6, 0]),
      new QiZiInfo('红', '兵', [6, 2]),
      new QiZiInfo('红', '兵', [6, 4]),
      new QiZiInfo('红', '兵', [6, 6]),
      new QiZiInfo('红', '兵', [6, 8]),
    ],
  }
})

function pick(q) {
  // 拿起后放下棋子
  if (isPicked(q)) {
    state.picked = null
    return
  }
  // 吃子
  if (state.picked && state.picked.color !== q.color) {
    if (isMoveCheckOk(q.location[0], q.location[1])) {
      state.picked.location[0] = q.location[0]
      state.picked.location[1] = q.location[1]
      state.picked = null
      q.location[0] = -100
      q.location[1] = -100
    } else {
      console.warn('吃子违反规则!')
    }
    return
  }
  // 拿起棋子
  state.picked = q
}

function isPicked(q) {
  return q === state.picked
}

function move(r, c) {
  if (state.picked) {
    if (isMoveCheckOk(r, c)) {
      state.picked.location[0] = r
      state.picked.location[1] = c
      state.picked = null
    } else {
      console.warn('走子违反规则！')
    }
  }
}
// 各种棋子走动时是否符合规则的判断逻辑
function isMoveCheckOk(r, c) {
  if (!state.picked) return false
  const p = state.picked
  if (p.name === '兵') {
    if (p.location[0] < 5) { // 过河兵
      return p.location[0] === r && (p.location[1] + 1 === c || p.location[1] - 1 === c) || // 可以拐弯一步
      p.location[0] === r + 1 && p.location[1] === c // 或往前一步
    } else { // 非过河兵
      return p.location[0] === r + 1 && p.location[1] === c // 只能往前一步
    }
  } else if (p.name === '卒') {
    if (p.location[0] > 4) { // 过河卒
      return p.location[0] === r && (p.location[1] + 1 === c || p.location[1] - 1 === c) || // 可以拐弯一步
      p.location[0] === r - 1 && p.location[1] === c // 或往前一步
    } else { // 非过河卒
      return p.location[0] === r - 1 && p.location[1] === c // 只能往前一步
    }
  } else {

  }
  return false
}
</script>
<template>
  <div class="qipan">
    <div>
      <div v-for="r in 10" :class="`row row-${ r - 1 }`">
        <div v-for="c in 9" :class="`cell cell-${ c - 1 }`" @mousedown="move(r - 1, c - 1)">
          <div class="line-x"></div>
          <div class="line-y"></div>
        </div>
      </div>
    </div>
    <QiZi v-for="q in state.situation.qiZiInfoList" 
      :color="q.color" 
      :name="q.name" 
      :location="q.location.slice()"
      :picked="isPicked(q)"
      @mousedown="pick(q)"
      ></QiZi>
  </div>
</template>
<style scoped lang="less">
@line-color: blue;
.qipan {
  position: relative;
}
.row {
  display: flex;
}
.cell {
  /* outline:1px dotted green; */
  width: 3rem;
  height: 3rem;
  position: relative;
}
.cell .line-x {
  height: 1px;
  width: 100%;
  background-color: @line-color;
  position: absolute;
  top: 50%;
}
.cell .line-y {
  width: 1px;
  height: 100%;
  background-color: @line-color;
  position: absolute;
  left: 50%;
}
.row:first-child .cell .line-y {
  height: 50%;
  top: 50%;
}
.row:last-child .cell .line-y {
  height: 50%;
}
.cell:first-child .line-x {
  width: 50%;
  left: 50%;
}
.cell:last-child .line-x {
  width: 50%;
}
.row:nth-child(5) .cell:not(:first-child):not(:last-child) .line-y {
  height: 50%;
}
.row:nth-child(6) .cell:not(:first-child):not(:last-child) .line-y {
  height: 50%;
  top: 50%;
}
</style>
