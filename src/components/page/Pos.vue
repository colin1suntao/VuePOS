<template>
  <div class="pos">
   	<el-row>
    	<el-col :span="7" class="pos-order" id="order-list">
				<el-tabs>
					<el-tab-pane label="点餐">
						<el-table :data="tableData" border style="width: 100%">
							<el-table-column prop="goodsName" label="商品名称" width="120"></el-table-column>
							<el-table-column prop="count" label="数量" width="70"></el-table-column>
							<el-table-column prop="price" label="价格" width="100"></el-table-column>
							<el-table-column label="操作" fixed="right" width="100">
								  <template scope="scope">
									  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
									  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
								  </template>
							</el-table-column>
						</el-table>
						<div class="total-div">
							<span>数量:<strong>{{totalCount}}</strong></span>
							<span>总计:<strong>{{totalMoney}}</strong></span>
						</div>
						<div class="div-btn">
							<el-button type="warning">挂单</el-button>
							<el-button type="danger" @click="delAllGoods()">删除</el-button>
							<el-button type="success" @click="checkout()">结账</el-button>
						</div>
					</el-tab-pane>
					<el-tab-pane label="挂单">挂单</el-tab-pane>
					<el-tab-pane label="外卖">外卖</el-tab-pane>
				</el-tabs>
     	</el-col>
			<el-col :span="17">
				<div class="common-goods">
					<div class="title">常用商品</div>
					<div class="goods-list">
						<ul>
							<li v-for="goods in commonGoods" @click="addOrderList(goods)">
								<span class="goods-name">{{goods.goodsName}}</span>
								<span class="goods-price">￥{{goods.price}}元</span>
							</li>
						</ul>
					</div>
				</div>
				<div class="goods-type">
					<el-tabs>
						<el-tab-pane label="汉堡">
							<ul class='cookList'>
									<li v-for="goods in type0Goods" @click="addOrderList(goods)">
											<span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
											<span class="foodName">{{goods.goodsName}}</span>
											<span class="foodPrice">￥{{goods.price}}元</span>
									</li>
							</ul>
						</el-tab-pane>
						<el-tab-pane label="小食">
							<ul class='cookList'>
									<li v-for="goods in type1Goods" @click="addOrderList(goods)">
											<span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
											<span class="foodName">{{goods.goodsName}}</span>
											<span class="foodPrice">￥{{goods.price}}元</span>
									</li>
							</ul>
						</el-tab-pane>
						<el-tab-pane label="饮料">
							<ul class='cookList'>
									<li v-for="goods in type2Goods" @click="addOrderList(goods)">
											<span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
											<span class="foodName">{{goods.goodsName}}</span>
											<span class="foodPrice">￥{{goods.price}}元</span>
									</li>
							</ul>
						</el-tab-pane>
						<el-tab-pane label="套餐">
							<ul class='cookList'>
									<li v-for="goods in type3Goods" @click="addOrderList(goods)">
											<span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
											<span class="foodName">{{goods.goodsName}}</span>
											<span class="foodPrice">￥{{goods.price}}元</span>
									</li>
							</ul>
						</el-tab-pane>
						
					</el-tabs>
				</div>
			</el-col>
   </el-row>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'pos',
	data() {
		return {
			tableData: [],
 			commonGoods: [],
			type0Goods: [],
			type1Goods: [],
			type2Goods: [],
			type3Goods: [],
      totalCount: 0,
      totalMoney: 0
		}
	},
	created:function() {
		//拉取常用商品列表
		axios.get('http://jspang.com/DemoApi/oftenGoods.php')
		.then(responese => {
			console.log(responese);
			this.commonGoods = responese.data;
		})
		.catch(error => {
			console.log(error);
			alert('网络错误，不能访问');
		}),
		//拉取分类商品列表
		axios.get('http://jspang.com/DemoApi/typeGoods.php')
		.then(responese => {
			console.log(responese);
			this.type0Goods = responese.data[0];
			this.type1Goods = responese.data[1];
			this.type2Goods = responese.data[2];
			this.type3Goods = responese.data[3];
		})
		.catch(error => {
			console.log(error);
			alert('网络错误，不能访问');
		})

	},
	mounted:function() {
		var orderHeight =document.body.clientHeight;
		document.getElementById('order-list').style.height = orderHeight + 'px';
	},
	 methods: {
        //添加订单列表的方法
        addOrderList(goods) {
            //console.log(goods);
            this.totalCount = 0; //汇总数量清0
            this.totalMoney = 0;
            let isHave = false;
            //判断是否这个商品已经存在于订单列表
            for (let i = 0; i < this.tableData.length; i++) {
                console.log(this.tableData[i].goodsId);
                if (this.tableData[i].goodsId == goods.goodsId) {
                    isHave = true; //存在
                }
            }
            //根据isHave的值判断订单列表中是否已经有此商品
            if (isHave) {
                //存在就进行数量添加
                let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
                arr[0].count++;
                //console.log(arr);
            } else {
                //不存在就推入数组
                let newGoods = { goodsId: goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1 };
                this.tableData.push(newGoods);
            }
            this.getAllMoney();
        },
        //删除单个商品
        delSingleGoods(goods) {
            console.log(goods);
            this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
            this.getAllMoney();
        },
        //删除所有商品
        delAllGoods() {
            this.tableData = [];
            this.totalCount = 0;
            this.totalMoney = 0;
        },
        //汇总数量和金额
        getAllMoney() {
            this.totalCount = 0;
            this.totalMoney = 0;
            if (this.tableData) {
                this.tableData.forEach((element) => {
                    this.totalCount += element.count;
                    this.totalMoney = this.totalMoney + (element.price * element.count);
                });
            }
        },
        //结账方法模拟
        checkout() {
            if (this.totalCount!=0) {
                this.tableData = [];
                this.totalCount = 0;
                this.totalMoney = 0;
                this.$message({
                    message: '结账成功，感谢你又为店里出了一份力!',
                    type: 'success'
                });
            }else{
                this.$message.error('不能空结。老板了解你急切的心情！');
            }
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul li {
	list-style: none;
}

.pos-order {
	background-color: #F9FAFC;
	border-right: 1px solid #C0CCDA;
	height: 100%;
}

.div-btn {
	margin-top: 10px;
}

.title {
	height: 20px;
	line-height: 20px;
	border-bottom: 1px solid #D3dce6;
	background-color: #F9FAFC;
	padding: 10px;
	text-align: left;
}

.goods-list ul li {
	float: left;
	border: 1px solid #E5E9F2;
	padding: 10px;
	margin: 10px;
	background-color: #ffffff;
  cursor: pointer;
}

.goods-price {
	color: #58b7ff;
}

.goods-type {
	clear: both;
}

.cookList li {
       list-style: none;
       width:23%;
       border:1px solid #E5E9F2;
       height: auot;
       overflow: hidden;
       background-color:#fff;
       padding: 2px;
       float:left;
       margin: 2px;
			 cursor: pointer;
 
}
.cookList li span {
        display: block;
        float:left;
}

.foodImg {
       width: 40%;
}

.foodName {
       font-size: 18px;
       padding-left: 10px;
       color:brown;
}

.foodPrice {
       font-size: 16px;
       padding-left: 10px;
       padding-top:10px;
}

.total-div {
	height: auto;
	overflow: hidden;
	text-align: center;
	font-size: 16px;
	background-color: #ffffff;
	border-bottom: 1px solid #E5E9F2;
}

.total-div span {
	padding: 10px;
}

</style>
