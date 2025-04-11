<script>
import axios from "axios";

export default {
  props: ["name"],
  data() {
    return {
      pokemon: null,
    };
  },
  async created() {
    try {
      const res = await axios.get(
        `https://pokeapi.co/api/v2/pokemon/${this.name}`
      );
      this.pokemon = res.data;
    } catch (error) {
      console.error("Erro ao buscar Pokémon:", error);
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
      if (!value) return "";
      return value.charAt(0).toUpperCase() + value.slice(1);
    },
  },
};
</script>
