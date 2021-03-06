<template >
  <q-layout view="lHh Lpr lFf" :style="getColor">
    <NavbarPokemonPage />
    <q-page-container>
    <q-page>
      <div  class="row justify-center">
        <div class="col-sm-5 col-xs-12">
          <div class="row justify-center">
            <div class="col-12 text-center ">
              <img v-if="pokemon" :src="getImageUrl()" class="mainImage" />
            </div>

            <div v-if="pokemon" class="highlight-box col-12">
              <p  class="text-h3 text-center pokemon-name">
                {{ pokemon.name[0].toUpperCase() + pokemon.name.slice(1) }}
              </p>

              <div  class="type-chips">
                <div v-for="type in types" 
                  :key="type" 
                  class="type-chip text-center">
                  {{type}}
                </div>
              </div>
              <p  class="height-weight">Height  <i>{{ height }}</i></p>
              <p  class="height-weight">Weight  <i>{{ weight }}</i></p>
            </div>
          </div>
        </div>
      
        <div class="col-sm-7 col-xs-12 q-pt-lg chart-container">
          <canvas id="statsChart" class="statsChart" ref="statsChart" ></canvas> 
        </div>
      </div>

    </q-page>
    </q-page-container>
  </q-layout>
</template>


<script>
import NavbarPokemonPage from '../components/NavbarPokemonPage.vue'
import PokemonService from '../services/PokemonService'
import { Type_Icon_Colors } from '../utils/constants'
import Chart from 'chart.js'

export default {
  name: 'PokemonPage',

components: { NavbarPokemonPage },

  data() {
    return{
      pokemon: null
    }
  },

  async created() {
    PokemonService
      .getSinglePokemon(this.id)
      .then(res => {
        this.pokemon = res
        document.title = res.name[0].toUpperCase() + res.name.slice(1) +' - Pokédex'
        
      })
      .catch( e => {
        this.$q.loading.hide()
        this.showErrorNotif(e.message)
      })
  },

  mounted(){
    this.isPokemonLoaded()
  },

  computed: {
    getColor() {
      return {
        '--color': this.pokemon ? 
          this.getNavbarColor(this.pokemon.types[0].type.name) :
          'fff'
      }
    },
    types(){
      return this.pokemon.types.map(type => 
        type.type.name[0].toUpperCase() + type.type.name.slice(1)
      )
    },
    height(){
      // Height is received in decimeters ¯\_(ツ)_/¯
      let heightInInches = this.pokemon.height * 3.937
      return Math.trunc(heightInInches / 12) + '\'' + ' ' + Math.floor(heightInInches % 12 ) + '"'
    }, 
    weight(){
      return this.pokemon.weight / 10 + ' kg'
    },
    baseStats(){
      return this.pokemon.stats.map(stat => stat.base_stat)
    },
    id() {
      return this.$route.params['id']
    }
  },  

  methods: {
    getImageUrl(){
      return this.pokemon? this.pokemon.sprites.other.dream_world.front_default || 
        this.pokemon.sprites.front_default : 
        '/imagenotavailable.png'
    },
    isPokemonLoaded() {
      if(!this.pokemon) {
        this.$q.loading.show({
        message: 'Loading....'
        })
      }
      else{
        this.$q.loading.hide()
        this.$nextTick(() => 
          this.showChart()
        )
      }
    },
    showErrorNotif(message) {
      this.$q.notify({
        message,
        color: 'deep-orange-9',
        icon: 'warning',
        badgeColor: 'transparent',
        badgeTextColor: 'transparent',
        badgeClass: 'shadow-0',
      })
    },
    getNavbarColor(type){
      return Type_Icon_Colors[type]
    },
    showChart() {
      new Chart(this.$refs.statsChart, {
        type: 'radar',
        data: {
          labels: ['HP', 'Attack', 'Defense', 'Sp. Atk', 'Sp. Def', 'Speed'],
          datasets: [{
            label: 'Abilities',
            data: this.baseStats,
            backgroundColor: 'rgba(57, 59, 68, 0.8)', 
            borderWidth: 3,
            pointRadius: 0.5
          }]
        },
      
        options: {
          title: {
            display: true,
            text: 'Base Stats',
            fontSize: 18,
          },
          scale: {
            angleLines: {
              lineWidth: 2,
              display: true
            },
            ticks: {
              min: 0,
              max: 260,
              stepSize: 52,
              fontColor: '#0C0B0A'
            },
            pointLabels: {
              fontSize: 13,
              fontColor: '#0C0B0A'
            }
          },
          legend: {
          display: false
          },
          tooltips:{
            enabled:false,
          },
          responsive: false
          }
      })
    }
  },

  watch: {
    pokemon() {
      this.isPokemonLoaded()
    }
  }
}
</script>

<style lang="scss" scoped>

   $color: var(--color);

  .q-page-container {
    overflow-x: hidden; 
  }
  .empty{
    height: 0px;
    background-color: black;
  }
  .mainImage {
    width: 260px;
    height: 260px;
    padding-top: 20px;
    margin-left:auto;
  }
  .highlight-box{
    background-color: $mainNav;
    margin-top: 15px;
    width: 360px;
    height: 210px;
    border-radius: 10% 1%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    color: $cardWhite
  }
  .pokemon-name{
    font-style: italic;
  }
  .height-weight{
    font-size: 1rem;
    margin: 0 0 8px;
  }
  .type-chips{
    display: flex;
    flex-direction: row;
    margin-top: 0px;
    margin-bottom: 15px;
  }
  .type-chip {
    background: $cardWhite;
    color: $mainNav;
    border-radius: 8px;
    margin-left: 15px;
    font-size: 0.9rem;
    padding: 2px 8px;
  }
  .chart-container {
    position: relative;
    height: 400px;
  }
  .statsChart{
    display: block; 
    height: 379px !important; 
    width: 759px !important;
  }

  @media only screen 
  and (min-device-width: 300px) 
  and (max-device-width: 380px)
  {
    .statsChart{
      display: block; 
      height: 222px !important; 
      width: 444px !important;
    }
    .chart-container {
      left: -40px
    }
  }

  @media only screen 
  and (min-device-width: 380px) 
  and (max-device-width: 420px)
  {
    .statsChart{
      display: block; 
      height: 222px !important; 
      width: 444px !important;
    }
  }

  @media only screen 
  and (min-device-width: 300px) 
  and (max-device-width: 599px)
  {
    .highlight-box{
      margin-top: 25px;
      width: 100vw;
      border-radius: 0;
    }
  }

  @media only screen 
  and (min-device-width: 420px) 
  and (max-device-width: 1024px)
  {
    .statsChart{
      display: block; 
      height: 253px !important; 
      width: 506px !important;
    }
    .highlight-box{
      margin-top: 25px;
    }
  }
</style>