<template>
     <section class="main-part-content">
         <form style="display:none;" :action="actionAdress" :method="actionMethod">
             <div v-for="(object, index) in actionFieds" :key="index" v-html="object"/>
             <button type="submit" id="submit_button" />
         </form>
        <modalDep v-if="paySelected !== false" @close="cleanForm">
            <div class="modal-center">
                <div class="paymentSystemText">
                    {{$ml.get('depositPage.paySys')}}
                </div>
                <img :src="paySystems[paySelected].img" :alt="paySystems[paySelected].name">
                <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.account')}}</h5>
                    <select name="nettAcc" v-model="payData.login">
                        <option v-for="(acc, index) in accounts" :key='index' :value="acc.LOGIN">{{acc.LOGIN}}</option>
                    </select>
                </div>
                <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.currency')}}</h5>
                    <select name="nettAcc" v-model="payData.currency">
                        <option v-for="(system, index) in paySystems[paySelected].currencyes" :key="index" :value="system">{{system}}</option>
                    </select>
                </div>
                <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.amount')}}</h5>
                    <input type="number" v-model="payData.amount" min="0" @change="checkPay()">
                </div>
                <div class="dataPayment"  v-if="paySystems[paySelected].show_name === true">
                    <h5>{{$ml.get('depositPage.name')}}</h5>
                    <input type="text" v-model="payData.name">
                </div>
                <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.bcode')}}</h5>
                    <input type="text" v-model="payData.bonus_code">
                </div>
                <p v-if="deposit_error" class="error_message">{{deposit_error}}</p>
                <div class="deposit">
                    <button @click="deposit()">{{$ml.get('depositPage.createReq')}}</button>
                </div>
            </div>
        </modalDep>
        <modalDep 
            v-if="showLocalDep"
            @close="hideLocalDep"
            >
            <div class="modal-center">
                 <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.account')}}</h5>
                   <select name="localAcc" v-model="localDeposit.account" @change="localDeposit.cur = localDeposit.account.CURRENCY">
                        <option v-for="(acc, index) in accounts" :key='index' :value="acc">{{acc.LOGIN}}</option>
                    </select>
                </div>
                <div class="row">
                    <div class="col-md-6">
                        <div class="dataPayment">
                            <h5>{{$ml.get('depositPage.amount')}} {{$ml.get('depositPage.from')}}</h5>
                            <input type="number" min=0 @change="amountTo()" @keyup="amountTo()"  v-model="localDeposit.ammount">
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="dataPayment">
                            <h5>{{$ml.get('depositPage.amount')}} {{$ml.get('depositPage.to')}}</h5>
                            <input type="number" min=0  @change="amountFrom()" @keyup="amountFrom()"  v-model="ammountTo">
                        </div>
                    </div>
                </div>
                    <div class="dataPayment">
                        <h5>{{$ml.get('depositPage.currency')}}</h5>
                        <input type="text" disabled v-model="localDeposit.cur">
                    </div>
                    <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.bcode')}}</h5>
                    <input type="text"  v-model="localDeposit.bcode">
                </div>
                <div class="dataPayment">
                    <h5>{{$ml.get('depositPage.comment')}}</h5>
                    <textarea v-model="localDeposit.comment" />
                </div>       
                <p v-if="deposit_error" class="error_message">{{deposit_error}}</p>
                <div class="deposit">
                    <button @click="DepositLocal()">{{$ml.get('depositPage.createReq')}}</button>
                </div>
            </div>
        </modalDep>
         <h2 class="text-center" style="padding-bottom: 20px">{{$ml.get('depositPage.header')}}</h2>
                <div class="deposit" >
                    <div class="switcher">
                        <div @click="showPayment" class="switcher-item" :class="{active: isPayment}">{{$ml.get('depositPage.paySys')}}</div>
                        <div  @click="showLocal" class="switcher-item" :class="{active: isLocal}">{{$ml.get('depositPage.locDep')}}</div>
                    </div>                   
                    <div v-if="isPayment">
                        <div class="deposit-paySys">
                            <div class="deposit-paySys-main row">
                                <div 
                                    class="col-md-6 col-sm-12 col-lg-4 col-xl-3 deposit-paySys-main-item"
                                    v-for="(object, index) in paySystems" :key="index"
                                >
                                    <div class="deposit-paySys-main-item-container">
                                        <img :src="object.img" class="img-responsive" :alt="object.name"> 
                                        <div class="deposit-paySys-main-item-container-text">
                                            <p>{{$ml.get('depositPage.currency')}} - <span>{{object.currencyes.join(', ')}}</span> </p>
                                            <p>{{$ml.get('depositPage.commis')}} - <span>{{object.commission}}</span></p>
                                            <p>{{$ml.get('depositPage.procesTime')}} - <span>{{$ml.get('depositPage.inst')}}</span></p>
                                        </div>
                                        <div class="deposit_button"><button @click="paySelected = index">{{$ml.get('depositPage.deposit')}}</button></div>
                                        </div>                
                                </div>
                            </div>                        
                        </div>
                    </div>
                    <div v-if="isLocal" class="deposit-locWith">
                        <div class="deposit-locWith-main row">
                                <div v-for="(item, key) in depositers" :key='key' class="deposit-locWith-main-item col-md-6">
                                    <div class="deposit-locWith-main-item-container">
                                        <div class="deposit-locWith-main-item-container-left col-md-6 col-12">
                                            <h4 class="text-center">{{item.name}}</h4>
                                            <img :src="$store.state.clean_address +'avatars/mld/' + item.avatar" class="responsive" style="min-height: 100px; min-width: 100px; max-width: 100px" alt="avatar">
                                            <div v-for="bank in item.banks" :key="bank.name" class="forBanks" @mouseenter="currentBank = bank.bankName; currentMLD = key" @mouseleave="currentMLD = '';currentBank = 'key'">
                                                <span>{{bank.bankName}}</span> 
                                                <div v-if="currentBank == bank.bankName && currentMLD == key"  class="bankInfo">
                                                    <p><span style="font-size:14px;font-weight:bold;margin-right:5px;">Name:</span> {{bank.name}}</p> 
                                                    <p v-if="bank.bankAccount.length > 0"><span style="font-size:14px;font-weight:bold;margin-right:5px;">Bank account:</span> {{bank.bankAccount}}</p> 
                                                </div>
                                            </div>
                                        </div>
                                        <div class="deposit-locWith-main-item-container-right col-md-6 col-12">
                                           <p><i class="fa fa-envelope"></i>{{item.email}}</p>
                                           <p v-if="item.whatsapp && item.whatsapp != '-'"><i class="fa fa-phone"></i>{{item.whatsapp}}</p>
                                           <p v-if="item.telegram && item.telegram != '-'"><i class="fa fa-telegram"></i>{{item.telegram}}</p>
                                           <p v-if="item.skype && item.skype != '-'"><i class="fa fa-skype"></i>{{item.skype}}</p>
                                           <p v-if="item.facebook && item.facebook != '-'"><i class="fa fa-facebook"></i>{{item.facebook}}</p>
                                        </div> 
                                        <div class="deposit-locWith-main-item-container-bottom row">
                                            <div class="col-md-6 col-sm-6 text-center">
                                                <!-- <p>{{$ml.get('depositPage.from')}}</p> -->
                                                <p>Deposit rate</p>
                                                <h4> 1 USD = {{item.deposit_rate_from}} {{item.deposit_from_currency}}</h4>
                                            </div>                                       
                                            <div class="col-md-6 col-sm-6 bottom_button">
                                                <button @click="showLocalDep = true; localDepositerNumber = key">{{$ml.get('leftMenu.deposit')}}</button>
                                            </div>                                       
                                        </div>                                      
                                    </div>
                                </div>
                                
                              
                    </div>
                    </div>
                </div>
        </section>
</template>
<script>
import modalDep from '../components/depModal'
import axios from 'axios'
import { async } from 'q'
import qs from 'qs'
export default {
    components: {
        modalDep,
    },
    mounted(){
        this.totalAccs = this.$store.state.totalAccs
        this.getAccounts()
        this.getLocalDepositers()
        this.payData.name = this.get_cookie('name')
    },
    computed: {
        getAvatar: function(){
            return this.$store.state.address
        } 
    },
    data(){
        return{
            actionFieds: [],
            actionAdress: '',
            actionMethod: '',
            actionClick: false,

            payData: {                
                login: null,
                currency: '',
                amount: 0,
                bonus_code: null,
                payment_system: '',
                name: '',
            },
            paySelected: false,
            currentBank: '',
            currentMLD: '',
            paySystems: [
                /*{
                    name:       'neteller',
                    img:        require('@/assets/withdrawalsImgs/neteller.svg'),
                    currencyes: [ 'USD', 'EUR' ],
                    commission: "0%",
                },*/
                {
                    name:       'skrill',
                    img:        require('@/assets/withdrawalsImgs/skrill.svg'),
                    currencyes: [ 'USD' ],
                    commission: "0%",
                },
                {
                    name:       'pay_trust',
                    img:        require('@/assets/withdrawalsImgs/123.svg'),
                    currencyes: [ 'IDR', 'VND', 'THB', 'MYR' ],
                    commission: "0%",
                    show_name: true,
                },
                {
                    name:       'fasapay',
                    img:        require('@/assets/withdrawalsImgs/fasapay.svg'),
                    currencyes: [ 'USD', 'IDR' ],  
                    commission: "0%",
                },
                {
                    name:       'perfect_money',
                    img:        require('@/assets/withdrawalsImgs/perMoney.svg'),
                    currencyes: [ 'USD', 'EUR' ],
                    commission: "0%",
                },
                /*{
                    name:       'pay_center',
                    img:        require('@/assets/withdrawalsImgs/card.svg'),
                    currencyes: [ 'USD' ],
                    commission: "0%",
                },*/
                /*{
                    name:       'fx',
                    img:        require('@/assets/withdrawalsImgs/card.svg'),
                    currencyes: [ 'USD', 'EUR' ],
                    commission: "0%",
                },*/
                /*{
                    name:      'crypto',
                    img:        require('@/assets/withdrawalsImgs/crypto.svg'),
                    currencyes: [ 'BTC', 'LTC', 'ETC' ], 
                    commission: "0%",
                },*/
            ],
            localDeposit: {
                cur: 'usd',
                account: [], 
                ammount: 0,
                bcode: null,
                name: null,
                comment: null
            },
            ammountTo: null,
            totalAccs: null,
            accounts: [],
            depositers: null,
            isPayment: true,
            isLocal: false,
            showLocalDep: false,
            localDepositerNumber: 1,
            deposit_error: null,
        }
    },
    updated: function () {
        this.$nextTick(function (){
            if (this.actionClick) {
                this.actionClick = false
                document.getElementById('submit_button').click()
                }
        })
    },
    created(){ 
        document.addEventListener('click', this.dropdown)
    },
    destroyed () {
        document.removeEventListener('click', this.dropdown)
    },
    methods:{
        dropdown(e){
            let el = this.$refs.dropdown;
            let target = e.target;
            if (el !== target &&el && !el.contains(target)){
                this.currentBank = '',
                this.currentMLD = ''
            }
        },
        hideLocalDep: function()
        {
            this.showLocalDep = false
        },
        get_cookie: function (string)
            {
            let results = document.cookie.match('(^|;) ?' + string + '=([^;]*)(;|$)');
            if ( results )
                return ( unescape ( results[2] ) )
            else
                return null;
        },  
        amountTo: function(){
            let to = this.localDeposit.ammount*(1/this.depositers[this.localDepositerNumber].deposit_rate_from)
            this.ammountTo = to.toFixed(2)
        },
        amountFrom: function(){
            let from = this.ammountTo*(this.depositers[this.localDepositerNumber].deposit_rate_from/1)
            this.localDeposit.ammount = from.toFixed(2)
        },
        getLocalDepositers: async function(){
            this.$sendMessage('finances/local-depositors', 0, {
                callback: response => { this.depositers = response.data }
            })
        },
        getAccounts: async function(){
            this.$sendMessage('mt4/trade-accounts', 0, {
                params: {'per-page' : 100},
                callback: response => { this.accounts = response.data }
            })
        },
        showPayment: function(){
            this.isPayment = true,
            this.isLocal = false
        },
         showLocal: function(){
            this.isPayment = false,
            this.isLocal = true
        },
        cleanForm: function()
        {
            this.deposit_error = ''
            this.paySelected = false            
            this.payData = {
                login: null,
                currency: '',
                amount: 0,
                bonus_code: null,
                payment_system: '',
                name: this.get_cookie('name'),
            };
        },
        checkPay: function() {
            let min = 1
            let max = 1000000
            let system = this.paySystems[this.paySelected]
            if (typeof(system.min) === 'number') min = system.min
            if (typeof(system.max) === 'number') max = system.max
            this.payData.amount = Math.min(max, Math.max(min, parseFloat(this.payData.amount))).toFixed(2)
        },
        deposit: async function()
        {
            this.payData.payment_system = this.paySystems[this.paySelected].name;
            if (this.paySystems[this.paySelected].show_name !== true) this.payData.name = null
             this.$sendMessage("finances/deposits", 1, 
                {                    
                    form: this.payData,                   
                    callback: response => {
                        this.actionAdress = response.data.address
                        this.actionMethod = response.data.method
                        this.actionFieds = new Array();
                        for(let [key, value] of Object.entries(response.data.fields))
                            this.actionFieds.push('<input name="' + key + '" value="' + value + '"/>')                    
                        this.actionClick = true
                    },
                    error_handler: error => {
                        this.deposit_error = error.response.data.code
                    }
                }
            )
        },
         DepositLocal: async function(){
            this.$sendMessage('finances/local-deposits', 1, {
                form : {
                    local_depositor: this.depositers[this.localDepositerNumber].id,
                    login: this.localDeposit.account.LOGIN,
                    currency: this.localDeposit.cur,
                    amount: this.localDeposit.ammount,
                    bonus_code: this.localDeposit.bcode,
                    comment: this.localDeposit.comment
                },
                callback: response => {
                    this.hideLocalDep()
                    this.$router.push('/finance-history')
                },
                error_handler: response => {
                    this.deposit_error = error.response.data.message
                }
            })
        },


    }
}
</script>
<style lang="sass" scoped>
.forBanks
    border-radius: 5px
    margin-top: 10px
    padding: 3px 5px
    font-size: 13px
    background: rgba(255, 255, 255, 0.1)
    min-width: 150px
    position: relative
    display: flex
    justify-content: center
    align-items: center
    .bankInfo
        position: absolute
        padding: 5px 10px
        left: 50%
        top: 100%
        transform: translateX(-50%)
        width: 100%
        background-color: #001c22
        z-index: 500
h2
    padding: 30px 0
    text-transform: uppercase
    font-weight: 700
    font-size: 36px
    line-height: 1.1
    color: white
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button 
  -webkit-appearance: none
  margin: 0
input[type=number] 
  -moz-appearance: textfield

input:disabled
    background-color: rgba(255, 255, 255, 0.2)
.error_message
    color: #ea6363
    margin: 5px 0
.dataPayment
    width: 100%
    h5
        width: 100%
.paymentSystemText
    position: absolute
    left: 8px
    top: 8px
.modal-center
    width: 100%
    display: flex
    flex-direction: column
    align-items: center
    justify-content: center
    img
        width: 200px
    .dataPayment
        margin-top: 10px
        position: relative
        h5
            font-size: 16px
            margin-bottom: 3px
        select
            outline: none
            width: 100%
            border: 1px solid #39c5c5
            padding: 8px 12px
            option
                color: black
        input
            width: 100%
            outline: none
            border: 1px solid #39c5c5
            padding: 8px 12px
        textarea
            width: 100%
            outline: none
            border: 1px solid #39c5c5
            padding: 8px 12px
        .minDep
            position: absolute
            bottom: -15px
            right: 0
            font-size: 10px
            opacity: 0.7

    .deposit
        display: flex
        justify-content: center
        align-items: center
        button
            width: 100%
            padding: 5px 10px
            font-weight: 700
            text-transform: uppercase
            font-size: 15px
            border: 2px solid #39c5c5
            transition: all .5s
            margin-top: 15px
            min-width: 240px
            text-align: center
            &:hover
                background-color: #39c5c5
                box-shadow: 0px 0px 25px 10px #39c5c595
.deposit
    display: flex
    flex-direction: column
    .deposit-locWith
        border: 1px solid rgba(255, 255, 255, 0.2)
        padding: 6px 12px
        &-main
            border: 1ps solid white
            &-item
                padding: 5px 10px
                display: flex
                flex-direction: column
                justify-content: space-between
                &-container
                    display: flex
                    flex-wrap: wrap
                    height: 100%
                    width: 100%
                    border: 1px solid #ddd
                    &-left
                        width: 50%
                        border-right: 1px solid rgba(0, 0, 0, 0.4)
                        padding: 10px
                        display: flex
                        flex-direction: column
                        align-items: center
                        justify-content: flex-start
                        
                        img
                            border-radius: 50%
                            max-width: 100px
                          
                        h4	
                            font-size: 25px
                            font-weight: bold
                    &-right
                        width: 50%
                        padding: 50px 20px
                        min-height: 270px
                        i
                            margin-right: 15px
                            width: 10px
                    .row
                        margin-left: 0 !important
                        margin-right: 0 !important
                    p
                        font-size: 12px
                        line-height: 12px
                        color: rgba(255, 255, 255, 0.9)
                    &-bottom
                        background-color: rgba(255, 255, 255, 0.4)
                        display: flex
                        justify-content: space-between
                        width: 100%
                        color: white
                        padding: 10px 0
                        .bottom_button
                            width: 100%
                            display: flex
                            justify-content: center
                            align-items: center
                            button
                                padding: 5px 10px
                                font-weight: 700
                                text-transform: uppercase
                                font-size: 15px
                                border: 2px solid #39c5c5
                                transition: all .5s
                                text-align: center
                                &:hover
                                    background-color: #39c5c5
                                    box-shadow: 0px 0px 25px 10px #39c5c595
                        
            
    &-top
        margin: 0 
        border: 1px solid rgba(255, 255, 255, 0.2)
        background-color: rgba(255, 255, 255, 0.1)
        text-align: center
        &-left, &-right
            padding: 10px 0
        .active
            background-color: rgba(255, 255, 255, 0.4)
    &-border
        border: 1px solid #ddd
        border-radius: 5px
    &-paySys
        border: 1px solid rgba(255, 255, 255, 0.2)
        padding: 0px 12px
        &-top
            border-radius: 5px
            text-align: center
            font-size: 17px
            font-weight: bold
            text-transform: uppercase
            padding: 15px 0 10px 0
        &-main
            border-radius: 5px
            &-item
                &-container		
                    justify-content: space-between
                    display: flex
                    flex-direction: column	
                    align-items: center
                    border-radius: 5px
                    width: 100%
                    height: 100%
                    background-color: rgba(255, 255, 255, 0.15)
                    border: 1px solid rgba(255, 255, 255, 0.2)
                    text-align: center
                    padding-top: 20px
                    &-text
                        padding-right: 10px
                        padding-left: 10px
                        line-height: 100%
                        font-size: 13px
                        span
                            font-weight: bold
                    img

                        width: 100%
                        max-height: 100px
                        margin-bottom: 15px
                    .deposit_button
                        margin-top: 15px
                        width: 100%
                        height: 60px
                        display: flex
                        justify-content: center
                        align-items: center
                        border-top: 1px solid rgba(255, 255, 255, 0.2)
                        button
                            width: 80%
                            padding: 5px 10px
                            font-weight: 700
                            text-transform: uppercase
                            font-size: 15px
                            border: 2px solid #39c5c5
                            transition: all .5s
                            min-width: 100px
                            text-align: center
                            &:hover
                                background-color: #39c5c5
                                box-shadow: 0px 0px 25px 10px #39c5c595

.switcher
    display: flex
    &-item
        font-weight: 600
        text-transform: uppercase
        font-size: 15px
        display: inline-block
        padding: 13px 15px
        margin: 0
        list-style: none
        cursor: pointer
        float: left
        border: 1px solid #ffffff17
    .active
        background-color: #53be76
        
@media screen and (max-width: 580px) 
   .switcher
    display: flex
    flex-direction: column
    margin-bottom: 20px
    &-item
        display: flex !important
        justify-content: center
        align-items: center
        padding: 13px 15px
        margin: 0
        width: 100%
</style>