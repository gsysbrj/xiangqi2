<script setup>
defineProps({
    color: String,
    name: String,
    location: Array,
    picked: Boolean,
})
</script>
<template>
    <div class="qizi" :class="{
        red: color === '红',
        picked: picked === true
    }" :style="{
        top: `${picked ? location[0] * 3 - 0.1 : location[0] * 3 + 0.1}rem`,
        left: `${location[1] * 3 + 0.1}rem`,
    }">
        <span>{{ name }}</span>
    </div>
</template>
<style scoped lang="less">
.qizi {
    height: 2.8rem;
    width: 2.8rem;
    display: flex;
    justify-content: center;
    align-items: center;
    border: 1px solid;
    border-radius: 50%;
    font-size: 24px;
    user-select: none;
    background-color: wheat;
    position: absolute;

    &.red {
        border-color: red;
        color: red;
    }
    &.picked {
        box-shadow: 0 3px 2px 1px black;
    }
}
</style>
