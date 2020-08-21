<template>
  <div>
    <b-container>
      <b-row class="justify-content-md-start mb-3">
        <b-col col lg="1">
        </b-col>
        <b-col col lg="3">
          <b-form-select v-model="selected" :options="options" @change="changstate(selected)"></b-form-select>
        </b-col>
        <b-col col lg="1">
          <b-button variant="primary" @click="reset()"><b-icon icon="arrow-counterclockwise" animation="spin-reverse" font-scale="1"></b-icon></b-button>
        </b-col>
        <b-col col lg="1">
        </b-col>
        <b-col col lg="3">
          <b-form @submit.prevent="filter">
            <b-form-input v-model="text" placeholder="Nombre o Email y Enter"></b-form-input>
          </b-form>
        </b-col>
      </b-row>
    </b-container>
    <b-container>
      <b-row class="justify-content-md-center">
        <b-col col lg="10">
          <b-table striped hover :items="items"></b-table>
        </b-col>
      </b-row>
    </b-container>
    <b-container>
		  <b-row class="justify-content-md-center">
        <b-col col lg="6">
          <b-pagination
              v-model="currentPage"
              :total-rows="rows"
              :per-page="perPage"
              class="mt-4"
          >
            <template v-slot:first-text><span class="text-success" @click="paginate(first)">Primero</span></template>
            <template v-slot:prev-text><span class="text-danger" @click="paginate(prev)">Antes</span></template>
            <template v-slot:next-text><span class="text-warning" @click="paginate(next)">Siguiente</span></template>
            <template v-slot:last-text><span class="text-info" @click="paginate(last)">Ultimo</span></template>
            <template v-slot:ellipsis-text>
              <b-spinner small type="grow"></b-spinner>
              <b-spinner small type="grow"></b-spinner>
              <b-spinner small type="grow"></b-spinner>
            </template>
            <template v-slot:page="{ page, active }">
              <span @click="PagiNum(page)">
              <b v-if="active">{{ page }}w</b>
              <i v-else>{{ page }}t</i>
              </span>
            </template>
          </b-pagination>
        </b-col>
	    </b-row>
	  </b-container>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        items: [
          
        ],
        selected: null,
        text: '',
        options: [
          { value: null, text: 'Selecciona una opción' },
          { value: '1', text: 'Activos' },
          { value: '0', text: 'Desactivos' }
        ],
        errors:[],
        rows: 1,
        perPage: 1,
        currentPage:1 ,
        first:'',
        last:'',
        prev:'',
        next:'',
        page_url:'/user/all',
		    number_page:'/user/all?page=',
      }
    },
    methods:{
        filter(){
          axios.get(`/user/search/${this.text}`).then((response)=>{
            this.number_page = `/user/search/${this.text}?page=`;
            this.parsePagination(response);
          }).catch((error)=>{
            console.log(error);
          })
        },
        listUsers(){
          axios.get(this.page_url).then((response)=>{
            this.parsePagination(response);
          }).catch((error)=>{
            console.log(error);
          });
        },
        paginate(page_url){
          this.page_url = page_url;
          this.listUsers();
        },
        PagiNum(page){
          this.page_url = this.number_page+(parseInt(page));
          this.listUsers();
        },
        changstate(state){
          axios.get(`/user/status/${state}`)
          .then((response)=>{
            this.number_page = `/user/status/${state}?page=`;
            this.parsePagination(response);
          }).catch((error)=>{
            console.log(error)
          });
        },
        parsePagination(response){
          this.rows = response.data.total;
          this.perPage = response.data.per_page;
          this.currentPage = response.data.current_page;
          this.first = response.data.first_page_url;
          this.last = response.data.last_page_url;
          this.prev = response.data.prev_page_url;
          this.next = response.data.next_page_url;
          this.items=response.data.data;
        },
        reset(){
          this.page_url='/user/all';
          this.number_page='/user/all?page=';
          this.selected=null;
          this.text="";
          this.listUsers();
        }
    },
    created(){
      this.listUsers();
    }
  }
</script>