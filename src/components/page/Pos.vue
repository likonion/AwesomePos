<template>
  <div class="pos">
    <div class="order-list">
      <el-tabs>
        <el-tab-pane label="点餐">
          <el-table :data="tableData" stripe border show-summary style="width: 100%" height="100%"
                    :summary-method="total">

            <el-table-column prop="goodsName" label="商品"></el-table-column>
            <el-table-column sortable prop="count" label="量"></el-table-column>
            <el-table-column sortable prop="price" label="单价"></el-table-column>
            <el-table-column label="操作" width="100" fixed="right">
              <template scope="scope">
                <el-button type="danger" size="mini" icon="minus" @click="removeChat(scope.row)"></el-button>
                <el-button type="primary" size="mini" icon="plus" @click="addChat(scope.row)"></el-button>

              </template>
            </el-table-column>
          </el-table>


          <div class="order-btn">
            <el-button type="warning" @click="setOrder">挂单</el-button>
            <el-button type="danger" @click="clearChat">清空</el-button>
            <el-button type="success" @click="pay">结账</el-button>
          </div>
        </el-tab-pane>
        <el-tab-pane label="挂单">
          挂单
        </el-tab-pane>
        <el-tab-pane label="外卖">
          外卖
        </el-tab-pane>
      </el-tabs>
    </div>
    <div class="food-list">
      <div class="often-goods clearfix">
        <div class="title">常用商品</div>
        <div class="often-goods-list">
          <ul>
            <li v-for="goods in oftenGoods" v-if="oftenGoods[0]!==undefined" @click="addChat(goods)">
              <span>{{goods.goodsName}}</span>
              <span class="o-price">¥{{goods.price}}元</span>
            </li>
          </ul>
        </div>
      </div>
      <div class="goods-type">

        <el-tabs>
          <el-tab-pane label="主食">
            <ul class='cookList'>
              <li v-for="goods in foods" @click="addChat(goods)">
                <span class="foodImg"><img :src="'static/images/foods/'+goods.goodsImg+'.jpg'" width="180" height="170"></span>
                <span class="foodName">{{goods.goodsName}}</span>
                <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="小食">
            <ul class='cookList'>
              <li v-for="goods in snacks" @click="addChat(goods)">
                <span class="foodImg"><img :src="'static/images/snacks/'+goods.goodsImg+'.jpg'" width="180"
                                           height="170"></span>
                <span class="foodName">{{goods.goodsName}}</span>
                <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="饮料">
            <ul class='cookList'>
              <li v-for="goods in drinks" @click="addChat(goods)">
                <span class="foodImg"><img :src="'static/images/drinks/'+goods.goodsImg+'.jpg'" width="180"
                                           height="170"></span>
                <span class="foodName">{{goods.goodsName}}</span>
                <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="套餐">
            <ul class='cookList'>
              <li v-for="goods in combo" @click="addChat(goods)">
                <span class="foodImg"><img :src="'static/images/combo/'+goods.goodsImg+'.jpg'" width="180" height="170"></span>
                <span class="foodName">{{goods.goodsName}}</span>
                <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>

        </el-tabs>
      </div>
    </div>
    <el-dialog title="结账" :visible.sync="dialogTableVisible">
      <el-steps :space="200" :active="active">
        <el-step title="步骤 1" description="这是一段很长很长很长的描述性文字"></el-step>
        <el-step title="步骤 2" description="这是一段很长很长很长的描述性文字"></el-step>
        <el-step title="步骤 3" description="这是一段很长很长很长的描述性文字"></el-step>
      </el-steps>
      <el-button style="margin-top: 12px;" @click="next">下一步</el-button>
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  import data from '../../mock/mock'
  import Mock from 'mockjs'
  export default {
    name: 'Pos',
    data() {
      return {
        tableData: [],
        foods: [],
        snacks: [],
        combo: [],
        drinks: [],
        totalMoney: 0,
        dialogTableVisible: false,
        active: 0
      }
    },
    computed: {
      oftenGoods() {
        return [this.foods[1], this.snacks[3], this.combo[2], this.drinks[8], this.foods[4], this.snacks[6], this.combo[8], this.drinks[3]]
      }
    },
    methods: {
      // table中增加购买数量
      addChat(goods) {
        let isHave = false
        this.tableData.forEach(data => {
          if (data.goodsId === goods.goodsId) {
            data.count++
            isHave = true
          }
        })
        if (!isHave) {
          let good = {goodsName: goods.goodsName, goodsId: goods.goodsId, count: 1, price: goods.price}
          this.tableData.push(good)
        }
      },
      // table中减少购物数量
      removeChat(goods) {
        this.tableData.forEach((data, index) => {
          if (data.goodsId === goods.goodsId) {
            if (data.count === 1) {
              this.tableData.splice(index, 1)
            } else {
              data.count--
            }
          }
        })

      },
      // 使用summary-method自定义合计逻辑
      total(param) {
        const {columns, data} = param;
        let totalMoney = 0;
        let totalCount = 0;
        data.forEach(data => {
          totalMoney += data.price * data.count
          totalCount += data.count
        })
        this.totalMoney = totalMoney
        let sums = ['总价', totalCount, totalMoney]
        return sums;

      },
      clearChat() {
        if (this.tableData.length === 0) {
          this.$notify.error({
            title: '错误',
            message: '您的订单为空，请添加订单！'
          });
        } else {
          this.$confirm('确定要清空购物车吗？', {
            confirmButtonText: '确定',
            cancelButtonText: '点错了',
            type: 'warning'
          }).then(() => {
            this.tableData = [];
            this.$message({
              type: 'success',
              message: '清空购物车成功!'
            });
          }).catch(() => {

          });
        }
      },
      // 挂单
      setOrder() {

        if (this.tableData.length === 0) {
          this.$notify.error({
            title: '错误',
            message: '您的订单为空，请添加订单！'
          });
        } else {
          this.tableData = [];
          this.$notify({
            title: '成功',
            message: '挂单成功',
            type: 'success'
          });
        }

      },
      // 结账
      pay() {
        if (this.totalMoney !== 0) {
          this.dialogTableVisible = true;
        } else {
          this.$message.error('您还没有点餐哦，请添加您想吃的！');
        }

      },
      // Steps 步骤条
      next() {
        if (this.active++ > 2) {
          this.active = 0;
          this.dialogTableVisible = false;
          this.tableData = [];
        }
      }
    },
    created() {
      // 使用axios获取json数据
      axios.get('static/data.json').then(resolve => {
        this.foods = resolve.data.foods
        this.snacks = resolve.data.snacks
        this.combo = resolve.data.combo
        this.drinks = resolve.data.drinks
      }).catch((err) => {
        console.log(err)
      })
      // mockjs拦截data.json请求，返回mock数据
      Mock.mock('static/data.json', data)
    }
  }
</script>
<style lang="scss">
  .pos {
    width: 95%;
    height: 100%;
    display: flex;
    align-items: flex-start;
    flex-flow: row nowrap;
    justify-content: center;
    flex: 1 1 auto;
    .order-list {
      flex: 0;
      height: 100%;
      background-color: #f9fafc;
      border-right: 1px solid #eee;
      .el-tabs {
        width: 350px;
        height: 100%;
        display: flex;
        flex-flow: column nowrap;
        .el-tabs__header {
          margin: 0;
        }
        .el-tabs__content {
          height: 100%;
          flex: 1 1 auto;
          .el-tab-pane {
            .el-table {
              position: absolute;
              left: 0;
              top: 0;
              bottom: 44px;
              overflow: hidden;
            }
            .order-btn {
              position: absolute;
              bottom: 0;
              height: 44px;
              padding: 5px;
              width: 100%;
            }

          }
        }
      }

    }
    .food-list {
      display: flex;
      height: 100%;
      flex-flow: column nowrap;
      flex: 1 1 auto;
      .often-goods {
        flex: 0 0 auto;
        .title {
          border-bottom: 1px solid #d3dce6;
          background-color: #f9fafc;
          padding: 10px;
          text-align: left;
        }
        .often-goods-list {
          li {
            float: left;
            border: 1px solid #E5E9F2;
            padding: 10px;
            margin: 5px;
            background-color: #fff;
            cursor: pointer;
            &:hover {
              color: white;
              background-color: #58B7FF;
              .o-price {
                color: white;
              }
            }
            .o-price {
              color: #58B7FF;
              font-weight: 700;
            }
          }

        }
      }
      .goods-type {
        height: 100%;
        flex: 1 1 auto;
        overflow: hidden;
        .el-tabs {
          height: 100%;
          .el-tabs__content {
            height: 100%;
            overflow: auto;
            .cookList {
              padding-left: 10px;
              padding-bottom: 60px;
              overflow: auto;
              text-align: left;
              li {
                list-style: none;
                width: 186px;
                height: 226px;
                border: 1px solid #E5E9F2;
                overflow: hidden;
                background-color: #fafafa;
                padding: 2px;
                margin: 2px;
                display: inline-block;
                position: relative;
                cursor: pointer;
                span {

                  display: inline-block;
                }
                .foodImg {
                }
                .foodName {
                  font-size: 12px;
                  padding-left: 10px;
                  color: brown;

                }
                .foodPrice {
                  display: block;
                  padding-left: 6px;
                  font-size: 16px;
                  font-weight: bold;
                  color: red;

                }
              }
            }
          }
          .el-tabs__header {

          }
        }

      }
    }
  }

</style>
