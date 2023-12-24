<styles lang="scss" scoped>
.main-wrapper {
  max-width: 900px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 70px;

  .balloon-wrapper {
    .balloon {
      display: inline-block;
      aspect-ratio: 0.8/1;
      border-radius: 80%;
      position: relative;
      box-shadow: inset -15px -15px 8px rgba(0, 0, 0, 0.1);
      margin: 20px 30px;
      transition: transform 2s ease;
      z-index: 10;
      transition: transform 2s ease, height 2s ease;
      animation: balloon 4s ease-in-out infinite;
      transform-origin: bottom center;
    }

    @keyframes balloon {
      0%,
      100% {
        transform: translateY(0) rotate(-4deg);
      }
      50% {
        transform: translateY(-25px) rotate(4deg);
      }
    }

    .balloon:before {
      content: "▲";
      font-size: 20px;
      color: inherit;
      display: block;
      text-align: center;
      width: 100%;
      position: absolute;
      bottom: -18px;
      z-index: -100;
    }

    .shine {
      position: absolute;
      top: 20%;
      left: 20%;

      background-color: white;
      width: 25px;
      height: 33px;
      opacity: 0.5;
      z-index: 15;
      border-radius: 50%;
      transform: rotate(20deg);
      filter: blur(3px);
    }
  }

  .code-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 50px;
    max-width: 400px;
  }

  .code-hideout-5 {
    position: absolute;
    top: 12;
    color: #ddd;
  }
}
</styles>

<template>
  <v-container class="main-wrapper">
    <div class="balloon-wrapper" :style="{ color: balloon.tailColor }">
      <div
        v-show="!balloon.exploded"
        class="balloon"
        :style="{
          background: balloon.color,
          height: `${balloon.inflation * 5}px`,
        }"
      >
        <div class="shine"></div>
      </div>
    </div>
    <v-btn
      color="primary"
      icon="mdi-weather-windy"
      size="x-large"
      @click="inflateBalloon"
    ></v-btn>

    <v-row cols="12" class="d-flex justify-center align-center w-100">
      <v-col cols="4">
        <v-text-field
          v-model="codeInput"
          label=""
          prepend-inner-icon="mdi-chevron-right"
          variant="underlined"
          autofocus
          clearable
          width="'200px'"
          @keyup.enter="submitCode"
        ></v-text-field>
      </v-col>
      <v-col cols="3" class="d-flex justify-center">
        <v-btn color="primary" @click="submitCode">Submit Code</v-btn>
      </v-col>
    </v-row>

    <v-dialog v-model="showDialog" max-width="600" persistent>
      <div v-if="hasWon" class="d-flex justify-center">
        <ConfettiExplosion />
      </div>
      <v-card class="pa-4">
        <v-card-title class="text-center">{{ dialogTitle }}</v-card-title>
        <v-card-text class="text-center">
          <div v-html="dialogMessage"></div>
        </v-card-text>
        <v-card-actions class="justify-center">
          <v-btn color="secondary" @click="resetGame">{{
            isFirstRound ? "Start game!" : "Play again!"
          }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <span v-if="codeHideout == 5" class="code-hideout-5"
      >code: {{ hiddenCode }}</span
    >

    <audio ref="balloonAir" preload="none">
      <source src="/balloonAir.mp3" type="audio/mpeg" />
    </audio>

    <audio ref="pop" preload="none">
      <source src="/pop.mp3" type="audio/mpeg" />
    </audio>

    <audio ref="loser" preload="none">
      <source src="/loser.mp3" type="audio/mpeg" />
    </audio>

    <audio ref="kids" preload="none">
      <source src="/kids.mp3" type="audio/mpeg" />
    </audio>

    <audio ref="bgm" preload="none" loop>
      <source src="/bgm.mp3" type="audio/mpeg" />
    </audio>
  </v-container>
</template>

<script lang="ts">
import ConfettiExplosion from "vue-confetti-explosion";

export default {
  data() {
    return {
      codeInput: "",
      balloon: {
        color: "",
        tailColor: "",
        inflation: 50,
        exploded: false,
      },
      maxInflation: 100,
      timer: null as NodeJS.Timeout | null,
      hasWon: false,
      gameOver: false,
      isFirstRound: true,
      hiddenCode: 0,
      codeHideout: 0,
      showDialog: true,
      dialogTitle: "Welcome to this odd game!",
      dialogMessage: `
        <div class="text-left">
          Your goal is to rescue a damaged balloon by entering the correct code. However, it's not as simple as it seems! The balloon has a minimum and maximum inflation level, and if it goes beyond these limits, it will either burst or completely deflate and you'll lose.<br><br>
          <strong>Key Rules:</strong><br><br>
          <ol>
            <li>
              <strong>Balloon Inflation:</strong> Press the 'Inflate' button to gradually increase the balloon's size. Be cautious, though! The balloon will start deflating slowly as soon as you stop inflating.
            </li>
            <br>
            <li>
              <strong>Code Concealment:</strong> In each round, the correct code needed to repair the balloon is randomly hidden anywhere within the browser window, even within the DevTools. It's your task to explore and search everywhere to find it. <br><br> The code consists of a numerical sequence with up to four digits, and it will be unveiled in the following format: "code: 1234."
            </li>
            <br>
            <li>
              <strong>Challenge Management:</strong> Balancing the balloon's inflation level while searching for the hidden code adds an extra layer of challenge. Keep an eye on the balloon's size and make strategic decisions to avoid it bursting or completely deflating.
            </li>
            <br>
            <li>
              <strong>Winning the Game:</strong> Successfully enter the correct code before the balloon reaches its maximum inflation or deflates completely. If you manage to repair the balloon, you win the game!
            </li>
          </ol>
          <br>
          It's a race against time and a test of your coding skills and observational abilities. Are you ready to take on the Balloon Repair Challenge and save the balloon from bursting? Good luck!
        </div
      `,
    };
  },

  computed: {
    inflation(): HTMLAudioElement {
      return this.$refs.balloonAir as HTMLAudioElement;
    },

    pop(): HTMLAudioElement {
      return this.$refs.pop as HTMLAudioElement;
    },

    loser(): HTMLAudioElement {
      return this.$refs.loser as HTMLAudioElement;
    },

    kids(): HTMLAudioElement {
      return this.$refs.kids as HTMLAudioElement;
    },

    bgm(): HTMLAudioElement {
      return this.$refs.bgm as HTMLAudioElement;
    },
  },

  methods: {
    inflateBalloon() {
      if (this.gameOver) {
        return;
      }

      this.balloon.inflation += 10;

      if (this.balloon.inflation > this.maxInflation) {
        this.explodeBalloon();
        return;
      }

      this.loadSound("balloonAir");
    },

    explodeBalloon() {
      clearInterval(this.timer as NodeJS.Timeout);
      this.balloon.exploded = true;
      this.gameOver = true;
      this.inflation.pause();
      this.loadSound("pop");

      this.dialogTitle = "You lose!";
      this.dialogMessage =
        "You inflated the balloon too much and it exploded! 💥";
      this.showDialog = true;
    },

    deflateBalloon() {
      if (!this.gameOver && !this.isFirstRound) {
        this.balloon.inflation -= 10;

        if (this.balloon.inflation <= 10) {
          clearInterval(this.timer as NodeJS.Timeout);
          this.gameOver = true;
          this.inflation.pause();

          this.loadSound("loser");
          this.dialogTitle = "You lose!";
          this.dialogMessage = "The balloon fully deflated 🥴";
          this.showDialog = true;
        }
      }
    },

    checkCode() {
      if (isNaN(+this.codeInput)) return;

      return +this.codeInput === this.hiddenCode;
    },

    submitCode() {
      if (this.checkCode() && !this.gameOver) {
        this.inflation.pause();
        this.loadSound("kids");
        clearInterval(this.timer as NodeJS.Timeout);
        this.hasWon = true;
        this.gameOver = true;
        this.dialogTitle = "You win!";
        this.dialogMessage = "Congratulations! You entered the correct code 🎉";
        this.showDialog = true;
      }
    },

    generateRandomCode() {
      const randomCode = Math.floor(Math.random() * 10000);
      this.hiddenCode = randomCode;

      return randomCode;
    },

    generateCodeHideout() {
      const hideoutCode = Math.floor(Math.random() * 5) + 1;
      const code = `code: ${this.hiddenCode}`;
      this.codeHideout = hideoutCode;

      switch (hideoutCode) {
        case 1:
          console.log(code);
          break;
        case 2:
          document.title = code;
          break;
        case 3:
          localStorage.setItem("code", code);
          break;
        case 4:
          const appContainer = document.querySelector("#app");
          appContainer?.classList.add(`code:${this.hiddenCode}`);
          break;
      }
    },

    loadSound(sound: string) {
      switch (sound) {
        case "balloonAir":
          this.inflation.load();
          this.inflation.volume = 0.5;
          this.inflation.play();
          break;
        case "pop":
          this.pop.load();
          this.pop.volume = 1;
          this.pop.play();
          break;
        case "loser":
          this.loser.load();
          this.loser.volume = 0.5;
          this.loser.play();
          break;
        case "kids":
          this.kids.load();
          this.kids.volume = 1;
          this.kids.play();
          break;
        case "bgm":
          this.bgm.load();
          this.bgm.volume = 0.2;
          this.bgm.play();
          break;
      }
    },

    startDeflationTimer() {
      this.timer = setInterval(() => this.deflateBalloon(), 2000);
    },

    setRandomBalloonColor() {
      const randomColor = Math.floor(Math.random() * 255);
      this.balloon.color = `hsl(${randomColor}, 50%, 65%)`;
      this.balloon.tailColor = `hsl(${randomColor}, 41%, 60%)`;
    },

    resetCodeHideouts() {
      console.clear();
      document.title = "Lo Que El Viento Se Petó";
      localStorage.removeItem("code");
      const appContainer = document.querySelector("#app");
      appContainer?.classList.remove(`code:${this.hiddenCode}`);
    },

    resetGame() {
      this.balloon.exploded = false;
      this.gameOver = false;
      this.showDialog = false;
      this.balloon.inflation = 50;
      this.codeInput = "";
      this.hasWon = false;

      if (this.isFirstRound) {
        this.loadSound("bgm");
      }

      this.isFirstRound = false;
      this.resetCodeHideouts();
      this.setRandomBalloonColor();
      this.generateRandomCode();
      this.generateCodeHideout();
      this.loadSound("balloonAir");
      this.startDeflationTimer();
    },
  },

  mounted() {
    this.generateRandomCode();
    this.setRandomBalloonColor();
  },
};
</script>
