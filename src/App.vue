<template>
  <v-app>
    <v-row class="d-flex justify-center pa-6 poke-logo">
      <img src="./assets/Pokémon_logo.svg" alt="Pokémon Logo" width="100%" height="100%">
    </v-row>
    <v-container>
        <v-container>
          <v-text-field
            v-model="search"
            label="Pesquisar"
            placeholder="Pikachu"
            solo
          ></v-text-field>
          <v-row>
            <v-col 
            cols="6"
            sm="4"
            md="3"
            lg="2"
            xl="2"
              v-for="pokemon in filtered_pokemons" 
              :key="pokemon.name"
            >
              <v-card @click="show_pokemon(get_id(pokemon))">
                <v-container>
                  <v-row class="mx-0 d-flex justify-center">
                    <img 
                      :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${get_id(pokemon)}.png`" 
                      :alt="`${get_name(pokemon)}`"
                      width="85%"
                    >
                  </v-row>
                  <h2 class="text-center poke-description">{{ get_name(pokemon) }}</h2>
                </v-container>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
    </v-container>

    <v-dialog v-model="show_dialog" width="500">
      <v-card v-if="selected_pokemon">
        <v-container>
          <v-row class="d-flex flex-column justify-center align-center">
            <v-col cols="4">
              <img 
                :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selected_pokemon.id}.png`" 
                :alt="`${selected_pokemon.name}`"
                width="100%"
              >
            </v-col>
            <v-col cols="10" class="poke_name text-center">
              <h1>{{ get_name(selected_pokemon) }}</h1>
            </v-col>
            <v-col cols="10" class="details d-flex justify-center flex-wrap">
              <v-chip class="details-pokemon white--text" color="lime lighten-3" label>HP: {{ selected_pokemon.stats[0].base_stat }}<br></v-chip>
              <v-chip class="details-pokemon white--text" color="red lighten-3" label>Attack: {{ selected_pokemon.stats[1].base_stat }}<br></v-chip>
              <v-chip class="details-pokemon white--text" color="blue lighten-3" label>Defense: {{ selected_pokemon.stats[2].base_stat }}<br></v-chip>
              <v-chip class="details-pokemon white--text" color="red accent-2" label>Special Attack: {{ selected_pokemon.stats[3].base_stat }}<br></v-chip>
              <v-chip class="details-pokemon white--text" color="indigo lighten-3" label>Special Defense: {{ selected_pokemon.stats[4].base_stat }}<br></v-chip>
              <v-chip class="details-pokemon white--text" color="light-green lighten-3" label>Speed: {{ selected_pokemon.stats[5].base_stat }}<br></v-chip>
            </v-col>
    
            <v-col cols="8" class="d-flex justify-center align-center">
              <h2>Evoluções</h2>
            </v-col>
            <h1 class="hide">{{ get_evolutions(get_evolution_id(selected_pokemon.id)) }}</h1>
            <v-col cols="10" class="d-flex justify-center align-center">
              <v-chip 
                class="details-pokemon white--text" 
                color="purple lighten-3" 
                label 
                v-for="item in pokemon_evolution"
                :key="item.name" 
              >
                {{ item }}
              </v-chip>
            </v-col>
          </v-row>
        </v-container>
          
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',

  components: {},

  data() {
    return {
      pokemons: [],
      search: "",
      show_dialog: false,
      selected_pokemon: null,
      pokemon_evolution: [],
      evolution_id: null,
    };
  },

  mounted () {
    axios.get("https://pokeapi.co/api/v2/pokemon?limit=500").then((response) => {
      this.pokemons = response.data.results
    });
  },

  methods: {
    get_id(pokemon){
      return pokemon.url.split("/")[6]
    },
    get_name(pokemon){
      return pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1)
    },
    get_evolution_id(id) {
      axios.get(`https://pokeapi.co/api/v2/pokemon-species/${id}`).then((response) => {
        this.evolution_id = response.data.evolution_chain.url.split("/")[6]
      });
      return this.evolution_id
    },
    show_pokemon(id){
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`).then((response) => {
        this.selected_pokemon = response.data
        this.show_dialog = !this.show_dialog
      });
    },
    get_evolutions(id){
      axios.get(`https://pokeapi.co/api/v2/evolution-chain/${id}`).then((response) => {
        let evoChain = [];

        function getEvo(arr) {
          if (arr[0].evolves_to.length > 0) {
            evoChain.push(arr[0].species.name);
            getEvo(arr[0].evolves_to);
          } else {
            evoChain.push(arr[0].species.name);
            return 0;
          }
        }
        getEvo([response.data.chain]);

        this.pokemon_evolution = evoChain
      });
      return this.pokemon_evolution
    }
  },
  computed:{
    filtered_pokemons(){
      return this.pokemons.filter((item) => {
        return item.name.includes(this.search.toLowerCase())
      });
    },
  },
};
</script>

<style>
#app {
  background: linear-gradient(100deg, #d4f1ff, #ffffff)
    no-repeat center center fixed !important;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover !important;
  background-position: center;
  min-height: 100vh;

  font-family: 'Open Sans', sans-serif;
}

.poke-logo {
  max-height: 20vh;
}

theme--light.v-chip:not(.v-chip--active) {
  background: yellow;
  color: white;
}

.details-pokemon {
  margin: 0 1em 1em 0;
}

.details {
  margin-bottom: 30px;
}

.hide{
  display: none;
}

.poke_name {
  margin-top: -30px;
}

@media screen and (max-width: 600px) {
  .poke-description{
    font-size: 92%;
  }
}
</style>
