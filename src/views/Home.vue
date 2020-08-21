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
        <b-col col lg="11">
          <b-table :items="items" :fields="fields" striped responsive="sm">
            <template v-slot:cell(status_details)="row">
                <b-button v-if="row.detailsShowing" variant="danger" @click="openmodal(row.item.id,row.item.status,row.item.name)" class="mr-2">Desactivar</b-button>
                <b-button v-if="!row.detailsShowing" variant="primary" @click="openmodal(row.item.id,row.item.status,row.item.name)" class="mr-2">Activar</b-button>
            </template>
          </b-table>
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
      <div>
        <b-modal id="change-status" centered hide-footer>
            <template v-slot:modal-title>
                Cambiar el estado del Usuario
            </template>
            <div class="d-block text-center">
                <h5 v-if="state">¿Desea  Desactivar la cuenta del Usuario {{user}}?</h5>
                <h5 v-if="!state">¿Desea Activar la cuenta del Usuario {{user}}?</h5>
            </div>
            <b-button v-if="state" variant="danger" @click="changestatus()" class="mr-2">Desactivar</b-button>
            <b-button v-if="!state" variant="primary" @click="changestatus()" class="mr-2">Activar</b-button>

        </b-modal>
      </div>
	  </b-container>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        fields: ['id','status','name', 'email','created_at','updated_at','status_details'],
        items: [
          
        ],
        selected: null,
        text: '',
        options: [
          { value: null, text: 'Selecciona una opción' },
          { value: '1', text: 'Activos' },
          { value: '0', text: 'Inactivo' }
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
        id:1,
        state:1,
        user:'Demo'
      }
    },
    methods:{
        filter(){
          axios.get(`/user/search/${this.text}`).then((response)=>{
            this.page_url = `/user/search/${this.text}`;
            this.number_page = `${this.page_url}?page=`;
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
            this.page_url = `/user/status/${state}`;
            this.number_page = `${this.page_url}?page=`;
            this.parsePagination(response);
          }).catch((error)=>{
            console.log(error)
          });
        },
        parsePagination(response){
          for(let i in response.data.data){
            response.data.data[i]['_showDetails'] = response.data.data[i]["status"] ? true : false;
          }
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
        },
        openmodal(id,state,user){
            this.$bvModal.show('change-status');
            this.id =id;
            this.state = state;
            this.user = user;
        },
        changestatus(){
          this.$bvModal.hide('change-status');
          this.state = this.state ? 0 : 1;
          axios.get(`/user/changestate/${this.id}/${this.state}`)
          .then(()=>this.listUsers())
          .catch((error)=>{
              console.log(error);
          })
        }
    },
    created(){
      this.listUsers();
    }
  }
</script>