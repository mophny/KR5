<template>
  <div class="filters">
    <button
      v-for="f in filters"
      :key="f"
      :class="{ active: currentFilter === f }"
      @click="$emit('update-filter', f)"
    >
      {{ labels[f] }}
    </button>
  </div>
</template>

<script>
export default {
  name: "TaskFilter",
  props: {
    currentFilter: { type: String, required: true }
  },
  emits: ["update-filter"],

  setup() {
    const filters = ["all", "active", "completed"];
    const labels = {
      all: "Все",
      active: "Активные",
      completed: "Завершённые"
    };

    return { filters, labels };
  }
};
</script>

<style scoped>
.filters {
  display: flex;
  gap: 10px;
  margin: 10px 0 20px;
}
button {
  padding: 6px 10px;
  border: 1px solid #ccc;
  background: white;
  cursor: pointer;
  border-radius: 6px;
}
button.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}
</style>
