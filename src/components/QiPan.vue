<script setup>
import { reactive } from 'vue';
import QiZi from './QiZi.vue'

// 棋子信息
class QiZiInfo {
  color = ''
  name = ''
  location = [-1 , -1]

  constructor(color, name, location) {
    this.color = color
    this.name = name
    this.location = location
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
  } else if (p.name === '炮') {
    // 只能直着走或者横着走，与目标之间最多只能有一个棋子
    const qzBt = []
    if (p.location[0] === r) { // 横着走
      for (const qz of state.situation.qiZiInfoList) {
        if (qz.location[0] === r && betweenInclude(qz.location[1], p.location[1], c)) {
          qzBt.push(qz)
        }
      }
      // qiZiInfoList中的棋子的顺序是不确定的，因此要根据棋子的位置排序，以便后面比较
      // 横着走时根据纵坐标排序；
      qzBt.sort((a, b) => a.location[1] - b.location[1])
    } else if(p.location[1] === c) { // 竖着走
      for (const qz of state.situation.qiZiInfoList) {
        if (qz.location[1] === c && betweenInclude(qz.location[0], p.location[0], r)) {
          qzBt.push(qz)
        }
      }
      // 竖着走时根据横坐标排序
      qzBt.sort((a, b) => a.location[0] - b.location[0])
    } else {
      // 不能斜着走
      return false
    }
    if (qzBt.length === 1) { // 之间除了自己没有其他棋子，即为普通走子
      return true
    }
    if (qzBt.length === 3) { // 之间包含自己和目标共3个棋子，即为翻山吃子，则目标必须为对方棋子
      // return qzBt[0] === p && qzBt[2].color !== p.color || qzBt[2] === p && qzBt[0].color !== p.color
      return true
    }
  } else if (p.name === '馬') {
    if (Math.abs(p.location[0] - r) === 1 && Math.abs(p.location[1] - c) === 2 || 
        Math.abs(p.location[0] - r) === 2 && Math.abs(p.location[1] - c) === 1) {
        // 蹩马腿判断
        if (Math.abs(p.location[1] - c) === 2) {
          if (state.situation.qiZiInfoList.some(qz => qz.location[0] === p.location[0] && between(qz.location[1], p.location[1], c))) {
            console.warn('蹩马腿了！')
            return false
          }
        } else {
          if (state.situation.qiZiInfoList.some(qz => qz.location[1] === p.location[1] && between(qz.location[0], p.location[0], r))) {
            console.warn('蹩马腿了！')
            return false
          }
        }
      return true
    }
  } else if (p.name === '車') {
    // 只能直着走或者横着走，与目标之间最多只能有0个棋子
    const qzBt = []
    if (p.location[0] === r) { // 横着走
      for (const qz of state.situation.qiZiInfoList) {
        if (qz.location[0] === r && between(qz.location[1], p.location[1], c)) {
          qzBt.push(qz)
        }
      }
    } else if(p.location[1] === c) { // 竖着走
      for (const qz of state.situation.qiZiInfoList) {
        if (qz.location[1] === c && between(qz.location[0], p.location[0], r)) {
          qzBt.push(qz)
        }
      }
    } else {
      // 不能斜着走
      return false
    }
    if (qzBt.length === 0) { // 之间除了自己没有其他棋子，即为普通走子
      return true
    }
  } else if (p.name === '相' || p.name === '象') { // 只能走田字格
    if (Math.abs(p.location[0] - r) === 2 && Math.abs(p.location[1] - c) === 2) {
      // 不能被塞了象眼
      if (state.situation.qiZiInfoList.some(e => e.location[0] === (p.location[0] + r)/2 &&
                                                   e.location[1] === (p.location[1] + c)/2)
      ) {
        console.warn('象眼被塞！')
        return false
      }
      // 不能过河
      if (p.name === '相' && r < 5) {
        return false
      }
      if (p.name === '象' && r > 4) {
        return false
      }
      return true
    }
  } else if (p.name === '士' || p.name === '仕') {
    if (Math.abs(p.location[0] - r) === 1 && Math.abs(p.location[1] - c) === 1) {
      if (p.name === '仕' && (r < 7 || c < 3 || c > 5)) { // 不能出九宫格
        return false
      }
      if (p.name === '士' && (r > 2 || c < 3 || c > 5)) { // 不能出九宫格
        return false
      }
      return true
    }
  } else { // 将帅只能在九宫格内走一步直线
    if (p.location[0] === r && Math.abs(p.location[1] - c) === 1 ||
        p.location[1] === c && Math.abs(p.location[0] - r) === 1
    ) {
      if (p.name === '帥' && (r < 7 || c < 3 || c > 5)) { // 不能出九宫格
        return false
      }
      if (p.name === '將' && (r > 2 || c < 3 || c > 5)) { // 不能出九宫格
        return false
      }
      return true
    }
  }
  return false
}

// 判断a是否在b和c之间
function betweenInclude (a, b, c) {
  return a >= b && a <= c || a <= b && a >= c
}
function between (a, b, c) {
  return a > b && a < c || a < b && a > c
}

</script>
<template>
  <div class="qipan">
    <div class="diagonal diagonal-1"></div>
    <div class="diagonal diagonal-2"></div>
    <div class="diagonal diagonal-3"></div>
    <div class="diagonal diagonal-4"></div>
    <div class="locations">
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
// .locations {
//   position: relative;
//   top: 0;
//   left: 0;
// }
.diagonal {
  height: 1px;
  background-color: @line-color;
  width: 8.485281374238571rem;
  position: absolute;
  transform-origin: 0px 0px;

  &-1 {
    top: 1.5rem;
    left: 10.5rem;
    transform: rotate(45deg) scaleY(0.5);
  }
  &-2 {
    top: 7.5rem;
    left: 10.5rem;
    transform: rotate(-45deg) scaleY(0.5);
  }
  &-3 {
    top: 22.5rem;
    left: 10.5rem;
    transform: rotate(45deg) scaleY(0.5);
  }
  &-4 {
    top: 28.5rem;
    left: 10.5rem;
    transform: rotate(-45deg) scaleY(0.5);
  }
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
