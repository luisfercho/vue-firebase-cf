<template lang="html">
    <div class="container">
      <div class="screen">
        <p class="text-light font-weight-bold">Pantalla</p>
      </div>
      <div class="chairs">
        <div class="row">
          <div class="col-2 chair"
          v-for="chair in chairs"
          :key="chair.id"
          v-text="chair.id"
          :id="chair.id"
          :class="{isAvailable:chair.available,noAvailable:!chair.available,pointer:chairIsAvailable(chair)}"
          @click="selectChair"
          ></div>
        </div>
      </div>
      <div class="mt-3 text-center">
        <b-button :disabled="counter==0" class="mr-2" variant="success" @click="save">
            Guardar
        </b-button>
        <b-button :disabled="counter==0" class="mr-2" variant="danger" @click="cancel">
              Cacelar
        </b-button>
        <b-button class="ml-2" @click="liberate">
            Liberar
        </b-button>
      </div>
      <div class="m-3">
        <b>Asientos seleccionados {{ counter }}</b>
      </div>
    </div>
</template>

<script>
import firebase from 'firebase';
const PATH = "rooms";
const ID = "1"
export default {
  data(){
    return {
      id:"",
      counter:0,
      chairs:[]
    }
  },
  created(){
    this.getChairs();
  },
  methods:{
    selectChair(e){
      let chair = this.chairs.find(a => a.id == e.target.id);
      if(
          chair.occupied ||
          (chair.user_id != null && chair.user_id != this.id)
      ){
        this.$notify({
          group: 'tallerfc',
          type: 'error',
          title: 'Alerta',
          text: "No se puede solicitar el asiento "+chair.id
        });
        return
      }
      chair.available = !chair.available;
      chair.user_id = !chair.available?this.id:null;
      this.updateElements();
      this.counter = this.chairsSelected().length;
    },
    getChairs(){
      this.id = firebase.database().ref("/users").push().key;

      firebase.database()
        .ref(PATH)
        .child(ID)
        .on('value',(snap)=>{
          this.chairs = snap.val();
        })
    },
    updateElements(){
      firebase.database()
        .ref(PATH)
        .child(ID)
        .set(this.chairs);
    },
    save(){
      firebase.database()
        .ref(PATH)
        .child(ID)
        .transaction(
          values => this.checkTransaction(values),
          (err,commited,snap) => this.responseValidate(err,commited,snap)
        )
      //this.checkChairs();
      //this.updateElements();
      this.counter = 0;
    },
    responseValidate(err,commited,snap){
      if(err){
        this.$notify({
          group: 'tallerfc',
          type: 'error',
          title: 'Alerta',
          text: "No ha sido posible actualizar la inforamción"
        });
      }
      if(commited){
        this.$notify({
          group: 'tallerfc',
          type: 'success',
          title: 'Exito',
          text: "Asientos adquiridos correctamente"
        });
      }
      console.log(snap.val());
    },
    checkTransaction(values){
      this.chairsSelected().forEach((chair) => {
        if(values.find(v => v.id == chair.id).occupied){
          return
        }
        chair.occupied = true
      });
      return this.chairs
    },
    chairIsAvailable(chair){
      return !chair.occupied && ( chair.user_id == null || chair.user_id == this.id );
    },
    chairsSelected(){
      return this.chairs.filter(a => !a.available && !a.occupied)
    },
    checkChairs(){
      this.chairsSelected().forEach((chair)=>{
        chair.occupied = true;
      });
    },
    liberate(){
      this.chairs.forEach((chair)=>{
        chair.occupied = false;
        chair.available = true;
        chair.user_id = null;
      });
      this.updateElements();
      this.counter = 0;
    },
    cancel(){
      this.chairsSelected().forEach((chair) => {
          chair.user_id = null;
          chair.available = true;
      });
      this.updateElements();
      this.counter = 0;
    }
  }
}
</script>

<style>
  .screen{
    background-color: #2b6282;
  }
  .chairs{
    margin-top: 60px;
  }
  .chair{
    color:white;
    border:3px solid white;
    font-weight: bold;
  }
  .pointer {
    cursor: pointer;
  }
  .isAvailable{
    background-color: #2d4d86
  }
  .noAvailable{
    background-color: #73264f
  }
  .btn{
    min-width: 85px;
  }
</style>
