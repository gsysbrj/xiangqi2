<script setup>
</script>
<template>
  <div>
    <div v-for="r in 10" :class="`row row-${ r }`">
      <div v-for="c in 9" :class="`cell cell-${ c }`">
        <div class="line-x"></div>
        <div class="line-y"></div>
      </div>
    </div>
  </div>
</template>
<style scoped lang="less">
@line-color: blue;
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
