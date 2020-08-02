<template>
  
  <div class="documents">
    <modal
        v-show="approveModal"
        @close="hideApproveModal">
        <div>
            <h4 class="text-center">
                <span>Approving</span>
            </h4>        
            <p class="text-center">Are sure that you want to approve this document? </p> 
            <div class="aprrovingDate">
            <input type="date" id="start" :disabled='perpetual' :min="formatedData" name="trip-start" v-model="approveDate" @change="checkDate()">
            <p style="font-size: 13px">Expire date</p>
            <div>
                <input type="checkbox" id="Perpetual" v-model="perpetual" @change="approveDate = null">
                <label style="margin-left: 5px" for="Perpetual">Perpetual document</label>
            </div>
            </div>
            <div class="modalButtons">
                <button @click="approveDoc(docId)" class="submitButton">OK</button> 
                <button @click="hideApproveModal" class="cancelButton">Cancel</button>
            </div>
            
        </div>    
        </modal>
         <modal
        v-show="declineModal"
        @close="hideDeclineModal">
        <div>
            <h4 class="text-center">
              Declining
            </h4>        
            <p class="text-center">Are sure that you want to decline this document?  </p> 
            <textarea name="comment" id="declineComm" rows="5" placeholder="Wright comment here" v-model="declineComment"></textarea>
            <div class="modalButtons">
                <button @click="declineDoc(docId)" class="submitButton">OK</button>
                <button @click="hideDeclineModal" class="cancelButton">Cancel</button>
            </div>
            
        </div>    
        </modal>
    <div class="wrapper">
      <preloader v-if="loaded"/>
      <transition>
      <div v-show="showMod" id="myModal" class="modal" @click="hideModal">
      <img v-if="imgDoc.extension" :src="'data:image/' + imgDoc.extension +';base64,' + imgDoc.document" alt="avatar" @click.stop>
      </div>
      </transition>

    <v-row>
        <v-col
          cols="9"
          sm="9"
        >
        <select v-model="statusSelect">
              <option value="">All</option>
              <option value="0">Pending</option>
              <option value="1">Approved</option>
              <option value="2">Declined</option>
              <option value="4">Expired</option>
              <option value="3">Expires soon</option>
        </select>
        </v-col>
        <v-col
          cols="3"
          sm="3"
        >
        <v-text-field  
        @keydown.enter="getDocuments"
          v-model="search"
          label="Search"
        ></v-text-field>
        </v-col>
      </v-row>
    <v-data-table
      :page.sync="page"
      :items-per-page="20"
      :headers="headers"
      :items="docs"
      @page-count="pageCount = $event"
      hide-default-footer
      class="elevation-1"
    >
    <template v-slot:item.user_id="{ item }">
        <router-link :to="{name: 'profile', params: {id:item.user_id}}">{{item.user_id}}</router-link>
       
      </template>
      <template v-slot:item.document="{ item }">
        <button class="watch-button" @click="showModal(item.id)"><font-awesome-icon class="icon-eye" :icon="['far', 'eye']" /><span>Watch</span></button>
      </template>
      <template v-slot:item.email="{ item }">
        {{item.email}}
        <!-- <img :src="'data:image/png;base64,' + countryList.flag" alt="flag"> -->
      </template>
      <template v-slot:item.status="{ item }">
        <button v-if="item.status.id == 0" class="wAprrove">{{item.status.name}}</button>
        <button v-else-if="item.status.id == 2" class="decline">{{item.status.name}}</button>
         <button v-else-if="item.status.id == 1" class="approve">{{item.status.name}}</button>
         <button v-else-if="item.status.id == 3" class="expired">{{item.status.name}}</button>
         <button v-else class="approve">{{item.status.name}}</button>
      </template>
       <template v-slot:item.type="{ item }">
         {{item.type.name}}
      </template>
      <template v-slot:item.confirm="{ item }">
        <div class="forButtons" v-if="item.status.id == 0">
          <button style="margin-right: 5px" @click="showApproveModal(item.id)" class="button-approve-all"><font-awesome-icon class="icon-eye" :icon="['fas', 'check']"/></button> 
          <button @click="showDeclineModal(item.id)" class="button-close-all"><font-awesome-icon class="icon-eye" :icon="['fas', 'times']"/></button>
        </div>
        <div v-else-if="item.status.id == 3 || item.status.id == 4">
        </div>
        <div v-else class="buttons-in-ends">
          <button @click="showDeclineModal(item.id)" v-if="item.status.id == 1" class="button-close-all"><font-awesome-icon class="icon-eye" :icon="['fas', 'times']"/></button>
          <button @click="showApproveModal(item.id)" v-else class="button-approve-all"><font-awesome-icon class="icon-eye" :icon="['fas', 'check']"/></button> 
        </div>
      </template>
    </v-data-table>
     <div class="text-center pt-2">
        <v-pagination v-if="allPages"
        total-visible="7"
        class="pag"
        v-model="page" 
        :length="allPages"
        color="rgba(255, 0, 0, 0.75)"
        circle
        ></v-pagination>
    </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
import { async } from 'q'
import preloader from '../components/preloader'
import modal from '../components/testModal'
import qs from 'qs'
export default {
components: {
  preloader,
  modal
},
data () {
    return {
      perpetual: false,
      test: '',
      loaded: true,
      approveDate: null,
      approveModal: false,
      declineModal: false,
      statusSelect: '',
      formatedData: '',
      DocNumber: "",
      imgDoc: 2,
      showMod: false,
      declineComment: '',
      docs: [],
      docId: null,
      page: 1,
      allPages: null,
      search: '',
      test: 'wwwaaaa',
      ImgUrl: 'wwww',
      headers: [
        { text: 'ID', value: 'id', align: 'center' },
        { text: 'UID', value: 'user_id', width: "6%", sortable: false, divider: false},
        { text: 'Email', value: 'user_email', width: "20%", align: 'center' },
        { text: 'Country', value: 'country',  align: 'center' },
        { text: 'Document', value: 'document',  align: 'center' },
        { text: 'Uploaded', value: 'timestamp_upload',  align: 'center' },
        { text: 'Type', value: 'type',  align: 'center' },
        { text: 'Status', value: 'status',  align: 'center' },
        { text: 'Expiration date', value: 'expire_date',  align: 'center' },
        { text: '', value: 'confirm',  width: "7%"},
      ],
    }
},
watch: {
  page: function (){
    this.getDocuments()
  },
  statusSelect: function(){
    this.getDocuments()
  }
},
mounted(){
  this.todayDate()
  this.getDocuments()
},
created(){
    this.$store.commit('setPageName', 'Documents')
    this.$store.commit('setinfAboutPage', 'LIST OF USER DOCUMENTS FOR VERIFICATION')
},
computed: {
    getUrl: function(){
      return this.ImgUrl
    },
    countryList() {
    return this.$store.getters.getCountres;
  }
},
methods: {
    todayDate(){
      let date = new Date()
      let day, month
      if (date.getMonth() < 10){
        month = '0' + (date.getMonth() + 1)
      } else {month = (date.getMonth() + 1)}
      if (date.getDate() < 10){
        day = '0' + date.getDate()
      } else { day = date.getDate()}
      this.formatedData = date.getFullYear() + '-' + month + '-' + day
    },
    showDeclineModal: function(id){
      this.declineModal = true
      this.docId = id
    },
    hideDeclineModal: function(){
      this.declineModal = false
    },
    showApproveModal: function(id){
      this.approveModal = true
      this.docId = id
    },
    hideApproveModal: function(){
      this.approveModal = false
    },
    getDocuments: async function(){
      let filter = '&filtering=*=' + this.search + ',status=' + this.statusSelect
      // let filter = '&filterField=status&filterValue=' + this.statusSelect
      // if (this.statusSelect == ''){
      //   filter = ''
      // }
      let token = {
        Authorization: 'Bearer ' + localStorage.getItem('TOKEN'),
      }
       await axios.get(this.$store.state.linkToServer + 'documents?page=' + this.page + filter, {
            headers: token
          })
      .then(response => {
            this.allPages = parseInt(response.headers['x-pagination-page-count'])
            this.docs = response.data    
            this.loaded = false
      })
    },
    getColor () {
      return 'green'
    },
    approveDoc: function(id){
      let expDate;
      if (this.approveDate == null){
        expDate = 'never'
      } else {
        expDate = this.approveDate + ' 00:00:00'
      }
      let token = {
        Authorization: 'Bearer ' + localStorage.getItem('TOKEN'),
      }
      axios.post(this.$store.state.linkToServer + 'documents/' + id + '/approve', qs.stringify({
          expire: expDate
        }), {
            headers: token
          })
        .then(response =>{
          this.getDocuments()
          this.approveModal = false
        })
      
    },
    declineDoc: function(id){
      let token = {
        Authorization: 'Bearer ' + localStorage.getItem('TOKEN'),
      }
      axios.post(this.$store.state.linkToServer + 'documents/' + id + '/decline', qs.stringify({
          comment: this.declineComment
        }), {
            headers: token
          })
      .then(response =>{
        this.getDocuments()
        this.declineModal = false
        this.declineComment = ''
      })
    },
    checkDate: function(){
      let date = new Date()
     if (this.approveDate.substring(4,5) != '-'){
        this.approveDate = this.formatedData
      }
    },
    showModal(test){
      this.showMod = true
      let token = {
        Authorization: 'Bearer ' + localStorage.getItem('TOKEN'),
      }
      axios.get(this.$store.state.linkToServer + 'documents/' + test, {
        headers: token
      })
      .then(response => {
            this.imgDoc = response.data     
      })
    },
    hideModal(){
      this.showMod = false
      this.imgDoc = {}
    }
  }
}
</script>

<style lang="sass" scoped>
.disabledClass
  background-color: rgba(0,0,0,0.2) !important
.aprrovingDate
  display: flex
  flex-direction: column
  justify-content: center
  input
    padding: 5px
    border: 1px solid rgba(0,0,0,0.15)
    &:disabled
      opacity: 0.4
  p
    opacity: 0.4
#declineComm
  border: 1px solid rgba(0,0,0,0.15)
  padding: 3px 8px
  width: 100%
.modalButtons
    margin-top: 15px
    display: flex
    justify-content: center
    align-items: center
.submitButton
    border-radius: 5px
    padding: 6px 25px
    color: white
    margin: 10px
    background-color: rgb(66, 140, 1)
.cancelButton
    border-radius: 5px
    padding: 6px 25px
    color: white
    margin: 10px
    background-color: rgb(221, 51, 51)  
.buttons-in-ends
  display: flex
  justify-content: center
.forButtons
  display: flex
.documents
    background-color: rgb(242, 242, 242)
    padding: 20px
.wrapper
    position: relative
    padding: 20px
    border: 1px solid rgb(218, 218, 218)
    background-color: rgb(255, 255, 255)
@media screen and (max-width: 576px)
  .documents
    background-color: rgb(242, 242, 242)
    padding: 10px
  .wrapper
    padding: 10px
    border: 1px solid rgb(218, 218, 218)
    background-color: rgb(255, 255, 255)
select
  border-radius: 3px
  width: 200px
  padding: 5px
  margin-left: 20px
  margin-bottom: 20px
  margin-top: 20px
  outline: none
  border: 1px solid rgba(0,0,0, 0.2)
.watch-button
  display: flex
  align-items: center
  justify-content: center
  width: 100%
  background-color: #fff
  padding: 5px
  border-radius: 5px
  border: 1px solid black
  transition: all 0.5s ease
  span
    font-size: 12px
    margin-left: 4px
  i
    margin-bottom: 2px
  &:hover
    color: white
    border: 1px solid white
    background-color: black

#myImg 
  border-radius: 5px
  cursor: pointer
  transition: 0.3s


#myImg:hover 
  opacity: 0.7
  
.modal 
  position: fixed
  z-index: 2200
  display: flex
  justify-content: center
  align-items: center
  left: 0
  top: 0
  width: 100%
  height: 100%
  background-color: rgb(0,0,0)
  background-color: rgba(0,0,0,0.9)
  img
    overflow-y: auto
    max-width: 80%
    max-height: 100%
    z-index: 21


.modal-content 
  margin: auto
  display: block
  width: 80%
  max-width: 700px

#caption 
  margin: auto
  display: block
  width: 80%
  max-width: 700px
  text-align: center
  color: #ccc
  padding: 10px 0
  height: 150px

.modal-content, #caption 
  -webkit-animation-name: zoom
  -webkit-animation-duration: 0.6s
  animation-name: zoom
  animation-duration: 0.6s

.close
  color: white
  position: absolute
  top: 20px
  right: 20px 
  cursor: pointer
.expiredS
  width: 100%
  font-size: 12px
  background-color: rgba(128, 0, 128, 0.2)
  color: rgba(128, 0, 128, 0.7)
  border-radius: 5px	
  padding: 5px
  border: 1px solid rgba(128, 0, 128, 1)
.expired
    width: 100%
    font-size: 12px
    background-color: rgba(244, 215, 89, 0.2) 
    color: rgba(244, 215, 89, 1)
    border-radius: 5px	
    padding: 5px
    border: 1px solid rgba(244, 215, 89, 1)
.decline
    width: 100%
    font-size: 12px
    background-color: rgba(210, 120, 120, 0.2)
    color: rgba(210, 120, 120, 0.7)
    border-radius: 5px	
    padding: 5px
    border: 1px solid rgba(210, 120, 120, 1)
.wAprrove
    width: 100%
    font-size: 12px
    background-color: rgba(120, 120, 210, 0.2)
    color: rgba(120, 120, 210, 0.7)
    border-radius: 5px	
    padding: 5px
    border: 1px solid rgba(120, 120, 210, 1)
.approve
    width: 100%
    font-size: 12px
    background-color: rgba(120, 210, 120, 0.2)
    color: rgba(120, 210, 120, 0.7)
    border-radius: 5px	
    padding: 5px
    border: 1px solid rgba(120, 210, 120, 1)



.button-close-all
    outline: none
    background-color: #fff
    border: none
    width: 35px
    height: 35px
    border: 1px solid rgb(186, 48,48)
    color: rgb(186, 48,48)
    border-radius: 5px
    transition: all 0.4s ease
    &:hover
        background-color: rgb(186, 48,48)
        color: white
.button-approve-all
    outline: none
    background-color: #fff
    width: 35px
    height: 35px
    border: none
    border: 1px solid rgb(66, 140, 1)
    color: rgb(66, 140, 1)
    border-radius: 5px
    transition: all 0.2s ease
    &:hover
        background-color: rgb(66, 140, 1)
        color: white
.profileButton
  background-color: black
  color: white
  width: 30px
  height: 30px
  border-radius: 5px
  border: none
.flag
  width: 30px
.about
  overflow-y: hidden
.test
  display: flex
  justify-content: flex-end
</style>