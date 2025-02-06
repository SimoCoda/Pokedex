<script setup lang="ts">
import axios from 'axios'
import { ref, watch, onMounted, nextTick } from 'vue'
import GLightbox from 'glightbox'
import 'glightbox/dist/css/glightbox.css'

interface Pokemon {
  name: string
  base_experience: number
  sprites: {
    front_default: string
    other: {
      dream_world: {
        front_default: string
      }
    }
  }
}

const isLoading = ref(false)
const isError = ref(false)
const searchPokemon = ref('')
const fullPokemonList = ref<Pokemon[]>([])
const filteredPokemonList = ref<Pokemon[]>([])
const lightbox = ref<null | ReturnType<typeof GLightbox>>(null)

const fetchData = async () => {
  try {
    isLoading.value = true
    const nameResponse = await axios.get('https://pokeapi.co/api/v2/pokemon-form/?limit=1025')
    const pokemonPromises = nameResponse.data.results.map(
      (pokemon: { name: string }, index: number) => {
        return axios.get(`https://pokeapi.co/api/v2/pokemon/${index + 1}`)
      },
    )

    const pokemonResponses = await Promise.all(pokemonPromises)
    fullPokemonList.value = pokemonResponses.map((response) => ({
      name: response.data.name,
      base_experience: response.data.base_experience,
      sprites: {
        front_default: response.data.sprites.front_default,
        other: {
          dream_world: {
            front_default: response.data.sprites.other.dream_world.front_default,
          },
        },
      },
    }))
    filteredPokemonList.value = [...fullPokemonList.value]

    isLoading.value = false
    // Inizializza GLightbox dopo che il DOM è stato aggiornato
    nextTick(() => {
      if (lightbox.value) {
        lightbox.value.destroy()
      }
      lightbox.value = GLightbox({
        touchNavigation: true,
        loop: true,
        autoplayVideos: true,
      })
    })
  } catch (error) {
    isError.value = true
    console.error(error)
  }
}

watch(searchPokemon, (nameOfPokemon) => {
  if (nameOfPokemon.trim() === '') {
    filteredPokemonList.value = [...fullPokemonList.value]
  } else {
    const filteredPokemons = fullPokemonList.value.filter((pokemon) =>
      pokemon.name.toLowerCase().includes(nameOfPokemon.toLowerCase()),
    )
    filteredPokemonList.value = filteredPokemons
  }
  // Inizializza GLightbox dopo che il DOM è stato aggiornato
  nextTick(() => {
    if (lightbox.value) {
      lightbox.value.destroy()
    }
    lightbox.value = GLightbox({
      touchNavigation: true,
      loop: true,
      autoplayVideos: true,
    })
  })
})

onMounted(() => {
  fetchData()
})
</script>

<template>
  <div
    class="min-h-[80vh] bg-white shadow-2xl shadow-black border-2 border-black rounded-4xl my-8 flex flex-col gap-10 py-10 relative"
  >
    <img
      src="../assets/images/ash.png"
      alt="foto ash"
      class="w-28 absolute right-0 lg:right-16 top-[-135px] drop-shadow-2xl shadow-black"
    />
    <div v-if="isError" class="w-full text-center">
      <h1 class="text-3xl font-semibold">Ops! Qualcosa è andato storto!</h1>
      <h3 class="text-2xl font-semibold">Ricarica la pagina</h3>
    </div>
    <div v-if="isLoading" class="loader"></div>
    <div v-else>
      <div class="bg-blue mx-auto flex justify-center w-72 relative items-center">
        <input
          type="text"
          name=""
          id=""
          placeholder="Cerca il tuo pokemon"
          class="border-2 px-3 py-2 w-full relative rounded-2xl"
          v-model="searchPokemon"
        />
        <i class="pi pi-search text-xl absolute right-2"></i>
      </div>
      <ul class="grid grid-cols-2 gap-10 my-10 md:grid-cols-3 lg:grid-cols-4">
        <li
          v-for="(pokemon, idx) in filteredPokemonList"
          :key="idx"
          class="cursor-pointer flex flex-col items-center gap-5"
        >
          <div class="w-28 h-28 lg:w-32 lg:h-32 xl:w-32 xl:h-32">
            <a
              :href="
                pokemon.sprites.other.dream_world.front_default ?? pokemon.sprites.front_default
              "
              class="glightbox"
            >
              <img
                :src="
                  pokemon.sprites.other.dream_world.front_default ?? pokemon.sprites.front_default
                "
                :alt="pokemon.name"
                class="hover:scale-125 transition-all w-full h-full"
                loading="lazy"
              />
            </a>
          </div>
          <div class="flex gap-3 items-center">
            <h5 class="capitalize text-xl font-semibold">{{ pokemon.name }}</h5>
            <span class="font-lg italic">({{ pokemon.base_experience }})</span>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>
