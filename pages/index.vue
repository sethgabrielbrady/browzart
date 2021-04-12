<template>
  <div class="container">
    <p>{{resImage.primaryImage}}</p>
     <p>{{randomId}}</p>
    <div class="primaryImage" :style="{'background-image': 'url(' + resImage.primaryImage + ')'}" />
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      resImage: 'Initial State',
      randomId: ''
    }
  },
  created () {
    this.randomId = this.rngId(47213)
    return this.getPrimaryImage()
  },
  methods: {
    async getPrimaryImage () {
      const resArt = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + this.randomId)
      this.resImage = resArt.data
    },
    rngId (max) {
      const newId = Math.floor(Math.random() * max);
      return newId.toString();
    }

  }
}
</script>

<style>
.primaryImage {
  width: 100vw;
  height: 100vh;
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
  background-color: black;
}
</style>
