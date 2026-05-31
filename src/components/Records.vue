<template>
  <div class="records-wrapper">
    <h3>🏅 Speed Records</h3>

    <div class="records-buttons">
      <button class="update-button" @click="getRecords">🔄 Refresh</button>
      <button class="clear-button"  @click="clearRecords">🗑️ Clear Records</button>
    </div>

    <div v-if="records.length === 0" class="no-records">
      No records yet — finish a puzzle to set one!
    </div>

    <transition-group name="record-list" tag="div" class="records-list">
      <div
        v-for="(r, index) of records"
        :key="r.elapsedTime + '-' + index"
        class="record-row"
      >
        <span class="record-rank">
          {{ index === 0 ? '🥇' : index === 1 ? '🥈' : index === 2 ? '🥉' : `#${index + 1}` }}
        </span>
        <span class="record-time">{{ r.elapsedTime }}</span>
        <span class="record-moves" v-if="r.moveCount">({{ r.moveCount }} moves)</span>
      </div>
    </transition-group>
  </div>
</template>

<script>
export default {
  name: "RecordsComponent",
  data() {
    return {
      records: [],
    };
  },
  created() {
    this.getRecords();
  },
  methods: {
    getRecords() {
      try {
        const data = localStorage.getItem("puzzle_records");
        this.records = data ? JSON.parse(data) : [];
      } catch (e) {
        console.error("Failed to parse local records storage:", e);
        this.records = [];
      }
    },
    clearRecords() {
      if (confirm("Are you sure you want to delete all historical puzzle times?")) {
        localStorage.removeItem("puzzle_records");
        this.records = [];
      }
    }
  }
};
</script>

<style scoped>
.records-wrapper {
  background: rgba(255, 255, 255, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 16px;
  padding: 15px;
  max-width: 320px;
  margin: 0 auto 25px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.03);
}

h3 {
  margin: 0 0 10px;
  font-size: 1rem;
  color: #1e3322;
}

.records-buttons {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-bottom: 10px;
}

button {
  min-width: unset !important;
  border-radius: 20px !important;
  padding: 5px 14px !important;
  font-size: 0.8rem !important;
  border: none !important;
  cursor: pointer;
  color: white;
  transition: background 0.2s ease;
}
button.update-button { background: rgba(45, 106, 79, 0.8); }
button.update-button:hover { background: rgba(45, 106, 79, 1); }
button.clear-button { background: rgba(160, 50, 50, 0.8); }
button.clear-button:hover { background: rgba(190, 40, 40, 0.95) !important; }

.no-records {
  font-size: 0.8rem;
  color: #5a7a64;
  padding: 5px 0;
  font-style: italic;
}

.records-list {
  display: flex;
  flex-direction: column;
  gap: 5px;
  max-height: 150px;
  overflow-y: auto;
  padding-right: 4px;
}

/* Custom minimal scrollbar styling */
.records-list::-webkit-scrollbar { width: 4px; }
.records-list::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.1); border-radius: 4px; }

.record-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.7);
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 0.85rem;
  border: 1px solid rgba(0,0,0,0.02);
}

.record-rank { font-weight: bold; width: 30px; text-align: left; }
.record-time { font-family: monospace; color: #2d6a4f; font-weight: 600; flex-grow: 1; text-align: left; }
.record-moves { font-size: 0.75rem; color: #7a9a84; }

/* Transition Animations for List Additions */
.record-list-enter-active, .record-list-leave-active {
  transition: all 0.4s ease;
}
.record-list-enter {
  opacity: 0;
  transform: translateY(-10px);
}
.record-list-leave-to {
  opacity: 0;
  transform: scale(0.9);
}
</style>