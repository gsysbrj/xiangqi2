<script setup>
import { reactive } from 'vue';
import QiZi from './QiZi.vue'

// 棋子信息
class QiZiInfo {
  color = 'black'
  name = ''
  location = [-1 , -1]
  picked = false
}

// 局面信息
class Situation {
  qiZiInfoList = []
}

const state = reactive({
  pickedLocation: [-1, -1],
  situation: {
    qiZiInfoList: [
      {name: '車', location: [0, 0], },
      {name: '車', location: [0, 8], },
      {name: '馬', location: [0, 1], },
      {name: '馬', location: [0, 7], },
      {name: '象', location: [0, 2], },
      {name: '象', location: [0, 6], },
      {name: '士', location: [0, 3], },
      {name: '士', location: [0, 5], },
      {name: '將', location: [0, 4], },
      {name: '炮', location: [2, 1], },
      {name: '炮', location: [2, 7], },
      {name: '卒', location: [3, 0], },
      {name: '卒', location: [3, 2], },
      {name: '卒', location: [3, 4], },
      {name: '卒', location: [3, 6], },
      {name: '卒', location: [3, 8], },
      {color: 'red', name: '車', location: [9, 0], },
      {color: 'red', name: '車', location: [9, 8], },
      {color: 'red', name: '馬', location: [9, 1], },
      {color: 'red', name: '馬', location: [9, 7], },
      {color: 'red', name: '相', location: [9, 2], },
      {color: 'red', name: '相', location: [9, 6], },
      {color: 'red', name: '仕', location: [9, 3], },
      {color: 'red', name: '仕', location: [9, 5], },
      {color: 'red', name: '帥', location: [9, 4], },
      {color: 'red', name: '炮', location: [7, 1], },
      {color: 'red', name: '炮', location: [7, 7], },
      {color: 'red', name: '兵', location: [6, 0], },
      {color: 'red', name: '兵', location: [6, 2], },
      {color: 'red', name: '兵', location: [6, 4], },
      {color: 'red', name: '兵', location: [6, 6], },
      {color: 'red', name: '兵', location: [6, 8], },
    ],
  }
})

function pick(loc) {
  if (isPicked(loc)) {
    state.pickedLocation[0] = -1
    state.pickedLocation[1] = -1
    return;
  }
  state.pickedLocation[0] = loc[0]
  state.pickedLocation[1] = loc[1]
}

function isPicked(loc) {
  return state.pickedLocation[0] === loc[0] && state.pickedLocation[1] === loc[1]
}
</script>
<template>
  <div class="qipan">
    <div>
      <div v-for="r in 10" :class="`row row-${ r }`">
        <div v-for="c in 9" :class="`cell cell-${ c }`">
          <div class="line-x"></div>
          <div class="line-y"></div>
        </div>
      </div>
    </div>
    <QiZi v-for="q in state.situation.qiZiInfoList" 
      :color="q.color" 
      :name="q.name" 
      :location="q.location.slice()"
      :picked="isPicked(q.location)"
      @mousedown="pick(q.location)"
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
