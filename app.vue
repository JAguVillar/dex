<script setup lang="ts">
import { ref, onMounted, watch, computed } from "vue";
import { createClient, SupabaseClient } from "@supabase/supabase-js";
import pokeballs from "./lib/data/pokeballs";
import pokemons from "./lib/data/pokemon";

interface Pokemon {
  id: number;
  number: number;
  cardName: string;
  name: string;
  generation: number;
  sprite: string;
  sprite_shiny: string;
  artwork: string;
  artwork_shiny: string;
  catched: boolean;
  status: { id: number; label: string };
  tipo: { id: number; name: string };
}

interface Estado {
  id: number;
  label: string;
}

interface Column {
  key: string;
  label: string;
  class?: string;
}

// Constants
const SUPABASE_URL = "https://qkmelseamixaebntalzk.supabase.co";
const SUPABASE_ANON_KEY =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InFrbWVsc2VhbWl4YWVibnRhbHprIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MzQ5MjA2MjQsImV4cCI6MjA1MDQ5NjYyNH0.PPy_DP4dxPOwsyRfAiscJe1p1LpsAfThSIHWLU-WWNA";

const POKEBALLS = pokeballs;

const GENERATIONS = [
  { label: "Generation I", value: 1, region: 'Kanto' },
  { label: "Generation II", value: 2, region: 'Johto' },
  { label: "Generation III", value: 3, region: 'Hoenn' },
  { label: "Generation IV", value: 4, region: 'Sinnoh' },
  { label: "Generation V", value: 5, region: 'Unova' },
  { label: "Generation VI", value: 6, region: 'Kalos' },
  { label: "Generation VII", value: 7, region: 'Alola' },
  { label: "Generation VIII", value: 8, region: 'Galar' },
  { label: "Generation IX", value: 9, region: 'Paldea' },
];

const STATUSES = [
  { label: "Buscada", value: 1 },
  { label: "Claim", value: 2 },
  { label: "Conseguida", value: 3 },
];

const COLUMNS: Column[] = [
  { key: "catched", label: "", class: "w-fit" },
  { key: "number", label: "#", class: "w-fit" },
  { key: "name", label: "Pokémon" },
  { key: "gen", label: "Generation" },
  { key: "status", label: "Estado" },
  { key: "actions", label: "Acciones" },
];

// State Management
const state = {
  currentGeneration: ref(GENERATIONS[0]),
  currentPokemon: ref<Pokemon | null>(null),
  modalEstado: ref(false),
  modalFiltro: ref(false),
  selectedStatus: ref(STATUSES[0]),
  estados: ref<Estado[]>([]),
  pokemons: ref<Pokemon[]>([]),
  loadingTable: ref(false),
  updatingStatus: ref(false),
  search: ref(""),
};

// Supabase Client
const supabase: SupabaseClient = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// Fetch Functions
async function fetchPokemons() {
  let query = supabase.from("pokedex").select("*, status (*), tipo (*)")

  if (!state.search.value) {
    query = query.eq("generation", state.currentGeneration.value.value)
  }

  state.loadingTable.value = true;
  try {
    const { data, error } = await query
      .like("name", `%${state.search.value}%`)
      .order("number");
    if (error) throw error;
    state.pokemons.value = data || [];
  } catch (error) {
    console.error("Error fetching pokemons:", error);
  }
  state.loadingTable.value = false;
}

async function fetchPokemon(id: number) {
  state.loadingTable.value = true;
  try {
    const { data, error } = await supabase
      .from("pokedex")
      .select("*, status (label)")
      .eq("id", id);
    if (error) throw error;
    // state.pokemons.value = data || [];
  } catch (error) {
    console.error("Error fetching pokemons:", error);
  }
  state.loadingTable.value = false;
}

async function fetchEstados() {
  try {
    const { data, error } = await supabase.from("estados").select("*");
    if (error) throw error;
    state.estados.value = data || [];
    console.log("Estados:", state.estados.value);
  } catch (error) {
    console.error("Error fetching estados:", error);
  }
}

// Utility Functions
function createMenuItems(pokemonName: string) {
  return [
    [
      {
        label: "Troll&Toad",
        click: () =>
          window.open(
            `https://www.trollandtoad.com/category.php?search-words=${pokemonName}`,
            "_blank"
          ),
        colors: ["blue"],
      },
      {
        label: "TCGCollector",
        click: () =>
          window.open(
            `https://www.tcgcollector.com/cards/intl?cardSearch=${pokemonName}`,
            "_blank"
          ),
        colors: ["orange"],
      },
      {
        label: "PriceCharting",
        click: () =>
          window.open(
            `https://www.pricecharting.com/search-products?q=${pokemonName}`,
            "_blank"
          ),
        colors: ["green"],
      },
      {
        label: "TCGPlayer",
        click: () =>
          window.open(
            `https://www.tcgplayer.com/search/all/product?q=${pokemonName}`,
            "_blank"
          ),
        colors: ["red"],
      },
    ],
  ];
}

// Debounce utility
function debounce(func: Function, wait: number) {
  let timeout: NodeJS.Timeout;
  return (...args: any[]) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), wait);
  };
}


async function updateStatus() {
  const statusId = state.selectedStatus.value.value;
  const pokemonId = state.currentPokemon.value?.id;
  state.updatingStatus.value = true;
  if (statusId === 3) {
    try {
      const { error } = await supabase
        .from("pokedex")
        .update({ status: statusId, catched: true })
        .eq("id", pokemonId);
      if (error) throw error;
    } catch (error) {
      console.error("Error fetching pokemons:", error);
    }
  } else {
    try {
      const { error } = await supabase
        .from("pokedex")
        .update({ status: statusId, catched: false })
        .eq("id", pokemonId);
      if (error) throw error;
    } catch (error) {
      console.error("Error fetching pokemons:", error);
    }
  }

  state.updatingStatus.value = false;
  state.modalEstado.value = false;
  fetchPokemons();
}

const debouncedSearch = debounce(() => {
  fetchPokemons();
}, 300);

async function openModalEstado(row: Pokemon) {
  state.currentPokemon.value = row;
  state.modalEstado.value = true;
  // const endpoint = 'https://api.pokemontcg.io/v2/cards?q=name:charmander&pageSize=10';

  // try {
  //   const response = await fetch(endpoint);
  //   if (!response.ok) {
  //     throw new Error(`Error fetching cards: ${response.statusText}`);
  //   }
  //   const data = await response.json();
  //   console.log('Cards:', data);
  // } catch (err) {

  //   console.error('Error fetching cards:', err);
  // }
}

function aplicarFiltros() {

  fetchPokemons();
  state.modalFiltro.value = false;
}

// watch(state.currentGeneration, (value) => {
//   if (value) {
//     console.log("Selected:", null);
//     state.currentGeneration.value = value;
//     fetchPokemons();
//   }
// });

watch(state.search, (value) => {
  debouncedSearch(value)
});


// Lifecycle
onMounted(fetchPokemons);
</script>

<template>
  <UContainer class="py-8">
    <div class="flex gap-4 justify-between mb-4">
      <UInput v-model="state.search.value" placeholder="Buscar Pokemon..." class="w-1/2" />
      <UButton @click="state.modalFiltro.value = true" icon="i-heroicons-adjustments-vertical" size="sm" color="primary"
        variant="solid" label="Filtrar" :trailing="false" />
    </div>
    <UTable :rows="state.pokemons.value" :columns="COLUMNS" :loading="state.loadingTable.value">
      <template #catched-data="{ row }">
        <template v-if="row.catched == true">
          <img :src="pokeballs[row.tipo.id - 1]" alt="" srcset="" width="32" />
        </template>
        <template v-else">
          <img :src="pokeballs[pokeballs.length - 1]" alt="" srcset="" width="32" />
        </template>
      </template>
      <template #number-data="{ row }">
        <span>{{ row.number }}</span>
      </template>

      <template #name-data="{ row }">
        <span class="capitalize">{{ row.name }}</span>
      </template>

      <template #gen-data="{ row }">
        <span>{{ row.generation }}</span>
      </template>

      <template #status-data="{ row }">
        <UBadge class="cursor-pointer" :color="row.status.label === 'Buscada'
          ? 'red'
          : row.status.label === 'Claim'
            ? 'purple'
            : 'green'
          " variant="solid" @click="openModalEstado(row)">
          {{ row.status.label }}
        </UBadge>
      </template>

      <template #actions-data="{ row }">
        <UDropdown :items="createMenuItems(row.cardName)" :popper="{ placement: 'bottom-start' }"
          @click="() => (state.currentPokemon = row.cardName)">
          <template #item="{ item }">
            <span v-for="color in item.colors" :key="color.id" class="h-2 w-2 rounded-full"
              :class="`bg-${color}-500 dark:bg-${color}-400`"></span>
            <span>{{ item.label }}</span>
          </template>
          <UButton color="white" label="Ver en" trailing-icon="i-heroicons-chevron-down-20-solid" />
        </UDropdown>
      </template>
    </UTable>

    <UModal v-model="state.modalEstado.value">
      <UCard>
        <template #header>Cambiar estado</template>
        <USelectMenu v-model="state.selectedStatus.value" :options="STATUSES" />
        <template #footer>
          <div class="flex justify-end">
            <UButton @click="updateStatus" :loading="state.updatingStatus.value">Actualizar estado</UButton>
          </div>
        </template>
      </UCard>
    </UModal>
    <UModal v-model="state.modalFiltro.value">
      <UCard>
        <template #header>Filtros</template>
        <div>
          <h1 class="font-medium text-gray-200 pb-2">Generacion</h1>
          <USelectMenu v-model="state.currentGeneration.value" :options="GENERATIONS" class="pb-8">
            <template #label>
              <span class="capitalize">{{ state.currentGeneration.value.label }} </span>
              <span class="capitalize text-gray-500">{{ state.currentGeneration.value.region }}</span>
            </template>
            <template #option="{ option: GENERATIONS }">
              <span class="capitalize">{{ GENERATIONS.label }} </span>
              <span class="capitalize text-gray-500">{{ GENERATIONS.region }}</span>
            </template>
          </USelectMenu>
        </div>
        <div>
          <h1 class="font-medium text-gray-200 pb-2">Estado</h1>
          <USelectMenu v-model="state.selectedStatus.value" :options="STATUSES" class="pb-8">
            <template #option="{ option: STATUSES }">
              <UBadge class="cursor-pointer" :color="STATUSES.label === 'Buscada'
                ? 'red'
                : STATUSES.label === 'Claim'
                  ? 'purple'
                  : 'green'
                " variant="solid">
                {{ STATUSES.label }}
              </UBadge>
            </template>
          </USelectMenu>
        </div>
        <template #footer>
          <div class=" flex justify-end">
            <UButton @click="aplicarFiltros">Aplicar</UButton>
          </div>
        </template>
      </UCard>
    </UModal>
  </UContainer>
</template>
