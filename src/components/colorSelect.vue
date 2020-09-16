<template>
  <div class="color-select-content">
    <span class="color-item"
          :class="{active:select==item}"
          :style="{backgroundColor:item}"
          v-for="(item,index) in colorList"
          :key="index"
          @click="colorClick(item)"></span>
  </div>
</template>

<script>
export default {
  name: "colorSelect",
  model:{
    prop:'select',
    event:'change'
  },
  props:{
    select:String,
    colorList:{
      type:Array,
      required:true
    }
  },
  methods:{
    colorClick(index){
      this.$emit('change',index)
    }
  }
}
</script>

<style scoped lang="less">
  .color-item{
    position: relative;
    width: 30px;
    height: 30px;
    margin: 9px;
    border-radius: 50%;
    transition-property: width, height, margin, border-radius;
    transition-duration: .25s;
    transition-timing-function: linear;
    cursor: pointer;
    z-index: 0;
    display: inline-block;
    &:before{
      content: " ";
      position: absolute;
      top: -5px;
      left: -5px;
      right: -5px;
      bottom: -5px;
      border: 1px solid #e5e5e5;
      border-radius: 2px;
      transition: margin .25s linear, border-color .25s linear;
      z-index: -1;
    }
    &.active{
      width: 40px;
      height: 40px;
      margin: 4px;
      border-radius: 2px;
      &:before{
        margin: 4px;
        border-color: transparent;
      }
    }
  }
</style>
