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
        :key="index"
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
      this.records = JSON.parse(localStorage.getItem("records")) || [];
    },
    clearRecords() {
      localStorage.removeItem("records");
      this.records = [];
    },
  },
};
</script>

<style scoped>
.records-wrapper {
  margin: 15px auto 5px;
  max-width: 380px;
  background: rgba(255,255,255,0.4);
  border-radius: 16px;
  padding: 12px 20px;
  border: 1px solid rgba(255,255,255,0.5);
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
}
button.update-button { background: rgba(45, 106, 79, 0.8); }
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
  max-height: 140px;
  overflow-y: auto;
}

.record-row {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 12px;
  background: rgba(255,255,255,0.5);
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.5);
}

.record-rank {
  font-size: 1rem;
  min-width: 24px;
}

.record-time {
  font-size: 0.9rem;
  font-weight: 700;
  color: #1e3322;
}

.record-moves {
  font-size: 0.78rem;
  color: #5a7a64;
}

.record-list-enter-active { transition: all 0.3s ease; }
.record-list-leave-active  { transition: all 0.2s ease; }
.record-list-enter         { opacity: 0; transform: translateX(-10px); }
.record-list-leave-to      { opacity: 0; transform: translateX(10px); }
</style>