<template>
    <div class="goods-sheet" v-if="show">
        <div class="goods-sheet-mask" @click="acionSheet"></div>
        <div :class="show ? 'putShow goods-sheet-view' : 'goods-sheet-view'">
            <div class="goods-thumb" :style="{backgroundImage: 'url(' + data.pro_thumb + '!_600x600)' }"></div>
            <div class="goods-sheet-info">
                <div class="goods-sheet-name font-break">{{data.pro_name}}</div>
                <div class="goods-sheet-price">￥{{data.pro_price}}</div>
            </div>
            <div class="goods-sheet-item">
                <div class="goods-sheet-item-title">颜色</div>
                <div class="goods-sheet-item-context">
                    <span v-for="(item, index) in data.pro_sizeTable" :key="'attr' + index" :class="colorIndex === index ? 'active-select-tab' : ''" @click="setColor" :data-index="index">{{item.color}}</span>
                </div>
            </div>
            <div class="goods-sheet-item">
                <div class="goods-sheet-item-title">尺码</div>
                <div class="goods-sheet-item-context">
                    <span :class="sizeIndex === index ? 'active-select-tab' : ''" v-for="(item, index) in data.pro_sizeTable[0].attr" :key="'attr-sku' + index" @click="setSize" :data-index="index">{{item.size}}</span>
                </div>
            </div>
            <div class="goods-sheet-item">
                <div class="goods-sheet-item-title">数量</div>
                <div class="goods-sheet-item-count">
                    <div class="goods-sheet-remove" @click="removeCount">-</div>
                    <div class="goods-sheet-review">{{param.num}}</div>
                    <div class="goods-sheet-add" @click="addCount">+</div>
                </div>
            </div>
            <div class="goods-sheet-bar"><span @click="saveOrder">立即购买</span><span @click="saveCart">加入购物车</span></div>
        </div>
    </div>
</template>

<script>
    import { Toast } from 'mint-ui'
    import API from '@/util/api'

    export default {
        name: "GoodsSheet",
        props: {
            data: {
                type: Object
            },
            show: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                param: {
                    sku: null,
                    id: null,
                    num: 1,
                    attr: []
                },
                colorIndex: null,
                sizeIndex: null
            }
        },
        components: {
            
        },
        created() {
            
        },
        methods: {
            saveCart() {
                if(this.sizeIndex === null || this.colorIndex === null) {
                    Toast({
                        message: "请选择参数!",
                        duration: 1000
                    })
                    return
                }
                let data = {
                    id: this.data.pro_code_bar,
                    attr: [this.data.pro_sizeTable[this.colorIndex].color, this.data.pro_sizeTable[this.colorIndex].attr[this.sizeIndex].size],
                    num: this.param.num,
                    sku: this.data.pro_sizeTable[this.colorIndex].attr[this.sizeIndex].sku
                }
                this.$ajax.post(API.addCart(), {
                    order: [data]
                }).then(res => {
                    Toast({
                        message: res.msg,
                        duration: 1000
                    })
                    this.$store.commit("cart/initCart", res.result)
                    this.$emit("toggleSheet")
                }).catch(err => {
                    console.log("添加购物车失败", err)
                })
            },
            saveOrder() {
                if(this.sizeIndex === null || this.colorIndex === null) {
                    Toast({
                        message: "请选择参数!",
                        duration: 800
                    })
                    return
                }
                let data = [{
                    pro_code_bar: this.data.pro_code_bar,
                    pro_attr: [this.data.pro_sizeTable[this.colorIndex].color, this.data.pro_sizeTable[this.colorIndex].attr[this.sizeIndex].size],
                    pro_num: this.param.num,
                    pro_sku: this.data.pro_sizeTable[this.colorIndex].attr[this.sizeIndex].sku,
                    pro_thumb: this.data.pro_thumb,
                    pro_name: this.data.pro_name,
                    pro_price: this.data.pro_price,
                    pro_brand: this.data.pro_brand
                }]
                this.$store.dispatch("order/createNewOrder",{order:data})
                this.$router.push({
                    name: "YohoOrderConfirm"
                })
            },
            acionSheet() {
                this.$emit("toggleSheet")
            },
            setColor(e) {
                this.colorIndex = parseInt(e.target.dataset.index)
            },
            setSize(e) {
                this.sizeIndex = parseInt(e.target.dataset.index)
            },
            removeCount() {
                this.param.num = (this.param.num > 1 ? this.param.num - 1 : 1)
            },
            addCount() {
                this.param.num++
            },
            rand(min, max) {
                return Math.floor(Math.random() * (max - min + 1)) + min
            },
            crypt(num) {
                let len = num,
                    str = "0123456789",
                    result = ""
                while(len--) {
                    result += str[this.rand(0, str.length - 1)]
                }
                return result
            }
        },
        activated() {
            this.param = {
                sku: null,
                id: null,
                num: 1,
                attr: []
            }
            this.colorIndex = null
            this.sizeIndex = null
        },
        filters: {

        }
    }
</script>

<style>
    .goods-sheet {
        width: 100vw;
        height: 100vh;
        position: fixed;
        left: 0;
        top: 0;
        z-index: 2000;
        overflow: hidden;
    }
    
    .goods-sheet-mask {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background: rgba(0, 0, 0, 0.8);
        animation-name: maskShow;
        animation-duration: .2s;
        animation-fill-mode: forwards;
        animation-timing-function: ease-in;
    }
    
    @keyframes maskShow {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }
    
    .putShow {
        animation-name: putInfo;
        animation-duration: .2s;
        animation-fill-mode: forwards;
        animation-timing-function: ease-out;
    }
    
    @keyframes putInfo {
        from {
            bottom: -60vw;
        }
        to {
            bottom: 0px;
        }
    }
    
    .goods-sheet-view {
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100vw;
        height: 60vh;
        background: #FFF;
        padding: 4vw;
    }
    
    .goods-thumb {
        width: 28vw;
        height: 32vw;
        position: absolute;
        left: 5vw;
        top: -12vw;
        background-size: cover;
        background-position: center center;
        background-repeat: no-repeat;
        box-shadow: 0 2px 7px rgba(0, 0, 0, .15);
    }
    
    .goods-sheet-bar {
        width: 100vw;
        height: 46px;
        position: absolute;
        left: 0;
        bottom: 0;
        color: #FFF;
        font-size: 14px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    
    .goods-sheet-bar span {
        display: inline-block;
        width: 50%;
        height: 46px;
        line-height: 46px;
        text-align: center;
        background: #D0021B;
    }
    
    .goods-sheet-bar span:nth-child(1) {
        background: #039EE6;
    }
    
    .goods-sheet-info {
        width: 92vw;
        height: 20vw;
        border-bottom: 1px solid #F4F4F4;
        padding-left: 32vw;
    }
    
    .goods-sheet-name {
        font-size: 14px;
        font-weight: 550;
        line-height: 20px;
        max-height: 40px;
        -webkit-line-clamp: 2;
    }
    
    .goods-sheet-price {
        font-size: 14px;
        font-weight: 550;
        color: #D0021B;
        line-height: 26px;
    }
    
    .goods-sheet-item {
        width: 92vw;
        min-height: 12vw;
        display: flex;
        align-items: center;
        font-size: 14px;
        border-bottom: 1px solid #F4F4F4;
    }
    
    .goods-sheet-item-title {
        font-weight: 550;
        color: #333;
        width: 12vw;
        flex: 0 0 auto;
    }
    
    .goods-sheet-item-context {
        width: 80vw;
        flex: 1 1 auto;
        min-height: 10vw;
        display: flex;
        align-items: center;
        justify-content: flex-start;
        flex-wrap: wrap;
        padding-top: 10px;
    }
    
    .goods-sheet-item-context span {
        min-width: 12vw;
        height: 32px;
        background: #FFF;
        border: 1px solid #333;
        border-radius: 6px;
        display: inline-block;
        padding: 0 10px;
        line-height: 30px;
        text-align: center;
        margin-right: 4vw;
        margin-bottom: 10px;
    }
    
    .goods-sheet-item-context .active-select-tab {
        background: #D0021B;
        border: 1px solid #D0021B;
        color: #FFF;
    }
    
    .goods-sheet-item-count {
        width: 120px;
        height: 28px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    
    .goods-sheet-remove,
    .goods-sheet-add {
        width: 30px;
        height: 28px;
        text-align: center;
        line-height: 26px;
        border: 1px solid #999;
        color: #999;
    }
    
    .goods-sheet-review {
        font-weight: 550;
        color: #333;
    }
</style>