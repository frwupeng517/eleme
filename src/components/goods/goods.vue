<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menu">
      <ul>
        <li v-for="(item, index) in goods" class="menu-item" :class="{'current': currentIndex === index}" @click="selectMenu(index, $event)">
          <span class="text border-1px">
            <span v-show="item.type > 0" class="icon" :class="classMap"></span>{{item.name}}</span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foods">
      <ul>
        <li class="food-list food-list-hook" v-for="item in goods">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li class="food-item border-1px" @click="selectFood(food, $event)" v-for="food in item.foods">
              <div class="icon">
                <img width="57" height="57" :src="food.icon" alt="">
              </div>
              <div class="contents">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span>月售{{food.sellCound}}份</span>
                  <span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span>￥{{food.price}}</span>
                  <span v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcount-wrapper">
                  <cartcount @add="addFood" :food="food"></cartcount>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart ref="shopcart" :delivery-price="seller.deliveryPrice" :select-foods="selectFoods" :min-price="seller.minPrice"></shopcart>
    <food :food-list="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import shopcart from '../shopcart/shopcart'
  import cartcount from '../cartcount/cartcount'
  import food from '../food/food'
  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data () {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      }
    },
    components: {
      shopcart, cartcount, food
    },
    computed: {
      currentIndex () {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i]
          let height2 = this.listHeight[i + 1]
          if (!height2 || height1 <= this.scrollY < height2) {
            return i
          }
          return 0
        }
      },
      selectFoods () {
        let foods = []
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
        return foods
      }
    },
    created () {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
      this.$http.get('../../../static/data/data.json').then((response) => {
        this.goods = response.body.goods
        this.$nextTick(() => {
          this.initScroll()
          this.caclHeight()
        })
      })
    },
    methods: {
      initScroll () {
        this.foodsScroll = new BScroll(this.$refs.foods, {
          click: true
        })
        this.menuScroll = new BScroll(this.$refs.menu, { 
          click: true,
          probeType: 3
        })       
        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y))
        }) 
      },
      caclHeight () {
        let foodList = this.$refs.foods.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectMenu (index, event) {
        if (!event._constructed) {
          return
        }
        let foodList = this.$refs.foods.getElementsByClassName('food-list-hook')
        let el = foodList[index]
        this.foodsScroll.scrollToElement(el, 300) // scrollToElement(el, time, offsetX, offsetY, easing)  作用：滚动到指定的目标元素
      },
      addFood (target) {
        this.drop(target)
      },
      drop (target) {
        // 体验优化,异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target)
        })
      },
      selectFood (food, event) {
        if (!event._constructed) {
          return
        }
        this.selectedFood = food
        this.$refs.food.show()
      }
    }
  }

</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../common/scss/mixin";
  .goods{
    display: flex;
    position: absolute;
    top: 178px;
    bottom: 46px;
    width: 100%;
    overflow: hidden;
    .menu-wrapper{
      flex: 0 0 80px;
      width: 80px;
      background-color: #f3f5f7;
      .menu-item{
        display: table;
        width: 56px;
        height: 54px;
        padding: 0 12px;
        line-height: 14px;
        &.current{
          position: relative;
          z-index: 10;
          margin-top: -1px;
          background-color: #fff;
          font-weight: 700;
          .text{
            border-bottom: none;
          }
        }
        .text{
          display: table-cell;
          vertical-align: middle;
          font-size: 12px;   
          @include border-1px(rgba(7, 17, 27, 0.1));      
          .icon{
            display: inline-block;
            vertical-align: top;
            width: 12px;
            height: 12px;
            margin-right: 2px;
            background-size: 12px 12px;
            background-repeat: no-repeat;
            &.decrease{
              @include bg-img('decrease_3');
            }
            &.discount{
              @include bg-img('discount_3');
            }
            &.special{
              @include bg-img('special_3');
            }
            &.invoice{
              @include bg-img('invoice_3');
            }
            &.guarantee{
              @include bg-img('guarantee_3');
            }
            color: rgb(240, 20, 20);
          }
        }
      }
    }
    .foods-wrapper{
      flex: 1;
      .title{
        padding-left: 14px;
        height: 26px;
        line-height: 26px;
        font-size: 12px;
        color: rgb(147,153,159);
        background-color: #f3f5f7;
        border-left: 2px solid #d9dde1;
      }
      .food-item{
        display: flex;
        margin: 18px;
        padding-bottom: 18px;
        &:not(last-child){
          @include border-1px(rgba(7, 17, 27, 0.1)); 
          margin-bottom: 0; 
        }
        .icon{
          flex: 0 0 57px;
        }
        .contents{
          flex: 1;
          padding-left: 10px;
          padding-top: 2px;
          .name{
            height: 14px;
            line-height: 14px;
            font-size: 14px;
            color: rgb(7,17,27);
          }
          .desc, .extra{
            line-height: 12px;
            font-size: 10px;
            color: rgb(147,153,159);
          }
          .desc{
            margin: 8px 0;
          }
          .extra :first-child{
            margin-right: 12px;
          }
          .price{
            line-height: 24px;
            :first-child{
              margin-right: 8px;
              font-size: 14px;
              font-weight: 700;
              color: rgb(240, 20, 20);
            }
            :last-child{
              font-size: 10px;
              font-weight: 700;              
              color: rgb(147,153,159);
              text-decoration: line-through;
            }
          }
          .cartcount-wrapper{
            position: absolute;
            bottom: 12px;
            right: 0;
          }
        }
      }
    }
  }
</style>
