<template>
  <v-container grid-list-md>
    <v-row align="center" justify="center">
      <v-col class="align-center">
        <v-row class="align-center" justify="center">
          <v-col cols="12" sm="10" md="10" lg="6" class="align-center">
            <v-alert v-if="showMsg === 'error'"
                     dismissible
                     :value="true"
                     type="error"
            >
              Please verify Stock information.
            </v-alert>
          </v-col>
        </v-row>
        <v-row align="center" justify="center">
          <v-col cols="12" sm="10" md="10" lg="6" class="align-center">
            <v-card class="login-card">
              <v-card-title>
                <span class="headline">{{pageTitle}}</span>
              </v-card-title>
              <v-spacer/>

              <v-card-text>
                <v-form ref="form" lazy-validation>
                  <v-container>

                    <!--v-text-field
                    v-model="stock.cust_number"
                    label="Customer"
                    required
                    type="number"
                    /-->
                    <v-select
                    v-model="stock.customer"
                    label="Customer Number"
                    :items="list"
                    item-value='pk'
                    item-text='cust_number'
                    ></v-select>

                    <v-text-field
                    v-model="stock.symbol"
                    label="Symbol"
                    required
                    />
                    <v-text-field
                    v-model="stock.name"
                    label="Name"
                    required
                    />
                    <v-text-field
                    v-model="stock.shares"
                    label="shares"
                    required
                    type="number"
                    />
                    <v-text-field
                    v-model="stock.purchase_price"
                    label="purchase price"
                    required
                    type="number"

                    />
                    <v-text-field
                    v-model="stock.purchase_date"
                    label="purchase date"
                    required
                    type="date"
                    />
                    

                </v-container>
                <v-btn v-if="!isUpdate" class="blue white--text" @click="createStock">Save</v-btn>
                <v-btn v-if="isUpdate" class="blue white--text" @click="updateStock">Update</v-btn>
                <v-btn class="white black--text" @click="cancelOperation">Cancel</v-btn>
                </v-form>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>  
      </v-col>
    </v-row>
  </v-container>
</template>


<script>
  import router from '../router';
  import {APIService} from '../http/APIService';
  const apiService = new APIService();

  export default {
    name: 'StockCreate',
    components: {},
    data() {
      return {
        customers: [],
        showError: false,
        stock: {},
        pageTitle: "Add New Stock",
        isUpdate: false,
        showMsg: '',
      };
    },
    computed:{
      list:{
      get () {
            return this.customers
        },
          set (newValue) {
            this.customers = newValue
          }
      }
    },
    methods: {
      getCustomers() {
        apiService.getCustomerList().then(response => {
          this.customers = response.data.data;
          if (localStorage.getItem("isAuthenticates")
            && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
            this.validUserName = JSON.parse(localStorage.getItem("log_user"));
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      },
      createStock() {
        apiService.addNewStock(this.stock).then(response => {
          if (response.status === 201) {
            this.stock = response.data;
             this.showMsg = "";
            router.push('/stock-list/new');
          }else{
              this.showMsg = "error";
          }
        }).catch(error => {
          if (error.response.status === 401) {
            router.push("/auth");
          }else if (error.response.status === 400) {
            this.showMsg = "error";
          }
        });
      },
      cancelOperation(){
         router.push("/stock-list");
      },
      updateStock() {
        apiService.updateStock(this.stock).then(response => {
          if (response.status === 200) {
            this.stock = response.data;
            router.push('/stock-list/update');
          }else{
              this.showMsg = "error";
          }
        }).catch(error => {
          if (error.response.status === 401) {
            router.push("/auth");
          }else if (error.response.status === 400) {
            this.showMsg = "error";
          }
        });
      }
    },
    mounted() {
      this.getCustomers();
      if (this.$route.params.pk) {
        this.pageTitle = "Edit Stock";
        this.isUpdate = true;
        apiService.getStock(this.$route.params.pk).then(response => {
          this.stock = response.data;
        }).catch(error => {
          if (error.response.status === 401) {
            router.push("/auth");
          }
        });
      }
    },
  }
</script>
<style scoped>
  .aform {
    margin-left: auto;
    width: 60%;
  }
</style>