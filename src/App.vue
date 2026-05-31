<template>
  <div id="app-wrapper">
    <div id="app">
      <h1>Dieu Merci @Puzzle Game</h1>
      <PuzzlePage @puzzle-changed="($event) => (selectedPuzzleID = $event)" />
      <RecordsPage />
      <SliderPuzzle 
        :puzzleId="selectedPuzzleID" 
        @next-level="($event) => (selectedPuzzleID = $event)"
      />
    </div>
  </div>
</template>

<script>
import SliderPuzzle from "./components/SliderPuzzle.vue";
import PuzzlePage from "./components/Puzzles.vue";
import RecordsPage from "./components/Records.vue";

export default {
  name: "App",
  components: {
    SliderPuzzle,
    PuzzlePage,
    RecordsPage,
  },
  data() {
    return { selectedPuzzleID: "cut-easy" };
  },
};
</script>

<style>
/* Smooth core green-white gradient backdrop */
body, html {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5fafd 0%, #e3f2e6 50%, #ccead4 100%);
  background-attachment: fixed;
}

#app-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  box-sizing: border-box;
}

/* Core White-Green Glassmorphism Card Frame */
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #1e3322;
  
  width: 100%;
  max-width: 1100px;
  padding: 40px 20px;
  
  /* Glassmorphism Specs */
  background: rgba(255, 255, 255, 0.45); 
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.35);
  box-shadow: 0 12px 40px rgba(31, 66, 45, 0.08);
}

h1 {
  font-weight: 800;
  font-size: 1.8rem;
  color: #2d6a4f;
  margin-top: 0;
  margin-bottom: 20px;
  letter-spacing: -0.01em;
}

/* Base button UI definitions */
button {
  padding: 11px 22px;
  min-width: 135px;
  background: rgba(45, 106, 79, 0.85);
  color: white;
  font-weight: bold;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  cursor: pointer;
  transition: background-color 0.2s, transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.06);
}
button:hover {
  background: rgba(45, 106, 79, 0.98);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}
button:active {
  transform: translateY(0);
}
</style>