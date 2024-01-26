<script>
  import axios from 'axios';
  import Header  from './components/header.vue';
  import {orderBy} from 'lodash';
  import {ref } from 'vue';

  const emisor = ref(null);
  //llamar por defecto dandole nombres
  export default {
    components: {
      //traer el header
      Header
    },
    data(){
      return{
        //arreglo donde se guarda la informacion
        datos:[]
      };
    },
    //metodos para obtener la informacion
    methods:{
      async obtenerData(){
        try{

          const fecha = new Date();

          // Obtener el año, mes y día
          const year = fecha.getFullYear();
          const month = (fecha.getMonth() + 1).toString().padStart(2, '0'); // Agregar un cero si es necesario
          const day = fecha.getDate().toString().padStart(2, '0'); // Agregar un cero si es necesario

          // Crear la cadena en el formato yyyymmdd
          const fechaHoy = `${year}${month}${day}`;
          console.log(fechaHoy)
          const response = await axios.get(`http://172.19.10.8:8000/obtener_datos?parametro1=${fechaHoy}&parametro2=${fechaHoy}`);
          // Asignar los datos a la lista
          this.datos = response.data;

          //imprime en consola para hacer la prueba
          console.log('datos asignados:', this.datos); // Muestra los datos en la consola para verificar
        } catch (error) {
          console.error(error);
        }
      },
      // Método para filtrar los datos por tipo (RO o AL)
      getFilteredDatUp: function(tipo) {
        //filtrar ordenar rosa o astromelia y orenar de manera descendente
        return orderBy((this.datos.filter(item => item.CODPRO === tipo)),'REND_HR','desc').slice(0,5);
      },
      getFilteredDatDown: function(tipo) {
        //filtrar ordenar rosa o astromelia y orenar de manera descendente
        const valores = orderBy((this.datos.filter(item => item.CODPRO === tipo)),'REND_HR','desc');
        //cantidad de registros
        const cantidaValores = valores.length
        //total resta para obtener -5
        const recorrederValores = cantidaValores + 5
        //valores a reccorer negativos
        const negativos = cantidaValores - recorrederValores 
        return valores.slice(negativos)
      }
    },
    computed:{
      firstFiveAL: function() {
      // Filtrar los primeros cinco registros de tipo 'RO'
      return this.getFilteredDatUp('AL')
      },

      firstFiveRO: function() {
        return this.getFilteredDatUp('RO')
        
      },

      lastFiveAL: function() {
        // Filtrar los primeros cinco registros de tipo 'RO'
        return this.getFilteredDatDown('AL')
      },
      lastFiveRO: function() {
        return  this.getFilteredDatDown('RO')
      }
    },mounted(){
      this.obtenerData(); // Llamada inicial
      // Ejecutar la función cada 5 minutos
      this.intervalID = setInterval(this.obtenerData, 60000);
    }
  }
  
</script>
<template>
  <Header/>
  <div id="app">
    <div class="app">
      <div class="section">
        <h1>ALSTROEMERIA</h1>
        <div class="table-container" v-if="datos.length">
          <div class="table-row header">
            <div class="table-cell">Nombre</div>
            <div class="table-cell">Tallos</div>
            <div class="table-cell">Rendimiento</div>
          </div>
          <!--Cargar los primeros 5 nombres -->
          <div v-for="(fila, index) in firstFiveAL" :key="index">
            <div v-if="fila.CODPRO === 'AL'" class="table-row">
              <div class="table-cell">{{ fila.NOM2 }}</div>
              <div class="table-cell">{{ fila.TALLOS }}</div>
              <div class="table-cell">{{ fila.REND_HR !== null && fila.REND_HR !== undefined ? fila.REND_HR.toFixed(0) : ''}} 🔼</div>
            </div>
          </div>
          <div v-for="(fila, index) in lastFiveAL" :key="index">
            <div v-if="fila.CODPRO === 'AL'" class="table-row">
              <div class="table-cell">{{ fila.NOM2 }}</div>
              <div class="table-cell">{{ fila.TALLOS }}</div>
              <div class="table-cell">{{ fila.REND_HR !== null && fila.REND_HR !== undefined ? fila.REND_HR.toFixed(0) : ''}} 🔻</div>
            </div>
          </div>
        </div>
        <p v-else>No hay datos disponibles</p>
      </div>
      <div class="section">
        <h1>ROSA</h1>
        <div class="table-container" v-if="datos.length">
          <div class="table-row header">
            <div class="table-cell"> Nombre</div>
            <div class="table-cell">Tallos</div>
            <div class="table-cell">Rendimiento</div>
          </div>
          <div v-for="(fila, index) in firstFiveRO" :key="index">
            <div v-if="fila.CODPRO === 'RO'" class="table-row">
              <div class="table-cell">{{ fila.NOM2 }}</div>
              <div class="table-cell">{{ fila.TALLOS }}</div>
              <div class="table-cell">{{ fila.REND_HR !== null && fila.REND_HR !== undefined ? fila.REND_HR.toFixed(0) : ''}}🔼</div>
            </div>
          </div>
          <div v-for="(fila, index) in lastFiveRO" :key="index">
            <div v-if="fila.CODPRO === 'RO'" class="table-row">
              <div class="table-cell">{{ fila.NOM2 }}</div>
              <div class="table-cell">{{ fila.TALLOS }} </div>
              <div class="table-cell">{{ fila.REND_HR !== null && fila.REND_HR !== undefined ? fila.REND_HR.toFixed(0) : ''}}🔻</div>
            </div>
          </div>
        </div>
        <p v-else>No hay datos disponibles</p>
      </div>
    </div>
  </div>
</template>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
}

.app {
  display: flex;
  justify-content: space-around;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

.section {
  flex-basis: calc(45% - 20px);
  margin: 10px;
  border: 1px solid #ddd;
  padding: 10px;
}

.table-container {
  margin-top: 10px;
}

.table-row {
  display: flex;
}


.table-cell {
  flex: 1;
  padding: 8px;
  border: 1px so;
  text-align: left;
}

ul {
  list-style-type: none;
  padding: 0;
}

.table-row.header {
  font-weight: bold;
}

</style>