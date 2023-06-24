<template>
    <div id="content" style="position:relative; z-index: 2;">
        <LogoImg/>
        <h1>onFire</h1>
    </div>
</template>

<script>
import LogoImg from './icons/logo.vue'
export default {
    name: 'HeadingComponent',
    components: {
    LogoImg
  }
}
</script>

<style scoped>

#content{
    align-items: center;
    display: flex;
    background: rgb(255, 36, 36);
}
</style>
