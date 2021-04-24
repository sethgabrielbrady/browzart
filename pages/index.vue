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
      newId: 0,
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
        console.log("passed id", id)
        const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + id)
        console.log("Art", response.status)
        if (response.data.primaryImage === '') {
          this.isLoading = true;
          console.log("No image-reset")
          this.process();
        }else {
          this.resImage = response.data
          this.isLoading = false;
        }
      }
    },
    async getRandomId () {
      const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/search?q='+ this.searchValue)
      console.log("status", response.status)
      //TODO - check for bad response
      if (response.status ==! 200){
        console.log("Xid", response.status)
        console.log("Response data", response.data.objectIDs)
        this.searchValue = "paintings";
        this.process();
      }else {
        console.log("something", response.data.objectIDs)
        const Arrayid = this.rngId (response.data.objectIDs.length)
        console.log("newID", Arrayid)
        console.log("arr",response.data.objectIDs[Arrayid] )
        return response.data.objectIDs[Arrayid]
      }
    },
    rngId (max) {
      //combine into getRandomId()
      const id = Math.floor(Math.random() * max);
      console.log("randomId", id)
      return id
    },
    async process () {
      this.isLoading = true;
      this.newId = await this.getRandomId()
      this.randomId = this.newId
      return this.getPrimaryImage(this.randomId.toString())
    },
    getSearchValue () {
      console.log("Searchvalue", this.searchValue)
      return this.searchValue;
    }
  }
}
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Open+Sans:wght@400;600&display=swap');

.infobox {
  font-family: 'Open Sans', sans-serif;
  position:absolute;
  opacity: 1;
  border-radius: 12px;
  margin: 1rem;
  width: 20vw;
  padding: 1em;
  color: white;
  background-image: linear-gradient(to right, rgba(0,0,0,.65), rgba(255,255,255,.15));
  -webkit-box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
  box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
  font-size: 1em;
  left: calc(-20vw);
  transition: opacity 200ms, left 300ms, background-image 200ms;
  height: 180px;
 }

.infobox:hover {
  min-width: 20vw;
  max-width: max-content;
  background-image: linear-gradient(to right, rgba(0,0,0,.65), rgba(0,0,0,.15));
  transition: opacity 200ms, left 300ms, background-image 1000ms;
  left: calc(0vw - 1em);
  opacity: 1;
}

.primaryImage {
  padding-top: 2rem;
  width: 100vw;
  height: 100vh;
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
  background-color: black;
}
</style>
