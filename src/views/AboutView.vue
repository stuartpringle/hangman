<script>
  import { defineComponent } from 'vue'

  export default defineComponent({
    mounted(){
      this.emitter.emit("updateInfoPaneBlurbEvent", 'Want to know more about me?  Brace yourself for an heroic (if short) tale of derring-do...');
    }
  })
</script>

<template>
  <div class="about">
    <div class="content">
      <h1>Hi, I'm Stuart.</h1>
      <p>Nice to meet you.  I do web stuff.  Now shoo!  <RouterLink to="/">Play my game</RouterLink>.</p>
    </div>
  </div>
</template>

<style>
.content {
  text-align: center;
  margin: auto;
  margin-top: 30px;
}
@media (min-width: 1024px) {
  .about {
    min-height: calc(100vh - 60px);
    display: flex;
    align-items: center;
    padding: 20px;
    flex-wrap: wrap;
    align-content: center;
  }
}
</style>
