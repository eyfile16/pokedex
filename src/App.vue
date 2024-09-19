<template>
  <div class="pokedex-container">
    <!-- Iframe oculto para la música de fondo de YouTube -->
    <iframe
      ref="youtubeIframe"
      width="0"
      height="0"
      src="https://www.youtube.com/embed/im6tbN9SZXs?autoplay=1&loop=1&playlist=im6tbN9SZXs"
      frameborder="0"
      allow="autoplay; encrypted-media"
      allowfullscreen
    ></iframe>

    <div class="pokedex">
      <div class="left-panel">
        <div class="screen">
          <img :src="pokemonImage" :alt="pokemonName" />
        </div>

        <div class="buttons">
          <div class="button blue-button" @click="prevPokemon"></div>
          <div class="button green-button" @click="nextPokemon"></div>
          <div class="button yellow-button" @click="randomPokemon"></div>
        </div>

        <div class="directional-pad">
          <div v-for="n in 9" :key="n" class="number-button" @click="enterNumber(n)">
            {{ n }}
          </div>
          <div class="center-button" @click="clearLastDigit">Borrar</div>
        </div>

        <div class="search-container">
          <input
            type="text"
            v-model="searchInput"
            placeholder="Número o nombre de Pokémon"
            @keyup.enter="searchPokemonByInput"
          />
          <button @click="searchPokemonByInput">Buscar</button>
        </div>
      </div>

      <div class="right-panel">
        <div class="info-box">
          <p>Pokémon Nº {{ pokemonNumber }}</p>
          <p>{{ pokemonName }}</p>
        </div>

        <div class="type-box">
          <p>Tipo(s):</p>
          <div
            v-for="(type, index) in pokemonTypes"
            :key="index"
            :style="{ backgroundColor: pokemonTypeColors[index] }"
            class="type"
          >
            {{ type.charAt(0).toUpperCase() + type.slice(1) }}
          </div>
        </div>

        <div class="stats-box">
          <div v-for="stat in pokemonStats" :key="stat.stat.name" class="stat-bar">
            <p>{{ stat.stat.name }}: {{ stat.base_stat }}</p>
            <div class="progress">
              <div class="progress-bar" :style="{ width: stat.base_stat + '%' }"></div>
            </div>
          </div>
        </div>

        <div class="control-buttons">
          <div class="control-button red-button" @click="prevPokemon"></div>
          <div class="control-button green-button" @click="nextPokemon"></div>
        </div>
      </div>
    </div>

    <!-- Botón para controlar la música -->
    <div class="music-control">
      <button @click="toggleMusic">
        {{ isMusicPlaying ? 'Pausar Música' : 'Reproducir Música' }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pokemonNumber: 1, 
      pokemonName: "Bulbasaur", 
      pokemonImage: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/1.png", 
      pokemonStats: [], 
      pokemonTypes: [], 
      pokemonTypeColors: [], 
      totalPokemons: 900, 
      searchInput: "", 
      isMusicPlaying: true, // Control para la música
    };
  },
  methods: {
    async fetchPokemon(pokemonIdentifier) {
      try {
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemonIdentifier}`);
        if (!response.ok) {
          throw new Error('Pokémon no encontrado');
        }
        const data = await response.json();
        this.pokemonNumber = data.id;
        this.pokemonName = data.name.charAt(0).toUpperCase() + data.name.slice(1);
        this.pokemonImage = data.sprites.front_default;
        this.pokemonStats = data.stats;
        this.pokemonTypes = data.types.map(typeInfo => typeInfo.type.name);
        this.pokemonTypeColors = data.types.map(typeInfo => this.getTypeColor(typeInfo.type.name));
      } catch (error) {
        this.pokemonNumber = null;
        this.pokemonName = 'Pokémon no encontrado';
        this.pokemonImage = 'https://via.placeholder.com/300x250?text=Error';
        this.pokemonStats = [];
        this.pokemonTypes = [];
        this.pokemonTypeColors = [];
      }
    },
    getTypeColor(type) {
      const typeColors = {
        bug: '#A8B820',
        dark: '#705848',
        dragon: '#7038F8',
        electric: '#F8D030',
        fairy: '#F0B6BC',
        fighting: '#C03028',
        fire: '#F08030',
        flying: '#A890F0',
        ghost: '#705898',
        grass: '#78C850',
        ground: '#E0C068',
        ice: '#98D8D8',
        normal: '#A8A878',
        poison: '#A040A0',
        psychic: '#F85888',
        rock: '#B8A038',
        steel: '#B8B8D0',
        water: '#6890F0'
      };
      return typeColors[type] || '#000000'; 
    },
    nextPokemon() {
      if (this.pokemonNumber < this.totalPokemons) {
        this.pokemonNumber++;
      } else {
        this.pokemonNumber = 1; 
      }
      this.fetchPokemon(this.pokemonNumber);
    },
    prevPokemon() {
      if (this.pokemonNumber > 1) {
        this.pokemonNumber--;
      } else {
        this.pokemonNumber = this.totalPokemons; 
      }
      this.fetchPokemon(this.pokemonNumber);
    },
    randomPokemon() {
      const randomNum = Math.floor(Math.random() * this.totalPokemons) + 1;
      this.fetchPokemon(randomNum);
    },
    enterNumber(number) {
      this.searchInput += number.toString();
    },
    clearLastDigit() {
      this.searchInput = this.searchInput.slice(0, -1);
    },
    searchPokemonByInput() {
      const identifier = this.searchInput.toLowerCase().trim();
      if (identifier) {
        this.fetchPokemon(identifier);
      }
      this.searchInput = ""; 
    },
    toggleMusic() {
      const iframe = this.$refs.youtubeIframe;
      if (this.isMusicPlaying) {
        iframe.src = "";
        this.isMusicPlaying = false;
      } else {
        iframe.src =
          "https://www.youtube.com/embed/im6tbN9SZXs?autoplay=1&loop=1&playlist=im6tbN9SZXs";
        this.isMusicPlaying = true;
      }
    }
  },
  mounted() {
    this.fetchPokemon(this.pokemonNumber); 
  }
};
</script>

<style scoped>
body {
  margin: 0;
  padding: 0;
  background-color: #f4f4f4;
  font-family: 'Press Start 2P', cursive;
}

.pokedex-container {
  display: flex;
  width: 100%;
  height: 100vh;
  justify-content: center;
  align-items: center;
  background-color: #d32f2f;
  padding: 10px;
}

.pokedex {
  display: flex;
  width: 100%;
  max-width: 1000px;
  height: 600px;
  background-color: #c2185b;
  border: 5px solid #000;
  border-radius: 10px;
  overflow: hidden;
  flex-direction: row;
}

.left-panel, .right-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.left-panel {
  background-color: #b71c1c;
  border-right: 5px solid #000;
}

.right-panel {
  background-color: #d81b60;
}

.screen {
  width: 100%;
  max-width: 300px;
  height: auto;
  background-color: #9c9c9c;
  border: 4px solid #000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.screen img {
  width: 100%;
  height: 100%;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.buttons {
  margin-top: 20px;
  display: flex;
  gap: 15px;
}

.button {
  width: 40px;
  height: 40px;
  border: 2px solid #000;
  border-radius: 50%;
}

.blue-button {
  background-color: #1976d2;
}

.green-button {
  background-color: #43a047;
}

.yellow-button {
  background-color: #ffca28;
}

.directional-pad {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 2px;
  margin-top: 20px;
}

.number-button {
  height: 28px;
  background-color: #333;
  color: white;
  font-size: 14px;
  border: 2px solid #000;
  text-align: center;
  line-height: 28px;
}

.center-button {
  width: 85px;
  height: 28px;
  background-color: #000;
  color: white;
  border: 2px solid #000;
  text-align: center;
  line-height: 28px;
  cursor: pointer;
}

.search-container {
  display: flex;
  margin-top: 20px;
  width: 100%;
  justify-content: center;
  align-items: center;
}

.search-container input, .search-container button {
  height: 40px;
  font-size: 18px;
  margin: 0 5px;
}

.info-box {
  background-color: #333;
  border: 2px solid #000;
  text-align: center;
  padding: 10px;
  margin-bottom: 20px;
}

.type-box {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.type {
  color: white;
  margin: 5px;
  padding: 10px;
  border-radius: 10px;
}

.stats-box p {
  font-size: 16px;
  margin: 5px 0;
}

.stat-bar {
  display: flex;
  flex-direction: column;
  margin: 10px 0;
}

.progress {
  width: 100%;
  height: 10px;
  background-color: #ccc;
  border-radius: 5px;
}

.progress-bar {
  height: 10px;
  background-color: #4caf50;
  border-radius: 5px;
}

.control-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.control-button {
  width: 40px;
  height: 40px;
  border: 2px solid #000;
  border-radius: 50%;
}

.red-button {
  background-color: #e53935;
}

.green-button {
  background-color: #43a047;
}

/* Estilo para el botón de control de música */
.music-control {
  position: absolute;
  top: 20px;
  right: 20px;
}

.music-control button {
  background-color: #ffca28;
  border: 2px solid #000;
  border-radius: 5px;
  padding: 10px;
  font-size: 14px;
  cursor: pointer;
}
</style>
