<template>
  <div class="stepper">
    <div class="right-count1" v-if="count>0">
      <span class="count-reduce" @click="reduceProduct">-</span>
      <!-- <span class="count-number" v-if="count!=product.count">{{product.count}}</span> -->
      <span class="count-number">{{count}}</span>
      <span class="count-add" @click="addProduct">+</span>
    </div>
    <div class="right-count2" v-else>
      <span class="count-none" @click="addProduct">+</span>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    };
  },
  props: {
    product: {
      type: Object,
      default: {
        count: 0
      }
    },
    index: {
      type: [Number, String],
      default: 0
    }
  },
  components: {},
  computed: {},
  methods: {
    reduceProduct() {
      this.product.count--;
      let obj = wx.getStorageSync("" + this.product.productId);
      if (this.product.count > 0 && obj) {
        obj.count = this.product.count;
        wx.setStorageSync("" + this.product.productId, obj);
      } else {
        wx.removeStorageSync("" + this.product.productId);
      }
      this.changeCount();
    },
    addProduct() {
      this.product.count++;
      let obj = wx.getStorageSync("" + this.product.productId);
      if (obj) {
        obj.count = this.product.count;
        wx.setStorageSync("" + this.product.productId, obj);
      } else {
        wx.setStorageSync("" + this.product.productId, this.product);
      }
      this.changeCount();
    },
    changeCount() {
      this.count = this.product.count;
      this.$emit('changeNum')
      // this.$emit('changeCount',this.product.count,this.index)
    }
  },
  mounted() {
    this.count = this.product.count;
  },
  created() {}
};
</script>
<style lang="less" scoped>
.stepper {
  display: inline-block;
}
.right-count1,
.right-count2 {
  background-color: rgb(243, 245, 249);
  // padding: 10rpx;
  font-size: 30rpx;
  text-align: center;
}
.right-count1 {
  width: 132rpx;
  height: 50rpx;
  line-height: 50rpx;
  border-radius: 30rpx;
  display: flex;
  justify-content: space-around;
}
.right-count2 {
  border-radius: 50%;
  width: 50rpx;
  height: 50rpx;
  line-height: 50rpx;
  text-align: center;
}
.count-reduce,.count-add,.count-none{

// line-height: 30rpx;
display: inline-block;
height: 50rpx;
width: 50rpx;
}
</style>
