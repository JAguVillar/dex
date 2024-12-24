<script setup lang="ts">
import { ref, onMounted } from "vue";
import { createClient, SupabaseClient } from "@supabase/supabase-js";
// import pokemons from "./lib/data/pokemon.js";

interface Pokemon {
  number: number;
  cardName: string;
  name: string;
  generation: number;
  sprite: string;
  sprite_shiny: string;
  artwork: string;
  artwork_shiny: string;
  catched: boolean;
}

interface Column {
  key: string;
  label: string;
  class?: string;
}

const supabaseUrl = "https://qkmelseamixaebntalzk.supabase.co";
const supabaseAnonKey =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InFrbWVsc2VhbWl4YWVibnRhbHprIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MzQ5MjA2MjQsImV4cCI6MjA1MDQ5NjYyNH0.PPy_DP4dxPOwsyRfAiscJe1p1LpsAfThSIHWLU-WWNA";
// const supabaseUrl = import.meta.env.SUPABASE_URL as string;
// const supabaseAnonKey = import.meta.env.SUPABASE_KEY as string;

const supabase: SupabaseClient = createClient(supabaseUrl, supabaseAnonKey);
const pokemons = ref<Pokemon[]>([]);

async function getPokemons(): Promise<void> {
  try {
    const { data, error } = await supabase
      .from("pokedex")
      .select("*")
      .order("number");
    if (error) throw error;
    pokemons.value = data;
  } catch (error) {
    console.error("Error fetching pokemons:", error);
  }
}

const columns: Column[] = [
  { key: "number", label: "#", class: "w-24" },
  { key: "sprite", label: "", class: "w-24" },
  { key: "name", label: "Pokémon" },
  { key: "gen", label: "Generation" },
  { key: "actions", label: "Acciones" },
];

onMounted(() => {
  getPokemons();
});
</script>

<template>
  <div class="p-4">
    <div v-for="pokemon in pokemons" :key="pokemon.number" class="gap-8">
      <UCard
        class="my-4 border-solid border-2"
        :class="pokemon.catched ? 'border-green-300' : 'border-red-300'"
      >
        <template #header>
          <div class="flex items-center gap-4">
            <img :src="pokemon.sprite" alt="" srcset="" />
            <span class="capitalize text-2xl font-bold">
              {{ pokemon.name }}
            </span>
          </div>
        </template>
        Body
        <template #footer>
          <div class="flex items-center gap-4">
            <span>Ver en: </span>
            <a
              target="_blank"
              :href="
                'https://www.trollandtoad.com/category.php?selected-cat=0&search-words=' +
                pokemon.cardName +
                '&token=q0WqiCtxwMaWOA9JW8YISR4q4aJPTiB0hOw%2FS6RBIe64YWY6%2BjdRC3cOdLa7uGLNPsCjFQqyV%2F0QFp%2BLtO7%2Bmg%3D%3D'
              "
            >
              <img
                width="100"
                src="https://52f4e29a8321344e30ae-0f55c9129972ac85d6b1f4e703468e6b.ssl.cf2.rackcdn.com/media/MockUp/TNTLogoNoHexUpdated.png"
                alt=""
                srcset=""
              />
            </a>
            <a
              target="_blank"
              :href="
                'https://www.trollandtoad.com/category.php?selected-cat=0&search-words=' +
                pokemon.cardName +
                '&token=q0WqiCtxwMaWOA9JW8YISR4q4aJPTiB0hOw%2FS6RBIe64YWY6%2BjdRC3cOdLa7uGLNPsCjFQqyV%2F0QFp%2BLtO7%2Bmg%3D%3D'
              "
            >
              <img
                width="100"
                src="https://static.tcgcollector.com/build/images/pages/_components/navbar-logo.15508095.svg"
                alt=""
                srcset=""
              />
            </a>
            <a
              target="_blank"
              :href="
                'https://www.pricecharting.com/search-products?type=prices&q=' +
                pokemon.cardName +
                '&go=Go'
              "
            >
              <img
                width="100"
                src="https://www.pricecharting.com/images/logo-pricecharting-new.png"
                alt=""
                srcset=""
              />
            </a>
            <a
              target="_blank"
              :href="
                'https://www.tcgplayer.com/search/all/product?q=' +
                pokemon.cardName +
                '&view=grid'
              "
            >
              <img
                width="100"
                src="https://mp-assets.tcgplayer.com/img/TCGplayer-logo-primary@2x.png"
                alt=""
                srcset=""
              />
            </a>
          </div>
        </template>
      </UCard>
    </div>
  </div>

  <!-- <UTable
    :rows="pokemons"
    :columns="columns"
    :ui="{ td: { base: 'max-w-[0] truncate' } }"
  >
    <template #number-data="{ row }">
      <span>{{ row.number }}</span>
    </template>
    <template #sprite-data="{ row }">
      <img :src="row.sprite" alt="" srcset="" class="width-24 m-auto" />
    </template>
    <template #name-data="{ row }">
      <span class="capitalize">{{ row.name }}</span>
    </template>
    <template #gen-data="{ row }">
      <span>{{ row.generation }}</span>
    </template>
    <template #actions-data="{ row }">
      <div class="flex">

      </div>
    </template>
  </UTable> -->
</template>
