<template>
  <div class="simon" :class="!main.isPanelsSequenceCurrentlyFiring ? 'allow' : 'restrict'">
    <div class="outer_circle">
      <template v-for="panel of simon.panels" :key="panel">
        <Panel v-bind:panel="panel" v-on:panelclick="onPanelClick"/>
      </template>        
      <div class="inner_circle">
        <span>{{simon.panelsSequence.length}}</span>
      </div>
    </div>
    <div class="interface">
      <button v-on:click="onStart" :disabled="!main.isPlayerAllowedToClickStartButton">Start</button>
      <select name="" id="" v-model="simon.difficultyLevel" :disabled="!main.isPlayerAllowedToChangeDifficultyLevel">
        <option value="dlEasy">{{simon.difficultyLevels.get('dlEasy').name}}</option>
        <option value="dlNormal">{{simon.difficultyLevels.get('dlNormal').name}}</option>
        <option value="dlHard">{{simon.difficultyLevels.get('dlHard').name}}</option>
      </select>
    </div>
  </div>
</template>

<script>
import Panel from "@/components/Panel.vue";

export default {
  name: "Simon",
  components: {
    Panel
  },
  data() {
      return {
          simon: {
              panels: [
                { id: 1, fired: false, sound: new Audio(require('@/assets/simonSound1.mp3')), failed: false },
                { id: 2, fired: false, sound: new Audio(require('@/assets/simonSound2.mp3')), failed: false },
                { id: 3, fired: false, sound: new Audio(require('@/assets/simonSound3.mp3')), failed: false },
                { id: 4, fired: false, sound: new Audio(require('@/assets/simonSound4.mp3')), failed: false }
              ],
              canPlayerClickPanel: false,
              difficultyLevels: new Map([
                ['dlEasy', { name: "Легко", panelFiringTime: 1300, panelCoolingTime: 500, timeGapBetweenRounds: 800 }],
                ['dlNormal', { name: "Нормально", panelFiringTime: 800, panelCoolingTime: 300, timeGapBetweenRounds: 600 }],
                ['dlHard', { name: "Тяжело", panelFiringTime: 600, panelCoolingTime: 100, timeGapBetweenRounds: 400 }]
              ]),
              difficultyLevel: 'dlEasy',
              panelsSequence: [],
              playerSequence: [],
              failSound: new Audio(require('@/assets/failSound.mp3'))
          },
          main: {
            interval: null,
            playerClickedPanelID: 0,
            isPlayerClickedPanel: false,
            isPlayerAllowedToClickPanel: false,
            isPlayerAllowedToClickStartButton: true,
            isPlayerAllowedToChangeDifficultyLevel: true,
            isPanelsSequenceCurrentlyFiring: false,
            isMainLoopCurrentlyExecuting: false,
            isMainLoopCurrentlyProcessingPlayeClick: false,
            isTimeToFirePanelsSequence: false,
            isTimeToStartNewRound: false,
            isTimeToStartNewGame: false,
            FPS: 60,
            playerClickSequence: []
          }
      }
  },
  methods: {
    onPanelClick(panelID) {
      if (!this.main.isPlayerAllowedToClickPanel) return;
      this.main.isPlayerClickedPanel = true;
      this.main.playerClickedPanelID = panelID;
    },
    onStart() {
      if (!this.main.isPlayerAllowedToClickStartButton) return;
      this.main.isPlayerAllowedToClickStartButton = false;

      this.main.isTimeToStartNewGame = true;
      if (this.main.interval == null) {
        this.main.interval = setInterval(this.mainLoop, 1000 / this.main.FPS);
      }
    },
    async mainLoop() {
      if (this.main.isMainLoopCurrentlyExecuting) {
        return;
      } else {
        this.main.isMainLoopCurrentlyExecuting = true;
      }
      
      if (this.main.isPanelsSequenceCurrentlyFiring || this.main.isMainLoopCurrentlyProcessingPlayeClick) {
        return;
      }

      if (this.main.isTimeToStartNewGame) {
        this.main.isTimeToStartNewRound = true;
        this.simon.panelsSequence = [];
        this.main.isPlayerAllowedToChangeDifficultyLevel = false;
        this.main.isTimeToStartNewGame = false;
      }

      if (this.main.isTimeToStartNewRound) {
        this.simon.panelsSequence.push(Math.floor(Math.random() * 4) + 1);
        this.simon.playerSequence = this.simon.panelsSequence.slice(0);
        this.main.isTimeToStartNewRound = false;
        this.main.isTimeToFirePanelsSequence = true;
      }

      if (this.main.isTimeToFirePanelsSequence) {
        this.main.isPanelsSequenceCurrentlyFiring = true;
        for (let ix = 0; ix < this.simon.panelsSequence.length; ix++) {
          this.simon.panels[this.simon.panelsSequence[ix] - 1].fired = true;
          this.simon.panels[this.simon.panelsSequence[ix] - 1].sound.play();
          await (new Promise(resolve => setTimeout(resolve, this.simon.difficultyLevels.get(this.simon.difficultyLevel).panelFiringTime)));
          this.simon.panels[this.simon.panelsSequence[ix] - 1].fired = false;
          await (new Promise(resolve => setTimeout(resolve, this.simon.difficultyLevels.get(this.simon.difficultyLevel).panelCoolingTime)));
        }
        this.main.isPanelsSequenceCurrentlyFiring = false;
        this.main.isTimeToFirePanelsSequence = false;
        this.main.isPlayerAllowedToClickPanel = true;   
      }

      if (this.main.isPlayerClickedPanel) {
        this.main.isPlayerClickedPanel = false;

        if (this.main.playerClickedPanelID != this.simon.playerSequence.shift()) {
          this.main.isPlayerAllowedToClickPanel = false;
          this.main.isPlayerAllowedToClickStartButton = false;
          this.simon.panels[this.main.playerClickedPanelID - 1].failed = true;
          this.simon.failSound.play();
          await (new Promise(resolve => setTimeout(resolve, 5000)));
          this.simon.panels[this.main.playerClickedPanelID - 1].failed = false;
          this.main.isPlayerAllowedToClickStartButton = true;
          this.main.isPlayerAllowedToChangeDifficultyLevel = true;

          this.main.isMainLoopCurrentlyExecuting = false;
          return;
        }

        this.main.isPlayerAllowedToClickPanel = false;
        this.main.isPlayerAllowedToClickStartButton = false;

        this.simon.panels[this.main.playerClickedPanelID - 1].fired = true;
        this.simon.panels[this.main.playerClickedPanelID - 1].sound.play();
        await (new Promise(resolve => setTimeout(resolve, this.simon.difficultyLevels.get(this.simon.difficultyLevel).panelFiringTime)));
        this.simon.panels[this.main.playerClickedPanelID - 1].fired = false;
        this.main.isPlayerAllowedToClickPanel = true;
        await (new Promise(resolve => setTimeout(resolve, this.simon.difficultyLevels.get(this.simon.difficultyLevel).panelCoolingTime)));

        if (this.simon.playerSequence.length == 0) {
          await (new Promise(resolve => setTimeout(resolve, this.simon.difficultyLevels.get(this.simon.difficultyLevel).timeGapBetweenRounds)));
          this.main.isTimeToStartNewRound = true;
          this.main.isPlayerAllowedToClickPanel = false;
        } else {
          this.main.isPlayerAllowedToClickPanel = true;
        }

        this.main.isMainLoopCurrentlyExecuting = false;

      }
      this.main.isMainLoopCurrentlyExecuting = false;
    }    
  }
};
</script>

<style scoped>
.simon {
  padding: .5rem 0;
  width: 100%;
  height: 100%;
  background: rgb(255, 255, 255);
}

.outer_circle {
  background: #385a94;
  border-radius: 100%;
  height: 420px;
  width: 420px;
  position: relative;
  border-style: solid;
  border-width: 10px;
  margin: auto;
  margin-top: 60px;
  overflow: hidden;
}

.inner_circle {
  position: absolute;
  background: rgb(182, 170, 170);
  border-radius: 100%;
  height: 200px;
  width: 200px;
  border-style: solid;
  border-width: 10px;
  top: 50%;
  left: 50%;
  margin: -100px 0px 0px -100px;
  box-sizing: border-box;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;

  display: flex;
  justify-content: center;
  align-items: center;

  font-size: 5rem;
}

.interface {
  margin: 2rem auto;
  padding: 1rem;

  display: flex;
  justify-content: center;
  align-items: center;
}

button {
  padding: .5rem 2rem;
  font-size: 1.5rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 2px solid rgb(47, 69, 90);
  border-radius: 5px;
  background-color: rgb(128, 128, 128);
  color: rgb(44, 62, 80);
  cursor: pointer;
}
button:disabled {
  border: 2px solid rgba(44, 62, 80, 0.5);
  background-color: rgba(128, 128, 128, .5);
  color: rgba(44, 62, 80, 0.7);
  cursor: not-allowed;
}

select {
  margin-left: 1rem;
  padding: .33rem 1rem;
  font-size: 1.5rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 2px solid rgb(44, 62, 80);
  border-radius: 5px;
  background-color: rgb(128, 128, 128);
  color: rgb(44, 62, 80);
  cursor: pointer;
}
</style>
