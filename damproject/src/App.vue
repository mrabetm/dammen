<script>
import Damboard from "./components/Damboard.vue"
import Statistics from "./components/Statistics.vue"

export default {
  components: { Damboard, Statistics },
  data() {
    return {
      player_1: {
        id: 1,
        name: "John Doe",
        team: "white",
        score: 0,
      },
      player_2: {
        id: 2,
        name: "John Doe 2",
        team: "black",
        score: 0,
      },
    }
  },
}
</script>

<template>
  <div>
    <Damboard :player_1="this.player_1" :player_2="this.player_2"></Damboard>
  </div>
</template>

<style scoped></style>
