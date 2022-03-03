<template>
  <div class="prodact-list">
      
    <div v-if="showEditModal" class="modal-prodact">
      <button @click="showEditModal=false" type="button" class="btn btn-secondary close-btn">X</button>
      <h1 class="text-center mt-3 text-primary">Update Prodact</h1>
      <form class="form-prodact" @submit="updateProdact">
        <input v-model="changeProdact.name_uz" type="text" placeholder="Enter Prodact Name..." class="form-control mb-3" required/>
        <select v-model="changeProdact.product_type_id" class="form-control mb-3" name="product_type_id" id="product_type_id" aria-valuenow="hhhh" required>
          <option :value="changeProdact.product_type_id" disabled selected hidden>{{changeProdact.product_type_id}}</option>
          <option v-for="type in prodactTypes" :value="type.id" :key="type.id">{{type.name_uz}}</option>
        </select>
        <input v-model="changeProdact.cost" type="number" placeholder="Enter Prodact Coast..." class="form-control mb-3" required/>
        <input v-model="changeProdact.address" type="text" placeholder="Enter Address..." class="form-control mb-3" required/>
        <div class="d-flex flex-row-reverse">
          <button class="btn btn-success my-3" type="submit">Save Prodact</button>
        </div>
      </form>
    </div>
  

    <h2 v-if="products.length == 0" class="text-center text-danger">There are no products on page {{page}}!</h2>
    <transition-group name="list">
      <div class="prodact-item w-50 m-auto my-3" v-for="product in products" :key="product.id">
        <h5>Prodact Name: {{product.name_uz}}</h5> 
        <div class="d-flex w-75">
          <p class="me-5"><b class="text-danger">Address:</b> {{product.address}}</p>
          <p><b class="text-danger">Cost:</b> {{product.cost}}</p>
        </div>
        <div class="d-flex w-75">
          <p class="me-4"><b>prodact type id:</b> {{product.product_type_id}}</p>
          <p><b>Date:</b> {{(new Date(product.created_date).toDateString())}}</p>
        </div>
        <div class="d-flex justify-content-end">
          <button @click="openEditModal(product)" type="button" class="btn btn-info mx-2">Edit Prodact</button>
          <button @click="deleteProdact" type="button" class="btn btn-danger" :id="product.id">Delete Prodact</button>
        </div>
      </div>
    </transition-group>
  </div>
  <div v-if="!showEditModal" class="pagination d-flex justify-content-between w-25 m-auto pt-3">
    <button :disabled="page==1" @click="page=page-1" type="button" class="btn btn-primary">Prev -</button>
    <h5 class="mt-2">{{page}}</h5>
    <button :disabled="products.length<3" @click="page=page+1" type="button" class="btn btn-primary">+ Next</button>
  </div>
</template>

<script>
import axios from "axios";
import { defineComponent } from 'vue';
import { createToast } from 'mosha-vue-toastify';
import 'mosha-vue-toastify/dist/style.css';

export default defineComponent({
  name: 'myApp',
  setup () {
    const toast = (message) => {
      createToast(message)
    }
    return { toast }
  },
  props: {
    addedProdact: {
      type: Boolean,
      required: true,
    }
  },
  data() {
    return {
      loading: false,
      page: 1,
      showEditModal: false,
      products: [],
      changeProdact: {},
      prodactTypes:[],
    }
  },
  methods: {
    async getProdacts(page) {
      try {
        const res = await axios.get(`http://94.158.54.194:9092/api/product?page=${page}&perPage=3`);
        this.products = res.data;
        this.loading = false;
      } catch (error) {
        console.log(error);
      }
    },  
    async deleteProdact(e) {
      try {
        const check = confirm("Do you really want to delete the product?");
        if(check){
          const res = await axios.delete(`http://94.158.54.194:9092/api/product/${e.target.id}`)
          this.toast(res.data)
          this.loading = true;
        }
      } catch (error) {
        console.log(error);
      }
    },  
    
    async getProdactId() {    
      try {
        const res = await axios.get(`http://94.158.54.194:9092/api/product/get-product-types`);
        this.prodactTypes = res.data
        console.log(this.prodactTypes);
      } catch (error) {
        console.log(error);
      }
    },

    openEditModal(product){
      this.changeProdact = product;
      this.showEditModal=true;
      this.getProdactId()
    },

    async updateProdact(e){
      e.preventDefault();
      
      try {
        const res = await axios.put(`http://94.158.54.194:9092/api/product/`, this.changeProdact)
        this.toast(res.data)
        this.showEditModal=false;
        this.loading = true;
      } catch (error) {
        console.log(error);
      }
    }
  },

  mounted() {
    this.getProdacts(this.page);
  },

  watch: { 
    loading(value) {
      if(value){
        this.getProdacts(this.page);
      }
    },
    page(value) {
      this.getProdacts(value);
    },
    addedProdact(value){
      if(value){
        this.toast('New product is inserted!')
        this.getProdacts(this.page);
      }
    }
    
  }
})

</script>

<style scoped>
  .prodact-item {
    border: 2px solid rgb(228, 217, 236);
    border-radius: 8px;
    margin: 15px auto;
    padding: 10px;
    box-shadow: -8px 8px rgb(238, 229, 229);
    transition: 0.3s ease;
  }

  .prodact-item:hover {
    cursor: pointer;
    transform: scale(1.02);
  }

  .prodact-list{
    min-height: 590px;
    overflow: hidden;
    position: relative;
  }

  .list-enter-active {
    transition: all 0.5s ease;
  }
  .list-leave-active {
    transition: all 0s;
    z-index: -5;
  }

  .list-enter-from,
  .list-leave-to {
    opacity: 0;
    transform: translateX(30px);
  }
</style>