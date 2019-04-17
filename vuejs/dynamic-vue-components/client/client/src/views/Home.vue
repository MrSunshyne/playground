<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Local component"/>
    <div v-if="library">
      <Component v-for="comp in library" :key="comp.name" :is="comp.comp" />
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import externalComponent from '../utils/external-component';

export default {
  name: 'home',
  components: {
    HelloWorld
  },
  data() {
    return {
      library : null
    }
  },
  mounted() {
    fetch('/api/component-library.json')
    .then(response => {
      return response.json()
    })
    .then(response => {
      this.library = response.map(res => {
        let result = {
          ...res,
          comp : () => externalComponent(`http://localhost:4444/${res.path}`)
        } 
        return result;
      });
    })
  }
}
</script>
