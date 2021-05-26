<template>
  <div>
    <div class="container">
      <div class="infobox">
        <p class="title" v-if="resImage.title">{{ resImage.title }}</p>
        <p v-if="resImage.artistDisplayName">{{ resImage.artistRole +": " + resImage.artistDisplayName}}</p>
        <p v-else>Artist: unknown</p>
        <p v-if="resImage.artistDidsplayBio">{{ resImage.artistDidsplayBio}}</p>
        <p v-if="resImage.department">{{ "Dept: " + resImage.department}}</p>
        <p v-if="resImage.period">{{ resImage.period}}</p><br>
        <input type="text" v-model="searchValue" @keyup="getSearchValue"/>
        <button :class="[isDisabled ? 'disable' : '']" @click="refresh">Search</button><br>
        <!-- <button @click="disableButton"> {{ isDisabled ? 'Enable' : 'Disable' }}</button><br> -->
      </div>
      <img
        :src="resImage.primaryImage"
        :class="[isZoomed ? 'zoomedImg' : '', 'primaryImage', isLoading ? 'hide' : '']"
        :aria-label="[resImage.title ? resImage.title : 'No img title available.', ]"
      />
      <div class="iconbox">
        <div  class="buttonContainer">
          <button @click="zoom">
            {{ isZoomed ? '✦' : '✥' }}
          </button>
          <button :class="[isDisabled ? 'disable' : '']" @click="refresh">★</button>
          <button @click="addToFavorites">♡</button>
        </div>
      </div>
      <div class="imagebox">
        <ul style="list-style-type: none;">
          <li v-for="(favorites, index) in favoriteArray">
            <a :href="favoriteArray[index]" target="_blank" >
              <img :src="favoriteArray[index]" width="80vw" class="thumbPreview" />
            </a>
          </li>
        </ul>
      </div>
    </div>
    <base-spinner v-if="isLoading" />
  </div>
</template>

<script>
// Finish setting up buttons and info box positioning
// Hide Buttons and info box after 5 seconds
// Add carousel of  last viewed and favorites

import axios from 'axios'
import BaseSpinner from '../components/BaseSpinner.vue'

export default {
  component: {
    BaseSpinner
  },
  data () {
    return {
      favX: 0,
      favoriteCount: 0,
      favoriteArray: [],
      isLoading: false,
      isZoomed: false,
      isMetLogo: false,
      imageArray: [],
      isDisabled: false,
      resImage: '',
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
          this.process();
        }else {
          this.resImage = response.data
          this.imageKeep(this.resImage.primaryImage)
          this.isLoading = false;
        }
      }
    },
    async getRandomId () {
      this.isZoomed = false;
      const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/search?q='+ this.searchValue)
      if (response.status ==! 200 ){
        this.searchValue = "paintings";
        this.process ();
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
    },
    zoom () {
        this.isZoomed = !this.isZoomed
    },
    refresh () {
        this.resImage.primaryImage = "";
        this.isZoomed = false;
        this.process();
    },
    imageKeep (image) {
      if (this.imageArray.length < 10) {
          this.imageArray.unshift(image)
        } else {
          this.imageArray.pop();
          this.imageArray.unshift(image)
      }
    },
    addToFavorites(){
      //this isnt quite right but a start
      if ( !this.favoriteArray.includes(this.resImage.primaryImage) ){
        this.favoriteCount += 1;
        this.favX += 1
        let favId = 'favorite_' + this.favX
        console.log("favoriteCount", this.favoriteCount);
        if ( this.favoriteArray.length === 10) {
          this.favoriteArray.pop();
          this.favoriteCount = 9;
        }
        localStorage.setItem(favId , this.resImage.primaryImage);
        this.favoriteArray.unshift(localStorage.getItem(favId));
      }
      console.log("favorites",this.favoriteArray);
    },
    disableButton() {
      this.isDisabled = !this.isDisabled
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Open+Sans:wght@400;600&display=swap');

.container {
  background-color: black;
}
.infobox {
  background-image: linear-gradient(to right, rgba(0,0,0,.65), rgba(200,200,200,.15));
  border-radius: 10px;
  -webkit-box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
  box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
  color: white;
  font-family: 'Open Sans', sans-serif;
  font-size: 1em;
  height: 200px;
  left: calc(-20vw);
  margin: 1rem;
  position: fixed;
  padding: 1rem 2em;
  transition: left 300ms, background-image 200ms;
  width: 20vw;
 }

.infobox:hover {
  left: calc(0vw - 2em);
  max-width: max-content;
  min-width: 20vw;
  transition: left 300ms, background-image 1000ms;
}

.infobox > .title {
  font-style:italic;
  padding-bottom:0.5em;
}

.iconbox {
  bottom:20px;
  opacity: 0.3;
  padding: 4px;
  position: fixed;
  width: 100px;
 }

 .buttonContainer {
  display:flex;
  flex-direction:row;
  justify-content: space-around;
 }

 .imagebox {
  right: 0px;
  color:red;
  bottom:20px;
  padding: 4px;
  position: fixed;
  width: auto;
 }

.primaryImage {
  background-color: black;
  border: none;
  height: 100vh;
  object-fit: contain;
  overflow: hidden;
  width: 100vw;
}

.hide {
  background-color: black;
  opacity: 0;
}

.zoomedImg {
  height: 100%;
  width: 100%;
}

.disable {
  pointer-events: none;
}

.thumbPreview {
  opacity: 0.5
}
.thumbPreview:hover {
  transform: scale(1.5);
  opacity: 1;
  z-index: 1;
  position: relative;
}
</style>
