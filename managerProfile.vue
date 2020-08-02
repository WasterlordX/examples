<template>

    <div class="profile-container">
      <modal
        v-show="isSuccesModal"
        @close="hideSuccesModal">
        <div>
            <h4 class="text-center">
                <span>Successfull</span>
            </h4>        
            <p class="text-center">You successfully update manager</p> 

            <div class="modalButtons">
                <button class="submitButton"  @click="hideSuccesModal" >OK</button> 
            </div> 
        </div>    
        </modal>
        <div class="row">
            <div class="col-md-6 personalInfo">
                <div class="personalInfo-container">
                    <div class="top">
                        <h5>Personal Info</h5>
                        <p>CHANGE MANAGER'S PERSONAL DATA</p>
                    </div>
                    <div class="bottom-pers">
                        <div class="bottom-pers-item">
                            <h5>ID</h5>
                            <input type="number" name="id" id="id" v-model="mainInfo.id">
                        </div>
                        <div class="bottom-pers-item">
                            <h5>Email</h5>
                            <input type="email" name="email" id="email" v-model="mainInfo.email">
                        </div>
                        <div class="bottom-pers-item">
                            <h5>Name</h5>
                            <input  type="text" name="name" id="name" v-model="mainInfo.name">
                        </div>
                        <div class="bottom-pers-item">
                            <h5>Phone</h5>
                            <input type="number" name="phone" id="phone" v-model="mainInfo.phone">
                        </div>
                        <div class="bottom-pers-item">
                            <h5>Country</h5>
                            <input v-if="mainInfo.country" type="email" name="contry" id="country" v-model="mainInfo.country.name">
                        </div>
                    </div>
                </div>
                <div class="methods_modules">
                    <div class="top">
                        <h5>Personal Info</h5>
                        <p>CHANGE MANAGER'S PERSONAL DATA</p>
                    </div>
                    <div class="modulesBlock">
                  <div class="modules">
                    <div class="modules-item"  v-for="(value, key) in methods" :key="key">
                      <div class="modulesLi" :style="accessColor(key)"> 
                        <div style="display:block; width: 100%;">
                          <div>{{key}}</div>
                          <div style="font-size:10px;background-color:rgba(0, 0, 0, 0.15);">{{value.description}}</div>
                        </div>
                      </div>
                      <div class="arrow" 
                        :class="{active: selected_method == key}" 
                        @click="changeSelected(key)" 
                        >
                        <font-awesome-icon class="icon-eye" :icon="['fas', 'angle-double-right']" size="2x" />
                      </div>
                    </div>
                  </div>
                 <div class="methods">
                   <div class="methods-container" v-if="selected_method !== null">
                     <div class="block-header">
                        <h4 class="text-left" style="text-transform:uppercase;">{{selected_method}}</h4>
                        <button v-if="!isAllSelected()" @click="selectAll()">Select all</button>
                        <button v-if="methods[selected_method].count !== 0" @click="unselectAll()">Unselect all</button>
                     </div>
                        <div style="display:block;width:100%;margin-bottom:10px;border-bottom: 1px solid rgba(0,0,0, 0.3);padding-bottom: 5px">
                          <span class="text-left" style="font-width: 12px;color:#555;">{{methods[selected_method].description}}</span>
                        </div>
                     <div class="methods-container-item" v-for="(item, vkey) in methods[selected_method]" :key="vkey">
                        <div v-if="typeof(item) === 'object'" style="width: 100%;border-bottom: 1px solid #CCC;margin-bottom:5px;">
                          <div style="width:10%;float:left"><input type="checkbox" :id="item.id" v-model="item.enabled" @change="switchModule(vkey)"/></div>
                          <div style="width:90%;float:left"><label :for="item.id">{{item.description}}</label></div>
                        </div>
                     </div>
                  </div>
                </div>
              </div>
                </div>
            </div>
            <div class="col-md-6 settings">
                <div class="settings-container">
                    <div class="top">
                        <h5>settings</h5>
                        <p>CHANGE MANAGER SETTINGS</p>
                    </div>
                    <div class="bottom-settings">
                       <div class="bottom-settings-item">
                            <h5>Rank</h5>
                            <select v-if="mainInfo.rank" name="rank" id="rank" v-model="mainInfo.rank.id">
                                <option value="1">Super admin</option>
                                <option value="2">Admin</option>
                                <option value="3">Supervisor</option>
                                <option value="4">Manager</option>
                                <option value="5">Support manager</option>
                                <option value="6">Regional partner</option>
                            </select>
                        </div>
                        <div class="bottom-settings-item">
                            <h5>Supervisor</h5>
                             <select name="rank" id="rank">
                                <!-- <option v-for="item in mainInfo.supervisor" :value="item.name" :key="item">{{item.name}}</option> -->
                                
                            </select>
                        </div>
                        <div class="bottom-settings-item">
                            <h5>Countries access</h5>
                                <div class="selectElement">
                                    <div v-if="!checkboxCountry">
                                        <multiselect v-model="mainInfo.countries_access" tag-placeholder="Add this as new tag" placeholder="Search or add a tag" label="name" track-by="code" :options="countryList" :multiple="true" :taggable="true"></multiselect>
                                    </div>
                                    <div class="forCheckbox">
                                        <input type="checkbox" id="countryCheck" class="checkClass" v-model="checkboxCountry">
                                        <label for="countryCheck">Allow access to all countries</label>
                                    </div>
                                </div>
                        </div>
                        <div class="bottom-settings-item">
                            <h5>Partners access</h5>
                            <div class="selectElement">
                                <div v-if="!checkboxPartner">
                                    <multiselect v-model="mainInfo.partners_access" tag-placeholder="Add this as new tag" placeholder="Search or add a tag" label="user_email" track-by="id" :options="subIbsList" :multiple="true" :taggable="true"></multiselect>
                                </div>
                                <div class="forCheckbox">
                                    <input type="checkbox" class="checkClass" id="partnerCheck" v-model="checkboxPartner">
                                    <label for="partnerCheck">Allow to view referrals from all partners</label>
                                </div>
                            </div>
                        </div>
                       
                    </div>
                </div>
                 <div class="adi-container">
                    <div class="top">
                        <h5>Additional settings</h5>
                        <p>SET ADDITIONAL SETTINGS FOR MANAGERS</p>
                    </div>
                    <div class="bottom" style="padding: 20px 0">
                        <div class="forCheckbox">
                            <input type="checkbox" class="checkClass" id="furtherCheck" @change="checkPartnerMode()" v-model="mainInfo.reg_partner">
                            <label for="furtherCheck">Regional Partner Mode</label>
                        </div>
                    </div>
                </div>
            </div>
            <div class="center-button">
                <v-btn small class="btn-send" @click="updateProfile">Update profile</v-btn>
            </div>
        </div>
    </div>
</template>
<script>
import modal from '@/components/testModal'
import Multiselect from 'vue-multiselect'
import axios from 'axios'
import { isObject } from 'util'
import qs from 'qs'
export default {
    components: {
        Multiselect,
        modal
    },
    data: () => ({
      isSuccesModal: false,
      mainInfo: [],
      checkboxCountry: false,
      checkboxPartner: false,
      subIbsList: [],
      selected_method: null,
      methods: {}
    }),
    computed: {
      countryList() {
        return this.$store.getters.getCountres.reverse();
      },
      countryCheck(){
        if (this.mainInfo.countries_access == null) {
          return true
        } else {
          return false
        }
      }      
    },
    mounted(){
        this.getInfo()
        this.getSubs()        
    },
    methods: {    
      switchModule: function(key)
      {
        if (this.methods[this.selected_method][key].enabled) this.methods[this.selected_method].count++
        else this.methods[this.selected_method].count--
      },
      changeSelected: function(key)
      {
        if (this.selected_method === key) this.selected_method = null;
        else this.selected_method = key;
      },
      isAllSelected: function()
      {
        return Object.keys(this.methods[this.selected_method]).length - 3 === this.methods[this.selected_method].count;
      },
      selectAll()
      {
        for(let [key, value] of Object.entries(this.methods[this.selected_method]))
          if (isObject(value) && value.enabled === false) {
            this.methods[this.selected_method][key].enabled = true
            this.methods[this.selected_method].count++
          }
      },
      unselectAll()
      {
        for(let [key, value] of Object.entries(this.methods[this.selected_method]))
          if (isObject(value) && value.enabled === true) {
            this.methods[this.selected_method][key].enabled = false
            this.methods[this.selected_method].count--
          }
      },
      accessColor: function(key)
      {
        if (this.methods[key].count === 0) return '';
        if (Object.keys(this.methods[key]).length - 3 === this.methods[key].count) return "background-color:#CFC";
        else return "background-color:#CCF";
      },    
      hideSuccesModal: function(){
        this.isSuccesModal = false
      },
      checkPartnerMode: function(){
        if (this.mainInfo.reg_partner){
          this.mainInfo.reg_partner = 1
        } else {this.mainInfo.reg_partner = 0}
      },
      getInfo: function(){
            this.$sendMessage('admin/' + this.$route.params.id, 0, {
              callback: response => {
                this.methods = response.data[0].accesses
                this.mainInfo = response.data[0]
                if (this.mainInfo.countries_access == null)
                  this.checkboxCountry = true
                if (this.mainInfo.partners_access == null)
                  this.checkboxPartner = true
              }
            })
        },
         updateProfile: function(){
            let sendCountry = '';
            let sendPartner = '';
            let moduleString = '';

            for (let [key, value] of Object.entries(this.methods))
            {
              let val = 0;
              for (let item of Object.values(value)) if (typeof item === 'object' && item.enabled === true) val += item.id;
              if (val > 0) moduleString += (moduleString === '') ? key + ":" + val : "," + key + ":" + val;
            }           

            if (this.checkboxPartner) {
              sendPartner = null
            } else {
              let counter = 0;
              let partnerValues = '';
             for (let key of this.mainInfo.partners_access){
               if ((this.mainInfo.partners_access.length - 1) != counter){  
                  partnerValues += key.id + ',';
                } else {
                 partnerValues += key.id;
               }
               counter++
             }
             sendPartner =  partnerValues
           }
           if (this.checkboxCountry){
             sendCountry = ''
           } else {
             let counter = 0;
             for (let item of this.mainInfo.countries_access){
               if ((this.mainInfo.countries_access.length - 1) != counter){
                 sendCountry += item.code + ','
                 counter++
                } else {sendCountry += item.code}
             }
           }
          this.$sendMessage('admin/' + this.$route.params.id, 1, {
            form: {
              email: this.mainInfo.email,
              name: this.mainInfo.name,
              country: this.mainInfo.country.code,
              phone: this.mainInfo.phone,
              modules_access: moduleString,
              partners_access: sendPartner,
              countries_access: sendCountry,
              reg_partner: this.mainInfo.reg_partner,
              rank: this.mainInfo.rank.id,
            },
            callback: response => {
              this.isSuccesModal = true
              this.getInfo()
            }
          })
        },
         getSubs: function(){
           this.$sendMessage('partnership/partners', 0, {
             params: { 'per-page': 1000 },
             callback: response => {this.subIbsList = response.data }
           })   
        },
    }

}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style lang="sass" scoped>
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
.block-header
  display: flex
  justify-content: space-between
  align-items: center
  button  
    border-radius: 5px
    font-size: 13px
    border: 1px solid red
    padding: 3px 5px
  h4
    margin: 0
.selectElement
    width: 70%
    select
        border-radius: 6px
        padding-left: 7px
        width: 100%
        border: 1px solid rgba(0, 0, 0, 0.2)
        margin: 5px
    p
        padding-left: 10px
        font-size: 13px
        opacity: 0.6
.forCheckbox
  padding-left: 10px
  display: flex
  justify-content: flex-start
  align-items: center
  .checkClass
    width: 15px !important
    height: 15px
    margin-bottom: 2px
    margin-right: 5px
  label
    margin: 0
.personalInfo
    padding-left: 15px
    display: flex
    flex-direction: column
    justify-content: center
    align-items: center
    &-container
        border-radius: 4px
        width: 95%
        border: 1px solid rgba(0,0,0, 0.2)
        height: 100%
        .top
            padding: 15px
            width: 100%
            border-bottom: 1px solid rgba(0,0,0, 0.2)
            display: flex
            flex-direction: column
            justify-content: center
            p
                opacity: 0.6
                font-size: 11px
                text-transform: uppercase
                margin: 0
            h5
                font-size: 15px
                text-transform: uppercase
                margin: 0
        .bottom-pers
            display: flex
            flex-direction: column
            padding: 20px 0
            &-item
                padding: 12px 0
                display: flex
                align-items: center
                width: 100%
                h5
                    padding-left: 20px
                    width: 25%
                input
                    border-radius: 4px
                    width: 70%
                    padding: 5px 10px
                    border: 1px solid rgba(0,0,0, 0.3)
    .methods_modules
        margin-top: 25px
        border-radius: 4px
        width: 95%
        border: 1px solid rgba(0,0,0, 0.2)
        height: 100%
        .top
            padding: 15px
            width: 100%
            border-bottom: 1px solid rgba(0,0,0, 0.2)
            display: flex
            flex-direction: column
            justify-content: center
            p
                opacity: 0.6
                font-size: 11px
                text-transform: uppercase
                margin: 0
            h5
                font-size: 15px
                text-transform: uppercase
                margin: 0
        
        .modulesBlock
            display: flex
            padding: 10px 0
            .modules
                padding-left: 10px
                width: 35%
                &-item
                    display: flex
                    margin-top: 5px
                    align-items: center
                .modulesLi
                    margin-right: 5px
                    width: 90%
                    height: 35px
                    border: 2px solid rgba(0,0,0, 0.3)
                    border-radius: 5px
                    display: flex
                    justify-content: center
                    align-items: center
                    text-transform: uppercase
                    font-size: 13px
                    font-weight: bold
                .arrow
                    display: flex
                    justify-content: center
                    border-radius: 5px
                    align-items: center
                    height: 35px
                    width: 35px
                    border: 2px solid rgba(0,0,0, 0.3)
                .active
                    border: 2px solid rgba(0,200,0, 0.3)
                    background-color: rgba(0, 200, 0, 0.3)
                .actives
                    border: 2px solid rgba(0,200,0, 0.3)
                    background-color: rgba(0, 200, 0, 0.3)
            .methods  
                width: 65%
                padding-left: 7px
                &-container
                    padding: 5px
                    margin-top: 4px
                    height: calc(100% - 4px)
                    &-item
                        display: flex
                        align-items: center
                        input
                            width: 15px
                            min-width: 15px
                            height: 15px
                        label
                            margin-bottom: 0
                            margin-left: 5px

                    
.settings
    padding-left: 15px
    &-container
        border-radius: 4px
        width: 95%
        border: 1px solid rgba(0,0,0, 0.2)
        .top
            padding: 15px
            width: 100%
            border-bottom: 1px solid rgba(0,0,0, 0.2)
            display: flex
            flex-direction: column
            justify-content: center
            p
                opacity: 0.6
                font-size: 11px
                text-transform: uppercase
                margin: 0
            h5
                font-size: 15px
                text-transform: uppercase
                margin: 0
        .bottom-settings
            display: flex
            flex-direction: column
            padding: 20px 0
            &-item
                padding: 12px 0
                display: flex
                align-items: center
                width: 100%
                h5
                    padding-left: 20px
                    width: 25%
                input
                    border-radius: 4px
                    width: 70%
                    padding: 5px 10px
                    border: 1px solid rgba(0,0,0, 0.3)
                select
                    border-radius: 4px
                    width: 70%
                    padding: 5px 10px
                    border: 1px solid rgba(0,0,0, 0.3)

                    
.adi
    padding-left: 15px
    &-container
        margin-top: 25px
        border-radius: 4px
        width: 95%
        border: 1px solid rgba(0,0,0, 0.2)
        .top
            padding: 15px
            width: 100%
            border-bottom: 1px solid rgba(0,0,0, 0.2)
            display: flex
            flex-direction: column
            justify-content: center
            p
                opacity: 0.6
                font-size: 11px
                text-transform: uppercase
                margin: 0
            h5
                font-size: 15px
                text-transform: uppercase
                margin: 0
.center-button
    display: flex
    justify-content: center
    width: 100%
    padding: 15px 0
.btn-send
    background-color: red !important
    color: white !important
    outline: none
</style>

