<template>
  <div id="myAccs">
    <loader v-if="loader"/>
    <h2 class="text-center">{{$ml.get('financeHistoryPage.header')}}</h2>
    <div class="switcher">
        <div @click="showDeposits" class="switcher-item" :class="{active: isDeposits}">{{$ml.get('financeHistoryPage.deposits')}}</div>
        <div @click="showWithdrawals" class="switcher-item" :class="{active: isWithdrawals}">{{$ml.get('financeHistoryPage.withdrawals')}}</div>
        <div @click="showTransfers" class="switcher-item" :class="{active: isTransfers}"> {{$ml.get('financeHistoryPage.inTrans')}} </div>
    </div>
    <div v-if="isDeposits">
      <v-row>
        <v-col
          cols="12"
          sm="8"
          md='9'
        >
        <h3>{{$ml.get('financeHistoryPage.dHistory')}}</h3>
        </v-col>
        <v-col
          cols="12"
          sm="4"
          md='3'
        >
        <input type="text" :placeholder="$ml.get('financeHistoryPage.search')" v-model="searchD" @change="getDeposits()">
        </v-col>
      </v-row>
    <v-data-table
      :search="search"
      color='white'
      :loading="loadTable"
      :page.sync="pageDep"
      :items-per-page="10"
      :headers="$ml.get('financeHistoryPage.headersDep')"
      :items="depos"
      @page-count="pageCount = $event"
      hide-default-footer
      class="elevation-1"
    >
    <template slot="no-data">
          {{$ml.get('financeHistoryPage.noData')}}
        </template>
      <template v-slot:item.status="{ item }">
        <div class="statusClass" v-if="item.status">
            <div v-if="item.status.id == 1" class="approvedClass">{{$ml.get('myDocuments.aStatus')}}</div>
            <div v-else-if="item.status.id == 2" class="decline">{{$ml.get('myDocuments.dStatus')}}</div>
            <div v-else-if="item.status.id == 0" class="pendingClass">{{$ml.get('myDocuments.pStatus')}}</div>
        </div>
    </template>
    <template v-slot:item.amount="{ item }">
        {{item.amount}} <span style="text-transform: uppercase; color: white !important">  {{item.currency}}</span>
    </template>
    </v-data-table>
     <div class="text-center pt-2">
        <v-pagination 
        total-visible=7
        class="pag"
        v-model="pageDep"
        :length="depPages"
        @change="getDeposits()"
        color="none"
        ></v-pagination>
    </div>
    <v-row>
        <v-col
          cols="12"
          sm="8"
          md='9'
        >
        <h3>{{$ml.get('financeHistoryPage.ldHistory')}}</h3>
        </v-col>
        <v-col
          cols="12"
          sm="4"
          md='3'
        >
        <input type="text" :placeholder="$ml.get('financeHistoryPage.search')" v-model="searchLD" @change="getLocalDeposits()">
        </v-col>
      </v-row>
    <v-data-table
      :search="search"
      color='white'
      :loading="loadTable"
      :page.sync="pageLocalDep"
      :items-per-page="10"
      :headers="$ml.get('financeHistoryPage.headersLocDep')"
      :items="LocalDep"
      @page-count="pageCount = $event"
      hide-default-footer
      class="elevation-1"
    >
    <template slot="no-data">
          {{$ml.get('financeHistoryPage.noData')}}
        </template>
     <template v-slot:item.status="{ item }">
        <div class="statusClass">
            <div v-if="item.status == 1" class="approvedClass">{{$ml.get('myDocuments.aStatus')}}</div>
            <div v-else-if="item.status == 2" class="decline">{{$ml.get('myDocuments.dStatus')}}</div>
            <div v-else-if="item.status == 0" class="pendingClass">{{$ml.get('myDocuments.pStatus')}}</div>
        </div>
    </template>
    <template v-slot:item.amount_from="{ item }">
        {{item.amount_from}} <span style="text-transform: uppercase; color: white !important">  {{item.currency_from}}</span>
    </template>
    <template v-slot:item.amount_to="{ item }">
        {{parseFloat(item.amount_to).toFixed(2)}} <span style="text-transform: uppercase; color: white !important">  {{item.currency_to}}</span>
    </template>
    </v-data-table>
     <div class="text-center pt-2">
        <v-pagination 
        total-visible=7
        class="pag"
        v-model="pageLocalDep" 
        :length ='LocalDepPages'
        color="none"
        @change="getLocalDeposits()"
        ></v-pagination>
    </div>
  </div>
  <div v-if="isWithdrawals">
       <v-row>
        <v-col
          cols="12"
          sm="8"
          md='9'
        >
        <h3>{{$ml.get('financeHistoryPage.wHistory')}}</h3>
        </v-col>
        <v-col
          cols="12"
          sm="4"
          md='3'
        >
        <input type="text" :placeholder="$ml.get('financeHistoryPage.search')" v-model="search" @change="getWithdrawals()">
        </v-col>
      </v-row>
    <v-data-table
      :search="search"
      color='white'
      :loading="loadTable"
      :page.sync="pageWithd"
      :items-per-page="10"
      :headers="$ml.get('financeHistoryPage.headersWith')"
      :items="withd"
      @page-count="pageCount = $event"
      hide-default-footer
      class="elevation-1"
    >
    <template slot="no-data">
          {{$ml.get('financeHistoryPage.noData')}}
        </template>
      <template v-slot:item.status="{ item }">
        <div class="statusClass" v-if="item.status">
            <div v-if="item.status.id == 1" class="approvedClass">{{$ml.get('myDocuments.aStatus')}}</div>
            <div v-else-if="item.status.id == 2" class="decline">{{$ml.get('myDocuments.dStatus')}}</div>
            <div v-else-if="item.status.id == 0" class="pendingClass">{{$ml.get('myDocuments.pStatus')}}</div>
            <div v-else-if="item.status.id == 3" class="decline">{{item.status.name}}</div>
            <div v-else-if="item.status.id == 4" class="pendingClass">{{item.status.name}}</div>
            <p class="declineComment" v-if="item.status.id == 2">{{item.status.comment}}</p>
        </div>
    </template>
    <template v-slot:item.amount="{ item }">
        {{item.amount}} <span style="text-transform: uppercase; color: white !important">  {{item.currency}}</span>
    </template>
    </v-data-table>
     <div class="text-center pt-2">
        <v-pagination 
        total-visible=7
        class="pag"
        v-model="pageWithd" 
        :length ='withdPages'
        @change="getWithdrawals()"
        color="none"
        ></v-pagination>
    </div>
    <v-row>
        <v-col
          cols="12"
          sm="8"
          md='9'
        >
        <h3>{{$ml.get('financeHistoryPage.lwHistory')}}</h3>
        </v-col>
        <v-col
          cols="12"
          sm="4"
          md='3'
        >
        <input type="text" :placeholder="$ml.get('financeHistoryPage.search')" v-model="search" @change="getLocalWithdrawals()">
        </v-col>
      </v-row>
    <v-data-table
      :search="search"
      color='white'
      :loading="loadTable"
      :page.sync="pageLocalWithd"
      :items-per-page="10"
      :headers="$ml.get('financeHistoryPage.headersLocWith')"
      :items="LocWithd"
      @page-count="pageCount = $event"
      hide-default-footer
      class="elevation-1"
    >
    <template slot="no-data">
          {{$ml.get('financeHistoryPage.noData')}}
        </template>
      <template v-slot:item.status="{ item }">
        <div class="statusClass">
            <div v-if="item.status == 1" class="approvedClass">{{$ml.get('myDocuments.aStatus')}}</div>
            <div v-else-if="item.status == 2" class="decline">{{$ml.get('myDocuments.dStatus')}}</div>
            <div v-else-if="item.status == 0" class="pendingClass">{{$ml.get('myDocuments.pStatus')}}</div>
        </div>
    </template>
    <template v-slot:item.amount_to="{ item }">
        {{parseFloat(item.amount_to).toFixed(2)}} {{item.currency_to}} 
    </template>
    <template v-slot:item.amount_from="{ item }">
        {{parseFloat(item.amount_from).toFixed(2)}} {{item.currency_from}} 
    </template>
    </v-data-table>
     <div class="text-center pt-2">
        <v-pagination 
        total-visible=7
        class="pag"
        v-model="pageLocalWithd" 
        :length ='LocWithdPages'
        @change="getLocalWithdrawals()"
        color="none"
        ></v-pagination>
    </div>

  </div>
  <div v-if="isTransfers">
      <v-row>
        <v-col
          cols="12"
          sm="8"
          md='9'
        >
        <h3>{{$ml.get('financeHistoryPage.tHistory')}}</h3>
        </v-col>
        <v-col
          cols="12"
          sm="4"
          md='3'
        >
        <input type="text" :placeholder="$ml.get('financeHistoryPage.search')" v-model="searchTr" @change="getTransfers()">
        </v-col>
      </v-row>
    <v-data-table
      :search="search"
      color='white'
      :loading="loadTable"
      :page.sync="pageTrans"
      :items-per-page="10"
      :headers="$ml.get('financeHistoryPage.headersTrans')"
      :items="transfers"
      @page-count="pageCount = $event"
      hide-default-footer
      class="elevation-1"
    >
    <template slot="no-data">
          {{$ml.get('financeHistoryPage.noData')}}
        </template>
    <template v-slot:item.PROFIT="{ item }">
       <span style="text-transform: uppercase; color: white !important" v-if="item.PROFIT < 0">{{item.PROFIT * (-1)}}</span>  
       <span style="text-transform: uppercase; color: white !important" v-else>{{item.PROFIT}}</span>
       <span style="text-transform: uppercase; color: white !important">  {{item.CURRENCY}}</span>
    </template>
      <template v-slot:item.status="{ item }">
        <div class="statusClass" v-if="item.status">
            <div v-if="item.status.id == 1" class="approvedClass">{{item.status.name}}</div>
            <div v-else-if="item.status.id == 2" class="decline">
              {{item.status.name}}
              </div>
            <div v-else-if="item.status.id == 0" class="pendingClass">{{item.status.name}}</div>
        </div>
    </template>
    </v-data-table>
     <div class="text-center pt-2">
        <v-pagination 
        total-visible=7
        class="pag"
        v-model="pageTrans" 
        :length ='transfersPages'
        color="none"
        ></v-pagination>
    </div>
  </div>
  </div>
</template>

<script>
import axios from 'axios'
import loader from '../components/loader'
export default {
  components: {
    loader
  },
  data () {
    return {
      loader: true,
      isDeposits: true,
      isWithdrawals: false,
      isTransfers: false,
      loadTable: false,
      loaded: true,
      slength: 3,
      ssss: '',
      usersTest: [],
      accounts: [],
      depPages: null,
      LocalDepPages: null,
      withdPages: null,
      locWithd: [],
      LocWithdPages: null,
      transfersPages: null,

      pageDep: 1,
      pageLocalDep: 1,
      pageWithd: 1,
      pageLocalWithd: 1,
      pageTrans: 1,
      allPages: null,


      searchD: '',
      searchLD: '',
      searchTr: '',
      searchLW: '',
      searchW: '',


      ImgUrl: 'wwww',
      depos: [],
      LocalDep: [],
      withd: [],
      LocalWithd: [],
      transfers: [],
      headers: [
        { text: 'Account#', value: 'account', sortable: false, divider: false},
        { text: 'Payment system', value: 'payment_value', align: 'center', sortable: false },
        { text: 'Amount', value: 'amount',  align: 'center', sortable: false },
        { text: 'Date', value: 'date',  align: 'center', sortable: false },
        { text: 'Status', value: 'status',  align: 'center', sortable: false },
      ],
      headersLocalDep: [
        { text: 'Account#', value: 'account', sortable: false, divider: false},
        { text: 'Payment system', value: 'payment_value', align: 'center', sortable: false },
        { text: 'Amount', value: 'amount',  align: 'center', sortable: false },
        { text: 'Date', value: 'date',  align: 'center', sortable: false },
        { text: 'Status', value: 'status',  align: 'center', sortable: false },
      ],
      headersWithdrawal: [
        { text: 'Account', value: 'account', sortable: false, divider: false},
        { text: 'Payment system', value: 'payment', align: 'center', sortable: false },
        { text: 'Amount', value: 'amount',  align: 'center', sortable: false },
        { text: 'Date', value: 'date',  align: 'center', sortable: false },
        // { text: 'Wallet', value: 'wallet',  align: 'center', sortable: false },
        { text: 'Status', value: 'status',  align: 'center', sortable: false },
      ],
      headersTrans: [
        { text: 'To accaunt', value: 'LOGIN', sortable: false, divider: false},
        { text: 'From accaunt', value: 'COMMENT', align: 'center', sortable: false },
        { text: 'Amount', value: 'PROFIT',  align: 'center', sortable: false },
        { text: 'Date', value: 'OPEN_TIME',  align: 'center', sortable: false },
      ],
    }
},
mounted() {
this.getAllData()
if (this.$store.state.withdrawalRedirect == true){
  this.showWithdrawals()
  this.$store.state.withdrawalRedirect = false
}
if (this.$store.state.transferRedirect == true){
  this.showTransfers()
  this.$store.state.transferRedirect = false
}
},

watch: {  
  pageTrans: function(){
    this.getTransfers()
  },
  pageDep: function(){
    this.getDeposits()
  },
  pageLocalDep: function(){
    this.getLocalDeposits()
  },
  pageWithd: function(){
    this.getWithdrawals()
  },
  pageLocalWithd: function(){
    this.getLocalWithdrawals()
  }

},
methods:{
    showDeposits: function(){
        this.isDeposits = true,
        this.isWithdrawals = false,
        this.isTransfers = false
    },
    showWithdrawals: function(){
        this.isDeposits = false,
        this.isWithdrawals = true,
        this.isTransfers = false
    },
    showTransfers: function(){
        this.isDeposits = false,
        this.isWithdrawals = false,
        this.isTransfers = true
    },




    getDeposits: async function(){
          this.$sendMessage('finances/deposits', 0, {
              params: { 
                  page: this.pageDep,
                  'filtering=*' : this.searchD
              },
              callback: response => { 
                this.depos = response.data
                this.depPages = parseInt(response.headers['x-pagination-page-count'])
              }
          })
    },
    getLocalDeposits: async function(){
          this.$sendMessage('finances/local-deposits', 0, {
            params: { 
                  page: this.pageLocalDep,
                  'filtering=*' : this.searchLD
              },
              callback: response => { 
                this.LocalDep = response.data
                this.LocalDepPages = parseInt(response.headers['x-pagination-page-count'])
              }
          })
    },
     getWithdrawals: async function(){
          this.$sendMessage('finances/withdrawals', 0, {
            params: { 
                  page: this.pageWithd,
                  'filtering=*' : this.searchW
              },
              callback: response => { 
                this.LocalDep = response.data
                this.LocalDepPages = parseInt(response.headers['x-pagination-page-count'])
              }
          })
    },
    getLocalWithdrawals: async function(){
          this.$sendMessage('finances/local-withdrawals', 0, {
            params: { 
                  page: this.pageLocalWithd,
                  'filtering=*' : this.searchLW
              },
              callback: response => { 
                this.LocalDep = response.data
                this.LocalDepPages = parseInt(response.headers['x-pagination-page-count'])
              }
          })
    },
    getTransfers: function(){
            this.$sendMessage('finances/internal-transfers', 0, {
                params: { 
                  page: this.pageTrans,
                  'filtering=*' : this.searchTr
                },
                callback: response => {
                  this.transfers = response.data
                  this.transfersPages = parseInt(response.headers['x-pagination-page-count'])
                }
            })
      },
    getAllData: function(){
      let token = {
                Authorization: 'Bearer ' + localStorage.getItem('TOKEN'),
                'content-type': 'x-www-form-urlencoded'
            }
        axios.all([  
          axios.get(this.$store.state.address + 'finances/deposits?page=' + this.pageDep, { headers: token }),
          axios.get(this.$store.state.address + 'finances/local-deposits?page=' + this.pageLocalDep, { headers: token }),
          axios.get(this.$store.state.address + 'finances/withdrawals?page=' + this.pageWithd, { headers: token }),
          axios.get(this.$store.state.address + 'finances/local-withdrawals?page=' + this.pageLocalWithd, { headers: token }),
          axios.get(this.$store.state.address + 'finances/internal-transfers?page=' + this.pageTrans, { headers: token })
        ])
        .then((response)=>{
          this.depos = response[0].data
          this.depPages = parseInt(response[0].headers['x-pagination-page-count'])
          this.LocalDep = response[1].data
          this.LocalDepPages = parseInt(response[1].headers['x-pagination-page-count'])
          this.withd = response[2].data
          this.withdPages = parseInt(response[2].headers['x-pagination-page-count'])
          this.LocWithd = response[3].data
          this.LocWithdPages = parseInt(response[3].headers['x-pagination-page-count'])
          this.transfers = response[4].data
          this.transfersPages = parseInt(response[4].headers['x-pagination-page-count'])
          this.loader = false
        })
    }
}
}
</script>
<style lang="sass" scoped>
.statusClass
  display: flex
  flex-direction: column
  justify-content: center
  align-items: center
.declineComment
  width: 100%
  margin: 0
  padding: 3px 10px
  color: rgba(230, 125, 125, 1)
  text-align: center
#myAccs
  padding: 0 55px
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
  h2
    font-weight: 700
    font-size: 36px
    padding: 30px 0
    line-height: 1.1
    color: White
    text-transform: uppercase
  .history
    padding: 5px 7px
    font-weight: 700
    text-transform: uppercase
    font-size: 12px
    border: 2px solid #39c5c5
    transition: all .5s
    &:hover
      background-color: #39c5c5
  span
    color: red !important
  input 
    border: 2px solid rgb(57, 197, 197)
    width: 100%
    max-width: 250px
    padding: 5px 5px
.decline
    padding: 3px 10px
    background-color: #d2155533
    border: 1px solid #d21555
    color: #dc395f
    max-width: 150px
    min-width: 150px
    text-align: center
.pendingClass
    padding: 3px 10px
    background-color: #eaf18b33
    border: 1px solid #eaf18b
    color: #eaf18b
    max-width: 150px
    min-width: 150px
    text-align: center
.approvedClass
    padding: 3px 10px
    background-color: #90e25233
    border: 1px solid #90e252
    color: #90e252
    max-width: 150px
    min-width: 150px
    text-align: center
.v-data-table
  background-color: rgba(255, 255, 255, 0)
  tbody
    color: red !important
.v-text-field  
  color: white !important

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

@media screen and (max-width: 576px)
  #myAccs
    padding: 0 0
    input
      max-width: 100% !important
  h3
    text-align: center
</style>