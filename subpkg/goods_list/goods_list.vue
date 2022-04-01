<template>
  <view>
    <view class="goods-list">
      <view v-for="(item,i) in goodsList" :key="i" @click="gotoDetail(item)">
        <my-goods :goods="item"></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        queryObj: {           // 请求参数对象
          query: '',            // 查询关键词
          cid: '',            // 商品分类Id
          pagenum: 1,           // 页码值
          pagesize: 10,            // 每页显示多少条数据
        },
        goodsList:[], // 商品列表的数据
        total:0,  // 总数量，用来实现分页
        isLoading:false,  // 是否正在请求数据
      }
    },
    onLoad(options){
      // 将页面参数转存到 this.queryObj 对象中
      this.queryObj.query=options.query||''
      this.queryObj.cid=options.cid||''
      this.getGoodsList()
    },
    methods: {
      // 获取商品列表数据的方法
      async getGoodsList(cb){
        this.isLoading=true // 打开节流阀
        const {data:res}=await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
        this.isLoading=false  // 关闭节流阀
        // 只要数据请求完毕，就立即按需调用 cb 回调函数
        cb&&cb()
        if (res.meta.status !== 200) return uni.$showMsg()
        // 为数据赋值
        this.goodsList = [...this.goodsList,...res.message.goods]
        this.total = res.message.total
      },
      gotoDetail(item){
        uni.navigateTo({
          url:'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })
      }
    },
    onReachBottom(){      // 触底事件
      // 判断是否还有下一页数据  当前的页码值 * 每页显示多少条数据 >= 总数条数
      if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！')
      // 判断是否正在请求其它数据，如果是，则不发起额外的请求
      if (this.isloading) return
      // 让页码值自增 +1
      this.queryObj.pagenum += 1
      // 重新获取列表数据
      this.getGoodsList()
    },
    onPullDownRefresh() {    // 下拉刷新的事件
      // 重置关键数据
      this.queryObj.pagenum=1 
      this.total=0
      this.isLoading=false
      this.goodsList=[]
      // 重新发起请求
      this.getGoodsList(()=>uni.stopPullDownRefresh())
    }
  }
</script>

<style>

</style>
