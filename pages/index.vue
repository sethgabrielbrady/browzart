<template>
  <div class="container">
    <div class="infobox">
      <p>{{ resImage.title }}</p><br>
      <p>{{ resImage.artistDisplayName}}</p>
       <p>{{ resImage.artistDidsplayBio}}</p>
        <p>{{ resImage.artistBeginDate + " to " + resImage.artistEndDate}}</p>
      <p>{{ resImage.department}}</p>
      <p>{{ resImage.period}}</p>
      <p>{{ resImage.artistRole}}</p>
    </div>
    <div class="primaryImage" :style="{'background-image': 'url(' + resImage.primaryImage + ')'}" />
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      resImage: 'Initial State',
      randomId: 0,
      newId: 0
    }
  },
  created () {
    return this.process();
  },
  //move getting and image to isolated component
  //add spinner component
  methods: {
    async getPrimaryImage (id) {
      if( this.randomId > 0){
        console.log("passed id", id)
        const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + id)
        console.log("Art", response.status)
        if (response.data.primaryImage === '') {
          console.log("No image-reset")
          this.process();
        }else {
          this.resImage = response.data
        }
      }
    },
    async getRandomId () {
      const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/search?q=sculpture')
      console.log("status", response.status)
      //TODO - check for bad response
      if (response.status ==! 200){
        console.log("Xid", response.status)
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
      this.newId = await this.getRandomId()
      // this.randomId = this.rngId(this.newId)
      this.randomId = this.newId
      return this.getPrimaryImage(this.randomId.toString())
    },
  }
  //add favorites
}
</script>

<style>
.infobox {
  position:absolute;
  opacity: 0;
  width: 100%;
}
.infobox:hover {
  color: white;
  background-color: black;
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
