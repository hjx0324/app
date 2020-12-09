<template>
  <div class="cartcontrol">
    <!--小球滚动动画-->
    <transition name="move">
      <div class="cart-decrease" v-show="food.count>0" @click.stop.prevent="decreaseCart">
        <!--inner 平移 inner1 滚动 -->
        <span class="inner inner1 icon-remove_circle_outline"></span>
      </div>
    </transition>
    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>

<script type="text/ecmascript-6">
import Vue from 'vue'

export default {
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    addCart(event) {
      if (!event._constructed) {  //  让pc端点击只有一次效果
        return
      }
      if (!this.food.count) {
        Vue.set(this.food, 'count', 1)  // 用Vue将this.foo.count设置为1
      } else {
          this.food.count++
      }
      this.$emit('add', event.target)  //  将DOM对象作为事件的参数传递出去，用于计算小球的位置
    },
    decreaseCart(event) {
      if (!event._constructed) {  //  让pc端点击只有一次效果
        return
      }
      if (this.food.count) {
        this.food.count--
      }
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
.cartcontrol
  font-size: 0
  .cart-decrease
    display: inline-block
    padding: 6px
    opacity: 1
    transform: translate3d(0, 0, 0)
    .inner
      display: inline-block
      line-height: 24px
      font-size: 24px
      color: rgb(0, 160, 220)
      transition: all 0.4s linear
      transform: rotate(0)
    &.move-enter-active, &.move-leave-active
      transition: all 0.4s linear
    &.move-enter, &.move-leave-active
      opacity: 0
      transform: translate3d(24px, 0, 0)
      .inner1
       transform: rotate(90deg)
  .cart-count
    display: inline-block
    vertical-align: top
    width: 12px
    padding-top: 6px
    line-height: 24px
    text-align: center
    font-size: 10px
    color: rgb(147, 153, 159)
  .cart-add
    display: inline-block
    padding: 6px
    line-height: 24px
    font-size: 24px
    color: rgb(0, 160, 220)
</style>