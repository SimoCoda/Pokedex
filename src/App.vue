<script setup lang="ts">
import listPokemon from './components/listPokemon.vue'
</script>

<template>
  <div class="container">
    <h1
       class="text-5xl lg:text-6xl xl:text-7xl text-left pl-8 md:text-center md:pl-0 lg:text-center lg:pl-0 xl:text-center xl:pl-0 text-white font-bold pt-16"
    >
      Pokedex
    </h1>
    <listPokemon />
  </div>
</template>
