<template>
  <!--左右两栏,采用flex布局-->
  <div class="goods">
    <!--左侧菜单栏-->
    <div class="menu-wrapper" ref="menuWrapper">
      <!--通过ref拿到该原生DOM-->
      <ul>
        <li v-for="(item,index) in goods" class="menu-item" :class="{'current':currentIndex===index}"
          @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!--右侧食品栏-->
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="food in item.foods" class="food-item border-1px" @click="selectFood(food,$event)">
              <div class="icon">
                <img :src="food.icon" width="57" height="57">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span>
                  <span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="addFood" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <!--底部购物栏-->
    <shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice"
      :min-price="seller.minPrice"></shopcart>
    <!--商品详情层-->
    <food :food="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
import BScroll from 'better-scroll'
import shopcart from '../shopcart/shopcart'  //  引用组件
import cartcontrol from '../cartcontrol/cartcontrol'  //  引用组件
import food from '../food/food'  //  引用组件

const ERR_OK = 0

export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0,  //  scrollY右边每个区域的高度
      selectedFood: {}
    }
  },
  computed: {
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 =this.listHeight[i]
        let height2 =this.listHeight[i+1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {  //  如果scollY不是最后一个或者在这个区间
          return i  //  返回索引
        }
      }
      return 0
    },
    selectFoods() {
      let foods = []
      //  遍历商品分类
      this.goods.forEach((good) =>{
      //  遍历分类下的商品
        good.foods.forEach((food) => {
      //  如果food.count存在，即大于0，证明商品被选中，将被选中商品放进数组里
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  created() {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']

    this.$http.get('/api/goods').then((response) => {
      response = response.body
      if (response.errno === ERR_OK) {
        this.goods = response.data
        this.$nextTick(() => {  //  想要计算与DOM相关的东西时就调用该函数
          this._initScroll()
          this._calculateHeight()
        })
      }
    })
  },
  methods: {
    selectFood(food, event) {
      if (!event._constructed) {
        return
      } 
      this.selectedFood = food
      this.$refs.food.show()
    },
    selectMenu(index, event) {  //  实现点击左侧时，右侧跳到相应位置
      if (!event._constructed) {
        return
      }
      let foodlist = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let el = foodlist[index]
      this.foodsScroll.scrollToElement(el, 300)
      console.log(index)
    },
    _initScroll() {  //  让右边物品栏实现滚动
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {  //  通过ref
        click: true
      })
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3  //  探针效果，当scroll滚动时能实时告诉滚动的位置
      })
      this.foodsScroll.on('scroll', (pos) => {  // .on实现监听事件，监听scroll,回调函数参数为pos
        this.scrollY = Math.abs(Math.round(pos.y))  //  roud取整 abs取正数
      })
    },
    _calculateHeight() {  //  计算右边物品栏每个类区间的高度，以便通过高度来绑定左侧对应区域
      let foodlist = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodlist.length; i++) {
        let item = foodlist[i]
        height += item.clientHeight  //  通过celientHeight拿到每个dom高度，再累加计算总高度
        this.listHeight.push(height)
      }
    },
    addFood(target) {
      this._drop(target)
    },
    _drop(target) {  //  _drop方法(父组件)调用(子组件)shopcart.vue里的drop方法
      this.$refs.shopcart.drop(target)
    }
  },
  components: {  // 注册组件
    shopcart,
    cartcontrol,
    food
  },
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin.styl"

.goods
  display: flex
  position: absolute
  top: 174px  //  header为134,tab为40
  bottom: 46px  //  下端购物篮的高度
  width: 100%
  overflow: hidden
  .menu-wrapper  //  左侧菜单栏
    flex: 0 0 80px 
    width: 80px
    background: #f3f5f7
    .menu-item
      display: table  //  让左侧菜单栏的文字垂直居中
      height: 54px
      width: 56px
      padding: 0 12px
      line-height: 14px
      &.current
        position: relative
        z-index: 10
        margin-top: -1px
        background: #fff
        font-weight: 700
        .text
          border-none()
      .icon
        display: inline-block
        vertical-align: top
        width: 12px
        height: 12px
        margin-right: 2px
        background-size: 12px 12px
        background-repeat: no-repeat
        &.decrease
          bg-image('decrease_3')
        &.discount
          bg-image('discount_3')
        &.guarantee
          bg-image('guarantee_3')
        &.invoice
          bg-image('invoice_3')
        &.special
          bg-image('special_3')
      .text
        display: table-cell
        width: 56px
        vertical-align: middle
        border-1px(rgba(7, 17, 27, 0.1))
        font-size:12px
  .foods-wrapper
    flex: 1
    .title
      padding-left: 14px
      height: 26px
      line-height: 26px
      border-left: 2px solid #d9dde1
      font-size: 12px
      color: rgb(147, 153, 159)
      background: #f3f5f7
    .food-item
      display: flex
      margin: 18px
      padding-bottom: 18px
      border-1px(rgba(7, 17, 27, 0.1))
      &:last-child
        border-none()
        margin-bottom: 0
      .icon
        flex: 0 0 57px
        margin-right: 10px
      .content
        flex: 1
        .name
          margin: 2px 0 8px 0
          height: 14px
          line-height: 14px
          font-size: 14px
          color: rgb(7, 17, 27)
        .desc
          margin-bottom: 8px
          line-height: 12px
          font-size: 10px
          color: rgb(147, 153, 159)
        .extra
          line-height: 10px
          font-size: 10px
          color: rgb(147, 153, 159)
          &.count
            margin-right: 12px
        .price
          font-weight: 700
          line-height: 24px
          .now
            margin-right: 8px
            font-size: 14px
            color: rgb(240, 20, 20)
          .old
            text-decoration: line-through
            font-size: 10px
        .cartcontrol-wrapper
          position: absolute
          right:0
          bottom: 12px
</style>
