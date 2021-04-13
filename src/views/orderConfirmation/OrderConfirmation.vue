<template>
  <div class="wrapper">
    <div class="top">
      <div class="top__header">
        <div
          class="iconfont top__header__back"
          @click="handleBackClick"
        >&#xe6f2;</div>
        确认订单
      </div>
      <div class="top__receiver">
        <div class="top__receiver__title">收获地址</div>
        <div class="top__receiver__address">上海市杨浦区阿狸阿狸阿狸阿狸</div>
        <div class="top__receiver__info">
          <span>张渠（先生）</span>
          <span>18502113035</span>
        </div>
        <div class="iconfont top__receiver__icon">&#xe6f2;</div>
      </div>
    </div>
    <div class="products">
      <div class="products__title">
        {{shopName}}
      </div>
      <div class="products__list">
        <template
          v-for="item in productList"
          :key="item._id"
        >
          <div class="products__item" v-if="item.count > 0">
            <img class="products__item__img" :src="item.imgUrl" />
            <div class="products__item__detail">
              <h4 class="products__item__title">{{item.name}}</h4>
              <p class="products__item__price">
                <span class="products__item__single">
                  <span class="products__item__yen">&yen; </span>
                  {{item.price}} * {{item.count}}
                </span>
                <span class="products__item__totla">
                  <span class="products__item__yen">&yen; </span>
                  {{(item.price * item.count).toFixed(2)}}
                </span>
              </p>
            </div>
          </div>
        </template>
      </div>
    </div>
    <div class="order">
      <div class="order__price">实付金额 ¥{{calculations.price}}</div>
      <div class="order__btn" @click="() => handleSub(true)">提交订单</div>
    </div>
    <div class="mask" v-show="showConfirm"  @click="() => handleSub(false)">
      <div class="mask__content" @click.stop>
        <h3 class="mask__content__title">确定要离开收银台吗</h3>
        <p class="mask__content__desc">请尽快完成支付，否则将被取消</p>
        <div class="mask__content__btns">
          <div
            class="mask__content__btn mask__content__btn--first"
            @click="() => handleOrder(true)"
          >取消订单</div>
          <div
            class="mask__content__btn mask__content__btn--last"
            @click="() => handleOrder(false)"
          >确认支付</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useCommonCartEffect } from '../../effects/cartEffects'
import { post } from '../../utils/request'

const useMakeOderEffect = (productList,shopName,shopId) => {
  const store = useStore()
  const router = useRouter()
  const handleOrder = async (isCaneled) => {
    const products = []
    for (let i in productList.value) {
      const product = productList.value[i]
      products.push({id: product._id, num: product.count})
    }
    try {
      const result = await post('/api/order', {
        addressId: 1,
        shopId,
        shopName: shopName.value,
        isCaneled: isCaneled,
        products
      })
      if (result?.errno === 0) {
        store.commit('clearCartData', shopId)
        router.push({ name: 'OrderList' })
      }
    } catch (e) {
      // showToast('请求失败')
    }
  }
  return { handleOrder }
}

export default {
  name: 'OrderConfirmation',
  setup() {
    const route = useRoute()
    const router = useRouter()
    const shopId = parseInt(route.params.id, 10)
    const { productList, shopName, calculations } = useCommonCartEffect(shopId)
    const { handleOrder } = useMakeOderEffect(productList,shopName,shopId)
    const showConfirm = ref(false)
    const handleBackClick = () => {
      router.back()
    }
    

    const handleSub = (states) => {
      showConfirm.value = states
    }
    return { productList,shopName, calculations, handleBackClick, handleOrder, showConfirm, handleSub}
  }
}
</script>

<style lang="scss" scoped>
@import '../../style/viriables.scss';
@import '../../style/mixins.scss';
.wrapper {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  background-color: #eee;
  overflow-y: scroll;
}
.top{
  position: relative;
  height: 1.96rem;
  background-size: 100% 1.59rem;
  background: linear-gradient(0deg, rgba(0, 145, 255, 0) 4%, #0091FF 50%);
  background-repeat: no-repeat;
  &__header {
    position: relative;
    padding-top: .26rem;
    line-height: .24rem;
    color: #fff;
    text-align: center;
    font-size: .16rem;
    &__back {
      position: absolute;
      left: .18rem;
      font-size: .22rem;
    }
  }
  &__receiver{
    position: absolute;
    left: .18rem;
    right: .18rem;
    bottom: 0;
    background: #fff;
    height: 1.11rem;
    border-radius: .04rem;
    &__title {
      padding: .16rem 0 .14rem .16rem;
      font-size:  .16rem;
      color: #333;
      line-height: .22rem;
    }
    &__address {
      padding: 0 .4rem 0 .16rem;
      font-size: .14rem;
      color: #333;
      line-height: .2rem;
    }
    &__info {
      padding: .06rem 0 0 .16rem;
      &__name{
        margin-right: .06rem;
        color: #666;
        line-height: .18rem;
        font-size: .12rem;
      }
    }
    &__icon {
      position: absolute;
      right: .16rem;
      top: .5rem;
      color: #666;
      font-size: .2rem;
      transform:  rotate(180deg);
    }
  }
}
.products {
  margin:  .16rem .18rem .55rem .18rem;
  background: #fff;
  &__title {
    padding: .16rem ;
    font-size: .16rem;
    color: #333;
  }
  &__list {
    overflow-y: hidden;
    position: absolute;
    margin:  0 .18rem;
    left: 0;
    right: 0;
    bottom: .6rem;
    top: 2.44rem;
    background: #fff;
  }
  &__item {
    position: relative;
    display: flex;
    padding: .16rem;
    &__img {
      width: .46rem;
      height: .46rem;
      margin-right: .16rem;
    }
    &__detail {
      flex: 1;
    }
    
    &__title {
      margin: 0;
      line-height: .2rem;
      font-size: .14rem;
      color: $content-fontcolor;
      @include ellipsis;
    }
    &__price {
      display: flex;
      margin: .06rem 0 0 0;
      line-height: .2rem;
      font-size: .14rem;
      color: $hightlight-fontColor;
    }
    &__totla {
      color: #000;
      text-align: right;
      flex: 1;
    }
    &__yen {
      font-size: .12rem;
    }
  }
}

.order {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  height: .49rem;
  line-height: .49rem;
  background: #fff;
  &__price {
    flex: 1;
    text-indent: .24rem;
    font-size: .14rem;
  }
  &__btn {
    width: .98rem;
    background: #4fb0f9;
    font-size: .14rem;
    color: #fff;
    text-align: center;
  }
}
.mask{
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  z-index: 1;
  background: rgba(0,0,0,0.50);
  &__content {
    width: 3rem;
    height: 1.56rem;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: #fff;
    border-radius: .04rem;
    text-align: center;
    &__title{
      font-size: .18rem;
      color: #333;
      line-height:  .26rem;
    }
    &__desc{
      font-size: .14rem;
      color: #666;
      margin:  .08rem 0 0 0;
    }
    &__btns{
      margin: .24rem .6rem 0 .6rem;
      display: flex;
      
    }
    &__btn{
      flex: 1;
      width: .8rem;
      height: .32rem;
      line-height: .32rem;
      border-radius: .16rem;
      &--first{
        margin-right: .12rem;
        border: 0.01rem solid #4fb0f9;
        color: #4fb0f9;
      }
      &--last{
        margin-left: .12rem;
        background: #4fb0f9;
        color: #fff;
      }
    }
  }
}
</style>