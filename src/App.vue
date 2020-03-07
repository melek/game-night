<template>
  <div id="app">    
    <AppHeader><code v-show="showHeaderInfo">{{ msgLog.length > 0 ? msgLog[msgLog.length - 1].msg : "" }}</code></AppHeader>
    <main>   
      <AddGame @addGame="addGame($event)" @logError="updateLog($event)"></AddGame>       
      <FetchGames @fetchGames="fetchGames($event)" @logError="updateLog($event)"></FetchGames>      
      <hr>
      <h2>Add a game night</h2>
        <label>Game Date<br><input type="date" placeholder="Game Date" v-model="newGameNight.date"></label>
        <label>Game<br>
          <select v-if="games.length > 0" v-model="newGameNight.game">
            <option v-for="(game, index) in games" :value="game" :key="index">{{ game.name }}</option>        
          </select>
          <select v-else>
            <option><em>No games to select!</em></option>
          </select>
        </label>
        <button @click="addGameNight">Schedule</button>
      <hr>      
      <h2>Game Night Schedule</h2>
        <table>
          <thead>
            <tr>
              <td>Date</td>
              <td>Game</td>
              <td>RSVPs</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(gameNight, index) in gameNights" :key="index"
            :style=" gameNight.game ? 
                { 'backgroundImage': 'linear-gradient(to right, rgba(62,14,62,0.75) 0%, rgba(62,14,62,0.95) 100%), url(' + gameNight.game.image + ')',
                      'backgroundSize': 'cover',
                      'backgroundPosition': 'center center'} : 
                {}">
              <td>{{ gameNight.date}}</td>
              <td>{{ gameNight.game ? gameNight.game.name : "No game selected" }}</td>
              <td>{{ gameNight.rsvp ? gameNight.rsvp.join(",") : "No RSVPs" }}</td>
            </tr>        
          </tbody>
        </table>
      <h2>Game List</h2>
        <table>
          <thead>
            <tr>
              <td>Game</td>
              <td>Players</td>
              <td>Time (BGG)</td>
              <td>Actions</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(game, index) in games" :key="index"
            :style=" game.image ? 
                { 'backgroundImage': 'linear-gradient(to right, rgba(62,14,62,0.75) 0%, rgba(62,14,62,0.95) 100%), url(' + game.image + ')',
                      'backgroundSize': 'cover',
                      'backgroundPosition': 'center center'} :
                {}">
              <td>
                {{ game.name }}</td>
              <td>{{ game.minPlayers != game.maxPlayers ? game.minPlayers + " - " + game.maxPlayers : game.minPlayers }}</td>
              <td>{{ game.playingTime }}</td>
              <td><span data-clickable @click="removeGame(index)">🗑️</span></td>
            </tr>
          </tbody>
        </table>        
    </main>
    <footer>      
      <p>Happiness &amp; Boardgames</p>
    </footer>
  </div>
</template>

<script>
import AppHeader from './components/AppHeader.vue'
import AddGame from './components/AddGame.vue'
import FetchGames from './components/FetchGames.vue'

export default {
  name: 'App',
  data() {
    return {
      // Feature: Info Logging (INFO)
      msgLog: [],
      debugConsoleLog: true,       
      debugHeaderLog: true,
      headerInfoDuration: 3000,
      showHeaderInfo: false,

      // Feature: Games Database (GAMES)
      games: [],      

      // Feature: Game Night Calendar Database (CALENDAR)
      gameNights: [],

      // Sub-feature: CALENDAR > Game Night Scheduling    
      newGameNight: {},

    };
  },  
  components: {
    AppHeader,
    AddGame,
    FetchGames
  },
  methods: {
    // Sub-feature: INFO > Update Info Log
    updateLog(msg, toHeader = true, toConsole = true) {
      this.msgLog.push({msg: msg, toHeader: toHeader, toConsole: toConsole});
      if(this.debugHeaderLog && toHeader) {
        this.showHeaderInfo = true;
        let vm = this;
        setTimeout( function() { vm.showHeaderInfo = false; }, this.headerInfoDuration);
      }
      if(this.debugConsoleLog && toConsole) {
        console.log(msg);
      }
    },
    // Sub-feature: GAMES > Adding Custom Games
    addGame(newGame) {      
      if(newGame.name.length > 0) {
        this.games.push(newGame);
        this.updateLog("addGame: " + newGame.name + " added!");        
      } else {
        this.updateLog("addGame: No new game to add!");
      }
    },
    // Sub-feature: GAMES > Fetching BGG Games by username
    fetchGames(bggName) {
      let vm = this;
      let theUrl = "https://bgg-json.azurewebsites.net/collection/" + bggName;
      
      let callback = function(response) {
        response = JSON.parse(response);        
        for(let index in response) {    
          if(response[index].name)      
          vm.games.push(response[index]);
        }
        vm.updateLog("fetchGames: " + response.length + " game" + ( response.length > 1 ? "s" : "" ) + " added!");
      };

      var xmlHttp = new XMLHttpRequest();      
      xmlHttp.onreadystatechange = function() { 
          if (xmlHttp.readyState == 4 && xmlHttp.status == 200)
              callback(xmlHttp.responseText);
      }
      xmlHttp.open("GET", theUrl, true); // true for asynchronous 
      xmlHttp.send(null);
  
    },
    removeGame(index) {      
      this.updateLog("removeGame: " + this.games[index].name + " (" + index + ") removed.");
      this.games.splice(index, 1);      
    },
    addGameNight() {
      if(this.newGameNight.date) {
        this.gameNights.push(this.newGameNight);
        this.updateLog("addGameNight: Scheduled " + this.newGameNight.date + ( this.newGameNight.game ? " - " + this.newGameNight.game.name : " - No game selected" ));
        this.newGameNight = {};
      } else {
        this.updateLog("addGameNight: No new game night to add!");
      }
    },
    toggleGameNightRSVP(gameNightIndex, player) {
      let rsvpIndex = this.gameNights[gameNightIndex].rsvp.indexOf(player);
      if(rsvpIndex >= 0) {
        this.updateLog("toggleGameNightRSVP: Removing RSVP from game night " + gameNightIndex + ": " + this.gameNights[gameNightIndex].rsvp.splice(rsvpIndex, 1).join(", "));
      } else {
        this.gameNights[gameNightIndex].rsvp.push(player);
      }
    },
    removeGameNight(gameNightIndex) {
      if(0 <= gameNightIndex < this.gameNights.length) {
        this.gameNights.splice(gameNightIndex, 1);
      } else {
        this.updateLog("removeGameNight: Invalid game night provided, nothing removed.");
      }
    }
  }
}
</script>

<style scoped>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

table {
  width:100%;
}

tr {
  padding:.5rem;
  border-radius:.25rem;
  background-color:#3e0e3e;
  border-bottom: 1px solid #5e2e5e;
  color:white;
  /*margin-bottom:.25rem;*/
}

[data-clickable] {
  cursor: pointer;
}
</style>
