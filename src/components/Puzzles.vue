<template>
  <div class="options-container">
    <h3>Select a Puzzle Challenge</h3>
    <div class="card">
      <div v-for="p of puzzles" :key="p.id" class="row">
        <div>
          <img :src="require(`../assets/${p.image}`)" />
        </div>
        <div>
          <h2>{{ p.title }}</h2>
        </div>
        <div class="play-button">
          <button @click="selectPuzzle(p)">Play</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Puzzle-component",
  data() {
    return {
      puzzles: [
        {
          id: "cut-easy",
          image: "easy.jpg",
          title: "Easy",
        },
        {
          id: "cut-medium",
          image: "medium.jpg",
          title: "Medium",
        },
        {
          id: "cut-hard",
          image: "hard.jpg",
          title: "Hard",
        },
      ],
    };
  },
  methods: {
    selectPuzzle(puzzle) {
      console.log(puzzle)
      this.$emit("puzzle-changed", puzzle.id);
    },
  },
};
</script>

<style scoped>
.row {
  display: flex;
  max-width: 90vw;
  flex-wrap: wrap;
  flex-direction: column;
  justify-content: space-between;
  background: rgba(255, 0, 0, 0.093);
  margin: auto;
  padding: 20px;
}
.card {
  display: flex;
  max-width: 90vw;
  flex-wrap: wrap;
  justify-content: space-between;
}
.row img {
  width: 300px;
  height: auto;
}

.row .play-button {
  padding-top: 25px;
}
.play-buttom {
  display: flex;
  align-items: center;
  padding: 10px;
}
@media (max-width: 768px) {
  .row img {
    width: 100px;
    height: auto;
  }
}
@media (max-width: 350px) {
  .row {
    flex-wrap: nowrap;
  }
  h2 {
    font-size: 1em;
  }
  button {
    width: 100px;
  }
}
</style>