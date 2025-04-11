<template>
  <div>
    <!-- Filtros -->
    <div class="filtros">
      <input v-model="filtros.nome" placeholder="Filtrar por nome" />
      <input v-model="filtros.id" placeholder="Filtrar por ID" />

      <!-- Select para tipo -->
      <select v-model="filtros.tipo">
        <option value="">Todos os tipos</option>
        <option v-for="tipo in tiposPokemon" :key="tipo" :value="tipo">
          {{ tipo }}
        </option>
      </select>

      <!-- Select para espécie -->
      <select v-model="filtros.especie">
        <option value="">Todas as espécies</option>
        <option
          v-for="especie in especiesDisponiveis"
          :key="especie"
          :value="especie"
        >
          {{ especie }}
        </option>
      </select>
    </div>

    <!-- Lista de Pokémon -->
    <div class="lista-pokemon">
      <div
        v-for="pokemon in listaPokemon"
        :key="pokemon.id"
        class="item-pokemon"
        v-show="deveExibirPokemon(pokemon)"
        @click="abrirModal(pokemon.name)"
      >
        <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
        <p>{{ pokemon.name }}</p>
      </div>
      <div v-if="carregando" class="carregando">Carregando mais Pokémon...</div>
    </div>

    <!-- Modal -->
    <PokemonModal
      v-if="pokemonSelecionado"
      :pokemon="pokemonSelecionado"
      @close="pokemonSelecionado = null"
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
      listaPokemon: [],
      urlProxima: "https://pokeapi.co/api/v2/pokemon?limit=20&offset=0",
      carregando: false,
      pokemonSelecionado: null,
      filtros: {
        nome: "",
        id: "",
        tipo: "",
        especie: "",
      },
      tiposPokemon: [
        "normal",
        "fire",
        "water",
        "grass",
        "electric",
        "ice",
        "fighting",
        "poison",
        "ground",
        "flying",
        "psychic",
        "bug",
        "rock",
        "ghost",
        "dragon",
        "dark",
        "steel",
        "fairy",
      ],
      especiesDisponiveis: [],
    };
  },
  mounted() {
    this.carregarPokemons();
    window.addEventListener("scroll", this.verificarScroll);
    this.$nextTick(this.verificarAlturaPagina);
  },
  beforeUnmount() {
    window.removeEventListener("scroll", this.verificarScroll);
  },
  methods: {
    async verificarScroll() {
      const posicaoScroll = window.innerHeight + window.pageYOffset;
      const alturaPagina = document.documentElement.scrollHeight;

      if (
        posicaoScroll >= alturaPagina - 200 &&
        !this.carregando &&
        this.urlProxima
      ) {
        await this.carregarPokemons();
      }
    },

    async carregarPokemons() {
      if (this.carregando || !this.urlProxima) return;
      this.carregando = true;

      try {
        const resposta = await axios.get(this.urlProxima);
        this.urlProxima = resposta.data.next;

        const promessas = resposta.data.results.map(async (p) => {
          const detalhe = await axios.get(p.url);
          const especie = await axios.get(detalhe.data.species.url);

          // Evoluções
          const cadeiaUrl = especie.data.evolution_chain?.url;
          let evolucoes = [];
          if (cadeiaUrl) {
            const cadeia = await axios.get(cadeiaUrl);
            evolucoes = this.extrairEvolucoes(cadeia.data.chain);
          }

          // Atualizar espécies únicas para o select
          if (!this.especiesDisponiveis.includes(especie.data.name)) {
            this.especiesDisponiveis.push(especie.data.name);
          }

          return {
            ...detalhe.data,
            especie: especie.data,
            evolucoes,
          };
        });

        const detalhados = await Promise.all(promessas);
        this.listaPokemon.push(...detalhados);

        this.$nextTick(this.verificarAlturaPagina);
      } catch (erro) {
        console.error("Erro ao carregar Pokémon:", erro);
      } finally {
        this.carregando = false;
      }
    },

    verificarAlturaPagina() {
      const altura = document.documentElement.scrollHeight;
      const alturaJanela = window.innerHeight;

      if (altura <= alturaJanela + 100 && this.urlProxima && !this.carregando) {
        this.carregarPokemons();
      }
    },

    deveExibirPokemon(pokemon) {
      const { nome, id, tipo, especie } = this.filtros;

      const correspondeNome =
        !nome || pokemon.name.toLowerCase().includes(nome.toLowerCase());
      const correspondeId = !id || pokemon.id.toString() === id;
      const correspondeTipo =
        !tipo ||
        pokemon.types.some(
          (t) => t.type.name.toLowerCase() === tipo.toLowerCase()
        );
      const correspondeEspecie =
        !especie ||
        pokemon.especie.name.toLowerCase() === especie.toLowerCase();

      return (
        correspondeNome &&
        correspondeId &&
        correspondeTipo &&
        correspondeEspecie
      );
    },

    abrirModal(nome) {
      const encontrado = this.listaPokemon.find((p) => p.name === nome);
      if (encontrado) {
        this.pokemonSelecionado = encontrado;
      } else {
        console.warn("Pokémon não encontrado:", nome);
      }
    },

    extrairEvolucoes(cadeia) {
      const resultado = [];

      function percorrer(c) {
        resultado.push(c.species.name);
        if (c.evolves_to && c.evolves_to.length > 0) {
          percorrer(c.evolves_to[0]);
        }
      }

      percorrer(cadeia);
      return resultado;
    },
  },
};
</script>

<style scoped>
.lista-pokemon {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  padding: 1rem;
  justify-content: center;
}

.item-pokemon {
  border: 1px solid #ccc;
  padding: 0.5rem;
  text-align: center;
  width: 120px;
  background-color: #f9f9f9;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s;
}

.item-pokemon:hover {
  transform: scale(1.05);
}

.item-pokemon img {
  width: 100px;
}

.carregando {
  text-align: center;
  width: 100%;
  padding: 2rem;
  font-weight: bold;
  color: #555;
}

.filtros {
  display: flex;
  gap: 1rem;
  margin: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

.filtros input,
.filtros select {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 8px;
}
</style>
