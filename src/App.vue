<script setup>
import  HeadingComponent  from "./components/header.vue";
import  MapsComponent  from "./components/MapsComponent.vue";

</script>

<template>
  <div>
    <HeadingComponent/>
    <MapsComponent/>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
    HeadingComponent,
    MapsComponent
  }
}
</script>


<style scoped>
  div{
    width: 100%;
    }
</style>
