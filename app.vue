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
  card_id: string;
  note: string
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
  { key: "catched", label: "", class: "mx-auto" },
  { key: "number", label: "#" },
  { key: "name", label: "Pokémon" },
  { key: "card", label: "Carta" },
  { key: "status", label: "Estado", class: "flex items-center" },
  { key: "actions", label: "Acciones" },
];

const cartas = ref([]);
const currentPage = ref(1);
const totalPages = ref(1);
const pageSize = 12; // Number of items per page
const totalCount = ref(0);
const loading = ref(false);

// State Management
const state = {
  currentGeneration: ref(),
  currentPokemon: ref<Pokemon | null>(null),
  modalEstado: ref(false),
  modalFiltro: ref(false),
  selectedStatus: ref(),
  estados: ref<Estado[]>([]),
  pokemons: ref<Pokemon[]>([]),
  loadingTable: ref(false),
  loadingCards: ref(false),
  updating: ref(false),
  search: ref(""),
  modalCartas: ref(false),
  modalCarta: ref(false),
  modalNota: ref(false),
  selectedCard: ref<any>(null),
  nota: ref("")
};

// Supabase Client
const supabase: SupabaseClient = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// Fetch Functions
const fetchCards = async () => {
  state.loadingCards.value = true;
  try {
    const response = await fetch(`https://api.pokemontcg.io/v2/cards?q=name:${state.currentPokemon.value?.cardName}&pageSize=${pageSize}&page=${currentPage.value}`);
    const data = await response.json();
    cartas.value = data.data;
    totalCount.value = data.totalCount;
    totalPages.value = Math.ceil(data.totalCount / pageSize);
  } catch (error) {
    console.error('Error fetching data:', error);
  } finally {
    state.loadingCards.value = false;
  }
  state.loadingCards.value = false;

};

const fetchCard = async (row: Pokemon) => {
  state.loadingCards.value = true;
  try {
    const response = await fetch(`https://api.pokemontcg.io/v2/cards/${row.card_id}`);
    const data = await response.json();
    state.selectedCard.value = data.data;
    console.log(state.selectedCard.value);

  } catch (error) {
    console.error('Error fetching data:', error);
  } finally {
    state.loadingCards.value = false;
  }
  state.loadingCards.value = false;

};

const toast = useToast()

async function fetchPokemons() {
  let query = supabase.from("pokedex").select("*, status (*), tipo (*)")
  if (state.currentGeneration.value) {
    query = query.eq("generation", state.currentGeneration.value.value);
  } else {
    if (state.search.value) {
    } else {
      query = query.eq("generation", 1);
    }
  }
  if (state.selectedStatus.value) {
    query = query.eq("status", state.selectedStatus.value.value);
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

function createMoreActionsItemsNoCard(pokemonName: string) {
  return [
    [
      {
        label: "Agregar carta",
        click: () => {
          state.selectedCard.value = null
          state.modalCartas.value = true;
          fetchCards()
        },

        icon: 'i-heroicons-plus'
      },
      {
        label: "Agregar nota",
        click: () => {
          state.modalNota.value = true
        },
        icon: 'i-heroicons-document-text'
      },
    ],
  ];
}

function createMoreActionsItemsCard(pokemonName: string) {
  return [
    [
      {
        label: "Cambiar carta",
        click: () => {
          state.selectedCard.value = null
          state.modalCartas.value = true;
          fetchCards()
        },

        icon: 'i-material-symbols:playing-cards'
      },
      {
        label: "Agregar nota",
        click: () => {
          state.modalNota.value = true
        },
        icon: 'i-heroicons-document-text'
      },
    ],
  ];
}

function createShareActionsItems() {
  return [
    [
      {
        label: "Enviar a Tincho",
        click: async () => { sendWapp('543854866131') },
        icon: 'i-heroicons-paper-airplane-solid'
      },
      {
        label: "Enviar a Agoto",
        click: async () => { sendWapp('543855048767') },
        icon: 'i-heroicons-paper-airplane-solid'
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

async function sendWapp(tel: string) {
  let lista = "Nos falta:%0A"
  try {
    const { data, error } = await supabase
      .from("pokedex")
      .select("*")
      .eq("catched", false)
      .order("number");
    if (error) throw error;
    const response = data || [];
    const generations: { [key: number]: string[] } = {};
    response.forEach(pokemon => {
      const { generation, name } = pokemon;
      if (!generations[generation]) {
        generations[generation] = [];
      }
      generations[generation].push(name + '%0A');
    });

    // Build the list
    for (let gen = 1; gen <= 9; gen++) {
      if (generations[gen]) {
        lista += `%0A*Generacion ${gen}*%0A`; // Add generation header
        lista += generations[gen].join("") + "%0A"; // Add Pokémon names
      }
    }

    window.open(`https://wa.me/${tel}?text=${lista}%0A*Enviado desde la app*`)
  } catch (error) {
    console.error("Error fetching pokemons:", error);
  }
  state.loadingTable.value = false;
}

function copyToClipboard() {
  navigator.clipboard.writeText(`${state.selectedCard.value.name} ${state.selectedCard.value.number}/${state.selectedCard.value.set.printedTotal}`);
  toast.add({
    title: 'Se copio al portapapeles!', description: `${state.selectedCard.value.name} ${state.selectedCard.value.number}/${state.selectedCard.value.set.printedTotal}`,
  })
}


async function updateStatus() {
  const statusId = state.selectedStatus.value.value;
  const pokemonId = state.currentPokemon.value?.id;
  state.updating.value = true;
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

  state.updating.value = false;
  state.modalEstado.value = false;
  fetchPokemons();
}

async function updateCard() {
  const cardId = state.selectedCard.value.id;
  const pokemonId = state.currentPokemon.value?.id;
  state.updating.value = true;

  try {
    const { error } = await supabase
      .from("pokedex")
      .update({ card_id: cardId })
      .eq("id", pokemonId);
    if (error) throw error;
  } catch (error) {
    console.error("Error fetching pokemons:", error);
  }
  state.updating.value = false;
  state.modalCartas.value = false;
  fetchPokemons();
}

async function updateNota() {
  const pokemonId = state.currentPokemon.value?.id;
  state.updating.value = true;

  try {
    const { error } = await supabase
      .from("pokedex")
      .update({ note: state.nota.value })
      .eq("id", pokemonId);
    if (error) throw error;
  } catch (error) {
    console.error("Error fetching pokemons:", error);
  }
  state.updating.value = false;
  state.modalNota.value = false;
  fetchPokemons();
}

const debouncedSearch = debounce(() => {
  fetchPokemons();
}, 300);

async function openModalEstado(row: Pokemon) {
  state.currentPokemon.value = row;
  state.modalEstado.value = true;
}

async function openModalCarta(row: Pokemon) {
  fetchCard(row)
  state.modalCarta.value = true;
}

function aplicarFiltros() {
  fetchPokemons();
  state.modalFiltro.value = false;
}

function getCurrentPokemon(row: Pokemon) {
  cartas.value = [];
  currentPage.value = 1
  totalPages.value = 1
  totalCount.value = 0
  state.currentPokemon.value = row;
  state.nota.value = row.note
}

watch(state.search, (value) => {
  debouncedSearch(value)
});

watch(currentPage, (newPage) => {
  fetchCards();
});

// Lifecycle
onMounted(fetchPokemons);
</script>


<template>
  <UContainer class="py-8">
    <div class="flex gap-4 justify-between mb-4">
      <div class="flex gap-4 w-1/2">
        <UInput v-model="state.search.value" placeholder="Buscar Pokemon..." />
        <UButton @click="state.modalFiltro.value = true" icon="i-heroicons-adjustments-vertical" size="sm"
          color="primary" variant="solid" label="Filtrar" :trailing="false" />
        <UButton v-if="state.currentGeneration.value || state.selectedStatus.value"
          @click="state.currentGeneration.value = null; state.selectedStatus.value = null; fetchPokemons()"
          icon="i-heroicons-x-circle-solid" size="sm" color="red" variant="soft" label="Limpiar filtros"
          :trailing="false" />
      </div>
      <UDropdown :items="createShareActionsItems()" :popper="{ placement: 'bottom-start' }">
        <template #item="{ item }">
          <span>{{ item.label }}</span>
          <UIcon :name="item.icon" class="flex-shrink-0 size-4 text-gray-400 dark:text-gray-500 ms-auto" />
        </template>
        <UButton color="white" trailing-icon="i-heroicons-share-20-solid" label="Compartir" />
      </UDropdown>

    </div>
    <UTable :rows="state.pokemons.value" :columns="COLUMNS" :loading="state.loadingTable.value">
      <template #catched-data="{ row }">

        <UIcon name="i-heroicons-check-badge-solid" class="text-green-400 ms-auto size-6 mx-auto" />

        <!-- <template v-if="row.catched == true">
          <img :src="`/images/Pokeball_${row.tipo.id}.png`" alt="" srcset="" width="32" />
        </template>
  <template v-else">
          <img :src="pokeballs[pokeballs.length - 1]" alt="" srcset="" width="32" />
        </template> -->
      </template>
      <template #number-data="{ row }">
        <span>{{ row.number }}</span>
      </template>

      <template #name-data="{ row }">
        <span class="capitalize">{{ row.name }}</span>
      </template>

      <template #card-data="{ row }">
        <template v-if="row.card_id">
          <UTooltip text="Ver carta">
            <UBadge class="bg-slate-200 cursor-pointer" icon="i-material-symbols:playing-cards" size="md" color="blue"
              variant="solid" @click="openModalCarta(row)" />
          </UTooltip>
        </template>
        <template v-else>
          <UTooltip text="No hay carta agregada">
            <UBadge class="bg-slate-200 cursor-not-allowed" icon="i-material-symbols:playing-cards" size="md"
              color="gray" variant="solid" @click="openModalCarta(row)" />
          </UTooltip>
        </template>
      </template>

      <template #status-data="{ row }">
        <UBadge size="md" class="cursor-pointer" :color="row.status.label === 'Buscada'
          ? 'red'
          : row.status.label === 'Claim'
            ? 'purple'
            : 'green'
          " variant="solid" @click="openModalEstado(row)">
          {{ row.status.label }}
        </UBadge>
      </template>

      <template #actions-data="{ row }">
        <div class="flex items-center gap-2">
          <UDropdown :items="createMenuItems(row.cardName)" :popper="{ placement: 'bottom-start' }">
            <template #item="{ item }">
              <span v-for="color in item.colors" :key="color.id" class="h-2 w-2 rounded-full"
                :class="`bg-${color}-500 dark:bg-${color}-400`"></span>
              <span>{{ item.label }}</span>
            </template>
            <UButton color="white" label="Ver en" trailing-icon="i-heroicons-chevron-down-20-solid" />
          </UDropdown>

          <UDropdown
            :items="!row.card_id ? createMoreActionsItemsNoCard(row.cardName) : createMoreActionsItemsCard(row.cardName)"
            :popper="{ placement: 'bottom-start' }">
            <template #item="{ item }">
              <span>{{ item.label }}</span>
              <UIcon :name="item.icon" class="flex-shrink-0 size-4 text-gray-400 dark:text-gray-500 ms-auto" />
            </template>
            <UButton color="white" trailing-icon="i-heroicons:ellipsis-vertical-20-solid"
              @click="getCurrentPokemon(row)" />
          </UDropdown>
        </div>
      </template>
    </UTable>

    <UModal v-model="state.modalCartas.value">
      <UCard>
        <template #header>
          <div class="flex items-center gap-2">
            <template v-if="state.selectedCard.value">
              <UButton icon="i-heroicons-arrow-left" size="sm" color="primary" square variant="ghost" class=""
                @click="state.selectedCard.value = null" />
            </template>
            <span>
              Agregar carta
            </span>
          </div>
        </template>
        <template v-if="state.selectedCard.value == null">
          <UProgress animation="carousel" v-if="state.loadingCards.value === true" />
          <template v-else>
            <div class="flex flex-wrap gap-4 justify-center">
              <template v-for="carta in cartas">
                <div
                  class="relative inline-block cursor-pointer bg-black opacity-75 hover:opacity-100 transition-opacity"
                  @click="state.selectedCard.value = carta">
                  <img :src="(carta as any).images.small" alt="" srcset="" width="100">
                  <div class="absolute bottom-0.5 right-1">
                    <img :src="(carta as any).set.images.symbol" alt="" srcset="" width="25">
                  </div>
                </div>
              </template>
            </div>
            <div class="flex justify-center">
              <UPagination v-model="currentPage" :page-count="12" :total="totalCount" class="mt-4 mx-auto" />
            </div>
          </template>
        </template>

        <template v-else>
          <div class="flex flex-col gap-4">
            <img :src="state.selectedCard.value.images.large" alt="" srcset="" width="300" class="mx-auto">
            <UDivider />
            <div class="font-bold flex flex-col justify-between gap-2 ml-4">
              <div class="flex gap-4 flex-col">
                <div class="flex flex-col">
                  <span class="text-gray-500 ">Nombre</span>
                  <span>{{ state.selectedCard.value.name }}</span>
                </div>
                <div class="flex flex-col">
                  <span class="text-gray-500 ">Serie</span>
                  <span>{{ state.selectedCard.value.set.series }}</span>
                </div>
                <div class="flex flex-col">
                  <span class="text-gray-500 ">Set</span>
                  <span>{{ state.selectedCard.value.set.name }}</span>
                </div>
              </div>
              <div class="text-gray-500 flex gap-2 items-center justify-between">
                <div class="flex gap-2">
                  <span class="uppercase"> {{ state.selectedCard.value.set.id }}</span>
                  <span> {{ state.selectedCard.value.number + '/' + state.selectedCard.value.set.printedTotal }}</span>
                </div>
                <UButton icon="i-heroicons-clipboard-document" size="sm" color="primary" square variant="ghost"
                  @click="copyToClipboard()" />
              </div>
            </div>
          </div>

        </template>
        <template #footer>
          <div class="flex justify-end">
            <UButton @click="updateCard" :loading="state.updating.value">Agregar</UButton>
          </div>
        </template>
      </UCard>
    </UModal>
    <UModal v-model="state.modalEstado.value">
      <UCard>
        <template #header>Cambiar estado</template>
        <USelectMenu v-model="state.selectedStatus.value" :options="STATUSES" />
        <template #footer>
          <div class="flex justify-end">
            <UButton @click="updateStatus" :loading="state.updating.value">Actualizar estado</UButton>
          </div>
        </template>
      </UCard>
    </UModal>
    <UModal v-model="state.modalNota.value">
      <UCard>
        <template #header>Cambiar estado</template>
        <UTextarea v-model="state.nota.value" />
        <template #footer>
          <div class="flex justify-end">
            <UButton @click="updateNota" :loading="state.updating.value">Actualizar nota</UButton>
          </div>
        </template>
      </UCard>
    </UModal>
    <UModal v-model="state.modalCarta.value">
      <UCard>
        <template #header>Carta en coleccion</template>
        <UProgress animation="carousel" v-if="state.loadingCards.value === true" />
        <template v-else>
          <div class="flex flex-col gap-4">
            <img :src="state.selectedCard.value.images.large" alt="" srcset="" width="300" class="mx-auto">
            <UDivider />
            <div class="font-bold flex flex-col justify-between gap-2 ml-4">
              <div class="flex gap-4 flex-col">
                <div class="flex flex-col">
                  <span class="text-gray-500 ">Nombre</span>
                  <span>{{ state.selectedCard.value.name }}</span>
                </div>
                <div class="flex flex-col">
                  <span class="text-gray-500 ">Serie</span>
                  <span>{{ state.selectedCard.value.set.series }}</span>
                </div>
                <div class="flex flex-col">
                  <span class="text-gray-500 ">Set</span>
                  <span>{{ state.selectedCard.value.set.name }}</span>
                </div>
              </div>
              <div class="text-gray-500 flex gap-2 items-center justify-between">
                <div class="flex gap-2">
                  <span class="uppercase"> {{ state.selectedCard.value.set.id }}</span>
                  <span> {{ state.selectedCard.value.number + '/' + state.selectedCard.value.set.printedTotal }}</span>
                </div>
                <UButton icon="i-heroicons-clipboard-document" size="sm" color="primary" square variant="ghost"
                  @click="copyToClipboard()" />
              </div>
            </div>
          </div>
        </template>
      </UCard>
    </UModal>
    <UModal v-model="state.modalFiltro.value">
      <UCard>
        <template #header>Filtros</template>
        <div>
          <div class="flex justify-between mb-2 items-center">
            <h1 class="font-medium text-gray-200">Generacion</h1>
            <UButton icon="i-heroicons-x-circle-solid" size="2xs" color="red" variant="outline" label="Limpiar filtro"
              :trailing="false" @click="state.currentGeneration.value = []" />
          </div>
          <USelectMenu v-model="state.currentGeneration.value" :options="GENERATIONS" class="pb-8"
            placeholder="Ekija una generacion">
            <template #option="{ option: GENERATIONS }">
              <span class="capitalize">{{ GENERATIONS.label }} </span>
              <span class="capitalize text-gray-500">{{ GENERATIONS.region }}</span>
            </template>
          </USelectMenu>
        </div>
        <div>
          <div class="flex justify-between mb-2 items-center">
            <h1 class="font-medium text-gray-200">Estado</h1>
            <UButton icon="i-heroicons-x-circle-solid" size="2xs" color="red" variant="outline" label="Limpiar filtro"
              :trailing="false" @click="state.selectedStatus.value = []" />
          </div>
          <USelectMenu v-model="state.selectedStatus.value" :options="STATUSES" class="pb-8"
            placeholder="Elija un estado">
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
    <UNotifications />

  </UContainer>
</template>
