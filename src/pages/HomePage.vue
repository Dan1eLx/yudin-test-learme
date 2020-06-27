<template lang="pug">
  Game
</template>

<script>
  import Game from '@/components/Game';

  export default {
      name: "HomePage",
      metaInfo() {
        return {
          title: 'Игра'
        }
      },
      components: {
        Game
      },
      
  }
</script>

<style lang="sass">
</style>
