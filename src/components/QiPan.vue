<script setup>
import _ from 'lodash'
import { reactive, ref } from 'vue';
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

// 棋谱
class Manual {
  moves = []
}

class Move {
  qz = null
  to = [-1, -1]
  oldSituation = []
  newSituation = []
  desc = ''
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
  },
  manual: {
    moves: [],
  },
})

function back() {
  const move = state.manual.moves.pop()
  if (move) {
    state.situation = move.oldSituation
  }
}

const scrollBoxManual = ref(null)

function pick(q) {
  // 拿起后放下棋子
  if (isPicked(q)) {
    state.picked = null
    return
  }
  // 吃子
  if (state.picked && state.picked.color !== q.color) {
    if (isMoveCheckOk(q.location[0], q.location[1])) {
      const old = _.cloneDeep(state.situation)
      const picked = _.cloneDeep(state.picked)
      state.picked.location[0] = q.location[0]
      state.picked.location[1] = q.location[1]
      state.picked = null // 方向棋子
      recordManual(picked, q.location, old, _.cloneDeep(state.situation))
      // 把被吃棋子移出棋盘
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

function recordManual(picked, to, oldSituation, newSituation) {
  state.manual.moves.push({
    qz: picked,
    to: to,
    oldSituation: oldSituation,
    newSituation: newSituation,
    desc: moveDesc(oldSituation, picked, to),
  })
  setTimeout(() => {
    scrollBoxManual.value.scrollTo(0, scrollBoxManual.value.scrollHeight)
  }, 0)
}

function move(r, c) {
  if (state.picked) {
    if (isMoveCheckOk(r, c)) {
      const old = _.cloneDeep(state.situation)
      const picked = _.cloneDeep(state.picked)
      state.picked.location[0] = r
      state.picked.location[1] = c
      state.picked = null
      recordManual(picked, [r, c], old, _.cloneDeep(state.situation))
      // state.manual.moves.push({
      //   qz: picked,
      //   to: [r, c],
      //   oldSituation: old,
      //   newSituation: _.cloneDeep(state.situation),
      //   desc: moveDesc(old, picked, [r, c]),
      // })
      // setTimeout(() => {
      //   scrollBoxManual.value.scrollTo(0, scrollBoxManual.value.scrollHeight)
      // }, 0)
    } else {
      console.warn('走子违反规则！')
    }
  }
}
function moveDesc(oldSituation, qz, to) {
  const name = qz.name
  let colFrom = ''
  let colTo = ''
  let direction = ''
  let offset = ''
  if (qz.color === '红') {
    colFrom = '一二三四五六七八九'.charAt(8 - qz.location[1])
    colTo = '一二三四五六七八九'.charAt(8 - to[1])
    if (to[0] < qz.location[0]) {
      direction = '进'
      offset = '一二三四五六七八九'.charAt(qz.location[0] - to[0] - 1)
    } else if (to[0] > qz.location[0]) {
      direction = '退'
      offset = '一二三四五六七八九'.charAt(to[0] - qz.location[0] - 1)
    } else {
      direction = '平'
    }
  } else {
    colFrom = '１２３４５６７８９'.charAt(qz.location[1])
    colTo = '１２３４５６７８９'.charAt(to[1])
    if (to[0] > qz.location[0]) {
      direction = '进'
      offset = '１２３４５６７８９'.charAt(to[0] - qz.location[0] - 1)
    } else if (to[0] < qz.location[0]) {
      direction = '退'
      offset = '１２３４５６７８９'.charAt(qz.location[0] - to[0] - 1)
    } else {
      direction = '平'
    }
  }
  // 棋子前后判断
  if (name === '車' ||
      name === '馬' ||
      name === '炮' ||
      name === '兵' ||
      name === '卒'
  ) {
    // 找出与选中棋子处于相同列的其他棋子
    const sameQzs = oldSituation.qiZiInfoList.filter(q => q.color === qz.color && q.name === qz.name && q.location[1] === qz.location[1])
    if (sameQzs.length > 1) {
      if (qz.color === '黑') { // 从大到小排序
        sameQzs.sort((a, b) => - a.location[0] + b.location[0])
      } else { // 从小到大排序
        sameQzs.sort((a, b) => a.location[0] - b.location[0])
      }
      const index = sameQzs.findIndex(e => e.location[0] === qz.location[0] && e.location[1] === qz.location[1])
      let prefix = ''
      if (sameQzs.length === 2) {
        if (index === 0) {
          prefix = '前' + name
        } else {
          prefix = '后' + name
        }
      } else if (sameQzs.length === 3) {
        if (index === 0) {
          prefix = '前' + name
        } else if (index === 1) {
          prefix = '中' + name
        } else {
          prefix = '后' + name
        }
      } else if (sameQzs.length === 4) {
        if (index === 0) {
          prefix = '前' + name
        } else if (index === 1) {
          prefix = '二' + name
        } else if (index === 2) {
          prefix = '三' + name
        } else {
          prefix = '四' + name
        }
      } else if (sameQzs.length === 5) {
        if (index === 0) {
          prefix = '前' + name
        } else if (index === 1) {
          prefix = '二' + name
        } else if (index === 2) {
          prefix = '三' + name
        } else if (index === 3) {
          prefix = '四' + name
        } else {
          prefix = '五' + name
        }
      }
      // 两列兵卒的情况------实战中极其少见
      if (name === '兵' || name === '卒') {
        if (sameQzs.length === 2 || sameQzs.length === 3) {
          // 如果其他任意一列上有2个以上兵（卒），即为 2列 多兵（卒） 的情况
          let flag = false
          for (let i = 0; i < 9; i++) {
            if (i === qz.location[1]) { // 越过当前列，只检查其他列
              continue
            }
            const otherSameQzs = oldSituation.qiZiInfoList.filter(q => q.color === qz.color && q.name === qz.name && q.location[1] === i)
            if (otherSameQzs.length > 1) {
              flag = true
              break;
            }
          }
          if (flag) {
            prefix = prefix.slice(0, 1) + colFrom
          }
        }
      }
      if (name === '馬' || direction === '平') {
        return prefix + direction + colTo
      } else {
        return prefix + direction + offset
      }
    }
  }
  if (direction === '平') {
    return name + colFrom + direction + colTo
  }
  if (name === '馬' || name === '相' || name === '象' || name === '士' || name === '仕') {
    return name + colFrom + direction + colTo
  }

  return name + colFrom + direction + offset
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
  <div class="qipan-wrapper">
    <div class="qipan">
      <div class="column-index column-index-black">
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
        <div>5</div>
        <div>6</div>
        <div>7</div>
        <div>8</div>
        <div>9</div>
      </div>
      <div class="column-index column-index-red">
        <div>一</div>
        <div>二</div>
        <div>三</div>
        <div>四</div>
        <div>五</div>
        <div>六</div>
        <div>七</div>
        <div>八</div>
        <div>九</div>
      </div>
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
    <div class="manual">
      <div style="padding-left: 7px;">棋谱：</div>
      <div class="moves" ref="scrollBoxManual">
        <div class="move" v-for="(m, i) in state.manual.moves">
          <span class="index">
            <template v-if="i % 2 === 0">{{ i/2+1 }}.</template>
          </span>
          <span>{{ m.desc }}</span>
        </div>
      </div>
      <div class="ops">
        <button @click="back()">后退</button>
      </div>
    </div>
  </div>
</template>
<style scoped lang="less">
@line-color: blue;
.qipan-wrapper {
  display: flex;
}
.qipan {
  position: relative;
}
.column-index {
  position: absolute;
  display: flex;
  & > div {
    width: 3rem;
    height: 1.5rem;
  }
  &-black {
    top: -1.5rem;
  }
  &-red {
    bottom: -1.5rem;
    flex-flow: row-reverse;
  }
}
.manual {
  text-align: left;
}
.moves {
  text-align: left;
  font-size: 14px;
  line-height: normal;
  box-sizing: border-box;
  height: 27rem;
  width: 9rem;
  overflow-y: scroll;
  .index {
    display: inline-block;
    width: 22px;
    padding-right: 2px;
    text-align: right;
  }
}
.ops {
  button {
    height: 1.5rem;
    font-size: 12px;
    line-height: 1.5rem;
    padding: 0 7px;
    margin: 0 5px;
  }
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
