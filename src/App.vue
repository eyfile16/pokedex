<template>
  <div class="pokedex-container">
    <iframe
      ref="youtubeIframe"
      width="0"
      height="0"
      src="https://www.youtube.com/embed/FL7ktlvZnH8?autoplay=1&loop=1&playlist=FL7ktlvZnH8"
      frameborder="0"
      allow="autoplay; encrypted-media"
      allowfullscreen
    ></iframe>

    <div class="pokedex">
      <div class="left-panel">
        <div class="screen">
          <img :src="pokemonImage" :alt="pokemonName" class="pokemon-image" />
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

        <div class="directional-pad">
          <div v-for="n in 9" :key="n" class="number-button" @click="enterNumber(n)">
            {{ n }}
          </div>
          <div class="number-button" @click="enterNumber(0)">0</div>
          <div class="clear-button" @click="clearInput">Borrar</div>
        </div>

        <div class="buttons">
          <div class="button large-button blue-button" @click="prevPokemon">Anterior</div>
          <div class="button large-button green-button" @click="nextPokemon">Siguiente</div>
          <div class="button large-button yellow-button" @click="randomPokemon">Aleatorio</div>
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

        <div class="weaknesses-box">
          <p>Debilidades:</p>
          <div v-for="weakness in pokemonWeaknesses" :key="weakness" 
               :style="{ backgroundColor: getTypeColor(weakness), color: '#000' }"
               class="weakness">
            {{ weakness.charAt(0).toUpperCase() + weakness.slice(1) }}
          </div>
        </div>

        <div class="stats-box">
          <p>Estadísticas:</p>
          <div v-for="stat in pokemonStats" :key="stat.stat.name" class="stat-bar">
            <p>{{ stat.stat.name }}: {{ stat.base_stat }}</p>
            <div class="progress">
              <div class="progress-bar" :style="{ width: stat.base_stat + '%' }"></div>
            </div>
          </div>
        </div>

        <div class="music-control">
          <button @click="toggleMusic">
            {{ isMusicPlaying ? 'Pausar Música' : 'Reproducir Música' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const pokemonNumber = ref(1);
const pokemonName = ref('Bulbasaur');
const pokemonImage = ref('https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/1.png');
const pokemonStats = ref([]);
const pokemonTypes = ref([]);
const pokemonWeaknesses = ref([]);
const pokemonTypeColors = ref([]);
const totalPokemons = 900;
const searchInput = ref('');
const isMusicPlaying = ref(true);

const fetchPokemon = async (pokemonIdentifier) => {
  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemonIdentifier}`);
    if (!response.ok) {
      throw new Error('Pokémon no encontrado');
    }
    const data = await response.json();
    pokemonNumber.value = data.id;
    pokemonName.value = data.name.charAt(0).toUpperCase() + data.name.slice(1);
    pokemonImage.value = data.sprites.front_default;
    pokemonStats.value = data.stats;
    pokemonTypes.value = data.types.map(typeInfo => typeInfo.type.name);
    pokemonTypeColors.value = data.types.map(typeInfo => getTypeColor(typeInfo.type.name));
    pokemonWeaknesses.value = getWeaknesses(pokemonTypes.value);
  } catch (error) {
    pokemonNumber.value = null;
    pokemonName.value = 'Pokémon no encontrado';
    pokemonImage.value = 'https://via.placeholder.com/300x250?text=Error';
    pokemonStats.value = [];
    pokemonTypes.value = [];
    pokemonWeaknesses.value = [];
    pokemonTypeColors.value = [];
  }
};

const getWeaknesses = (types) => {
  const weaknesses = {
    bug: ['fire', 'flying', 'rock', 'ghost', 'fairy'],
    dark: ['fighting', 'bug', 'fairy'],
    dragon: ['ice', 'dragon', 'fairy'],
    electric: ['ground'],
    fairy: ['poison', 'steel'],
    fighting: ['flying', 'psychic', 'fairy'],
    fire: ['water', 'ground', 'rock'],
    flying: ['electric', 'ice', 'rock'],
    ghost: ['ghost', 'dark'],
    grass: ['fire', 'ice', 'poison', 'flying', 'bug'],
    ground: ['water', 'grass', 'ice'],
    ice: ['fire', 'fighting', 'rock', 'steel'],
    normal: ['fighting'],
    poison: ['ground', 'psychic'],
    psychic: ['bug', 'ghost', 'dark'],
    rock: ['fighting', 'ground', 'steel', 'water', 'grass'],
    steel: ['fire', 'fighting', 'ground'],
    water: ['electric', 'grass']
  };

  const weaknessesList = new Set();
  types.forEach(type => {
    weaknesses[type]?.forEach(weakness => weaknessesList.add(weakness));
  });
  return Array.from(weaknessesList);
};

const getTypeColor = (type) => {
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
};

const nextPokemon = () => {
  if (pokemonNumber.value < totalPokemons) {
    pokemonNumber.value++;
  } else {
    pokemonNumber.value = 1;
  }
  fetchPokemon(pokemonNumber.value);
};

const prevPokemon = () => {
  if (pokemonNumber.value > 1) {
    pokemonNumber.value--;
  } else {
    pokemonNumber.value = totalPokemons;
  }
  fetchPokemon(pokemonNumber.value);
};

const randomPokemon = () => {
  const randomNum = Math.floor(Math.random() * totalPokemons) + 1;
  fetchPokemon(randomNum);
};

const enterNumber = (number) => {
  searchInput.value += number.toString();
};

const clearInput = () => {
  searchInput.value = '';
  fetchPokemon(1); 
};

const searchPokemonByInput = () => {
  const input = searchInput.value.trim();
  if (input) {
    const identifier = isNaN(input) ? input.toLowerCase() : parseInt(input);
    fetchPokemon(identifier);
    searchInput.value = ''; 
  }
};

const toggleMusic = () => {
  const iframe = document.querySelector('iframe');
  if (isMusicPlaying.value) {
    iframe.contentWindow.postMessage('{"event":"command","func":"pauseVideo","args":""}', '*');
  } else {
    iframe.contentWindow.postMessage('{"event":"command","func":"playVideo","args":""}', '*');
  }
  isMusicPlaying.value = !isMusicPlaying.value;
};

onMounted(() => {
  fetchPokemon(pokemonNumber.value);
});
</script>

<style scoped>
.pokedex-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px; 
}

.pokedex {
  display: flex;
  flex-direction: row;
  border: 3px solid red;
  border-radius: 10px;
  overflow: hidden;
  max-width: 1200px; 
  width: 100vw; 
  height: auto; 
}

.left-panel,
.right-panel {
  flex: 1;
  padding: 20px;
}

.left-panel {
  background-color: #cce7ff;
}

.right-panel {
  background-color: #e6ffe6;
}

.screen {
  width: 100%; 
  height: 400px; 
  background-color: #fff;
  border: 2px solid #000;
  border-radius: 5px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1);
}

.pokemon-image {
  max-width: 90%; 
  max-height: 90%; 
}

.directional-pad {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 5px;
  margin-top: 10px;
}

.button,
.number-button,
.control-button {
  width: 100%; 
  height: 50px; 
  border: 1px solid #000;
  border-radius: 5px;
  margin: 5px 0; 
  background-color: #ffcc00;
  color: #000;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s;
}

.button:hover,
.number-button:hover,
.control-button:hover {
  background-color: #ff9900;
  transform: scale(1.05);
}

.large-button {
  height: 60px; 
}

.clear-button {
  background-color: red; 
  color: white; 
  border: 2px solid #000; 
  transition: background-color 0.3s, transform 0.2s;
}

.clear-button:hover {
  background-color: darkred; 
  transform: scale(1.05); 
}

.search-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 10px;
  width: 100%; 
}

.search-container input {
  width: 100%;
  padding: 5px;
  border-radius: 5px;
  border: 1px solid #ccc;
  outline: none;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.info-box,
.type-box,
.weaknesses-box,
.stats-box {
  border: 1px solid #000;
  border-radius: 5px;
  padding: 10px;
  margin-top: 10px;
  background-color: #fff3cd;
  color: #000; 
}

.type,
.weakness {
  display: inline-block;
  margin-right: 5px;
  padding: 5px;
  border-radius: 3px;
  color: #000; 
}

.stats-box .stat-bar {
  margin-top: 5px;
  transition: transform 0.3s;
}

.progress {
  background-color: #e0e0e0;
  border-radius: 5px;
}

.progress-bar {
  height: 10px;
  background-color: #76c7c0;
  border-radius: 5px;
}

.music-control {
  margin-top: 20px;
  text-align: center; 
}

@media (max-width: 768px) {
  .pokedex {
    flex-direction: column; 
    max-width: 100%; 
  }

  .left-panel,
  .right-panel {
    padding: 10px; 
  }

  .screen {
    height: 300px; 
  }

  .button,
  .clear-button,
  .number-button {
    height: 40px; 
  }

  .large-button {
    height: 50px; 
  }
}
</style>
