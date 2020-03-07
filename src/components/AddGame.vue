<template>
<div>
    <h2>Add a custom game</h2>
        <label>Game Name<br><input type="text" v-model="newGame.name"></label>
        <label>Min. Players<br><input type="text" v-model="newGame.minPlayers"></label>
        <label>Max. Players<br><input type="text" v-model="newGame.maxPlayers"></label>
        <label>Time<br><input type="text" v-model="newGame.playingTime"></label>
        <button @click="emitGame" placeholder="Game Name">Add Game</button>    
</div>
</template>

<script>
export default {
    data() {
        return {
            newGame: {}
        };
    },
    methods: {
        emitGame() {
            if(this.newGame.name.length > 0) {
                this.$emit("addGame", this.newGame);                
                this.newGame = {};
                return;
            }
            this.$emit("logError", "AddGame.emitGame: No game name specified!");
        }
    }
}
</script>

<style>

</style>
