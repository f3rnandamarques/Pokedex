<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal">
      <button class="fechar" @click="$emit('close')">Fechar</button>
      <h2 class="titulo">{{ pokemon.name }}</h2>

      <!-- Abas -->
      <div class="tabs">
        <button
          v-for="tab in abas"
          :key="tab"
          :class="{ ativa: abaAtiva === tab }"
          @click="abaAtiva = tab"
        >
          {{ tab }}
        </button>
      </div>

      <!-- Conteúdo das Abas -->
      <div class="conteudo-tab">
        <!-- Informações -->
        <div v-if="abaAtiva === 'Informações'">
          <p><strong>Nome:</strong> {{ pokemon.name }}</p>
          <p><strong>ID:</strong> {{ pokemon.id }}</p>
          <p><strong>Tipo(s):</strong> {{ tipos }}</p>
          <p><strong>Espécie:</strong> {{ especie }}</p>
        </div>

        <!-- Sprites -->
        <div v-else-if="abaAtiva === 'Sprites'">
          <img
            v-for="(sprite, chave) in spritesFiltrados"
            :key="chave"
            :src="sprite"
            :alt="chave"
            class="sprite"
          />
        </div>

        <!-- Movimentos -->
        <div v-else-if="abaAtiva === 'Movimentos'">
          <ul>
            <li v-for="mov in pokemon.moves" :key="mov.move.name">
              {{ mov.move.name }}
            </li>
          </ul>
        </div>

        <!-- Evoluções -->
        <div v-else-if="abaAtiva === 'Evoluções'">
          <ul v-if="pokemon.evolucoes && pokemon.evolucoes.length">
            <li v-for="evo in pokemon.evolucoes" :key="evo">{{ evo }}</li>
          </ul>
          <p v-else>Nenhuma evolução encontrada.</p>
        </div>

        <!-- Games -->
        <div v-else-if="abaAtiva === 'Games'">
          <ul>
            <li v-for="g in pokemon.game_indices" :key="g.version.name">
              {{ g.version.name }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    pokemon: Object,
  },
  data() {
    return {
      abaAtiva: "Informações",
      abas: ["Informações", "Sprites", "Movimentos", "Evoluções", "Games"],
    };
  },
  computed: {
    tipos() {
      return this.pokemon.types.map((t) => t.type.name).join(", ");
    },
    especie() {
      // Acessa o campo genus corretamente
      return this.pokemon.genus || "Desconhecida";
    },
    spritesFiltrados() {
      const sprites = this.pokemon.sprites;
      const validSprites = {};
      for (const chave in sprites) {
        if (
          sprites[chave] &&
          typeof sprites[chave] === "string" &&
          sprites[chave].includes("https")
        ) {
          validSprites[chave] = sprites[chave];
        }
      }
      return validSprites;
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
  z-index: 999;
}

.modal {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  width: 95%;
  max-width: 500px;
  max-height: 95vh;
  overflow-y: auto;
  position: relative;
}

.fechar {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #eee;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 4px;
}

.titulo {
  text-transform: capitalize;
  text-align: center;
}

.tabs {
  display: flex;
  justify-content: space-around;
  margin: 1rem 0;
  flex-wrap: wrap;
}

.tabs button {
  flex: 1;
  padding: 0.5rem;
  background: #eee;
  border: none;
  cursor: pointer;
  border-radius: 4px;
  margin: 2px;
}

.tabs button.ativa {
  background: #c62828;
  color: white;
  font-weight: bold;
}

.conteudo-tab {
  text-align: center;
}

.sprite {
  width: 96px;
  margin: 0.5rem;
}
</style>
