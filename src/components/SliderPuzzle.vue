<template>
  <div class="puzzle-wrapper" :class="{ 'is-paused': isPaused }">

    <div class="top-bar">
      <div class="move-counter">
        🔀 Moves: <strong>{{ moveCount }}</strong>
      </div>
      <div class="music-bar">
        <span class="music-label">🎵 Music</span>
        <button
          class="music-toggle"
          :class="{ 'music-off': !bgMusicEnabled }"
          @click="toggleBgMusic"
        >
          {{ bgMusicEnabled ? '🔊 On' : '🔇 Off' }}
        </button>
      </div>
    </div>

    <h1 class="game-title">🎮 Swap the Images to Win</h1>
    
    <div class="mode-indicator-zone">
      Current Mode: <span class="mode-badge" :class="puzzleId">{{ formatModeName }}</span>
    </div>

    <div class="controls">
      <button class="btn-start" @click="start">▶ Start Game</button>
      <button
        v-if="isPlaying && !hasWon"
        class="btn-pause"
        :class="{ 'is-resume': isPaused }"
        @click="isPaused ? resume() : pause()"
      >
        {{ isPaused ? '▶ Resume' : '⏸ Pause' }}
      </button>
      <button class="btn-quit" @click="stop">✖ Quit</button>
    </div>

    <div class="timer-bar">
      <span class="timer-icon">⏱</span>
      <span class="timer-value">{{ elapsedTime }}</span>
      <span v-if="isPlaying && !isPaused && !hasWon" class="timer-live-dot"></span>
    </div>

    <transition name="pause-fade">
      <div v-if="isPaused" class="paused-overlay" @click="resume">
        <div class="paused-content">
          <div class="paused-icon">⏸</div>
          <div class="paused-title">PAUSED</div>
          <p class="paused-hint">tap anywhere to resume game</p>
          <button class="resume-btn" @click.stop="resume">▶ Resume Game</button>
        </div>
      </div>
    </transition>

    <div class="puzzle-grid-canvas-frame" :class="{ 'grid-locked': !isPlaying || isPaused || hasWon }">
      <div
        class="tile-panel"
        v-for="(s, index) of shuffledPuzzleArray"
        :key="s"
        :class="{
          'tile-selected':  indexesToSwap.includes(index),
          'tile-swapped':   swappedIndexes.includes(index),
          'tile-locked':    !isPlaying || isPaused || hasWon
        }"
        @click="swap(index)"
      >
        <img :src="require(`../assets/${puzzleId}/${s}`)" :alt="'Tile ' + (index+1)" />
        <div v-if="indexesToSwap.includes(index)" class="tile-select-ring"></div>
      </div>
    </div>

    <transition name="buzzer-fade">
      <div v-if="showBuzzer" class="golden-buzzer-overlay" @click="dismissBuzzer">
        
        <div v-for="b in 70" :key="'buzz'+b" class="rain-buzzer" :style="buzzerRainStyle(b)">⭐</div>
        <div v-for="n in 120" :key="'p'+n" class="confetti-particle" :style="confettiStyle(n)"></div>

        <div class="buzzer-content-centered" @click.stop>
          <div class="buzzer-trophy">🏆</div>
          <div class="win-title">YOU WON!</div>
          <div class="win-subtitle">🎉 Puzzle Solved Successfully! 🎉</div>
          
          <div class="win-stats">
            <div class="win-stat-item">⏱ Time Taken: <strong>{{ elapsedTime }}</strong></div>
            <div class="win-stat-item">🔀 Total Moves: <strong>{{ moveCount }} moves</strong></div>
          </div>

          <div class="win-action-buttons">
            <button class="play-again-btn" @click="start">🔄 Play Again</button>
            <button class="next-level-btn" @click="goToNextLevel">⏭ Next Level</button>
          </div>
          <p class="dismiss-hint">tap anywhere outside this card to review your completed board!</p>
        </div>

      </div>
    </transition>

  </div>
</template>

<script>
import moment from "moment";

const correctPuzzleArray = [
  "image_part_001.jpg",
  "image_part_002.jpg",
  "image_part_003.jpg",
  "image_part_004.jpg",
  "image_part_005.jpg",
  "image_part_006.jpg",
  "image_part_007.jpg",
  "image_part_008.jpg",
  "image_part_009.jpg",
];

const Audio = window.Audio;
let bgAudio    = null;   
let swapAudio  = null;   
let winAudio   = null;   

function initAudio() {
  try {
    if (!bgAudio) {
      // Clean, verified path targets matching your assets folder structure exactly
      bgAudio   = new Audio(require("../assets/sounds/background-music.mp3"));
      swapAudio = new Audio(require("../assets/sounds/tile-swap.wav"));
      winAudio  = new Audio(require("../assets/sounds/win-sound.wav"));
      
      bgAudio.loop     = true;
      bgAudio.volume   = 0.25; 
      swapAudio.volume = 0.85;
      winAudio.volume  = 0.95;
    }
  } catch (e) {
    console.warn("Audio Context setup deferred or asset missing:", e);
  }
}

function playBgMusic(isGameplay = false) {
  initAudio();
  if (bgAudio) {
    bgAudio.volume = isGameplay ? 0.06 : 0.25;
    if (bgAudio.paused) {
      bgAudio.play().catch((err) => console.log("Audio play blocked by browser policy until interaction:", err));
    }
  }
}

function pauseBgMusic() {
  if (bgAudio && !bgAudio.paused) bgAudio.pause();
}

function stopBgMusic() {
  if (bgAudio) {
    bgAudio.pause();
    bgAudio.currentTime = 0;
  }
}

function playSwapSound() {
  if (swapAudio) {
    swapAudio.currentTime = 0;
    swapAudio.play().catch(() => {});
  }
}

function playSelectSound() {
  if (swapAudio) {
    const clone = swapAudio.cloneNode();
    clone.volume = 0.40;
    clone.playbackRate = 1.4;
    clone.play().catch(() => {});
  }
}

function playWinSound() {
  stopBgMusic();
  if (winAudio) {
    winAudio.currentTime = 0;
    winAudio.play().catch(() => {});
  }
}

function rand(min, max) {
  return Math.random() * (max - min) + min;
}

export default {
  name: "SliderPuzzle",
  props: {
    puzzleId: {
      type: String,
      default: "cut-easy",
    },
  },
  data() {
    return {
      correctPuzzleArray,
      shuffledPuzzleArray: [],
      indexesToSwap:   [],
      swappedIndexes:  [],
      timer:           undefined,
      startDateTime:   new Date(),
      currentDateTime: new Date(),
      hasWon:          false,
      showBuzzer:      false,
      bgMusicEnabled:  true,
      isPlaying:       false,
      isPaused:        false,
      pausedDiff:      0,
      moveCount:       0,
    };
  },
  watch: {
    puzzleId() {
      this.stop();
      this.shufflePuzzle();
    }
  },
  created() {
    this.shufflePuzzle();
    // Attempt initialization safely during hook lifecycle
    try {
      initAudio();
    } catch(e) {
      console.warn("Audio initialization postponed:", e);
    }
  },
  mounted() {
    const unlockAudio = () => {
      if (this.bgMusicEnabled && !this.hasWon && !this.isPaused) {
        playBgMusic(this.isPlaying);
      }
    };
    window.addEventListener("click", unlockAudio, { once: true });
    window.addEventListener("touchstart", unlockAudio, { once: true });
    
    this.$nextTick(() => { 
      unlockAudio(); 
    });
  },
  beforeDestroy() {
    stopBgMusic();
    if (this.timer) clearInterval(this.timer);
  },
  computed: {
    isWinning() {
      if (this.shuffledPuzzleArray.length === 0) return false;
      for (let i = 0; i < this.correctPuzzleArray.length; i++) {
        if (this.correctPuzzleArray[i] !== this.shuffledPuzzleArray[i]) return false;
      }
      return true;
    },
    elapsedDiff() {
      return moment(this.currentDateTime).diff(moment(this.startDateTime));
    },
    elapsedTime() {
      return moment.utc(this.elapsedDiff).format("HH:mm:ss");
    },
    formatModeName() {
      if (this.puzzleId === "cut-medium") return "Medium Level";
      if (this.puzzleId === "cut-hard") return "Hard Level";
      return "Easy Level";
    }
  },
  methods: {
    shufflePuzzle() {
      let shuffled;
      let isMatchesCorrect = true;
      while (isMatchesCorrect) {
        shuffled = [...this.correctPuzzleArray].sort(() => Math.random() - 0.5);
        isMatchesCorrect = shuffled.every((val, i) => val === this.correctPuzzleArray[i]);
      }
      this.shuffledPuzzleArray = shuffled;
    },
    swap(index) {
      if (!this.timer || this.hasWon || this.isPaused) return;

      if (this.indexesToSwap.length === 0) {
        this.indexesToSwap.push(index);
        playSelectSound();
        return;
      }
      if (this.indexesToSwap[0] === index) {
        this.indexesToSwap = [];
        return;
      }

      this.indexesToSwap.push(index);
      const [i1, i2] = this.indexesToSwap;
      const temp = this.shuffledPuzzleArray[i1];
      this.$set(this.shuffledPuzzleArray, i1, this.shuffledPuzzleArray[i2]);
      this.$set(this.shuffledPuzzleArray, i2, temp);

      this.swappedIndexes = [i1, i2];
      setTimeout(() => { this.swappedIndexes = []; }, 450);

      playSwapSound();
      this.moveCount++;
      this.indexesToSwap = [];

      if (this.isWinning) {
        this.hasWon = true;
        this.showBuzzer = true; 
        clearInterval(this.timer);
        this.recordSpeedRecords();
        playWinSound();
      }
    },
    dismissBuzzer() {
      this.showBuzzer = false;
      if (this.bgMusicEnabled) playBgMusic(this.isPlaying);
    },
    goToNextLevel() {
      let nextLevel = "cut-easy";
      if (this.puzzleId === "cut-easy") nextLevel = "cut-medium";
      else if (this.puzzleId === "cut-medium") nextLevel = "cut-hard";
      else if (this.puzzleId === "cut-hard") nextLevel = "cut-easy";

      this.showBuzzer = false;
      this.$emit("next-level", nextLevel);
    },
    start() {
      initAudio();
      this.hasWon        = false;
      this.showBuzzer    = false;
      this.isPlaying     = true;
      this.isPaused      = false;
      this.pausedDiff    = 0;
      this.moveCount     = 0;
      this.indexesToSwap  = [];
      this.swappedIndexes = [];
      this.resetTime();
      this.shufflePuzzle();
      if (this.timer) clearInterval(this.timer);
      this.timer = setInterval(() => {
        this.currentDateTime = new Date();
      }, 1000);
      
      if (this.bgMusicEnabled) playBgMusic(true);
    },
    stop() {
      clearInterval(this.timer);
      this.timer      = undefined;
      this.isPlaying  = false;
      this.isPaused   = false;
      this.pausedDiff = 0;
      this.resetTime();
      
      if (this.bgMusicEnabled) playBgMusic(false);
    },
    pause() {
      if (!this.isPlaying || this.isPaused || this.hasWon) return;
      this.isPaused   = true;
      this.pausedDiff = this.elapsedDiff;
      clearInterval(this.timer);
      this.timer = undefined;
      pauseBgMusic();
    },
    resume() {
      if (!this.isPaused) return;
      this.isPaused        = false;
      this.startDateTime   = new Date(Date.now() - this.pausedDiff);
      this.currentDateTime = new Date();
      this.timer = setInterval(() => {
        this.currentDateTime = new Date();
      }, 1000);
      if (this.bgMusicEnabled) playBgMusic(true);
    },
    resetTime() {
      this.startDateTime   = new Date();
      this.currentDateTime = new Date();
    },
    toggleBgMusic() {
      this.bgMusicEnabled = !this.bgMusicEnabled;
      if (!this.bgMusicEnabled) {
        pauseBgMusic();
      } else {
        initAudio();
        playBgMusic(this.isPlaying);
      }
    },
    recordSpeedRecords() {
      let records = JSON.parse(localStorage.getItem("records")) || [];
      const { elapsedTime, elapsedDiff, moveCount } = this;
      records.push({ elapsedTime, elapsedDiff, moveCount });
      const sortedRecords = records.sort((a, b) => a.elapsedDiff - b.elapsedDiff).slice(0, 10);
      localStorage.setItem("records", JSON.stringify(sortedRecords));
    },
    confettiStyle(n) {
      return {
        left:               `${rand(0, 100)}%`,
        top:               `${rand(-15, 20)}%`,
        width:             `${rand(5, 15)}px`,
        height:            `${rand(5, 25)}px`,
        background:        ["#FFD700","#FFC200","#FFEC6E","#FFF4A3","#FF9900","#FFAA00","#ffffff"][n % 7],
        transform:         `rotate(${rand(0, 360)}deg)`,
        animationDelay:    `${rand(0, 2)}s`,
        animationDuration: `${rand(1.5, 3.5)}s`,
      };
    },
    buzzerRainStyle(b) {
      return {
        left:              `${((b - 1) * 1.67 + rand(0, 1.5)) % 98}%`,
        top:               `-${rand(30, 80)}px`,
        fontSize:          `${rand(28, 65)}px`,
        animationDelay:    `${rand(0, 2.5)}s`,
        animationDuration: `${rand(1.8, 4)}s`,
        '--spin':          `${rand(180, 720) * (b % 2 === 0 ? 1 : -1)}deg`,
      };
    },
  },
};
</script>

<style scoped>
.puzzle-wrapper {
  position: relative;
  border-radius: 20px;
  padding: 0 0 20px;
}
.puzzle-wrapper.is-paused .puzzle-grid-canvas-frame {
  filter: blur(5px) brightness(0.55);
  pointer-events: none;
}

/* TOP CONTROLS BAR */
.top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 4px;
}
.move-counter {
  font-size: 0.95rem;
  color: #2d6a4f;
  font-weight: 600;
  background: rgba(255,255,255,0.45);
  padding: 6px 14px;
  border-radius: 20px;
}
.music-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 6px 18px;
  background: rgba(255,255,255,0.45);
  border-radius: 30px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.04);
}
.music-label {
  font-size: 0.88rem;
  font-weight: 700;
  color: #1e3322;
}
.music-toggle {
  min-width: 76px !important;
  padding: 6px 14px !important;
  font-size: 0.85rem !important;
  font-weight: 800 !important;
  color: #fff !important;
  background: #2d6a4f !important;
  border-radius: 20px !important;
  cursor: pointer;
  border: none !important;
  transition: background-color 0.2s ease;
}
.music-toggle.music-off {
  background: #8c3c3c !important;
}

.game-title {
  font-size: 2.1rem;
  font-weight: 900;
  margin: 10px 0 6px;
  color: #112216;
  letter-spacing: -0.02em;
}

/* MODE INDICATOR UI STYLES */
.mode-indicator-zone {
  font-size: 1rem;
  font-weight: 600;
  color: #4a5d4e;
  margin-bottom: 20px;
}
.mode-badge {
  display: inline-block;
  padding: 4px 14px;
  font-weight: 800;
  font-size: 0.88rem;
  border-radius: 30px;
  text-transform: uppercase;
  margin-left: 4px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}
.mode-badge.cut-easy {
  background: #e3f2e6;
  color: #2d6a4f;
  border: 1px solid #ccead4;
}
.mode-badge.cut-medium {
  background: #fff3cd;
  color: #856404;
  border: 1px solid #ffeeba;
}
.mode-badge.cut-hard {
  background: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.controls {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-bottom: 16px;
}
.btn-start { background: rgba(28, 120, 60, 0.9) !important; }
.btn-pause { background: rgba(180, 130, 0, 0.88) !important; }
.btn-pause.is-resume {
  background: rgba(28, 120, 60, 0.9) !important;
  animation: resumePulse 1.1s ease-in-out infinite alternate;
}
@keyframes resumePulse {
  from { box-shadow: 0 0 0 0 rgba(28,120,60,0.5); }
  to   { box-shadow: 0 0 0 8px rgba(28,120,60,0);  }
}
.btn-quit { background: rgba(140, 40, 40, 0.85) !important; }

.timer-bar {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: rgba(255,255,255,0.5);
  border-radius: 30px;
  padding: 6px 20px;
  margin-bottom: 24px;
}
.timer-value {
  font-size: 1.2rem;
  font-weight: 800;
  color: #1a3322;
}
.timer-live-dot {
  width: 8px;
  height: 8px;
  background: #2ecc71;
  border-radius: 50%;
  animation: livePulse 1s ease-in-out infinite;
}
@keyframes livePulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50%       { opacity: 0.4; transform: scale(0.6); }
}

/* CORE CANVAS MATRIX ENGINE */
.puzzle-grid-canvas-frame {
  display: flex;
  flex-wrap: wrap;
  width: 100%;
  max-width: 650px;
  margin: 10px auto 25px;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 20px 50px rgba(0,0,0,0.22);
  background-color: #000; 
}
.tile-panel {
  flex: 0 0 33.3333%;
  max-width: 33.3333%;
  aspect-ratio: 1 / 1;
  box-sizing: border-box;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  margin: 0;
  padding: 0;
  border: none; 
}
.tile-panel img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  margin: 0;
  padding: 0;
  border: none;
}
.tile-panel.tile-selected img {
  filter: brightness(1.22) contrast(1.1);
}
.tile-select-ring {
  position: absolute;
  inset: 0;
  border: 6px solid #FFD700;
  box-shadow: inset 0 0 22px rgba(255,215,0,0.7);
  pointer-events: none;
}
.tile-panel.tile-swapped img {
  animation: tileSwapFlash 0.4s ease forwards;
}
@keyframes tileSwapFlash {
  0% { filter: brightness(1.7); }
  100% { filter: brightness(1); }
}

/* OVERLAYS */
.paused-overlay {
  position: absolute;
  inset: 0;
  z-index: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(8, 22, 12, 0.74);
  backdrop-filter: blur(8px);
  border-radius: 20px;
}
.paused-content {
  text-align: center;
  color: white;
}
.paused-icon {
  font-size: 3.5rem;
  margin-bottom: 6px;
  animation: telemetryPulse 1.6s ease-in-out infinite;
}
@keyframes telemetryPulse {
  0%, 100% { transform: scale(1); opacity: 0.9; }
  50% { transform: scale(1.12); opacity: 1; }
}
.paused-title {
  font-size: 2.6rem;
  font-weight: 900;
  color: #FFD700;
  letter-spacing: 0.05em;
  margin: 0 0 4px;
}
.paused-hint {
  font-size: 0.88rem;
  color: rgba(255,255,255,0.6);
  margin: 0 0 20px;
}
.resume-btn {
  background: linear-gradient(135deg, #2ecc71 0%, #27ae60 100%) !important;
  box-shadow: 0 6px 20px rgba(46,204,113,0.3) !important;
  border-radius: 30px !important;
  padding: 11px 28px !important;
}

.golden-buzzer-overlay {
  position: absolute;
  inset: 0;
  z-index: 999;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle, rgba(255,160,0,0.96) 0%, rgba(160,30,0,0.98) 100%);
  border-radius: 20px;
  overflow: hidden;
}
.buzzer-content-centered {
  position: relative;
  z-index: 10;
  text-align: center;
  background: rgba(255, 255, 255, 0.15);
  border: 2px solid rgba(255, 255, 255, 0.25);
  border-radius: 24px;
  padding: 40px 45px;
  backdrop-filter: blur(10px);
  box-shadow: 0 25px 55px rgba(0,0,0,0.35);
  max-width: 86%;
  width: 440px;
}
.buzzer-trophy {
  font-size: 4.8rem;
  line-height: 1;
  margin-bottom: 10px;
}
.win-title {
  font-size: 3.5rem;
  font-weight: 900;
  color: #fff;
  text-shadow: 0 4px 12px rgba(0,0,0,0.3);
  margin: 0;
}
.win-subtitle {
  font-size: 1.25rem;
  color: #fff;
  margin: 6px 0 24px;
  font-weight: 600;
}
.win-stats {
  display: flex;
  gap: 14px;
  justify-content: center;
  margin-bottom: 28px;
}
.win-stat-item {
  font-size: 1.05rem;
  color: #fff;
  background: rgba(0,0,0,0.22);
  padding: 8px 18px;
  border-radius: 20px;
  font-weight: 700;
}
.win-action-buttons {
  display: flex;
  gap: 12px;
  justify-content: center;
  flex-wrap: wrap;
  margin-top: 20px;
}
.play-again-btn, .next-level-btn {
  display: inline-block;
  font-size: 1.05rem !important;
  padding: 12px 28px !important;
  font-weight: 900 !important;
  border: none !important;
  border-radius: 50px !important;
  box-shadow: 0 6px 26px rgba(0,0,0,0.22) !important;
  min-width: 140px !important;
  transition: transform 0.2s ease, box-shadow 0.2s ease !important;
  cursor: pointer;
}
.play-again-btn {
  background: rgba(255,255,255,0.95) !important;
  color: #b84400 !important;
}
.next-level-btn {
  background: linear-gradient(135deg, #FFD700 0%, #FFA500 100%) !important;
  color: #fff !important;
}
.play-again-btn:hover, .next-level-btn:hover {
  transform: scale(1.06) translateY(-2px) !important;
  box-shadow: 0 10px 30px rgba(0,0,0,0.3) !important;
}
.dismiss-hint {
  font-size: 0.72rem;
  color: rgba(255,255,255,0.45);
  margin-top: 16px;
  letter-spacing: 0.06em;
}

/* ANIMATIONS ENGINE KEYFRAMES */
.rain-buzzer {
  position: absolute;
  pointer-events: none;
  animation: buzzerRainFall linear infinite;
}
@keyframes buzzerRainFall {
  0% { transform: translateY(-70px) rotate(0deg); opacity: 1; }
  100% { transform: translateY(750px) rotate(var(--spin, 360deg)); opacity: 0; }
}
.confetti-particle {
  position: absolute;
  pointer-events: none;
  animation: confettiFall linear infinite;
}
@keyframes confettiFall {
  0% { transform: translateY(-30px) rotate(0deg); opacity: 1; }
  100% { transform: translateY(750px) rotate(720deg); opacity: 0; }
}

.buzzer-fade-enter-active { animation: buzzerExpand 0.5s cubic-bezier(0.22,1,0.36,1) forwards; }
.buzzer-fade-leave-active { animation: buzzerExpand 0.35s cubic-bezier(0.22,1,0.36,1) reverse forwards; }
@keyframes buzzerExpand {
  0% { opacity: 0; transform: scale(0.85); }
  100% { opacity: 1; transform: scale(1); }
}

.pause-fade-enter-active, .pause-fade-leave-active { transition: opacity 0.35s ease; }
.pause-fade-enter, .pause-fade-leave-to { opacity: 0; }

@media (max-width: 768px) {
  .puzzle-grid-canvas-frame { max-width: 85vw; }
}
@media (max-width: 480px) {
  .puzzle-grid-canvas-frame { max-width: 94vw; }
  .game-title { font-size: 1.6rem; }
  .buzzer-content-centered { padding: 30px 20px; }
  .win-title { font-size: 2.6rem; }
}
</style>