<template>
  <div>
      <h2>Fetch Games from GBB</h2>
        <label>BGG Name<br><input type="text" v-model="bggName"></label>
        <button @click="emitFetchGames" placeholder="BGG Username">Add Games</button>        
  </div>
</template>

<script>
export default {
    data() {
        return {
            bggName: "melek"
        };
    },
    methods: {
        emitFetchGames() {
            if(this.bggName.length > 0) {
                this.$emit("fetchGames", this.bggName);                
            } else {
                this.$emit("logError", "FetchGames.emitFetchGames: No BGG name provided!");
            }
        }
    }
}
</script>

<style>

</style>
