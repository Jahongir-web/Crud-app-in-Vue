<template>
  <div class="container">
    <div class="d-flex align-items-baseline justify-content-around">
      <h2 class="text-center m-3">Our products</h2> 
      <button @click="openModal" type="button" class="btn btn-success" :disabled="showModal">+ Add Prodact</button>
    </div>
    <div class="components-box">      
      <div v-if="showModal" class="modal-prodact">
        <button @click="reset" type="button" class="btn btn-secondary close-btn">X</button>
        <h1 class="text-center mt-3 text-primary">Create Prodact</h1>
        <form class="form-prodact" @submit="createProdact">
          <input v-model="prodact.name_uz" type="text" placeholder="Enter Prodact Name..." class="form-control mb-3" required/>
          <select v-model="prodact.product_type_id" class="form-control mb-3" name="product_type_id" id="product_type_id" aria-valuenow="hhhh" required>
            <option value="" disabled selected hidden>Select Prodact Type...</option>
            <option v-for="type in prodactTypes" :value="type.id" :key="type.id">{{type.name_uz}}</option>
          </select>
          <input v-model="prodact.cost" type="number" placeholder="Enter Prodact Coast..." class="form-control mb-3" required/>
          <input v-model="prodact.address" type="text" placeholder="Enter Address..." class="form-control mb-3" required/>
          <div class="d-flex flex-row-reverse">
            <button class="btn btn-success my-3" type="submit">Save Prodact</button>
          </div>
        </form>
      </div>
      <prodact-list :addedProdact="addedProdact"/>
    </div>
  </div>
</template>


<script>
import ProdactList from "./components/ProdactList";
import axios from "axios";

export default {
  components: {
    ProdactList,
  },
  data() {
    return {
      showModal: false,
      addedProdact: false,
      prodactTypes:[],
      prodact: {
        product_type_id: "",
        name_uz: "",
        cost: "",
        address: "",
        created_date: new Date(),
      }
    }
  },

  methods: {
    reset() {
      this.showModal=false;
      this.prodact.name_uz = "";
      this.prodact.product_type_id = "";
      this.prodact.cost = "";
      this.prodact.address = "";
    },

    async createProdact(e) {
      e.preventDefault();      
      try {
        await axios.post(`http://94.158.54.194:9092/api/product`, this.prodact);
        this.addedProdact = true;
        this.reset();
      } catch (error) {
        console.log(error);
      }
    },

    async getProdactId() {    
      try {
        const res = await axios.get(`http://94.158.54.194:9092/api/product/get-product-types`);
        this.prodactTypes = res.data;
      } catch (error) {
        console.log(error);
      }
    },

    openModal() {
      this.showModal=true;
      this.addedProdact=false;
      this.getProdactId();
    }
  },
  
}
</script>

<style >
  .components-box {
    position: relative;
  }

  .modal-prodact {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: rgb(236, 231, 231);
    z-index: 20;
    border-radius: 8px;
  }

  .form-prodact {
    position: absolute;
    width: 500px;
    top: 20%;
    left: calc(50% - 250px);
    border: 2px solid gray;
    padding: 20px;
    border-radius: 8px;
    background-color: rgb(255, 255, 255);
  }

  .close-btn {
    position: absolute;
    right: 1%;
    top: 2%;
  }
</style>