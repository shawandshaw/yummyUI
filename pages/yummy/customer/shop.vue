<template>
    <v-app id="home">
        <Header color="primary">
            <v-badge color="red" overlap>
                <span slot="badge">{{cartItems.length}}</span>
                <v-btn icon @click="cartDialog=true">
                    <v-icon>shopping_cart</v-icon>
                </v-btn>
            </v-badge>
        </Header>

        <v-content>
            <v-img src="/img/dishes.jpg" style="filter: blur(5px)" height="150"></v-img>
            <v-layout justify-center>
                <v-container style="position:absolute;top:0;">
                    <v-layout class="lightbox white--text" align-center justify-space-between>
                        <div>
                              <v-avatar size="100">
                            <img :src="'/img/'+shop.id+'.jpg'">
                        </v-avatar>
                        <span class="title">{{shop.name}} <v-icon color="white">place</v-icon> {{shop.address.name}}  </span>
                        </div>
                        <span class="title">{{shop.description}}</span>
                        
                    </v-layout>
                </v-container>
            </v-layout>

            <v-container grid-list-lg>
                <v-layout row wrap fill-height>
                    <v-flex v-for="(good,index) in combos" v-bind:key="index+'combo'" lg3 md4 xs12>
                        <Card
                            :url="'/img/'+good.id+'.jpg'"
                            :good="good"
                            type="套餐"
                            v-on:chosen="addToCart"
                        ></Card>
                    </v-flex>
                    <v-flex v-for="(good,index) in goods" v-bind:key="index" lg3 md4 xs12>
                        <Card
                            :url="'/img/'+good.id+'.jpg'"
                            :good="good"
                            type="单品"
                            v-on:chosen="addToCart"
                        ></Card>
                    </v-flex>
                </v-layout>
            </v-container>
        </v-content>
        <v-dialog v-model="cartDialog" width="500">
            <v-card>
                <v-toolbar color="primary" dark>
                    <v-toolbar-title>购物车</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn icon @click="cartItems=[];refreshCart();fetchGoods();">
                        <v-icon>remove_shopping_cart</v-icon>
                    </v-btn>
                </v-toolbar>
                <v-list two-line>
                    <template v-for="(item, index) in cartItems">
                        <v-list-tile :key="index" avatar @click=";">
                            <v-list-tile-content>
                                <v-list-tile-title>{{ item.combo.name }}</v-list-tile-title>
                                <v-list-tile-sub-title
                                    class="text--primary"
                                >单价：{{ item.combo.price }}</v-list-tile-sub-title>
                                <v-list-tile-sub-title>{{ item.combo.description }}</v-list-tile-sub-title>
                            </v-list-tile-content>
                            <v-list-tile-action>
                                <v-btn icon small @click="item.number++;refreshCart();">
                                    <v-icon>add</v-icon>
                                </v-btn>
                                <v-btn icon disabled small>{{ item.number }}</v-btn>
                                <v-btn icon small @click="decrease(item,index);">
                                    <v-icon>remove</v-icon>
                                </v-btn>
                            </v-list-tile-action>
                        </v-list-tile>
                        <v-divider v-if="index + 1 < cartItems.length" :key="`divider-${index}`"></v-divider>
                    </template>
                </v-list>
                <v-divider></v-divider>

                <v-card-actions>
                    <v-chip outline >总价 {{theOrder.originPrice}}元</v-chip>
                    <v-spacer></v-spacer>
                    <v-btn
                        :disabled="this.cartItems.length==0"
                        color="primary"
                        class
                        @click="confirmOrder"
                    >下单</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="orderDialog" width="500">
            <v-card>
                <v-toolbar color="primary" dark>
                    <v-toolbar-title>核对订单和地址</v-toolbar-title>
                    <v-spacer></v-spacer>
                </v-toolbar>
                <v-list two-line>
                    <template v-for="(item, index) in cartItems">
                        <v-list-tile :key="index" avatar @click=";">
                            <v-list-tile-content>
                                <v-list-tile-title>{{ item.combo.name }}</v-list-tile-title>
                                <v-list-tile-sub-title
                                    class="text--primary"
                                >单价：{{ item.combo.price }}</v-list-tile-sub-title>
                                <v-list-tile-sub-title>{{ item.combo.description }}</v-list-tile-sub-title>
                            </v-list-tile-content>
                            <v-list-tile-action>
                                <span>{{ item.number+'份' }}</span>
                            </v-list-tile-action>
                        </v-list-tile>
                        <v-divider v-if="index + 1 < cartItems.length" :key="`divider-${index}`"></v-divider>
                    </template>
                </v-list>
                <v-divider></v-divider>
                <v-container>
                    <v-radio-group v-model="receiveInfo">
                        <v-radio
                            v-for="(receive, index) in customer.addressList"
                            :key="index"
                            :label="receive.address.name+receive.detailedAddress+','+receive.receiver+receive.gender+','+receive.phone"
                            :value="receive"
                        ></v-radio>
                    </v-radio-group>
                    <v-layout justify-space-between align-center>
                        <span>距离：{{distance}}KM</span>
                        <span v-if="distance>20">距离超过20km, 不可送货</span>
                        <v-btn @click="goToAddress">添加地址</v-btn>
                    </v-layout>
                </v-container>

                <v-divider></v-divider>
                <v-card-actions>
                    <v-chip outline>总价 {{theOrder.originPrice}}元</v-chip>
                    <v-chip
                        outline
                        @click="promotionDialog=true"
                        v-if="theOrder.promotion.benefit<1"
                    >优惠 {{theOrder.promotion.benefit}}</v-chip>
                    <v-chip outline>应付 {{theOrder.realPrice}}元</v-chip>
                    <v-spacer></v-spacer>
                    <v-btn
                        :disabled="this.cartItems.length==0||this.distance>20"
                        color="primary"
                        @click="makeOrder"
                    >确认下单</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="promotionDialog" width="500">
            <v-card>
                <v-toolbar color="primary" dark>
                    <v-toolbar-title>优惠详情</v-toolbar-title>
                    <v-spacer></v-spacer>
                </v-toolbar>
                <v-card-text>
                    <v-form ref="form" lazy-validation>
                        <v-text-field
                            prepend-icon="star"
                            v-model="theOrder.promotion.id"
                            required
                            placeholder="后端自动生成"
                            readonly
                            name="name"
                            label="优惠ID"
                            type="text"
                        ></v-text-field>
                        <v-text-field
                            prepend-icon="home"
                            v-model="theOrder.promotion.benefit"
                            readonly
                            required
                            name="name"
                            label="折扣"
                            type="text"
                        ></v-text-field>
                        <v-text-field
                            v-model="theOrder.promotion.minPrice"
                            readonly
                            prepend-icon="star"
                            name="minPrice"
                            label="最低适用价格"
                            id="minPrice"
                            type="text"
                        ></v-text-field>
                        <v-text-field
                            v-model="theOrder.promotion.startDate"
                            label="开始日期"
                            prepend-icon="event"
                            readonly
                        ></v-text-field>
                        <v-text-field
                            v-model="theOrder.promotion.endDate"
                            label="结束日期"
                            prepend-icon="event"
                            readonly
                        ></v-text-field>
                        <v-text-field
                            prepend-icon="star"
                            readonly
                            v-model="theOrder.promotion.minLevel"
                            label="最低适用会员等级"
                        ></v-text-field>
                    </v-form>
                </v-card-text>
            </v-card>
        </v-dialog>
        <v-dialog v-model="resultDialog" width="500">
            <v-card>
                <v-toolbar color="primary" dark>
                    <v-toolbar-title>{{this.orderResult.status=='success'?'下单成功':'下单失败'}}</v-toolbar-title>
                </v-toolbar>
                <v-card-text v-if="orderResult.status=='success'">
                    <div>订单编号:{{this.orderResult.order.id}}</div>
                    <div>应付:{{orderResult.order.realPrice.toFixed(2)}}</div>
                </v-card-text>
                <v-card-text v-if="orderResult.status=='fail'">{{orderResult.message}}</v-card-text>

                <v-card-actions>
                    <v-btn color="primary" @click="resultDialog=false">稍后支付</v-btn>
                    <v-spacer></v-spacer>
                    <v-btn color="primary" v-if="this.orderResult.status=='success'" @click="pay">立即支付</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <!-- <v-dialog v-model="show_QRcode" width="500">
      <v-card>
        <v-toolbar color="pink" dark>
          <v-toolbar-title>OrderId: {{this.orderResult.message}}</v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
        <v-card-text>RMB: {{this.orderResult.realPrice}}</v-card-text>
        <v-divider></v-divider>

        <v-card-actions>
          <v-btn color="primary" flat @click="show_QRcode=false">later</v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary" flat @click="pay()">paydone</v-btn>
        </v-card-actions>
      </v-card>
        </v-dialog>-->
    </v-app>
</template>

<script>
import Card from "@/components/GoodCard";
import Header from "@/components/HeaderC";
import axios from "@/plugins/axios";
import { distanceFunc } from "@/utils/util";

export default {
    components: {
        Card,
        Header
    },
    data() {
        return {
            customer: {
                addressList: []
            },
            shop: {
                address: {}
            },
            resultDialog: false,
            orderResult: {
                status: "",
                message: "",
                order: {}
            },
            show_QRcode: false,
            cartDialog: false,
            orderDialog: false,
            promotionDialog: false,
            cartItems: [],
            goods: [],
            combos: [],
            receiveInfo: {
                address: {}
            },
            promotion: {}
        };
    },
    computed: {
        theOrder() {
            let current = {
                originPrice: 0
            };
            current.receiveInfo = this.receiveInfo; //依赖receiveInfo
            current.promotion = this.promotion; //依赖promotion

            // 依赖cartItems
            for (let item of this.cartItems) {
                current.originPrice += item.combo.price * item.number;
            }
            const benefit = current.promotion.benefit;
            if (benefit > 0 && benefit < 1) {
                current.realPrice = (current.originPrice * benefit).toFixed(2);
            } else if (benefit < 0) {
                current.realPrice = (current.originPrice - benefit).toFixed(2);
            } else {
                current.realPrice = current.originPrice;
            }
            return current;
        },
        distance() {
            const lng1 = this.shop.address.lng;
            const lat1 = this.shop.address.lat;
            const lng2 = this.receiveInfo.address.lng;
            const lat2 = this.receiveInfo.address.lat;
            return distanceFunc(lng1, lat1, lng2, lat2);
        }
    },
    methods: {
        refreshCart() {
            const app = this;
            axios.post("/api/myCart", JSON.stringify(this.cartItems), {
                params: {
                    shopId: app.shop.id
                }
            });
        },
        decrease(cartItem, index) {
            cartItem.number--;
            if (cartItem.number == 0) {
                this.cartItems.splice(index, 1);
            }
            this.refreshCart();
        },

        addToCart(combo) {
            console.log(combo);
            let oldItem = this.cartItems.find(
                item => item.combo.id == combo.id
            );
            if (oldItem) {
                oldItem.number++;
            } else {
                if (combo.id.toString().indexOf("single") > 0) {
                    combo.id = null;
                }
                this.cartItems.push({
                    combo: combo,
                    number: 1,
                    dealPrice: combo.price
                });
            }
            this.refreshCart();
        },
        confirmOrder() {
            const vue = this;
            axios
                .get("/api/promotions", {
                    params: {
                        shopId: vue.shop.id,
                        price: vue.theOrder.originPrice,
                        level: vue.customer.level
                    }
                })
                .then(res => {
                    vue.promotion = res.data || {};
                    vue.cartDialog = false;
                    vue.orderDialog = true;
                });
        },
        makeOrder() {
            this.theOrder.items = this.cartItems;
            this.theOrder.receiveInfo = this.theOrder.receiveInfo;
            let vue = this;
            axios
                .post("/api/orders", this.theOrder, {
                    params: {
                        shopId: this.shop.id
                    }
                })
                .then(res => {
                    vue.cartItems = [];
                    vue.theOrder.originPrice = 0;
                    vue.refreshCart();
                    vue.fetchGoods();
                    vue.orderDialog = false;
                    vue.orderResult = res.data;
                    vue.orderResult.order = JSON.parse(res.data.message);
                    vue.resultDialog = true;
                });
        },
        // showQRcode() {
        //   let app = this;
        //   axios
        //     .get("/api/orders", {
        //       params: {
        //         orderId: this.orderResult.message
        //       }
        //     })
        //     .then(res => {
        //       app.orderResult.realPrice = res.data.realPrice;
        //       app.resultDialog = false;
        //       app.show_QRcode = true;
        //     });
        // },
        goToAddress() {
            window.location.href = "/yummy/customer/info";
        },
        pay() {
            let vue = this;
            axios
                .get("/api/pay", {
                    params: {
                        orderId: this.orderResult.order.id
                    }
                })
                .then(res => {
                    vue.resultDialog = false;
                    if (res.data.status == "fail") {
                        alert(res.data.message);
                    } else if (res.data.status == "success") {
                        alert("支付成功");
                    }
                });
        },
        fetchGoods() {
            let vue = this;
            this.goods = [];
            this.combos = [];
            vue.shop.id = window.location.search.split("shopId=")[1];
            axios
                .get("/api/goods", {
                    params: {
                        shopId: vue.shop.id
                    }
                })
                .then(res => {
                    vue.goods = res.data;
                });
            axios
                .get("/api/combos", {
                    params: {
                        shopId: vue.shop.id
                    }
                })
                .then(res => {
                    vue.combos = res.data;
                });
        }
    },
    mounted() {
        let vue = this;
        vue.shop.id = window.location.search.split("shopId=")[1];
        axios
            .get("/api/shops", {
                params: {
                    shopId: vue.shop.id
                }
            })
            .then(res => {
                vue.shop = res.data;
            });
        axios
            .get("/api/myCart", {
                params: {
                    shopId: vue.shop.id
                }
            })
            .then(res => {
                vue.cartItems = res.data;
            });

        axios
            .get("/api/customers", {
                params: {
                    customer: "currentUser"
                }
            })
            .then(res => {
                vue.customer = res.data;
                vue.receiveInfo = vue.customer.addressList[0] || {
                    address: {}
                };
            });
        this.fetchGoods();
        // axios
        //   .get("/api/promotions", {
        //     params: {
        //       shopId: vue.shopId
        //     }
        //   })
        //   .then(res => {
        //     vue.promotions = res.data;
        //   });
    }
};
</script>

<style>
</style>

