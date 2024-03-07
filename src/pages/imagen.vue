<script setup>
// imagen 
import { ref, onMounted } from 'vue';
const urlImagen = ref("https://picsum.photos/200/300");
const chiste = ref("");

async function cambiaImagen(){
	let respuesta= await fetch("https://picsum.photos/200/300");
	console.log(respuesta.url);
	urlImagen.value=respuesta.url;
}
// chiste

async function obtenerChisteAleatorio() {
  const respuesta = await fetch('https://api.chucknorris.io/jokes/random');
  const chisteNuevo = await respuesta.json();
  chiste.value = chisteNuevo.value;
}

onMounted(async () => {
  // lo necesito para tener un chiste aleatorio al cargar la página
  await obtenerChisteAleatorio();
});
</script>

<template>
    <v-card
      class="mx-auto"
      max-width="400"
    >
      <v-img
        class="align-end text-white"
        height="200"
        :src="urlImagen"
        cover
      >
        <v-card-title>Imagen de picsum</v-card-title>
      </v-img>
  
      <v-card-subtitle class="pt-4">
        Chiste de Chuck Norris
      </v-card-subtitle>
  
      <v-card-text>  
        <div>{{ chiste }}</div>
      </v-card-text>
  
      <v-card-actions>
        <v-btn color="orange" @click="cambiaImagen">
          Imagen
        </v-btn>

        <v-btn color="orange" @click="obtenerChisteAleatorio">
          Chiste 
        </v-btn>
  
      </v-card-actions>
    </v-card>
  </template>