<template>
  <div>
    <header>
      <h1>Monster Slayer</h1>
    </header>
    <div id="game">
      <section id="monster" class="container">
        <h2>Monster Health</h2>
        <div class="healthbar">
          <div class="healthbar__value" :style="monsterBarStyles">
            {{ monsterHealth }}
          </div>
        </div>
      </section>
      <section id="player" class="container">
        <h2>Your Health</h2>
        <div class="healthbar">
          <div class="healthbar__value" :style="playerBarStyles">
            {{ playerHealth }}
          </div>
        </div>
      </section>
      <div class="container" v-if="winner">
        <h2>Game over!</h2>
        <h3 v-if="winner == 'Monster'">You Lost!</h3>
        <h2 v-if="winner == 'Player'">You won!</h2>
        <h3 v-else>Draw!</h3>
        <button @click="startGame">Restart game</button>
      </div>
      <section id="controls" v-else>
        <button @click="attackMonster">ATTACK</button>
        <button :disabled="mayUseSpecialAttack" @click="playerSpecialAttack">
          SPECIAL ATTACK
        </button>
        <button :disabled="mayUseHeal" @click="healPlayer">HEAL</button>
        <button @click="surrender">SURRENDER</button>
      </section>
      <section id="log" class="container">
        <h2>Battle Log</h2>
        <h1>Counter = {{ healCounter }}</h1>
        <h1>Round = {{ currentRound }}</h1>
        <ul>
          <li v-for="log in logMessage" :key="log">
            <span
              :class="{
                'log--player': log.actionBy === 'player',
                'log--monster': log.actionBy === 'monster',
              }"
              >{{ log.actionBy === "player" ? "Player" : "Monster" }}</span
            >
            <span v-if="log.actionType === 'healed'">
              Heals himself for
              <span class="log--heal">{{ log.actionValue }}</span></span
            >
            <span v-else>
              attacks and deals
              <span class="log--damage"> {{ log.actionValue }}</span>
            </span>
          </li>
        </ul>
      </section>
    </div>
  </div>
</template>

<script>
function getRandomValue(min, max) {
  return Math.floor(Math.random() * (max - min) + min);
}

export default {
  name: "Game",
  props: {
    msg: String,
  },

  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      currentRound: 0,
      winner: 0,
      logMessage: [],
      healCounter: 0,
    };
  },

  computed: {
    monsterBarStyles() {
      if (this.monsterHealth < 0) {
        return { width: "0%" };
      }
      return { width: this.monsterHealth + "%" };
    },
    playerBarStyles() {
      if (this.playerHealth < 0) {
        return { width: "0%" };
      }
      return { width: this.playerHealth + "%" };
    },

    mayUseSpecialAttack() {
      return this.currentRound % 3 !== 0;
    },

    mayUseHeal() {
      return this.healCounter >= 1;
    },
  },

  methods: {
    currentRoundCheck() {
      if (this.currentRound % 2 == 0 && this.healCounter >= 2) {
        this.healCounter = 0;
      }
    },
    attackMonster() {
      const attatckValue = getRandomValue(5, 5);
      this.monsterHealth -= attatckValue;
      this.currentRound++;
      this.attackPlayer();
      this.healCounter++;
      this.addLogMessage("player", "attack", attatckValue);
      this.currentRoundCheck();
    },
    attackPlayer() {
      const attatckValue = getRandomValue(15, 5);
      this.playerHealth -= attatckValue;
      this.addLogMessage("monster", "attack", attatckValue);
      this.currentRoundCheck();
    },
    playerSpecialAttack() {
      const attatckValue = getRandomValue(10, 25);
      this.monsterHealth -= attatckValue;
      this.attackPlayer();
      this.currentRound++;
      this.addLogMessage("player", "special attack", attatckValue);
      this.currentRoundCheck();
    },

    healPlayer() {
      const healValue = getRandomValue(1, 6);
      this.healCounter++;
      if (this.playerHealth + healValue > 100) {
        this.playerHealth = 100;
        this.addLogMessage("player", "healed", healValue);
      } else {
        this.playerHealth += healValue;
        this.addLogMessage("player", "healed", healValue);
        this.currentRoundCheck();
      }
    },
    startGame() {
      (this.playerHealth = 100),
        (this.monsterHealth = 100),
        (this.currentRound = 0),
        (this.winner = 0);
      this.logMessage = [];
      this.healCounter = 0;
    },
    surrender() {
      this.winner = "monster";
      this.addLogMessage("player", "surrended", NaN);
    },

    addLogMessage(who, what, value) {
      this.logMessage.unshift({
        actionBy: who,
        actionType: what,
        actionValue: value,
      });
    },
  },

  watch: {
    playerHealth(value) {
      if (value <= 0 && this.monsterHealth <= 0) {
        this.winner = "Draw";
      } else if (value <= 0) {
        this.winner = "Monster";
      }

      if (this.playerHealth <= 0) {
        this.playerHealth = 0;
      }
    },
    monsterHealth(value) {
      if (value <= 0 && this.playerHealth <= 0) {
        this.winner = "Draw";
      } else if (value <= 0) {
        this.winner = "Player";
      }

      if (this.monsterHealth <= 0) {
        this.monsterHealth = 0;
      }
    },

    healCounter() {
      if (this.healCounter >= 2) {
        this.healCounter = 0;
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
* {
  box-sizing: border-box;
}

html {
  font-family: "Jost", sans-serif;
}

body {
  margin: 0;
}

header {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 0.5rem;
  background-color: #880017;
  color: white;
  text-align: center;
  margin-bottom: 2rem;
}

section {
  width: 90%;
  max-width: 40rem;
  margin: auto;
}

.who {
  font-size: 24px;
  text-transform: capitalize;
  color: green;
}

.what {
  font-size: 21px;
}

.value {
  font-size: 20px;
  color: red;
}

.healthbar {
  width: 100%;
  height: 40px;
  border: 1px solid #575757;
  margin: 1rem 0;
  background: #d42323;
}

.healthbar__value {
  background-color: #00a876;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  font-weight: bold;
  max-width: 100%;
  transition: 0.3s ease all;
}

.container {
  text-align: center;
  padding: 0.5rem;
  margin: 1rem auto;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;
}

#monster h2,
#player h2 {
  margin: 0.25rem;
}

#controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

button {
  font: inherit;
  border: 1px solid #88005b;
  background-color: #88005b;
  color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  margin: 1rem;
  width: 12rem;
  cursor: pointer;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.26);
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background-color: #af0a78;
  border-color: #af0a78;
  box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
}

button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  box-shadow: none;
  color: #3f3f3f;
  cursor: not-allowed;
}

#log ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#log li {
  margin: 0.5rem 0;
}

.log--player {
  color: #7700ff;
}

.log--monster {
  color: #da8d00;
}

.log--damage {
  color: red;
}

.log--heal {
  color: green;
}
</style>
