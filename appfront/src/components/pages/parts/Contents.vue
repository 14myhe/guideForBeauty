<template>
  <div class="contents" style="margin: 0; padding: 0">

    <!--搜索框-->
    <div class="search-bar">
      <div class="div-input">
        <el-input class="search-input" placeholder="请输入内容" v-model="input"></el-input>
        <el-button class="search-btn" slot="append" icon="el-icon-search" @click="search(order)"
                   v-loading.fullscreen.lock="fullscreenLoading"
        ></el-button>


      </div>
      <!--价格排序 下拉选择框-->
      <div>
        <el-button-group class="order-btn">
          <el-button plain @click="changeOrder('df')">默认排序</el-button>
          <el-button @click="changeOrder('pu')" icon="el-icon-arrow-up" plain>升价</el-button>
          <el-button @click="changeOrder('pd')" icon="el-icon-arrow-down" plain>降价</el-button>
        </el-button-group>
      </div>
    </div>


    <!--item 表-->
    <div class="hot-search">
      <h2>{{showWords}}</h2>
      <div class="goods-container">
        <div class="goods-item" v-for="(item, index) in searchGoods" :key="index">
          <a :href="item.address" :title="item.name" target="_blank">
            <img :src="item.img1_address" alt="item.name">
          </a>

          <div class="info">
            <a :href="item.address" :title="item.name" target="_blank"><p class="title">{{item.name}}</p></a>
            <p class="description">{{item.description}}</p>
            <p class="platform">平台： <span>{{item.platform}}</span></p>
            <p class="store">商店： <span>{{item.store}}</span></p>
            <p class="comment-no">评论数： <span>{{item.comment_count}}</span></p>
            <p class="comment-rate">好评率： <span>{{item.good_comment_percentage}}</span></p>
          </div>


          <div class="price-div">
            <div class="price-p">￥<span class="price">{{item.price}}</span></div>
            <div class="btn-div">
              <el-button @click="getCutPrice(item, username)" class="price-btn" type="text">降价通知</el-button>
              <!--<el-button @click="dialogVisible=true" class="price-btn" type="text">降价通知</el-button>-->
              <el-button @click="getSimilar(item)" class="similar-btn" type="text">找相似物</el-button>
            </div>
          </div>
          <!--<div class="btn-div">-->
          <!--</div>-->
        </div>
      </div>
    </div>


    <!--分页器-->
    <div style="margin: 50px 0 20px 0; padding: 0;">
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :page-size = "20"
        :total="pageCount"
        :current-page.sync="currentPage">
      </el-pagination>
    </div>


    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose">
      <p>尊敬的用户<span style="color: #FF8080">{{this.username}}</span>,您是否确认关注商品：</p>
      <p style="color: #409EFF">{{this.pname}}</p>
      <p>的降价变动?</p>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>


  </div>
</template>

<script>
  import ElButton from '../../../../node_modules/element-ui/packages/button/src/button'
  export default {
    components: {ElButton},

    data () {
      return {
        username: ' 13411984676',  // 从cookie中拿到的username，假数据
        currentPage: 1,   // 当前页
        dialogVisible: false,
        fullscreenLoading: false,
        order: 'df',
        pname: '',
        res: {},
        input: '',
        showWords: '',
        pageCount: 0,
        searchGoods: [],
      }
    },
    methods: {
      searchWithPage(pageNo, order){
        if (this.input === '') {
          this.$message.error('请输入搜索信息！')
        } else {
          this.fullscreenLoading = true;
          let kw = this.input.replace(' ', '%20');
          this.$http.get('http://127.0.0.1:8000/beauty/productsList/getProductsPage?wd='+kw+'&PageNo='+pageNo+'&order='+order)
            .then((response) => {
              this.res = response.data
              if (this.res.error_code === 0) {
                // 成功
                let ress = this.res['data']['item_list']
                // emmmmmm， 将小图换成大图
                for (let i in ress) {
                  ress[i]['img1_address'] = ress[i]['img1_address'].toString().replace('360buyimg.com/n5', '360buyimg.com/n7')
                }

                this.searchGoods = this.res['data']['item_list']
                // page数
                this.pageCount = this.res['page_count'] * 20

              } else {  // 失败
                this.$message.error('没有查找到对应的商品，请重试！')
                console.log(this.res['msg']);
              }
              this.fullscreenLoading = false;
            });

        }
      },
      search(order) {
        this.currentPage = 1;
        this.order = order;
        this
          .handleCurrentChange(1);
      },
      changeOrder(order) {
        this.order = order;
        this.search(order);
      },
      getCutPrice(item, username) {   // 增-降价通知
        this.dialogVisible = true;
        this.pname = item.name
        console.log(item);
        console.log(username);
      },
      getSimilar(item) {   // 相似商品
        this.$router.push({name: 'similar', params: {item: item}})
      },
      handleClose(done) {
        this.$message.success('成功！');
        this.dialogVisible = false;
      },
      handleCurrentChange(currentPage) {
        this.searchWithPage(currentPage, this.order);
      }
    }

  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .contents {
    .search-bar {
      display: -webkit-flex; /* Safari  chrome */
      display: flex;
      justify-content: center;
      .div-input {
        margin: 15px 0 15px 0;
        width: 400px;
        display: -webkit-flex; /* Safari  chrome */
        display: flex;
        flex-direction: row;
        .search-input {
        }
        .search-btn {
        }
      }
      .order-btn {
        margin: 15px 0 15px 40px;
      }
    }

    /*.order-btn{*/
    /*width: 70%;*/
    /*margin: 10px auto 0;*/
    /*}*/
    .hot-search {
      width: 1226px;
      margin: 26px auto 0;
      h2 {
        font-size: 22px;
        font-weight: 200;
        line-height: 58px;
        color: #333;
        text-align: left;
      }
      .goods-container {
        width: 1226px;
        display: -webkit-flex; /* Safari  chrome */
        display: flex;
        flex-direction: row;
        justify-content: center;
        flex-wrap: wrap; // 自动换行
        .goods-item {
          width: 80%;
          height: 180px;
          background: #fcfcfc;
          display: -webkit-flex; /* Safari  chrome */
          display: flex;
          flex-direction: row;
          justify-content: left;
          align-items: center;
          text-align: center;
          transition: all .2s linear;
          margin: 8px 5px 12px 5px;

          &:hover {
            box-shadow: 0 20px 35px rgba(0, 0, 0, 0.1);
            border-top: 3px solid #ff8080; //#e53935
            border-bottom: 3px solid #ff8080; //#e53935
            margin-top: -1px;
          }

          a {
            display: flex;
            flex-direction: row;
            justify-content: space-between;
            font-size: 12px;
            font-weight: 400;
            text-align: left;
            margin-left: 40px;
            img {
              width: 160px;
              height: 160px;
            }
          }

          .info {
            display: -webkit-flex; /* Safari  chrome */
            display: flex;
            flex-direction: column;
            justify-content: left;
            text-align: left;
            margin: 0 0 0 40px;
            width: 70%;

            a {
              margin: 0;
              .title {
                width: 100%;
                font-size: 14px;
                color: #096296;
                &:hover {
                  text-decoration: underline;
                }
              }
            }

            .description {
              color: #b0b0b0;
              font-size: 10px;
              margin: 10px 0 5px 0;
            }

            .platform {
              margin: 7px 0 0 0;
              span {
                color: #409EFF;
                font-size: 12px;
              }
            }
            .store {
              margin: 5px 0 0 0;
              span {
                color: #bbbbbb;
                font-size: 12px;
              }
            }

            .comment-no {
              margin: 7px 0 0 0;
              span {
                color: #ff8080;
                font-size: 12px;
              }
            }

            .comment-rate {
              margin: 7px 0 0 0;
              span {
                color: #ff8080;
                font-size: 12px;
              }
            }
          }

          .price-div {
            display: -webkit-flex; /* Safari  chrome */
            display: flex;
            flex-direction: row;
            flex-wrap: nowrap;
            margin: 0;
            padding: 0;
            text-align: right;
            .price-p {
              width: 150px;
              .price {
                color: #ff8080;
                font-size: 24px;
              }
            }

            .btn-div {
              margin-left: 40px;
              margin-right: 40px;
              display: -webkit-flex; /* Safari  chrome */
              display: flex;
              flex-direction: column;
              font-size: 12px;
              .price-btn {
                margin: 0;
                padding: 0;
                &:hover {
                  text-decoration: underline;
                }
              }
              .similar-btn {
                color: #ff8888;
                margin: 10px 0 0 0;
                padding: 0;
                &:hover {
                  text-decoration: underline;
                }
              }
            }

          }

        }
      }
    }
  }

</style>
