<template>
  <div>
    <div class="container">
      <div class="infobox">
        <p v-if="resImage.title" style="font-style:italic; padding-bottom:.5em;">{{ resImage.title }}</p>
        <p v-if="resImage.artistDisplayName">{{ resImage.artistRole +": " + resImage.artistDisplayName}}</p>
        <p v-else>Artist: unknown</p>
        <p v-if="resImage.artistDidsplayBio">{{ resImage.artistDidsplayBio}}</p>
        <p v-if="resImage.department">{{ "Dept: " + resImage.department}}</p>
        <p v-if="resImage.period">{{ resImage.period}}</p><br>
        <input type="text" v-model="searchValue" @keyup="getSearchValue"/>
        <button @click="process">Search</button>
      </div>
      <div class="primaryImage" :style="{'background-image': 'url(' + resImage.primaryImage + ')'}" />
    </div>
    <base-spinner v-if="isLoading" />
  </div>
</template>

<script>
import axios from 'axios'
import BaseSpinner from '../components/BaseSpinner.vue'

export default {
  component: {
    BaseSpinner
  },
  data () {
    return {
      resImage: 'Initial State',
      randomId: 0,
      isLoading: false,
      searchValue: "paintings"
    }
  },
  created () {
    return this.process();
  },
  methods: {
    async getPrimaryImage (id) {
      if( this.randomId > 0){
        const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + id)
        if (response.data.primaryImage === '') {
          this.isLoading = true;
          this.process();
        }else {
          this.resImage = response.data
          this.isLoading = false;
        }
      }
    },
    async getRandomId () {
      const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/search?q='+ this.searchValue)
      if (response.status ==! 200 ){
        this.searchValue = "paintings";
        this.process();
      }else {
        return response.data.objectIDs[this.rngId (response.data.objectIDs.length)]
      }
    },
    rngId (max) {
      return Math.floor(Math.random() * max);
    },
    async process () {
      this.isLoading = true;
      this.randomId = await this.getRandomId()
      return this.getPrimaryImage(this.randomId.toString())
    },
    getSearchValue () {
      return this.searchValue;
    }
  }
}
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Open+Sans:wght@400;600&display=swap');

.infobox {
  background-image: linear-gradient(to right, rgba(0,0,0,.55), rgba(0,0,0,.25));
  -webkit-box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
  border-radius: 10px;
  box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
  color: white;
  font-family: 'Open Sans', sans-serif;
  font-size: 1em;
  height: 180px;
  left: calc(-20vw);
  margin: 1rem;
  position: absolute;
  padding: 1em;
  width: 20vw;
  transition: left 300ms, background-image 200ms;
 }

.infobox:hover {
  background-image: linear-gradient(to right, rgba(0,0,0,.55), rgba(0,0,0,.15));
  left: calc(0vw - 1em);
  min-width: 20vw;
  max-width: max-content;
  transition: left 300ms, background-image 1000ms;
}

.primaryImage {
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
  background-color: black;
  padding-top: 2rem;
  height: 100vh;
  width: 100vw;
}

</style>
