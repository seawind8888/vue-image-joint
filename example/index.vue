<template>
  <div id="app">
    <ImageJoint ref="joint" :width='800'/>
     <input class="export-button" value="导出图片" type="button" @click="handleExportImage" />
     <img :src="imgSrc">
  </div>
</template>

<script>
import ImageJoint from '../src/vue-image-joint'
export default {
  name: 'app',
  data() {
    return {
      imgSrc: ''
    }
  },
  components: {
    ImageJoint
  },
  methods: {
    handleExportImage() {
      
       this.imgSrc = this.$refs["joint"].getImageDataURL();
       console.log('[base64]',this.imgSrc)
      
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: row;
  align-items: flex-start;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
.export-button {
  margin: 20px;
}
</style>