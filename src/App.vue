<template>
  <div>
    <!-- Filtros -->
    <div class="filters">
      <input v-model="filters.name" placeholder="Filtrar por nome" />
      <input v-model="filters.id" placeholder="Filtrar por ID" />
      <input v-model="filters.type" placeholder="Filtrar por tipo" />
      <input v-model="filters.species" placeholder="Filtrar por espécie" />
    </div>

    <!-- Lista de Pokémon -->
    <div class="pokemon-list">
      <div
        v-for="pokemon in pokemonList"
        :key="pokemon.id"
        class="pokemon-item"
        v-show="shouldShowPokemon(pokemon)"
        @click="openModal(pokemon.name)"
      >
        <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
        <p>{{ pokemon.name }}</p>
      </div>
      <div v-if="loading" class="loading">Carregando mais Pokémon...</div>
    </div>

    <!-- Modal -->
    <PokemonModal
      v-if="selectedPokemonName"
      :name="selectedPokemonName"
      @close="selectedPokemonName = null"
    />
  </div>
</template>

<script>
import axios from "axios";
import PokemonModal from "./components/PokemonModal.vue";

export default {
  components: {
    PokemonModal,
  },
  data() {
    return {
      pokemonList: [],
      nextUrl: "https://pokeapi.co/api/v2/pokemon?limit=20&offset=0",
      loading: false,
      selectedPokemonName: null,
      filters: {
        name: "",
        id: "",
        type: "",
        species: "",
      },
    };
  },
  mounted() {
    this.loadPokemons();
    window.addEventListener("scroll", this.onScroll);
    this.$nextTick(this.checkPageHeight);
  },
  beforeUnmount() {
    window.removeEventListener("scroll", this.onScroll);
  },
  methods: {
    async onScroll() {
      const scrollPosition = window.innerHeight + window.pageYOffset;
      const pageHeight = document.documentElement.scrollHeight;

      if (scrollPosition >= pageHeight - 200 && !this.loading && this.nextUrl) {
        await this.loadPokemons();
      }
    },

    async loadPokemons() {
      if (this.loading || !this.nextUrl) return;
      this.loading = true;

      try {
        const res = await axios.get(this.nextUrl);
        this.nextUrl = res.data.next;

        const detailedPromises = res.data.results.map(async (pokemon) => {
          const detail = await axios.get(pokemon.url);
          const species = await axios.get(detail.data.species.url);
          return {
            ...detail.data,
            species: species.data,
          };
        });

        const detailedPokemon = await Promise.all(detailedPromises);
        this.pokemonList.push(...detailedPokemon);

        this.$nextTick(this.checkPageHeight);
      } catch (error) {
        console.error("Erro ao carregar Pokémon:", error);
      } finally {
        this.loading = false;
      }
    },

    checkPageHeight() {
      const contentHeight = document.documentElement.scrollHeight;
      const windowHeight = window.innerHeight;

      if (
        contentHeight <= windowHeight + 100 &&
        this.nextUrl &&
        !this.loading
      ) {
        this.loadPokemons();
      }
    },

    shouldShowPokemon(pokemon) {
      const { name, id, type, species } = this.filters;

      const matchName =
        !name || pokemon.name.toLowerCase().includes(name.toLowerCase());

      const matchId = !id || pokemon.id.toString() === id;

      const matchType =
        !type ||
        pokemon.types.some((t) =>
          t.type.name.toLowerCase().includes(type.toLowerCase())
        );

      const matchSpecies =
        !species ||
        pokemon.species.name.toLowerCase().includes(species.toLowerCase());

      return matchName && matchId && matchType && matchSpecies;
    },

    openModal(name) {
      this.selectedPokemonName = name;
    },
  },
};
</script>

<style scoped>
.pokemon-list {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  padding: 1rem;
  justify-content: center;
}

.pokemon-item {
  border: 1px solid #ccc;
  padding: 0.5rem;
  text-align: center;
  width: 120px;
  background-color: #f9f9f9;
  border-radius: 8px;
  cursor: pointer;
}

.pokemon-item img {
  width: 100px;
}

.loading {
  text-align: center;
  width: 100%;
  padding: 2rem;
  font-weight: bold;
  color: #555;
}

.filters {
  display: flex;
  gap: 1rem;
  margin: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

.filters input {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 8px;
}
</style>
