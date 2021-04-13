<template>
  <div class="container">
    <div style="position:absolute; color: white;">
      <p>{{ resImage.title }}</p>
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
  methods: {
     async getPrimaryImage (id) {
      if( this.randomId > 0){
        const resArt = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + id)
        console.log("Art", resArt)
        this.resImage = resArt.data
      }
    },
    async getRandomId () {
      const resId = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects')
      console.log("id", resId.data.total)
      return resId.data.total
    },
    rngId (max) {
      const id = Math.floor(Math.random() * max);
      console.log("randomId", id)
      return id
    },
    async process () {
      this.newId =  await this.getRandomId()
      this.randomId = this.rngId(this.newId)
      return this.getPrimaryImage(this.randomId.toString())
    },
  }
}
</script>

<style>
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
