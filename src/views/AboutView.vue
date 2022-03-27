<script lang="ts">
  export default {
    mounted(){
      this.emitter.emit("updateInfoPaneBlurbEvent", 'The stirring tale of...  Us.  Me.');
    }
  }
</script>

<template>
  <div class="about">
    <div>
      <h1>Hi, I'm Stuart.</h1>
      <p>Nice to meet you.  Now go beat the game.</p>
    </div>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
