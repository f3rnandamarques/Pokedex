<template>
    <div class="modal-overlay" @click.self="$emit('close')">
      <div class="modal-content">
        <button class="close-button" @click="$emit('close')">X</button>
  
        <div v-if="pokemon">
          <h2>{{ capitalize(pokemon.name) }} (ID: {{ pokemon.id }})</h2>
  
          <div class="tabs">
            <button @click="activeTab = 'sprites'" :class="{ active: activeTab === 'sprites' }">Sprites</button>
            <button @click="activeTab = 'moves'" :class="{ active: activeTab === 'moves' }">Movimentos</button>
            <button @click="activeTab = 'evolutions'" :class="{ active: activeTab === 'evolutions' }">Evoluções</button>
            <button @click="activeTab = 'games'" :class="{ active: activeTab === 'games' }">Games</button>
          </div>
  
          <div v-if="activeTab === 'sprites'" class="sprites">
            <h3>Sprites</h3>
            <div>
              <img v-for="(img, i) in spriteList" :key="i" :src="img" :alt="pokemon.name" />
            </div>
          </div>
  
          <div v-if="activeTab === 'moves'">
            <h3>Movimentos</h3>
            <ul>
              <li v-for="(move, index) in pokemon.moves" :key="index">
                {{ capitalize(move.move.name) }}
              </li>
            </ul>
          </div>
  
          <div v-if="activeTab === 'evolutions'">
            <h3>Evoluções</h3>
            <ul v-if="evolutionChain.length">
              <li v-for="(evo, i) in evolutionChain" :key="i">{{ capitalize(evo) }}</li>
            </ul>
            <p v-else>Este Pokémon não possui evoluções.</p>
          </div>
  
          <div v-if="activeTab === 'games'">
            <h3>Games</h3>
            <ul>
              <li v-for="(game, index) in pokemon.game_indices" :key="index">
                {{ capitalize(game.version.name) }}
              </li>
            </ul>
          </div>
        </div>
  
        <div v-else>
          <p>Carregando detalhes...</p>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from "axios";
  
  export default {
    props: ["name"],
    data() {
      return {
        pokemon: null,
        evolutionChain: [],
        activeTab: "sprites",
      };
    },
    async created() {
      try {
        const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${this.name}`);
        this.pokemon = res.data;
  
        const speciesRes = await axios.get(this.pokemon.species.url);
        const evoRes = await axios.get(speciesRes.data.evolution_chain.url);
        this.evolutionChain = this.extractEvolutionChain(evoRes.data.chain);
      } catch (error) {
        console.error("Erro ao carregar detalhes do Pokémon:", error);
      }
    },
    computed: {
      spriteList() {
        if (!this.pokemon?.sprites) return [];
        return Object.values(this.pokemon.sprites).filter(
          (img) => typeof img === "string" && img !== null
        );
      },
    },
    methods: {
      capitalize(value) {
        return value ? value.charAt(0).toUpperCase() + value.slice(1) : "";
      },
      extractEvolutionChain(chain) {
        const evolutions = [];
        let current = chain;
        while (current) {
          evolutions.push(current.species.name);
          current = current.evolves_to[0];
        }
        return evolutions;
      },
    },
  };
  </script>
  
  <style scoped>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }
  
  .modal-content {
    background: white;
    border-radius: 12px;
    padding: 2rem;
    max-height: 90vh;
    overflow-y: auto;
    max-width: 600px;
    width: 90%;
    position: relative;
  }
  
  .close-button {
    position: absolute;
    top: 0.5rem;
    right: 1rem;
    border: none;
    background: red;
    color: white;
    font-size: 1rem;
    padding: 0.4rem 0.6rem;
    border-radius: 50%;
    cursor: pointer;
  }
  
  .sprites img {
    width: 80px;
    margin: 0.5rem;
  }
  
  .tabs {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  
  .tabs button {
    padding: 0.5rem 1rem;
    border: none;
    background: #eee;
    border-radius: 8px;
    cursor: pointer;
  }
  
  .tabs button.active {
    background: #3b82f6;
    color: white;
    font-weight: bold;
  }
  </style>
  