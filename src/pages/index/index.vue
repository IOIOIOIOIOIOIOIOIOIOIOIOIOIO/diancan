<template>
  <div class="index">
    <div class="index-search">
      <div class="search-wrap" @click="toShowSearch">
        <!-- <icon class="icon" type="search" size="14" /> -->
        <!-- <input type="text" placeholder="今天想吃什么" @focus.prevent="toShowSearch" /> -->
        <!-- <div class="input-text"></div> -->
        <icon class="icon" type="search" size="14" />今天想吃什么
      </div>
    </div>
    <div class="index-container">
      <scroll-view
        scroll-y
        :style="{height:wrapperHeight+'px'}"
        class="index-left page-loadmore-wrapper"
      >
        <ul class="left-list" v-if="productList">
          <li
            @click="chooseList(index)"
            :class="['left-item',{'ischoose':index==selectIndex}]"
            v-for="(item,index) in productList"
            :key="item.typeId"
          >{{item.typeName}}</li>
        </ul>
      </scroll-view>
      <scroll-view
        scroll-y
        :style="{height:wrapperHeight+'px'}"
        class="index-right page-loadmore-wrapper"
      >
        <ul class="right-list" v-if="productItem.length>0">
          <li class="right-item" v-for="item in productItem" :key="item.productId">
            <img :src="item.productPic" alt class="item-img" />
            <div class="item-right">
              <p class="item-right-top">{{item.productName}}</p>
              <p class="item-right-center">好评推荐</p>
              <p class="item-right-bottom">￥{{item.price}}/份</p>
              <div class="stepper">
                <stepper :product="item" :key="item.productId" ref="stepper" v-if="refresh" @changeNum="changeNum"></stepper>
              </div>
            </div>
          </li>

          
        </ul>
      </scroll-view>
    </div>
    <div class="search-page" v-if="showSearch">
      <div class="search-page-top">
        <div class="search-page-top-wrap">
          <icon class="icon" type="search" size="14" />
          <input type="text" placeholder="今天想吃什么" :focus="isFocus" v-model="searchValue" />
        </div>
        <span v-if="searchValue" @click="goSearch">搜索</span>
        <span v-else @click="showSearch=false">取消</span>
      </div>
      <div class="search-list" v-if="searchList.length>0">
        <div class="search-item" v-for="item in searchList" :key="item.productId">
          <div class="item-img">
            <img :src="item.productPic" alt />
          </div>
          <p class="item-name">{{item.productName}}</p>
          <p class="item-price">￥{{item.price}}/份</p>
          <div class="stepper">
            <stepper :product="item" :key="item.productId" v-if="refresh1" @changeNum="changeNum"></stepper>
          </div>
        </div>
      </div>
    </div>

    <div class="all-count" v-if="allCount>0">￥{{allCount}}</div>
  </div>
</template>

<script>
import stepper from "@/components/stepper";
import { getProductData, getSearch } from "@/utils/request";
export default {
  data() {
    return {
      wrapperHeight: 0,
      count: 0,
      showSearch: false,
      isFocus: true,
      searchValue: "",
      productList: [],
      selectIndex: 0,
      searchList: [],
      productItem: [],
      refresh: false,
      refresh1: false,
      allCount:0
    };
  },

  components: { stepper },
  methods: {
    changeNum(){
      this.getAllCount()
    },
    // 获取总价
    getAllCount(){
      let cost=0
      this.productList.forEach(e=>{
        if(e.productList){
          e.productList.forEach(even=>{
            even.count?cost+=parseFloat(even.price)*parseFloat(even.count):cost+=0
          })
        }
      })
      this.allCount=cost
    },
    // 搜索菜品
    goSearch() {
      getSearch({
        shopId: this.$store.state.shopId,
        productName: this.searchValue
      }).then(res => {
        this.searchList = res.list;
        this.getProductCount(this.searchList);
        // 强制组件刷新
        this.refresh1 = false;
        this.$nextTick(() => {
          this.refresh1 = true;
        });
      });
    },
    // 获取商品数
    getProductCount(data) {
      if (data instanceof Array) {
        data.forEach(e => {
          if (e.productId) {
            if (wx.getStorageSync("" + e.productId)) {
              let product = wx.getStorageSync("" + e.productId);
              e.count = product.count;
            } else {
              e.count = 0;
            }
          }
        });
        return data;
      }
    },
    goRefresh() {
      // 强制组件刷新
      this.refresh = false;
      this.$nextTick(() => {
        this.refresh = true;
      });
    },
    chooseList(index) {
      this.selectIndex = index;
      let productItem = this.productList[index].productList;
      this.productItem = this.getProductCount(productItem);
      this.goRefresh();
    },
    toShowSearch() {
      this.showSearch = true;
      this.searchValue=''
      this.searchList=[]
    },
    // 获取商品列表数据
    getData(id) {
      getProductData({ shopId: id }).then(res => {
        this.productList = res.productList;
        this.productList.forEach(e => {
          if (e.productList) {
            this.getProductCount(e.productList);
          }
        });
        // +++
        this.getAllCount()
        // +++
        this.chooseList(0);
      });
    }
  },
  mounted() {
    let shopId = this.$store.state.shopId;
    let self = this;
    // +++
    // self.getData(1);
    // +++

    if (shopId) {
      // self.getData(1);
      self.getData(shopId);
    }
    const query = wx.createSelectorQuery();
    query
      .selectViewport()
      .fields({
        size: true
      })
      .select(".page-loadmore-wrapper")
      .boundingClientRect()
      .exec(function(res) {
        const DW = res[0].height;
        self.wrapperHeight = DW - res[1].top - 20;
      });
  },
  onShow() {
    this.showSearch = false;
    this.productList.forEach(e => {
      if (e.productList) {
        this.getProductCount(e.productList);
      }
    });
    this.goRefresh();
    this.getAllCount()
  }
};
</script>

<style lang="less" scoped>
.search-page {
  padding-top: 104rpx;
  background-color: #f1f1f1;
  z-index: 999;
  position: fixed;
  left: 0;
  top: 0;
  bottom: 0;
  right: 0;
  overflow: auto;
  .search-page-top {
    background-color: #fff;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    height: 104rpx;
    padding: 24rpx 28rpx 24rpx 20rpx;
    box-sizing: border-box;
    z-index: 100;
    .search-page-top-wrap {
      display: inline-block;
      width: 545rpx;
      background-color: rgba(237, 237, 237);
      border-radius: 30rpx;
      padding-left: 80rpx;
      font-size: 28rpx;
      position: relative;
      .icon {
        position: absolute;
        left: 40rpx;
        top: 10rpx;
      }
      input {
        width: 545rpx;
      }
    }
    span {
      display: inline-block;
      font-size: 28rpx;
      color: #000;
      // margin-top: 14rpx;
      line-height: 50rpx;
      vertical-align: top;
      margin-left: 20rpx;
    }
  }
  .search-list {
    overflow: auto;
    padding: 12rpx 22rpx;
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
    .search-item {
      position: relative;
      .stepper {
        position: absolute;
        right: 20rpx;
        bottom: 14rpx;
      }
      border-radius: 10rpx;
      // display: inline-block;
      background-color: #fff;
      flex: 0 0 322rpx;
      width: 322rpx;
      padding: 10rpx 12rpx 21rpx;
      margin-bottom: 12rpx;
      // margin: 0 8rpx 12rpx 12rpx;
      font-size: 26rpx;
      .item-img {
        width: 322rpx;
        height: 250rpx;
        border-radius: 10rpx;
        overflow: hidden;
        img {
          width: 100%;
          height: 100%;
        }
      }
      .item-name {
        margin: 8rpx 0 48rpx;
      }
      .item-price {
        color: rgb(255, 72, 0);
      }
    }
  }
}
.index-search {
  padding: 24rpx 34rpx;
  .search-wrap {
    position: relative;
    background-color: rgba(237, 237, 237);
    border-radius: 30rpx;
    padding-left: 38%;
    font-size: 28rpx;
    height: 50rpx;
    line-height: 50rpx;
    color:rgba(128, 128, 128, 0.5);
  }
  .icon {
    position: absolute;
    left: 35%;
    margin-left: -35rpx;
    top: 10rpx;
  }
}
.index {
  position: relative;
  .index-container {
    display: flex;
  }
  .index-left {
    font-size: 30rpx;
    color: rgb(136, 136, 136);
    // line-height: 108rpx;
    flex: 0 0 160rpx;
    width: 160rpx;
    .left-list {
      padding: 0 20rpx;
      .left-item {
        text-align: justify;
        line-height: 108rpx;
        height: 108rpx;
        overflow: hidden;
      }
      .left-item::after {
        line-height: 0;
        width: 100%;
        display: inline-block;
        content: "";
      }
    }
  }
  .index-right {
    flex: 1;
    .right-list {
      .right-item {
        border-top: 1rpx solid rgba(128, 128, 128, 0.5);
        display: flex;
        justify-content: flex-start;
        align-items: center;
        height: 184rpx;
        overflow: hidden;
        .item-img {
          flex: 0 0 184rpx;
          width: 184rpx;
          height: 184rpx;
        }
        .item-right {
          flex: 1;
          position: relative;
          box-sizing: border-box;
          font-size: 24rpx;
          padding: 25rpx;
          overflow: hidden;
          .item-right-top {
            color: rgb(0, 0, 0);
            font-size: 26rpx;
          }
          .item-right-center {
            margin: 10rpx 0 47rpx;
            color: rgb(65, 65, 65);
          }
          .item-right-bottom {
            color: rgb(255, 72, 0);
          }
          .stepper {
            position: absolute;
            right: 64rpx;
            bottom: 30rpx;
          }
          // .right-count1,
          // .right-count2 {
          //   position: absolute;
          //   right: 64rpx;
          //   bottom: 30rpx;
          //   background-color: rgb(243, 245, 249);
          // }
          // .right-count1 {
          //   width: 132rpx;
          //   height: 30rpx;
          //   line-height: 30rpx;
          //   border-radius: 30rpx;
          //   display: flex;
          //   justify-content: space-around;
          // }
          // .right-count2 {
          //   border-radius: 50%;
          //   width: 30rpx;
          //   height: 30rpx;
          //   line-height: 30rpx;
          //   text-align: center;
          // }
        }
      }
      .right-item:last-child {
        border-bottom: 1rpx solid rgba(128, 128, 128, 0.5);
      }
    }
  }
}
.ischoose {
  color: #000;
}
.all-count {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 40rpx;
  background-color: rgba(236, 233, 33, 0.521);
  color:rgb(255, 72, 0);
  text-align: center;
  font-size: 30rpx;
}
input{
  height: 50rpx;
  line-height: 50rpx;
}
.input-text{
  height: 50rpx;
  line-height: 50rpx;
}
</style>

